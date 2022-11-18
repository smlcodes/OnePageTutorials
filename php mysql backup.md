https://github.com/daniloaz/myphp-backup/blob/master/myphp-backup.php

```php
<?php 
/**
 * This file contains the Backup_Database class wich performs
   $dbhost = 'sql301.epizy.com';
   $dbuser = 'epiz_32875882';
   $dbpass = 'AF6tE0XjpZUX';
   $dbname = "epiz_32875882_javabo"; 
 * a partial or complete backup of any given MySQL database
 * @author Daniel López Azaña <daniloaz@gmail.com>
 * @version 1.0
 */

/**
 * Define database parameters here
 */
define("DB_USER", 'epiz_32875882');
define("DB_PASSWORD", 'AF6tE0XjpZUX');
define("DB_NAME", 'epiz_32875882_javabo');
define("DB_HOST", 'sql301.epizy.com');
//define("BACKUP_DIR", 'myphp-backup-files'); // Comment this line to use same script's directory ('.')
define("TABLES", '*'); // Full backup
//define("TABLES", 'table1, table2, table3'); // Partial backup
define('IGNORE_TABLES',array(
    'tbl_token_auth',
    'token_auth'
)); // Tables to ignore
define("CHARSET", 'utf8');
define("GZIP_BACKUP_FILE", false); // Set to false if you want plain SQL backup files (not gzipped)
define("DISABLE_FOREIGN_KEY_CHECKS", true); // Set to true if you are having foreign key constraint fails
define("BATCH_SIZE", 1000); // Batch size when selecting rows from database in order to not exhaust system memory
                            // Also number of rows per INSERT statement in backup file

/**
 * The Backup_Database class
 */
class Backup_Database {
    /**
     * Host where the database is located
     */
    var $host;

    /**
     * Username used to connect to database
     */
    var $username;

    /**
     * Password used to connect to database
     */
    var $passwd;

    /**
     * Database to backup
     */
    var $dbName;

    /**
     * Database charset
     */
    var $charset;

    /**
     * Database connection
     */
    var $conn;

    /**
     * Backup directory where backup files are stored 
     */
    var $backupDir;

    /**
     * Output backup file
     */
    var $backupFile;

    /**
     * Use gzip compression on backup file
     */
    var $gzipBackupFile;

    /**
     * Content of standard output
     */
    var $output;

    /**
     * Disable foreign key checks
     */
    var $disableForeignKeyChecks;

    /**
     * Batch size, number of rows to process per iteration
     */
    var $batchSize;

    /**
     * Constructor initializes database
     */
    public function __construct($host, $username, $passwd, $dbName, $charset = 'utf8') {
        $this->host                    = $host;
        $this->username                = $username;
        $this->passwd                  = $passwd;
        $this->dbName                  = $dbName;
        $this->charset                 = $charset;
        $this->conn                    = $this->initializeDatabase();
        $this->backupDir               = BACKUP_DIR ? BACKUP_DIR : '.';
        $this->backupFile              = 'myphp-backup-'.$this->dbName.'-'.date("Ymd_His", time()).'.sql';
        $this->gzipBackupFile          = defined('GZIP_BACKUP_FILE') ? GZIP_BACKUP_FILE : true;
        $this->disableForeignKeyChecks = defined('DISABLE_FOREIGN_KEY_CHECKS') ? DISABLE_FOREIGN_KEY_CHECKS : true;
        $this->batchSize               = defined('BATCH_SIZE') ? BATCH_SIZE : 1000; // default 1000 rows
        $this->output                  = '';
    }

    protected function initializeDatabase() {
        try {
            $conn = mysqli_connect($this->host, $this->username, $this->passwd, $this->dbName);
            if (mysqli_connect_errno()) {
                throw new Exception('ERROR connecting database: ' . mysqli_connect_error());
                die();
            }
            if (!mysqli_set_charset($conn, $this->charset)) {
                mysqli_query($conn, 'SET NAMES '.$this->charset);
            }
        } catch (Exception $e) {
            print_r($e->getMessage());
            die();
        }

        return $conn;
    }

    /**
     * Backup the whole database or just some tables
     * Use '*' for whole database or 'table1 table2 table3...'
     * @param string $tables
     */
    public function backupTables($tables = '*') {
        try {
            /**
             * Tables to export
             */
            if($tables == '*') {
                $tables = array();
                $result = mysqli_query($this->conn, 'SHOW TABLES');
                while($row = mysqli_fetch_row($result)) {
                    $tables[] = $row[0];
                }
            } else {
                $tables = is_array($tables) ? $tables : explode(',', str_replace(' ', '', $tables));
            }

            $sql = 'CREATE DATABASE IF NOT EXISTS `'.$this->dbName.'`'.";\n\n";
            $sql .= 'USE `'.$this->dbName."`;\n\n";

            /**
             * Disable foreign key checks 
             */
            if ($this->disableForeignKeyChecks === true) {
                $sql .= "SET foreign_key_checks = 0;\n\n";
            }

            /**
             * Iterate tables
             */
            foreach($tables as $table) {
                if( in_array($table, IGNORE_TABLES) )
                    continue;
                $this->obfPrint("Backing up `".$table."` table...".str_repeat('.', 50-strlen($table)), 0, 0);

                /**
                 * CREATE TABLE
                 */
                $sql .= 'DROP TABLE IF EXISTS `'.$table.'`;';
                $row = mysqli_fetch_row(mysqli_query($this->conn, 'SHOW CREATE TABLE `'.$table.'`'));
                $sql .= "\n\n".$row[1].";\n\n";

                /**
                 * INSERT INTO
                 */

                $row = mysqli_fetch_row(mysqli_query($this->conn, 'SELECT COUNT(*) FROM `'.$table.'`'));
                $numRows = $row[0];

                // Split table in batches in order to not exhaust system memory 
                $numBatches = intval($numRows / $this->batchSize) + 1; // Number of while-loop calls to perform

                for ($b = 1; $b <= $numBatches; $b++) {
                    
                    $query = 'SELECT * FROM `' . $table . '` LIMIT ' . ($b * $this->batchSize - $this->batchSize) . ',' . $this->batchSize;
                    $result = mysqli_query($this->conn, $query);
                    $realBatchSize = mysqli_num_rows ($result); // Last batch size can be different from $this->batchSize
                    $numFields = mysqli_num_fields($result);

                    if ($realBatchSize !== 0) {
                        $sql .= 'INSERT INTO `'.$table.'` VALUES ';

                        for ($i = 0; $i < $numFields; $i++) {
                            $rowCount = 1;
                            while($row = mysqli_fetch_row($result)) {
                                $sql.='(';
                                for($j=0; $j<$numFields; $j++) {
                                    if (isset($row[$j])) {
                                        $row[$j] = addslashes($row[$j]);
                                        $row[$j] = str_replace("\n","\\n",$row[$j]);
                                        $row[$j] = str_replace("\r","\\r",$row[$j]);
                                        $row[$j] = str_replace("\f","\\f",$row[$j]);
                                        $row[$j] = str_replace("\t","\\t",$row[$j]);
                                        $row[$j] = str_replace("\v","\\v",$row[$j]);
                                        $row[$j] = str_replace("\a","\\a",$row[$j]);
                                        $row[$j] = str_replace("\b","\\b",$row[$j]);
                                        if ($row[$j] == 'true' or $row[$j] == 'false' or preg_match('/^-?[1-9][0-9]*$/', $row[$j]) or $row[$j] == 'NULL' or $row[$j] == 'null') {
                                            $sql .= $row[$j];
                                        } else {
                                            $sql .= '"'.$row[$j].'"' ;
                                        }
                                    } else {
                                        $sql.= 'NULL';
                                    }
    
                                    if ($j < ($numFields-1)) {
                                        $sql .= ',';
                                    }
                                }
    
                                if ($rowCount == $realBatchSize) {
                                    $rowCount = 0;
                                    $sql.= ");\n"; //close the insert statement
                                } else {
                                    $sql.= "),\n"; //close the row
                                }
    
                                $rowCount++;
                            }
                        }
    
                        $this->saveFile($sql);
                        $sql = '';
                    }
                }

                /**
                 * CREATE TRIGGER
                 */

                // Check if there are some TRIGGERS associated to the table
                /*$query = "SHOW TRIGGERS LIKE '" . $table . "%'";
                $result = mysqli_query ($this->conn, $query);
                if ($result) {
                    $triggers = array();
                    while ($trigger = mysqli_fetch_row ($result)) {
                        $triggers[] = $trigger[0];
                    }
                    
                    // Iterate through triggers of the table
                    foreach ( $triggers as $trigger ) {
                        $query= 'SHOW CREATE TRIGGER `' . $trigger . '`';
                        $result = mysqli_fetch_array (mysqli_query ($this->conn, $query));
                        $sql.= "\nDROP TRIGGER IF EXISTS `" . $trigger . "`;\n";
                        $sql.= "DELIMITER $$\n" . $result[2] . "$$\n\nDELIMITER ;\n";
                    }

                    $sql.= "\n";

                    $this->saveFile($sql);
                    $sql = '';
                }*/
 
                $sql.="\n\n";

                $this->obfPrint('OK');
            }

            /**
             * Re-enable foreign key checks 
             */
            if ($this->disableForeignKeyChecks === true) {
                $sql .= "SET foreign_key_checks = 1;\n";
            }

            $this->saveFile($sql);

            if ($this->gzipBackupFile) {
                $this->gzipBackupFile();
            } else {
                $this->obfPrint('Backup file succesfully saved to ' . $this->backupDir.'/'.$this->backupFile, 1, 1);
            }
        } catch (Exception $e) {
            print_r($e->getMessage());
            return false;
        }

        return true;
    }

    /**
     * Save SQL to file
     * @param string $sql
     */
    protected function saveFile(&$sql) {
        if (!$sql) return false;

        try {

            if (!file_exists($this->backupDir)) {
                mkdir($this->backupDir, 0777, true);
            }

            file_put_contents($this->backupDir.'/'.$this->backupFile, $sql, FILE_APPEND | LOCK_EX);

        } catch (Exception $e) {
            print_r($e->getMessage());
            return false;
        }

        return true;
    }

    /*
     * Gzip backup file
     *
     * @param integer $level GZIP compression level (default: 9)
     * @return string New filename (with .gz appended) if success, or false if operation fails
     */
    protected function gzipBackupFile($level = 9) {
        if (!$this->gzipBackupFile) {
            return true;
        }

        $source = $this->backupDir . '/' . $this->backupFile;
        $dest =  $source . '.gz';

        $this->obfPrint('Gzipping backup file to ' . $dest . '... ', 1, 0);

        $mode = 'wb' . $level;
        if ($fpOut = gzopen($dest, $mode)) {
            if ($fpIn = fopen($source,'rb')) {
                while (!feof($fpIn)) {
                    gzwrite($fpOut, fread($fpIn, 1024 * 256));
                }
                fclose($fpIn);
            } else {
                return false;
            }
            gzclose($fpOut);
            if(!unlink($source)) {
                return false;
            }
        } else {
            return false;
        }
        
        $this->obfPrint('OK');
        return $dest;
    }

    /**
     * Prints message forcing output buffer flush
     *
     */
    public function obfPrint ($msg = '', $lineBreaksBefore = 0, $lineBreaksAfter = 1) {
        if (!$msg) {
            return false;
        }

        if ($msg != 'OK' and $msg != 'KO') {
            $msg = date("Y-m-d H:i:s") . ' - ' . $msg;
        }
        $output = '';

        if (php_sapi_name() != "cli") {
            $lineBreak = "<br />";
        } else {
            $lineBreak = "\n";
        }

        if ($lineBreaksBefore > 0) {
            for ($i = 1; $i <= $lineBreaksBefore; $i++) {
                $output .= $lineBreak;
            }                
        }

        $output .= $msg;

        if ($lineBreaksAfter > 0) {
            for ($i = 1; $i <= $lineBreaksAfter; $i++) {
                $output .= $lineBreak;
            }                
        }


        // Save output for later use
        $this->output .= str_replace('<br />', '\n', $output);

        echo $output;


        if (php_sapi_name() != "cli") {
            if( ob_get_level() > 0 ) {
                ob_flush();
            }
        }

        $this->output .= " ";

        flush();
    }

    /**
     * Returns full execution output
     *
     */
    public function getOutput() {
        return $this->output;
    }
    /**
     * Returns name of backup file
     *
     */
    public function getBackupFile() {
        if ($this->gzipBackupFile) {
            return $this->backupDir.'/'.$this->backupFile.'.gz';
        } else
        return $this->backupDir.'/'.$this->backupFile;
    }

    /**
     * Returns backup directory path
     *
     */
    public function getBackupDir() {
        return $this->backupDir;
    }

    /**
     * Returns array of changed tables since duration
     *
     */
    public function getChangedTables($since = '1 day') {
        $query = "SELECT TABLE_NAME,update_time FROM information_schema.tables WHERE table_schema='" . $this->dbName . "'";

        $result = mysqli_query($this->conn, $query);
        while($row=mysqli_fetch_assoc($result)) {
            $resultset[] = $row;
            }		
        if(empty($resultset))
            return false;
        $tables = [];
        for ($i=0; $i < count($resultset); $i++) {
            if( in_array($resultset[$i]['TABLE_NAME'], IGNORE_TABLES) ) // ignore this table
                continue;
            if(strtotime('-' . $since) < strtotime($resultset[$i]['update_time']))
                $tables[] = $resultset[$i]['TABLE_NAME'];
        }
        return ($tables) ? $tables : false;
    }
}


/**
 * Instantiate Backup_Database and perform backup
 */

// Report all errors
error_reporting(E_ALL);
// Set script max execution time
set_time_limit(900); // 15 minutes

if (php_sapi_name() != "cli") {
    echo '<div style="font-family: monospace;">';
}

$backupDatabase = new Backup_Database(DB_HOST, DB_USER, DB_PASSWORD, DB_NAME, CHARSET);

// Option-1: Backup tables already defined above
$result = $backupDatabase->backupTables(TABLES) ? 'OK' : 'KO';

// Option-2: Backup changed tables only - uncomment block below
/*
$since = '1 day';
$changed = $backupDatabase->getChangedTables($since);
if(!$changed){
  $backupDatabase->obfPrint('No tables modified since last ' . $since . '! Quitting..', 1);
  die();
}
$result = $backupDatabase->backupTables($changed) ? 'OK' : 'KO';
*/


$backupDatabase->obfPrint('Backup result: ' . $result, 1);

// Use $output variable for further processing, for example to send it by email
$output = $backupDatabase->getOutput();

if (php_sapi_name() != "cli") {
    echo '</div>';
}

```

# Google DRIVE PHP integration

Before getting started to build a PHP script to **upload file to Google drive** using PHP, take a look at the file structure.

```
google_drive_file_upload_with_php/
├── config.php
├── dbConfig.php
├── index.php
├── upload.php
├── google_drive_sync.php
├── GoogleDriveApi.class.php
└── css/
    └── style.css
```


Create Google Project and Enable Drive API
------------------------------------------

Google Project is required to get the API keys that will be used to make API calls to Google Drive. If you already have an existing Google Application, API keys can be used from it. Just make sure that the Google Drive API is enabled in this existing Google project. If you don't any Google applications, follow the below steps to register your application on Google Developers Console and get the API keys.

-   Go to the [Google API Console](https://console.developers.google.com/).
-   Select an existing project from the projects list, or click **NEW PROJECT** to create a new project:

    ![google-api-console-project-create-codexworld](https://www.codexworld.com/wp-content/uploads/2022/01/google-api-console-project-create-codexworld-1024x362.png)

    -   Type the **Project Name**.
    -   The project ID will be created automatically under the Project Name field. (Optional) You can change this project ID by the **Edit** link, but it should be unique worldwide.
    -   Click the **CREATE** button.
-   Select the newly created project and enable the **Google Drive API** service.
    -   In the sidebar, select **Library** under the **APIs & Services** section.
    -   Search for the Google Drive API service in the API list and select **Google Drive API**.
    -   Click the **ENABLE** button to make the Google Drive API Library available.
-   In the sidebar, select **Credentials** under the **APIs & Services** section.
-   Select the **OAuth consent screen** tab, specify the consent screen settings.
    -   Enter the Application name.
    -   Choose a Support email.
    -   Specify the **Authorized domains** which will be allowed to authenticate using OAuth.
    -   Click the **Save** button.
-   Select the **Credentials** tab, click the Create credentials drop-down and select OAuth client ID.
    -   In the **Application type** section, select **Web application**.
    -   In the **Authorized redirect URIs** field, specify the redirect URL.
    -   Click the **Create** button.

A dialog box will appear with OAuth client details, note the Client ID and Client secret for later use in the script. This Client ID and Client secret allow you to access the Google Drive API.

![google-api-console-project-oauth-api-key-client-id-secret-codexworld](https://www.codexworld.com/wp-content/uploads/2022/01/google-api-console-project-oauth-api-key-client-id-secret-codexworld-1024x551.png)

Note that: The **Client ID** and **Client secret** need to be specified at the time of the Google Drive API call. Also, the Authorized redirect URIs must be matched with the Redirect URL specified in the script.

Create Database Table
---------------------

A table is required in the database to store file information on the local server. The following SQL creates a `drive_files` table with some basic fields in the MySQL database.
```
CREATE TABLE `drive_files` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `file_name` varchar(255) COLLATE utf8_unicode_ci DEFAULT NULL,
  `google_drive_file_id` varchar(255) COLLATE utf8_unicode_ci DEFAULT NULL,
  `created` datetime NOT NULL DEFAULT current_timestamp(),
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
```



Google Drive API PHP Library
----------------------------

Google provides a PHP client library to make Drive API calls, but it contains many additional services that come with a huge number of files and are large in size. To make the process simple, we will build a custom library to handle the Google Drive API calls with PHP.

Our **Google API Library** helps to authenticate with Google account and access the Drive API with REST API using PHP cURL. This custom PHP library will use **Google Drive v3 API** to handle the file upload process.

-   **GetAccessToken()** -- Fetch the access token from Google OAuth 2 API using authentication code.
-   **UploadFileToDrive()** -- Send file to Google Drive for upload using REST API.
-   **UpdateFileMeta()** -- Update metadata of the uploaded file in Google Drive.

```php
<?php
/**
 *
 * This Google Drive API handler class is a custom PHP library to handle the Google Drive API calls.
 *
 * @class        GoogleDriveApi
 * @author        CodexWorld
 * @link        http://www.codexworld.com
 * @version        1.0
 */
class GoogleDriveApi {
    const OAUTH2_TOKEN_URI = 'https://oauth2.googleapis.com/token';
    const DRIVE_FILE_UPLOAD_URI = 'https://www.googleapis.com/upload/drive/v3/files';
    const DRIVE_FILE_META_URI = 'https://www.googleapis.com/drive/v3/files/';

    function __construct($params = array()) {
        if (count($params) > 0){
            $this->initialize($params);
        }
    }

    function initialize($params = array()) {
        if (count($params) > 0){
            foreach ($params as $key => $val){
                if (isset($this->$key)){
                    $this->$key = $val;
                }
            }
        }
    }

    public function GetAccessToken($client_id, $redirect_uri, $client_secret, $code) {
        $curlPost = 'client_id=' . $client_id . '&redirect_uri=' . $redirect_uri . '&client_secret=' . $client_secret . '&code='. $code . '&grant_type=authorization_code';
        $ch = curl_init();
        curl_setopt($ch, CURLOPT_URL, self::OAUTH2_TOKEN_URI);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
        curl_setopt($ch, CURLOPT_POST, 1);
        curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, FALSE);
        curl_setopt($ch, CURLOPT_POSTFIELDS, $curlPost);
        $data = json_decode(curl_exec($ch), true);
        $http_code = curl_getinfo($ch,CURLINFO_HTTP_CODE);

        if ($http_code != 200) {
            $error_msg = 'Failed to receieve access token';
            if (curl_errno($ch)) {
                $error_msg = curl_error($ch);
            }
            throw new Exception('Error '.$http_code.': '.$error_msg);
        }

        return $data;
    }

    public function UploadFileToDrive($access_token, $file_content, $mime_type) {
        $apiURL = self::DRIVE_FILE_UPLOAD_URI . '?uploadType=media';

        $ch = curl_init();
        curl_setopt($ch, CURLOPT_URL, $apiURL);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
        curl_setopt($ch, CURLOPT_POST, 1);
        curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, FALSE);
        curl_setopt($ch, CURLOPT_HTTPHEADER, array('Content-Type: '.$mime_type, 'Authorization: Bearer '. $access_token));
        curl_setopt($ch, CURLOPT_POSTFIELDS, $file_content);
        $data = json_decode(curl_exec($ch), true);
        $http_code = curl_getinfo($ch,CURLINFO_HTTP_CODE);

        if ($http_code != 200) {
            $error_msg = 'Failed to upload file to Google Drive';
            if (curl_errno($ch)) {
                $error_msg = curl_error($ch);
            }
            throw new Exception('Error '.$http_code.': '.$error_msg);
        }

        return $data['id'];
    }

    public function UpdateFileMeta($access_token, $file_id, $file_meatadata) {
        $apiURL = self::DRIVE_FILE_META_URI . $file_id;

        $ch = curl_init();
        curl_setopt($ch, CURLOPT_URL, $apiURL);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
        curl_setopt($ch, CURLOPT_POST, 1);
        curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, FALSE);
        curl_setopt($ch, CURLOPT_HTTPHEADER, array('Content-Type: application/json', 'Authorization: Bearer '. $access_token));
        curl_setopt($ch, CURLOPT_CUSTOMREQUEST, 'PATCH');
        curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($file_meatadata));
        $data = json_decode(curl_exec($ch), true);
        $http_code = curl_getinfo($ch,CURLINFO_HTTP_CODE);

        if ($http_code != 200) {
            $error_msg = 'Failed to update file metadata';
            if (curl_errno($ch)) {
                $error_msg = curl_error($ch);
            }
            throw new Exception('Error '.$http_code.': '.$error_msg);
        }

        return $data;
    }
}
?>
```


Database and API Configuration (config.php)
-------------------------------------------

In the `config.php` file, database settings and Google API configuration constant variables are defined.

**Database constants:**

-   DB_HOST -- Specify the database host.
-   DB_USERNAME -- Specify the database username.
-   DB_PASSWORD -- Specify the database password.
-   DB_NAME -- Specify the database name.

**Google API constants:**

-   GOOGLE_CLIENT_ID -- Specify the Google Project Client ID.
-   GOOGLE_CLIENT_SECRET -- Specify the Google Project Client Secret.
-   GOOGLE_OAUTH_SCOPE -- Specify the OAuth scope for Google authentication (set to `https://www.googleapis.com/auth/drive`).
-   REDIRECT_URI -- Specify the Callback URL.

**Google OAuth URL:**

-   `$googleOauthURL` -- The URL that allows the user to authenticate with Google account.


```php
<?php
// Database configuration
define('DB_HOST', 'MySQL_Database_Host');
define('DB_USERNAME', 'MySQL_Database_Username');
define('DB_PASSWORD', 'MySQL_Database_Password');
define('DB_NAME', 'MySQL_Database_Name');

// Google API configuration
define('GOOGLE_CLIENT_ID', 'Google_Project_Client_ID');
define('GOOGLE_CLIENT_SECRET', 'Google_Project_Client_Secret');
define('GOOGLE_OAUTH_SCOPE', 'https://www.googleapis.com/auth/drive');
define('REDIRECT_URI', 'https://www.example.com/google_drive_sync.php');

// Start session
if(!session_id()) session_start();

// Google OAuth URL
$googleOauthURL = 'https://accounts.google.com/o/oauth2/auth?scope=' . urlencode(GOOGLE_OAUTH_SCOPE) . '&redirect_uri=' . REDIRECT_URI . '&response_type=code&client_id=' . GOOGLE_CLIENT_ID . '&access_type=online';

?>
```

Note that: The Client ID and Client Secret can be found on the Google API Manager page of the API Console project.

Database Connection (dbConfig.php)
----------------------------------

The `dbConfig.php` file is used to connect and select the MySQL database using PHP.
```
<?php
// Include configuration file
require_once 'config.php';

// Create database connection
$db = new mysqli(DB_HOST, DB_USERNAME, DB_PASSWORD, DB_NAME);

// Check connection
if ($db->connect_error) {
    die("Connection failed: " . $db->connect_error);
}
```


File Upload Form (index.php)
----------------------------

Create an HTML form to select file for upload any type of files (.jpg, .jpeg, .png, .pdf, .xlsx, .csv, etc).

-   The `enctype` attribute must be defined in the <form> tag to allow file upload.
-   On submission, the file data is posted to the server-side script (`upload.php`) for further processing.

```
<?php
// Include configuration file
include_once 'config.php';

$status = $statusMsg = '';
if(!empty($_SESSION['status_response'])){
    $status_response = $_SESSION['status_response'];
    $status = $status_response['status'];
    $statusMsg = $status_response['status_msg'];

    unset($_SESSION['status_response']);
}
?>

<!-- Status message -->
<?php if(!empty($statusMsg)){ ?>
    <div class="alert alert-<?php echo $status; ?>"><?php echo $statusMsg; ?></div>
<?php } ?>

<div class="col-md-12">
    <form method="post" action="upload.php" class="form" enctype="multipart/form-data">
        <div class="form-group">
            <label>File</label>
            <input type="file" name="file" class="form-control">
        </div>
        <div class="form-group">
            <input type="submit" class="form-control btn-primary" name="submit" value="Upload"/>
        </div>
    </form>
</div>

```


Store File in the Database (upload.php)
---------------------------------------

The `upload.php` file handles the [file upload in PHP](https://www.codexworld.com/php-file-upload/) and data insertion process to the MySQL database.

-   Get file info from the input field using **PHP $_FILES** variable.
-   Validate input to check whether mandatory fields are empty.
-   Upload file to the local server and insert file data in the database using PHP and MySQL.
-   Redirect user to the OAuth URL for authentication with Google account.

```
<?php
// Include database configuration file
require_once 'dbConfig.php';

$statusMsg = $valErr = '';
$status = 'danger';

// If the form is submitted
if(isset($_POST['submit'])){

    // Validate form input fields
    if(empty($_FILES["file"]["name"])){
        $valErr .= 'Please select a file to upload.<br/>';
    }

    // Check whether user inputs are empty
    if(empty($valErr)){
        $targetDir = "uploads/";
        $fileName = basename($_FILES["file"]["name"]);
        $targetFilePath = $targetDir . $fileName;

        // Upload file to local server
        if(move_uploaded_file($_FILES["file"]["tmp_name"], $targetFilePath)){

            // Insert data into the database
            $sqlQ = "INSERT INTO drive_files (file_name,created) VALUES (?,NOW())";
            $stmt = $db->prepare($sqlQ);
            $stmt->bind_param("s", $db_file_name);
            $db_file_name = $fileName;
            $insert = $stmt->execute();

            if($insert){
                $file_id = $stmt->insert_id;

                // Store DB reference ID of file in SESSION
                $_SESSION['last_file_id'] = $file_id;

                header("Location: $googleOauthURL");
                exit();
            }else{
                $statusMsg = 'Something went wrong, please try again after some time.';
            }
        }else{
            $statusMsg = 'File upload failed, please try again after some time.';
        }
    }else{
        $statusMsg = '<p>Please fill all the mandatory fields:</p>'.trim($valErr, '<br/>');
    }
}else{
    $statusMsg = 'Form submission failed!';
}

$_SESSION['status_response'] = array('status' => $status, 'status_msg' => $statusMsg);

header("Location: index.php");
exit();
?>
```



Upload File to Google Drive (google_drive_sync.php)
---------------------------------------------------

This script is set as Redirect URI in Google API configuration. This means after authentication with Google account, the user will be redirected to this script that handles the Google Drive file upload process with REST API using PHP.

-   Get OAuth code from the query string of the URL using **PHP $_GET** variable.
-   Include and initialize the Google Drive API handler class.
-   Get file reference ID of the local database from SESSION.
-   Fetch the file info from the database based on the reference ID.
-   Retrieve file from the server and define entire file into a string using **file_get_contents()** function in PHP.
-   Get MIME type of the file using **mime_content_type()** function in PHP.
-   Get access token by authentication code using `GetAccessToken()` function of the `GoogleDriveApi` class.
-   Send and upload file to Google Drive using `UploadFileToDrive()` function of the `GoogleDriveApi` class.
-   Update file meta data in Google Drive using the `UpdateFileMeta()` function of the `GoogleDriveApi` class.
-   Update google drive file reference ID in the database.
-   Display file upload status with Google drive file access link.


```php
<?php
// Include Google drive api handler class
include_once 'GoogleDriveApi.class.php';

// Include database configuration file
require_once 'dbConfig.php';

$statusMsg = '';
$status = 'danger';
if(isset($_GET['code'])){
    // Initialize Google Drive API class
    $GoogleDriveApi = new GoogleDriveApi();

    // Get file reference ID from SESSION
    $file_id = $_SESSION['last_file_id'];

    if(!empty($file_id)){

        // Fetch file details from the database
        $sqlQ = "SELECT * FROM drive_files WHERE id = ?";
        $stmt = $db->prepare($sqlQ);
        $stmt->bind_param("i", $db_file_id);
        $db_file_id = $file_id;
        $stmt->execute();
        $result = $stmt->get_result();
        $fileData = $result->fetch_assoc();

        if(!empty($fileData)){
            $file_name = $fileData['file_name'];
            $target_file = 'uploads/'.$file_name;
            $file_content = file_get_contents($target_file);
            $mime_type = mime_content_type($target_file);

            // Get the access token
            if(!empty($_SESSION['google_access_token'])){
                $access_token = $_SESSION['google_access_token'];
            }else{
                $data = $GoogleDriveApi->GetAccessToken(GOOGLE_CLIENT_ID, REDIRECT_URI, GOOGLE_CLIENT_SECRET, $_GET['code']);
                $access_token = $data['access_token'];
                $_SESSION['google_access_token'] = $access_token;
            }

            if(!empty($access_token)){

                try {
                    // Upload file to Google drive
                    $drive_file_id = $GoogleDriveApi->UploadFileToDrive($access_token, $file_content, $mime_type);

                    if($drive_file_id){
                        $file_meta = array(
                            'name' => basename($file_name)
                        );

                        // Update file metadata in Google drive
                        $drive_file_meta = $GoogleDriveApi->UpdateFileMeta($access_token, $drive_file_id, $file_meta);

                        if($drive_file_meta){
                            // Update google drive file reference in the database
                            $sqlQ = "UPDATE drive_files SET google_drive_file_id=? WHERE id=?";
                            $stmt = $db->prepare($sqlQ);
                            $stmt->bind_param("si", $db_drive_file_id, $db_file_id);
                            $db_drive_file_id = $drive_file_id;
                            $db_file_id = $file_id;
                            $update = $stmt->execute();

                            unset($_SESSION['last_file_id']);
                            unset($_SESSION['google_access_token']);

                            $status = 'success';
                            $statusMsg = '<p>File has been uploaded to Google Drive successfully!</p>';
                            $statusMsg .= '<p><a href="https://drive.google.com/open?id='.$drive_file_meta['id'].'" target="_blank">'.$drive_file_meta['name'].'</a>';
                        }
                    }
                } catch(Exception $e) {
                    $statusMsg = $e->getMessage();
                }
            }else{
                $statusMsg = 'Failed to fetch access token!';
            }
        }else{
            $statusMsg = 'File data not found!';
        }
    }else{
        $statusMsg = 'File reference not found!';
    }

    $_SESSION['status_response'] = array('status' => $status, 'status_msg' => $statusMsg);

    header("Location: index.php");
    exit();
}
?>

```



Checklist for Testing:
----------------------

**Google Application Verification:**
Google requires application verification to use Drive API. You need to submit the application for verification to make Google project public.

In the development mode, you can test it by adding Test Users in the OAuth consent screen of the Google application.

-   On the OAuth Client credentials page, click the **OAuth consent screen** link.

    ![google-api-console-oauth-client-credentials-codexworld](https://www.codexworld.com/wp-content/uploads/2022/01/google-api-console-oauth-client-credentials-codexworld.png)

-   In the OAuth consent screen page, add users under the **Test users** section.

    ![google-api-console-oauth-consent-test-user-codexworld](https://www.codexworld.com/wp-content/uploads/2022/01/google-api-console-oauth-consent-test-user-codexworld.png)





## Ref.

https://www.codexworld.com/upload-file-to-google-drive-using-php/

https://github.com/fareed543/Uploading-files-to-Google-Drive-with-PHP

https://www.youtube.com/watch?v=Af9Myjjz9yg


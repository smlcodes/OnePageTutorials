



# Flyway



<p id="gdcalert1" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image1.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert2">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image1.png "image_tooltip")




1. Creating Database Scripts 

<span style="text-decoration:underline;">Naming Convention</span>

Database Scripts consists of the following parts:



* **Prefix**: V for versioned ([configurable](https://flywaydb.org/documentation/configuration/parameters/sqlMigrationPrefix)), U for undo ([configurable](https://flywaydb.org/documentation/configuration/parameters/undoSqlMigrationPrefix)) and R for repeatable migrations ([configurable](https://flywaydb.org/documentation/configuration/parameters/repeatableSqlMigrationPrefix))
* **Version**: Version with dots or underscores separate as many parts as you like (Not for repeatable migrations)
* **Separator**: __ (two underscores) ([configurable](https://flywaydb.org/documentation/configuration/parameters/sqlMigrationSeparator))
* **Description**: Underscores or spaces separate the words
* **Suffix**: .sql ([configurable](https://flywaydb.org/documentation/configuration/parameters/sqlMigrationSuffixes))

 

<p id="gdcalert2" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image2.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert3">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image2.png "image_tooltip")


	

<span style="text-decoration:underline;">How Flyway works</span>

When ever we perfom **migration**, it will store the version history in CONTROL_TABLE



<p id="gdcalert3" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image3.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert4">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image3.png "image_tooltip")


<span style="text-decoration:underline;">Running Flyway</span>

Flyway supports a range of different options to run database migrations:



* via [command line](https://reflectoring.io/#command-line)
* via [Java API](https://reflectoring.io/#java-api),
* via **Maven** and [Gradle](https://reflectoring.io/#gradle-plugin) plugins, and
* via [community plugins and integrations](https://flywaydb.org/documentation/plugins/) including **[Spring Boot](https://reflectoring.io/#spring-boot-auto-configuration).**

<span style="text-decoration:underline;">Commands</span>

Flyway relies on seven commands to manage database version control.



* <code>migrate<strong>:</strong></code> Migrates the schema to the latest version. Flyway will create the schema history table automatically if it doesn’t exist.
* <code>clean<strong>:</strong></code> Drops all objects in the configured schemas.
* <code>info<strong>:</strong></code> Prints the details and status information about all the migrations.
* <code>validate<strong>:</strong></code> Validates the applied migrations against the available ones.
* <code>undo<strong>:</strong></code> Undoes the most recently applied versioned migration.
* <code>baseline<strong>:</strong></code> Baselines an existing database, excluding all migrations up to and including baselineVersion.
* <code>repair<strong>:</strong></code> Repairs the schema history table.

<strong><span style="text-decoration:underline;">Maven Build Commands </span></strong>


```
mvn flyway:migrate 

mvn clean install -DskipTests=true flyway:migrate
```


To perform Flyway migration at mvn build process itself, we need configure goals in flyway-maven-plugin


```
			<plugin>
				<groupId>org.flywaydb</groupId>
				<artifactId>flyway-maven-plugin</artifactId>
				<executions>
					<execution>
						<id>echo-intsall-phase</id>
						<phase>build/install/release</phase>
						<goals>
							<goal>migrate</goal>
						</goals>
					</execution>
				</executions>
				… 
			</plugin>
```


Now whenever we run `mvn install` command, Flyway migration automactally happen.



2. Flyway – Examples

<span style="text-decoration:underline;">Ex 1: SpringBoot Integration</span>

1.We need to Create SpringBoot project & add Flyway depency in pom.xml

		&lt;dependency>

			&lt;groupId>org.flywaydb&lt;/groupId>

			&lt;artifactId>flyway-core&lt;/artifactId>

		&lt;/dependency>

2. Create Database migration scripts under resources`/db/migration/` folder. We have created 3 scripts.

 

<p id="gdcalert4" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image4.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert5">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image4.png "image_tooltip")


3.Update `application.properties` with config. Details


```
#FlywayPOC Properties

spring.datasource.url=jdbc:postgresql://127.0.0.1:5432/flywaydb
spring.datasource.username=postgres
spring.datasource.password=postgres
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.hibernate.ddl-auto=update
spring.flyway.baseline-on-migrate = true

spring.mvc.view.prefix: /WEB-INF/jsp/
spring.mvc.view.suffix: .jsp
```


4. Start the application & it will execute scripts order by version numbers & saves history in `&lt;DATABASE>_schema_history` table



<p id="gdcalert5" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image5.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert6">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image5.png "image_tooltip")


<span style="text-decoration:underline;">Ex 2: Maven Plug-in Integration</span>

1. First, Disable flyway in SpringBoot


```
spring.flyway.enabled=false
```


2. Update Maven pom.xml-file in the build/plugins-section with Flyway Configuration details 


```
	<properties>
		<java.version>11</java.version>
		<database.url>jdbc:postgresql://localhost:5432/flywaydb</database.url>
		<database.user>postgres</database.user>
		<databese.password>postgres</databese.password>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.flywaydb</groupId>
			<artifactId>flyway-core</artifactId>
		</dependency>
		<dependency>
			<groupId>org.postgresql</groupId>
			<artifactId>postgresql</artifactId>
			<scope>runtime</scope>
		</dependency>
	</dependencies>

<plugin>
	<groupId>org.flywaydb</groupId>
	<artifactId>flyway-maven-plugin</artifactId>
	<executions>
		<execution>
			<id>echo-intsall-phase</id>
			<phase>install</phase>
			<goals>
				<goal>migrate</goal>
			</goals>
		</execution>
	</executions>
	<configuration>
		<sqlMigrationSeparator>__</sqlMigrationSeparator>
		<locations>
	<location>filesystem:src/main/resources/db/migration</location>
		</locations>
		<url>${database.url}</url>
		<user>${database.user}</user>
		<password>${databese.password}</password>
	</configuration>
	<dependencies>
		<dependency>
			<groupId>org.postgresql</groupId>
			<artifactId>postgresql</artifactId>
			<version>42.3.6</version>
		</dependency>
		</dependencies>
</plugin>
```


3.Run `mvn flyway:migrate` command to perform Flyway migration.

Or `mvn clean install`


```
satyakaveti@Satyas-MacBook-Pro-2 FlywayPOC % mvn flyway:migrate
[INFO] Scanning for projects...
[INFO] 
[INFO] -----------------------< com.dpworld:FlywayPOC >------------------------
[INFO] Building FlywayPOC 0.0.1-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- flyway-maven-plugin:8.0.5:migrate (default-cli) @ FlywayPOC ---
[INFO] Flyway Community Edition 8.0.5 by Redgate
[INFO] Database: jdbc:postgresql://localhost:5432/flywaydb (PostgreSQL 14.3)
[INFO] Successfully validated 3 migrations (execution time 00:00.010s)
[INFO] Creating Schema History table "public"."flyway_schema_history" ...
[INFO] Current version of schema "public": << Empty Schema >>
[INFO] Migrating schema "public" to version "1 - createTable"
[INFO] Migrating schema "public" to version "2 - updateTable"
[INFO] Migrating schema "public" to version "3 - insertData1"
[INFO] Successfully applied 3 migrations to schema "public", now at version v3 (execution time 00:00.042s)
[INFO] Flyway Community Edition 8.0.5 by Redgate
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  1.142 s
[INFO] Finished at: 2022-06-09T12:49:02+05:30
[INFO] ------------------------------------------------------------------------
```



# Liquibase



<p id="gdcalert6" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image6.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert7">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image6.png "image_tooltip")



## 1.Creation of Database Scripts 

<span style="text-decoration:underline;">How Liquibase works</span>

It is based on the concept of_ **changelogs** _and_ **changesets** _files which can be written in **SQL, XML, YAML, JSON**. 



<p id="gdcalert7" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image7.jpg). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert8">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image7.jpg "image_tooltip")


We need to create database scripts & place in `/resources/db/changelog `folder

 

We have mainly two files 



1. `ChangeSet` – `Database scripts with version information`
2. `ChangeLog `– `Scripts Execution order`

<span style="text-decoration:underline;">ChangeSets</span>


#### 1. XML ChangesetS


```
  <changeSet id="01" author="satya">


    <createTable tableName="users" remarks="A table to contain all users">
      <column name="id" type="int" autoIncrement="true">
        <constraints nullable="false" unique="true" primaryKey="true"/>
      </column>
      <column name="name" type="varchar(255)">
        <constraints nullable="false" unique="false"/>
      </column>
      <column name="email" type="varchar(255)">
        <constraints nullable="false"/>
      </column>
    </createTable>

  </changeSet>
```



####  2. SQL ChangesetS


```
--liquibase formatted sql
--changeset SQL_AUTHOR_Satya:04
INSERT INTO "public"."users" ("id", "city", "email", "mobile", "name") VALUES (201, 'HYDERABAD', 'aaa@dp.com', '78936 10980', 'SQL_AAA');

INSERT INTO "public"."users" ("id", "city", "email", "mobile", "name") VALUES (202, 'BANGLORE', 'bbb@dp.com', '78936 20980', 'SQL_BBB');

INSERT INTO "public"."users" ("id", "city", "email", "mobile", "name") VALUES (203, 'DELHI', 'ccc@dp.com', '78936 30980', 'SQL_CCC');
```



#### 3. YAML Changesets 


```
databaseChangeLog:
- changeSet:
    id: 1.0/06
    author: YAML_Auther_Satya
    comment: "Inserting YAML Data"
    changes:
     - insert:
         tableName: users
         columns:
         - column:
             name: id
             value: "601"
         - column:
             name: name
             value: "YAML_USER_1"
         - column:
             name: email
             value: "user1@yaml.com"
         - column:
             name: city
             value: "newYork"
```


<span style="text-decoration:underline;">ChangeLog</span>

We will define scripts execution order here


```
/LiquibasePOC/src/main/resources/db/changelog/db.changelog-master.yaml
databaseChangeLog:
  - include:
      file: db/changelog/xml/create-table-schema.xml
  - include:
      file: db/changelog/xml/update-table-schema.xml
  - include:
      file: db/changelog/xml/insert-data-xml.xml
  - include:
      file: db/changelog/sql/insert-data-sql.sql
  - include:
      file: db/changelog/yaml/insert-data-yaml.yaml
```


<span style="text-decoration:underline;">Commands</span>

`update`	: Updates database to current version.

`diff	`	: Writes description of differences between two databases to standard out.

`rollback`	: Rolls back the database to the state it was in when the tag was applied.

`history`	: Lists all deployed changesets and their deploymentIds.

`status	`: Outputs the count / list of changesets that have not been deployed.

`validate`	: Checks the changelog for errors.

`tag	`	: "Tags" the current database state for future rollback.

**<span style="text-decoration:underline;">Maven Build commands</span>**


```
mvn liquibase:update 
mvn clean install -DskipTests=true liquibase:update
```


To perform liquibase migration at mvn build process itself, we need configure goals in liquibase-maven-plugin


```
				<executions>
					<execution>
						<id>echo-intsall-phase</id>
						<phase>install</phase>
						<goals>
							<goal>update</goal>
						</goals>
					</execution>
				</executions>
```


Now whenever we run `mvn install` command, Flyway migration automactally happen.


## 2.Liquibase - Examples

<span style="text-decoration:underline;">EX 1 : SpringBoot Liquibase Integration</span>

1.Create SpringBoot project & Update `pom.xml` with Liquibase Dependency 

	&lt;dependency>

		&lt;groupId>org.liquibase&lt;/groupId>

		&lt;artifactId>liquibase-core&lt;/artifactId>

	&lt;/dependency>

2.Create ChangeSets & define Changelogs in `src/main/resources/db/changelog` folder 

<p id="gdcalert8" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image8.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert9">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image8.png "image_tooltip")


3.Update `application.properties` with Liquibase changelog classpath


```
server.port=8081

spring.datasource.url=jdbc:postgresql://127.0.0.1:5432/liquibasedb
spring.datasource.username=postgres
spring.datasource.password=postgres
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.hibernate.ddl-auto=update
spring.flyway.baseline-on-migrate = true
spring.mvc.view.prefix: /WEB-INF/jsp/
spring.mvc.view.suffix: .jsp

#spring.liquibase.change-log=classpath:db/changelog/liquibase-changelog.yml
spring.liquibase.enabled=true
```


4. Start the application & it will execute scripts order defined in `liquibase-changelog.yml` & saves history in `databasechangelog `table. 

<p id="gdcalert9" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image9.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert10">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image9.png "image_tooltip")


<span style="text-decoration:underline;">EX 2 : Maven Plugin – Liquibase Integration</span>

1.First, disable springboot property 

spring.liquibase.enabled=false

2.Update pom.xml with maven plugin details 


```
	<dependency>
		<groupId>org.liquibase</groupId>
		<artifactId>liquibase-maven-plugin</artifactId>
		<version>3.8.7</version>
	</dependency>
	<plugin>
		<groupId>org.liquibase</groupId>
		<artifactId>liquibase-maven-plugin</artifactId>
		<version>4.7.1</version>


		<executions>
			<execution>
				<id>echo-intsall-phase</id>
				<phase>install</phase>
				<goals>
					<goal>update</goal>
				</goals>
			</execution>
		</executions>


	</plugin>
```


3.Update `liquibase.properties`

changeLogFile=<span style="text-decoration:underline;">src</span>/main/resources/<span style="text-decoration:underline;">db</span>/<span style="text-decoration:underline;">changelog</span>/db.changelog-master.yaml

driver=org.postgresql.Driver

url=jdbc:postgresql://<span style="text-decoration:underline;">localhost</span>:5432/<span style="text-decoration:underline;">liquibasedb</span>

username=<span style="text-decoration:underline;">postgres</span>

password=<span style="text-decoration:underline;">postgres</span>

3.Run `mvn liquibase:update `command to perform Flyway migration.

Or `mvn clean install` 



<p id="gdcalert10" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image10.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert11">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image10.png "image_tooltip")



# Flyway vs Liquibase Features


## Diff Feature

The `diff` command in Liquibase allows you to compare two databases of the **same type, or different types, to one another**.



<p id="gdcalert11" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image11.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert12">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image11.png "image_tooltip")
 

<p id="gdcalert12" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image12.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert13">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image12.png "image_tooltip")


Running the diff command requires two URLs:



* **referenceURL** – the source for the comparison. The referenceURL attribute represents your source (reference) database which is the starting point and the basis for the database you want to compare.
* **url** – the target of the comparison. The URL attribute stands for your target database which you want to compare to the source (reference) database. You typically perform actions and run the commands against this database


```
mvn liquibase:diff
mvn clean install -DskipTests=true liquibase:diff

mvn liquibase:update
mvn clean install -DskipTests=true liquibase:update
```



#### Diff beween Hibernate Entities & Database

To get Diff between entity classes & application database, we need to update `liquibase.properties` with below values


```
outputChangeLogFile= src/main/resources/db/db.changelog-master.xml
driver=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://localhost:3306/liquibasedemo-dev
username=root
password=root

# Reference Properties
referenceUrl=hibernate:spring:com.dpworld.model?dialect=org.hibernate.dialect.MySQLDialect&hibernate.physical_naming_strategy=org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy&hibernate.implicit_naming_strategy=org.springframework.boot.orm.jpa.hibernate.SpringImplicitNamingStrategy

outputChangeLogFile=src/main/resources/db/db.changelog-master.xml
diffChangeLogFile=src/main/resources/db/changelog/liquibase-diff-changeLog.xml
```


The Diff change sets will be generared in <code><span style="text-decoration:underline;">liquibase</span>-<span style="text-decoration:underline;">diff</span>-changeLog.xml</code>


#### Diff beween different databases

If you want to get the diff between dev,test,prod databases of same/different type, we need to update reffrence properties in `liquibase.properties` with below values


```
outputChangeLogFile= src/main/resources/db/db.changelog-master.xml
driver=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://localhost:3306/liquibasedemo-dev
username=root
password=root

referenceUrl=jdbc:mysql://localhost:3306/liquibasedemo-dev
referenceDriver=com.mysql.cj.jdbc.Driver
referenceUsername=root
referencePassword=root

outputChangeLogFile=src/main/resources/db/db.changelog-master.xml
diffChangeLogFile=src/main/resources/db/changelog/liquibase-diff-changeLog.xml
```


The Diff change sets will be generared in <code><span style="text-decoration:underline;">liquibase</span>-<span style="text-decoration:underline;">diff</span>-changeLog.xml</code>

Intellij has plug-in Support

[https://www.baeldung.com/wp-content/uploads/2021/04/jpabuddy_intellij.gif](https://www.baeldung.com/wp-content/uploads/2021/04/jpabuddy_intellij.gif)


## Rollback Feature

To perform Rollback we should have &lt;rollback> scripts inside changesets, like for CREATE TABLE, DROP TABLE etc as shown below.


```
  <changeSet id="01" author="satya">


    <createTable tableName="users" remarks="A table to contain all users">
      <column name="id" type="int" autoIncrement="true">
        <constraints nullable="false" unique="true" primaryKey="true"/>
      </column>
      <column name="name" type="varchar(255)">
        <constraints nullable="false" unique="false"/>
      </column>
      <column name="email" type="varchar(255)">
        <constraints nullable="false"/>
      </column>
    </createTable>

   <rollback>
       <dropTable tableName="users"/>
   </rollback>
  </changeSet>
```


we have three constraints to limit the rollback operation when the condition is satisfied:



* `rollbackTag`
* `rollbackCount`
* `rollbackDate`


```
mvn liquibase:update
```


This executes rollback statements of all the changesets executed after tag “1.0”.


```
mvn liquibase:rollback -Dliquibase.rollbackTag=1.0
```


Here, we define how many changesets we need to be rolled back. If we define it to be one, the last changeset execute will be rolled back:


```
mvn liquibase:rollback -Dliquibase.rollbackCount=1
```


We can set a rollback target as a date, therefore, any changeset executed after that day will be rolled back:


```
mvn liquibase:rollback "-Dliquibase.rollbackDate= 2022-06-12 16:26:02"
```



# conclusion



<p id="gdcalert13" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image13.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert14">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image13.png "image_tooltip")


 


# Ref.

[https://dzone.com/articles/flyway-vs-liquibase](https://dzone.com/articles/flyway-vs-liquibase)

 [https://flywaydb.org/documentation/getstarted/how](https://flywaydb.org/documentation/getstarted/how)

[https://reflectoring.io/database-migration-spring-boot-flyway/](https://reflectoring.io/database-migration-spring-boot-flyway/)

 

[https://pavankjadda.medium.com/safely-evolving-database-with-liquibase-spring-data-and-spring-boot-9c8d2aab1537](https://pavankjadda.medium.com/safely-evolving-database-with-liquibase-spring-data-and-spring-boot-9c8d2aab1537)

<span style="text-decoration:underline;">Diff~~ ~~</span>

[https://github.com/pavankjadda/LiquibaseDemo](https://github.com/pavankjadda/LiquibaseDemo)

[https://github.com/tsk902000/LiquibaseSpringboot](https://github.com/tsk902000/LiquibaseSpringboot)<span style="text-decoration:underline;"> `:`</span>

[https://github.com/AbdullahG/springboot-liquibase](https://github.com/AbdullahG/springboot-liquibase)

[https://www.baeldung.com/liquibase-refactor-schema-of-java-app](https://www.baeldung.com/liquibase-refactor-schema-of-java-app)

<span style="text-decoration:underline;">Rollback</span>

[https://www.baeldung.com/liquibase-rollback](https://www.baeldung.com/liquibase-rollback)

Folder contains .mp3 songs. write powershell script which iterates over the songs, identity the alubm &  and creates a folder with album name then move all songs related to that alubm into album folder


```
# Get the list of .mp3 files in the current folder
$mp3Files = Get-ChildItem -Path $PSScriptRoot -Filter *.mp3

# Iterate over each mp3 file
foreach ($file in $mp3Files) {
    # Use Shell.Application COM object to read mp3 file properties
    $shell = New-Object -ComObject Shell.Application
    $folder = $shell.Namespace($file.Directory.FullName)
    $shellFile = $folder.ParseName($file.Name)
    
    # Get the album name
    $album = $folder.GetDetailsOf($shellFile, 14)

    # Create a folder for the album if it doesn't exist
    $albumFolder = Join-Path -Path $PSScriptRoot -ChildPath $album
    if (-not (Test-Path $albumFolder)) {
        New-Item -ItemType Directory -Path $albumFolder
    }

    # Move the mp3 file to the album folder
    Move-Item -Path $file.FullName -Destination (Join-Path -Path $albumFolder -ChildPath $file.Name)
}


```

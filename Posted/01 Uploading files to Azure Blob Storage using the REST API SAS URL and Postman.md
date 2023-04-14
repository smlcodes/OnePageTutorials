---
title: DevOps - Uploading files to Azure Blob Storage using the REST API SAS URL and Postman
date: 2022-08-19 00:00:00 Z
categories:
- DevOps
tags:
- DevOps
layout: article
cover: /assets/logo/devops.png
sharing: true
license: false
aside:
  toc: true
pageview: true
---


# Uploading files to Azure Blob Storage using the REST API

There are a plethora of different ways to upload files to blob storage. One of the most convenient is to use the HTTP REST API provided.

For this example, I'll be using Postman to complete the operation, but you could very easily use any other method capable of making HTTP requests.

Prepare Blob Storage Access
---------------------------

The first thing we need to do is to allow access to Postman to be able to upload the file.

We'll be making us of the Shared Access Signature or SAS Method of authorisation here.

From your Storage Account page in the portal, click the "Shared access signature" menu item;
![image](https://user-images.githubusercontent.com/20472904/189115120-aba04427-4d78-4ca7-adf7-581d8de50ce1.png)

Azure Portal -- Storage -- Shared Access Signature Menu Item

From here we can create a new Shared Access Signature Key;

![image](https://user-images.githubusercontent.com/20472904/189115180-3223bcfd-382a-4a72-89de-93f5c0d3ea03.png)

Azure Portal -- Storage -- Shared Access Signature Page

We want to give Postman enough privileges to be able to and and create blobs within containers.

For "Allowed Services", deselect "Queue" and "Table" from the options, leaving "Blob" and "File" selected;

![image](https://user-images.githubusercontent.com/20472904/189115464-fdf116f4-240d-40c5-b4f7-91bd78460148.png)

Azure Portal -- Storage -- Shared Access Signature -- Allowed Services



For "Allowed Resource Types", selected just "Object";

Azure Portal -- Storage -- Shared Access Signature -- Allowed Resource Types


For "Allowed Permissions", select only "Write", "Add" and "Create";

Azure Portal -- Storage -- Shared Access Signature -- Allowed Permissions


We can leave "Blob Versioning Permissions" and "Allowed Blog Index Permissions" at their default of selected for this example.

For "Start and expiry date/time", I've chosen to set this SAS to be valid for one year by setting the expiry to be one year from the start date;

Azure Portal -- Storage -- Shared Access Signature -- Validity

Leave the "IP addresses", "Allowed Protocols", "Preferred Routing Tier" and Signing Key at their defaults.



We can now hit the blue "Generate SAS and Connection String" button to generate our authorization values;

![image](https://user-images.githubusercontent.com/20472904/189115718-15a4c5e0-8aae-4468-be42-bc21fe7f230e.png)

Azure Portal -- Storage -- Shared Access Signature -- SAS and Connection Strings



We'll be copying the "Blob service SAS URL", so hit the copy button to the right of that box;

[![](https://www.petecodes.co.uk/wp-content/uploads/2021/09/image-41-1024x99.png)](https://www.petecodes.co.uk/wp-content/uploads/2021/09/image-41.png)

Azure Portal -- Storage -- Shared Access Signature -- Blob Service SAS URL

Create a Container
------------------

Next we need a container to house our file when we upload it.

Click on the "Containers" menu item on the left;

[![](https://www.petecodes.co.uk/wp-content/uploads/2021/09/image-42-1024x653.png)](https://www.petecodes.co.uk/wp-content/uploads/2021/09/image-42.png)

Azure Portal -- Storage -- Containers Menu Item

Click the "+ Container" button;

[![](https://www.petecodes.co.uk/wp-content/uploads/2021/09/image-43-1024x313.png)](https://www.petecodes.co.uk/wp-content/uploads/2021/09/image-43.png)

Azure Portal -- Storage -- Add Container

Give your container a name, I've chosen "fileuploads";

[![](https://www.petecodes.co.uk/wp-content/uploads/2021/09/image-44.png)](https://www.petecodes.co.uk/wp-content/uploads/2021/09/image-44.png)

Azure Portal -- Storage -- Add Container Options

Hit the blue "Create" button to create your container.

Sending a PUT request using Postman
-----------------------------------

We can now use Postman to upload our file to our new container using the SAS key we've generated.

First, create yourself a simple text file somewhere on your hard drive... I created a file called "test1.txt". Stick some simple text in it... I added "Hello!" to my file.

[![](https://www.petecodes.co.uk/wp-content/uploads/2021/09/image-45-1024x738.png)](https://www.petecodes.co.uk/wp-content/uploads/2021/09/image-45.png)

Windows -- Create test file for uploading

Open up Postman and hit the "+ New" button;

[![](https://www.petecodes.co.uk/wp-content/uploads/2021/09/image-46-1024x731.png)](https://www.petecodes.co.uk/wp-content/uploads/2021/09/image-46.png)

Postman "+ New" Button

Choose the "Request" option;

[![](https://www.petecodes.co.uk/wp-content/uploads/2021/09/image-47-1024x606.png)](https://www.petecodes.co.uk/wp-content/uploads/2021/09/image-47.png)

Postman -- New Request

Give your request a name, I chose "Upload File to Blob Storage", and select an existing collection, or click the "+ Create Folder" link and create a new collection. Then hit the "Save to xxx" button to save the request;

[![](https://www.petecodes.co.uk/wp-content/uploads/2021/09/image-48.png)](https://www.petecodes.co.uk/wp-content/uploads/2021/09/image-48.png)

Postman -- Save Request Dialog

From the "Method" drop down select the "PUT" Method;

[![](https://www.petecodes.co.uk/wp-content/uploads/2021/09/image-49-1024x459.png)](https://www.petecodes.co.uk/wp-content/uploads/2021/09/image-49.png)

Select the "Auth" tab below the "Method" drop down. From here, select "API Key" as the Type, then add a "Key" of "x-ms-blob-type" and a value of "BlockBlob";

[![](https://www.petecodes.co.uk/wp-content/uploads/2021/09/image-55-1024x568.png)](https://www.petecodes.co.uk/wp-content/uploads/2021/09/image-55.png)

Postman -- Authorisation Header

Click the "Body" tab. Select "binary" as the type, which will show us a "Select File" button;

[![](https://www.petecodes.co.uk/wp-content/uploads/2021/09/image-57-1024x415.png)](https://www.petecodes.co.uk/wp-content/uploads/2021/09/image-57.png)

Postman -- Binary Body Type

Click the "Select File" button and choose the text file you created earlier. the "Select File" button will be replaced with the name of your file;

[![](https://www.petecodes.co.uk/wp-content/uploads/2021/09/image-58-1024x410.png)](https://www.petecodes.co.uk/wp-content/uploads/2021/09/image-58.png)

Postman -- Selected Body File

Next, paste in the "Blob Service SAS URL" value you copied earlier into the "Enter request URL" box;

[![](https://www.petecodes.co.uk/wp-content/uploads/2021/09/image-60-1024x419.png)](https://www.petecodes.co.uk/wp-content/uploads/2021/09/image-60.png)

Postman -- Paste "Blob Service SAS URL" into Request URL Box

We now need to add in some information into the "Blob Service SAS URL" value we in order to select the container we'd like to upload our file into, and also give our uploaded file a name.

Our copied value will look something like this (I've randomised some values here of course;

```
https://mystorageaccount.blob.core.windows.net/?sv=2020-08-04&ss=bf&srt=o&sp=wactfx&se=2022-09-14T23:04:32Z&st=2021-09-14T15:04:32Z&spr=https&sig=xhfdhfhdfjkhfiufhuiHKJHrehuierhf%3D
```

After the "blob.core.windows.net/" section, add in the name of your container (I chose "fileuploads"), and the filename you'd like to give your file (I chose "test1.txt");

```
https://mystorageaccount.blob.core.windows.net/fileuploads/test1.txt?sv=2020-08-04&ss=bf&srt=o&sp=wactfx&se=2022-09-14T23:04:32Z&st=2021-09-14T15:04:32Z&spr=https&sig=xhfdhfhdfjkhfiufhuiHKJHrehuierhf%3D
```

With that, you should now be able to hit the "SEND" button to upload your file, all being well, you should see a "201 Created" response in green at the bottom of the Postman window;

[![](https://www.petecodes.co.uk/wp-content/uploads/2021/09/image-61-1013x1024.png)](https://www.petecodes.co.uk/wp-content/uploads/2021/09/image-61.png)

Postman -- "201 Created" response

Checking that our file was uploaded
-----------------------------------

If we head back to the portal and our "Containers" page, we can click on our "fileuploads" container;

[![](https://www.petecodes.co.uk/wp-content/uploads/2021/09/image-62-1024x488.png)](https://www.petecodes.co.uk/wp-content/uploads/2021/09/image-62.png)

Azure Portal -- Storage -- "fileuploads" container link

We should then see our newly uploaded "test1.txt" file;

[![](https://www.petecodes.co.uk/wp-content/uploads/2021/09/image-63-1024x433.png)](https://www.petecodes.co.uk/wp-content/uploads/2021/09/image-63.png)

Azure Portal -- Storage -- "test1.txt" file

If we click on that file, we can click the "Edit" tab to view it's contents;

[![](https://www.petecodes.co.uk/wp-content/uploads/2021/09/image-64-1024x757.png)](https://www.petecodes.co.uk/wp-content/uploads/2021/09/image-64.png)

Azure Portal -- Storage -- Edit File Tab

And with that, we've uploaded a file to blob storage using the REST API and Postman.

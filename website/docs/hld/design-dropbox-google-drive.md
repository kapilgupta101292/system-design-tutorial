---
id: design-dropbox-google-drive
title: Design Dropbox or Google Drive
sidebar_label: Design Dropbox or Google Drive
---

In this Article, we will learn "How to Design a Cloud Storage Service". This will help you understand how such services work behind the scenes and help you answer system design questions like "How to Design Google Drive", "How to Design Dropbox" or "How to Design One Drive"

Cloud Storage Services have become very popular as they allows you to access your data anywhere and anytime you want it, on multiple devices without manually transfering data or maintaining disks.

## Challenges

1. Heavy Write Volume - Read to Write ration roughly 1: 1.
2. ACIDity requirement - Strong ACID compliance is required, If we delete a file in a folder and then and then share with other people, it should not be the case that the folder is shared first and people are able to see the deleted file for some time, before it is vanished.
3. Data Deduplication - Files should be deduplicated, to prevent multiple copies of the same file on the server. To prevent data deduplication, dropbox like services breaks the files in chunks of 4MB and then maintain unique identifier for each chunk.
4. Delta Upload - Only the part of files that have changed should be synched on remote servers, For this client application should be intelligent enough to identify the chunks that have changed.

## Requirement Gathering

### Features of Dropbox, Google Drive and One Drive

- Upload, Download, Update and Delete of all files and folders in the workspace.
- Automatic Versioning and Syncronize data among different clients - Desktop, Mobile, Web etc.
- Ability to share files and folders with other people.
- Support Offline operation such as adding/deleting/updating files and folders in workspace, these changes should be synced to server and other clients once you are online.
- Security and Permissions - Data should be secured using encryption and should be visible to only those people who have the permission to view the files.

Above are the functional Requirements of the Dropbox service, Here are some non functional requirements

- High Availibility - Service should be always up to allow users to read their data any time.

## Estimations

According to the stats Dropbox has around 600M users, out of which we can assume that 100M users are active daily

On average if user adds/modifies close to 100 files, we will have a total of 10B uploads daily.

Storage Required: 10B \* 1MB = 10PB

## High Level Design

For uploading and synchronizing your files to Dropbox, you provide a folder that acts as the workspace, any files added, modified or deleted in workspace will be synchronized to Dropbox server and other Desktop and mobile devices.

## Detailed Design

### **Client Application**

Client Application is responsible to monitor the changes in the workspace, It interacts with the synchronization service to process metadata updates like change in file name, or contents. It is also responsible to index the file, and send the updated chunks to the cloud storage and retreiving the same in case other clients have updated the file.

Major Components of Client Application :

- **Watcher** : responsible to monitor and synchronize the files and folders for any Create, Update or Delete operation and then inform the **Indexer** component to handle the changes.
- **Chunker** : job of a chunker is to split the files and the incremental changes into chunks of a suitable size ~4MB. These chunks can be later joined in the same order to reconstruct the original file. This component can intelligently sense the changes that are done on the file and transmit only those parts to the storage server.
- **Indexer** : responsible to listen to the **Watcher** and maintains the information about the chunks of files. Indexer also syncs this data with the Metadata storage server using syncronization service on successful storage of chunks in cloud storage.
- **Internal Database** - keeps maintains a record of the chunks and associated metadata. It allows for the offline operation when the client is not connected to the Dropbox server.

### **Metadata Storage**

Responsible for maintaining the metadata of the files stored in the Cloud storage. It includes information like workspace, versioning, information regarding chunks, users and workspaces. This information can be stored in a SQL database owing to the strong ACID requirements for this data. Two users who are a part of same workspace should see a consistent view of the files and folders. The Synchronization service will use this metadata to syncronized data among different workspaces across multiple clients.

### **Synchornization Service**

One of the critical component of Cloud Storage design. It process all the updates made by the client on the file and syncronizes those updates across all the subscribers. It updates the client local database to be in sync with the Metadata storage on server. All Dropbox clients including Desktop, Mobile and Web clients talk to syncronization service to get updates from the server or push updates to the server. This way, all clients are in sync with the master copy that is stored in the Dropbox cloud. When the client is offline, all updates are stored locally and when the client becomes online, the syncronization service syncs the data to Metadata storage and the same is subsequently pushed to other clients or shared workspace users. It is also possible that two clients have made changes to the same file offline, so it should handle such conflicts. Dropbox handles such scenarios by creating a **Conflicted copy** and saving it with the editor’s username, and the save date. Users will be required to manually resolve that conflict.

### **Cloud Chunk Storage**

All the chunks of the files are uploaded to the cloud Storage, and their location is maintained in the metadata. To reduce load on the dropbox servers the client directly talks to the cloud storage to retreive users data. A good option for the cloud storage is the the Amazon Simple Storage Service (S3).

Take a file, divide into 4MB blocks, based upon hash they are mapped to the same object in the storage.
block level deduplication.

server_file_journal - logs of changes happended on the file.

**PreProcessing** :
Since we are dealing with large files, It is always a good idea to divide the data into chunks to allow upload and download in parallel, this also allows partial uploads/downloads in case of loss of network connectivity and provides efficient utilization of Bandwidht and storage space as we will see later.
Before upload, a large file is broken into multiple chunks. Each chunk has a metadata that will allow us to recreate the file later, Hence we need to store the name (hash of the chunk content), ordering information, size, last modification date etc.

**Synchronization** : Our client application will be installed on Desktop and mobile devices. Client will monitor the folders within the workspace and synchronize the data by interacting with the Synchronization service using file metadata.

Lets assume we have file upload client installed on computer/mobile
Desktop Client Application monitors the folders that are identified as workspace or sync folders and synchronizes them with the remote Cloud Storage. T
• Watcher monitors the sync folders and notifies the Indexer of any action performed by the user for example when user create, delete, or update files or folders.
• Chunker splits the files into smaller pieces called chunks. To reconstruct a file, chunks will be joined back together in the correct order. A chunking algorithm can detect the parts of the files that have been modified by user and only transfer those parts to the Cloud Storage, saving on cloud storage space, bandwidth usage, and synchronization time.
• Indexer processes the events received from the Watcher and updates the internal database with information about the chunks of the modified files. Once the chunks are successfully submitted to the Cloud Storage, the Indexer will communicate with the Synchronization Service using the Message Queuing Service to update the Metadata Database with the changes.
• Internal Database keeps track of the chunks, files, their versions, and their location in the file system

**Metadata Service**

Metadata service will be responsible for maintaining the metadata regarding the chunks of files included
Here we need to maintain strong data consistency to reliably maintain the files for the user.

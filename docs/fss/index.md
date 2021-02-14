# File Store System

The Video System requires a microservices which manages and handles IO operation for media files and object storage. We call this microservice File Store System.

Filestore system workes with RESTFUL endpoints or WebSockets, depending on the respective IO operation taking place.

## RESTFUL Endpoints

These endpoints intended to deal with files/object is size up to 2GB. However, to ensure the best-practices limit is assigned 32MegaBytes.

> **Some best practices**

> - Reduce/Eliminate any IO bottlenecks on the DB
> - Reduce query time
> - Prevent need of a single node to be very powerful
> - Follow a streaming mechanism for large files

## WebSocket

WebSocket transport is intended to perform IO (mostly upload) with large files, to ensure a fast, efficient & network failure resistant upload of files.

In Video System we have used it upload video files from the user's browser. We store the video files in chunks of data as objects in the DB of File Store System, and a `file_upload_manifest` is prepared, which can be used to retrieve the chunks of the file from the DB.

> **`file_upload_manifest`** is a JSON data, with holds the collection of `object_id` & `chunk_index` of the data chunks and other metadata of the file


## Video File Upload

We allow the user to upload a any video file, which can be as big as few GigaBytes. While upload such largefiles that can be certain problems which can hamper the upload process and require upload from the starting point.

Using conventional Http post request for uploading file can be slow, expensive and will fail in case of Network loss.

To resolve the issues in conventioal method of file upload we have developed a process to upload files using websockets, this allows us to upload files at much higher pace, efficiently and make the process network failure resistant.

During the upload a state full manifest of the file is maintined in the `video_uploads` collection of the Document Database. This allows us to retain the file upload state during network loss. If the network failure is too long, the junk data is cleared from MasterLess database, thus saving space on the infrastructure.

**Body of FileManifest**

```js
{
   _id: MongoDbObjectId,
   user_id: string,
   name: string,
   size: number,
   mime_type: string,
   upload_size: number,
   upload_end: boolean
   chunks: [
      {   
         object_id: string
         slice_start: number
         size: number
      }
   ]
}
```

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
   _id: MongoDbObjectId,   // Uuid
   user_id: string,        // 
   name: string,
   size: number,
   mime_type: string,
   upload_size: number,
   upload_end: boolean,
   chunks: [
      {   
         object_id: string
         slice_start: number
         size: number
      }
   ]
}
```

**Steps performed during upload Process**

- **`init`**: Intialize upload of a new video file. FileManifest is created and added to `video_uploads` collection of the Document Database.
- **`chunk`**: Send buffer data of video file in Chunks. All chunk are added as with `object_id` in Masterless DB and FileManifest is updated for the new chunk.
- **`reco`**: Reconnect to File Store System server with incomple file upload. FileManifest recovered from Masterless DB and the client side FileManifest is retured to the client.

#### BSON usage during file upload

**BSON (Binary JSON)** Is JSON like notation for storing JS Object in Binary format, which is more efficient than JSON in terms of size and parsing.

During the file upload process, we not only store inbuilt JS type but also Buffer data as Uint8Array, and storing Uint8Array in conventional JSON as a string is very **in**efficient.

While sending chunks we need to send the following attributes:-

- `type` : 'chunk'
- `upload_id` Uuid of the file upload
- `meta_data` : `{slice_start: number}` 
- `data` : Buffer

To package the above data together in a single socket emit, BSON happens to be the most efficient format.

## Object Storage

An Object is a key-value tuple. `Key` is Uuid assigned to each object and can be used to retrieve the object and thus the value.

**Fields in Object Storage tupple**

```js
{
   user_id: string,
   id: string,       //object_id - Uuid of the Object
   data: Blob        //buffer data - Data of the Object
}
```

**Storing a file with multiple objects**

Table of chunk set on Document DB:

| object_id | slice_start | size |
|--|--|--|
| 0d92feb0-5ba9-11eb-8176-dd41cb581801\|13496 | 0 | 102400 |
| 0db8af20-5ba9-11eb-9544-08b50d13edd5\|13496 | 102400 | 102400 |
| 0def2870-5ba9-11eb-a472-d9ce90888025\|13496 | 204800 | 60040 |

Table of Objects on Masterless DB


| user_id | id | data |
|--|--|--|
| 6009103f58d26934b82c6165 | 0d92feb0-5ba9-11eb-8176-dd41cb581801\|13496 | <Buffer...102400> |
| 6009103f58d26934b82c6165 | 0db8af20-5ba9-11eb-9544-08b50d13edd5\|13496 | <Buffer...102400> |
| 6009103f58d26934b82c6165 | 0def2870-5ba9-11eb-a472-d9ce90888025\|13496 | <Buffer...60040> |



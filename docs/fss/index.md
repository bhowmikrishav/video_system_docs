# File Store System

The Video System requires a microservices which manages and handles IO operation for media files and object storage. We call this microservice File Store System.

Filestore system workes with RESTFUL endpoints or WebSockets, depending on the respective IO operation taking place.

### RESTFUL Endpoints

These endpoints intended to deal with files/object is size up to 2GB. However, to ensure the best-practices limit is assigned 32MegaBytes.

> **Some best practices**

> - Reduce/Eliminate any IO bottlenecks on the DB
> - Reduce query time
> - Prevent need of a single node to be very large
> - Follow a streaming mechanism large files

### WebSocket

WebSocket transport is intended to perform IO (mostly upload) with large files, to ensure a fast, efficient & network failure resistant upload of files.

In Video System we have used it upload video files from user's browser. We store the video files in chunks of data as objects in the DB of File Store System, and a `file_upload_manifest` is prepared, which can be used to retrive the chunks of the file from the DB.

> **`file_upload_manifest`** is a JSON data, with holded the collection of `object_id` & `chunk_index` of the data chunks and other meta data of the file


# Jovial Video System

Jovial Video System is a ***Multi bitrate video processing & streaming system***.
This project is dedicated to building a fast, reliable and easy to scale system.

To achieve the above we are implementing:-

- ***Distributed micro-services architecture*** 

- ***Masterless NoSQL database ("Cassandra")*** for throughput intensive data

- ***Hardware failure resistant design***

- ***Shraded MongoDB database*** for high scalability, availability & required ACID-compliant model.
<i>Since, MongoDB ensures Atomic transaction to the level of a single document which is enough for the requirements in this project.</i>

## Components of this system

### User Management System [>](./UMS/)

### Minimal UI 

### File Store System [>](./fss/)

### Remote Processing demon [>](./vpd/)

### Data server [>](./DataServer/)

# User Management System (UMS)

[(Github Repo)](https://github.com/rishavbhowmik/videosystem_ums)

This microserivce is a rest API based system, designed to provide secured access to user data across the system.

## Rest API server

This server is build in NodeJS using Fastify framework. This server has CROS set to `*` for allowing easier development/testing of distributed microservices.

## Actions of the driver

### Establishing Connection to MongoDB database

We are connecting to MongoDB cluster using MongoDB official driver in NodeJS [`mongodb`](https://www.npmjs.com/package/mongodb). In this, we are using a custom-built class `DB` which can be used to ensure that all operations on the MongoDB clusters are only initiated once the connection is established to the Database, which also helps to prevent multiple connections on the database from a single server.

### User class

Class `User` provides a set of static functions to perform operations with user data, class `DB` is inherited class to `User` for establishing a connection and gain access to the MongoDB database.

Major member funtions:-

- *Register* new User
- *Login/Authorization* of user and return `user_token`
- *Verify* `user_token`
- *Retrive and return* user data
- *Update* User data

This class provides a set of function, with strict verification protocol to ensure an added layer of security, if the UMS server has some security flaw by any chance.

## Endpoints

### `/signup` - POST

To register new user to VideoSystem.

**Request Header**
```YAML
"Content-Type" : "application/json"
```

**Request Body**

```js
{
    username, //unique username string {type:'string', maxLength:32, minLength:3, "pattern": "^([a-z]|[0-9])*$"}
    password, //password for further authentication {type:'string', maxLength:32, minLength:3}
    name //user's full_name {type:'string', maxLength:63, minLength:1}
}
```

**Response Body**

- **On Success**

```js
{
    _id, //MongodbObjectId as String
    username, //String
    password, //String
    name //String
}
```

- **On Error**

```js
{error:e.message, result:null}
```


### `/login` - POST

Login user to Video system, and return user_token which can be used to verify user across the micro services.

**Request Header**
```YAML
"Content-Type" : "application/json"
```

**Request Body**

```js
{
    username, //unique username string {type:'string', maxLength:32, minLength:3, "pattern": "^([a-z]|[0-9])*$"}
    password, //password for further authentication {type:'string', maxLength:32, minLength:3}
}
```

**Response Body**

- **On Success**

```js
{
    _id, //MongodbObjectId as String
    username, //String
    user_token, //String (JWT Token)
}
```

- **On Error**

```js
{error:e.message, result:null}
```

### `/whoami` - POST

Verify the user's request and return user data

**Request Body**

```js
{
    user_token, //String (JWT Token)
}
```
**Response Body**

- **On Success**

```js
{
    _id, //MongodbObjectId as String
    username, //String
    name, //String
}
```

- **On Error**

```js
{error:e.message, result:null}
```
# File Store System

[(Github Repo)](https://github.com/rishavbhowmik/video_system_file_store)

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

Table of Objects on Masterless DB:

| user_id | id | data |
|--|--|--|
| 6009103f58d26934b82c6165 | 0d92feb0-5ba9-11eb-8176-dd41cb581801\|13496 | `<Buffer...102400>` |
| 6009103f58d26934b82c6165 | 0db8af20-5ba9-11eb-9544-08b50d13edd5\|13496 | `<Buffer...102400>` |
| 6009103f58d26934b82c6165 | 0def2870-5ba9-11eb-a472-d9ce90888025\|13496 | `<Buffer...60040>` |

# Video Processing Daemon

[GitHub Repo](https://github.com/rishavbhowmik/video_system_processing_demon)

This is Remote Daemon, which executes cron jobs that trigger video processing functions.

These Video processing function, search unprocessed video files and performs their processing and load the processed video_chunks along with the video_manifest back to their respective databases.

## FFMPEG
It is a free and open-source platform comprising of the software suite of libraries and programs for handling video, audio, and other
multimedia files and streams. 
> At its core is the **FFmpeg** program itself,
designed for command-line-based processing of video and audio files, and
widely used for format transcoding, basic editing, video scaling, video post-
production effects, and standards compliance.

It supports almost all [audio/video codecs](h264, h265, vp8, vp9, aac, opus, etc.) and [file formats](mp4, flv, mkv, ts, webm, mp3 etc.). Moreover it supports again all [streaming protocols](http, rtmp, rtsp, hls, etc.).

### Example -
For instance To convert a flv file to mp4:
>  ffmpeg -i input.flv output.mp4


## Basic conversion
The thing that trips up most people when it comes to converting audio and video is selecting the correct formats and containers. Luckily, FFmpeg is pretty clever with its default settings. Usually, it automatically selects the correct codecs and container without any complex configuration.<br />

For example, say you have an MP3 file and want it converted into an OGG file:<br />

> `ffmpeg -i input.mp3 output.ogg` 

This command takes an MP3 file called  **input.mp3**  and converts it into an OGG file called  **output.ogg**. From FFmpeg's point of view, this means converting the MP3 audio stream into a Vorbis audio stream and wrapping this stream into an OGG container. You didn't have to specify stream or container types, because FFmpeg figured it out for you.
## Selecting your codecs
We can select the codecs needed by using the  **-c**  flag.

This flag lets you set the different codec to use for each stream. For example, to set the audio stream to be Vorbis, you would use the following command:
> ffmpeg  -i input.mp3 -c:a libvorbis output.ogg

**libvorbis** package contains a general-purpose audio and music encoding format. This is useful for creating (encoding) and playing (decoding) sound in an open format.
The command **ffmpeg -codecs** will print every codec FFmpeg knows about. The output of this command will change depending on the version of FFmpeg you have installed.

### Influencing the quality
Now that we have a handle on the codecs, the next question is: How do we set the quality of each stream?
The simplest method is to change the bitrate, which may or may not result in different qualities. 

To set the bitrate of each stream, you use the  **-b**  flag, which works similarly to the  **-c**  flag, except instead of codec options we set a bitrate.

For example, to change the bitrate of the video, we would use it like this:

`ffmpeg  -i  input.webm -c:a copy -c:v vp9 -b:v 1M output.mkv`

This will copy the audio (**-c:a copy**) from  **input.webm**  and convert the video to a VP9 codec (**-c:v vp9**) with a bit rate of 1M/s (**-b:v**), all bundled up in a Matroska container (**output.mkv**).

Another way we can impact quality is to adjust the frame rate of the video using the  **-r**  option:

`ffmpeg  -i  input.webm -c:a copy -c:v vp9  -r  30  output.mkv`

We can also adjust the dimensions of your video using FFmpeg. The simplest way is to use a predetermined video size:

`ffmpeg  -i  input.mkv -c:a copy  -s  hd720 output.mkv`

This modifies the video to 1280x720 in the output, but we can set the width and height manually if we want:

`ffmpeg  -i  input.mkv -c:a copy  -s  1280x720 output.mkv`

This produces the same output as the earlier command. If we want to set custom sizes in FFmpeg, please remember that the width parameter (**1280**) comes before height (**720**).
## FFMPEG commands
#### Command

> ffmpeg -i pride.mp4 -c:v libvpx-vp9 -crf 30 -b:v 2000k -map 0 -segment_time 00:00:05 -f segment output%03d.webm

> ffmpeg -i pride.mp4 -c:v libvpx-vp9 -crf 30 -b:v 2000k -map 0 -segment_time 00:00:05 -f segment output%d.webm

#### Probe result of output001.webm

[Video Stream]
codec_name=vp9
codec_long_name=Google VP9
coded_width=1920
coded_height=800
display_aspect_ratio=12:5
start_time=5.346000
pix_fmt=yuv420p
ENCODER=Lavc58.112.101 libvpx-vp9
DURATION=00:00:10.685000000

[Audio Stream]
codec_name=opus
codec_long_name=Opus (Opus Interactive Audio Codec)
channel_layout=stereo
ENCODER=Lavc58.112.101 libopus
DURATION=00:00:10.700000000
start_time=5.353000


### FFMPEG installation

#### Ubuntu

sudo apt update
sudo apt install ffmpeg
ffmpeg -version



# FFPROBE
ffprobe gathers information from multimedia streams and prints it in human- and machine-readable fashion.

For example, it can be used to check the format of the container used by a multimedia stream and the format and type of each media stream contained in it.

If a url is specified in the input, ffprobe will try to open and probe the URL content. If the URL cannot be opened or recognized as a multimedia file, a positive exit code is returned.

ffprobe may be employed both as a standalone application or in combination with a textual filter, which may perform more sophisticated processing, e.g. statistical processing or plotting.

### Options

**Options** are used to list some of the formats supported by ffprobe or for specifying which information to display, and for setting how **ffprobe** will show it.
Some options are applied per-stream, e.g. 
**bitrate or codec**. 
Stream specifiers are used to precisely specify which stream(s) a given option belongs to.

### Generic options

These options are shared amongst the ff* tools.

> -L

*Show license.*

> -h, -?, -help, --help [arg]

*Show help. An optional parameter may be specified to print help about a specific item. If no argument is specified, only basic (non-advanced) tool options are shown.*

Possible values of  arg  are:

> long

*Print advanced tool options in addition to the basic tool options.*

> full

*Print complete list of options, including shared and private options for encoders, decoders, demuxers, muxers, filters, etc.*

> emphasized textdecoder=decoder_name

*Print detailed information about the decoder named  decoder_name. Use the `-decoders` option to get a list of all decoders.*

> encoder=encoder_name

*Print detailed information about the encoder named  encoder_name. Use the  `-encoders` option to get a list of all encoders.*

VPD is executed by [Cron Jobs](./cronjob)

# VIDEO PROCESSING DAEMON CRONJOB

**INTRODUCTION**:- Cron is a utility that schedules a command or script on your server to run automatically at a specified time and date. A cron job is the scheduled task itself. Cron jobs can be very useful to automate repetitive tasks.Cron Jobs are used for scheduling tasks to run on the server. They're most commonly used for automating system maintenance or administration. However, they are also relevant to web application development. There are many situations when a web application may need certain tasks to run periodically

**PROCESSING CRON JOBS**:- The cron daemon is a long-running process that executes commands at specific dates and times. You can use this to schedule activities, either as one-time events or as recurring tasks. To schedule one-time only tasks with cron, use the at or batch command. Data processing is most often triggered by user actions, either your end-user or internal users: back-office, admin superusers, or support users maybe. In some cases though, it may happen that some data processing needs to happen on its own, following a schedule rather than being triggered by direct user activity on the product.

**LOOKUP MECHANISM TO FIND A PROCESSING JOB** :- The look-up mechanism helps find a job , find a video which require processing in multiple resolutions. We perform a query on the database to search for unprocessed resolution of video upload. 

**MECHANISM TO LOAD FILE LOCALLY** :- First, We clear the temporary folder then load the raw video file in the temporary folder . 

```js
const raw_file_name = "temp/raw"+mime_to_ext(file_manifest.mime_type) . 
```

Mime_to_exe is a function which makes sure whether the video file is in the desrired format.

**PROCESSING AND MANIFEST CREATION** :- When a video stream is set up, a video manifest file is delivered to the player, itemizing the available streams. Like a restaurant menu, the manifest tells the player what resolutions and bitrates are available, and the player chooses an appropriate stream.

# Video Player Prototype

[(Github Repo)](https://github.com/rishavbhowmik/video_system_player_exp1)

## Introduction to Media Source API

The **MediaSource** interface of the [Media Source Extensions API](https://developer.mozilla.org/en-US/docs/Web/API/Media_Source_Extensions_API) represents a source of media data for an [`HTMLMediaElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement) object.
A `MediaSource` object can be attached to a [`HTMLMediaElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement) to be played in the user agent. 
A `MediaSource` represents a source of media for an audio or video element. Once a `MediaSource` object is instantiated and its `open` event has fired, `SourceBuffer`s can be added to it. These act as buffers for media segments.
## Properties:

- [`MediaSource.sourceBuffers`](https://developer.mozilla.org/en-US/docs/Web/API/MediaSource/sourceBuffers) : returns a [`SourceBufferList`](https://developer.mozilla.org/en-US/docs/Web/API/SourceBufferList) object containing the list of [`SourceBuffer`](https://developer.mozilla.org/en-US/docs/Web/API/SourceBuffer) objects associated with this `MediaSource`.

- [`MediaSource.activeSourceBuffers`](https://developer.mozilla.org/en-US/docs/Web/API/MediaSource/activeSourceBuffers) : Returns a [`SourceBufferList`](https://developer.mozilla.org/en-US/docs/Web/API/SourceBufferList) object containing a subset of the [`SourceBuffer`](https://developer.mozilla.org/en-US/docs/Web/API/SourceBuffer) objects contained within [`MediaSource.sourceBuffers`](https://developer.mozilla.org/en-US/docs/Web/API/MediaSource/sourceBuffers) — the list of objects providing the selected video track, enabled audio tracks, and shown/hidden text tracks.

- [`MediaSource.readyState`](https://developer.mozilla.org/en-US/docs/Web/API/MediaSource/readyState)  : returns an enum representing the state of the current  `MediaSource`, whether it is not currently attached to a media element (`closed`), attached and ready to receive  [`SourceBuffer`](https://developer.mozilla.org/en-US/docs/Web/API/SourceBuffer)  objects (`open`), or attached but the stream has been ended via  [`MediaSource.endOfStream()`](https://developer.mozilla.org/en-US/docs/Web/API/MediaSource/endOfStream)  (`ended`.)
- [`MediaSource.duration`](https://developer.mozilla.org/en-US/docs/Web/API/MediaSource/duration) : gets and sets the duration of the current media being presented.


## Video Player Implementation

Steps of implementing the Video player prototype

- Accessing dom video element using getElementById() function.

```html
<video height="400px" width="960px" controls src="" id="video"></video>
```

```js
const video = document.getElementById('video')
```

- Mime type is the `video/webm`. Webm is the media file format and a container for VP8, VP9, AV1, vorbis and opus file encodings. Here we are VP9 for video and opus for audio.

```js
var mimeCodec = 'video/webm;codecs="vp9,opus"'
```

- Creating new object for MediaSource class. 

```js
var mediaSource = new MediaSource()
```

- The URL.createObjectURL() static method creates a DOMString containing a URL representing the object given in the parameter. 

```js
video.src = window.URL.createObjectURL(mediaSource)
```

- Wait for the mediasource object to reach the state of 'Sourceopen'

```js
mediaSource.addEventListener('sourceopen', async () => {
  //After mediaSource/'sourceopen'
})
```

- Create source buffer using MimeCodec 

```js
var sourceBuffer = mediaSource.addSourceBuffer(mimeCodec)
```

- the Source buffer mode is set to 'segment'.

```js
sourceBuffer.mode = 'segment'
```

- Load the video chunk using an http request and then convert it into an Uint8Array

```js
  get(assetURL.replace('n', n++)
```

- appendBuffer function is used to append the source buffer with the loaded video chunk

```js
sourceBuffer.appendBuffer(
  new Uint8Array(
    await get(assetURL.replace('n', n++))
  )
)
```

- We perform iterations of loading the video chunks and appending the source buffer until all the video chunks are loaded.


## Video Chunks as Segments

Instead of encoding the entire video file with a target quality, the complexity of each video chunk is considered during the encoding process.

One of the clear benefits of dividing the whole video file into segments is that because the client player issues normal HTTP request for entire file,this type of streaming and rate adaptation is well supported by the existing HTTP ecosystem. Using segmented video chunks increases the streaming efficiency as the requested chunks are loaded as per the users' request, instead of the entire video file.


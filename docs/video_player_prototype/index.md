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

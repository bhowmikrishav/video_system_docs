# Video Player Prototype

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
- Mime type is the `video/webm`. Webm is the media file format and a container for VP8, VP9, AV1, vorbis and opus file encodings. Here we are VP9 for video and opus for audio.
- Creating new object for MediaSource class. 
- The URL.createObjectURL() static method creates a DOMString containing a URL representing the object given in the parameter. 

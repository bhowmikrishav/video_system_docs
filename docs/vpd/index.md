# FFMPEG
It is a free and open-source platform consisting of a vast
software suite of libraries and programs for handling video, audio, and other
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
The thing that trips up most people when it comes to converting audio and video is selecting the correct formats and containers. Luckily, FFmpeg is pretty clever with its default settings. Usually it automatically selects the correct codecs and container without any complex configuration.<br />

For example, say you have an MP3 file and want it converted into an OGG file:<br />

> `ffmpeg -i input.mp3 output.ogg` 

This command takes an MP3 file called  **input.mp3**  and converts it into an OGG file called  **output.ogg**. From FFmpeg's point of view, this means converting the MP3 audio stream into a Vorbis audio stream and wrapping this stream into an OGG container. You didn't have to specify stream or container types, because FFmpeg figured it out for you.
## Selecting your codecs
We can select the codecs needed by using the  **-c**  flag.

This flag lets you set the different codec to use for each stream. For example, to set the audio stream to be Vorbis, you would use the following command:
> ffmpeg  -i input.mp3 -c:a libvorbis output.ogg

**libvorbis** package contains a general purpose audio and music encoding format. This is useful for creating (encoding) and playing (decoding) sound in an open format.

The command **ffmpeg -codecs** will print every codec FFmpeg knows about. The output of this command will change depending on the version of FFmpeg you have installed.

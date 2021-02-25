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

### Influencing the quality
Now that we have a handle on the codecs, the next question is: How do we set the quality of each stream?
The simplest method is to change the bitrate, which may or may not result in a different quality. 

To set the bitrate of each stream, you use the  **-b**  flag, which works in a similar fashion to the  **-c**  flag, except instead of codec options we set a bitrate.

For example, to change the bitrate of the video, we would use it like this:

`ffmpeg  -i  input.webm -c:a copy -c:v vp9 -b:v 1M output.mkv`

This will copy the audio (**-c:a copy**) from  **input.webm**  and convert the video to a VP9 codec (**-c:v vp9**) with a bit rate of 1M/s (**-b:v**), all bundled up in a Matroska container (**output.mkv**).

Another way we can impact quality is to adjust the frame rate of the video using the  **-r**  option:

`ffmpeg  -i  input.webm -c:a copy -c:v vp9  -r  30  output.mkv`

We can also adjust the dimensions of your video using FFmpeg. The simplest way is to use a predetermined video size:

`ffmpeg  -i  input.mkv -c:a copy  -s  hd720 output.mkv`

This modifies the video to 1280x720 in the output, but we can set the width and height manually if we want:

`ffmpeg  -i  input.mkv -c:a copy  -s  1280x720 output.mkv`

This produces the exact same output as the earlier command. If we want to set custom sizes in FFmpeg, please remember that the width parameter (**1280**) comes before height (**720**).
## FFMPEG commands
#### Command

> ffmpeg -i pride.mp4 -c:v libvpx-vp9 -crf 30 -b:v 2000k -map 0 -segment_time 00:00:05 -f segment output%03d.webm

> ffmpeg -i pride.mp4 -c:v libvpx-vp9 -crf 30 -b:v 2000k -map 0 -segment_time 00:00:05 -f segment output%d.webm

#### Probe result of output001.webm

```
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

```
### FFMPEG installation

#### Ubuntu

sudo apt update
sudo apt install ffmpeg
ffmpeg -version


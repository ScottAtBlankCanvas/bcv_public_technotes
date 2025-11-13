# Random ffmpeg notes to organize

## Resources

### ffmpeg articles

Overview: https://img.ly/blog/ultimate-guide-to-ffmpeg/

Generating hls with ffmpeg: series of articles: https://www.martin-riedl.de/2020/04/17/using-ffmpeg-as-a-hls-streaming-server-overview/

And another good HLS w ffmpeg article: https://ottverse.com/hls-packaging-using-ffmpeg-live-vod/

### ffmpeg docs

And ffmpeg docs here:  https://ffmpeg.org/ffmpeg-formats.html#hls-2

### filters

Building filters:

- https://fossies.org/linux/ffmpeg/doc/writing_filters.txt
- https://wiki.multimedia.cx/index.php/FFmpeg_filter_HOWTO
- https://cldfire.dev/blog/writing-an-opencl-filter-for-ffmpeg/
- https://medium.com/@rafaelgalle1/building-a-custom-scalable-audio-transcription-pipeline-whisper-pyannote-ffmpeg-d0f03f884330


vf_scale example

xilinx sample filters: https://xilinx.github.io/video-sdk/v3.0/examples/ffmpeg/filters.html

# Steps

## Building ffmpeg

### Building ffmpeg on Mac

```nix
cd ~/code/oss
git clone https://github.com/FFmpeg/FFmpeg.git
cd FFmpeg
./configure  --prefix=/usr/local \
--enable-gpl \
--enable-nonfree \
--enable-libass \
--enable-libfdk-aac \
--enable-libfreetype \
--enable-libmp3lame \
--enable-libtheora \
--enable-libvorbis \
--enable-libvpx \
--enable-libx264 \
--enable-libx265 \
--enable-libopus \
--enable-libxvid \
--samples=fate-suite/
make

```

## Creating a foobar filter from edgedetect

https://fossies.org/linux/ffmpeg/doc/writing_filters.txt

Did similar to above except, built a filter for debugging avsf_debug from av_subtitles (burn in)


### Docker and ffmpeg

Docker ffmpeg build:  https://github.com/jrottenberg/ffmpeg

Docker build (from above

```nix
pull jrottenberg/ffmpeg:4.4-ubuntu
; start docker container
; runs ffmpeg once and exits

```

Use github script to build ffmpeg inside docker: https://github.com/srwareham/docker-ffmpeg-compiler

## Building with libav*

### Using boost on MacOS

I downloaded local version of boost inside my project

Boost not built with clang by default, so

https://stackoverflow.com/questions/8486077/how-to-compile-link-boost-with-clang-libc

Make sure stdlib and std match what it is used whereever boost is being usedcd ..
:

```nix
cd boost*
./bootstrap.sh --with-toolset=clang
./b2 clean
./b2 toolset=clang cxxflags="-std=c++14 -stdlib=libc++" linkflags="-stdlib=libc++"
```

Also need to build project with clang

## SRT and ffmpeg

https://srtlab.github.io/srt-cookbook/apps/ffmpeg.html

Restream VOD as SRT:

```nix
ffmpeg -i ~/sample.mp4 -c copy -f mpegts srt://<ip>:<port>
```

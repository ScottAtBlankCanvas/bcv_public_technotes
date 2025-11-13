# ffmpeg and whisper

## Background

Found the whisper ffmpeg info via [LinkedIn Post](https://www.linkedin.com/posts/ericfontaine13_ffmpeg-openai-whisper-activity-7366499799753281537-yYNk)

I followed this [Medium Article](https://medium.com/@vpalmisano/run-whisper-audio-transcriptions-with-one-ffmpeg-command-c6ecda51901f) by [Vittorio Palmisano](https://www.linkedin.com/in/vpalmisano/)

Good article on [Mac Dynamic Library Loading](https://clarkkromenaker.com/post/library-dynamic-loading-mac/)

## Difficulties I had with Medium Build Steps

I was building on a MacBook Pro 2019: 2.6 GHz 6-Core Intel Core i7

<b>Problem:</b> libpulse was not installed on my machine

<b>Solution:</b> Just needed to install it


<b>Problem:</b> ffmpeg scripts attempt to modify /usr/ instead of /usr/local/ (Mac SiP prevents this)

<b>Solution:</b> Adjusted build to /usr/local.  See script notes below


<b>Problem:</b> whisper library was not loading with ffmpeg

<b>Solution:</b> setting DYLD_LIBRARY_PATH

Used otool to verify Should find thgem in /usr/local/lib based on rpath setting (using otool) but had to force it via setting DYLD_LIBRARY_PATH.
See script notes below

## Future Work

* Running on Apple Silicon, should build whisper with [Core ML Support](https://github.com/ggml-org/whisper.cpp#core-ml-support)

## Summary of Modified Build steps:

### Build prerequisites
```nix
# pulsa library was not found, so install it
brew install pulseaudio
```

### Build whisper (no changes from Medium article)
```nix
cd ~/code
git clone https://github.com/ggml-org/whisper.cpp
cd whisper.cpp
sh ./models/download-ggml-model.sh base.en
cmake -B build
cmake --build build --config Release
sudo make install -C build
```

### Build ffmpeg (changes denoted below)
```nix
cd ~/code
git clone https://code.ffmpeg.org/FFmpeg/FFmpeg.git
# I renamed the ffmpeg directory bc I have several
mv FFmpeg FFmpeg.whisper
cd FFmpeg.whisper

# Mac does not like to modify /usr so change prefix to /usr/local
# note add srt

./configure --prefix=/usr/local --enable-version3 --disable-shared --enable-gpl \
  --enable-nonfree --enable-static --enable-pthreads --enable-filters \
  --enable-openssl --enable-runtime-cpudetect --enable-libvpx --enable-libx264 \
  --enable-libx265 --enable-libspeex --enable-libfreetype --enable-fontconfig \
  --enable-libzimg --enable-libvorbis --enable-libwebp --enable-libfribidi \
  --enable-libharfbuzz --enable-libpulse --enable-libass --enable-whisper \
  --enable-libsrt
make
sudo make install

# should not need this because whisper libraries were ijstalled to /usr/local/lib.  But only way ffmpeg executables wouod load whisper libs
export DYLD_LIBRARY_PATH=/usr/local/lib:$DYLD_LIBRARY_PATH

ffmpeg --help filter=whisper
```

## Running

ffprobe.  Need to turn off gpu (use_gpu=0) and use a different file
```nix
ffprobe -select_streams a -print_format compact -show_frames \
-show_entries 'frame_tags=lavfi.whisper.text,lavfi.whisper.duration' \
-f lavfi \
-i "amovie=katiesteve.wav,whisper=model=../whisper.cpp/models/ggml-base.en.bin:language=en:queue=3:use_gpu=0" \
| grep whisper
```

```nix
ffmpeg -i https://github.com/vpalmisano/webrtcperf/releases/download/videos-1.0/gvr.mp4 \
-vn -af "whisper=model=../whisper.cpp/models/ggml-base.en.bin\
:language=en\
:queue=3\
:use_gpu=0\
:destination=output.srt\
:format=srt" -f null -
```

```nix
ffmpeg -loglevel warning -re -i https://livecmaftest1.akamaized.net/cmaf/live/2099281/abr6s/master.m3u8 \
-vn -af 'whisper=model=../whisper.cpp/models/ggml-base.en.bin\
:language=en\
:queue=3\
:use_gpu=0\
:destination=-\
:format=json' -f null -
```

## Next Steps

1. Send in rtmp, pull out audio and do live trancribe
2.

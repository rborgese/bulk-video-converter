# Bulk Video Converter

Simple configurable bulk ffmpeg-based video converter.

It uses **YAML** configuration for generating several output files 
from one (or more) input video. 
Each output file is created  according to `profile` subsection 
in *YAML* configuration. See [examples](#quick-start-guide) below.

\
## Table of contents

* [Motivation](#motivation)
* [Requirements](#requirements)
* [Quick start guide](#quick-start-guide)
    * [Conversion to one video](#conversion-to-one-video)
    * [Conversion to several videos](#conversion-to-several-videos)
* [TODO](#todo)
  
## Motivation

The main purpose of this tool is transcoding single source to several
videos of different qualities and sizes. It is very helpful for 
handling raw videos from amateur camera. 

I use this tool to convert a great amout of `*.MTS` files 
to web-compatible formats.
 
## Requirements

It uses the latest version of standard ffmpeg pack.
This is my ffmpeg configuration.
Check it if something's gone wrong.
    
    ffmpeg version 2.8.6 Copyright (c) 2000-2016 the FFmpeg developers
    built with gcc 5.1.1 (GCC) 20150618 (Red Hat 5.1.1-4)
    configuration: 
        ...
        --arch=x86_64 
        --enable-bzlib 
        --disable-crystalhd 
        --enable-frei0r 
        --enable-gnutls 
        --enable-ladspa 
        --enable-libass 
        --enable-libcdio 
        --enable-libdc1394 
        --disable-indev=jack 
        --enable-libfreetype 
        --enable-libgsm 
        --enable-libmp3lame 
        --enable-openal 
        --enable-libopencv 
        --enable-libopenjpeg 
        --enable-libopus 
        --enable-libpulse 
        --enable-libschroedinger 
        --enable-libsoxr 
        --enable-libspeex 
        --enable-libtheora 
        --enable-libvorbis 
        --enable-libv4l2 
        --enable-libvpx 
        --enable-libx264 
        --enable-libx265 
        --enable-libxvid 
        --enable-x11grab 
        --enable-avfilter 
        --enable-avresample 
        --enable-postproc 
        --enable-pthreads 
        --disable-static 
        --enable-shared 
        --enable-gpl 
        --enable-runtime-cpudetect
        ...

## Quick start guide

* [Conversion to one video](#conversion-to-one-video)
* [Conversion to several videos](#conversion-to-several-videos)

### Conversion to one video

If you want to get one transcoded file from one source,
you can only run this command.

```bash
bulk_video_converter.bash -c config.yaml /path/to/video/file/00001.MTS
```

Where:

* `00001.MTS` — is you source video file.
* `config.yaml` — is configuration that describes you transcoded file.

#### Config example for one profile

```yaml
ffmpeg:
  bin: /usr/bin/ffmpeg  # If you have several ffmpeg installations 
  threads: 0            # 0 — use CPU autodetection
profile:
  base:
    abstract: 1     # This profile will be used only for inheritance.
    start: 00:00:10                 # Optional parametr.
    stop: 00:00:30                  # Optional parametr.
    output_dir_name: ./out          # Optional parametr.
    pass_log_dir_name: ./pass_log   # Optional parametr.
  my_profile:
    extends: base   # Other options are inherited from `base`.
    passes: 2       # Two passes of ffmpeg.
    video:
      preset: veryslow
      width: 1280
      height: 720
      bitrate: 2000k
      codec:
        name: h264
        weightp: 2
        bframes: 3
        opts: "keyint=96:min-keyint=96:no-scenecut"
        profile: high
        level: 3.1
    audio:
      codec:
        name: aac
      channels: 5.1
      bitrate: 320k
```


With such configuration the converter will generate output log
and perform two sequential ffmpeg passes.

#### Output log example for one profile

```yaml
bulk_video_converter.bash:
  /home/user/Video/00001.MTS :
    profile my_profile:
      global input: -ss '00:00:10' -threads '0' 
      video: -preset 'veryslow' -b:v '2000k' -vf 'scale=1280:720' -codec:v 'libx264' -profile:v 'high' -level:v '3.1' -weightp '2' -bf '3' -x264opts 'keyint=96:min-keyint=96:no-scenecut' 
      audio: -b:a '320k' -ac '6' -strict 'experimental' -codec:a 'aac' 
      global output: -ss '00:00:10' -to '00:00:30' 
      passes:
        pass 1: /usr/bin/ffmpeg -ss '00:00:10' -threads '0' -i '/home/user/Video/00001.MTS' -preset 'veryslow' -b:v '2000k' -vf 'scale=1280:720' -codec:v 'libx264' -profile:v 'high' -level:v '3.1' -weightp '2' -bf '3' -x264opts 'keyint=96:min-keyint=96:no-scenecut' -pass 1 -passlogfile ./pass_log/00001-my_profile.mts -b:a '320k' -ac '6' -strict 'experimental' -codec:a 'aac' -ss '00:00:10' -to '00:00:30' -f 'mp4' -y '/dev/null' 2>&1 | tee /var/log/bulk_video_converter.bash/2016-05-02_04-09-08-036081777/00001-my_profile-1-mp4.ffmpeg.log 1>&2;
        pass 2: /usr/bin/ffmpeg -ss '00:00:10' -threads '0' -i '/home/user/Video/00001.MTS' -preset 'veryslow' -b:v '2000k' -vf 'scale=1280:720' -codec:v 'libx264' -profile:v 'high' -level:v '3.1' -weightp '2' -bf '3' -x264opts 'keyint=96:min-keyint=96:no-scenecut' -pass 2 -passlogfile ./pass_log/00001-my_profile.mts -b:a '320k' -ac '6' -strict 'experimental' -codec:a 'aac' -ss '00:00:10' -to '00:00:30' -f 'mp4' -y './out/00001-my_profile.mp4' 2>&1 | tee /var/log/bulk_video_converter.bash/2016-05-02_04-09-08-036081777/00001-my_profile-2-mp4.ffmpeg.log 1>&2;
      # passes done
    # profile my_profile done
  # /home/user/Video/00001.MTS done
# bulk_video_converter.bash done

```

#### Performance example for one profile

##### Performance example: pass 1


```bash
/usr/bin/ffmpeg                                             \
    -ss '00:00:10'                                          \
    -threads '0'                                            \
    -i '/path/to/video/file/00001.MTS'                      \
    -preset 'veryslow'                                      \
    -b:v '2000k'                                            \
    -vf 'scale=1280:720'                                    \
    -codec:v 'libx264'                                      \
    -profile:v 'high'                                       \
    -level:v '3.1'                                          \
    -weightp '2'                                            \
    -bf '3'                                                 \
    -x264opts 'keyint=96:min-keyint=96:no-scenecut'         \
    -pass '1'                                               \
    -passlogfile './pass_log/00001-my_profile.mts'          \
    -b:a '320k'                                             \
    -ac '6'                                                 \
    -strict 'experimental'                                  \ 
    -codec:a 'aac'                                          \
    -ss '00:00:10'                                          \
    -to '00:00:30'                                          \
    -f 'mp4'                                                \
    -y '/dev/null';
```


##### Performance example: pass 2


```bash
/usr/bin/ffmpeg                                             \
    -ss '00:00:10'                                          \
    -threads '0'                                            \
    -i '/path/to/video/file/00001.MTS'                      \
    -preset 'veryslow'                                      \
    -b:v '2000k'                                            \
    -vf 'scale=1280:720'                                    \
    -codec:v 'libx264'                                      \
    -profile:v 'high'                                       \
    -level:v '3.1'                                          \
    -weightp '2'                                            \
    -bf '3'                                                 \
    -x264opts 'keyint=96:min-keyint=96:no-scenecut'         \
    -pass '2'                                               \
    -passlogfile './pass_log/00001-my_profile.mts'          \
    -b:a '320k'                                             \
    -ac '6'                                                 \
    -strict 'experimental'                                  \ 
    -codec:a 'aac'                                          \
    -ss '00:00:10'                                          \
    -to '00:00:30'                                          \
    -f 'mp4'                                                \
    -y './out/00001-my_profile.mp4';
```


### Conversion to several videos

If you want to get several videos from one source you should describe 
several profiles in the config. For example, let we use profiles from 
[H.264 web video encoding tutorial with FFmpeg.](https://www.virag.si/2012/01/web-video-encoding-tutorial-with-ffmpeg-0-9/)


#### Config example for virag's profiles

```yaml
ffmpeg:
  bin: /usr/bin/ffmpeg
  threads: 0
profile:
  base:
    abstract: 1 # It will be used only for inheritance.
    source: /home/user/Video/*.MTS  # also you can set file names here.
    start: 00:00:10
    stop: 00:00:30
    output_dir_name: ./out
    pass_log_dir_name: ./pass_log
  virag_h264x1_pal_sd:                  # PAL SD video.
    extends:   base
    passes: 1
    video:
      width:  0   # any
      height: 576 # SD
      preset: slower
      codec:
        profile: main
      bitrate: 1000k
    audio:
      codec:
        name: aac
      channels: 5.1
      bitrate: 196k
  virag_h264x1_480p_web:                # Standard web video.
    extends: virag_h264x1_pal_sd
    video:
      height: 480
      preset: slow
      bitrate: 500k
      maxrate: 500k
      bufsize: 1000k
    audio:
      bitrate: 128k
      channels: stereo
  virag_h264x1_480p_tablet:             # Video for iPads and tablets.
    extends: virag_h264x1_480p_web
    video:
      codec:
        profile: main
      bitrate: 400k
      maxrate: 400k
      bufsize: 800k
  virag_h264x1_360p_mobile:             # 360p video for old phones.
    extends: virag_h264x1_480p_tablet
    video:
      height: 360
      codec:
        profile: baseline
      bitrate: 250k
      maxrate: 250k
      bufsize: 500k
    audio:
      bitrate: 96k
      channels: mono
```


#### Output example for virag's profiles

You can check what you will get with dry-run option (`-d`).
So for virag's profiles you will have output with four ffmpeg-commands.
All profiles will be handled in parallel way.
It is same if you run ffmpeg four times at the same moment.

```bash
[user@host ~]$ ./bulk_video_converter.bash -c config.yaml -d
bulk_video_converter.bash:
# NOTICE 25:  bulk_video_converter.bash creates directory /tmp/bulk_video_converter.bash/01TaRYSd
  /home/user/Video/00001.MTS:
    profile virag_h264x1_360p_mobile:
      global input:
        -ss '00:00:10' -threads '0' 
      # global input done
      video:
        -preset 'slow' -b:v '250k' -maxrate '250k' -bufsize '500k' -vf 'scale=0:360' -codec:v 'libx264' -profile:v 'baseline' 
      # video done
      audio:
        -b:a '96k' -ac '1' -strict 'experimental' -codec:a 'aac' 
      # audio done
      global output:
        -ss '00:00:10' -to '00:00:30' 
      # global output done
      passes:
        pass 1:
          /usr/bin/ffmpeg -ss '00:00:10' -threads '0' -i '/home/user/Video/00001.MTS' -preset 'slow' -b:v '250k' -maxrate '250k' -bufsize '500k' -vf 'scale=0:360' -codec:v 'libx264' -profile:v 'baseline' -b:a '96k' -ac '1' -strict 'experimental' -codec:a 'aac' -ss '00:00:10' -to '00:00:30' -f 'mp4' -y './out/00001-virag_h264x1_360p_mobile.mp4' 2>&1 | tee /var/log/bulk_video_converter.bash/2016-05-02_05-14-20-695774334/00001-virag_h264x1_360p_mobile-1-mp4.ffmpeg.log 1>&2;
        # pass 1 done
      # passes done
    # profile virag_h264x1_360p_mobile done
  # /home/user/Video/00001.MTS done
  /home/user/Video/00001.MTS:
    profile virag_h264x1_480p_tablet:
      global input:
        -ss '00:00:10' -threads '0' 
      # global input done
      video:
        -preset 'slow' -b:v '400k' -maxrate '400k' -bufsize '800k' -vf 'scale=0:480' -codec:v 'libx264' -profile:v 'main' 
      # video done
      audio:
        -b:a '128k' -ac '2' -strict 'experimental' -codec:a 'aac' 
      # audio done
      global output:
        -ss '00:00:10' -to '00:00:30' 
      # global output done
      passes:
        pass 1:
          /usr/bin/ffmpeg -ss '00:00:10' -threads '0' -i '/home/user/Video/00001.MTS' -preset 'slow' -b:v '400k' -maxrate '400k' -bufsize '800k' -vf 'scale=0:480' -codec:v 'libx264' -profile:v 'main' -b:a '128k' -ac '2' -strict 'experimental' -codec:a 'aac' -ss '00:00:10' -to '00:00:30' -f 'mp4' -y './out/00001-virag_h264x1_480p_tablet.mp4' 2>&1 | tee /var/log/bulk_video_converter.bash/2016-05-02_05-14-20-695774334/00001-virag_h264x1_480p_tablet-1-mp4.ffmpeg.log 1>&2;
        # pass 1 done
      # passes done
    # profile virag_h264x1_480p_tablet done
  # /home/user/Video/00001.MTS done
  /home/user/Video/00001.MTS:
    profile virag_h264x1_480p_web:
      global input:
        -ss '00:00:10' -threads '0' 
      # global input done
      video:
        -preset 'slow' -b:v '500k' -maxrate '500k' -bufsize '1000k' -vf 'scale=0:480' -codec:v 'libx264' -profile:v 'main' 
      # video done
      audio:
        -b:a '128k' -ac '2' -strict 'experimental' -codec:a 'aac' 
      # audio done
      global output:
        -ss '00:00:10' -to '00:00:30' 
      # global output done
      passes:
        pass 1:
          /usr/bin/ffmpeg -ss '00:00:10' -threads '0' -i '/home/user/Video/00001.MTS' -preset 'slow' -b:v '500k' -maxrate '500k' -bufsize '1000k' -vf 'scale=0:480' -codec:v 'libx264' -profile:v 'main' -b:a '128k' -ac '2' -strict 'experimental' -codec:a 'aac' -ss '00:00:10' -to '00:00:30' -f 'mp4' -y './out/00001-virag_h264x1_480p_web.mp4' 2>&1 | tee /var/log/bulk_video_converter.bash/2016-05-02_05-14-20-695774334/00001-virag_h264x1_480p_web-1-mp4.ffmpeg.log 1>&2;
        # pass 1 done
      # passes done
    # profile virag_h264x1_480p_web done
  # /home/user/Video/00001.MTS done
  /home/user/Video/00001.MTS:
    profile virag_h264x1_pal_sd:
      global input:
        -ss '00:00:10' -threads '0' 
      # global input done
      video:
        -preset 'slower' -b:v '1000k' -vf 'scale=0:576' -codec:v 'libx264' -profile:v 'main' 
      # video done
      audio:
        -b:a '196k' -ac '6' -strict 'experimental' -codec:a 'aac' 
      # audio done
      global output:
        -ss '00:00:10' -to '00:00:30' 
      # global output done
      passes:
      # NOTICE 527:  bulk_video_converter.bash creates directory /var/log/bulk_video_converter.bash/2016-05-02_05-14-20-695774334
        pass 1:
          /usr/bin/ffmpeg -ss '00:00:10' -threads '0' -i '/home/user/Video/00001.MTS' -preset 'slower' -b:v '1000k' -vf 'scale=0:576' -codec:v 'libx264' -profile:v 'main' -b:a '196k' -ac '6' -strict 'experimental' -codec:a 'aac' -ss '00:00:10' -to '00:00:30' -f 'mp4' -y './out/00001-virag_h264x1_pal_sd.mp4' 2>&1 | tee /var/log/bulk_video_converter.bash/2016-05-02_05-14-20-695774334/00001-virag_h264x1_pal_sd-1-mp4.ffmpeg.log 1>&2;
        # pass 1 done
      # passes done
    # profile virag_h264x1_pal_sd done
  # /home/user/Video/00001.MTS done
# NOTICE 835:  bulk_video_converter.bash deletes directory /tmp/bulk_video_converter.bash
# bulk_video_converter.bash done
[user@host ~]$ 
```


## TODO

* Add error handling for parallel processes.

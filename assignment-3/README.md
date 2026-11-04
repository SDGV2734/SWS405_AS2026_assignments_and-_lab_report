# Assignment 3

![College of Science and Technology Logo.jpeg](Assignment%203/College_of_Science_and_Technology_Logo.jpeg)

$$
\huge \begin{array}{c} \mathbf{\textsf{Royal University of Bhutan,}} \\ \mathbf{\textsf{College of Science and Technology}} \end{array}
$$

$$
\Large \mathbf{\textsf{SWS405: Digital Forensics}}
$$

$$
\Large \mathbf{\textsf{Computing Technologies Department  }}
$$

$$
\large \mathbf{\textsf{Assignment 3  }}
$$

$$
\large \mathbf{\textsf{Submitted by: Sonam Dorji Ghalley  }}
$$

$$
\large \mathbf{\textsf{Student No: 02230299  }}
$$

## **1. Introduction**

This report examines four files from a fictional case at Research Lab 2. A staff member recorded a voice memo and phone video on 3 September 2026. Later, a WAV file described as an unedited memo and a shorter video were shared. The purpose of this investigation is to check whether the questioned files match the known originals. The case files are synthetic training evidence.

## **2. Objectives**

The investigation aims to preserve the evidence, compare the original and questioned audio files, compare the original and questioned video files, locate possible changes, and report findings supported by the actual examination.

## **3. Tools and evidence**

| Tool | Use |
| --- | --- |
| Kali Linux | Forensic working environment |
| SHA-256 / file | Integrity checks and file-type identification |
| FFprobe / FFmpeg | Inspect streams, frames and convert working copies |
| ExifTool | Inspect metadata |
| SoX | Spectrograms and audio measurements |
| Audacity / VLC | Listen, view and compare recordings |

| File | Role |
| --- | --- |
| audio_original.m4a | Known original voice memo |
| audio_questioned.wav | Questioned audio file |
| video_original.mov | Known original video |
| video_edited.mp4 | Questioned video file |

## **4. Method and results - Part 1: Evidence preservation**

### **Step 1. Install and check tools**

Install the necessary forensic software and noting the versions.

```bash
sudo apt update

sudo apt install ffmpeg sox libsox-fmt-all mediainfo libimage-exiftool-perl vlc audacity

ffmpeg -version

exiftool -ver

sox --version
```

```bash
sudo apt update

sudo apt install ffmpeg sox libsox-fmt-all mediainfo libimage-exiftool-perl vlc audacity

ffmpeg -version

exiftool -ver

sox --version
```

```bash
sudo apt update

sudo apt install ffmpeg sox libsox-fmt-all mediainfo libimage-exiftool-perl vlc audacity

ffmpeg -version

exiftool -ver

sox --version
```

```bash
sudo apt update

sudo apt install ffmpeg sox libsox-fmt-all mediainfo libimage-exiftool-perl vlc audacity

ffmpeg -version

exiftool -ver

sox --version
```

![image.png](Assignment%203/image.png)

### **Step 2. Create folders and extract evidence**

Use separate folders for original evidence and for analysis.

```bash
mkdir -p ~/demo/{Evidence,Working,Audio,Frames,Reports}

unzip SWS405_Demo_Audio_Video.zip -d ~/demo/Evidence

cd ~/demo
```

![image.png](Assignment%203/image%201.png)

### **Step 3. Verify evidence hashes**

Check the supplied SHA-256 manifest before editing or conversion.

```bash
cd demo/Evidence/SWS405_Assignment_03_Audio_Video && sha256sum -c SHA256SUMS.txt && cd ..
```

![image.png](Assignment%203/image%202.png)

### **Step 4. Create verified working copies**

Copy the files and compare their hashes with the evidence originals.

```bash
cp Evidence/SWS405_Assignment_03_Audio_Video/audio_* Evidence/SWS405_Assignment_03_Audio_Video/video_* Working/

sha256sum Evidence/SWS405_Assignment_03_Audio_Video/audio_original.m4a Working/audio_original.m4a

sha256sum Evidence/SWS405_Assignment_03_Audio_Video/audio_questioned.wav Working/audio_questioned.wav

sha256sum Evidence/SWS405_Assignment_03_Audio_Video/video_original.mov Working/video_original.mov

sha256sum Evidence/SWS405_Assignment_03_Audio_Video/video_edited.mp4 Working/video_edited.mp4
```

![image.png](Assignment%203/image%203.png)

### **Step 5. Identify real file types**

Check actual file signatures instead of trusting file extensions.

```bash
file Working/*
```

![image.png](Assignment%203/image%204.png)

## **5. Part 2: Audio examination**

### **Step 6. Listen to both recordings**

Play both recordings and compare spoken words, pauses and background sounds.

```bash
ffplay -autoexit -nodisp Working/audio_original.m4a

ffplay -autoexit -nodisp Working/audio_questioned.wav
```

![image.png](Assignment%203/image%205.png)

![image.png](Assignment%203/image%206.png)

### **Step 7. Compare audio properties**

Inspect format, duration, codec, sample rate and channel count.

```bash
ffprobe -v error -show_entries format=format_name,duration,bit_rate:stream=codec_name,sample_rate,channels -of default=noprint_wrappers=1 Working/audio_original.m4a

ffprobe -v error -show_entries format=format_name,duration,bit_rate:stream=codec_name,sample_rate,channels -of default=noprint_wrappers=1 Working/audio_questioned.wav
```

![image.png](Assignment%203/image%207.png)

| Property | original.m4a | questioned.wav |
| --- | --- | --- |
| Codec | aac | pcm_s16le |
| Sample rate | 48000 Hz | 44100 |
| Channels | 1 | 1 |
| Format name | mov,mp4,m4a,3gp,3g2,mj2 | wav |
| Duration | 16.618 s | 7.023764 s |
| Bit rate | 163461 Kb/s | 705754 kb/s |

### **Step 8. Check audio metadata**

Look for software, recording dates and descriptive comments.

```bash
exiftool -a -G1 Working/audio_original.m4a

exiftool -a -G1 Working/audio_questioned.wav
```

![image.png](Assignment%203/image%208.png)

![image.png](Assignment%203/image%209.png)

### **Step 9. Create audio analysis copies**

Create standard WAV working files for listening and comparison.

```bash
ffmpeg -i Working/audio_original.m4a -ar 44100 Audio/original_work.wav

cp Working/audio_questioned.wav Audio/questioned_work.wav
```

![image.png](Assignment%203/image%2010.png)

### **Step 10. Generate spectrograms**

Compare time and frequency patterns and inspect the suggested 4.44-second point.

```bash
sox Audio/original_work.wav -n spectrogram -x 1000 -z 90 -t "audio_original" -o Reports/spec_original.png

sox Audio/questioned_work.wav -n spectrogram -x 1000 -z 90 -t "audio_questioned" -o Reports/spec_questioned.png
```

Now checking for the image created from the command 

Original Audio spectrogram

![image.png](Assignment%203/image%2011.png)

Questioned audio Spectrogram

![image.png](Assignment%203/image%2012.png)

### **Step 11. Measure background levels**

Measure quiet intervals before and after the suspected join.

```bash
sox Audio/questioned_work.wav -n trim 0.1 0.6 stat 2>&1 | grep "RMS"

sox Audio/questioned_work.wav -n trim 6.2 0.7 stat 2>&1 | grep "RMS"
```

![image.png](Assignment%203/image%2013.png)

### **Step 12. Inspect possible electrical hum**

Inspect low-frequency audio for possible components near 50, 100 and 150 Hz.

```bash
sox Audio/questioned_work.wav -n remix 1 sinc 20-400 gain -n rate 1k spectrogram -X 100 -y 257 -z 85 -t "0-500 Hz" -o Reports/hum_questioned.png
```

![image.png](Assignment%203/image%2014.png)

### **Step 13. Compare waveform and speech in Audacity**

Import both WAV files, align speech as needed and zoom around the suspected change.

```bash
audacity
```

![image.png](Assignment%203/image%2015.png)

### **Step 14. Enhance a separate audio copy**

Use enhancement only on a working copy, then compare it with the unenhanced version.

```bash
ffmpeg -i Audio/questioned_work.wav -af "bandreject=f=50:width_type=h:w=6,bandreject=f=100:width_type=h:w=6,bandreject=f=150:width_type=h:w=6" Audio/questioned_enhanced.wav

ffplay -autoexit -nodisp Audio/questioned_enhanced.wav
```

![image.png](Assignment%203/image%2016.png)

![image.png](Assignment%203/image%2017.png)

## **6. Part 3: Video examination**

### **Step 15. View both videos**

Watch the original and questioned copies, especially around the laboratory door.

```bash
vlc Working/video_original.mov

vlc Working/video_edited.mp4
```

![image.png](Assignment%203/image%2018.png)

### **Step 16. Compare video properties**

Collect stream properties and save full FFprobe reports.

```bash
ffprobe -v error -show_entries format=format_name,duration,bit_rate:format_tags=creation_time,encoder:stream=codec_type,codec_name,width,height,r_frame_rate,nb_frames -of default=noprint_wrappers=1 Working/video_original.mov

ffprobe -v error -show_entries format=format_name,duration,bit_rate:format_tags=creation_time,encoder:stream=codec_type,codec_name,width,height,r_frame_rate,nb_frames -of default=noprint_wrappers=1 Working/video_edited.mp4

ffprobe -hide_banner -show_format -show_streams Working/video_original.mov > Reports/ffprobe_original.txt 2>&1

ffprobe -hide_banner -show_format -show_streams Working/video_edited.mp4 > Reports/ffprobe_edited.txt 2>&1
```

![image.png](Assignment%203/image%2019.png)

![image.png](Assignment%203/image%2020.png)

from `ffprobe_original.txt` 

```
Input #0, mov,mp4,m4a,3gp,3g2,mj2, from 'Working/video_original.mov':
  Metadata:
    major_brand     : qt  
    minor_version   : 512
    compatible_brands: qt  
    com.apple.quicktime.location.ISO6709: +27.4728+089.6390+2330.000/
    creation_time   : 2026-09-03T10:22:11.000000Z;2026-09-03T10:22:11Z
    com.apple.quicktime.make: Apple
    com.apple.quicktime.model: iPhone 15
    com.apple.quicktime.software: 17.6
    com.apple.quicktime.creationdate: 2026-09-03T16:22:11+0600
  Duration: 00:00:20.00, start: 0.000000, bitrate: 4192 kb/s
  Stream #0:0[0x1]: Video: h264 (High) (avc1 / 0x31637661), yuv420p(progressive), 1280x720, 4054 kb/s, 30 fps, 30 tbr, 15360 tbn (default)
    Metadata:
      creation_time   : 2026-09-03T10:22:11.000000Z
      handler_name    : Core Media Video
      vendor_id       : FFMP
      encoder         : H.264
  Stream #0:1[0x2]: Audio: aac (LC) (mp4a / 0x6134706D), 48000 Hz, mono, fltp, 128 kb/s (default)
    Metadata:
      creation_time   : 2026-09-03T10:22:11.000000Z
      handler_name    : Core Media Audio
      vendor_id       : [0][0][0][0]
[STREAM]
index=0
codec_name=h264
codec_long_name=H.264 / AVC / MPEG-4 AVC / MPEG-4 part 10
profile=High
codec_type=video
codec_tag_string=avc1
codec_tag=0x31637661
mime_codec_string=avc1.64001f
width=1280
height=720
coded_width=1280
coded_height=720
has_b_frames=2
sample_aspect_ratio=N/A
display_aspect_ratio=N/A
pix_fmt=yuv420p
level=31
color_range=unknown
color_space=unknown
color_transfer=unknown
color_primaries=unknown
chroma_location=left
field_order=progressive
is_avc=true
nal_length_size=4
id=0x1
r_frame_rate=30/1
avg_frame_rate=30/1
time_base=1/15360
start_pts=0
start_time=0.000000
duration_ts=307200
duration=20.000000
bit_rate=4054136
max_bit_rate=N/A
bits_per_raw_sample=8
nb_frames=600
nb_read_frames=N/A
nb_read_packets=N/A
extradata_size=46
DISPOSITION:default=1
DISPOSITION:dub=0
DISPOSITION:original=0
DISPOSITION:comment=0
DISPOSITION:lyrics=0
DISPOSITION:karaoke=0
DISPOSITION:forced=0
DISPOSITION:hearing_impaired=0
DISPOSITION:visual_impaired=0
DISPOSITION:clean_effects=0
DISPOSITION:attached_pic=0
DISPOSITION:timed_thumbnails=0
DISPOSITION:non_diegetic=0
DISPOSITION:captions=0
DISPOSITION:descriptions=0
DISPOSITION:metadata=0
DISPOSITION:dependent=0
DISPOSITION:still_image=0
DISPOSITION:multilayer=0
TAG:creation_time=2026-09-03T10:22:11.000000Z
TAG:handler_name=Core Media Video
TAG:vendor_id=FFMP
TAG:encoder=H.264
[/STREAM]
[STREAM]
index=1
codec_name=aac
codec_long_name=AAC (Advanced Audio Coding)
profile=LC
codec_type=audio
codec_tag_string=mp4a
codec_tag=0x6134706d
mime_codec_string=mp4a.40.2
sample_fmt=fltp
sample_rate=48000
channels=1
channel_layout=mono
bits_per_sample=0
initial_padding=1024
id=0x2
r_frame_rate=0/0
avg_frame_rate=0/0
time_base=1/48000
start_pts=0
start_time=0.000000
duration_ts=960000
duration=20.000000
bit_rate=128860
max_bit_rate=N/A
bits_per_raw_sample=N/A
nb_frames=939
nb_read_frames=N/A
nb_read_packets=N/A
extradata_size=5
DISPOSITION:default=1
DISPOSITION:dub=0
DISPOSITION:original=0
DISPOSITION:comment=0
DISPOSITION:lyrics=0
DISPOSITION:karaoke=0
DISPOSITION:forced=0
DISPOSITION:hearing_impaired=0
DISPOSITION:visual_impaired=0
DISPOSITION:clean_effects=0
DISPOSITION:attached_pic=0
DISPOSITION:timed_thumbnails=0
DISPOSITION:non_diegetic=0
DISPOSITION:captions=0
DISPOSITION:descriptions=0
DISPOSITION:metadata=0
DISPOSITION:dependent=0
DISPOSITION:still_image=0
DISPOSITION:multilayer=0
TAG:creation_time=2026-09-03T10:22:11.000000Z
TAG:handler_name=Core Media Audio
TAG:vendor_id=[0][0][0][0]
[/STREAM]
[FORMAT]
filename=Working/video_original.mov
nb_streams=2
nb_programs=0
nb_stream_groups=0
format_name=mov,mp4,m4a,3gp,3g2,mj2
format_long_name=QuickTime / MOV
start_time=0.000000
duration=20.000000
size=10480469
bit_rate=4192187
probe_score=100
TAG:major_brand=qt  
TAG:minor_version=512
TAG:compatible_brands=qt  
TAG:com.apple.quicktime.location.ISO6709=+27.4728+089.6390+2330.000/
TAG:creation_time=2026-09-03T10:22:11.000000Z;2026-09-03T10:22:11Z
TAG:com.apple.quicktime.make=Apple
TAG:com.apple.quicktime.model=iPhone 15
TAG:com.apple.quicktime.software=17.6
TAG:com.apple.quicktime.creationdate=2026-09-03T16:22:11+0600
[/FORMAT]
```

from `ffprobe_editied.txt`

```
Input #0, mov,mp4,m4a,3gp,3g2,mj2, from 'Working/video_edited.mp4':
  Metadata:
    major_brand     : isom
    minor_version   : 512
    compatible_brands: isomiso2avc1mp41
    creation_time   : 2026-09-05T02:41:07.000000Z
    encoder         : Lavf60.16.100
  Duration: 00:00:16.00, start: 0.000000, bitrate: 278 kb/s
  Stream #0:0[0x1](und): Video: h264 (High) (avc1 / 0x31637661), yuv420p(progressive), 1280x720, 173 kb/s, 30 fps, 30 tbr, 15360 tbn (default)
    Metadata:
      creation_time   : 2026-09-05T02:41:07.000000Z
      handler_name    : VideoHandler
      encoder         : Lavc60.31.102 libx264
  Stream #0:1[0x2](und): Audio: aac (LC) (mp4a / 0x6134706D), 48000 Hz, mono, fltp, 96 kb/s (default)
    Metadata:
      creation_time   : 2026-09-05T02:41:07.000000Z
      handler_name    : SoundHandler
[STREAM]
index=0
codec_name=h264
codec_long_name=H.264 / AVC / MPEG-4 AVC / MPEG-4 part 10
profile=High
codec_type=video
codec_tag_string=avc1
codec_tag=0x31637661
mime_codec_string=avc1.64001f
width=1280
height=720
coded_width=1280
coded_height=720
has_b_frames=2
sample_aspect_ratio=N/A
display_aspect_ratio=N/A
pix_fmt=yuv420p
level=31
color_range=unknown
color_space=unknown
color_transfer=unknown
color_primaries=unknown
chroma_location=left
field_order=progressive
is_avc=true
nal_length_size=4
id=0x1
r_frame_rate=30/1
avg_frame_rate=30/1
time_base=1/15360
start_pts=0
start_time=0.000000
duration_ts=245760
duration=16.000000
bit_rate=173034
max_bit_rate=N/A
bits_per_raw_sample=8
nb_frames=480
nb_read_frames=N/A
nb_read_packets=N/A
extradata_size=45
DISPOSITION:default=1
DISPOSITION:dub=0
DISPOSITION:original=0
DISPOSITION:comment=0
DISPOSITION:lyrics=0
DISPOSITION:karaoke=0
DISPOSITION:forced=0
DISPOSITION:hearing_impaired=0
DISPOSITION:visual_impaired=0
DISPOSITION:clean_effects=0
DISPOSITION:attached_pic=0
DISPOSITION:timed_thumbnails=0
DISPOSITION:non_diegetic=0
DISPOSITION:captions=0
DISPOSITION:descriptions=0
DISPOSITION:metadata=0
DISPOSITION:dependent=0
DISPOSITION:still_image=0
DISPOSITION:multilayer=0
TAG:creation_time=2026-09-05T02:41:07.000000Z
TAG:language=und
TAG:handler_name=VideoHandler
TAG:encoder=Lavc60.31.102 libx264
[/STREAM]
[STREAM]
index=1
codec_name=aac
codec_long_name=AAC (Advanced Audio Coding)
profile=LC
codec_type=audio
codec_tag_string=mp4a
codec_tag=0x6134706d
mime_codec_string=mp4a.40.2
sample_fmt=fltp
sample_rate=48000
channels=1
channel_layout=mono
bits_per_sample=0
initial_padding=1024
id=0x2
r_frame_rate=0/0
avg_frame_rate=0/0
time_base=1/48000
start_pts=0
start_time=0.000000
duration_ts=768000
duration=16.000000
bit_rate=96179
max_bit_rate=N/A
bits_per_raw_sample=N/A
nb_frames=751
nb_read_frames=N/A
nb_read_packets=N/A
extradata_size=5
DISPOSITION:default=1
DISPOSITION:dub=0
DISPOSITION:original=0
DISPOSITION:comment=0
DISPOSITION:lyrics=0
DISPOSITION:karaoke=0
DISPOSITION:forced=0
DISPOSITION:hearing_impaired=0
DISPOSITION:visual_impaired=0
DISPOSITION:clean_effects=0
DISPOSITION:attached_pic=0
DISPOSITION:timed_thumbnails=0
DISPOSITION:non_diegetic=0
DISPOSITION:captions=0
DISPOSITION:descriptions=0
DISPOSITION:metadata=0
DISPOSITION:dependent=0
DISPOSITION:still_image=0
DISPOSITION:multilayer=0
TAG:creation_time=2026-09-05T02:41:07.000000Z
TAG:language=und
TAG:handler_name=SoundHandler
[/STREAM]
[FORMAT]
filename=Working/video_edited.mp4
nb_streams=2
nb_programs=0
nb_stream_groups=0
format_name=mov,mp4,m4a,3gp,3g2,mj2
format_long_name=QuickTime / MOV
start_time=0.000000
duration=16.000000
size=557691
bit_rate=278845
probe_score=100
TAG:major_brand=isom
TAG:minor_version=512
TAG:compatible_brands=isomiso2avc1mp41
TAG:creation_time=2026-09-05T02:41:07.000000Z
TAG:encoder=Lavf60.16.100
[/FORMAT]
```

| Field | video_original.mov | video_edited.mp4 |
| --- | --- | --- |
| Container / codecs | QuickTime MOV (brand qt) | MP4 (brands isom, iso2, avc1, mp41) |
| Resolution, frame rate | 1280 × 720, 30 fps | 1280 × 720, 30 fps |
| Duration | 20.000 s | 16.000 s |
| Video frames | 600 | 480 |
| Audio | AAC, 48 kHz, mono | AAC, 48 kHz, mono |
| Bit rate | 4,192 kb/s | 279 kb/s |
| Create date (UTC) | 2026-09-03 10:22:11 | 2026-09-05 02:41:07 |
| Make / model / software | Apple / iPhone 15 / 17.6 | none |
| GPS | 27°28'22.08" N, 89°38'20.40" E | none |
| Encoder / compressor | H.264 (Apple), handler "Core
Media Video" | Lavf60.16.100 / Lavc60.31.102
libx264, handler "VideoHandler" |

### **Step 17. Compare video metadata**

Look for device, GPS, dates and encoder fields.

```bash
exiftool -ee -G3 Working/video_original.mov > Reports/exif_original.txt

exiftool -ee -G3 Working/video_edited.mp4 > Reports/exif_edited.txt

exiftool -Make -Model -Software -GPSPosition -CreationDate -CompressorName Working/video_original.mov Working/video_edited.mp4
```

output from `exif-original.txt` 

```
[Main]          ExifTool Version Number         : 13.55
[Main]          File Name                       : video_original.mov
[Main]          Directory                       : Working
[Main]          File Size                       : 10 MB
[Main]          File Modification Date/Time     : 2026:11:03 15:57:08-05:00
[Main]          File Access Date/Time           : 2026:11:03 15:57:50-05:00
[Main]          File Inode Change Date/Time     : 2026:11:03 15:57:08-05:00
[Main]          File Permissions                : -rw-r--r--
[Main]          File Type                       : MOV
[Main]          File Type Extension             : mov
[Main]          MIME Type                       : video/quicktime
[Main]          Major Brand                     : Apple QuickTime (.MOV/QT)
[Main]          Minor Version                   : 0.2.0
[Main]          Compatible Brands               : qt
[Main]          Media Data Size                 : 10457835
[Main]          Media Data Offset               : 36
[Main]          Movie Header Version            : 0
[Main]          Create Date                     : 2026:09:03 10:22:11
[Main]          Modify Date                     : 2026:09:03 10:22:11
[Main]          Time Scale                      : 1000
[Main]          Duration                        : 20.00 s
[Main]          Preferred Rate                  : 1
[Main]          Preferred Volume                : 100.00%
[Main]          Matrix Structure                : 1 0 0 0 1 0 0 0 1
[Main]          Preview Time                    : 0 s
[Main]          Preview Duration                : 0 s
[Main]          Poster Time                     : 0 s
[Main]          Selection Time                  : 0 s
[Main]          Selection Duration              : 0 s
[Main]          Current Time                    : 0 s
[Main]          Next Track ID                   : 3
[Main]          Track Header Version            : 0
[Main]          Track Create Date               : 2026:09:03 10:22:11
[Main]          Track Modify Date               : 2026:09:03 10:22:11
[Main]          Track ID                        : 1
[Main]          Track Duration                  : 20.00 s
[Main]          Track Layer                     : 0
[Main]          Track Volume                    : 0.00%
[Main]          Matrix Structure                : 1 0 0 0 1 0 0 0 1
[Main]          Image Width                     : 1280
[Main]          Image Height                    : 720
[Main]          Media Header Version            : 0
[Main]          Media Create Date               : 2026:09:03 10:22:11
[Main]          Media Modify Date               : 2026:09:03 10:22:11
[Main]          Media Time Scale                : 15360
[Main]          Media Duration                  : 20.00 s
[Main]          Media Language Code             : Unknown (32767)
[Main]          Handler Class                   : Media Handler
[Main]          Handler Type                    : Video Track
[Main]          Handler Description             : Core Media Video
[Main]          Graphics Mode                   : srcCopy
[Main]          Op Color                        : 0 0 0
[Main]          Handler Class                   : Data Handler
[Main]          Handler Type                    : URL
[Main]          Handler Description             : DataHandler
[Main]          Compressor ID                   : avc1
[Main]          Vendor ID                       : FFmpeg
[Main]          Source Image Width              : 1280
[Main]          Source Image Height             : 720
[Main]          X Resolution                    : 72
[Main]          Y Resolution                    : 72
[Main]          Compressor Name                 : H.264
[Main]          Bit Depth                       : 24
[Main]          Video Frame Rate                : 30
[Main]          Track Header Version            : 0
[Main]          Track Create Date               : 2026:09:03 10:22:11
[Main]          Track Modify Date               : 2026:09:03 10:22:11
[Main]          Track ID                        : 2
[Main]          Track Duration                  : 20.00 s
[Main]          Track Layer                     : 0
[Main]          Track Volume                    : 100.00%
[Main]          Matrix Structure                : 1 0 0 0 1 0 0 0 1
[Main]          Media Header Version            : 0
[Main]          Media Create Date               : 2026:09:03 10:22:11
[Main]          Media Modify Date               : 2026:09:03 10:22:11
[Main]          Media Time Scale                : 48000
[Main]          Media Duration                  : 20.02 s
[Main]          Media Language Code             : Unknown (32767)
[Main]          Handler Class                   : Media Handler
[Main]          Handler Type                    : Audio Track
[Main]          Handler Description             : Core Media Audio
[Main]          Balance                         : 0
[Main]          Handler Class                   : Data Handler
[Main]          Handler Type                    : URL
[Main]          Handler Description             : DataHandler
[Main]          Audio Format                    : mp4a
[Main]          Audio Channels                  : 1
[Main]          Audio Bits Per Sample           : 16
[Main]          Audio Sample Rate               : 48000
[Main]          Purchase File Format            : mp4a
[Main]          Layout Flags                    : Mono
[Main]          Audio Channels                  : 1
[Main]          Handler Type                    : Metadata Tags
[Main]          GPS Coordinates                 : 27 deg 28' 22.08" N, 89 deg 38' 20.40" E, 2330 m Above Sea Level
[Main]          Creation Time                   : 2026:09:03 10:22:11Z
[Main]          Make                            : Apple
[Main]          Model                           : iPhone 15
[Main]          Software                        : 17.6
[Main]          Creation Date                   : 2026:09:03 16:22:11+06:00
[Main]          Image Size                      : 1280x720
[Main]          Megapixels                      : 0.922
[Main]          Avg Bitrate                     : 4.18 Mbps
[Main]          GPS Altitude                    : 2330 m
[Main]          GPS Altitude Ref                : Above Sea Level
[Main]          GPS Latitude                    : 27 deg 28' 22.08" N
[Main]          GPS Longitude                   : 89 deg 38' 20.40" E
[Main]          Rotation                        : 0
[Main]          GPS Position                    : 27 deg 28' 22.08" N, 89 deg 38' 20.40" E
```

Output from `exif_edited.txt` 

```
[Main]          ExifTool Version Number         : 13.55
[Main]          File Name                       : video_edited.mp4
[Main]          Directory                       : Working
[Main]          File Size                       : 558 kB
[Main]          File Modification Date/Time     : 2026:11:03 15:57:08-05:00
[Main]          File Access Date/Time           : 2026:11:03 15:57:59-05:00
[Main]          File Inode Change Date/Time     : 2026:11:03 15:57:08-05:00
[Main]          File Permissions                : -rw-r--r--
[Main]          File Type                       : MP4
[Main]          File Type Extension             : mp4
[Main]          MIME Type                       : video/mp4
[Main]          Major Brand                     : MP4 Base Media v1 [IS0 14496-12:2003]
[Main]          Minor Version                   : 0.2.0
[Main]          Compatible Brands               : isom, iso2, avc1, mp41
[Main]          Movie Header Version            : 0
[Main]          Create Date                     : 2026:09:05 02:41:07
[Main]          Modify Date                     : 2026:09:05 02:41:07
[Main]          Time Scale                      : 1000
[Main]          Duration                        : 16.00 s
[Main]          Preferred Rate                  : 1
[Main]          Preferred Volume                : 100.00%
[Main]          Matrix Structure                : 1 0 0 0 1 0 0 0 1
[Main]          Preview Time                    : 0 s
[Main]          Preview Duration                : 0 s
[Main]          Poster Time                     : 0 s
[Main]          Selection Time                  : 0 s
[Main]          Selection Duration              : 0 s
[Main]          Current Time                    : 0 s
[Main]          Next Track ID                   : 3
[Main]          Track Header Version            : 0
[Main]          Track Create Date               : 2026:09:05 02:41:07
[Main]          Track Modify Date               : 2026:09:05 02:41:07
[Main]          Track ID                        : 1
[Main]          Track Duration                  : 16.00 s
[Main]          Track Layer                     : 0
[Main]          Track Volume                    : 0.00%
[Main]          Matrix Structure                : 1 0 0 0 1 0 0 0 1
[Main]          Image Width                     : 1280
[Main]          Image Height                    : 720
[Main]          Media Header Version            : 0
[Main]          Media Create Date               : 2026:09:05 02:41:07
[Main]          Media Modify Date               : 2026:09:05 02:41:07
[Main]          Media Time Scale                : 15360
[Main]          Media Duration                  : 16.00 s
[Main]          Media Language Code             : und
[Main]          Handler Type                    : Video Track
[Main]          Handler Description             : VideoHandler
[Main]          Graphics Mode                   : srcCopy
[Main]          Op Color                        : 0 0 0
[Main]          Compressor ID                   : avc1
[Main]          Source Image Width              : 1280
[Main]          Source Image Height             : 720
[Main]          X Resolution                    : 72
[Main]          Y Resolution                    : 72
[Main]          Compressor Name                 : Lavc60.31.102 libx264
[Main]          Bit Depth                       : 24
[Main]          Buffer Size                     : 0
[Main]          Max Bitrate                     : 173034
[Main]          Average Bitrate                 : 173034
[Main]          Video Frame Rate                : 30
[Main]          Track Header Version            : 0
[Main]          Track Create Date               : 2026:09:05 02:41:07
[Main]          Track Modify Date               : 2026:09:05 02:41:07
[Main]          Track ID                        : 2
[Main]          Track Duration                  : 16.00 s
[Main]          Track Layer                     : 0
[Main]          Track Volume                    : 100.00%
[Main]          Matrix Structure                : 1 0 0 0 1 0 0 0 1
[Main]          Media Header Version            : 0
[Main]          Media Create Date               : 2026:09:05 02:41:07
[Main]          Media Modify Date               : 2026:09:05 02:41:07
[Main]          Media Time Scale                : 48000
[Main]          Media Duration                  : 16.02 s
[Main]          Media Language Code             : und
[Main]          Handler Type                    : Audio Track
[Main]          Handler Description             : SoundHandler
[Main]          Balance                         : 0
[Main]          Audio Format                    : mp4a
[Main]          Audio Channels                  : 1
[Main]          Audio Bits Per Sample           : 16
[Main]          Audio Sample Rate               : 48000
[Main]          Buffer Size                     : 0
[Main]          Max Bitrate                     : 96179
[Main]          Average Bitrate                 : 96179
[Main]          Handler Type                    : Metadata
[Main]          Handler Vendor ID               : Apple
[Main]          Encoder                         : Lavf60.16.100
[Main]          Media Data Size                 : 538684
[Main]          Media Data Offset               : 19007
[Main]          Image Size                      : 1280x720
[Main]          Megapixels                      : 0.922
[Main]          Avg Bitrate                     : 269 kbps
[Main]          Rotation                        : 0
```

![image.png](Assignment%203/image%2021.png)

### **Step 18. Check frame types (GOP)**

Inspect I-, P- and B-frame patterns of both videos.

```bash
ffprobe -v error -select_streams v:0 -show_entries frame=best_effort_timestamp_time,pict_type -of csv=p=0 Working/video_original.mov > Reports/frames_original.csv

ffprobe -v error -select_streams v:0 -show_entries frame=best_effort_timestamp_time,pict_type -of csv=p=0 Working/video_edited.mp4 > Reports/frames_edited.csv

grep -c I Reports/frames_original.csv

grep -c I Reports/frames_edited.csv
```

![image.png](Assignment%203/image%2022.png)

### **Step 19. Compare decoded frame hashes**

Create frame-level MD5 reports to test whether decoded pictures match byte for byte.

```bash
ffmpeg -i Working/video_original.mov -map 0:v -f framemd5 Reports/original.md5

ffmpeg -i Working/video_edited.mp4 -map 0:v -f framemd5 Reports/edited.md5

awk -F, '!/^#/{print $NF}' Reports/original.md5 | sort > Reports/h1.txt

awk -F, '!/^#/{print $NF}' Reports/edited.md5 | sort > Reports/h2.txt

comm -12 Reports/h1.txt Reports/h2.txt | wc -l
```

![image.png](Assignment%203/image%2023.png)

![image.png](Assignment%203/image%2024.png)

## **Step 20. Extract video frames and inspect a cut**

Review frames near the suspected timeline jump.

```bash
ffmpeg -i Working/video_edited.mp4 -vf fps=1 Frames/edited_%04d.png

ffmpeg -i Working/video_edited.mp4 -vf "select='between(n,269,270)',scale=640:-1,tile=2x1" -frames:v 1 Reports/cut_frames.png
```

![image.png](Assignment%203/image%2025.png)

![image.png](Assignment%203/image%2026.png)

`cut_frame.png`

![image.png](Assignment%203/image%2027.png)

## **Step 21. Compare video soundtracks**

Extract each soundtrack and inspect audio events such as beeps and footsteps.

```bash
ffmpeg -i Working/video_original.mov -vn -acodec pcm_s16le Audio/video_original_audio.wav

ffmpeg -i Working/video_edited.mp4 -vn -acodec pcm_s16le Audio/video_edited_audio.wav

sox Audio/video_original_audio.wav -n spectrogram -x 1000 -z 80 -o Reports/events_original.png

sox Audio/video_edited_audio.wav -n spectrogram -x 800 -z 80 -o Reports/events_edited.png
```

Two audio extracted

![image.png](Assignment%203/image%2028.png)

`events_original.png`

![image.png](Assignment%203/image%2029.png)

`events_edited.png`

![image.png](Assignment%203/image%2030.png)

## **7. Part 4: Close the case**

### **Step 22. Recheck evidence integrity**

From the demo directory, verify the nested evidence folder hashes again after analysis. Record the timestamp and actual result.

```bash
cd Evidence/SWS405_Assignment_03_Audio_Video && sha256sum -c SHA256SUMS.txt
```

![image.png](Assignment%203/image%2031.png)

## **8. Investigation log**

Investigation log reconstructed from Kali shell history entries #1121–1182. Shell history establishes which commands were entered, but not whether they succeeded or what outputs they produced. Timestamps were not included; repeated attempts are retained.

| Date/time | Step or command | Result / issue |
| --- | --- | --- |
| Not captured
#1121–1127 | Checked folders; attempted SHA-256 verification with different paths; then changed directories | Multiple path attempts are recorded. Success/failure cannot be confirmed from history alone. |
| Not captured
#1128–1132 | Copied all four files to Working/; ran SHA-256 hashes on evidence and working copies | Copy and comparison commands recorded; hash values not supplied. |
| Not captured
#1133 | file Working/* | File identification command recorded; detected types not supplied. |
| Not captured
#1134–1135 | Played the original and questioned audio using ffplay | Both playback commands recorded; listening observations not supplied. |
| Not captured
#1136–1139 | Inspected both audio files with ffprobe and ExifTool | Property and metadata commands recorded; outputs not supplied. |
| Not captured
#1140–1143 | Created WAV working copies and generated two SoX spectrograms | Commands recorded; output images not yet supplied. |
| Not captured
#1144–1146 | Measured pre/post RMS and generated low-frequency spectrogram | RMS commands use grep "RMS"; readings and image not supplied. |
| Not captured
#1147–1149 | Opened Audacity, applied three notch filters to a separate audio copy, played enhanced copy | Commands recorded; waveform and listening observations not supplied. |
| Not captured
#1150–1153 | Opened original and edited videos in VLC twice | Repeated viewing is recorded; visual observations not supplied. |
| Not captured
#1154–1160 | Inspected video metadata with ffprobe, including repeat commands; exported full stream reports | Commands recorded; values and saved reports not yet supplied. |
| Not captured
#1161–1163 | Exported video ExifTool reports and requested selected metadata fields | Commands recorded; values and saved reports not yet supplied. |
| Not captured
#1164–1167 | Exported video frame-type CSVs and counted I-frames | Commands recorded; count results not supplied. |
| Not captured
#1168–1172 | Created framemd5 reports; extracted, sorted and compared decoded-frame hashes | Commands recorded; matching-frame count not supplied. |
| Not captured
#1173–1174 | Extracted one frame per second and generated a side-by-side suspected-cut image | Commands recorded; frame image not supplied. |
| Not captured
#1175–1178 | Extracted both video audio tracks and generated event spectrograms | Commands recorded; spectrogram images not supplied. |
| Not captured
#1179 | Re-ran SHA-256 manifest verification after analysis | Final hash-verification command recorded; output not supplied. |
| Not captured
#1180–1182 | Returned to home, saved command history, and navigated to demo | History export recorded; history does not include command timestamps. |

## **9. Summary of findings**

| Area | Evidence-based observation |
| --- | --- |
| Evidence integrity | The initial SHA-256 check returned OK for audio_original.m4a, audio_questioned.wav, video_original.mov and video_edited.mp4. This confirms that the evidence files matched the supplied hash manifest at that check. The screenshot alone does not establish that the final post-analysis verification also passed. |
| Audio duration and encoding | FFprobe reported 16.617 s, AAC, 48,000 Hz, mono and 163,471 b/s for the original; 7.023764 s, PCM 16-bit, 44,100 Hz, mono and 705,754 b/s for the questioned file. They are not identical media representations. |
| Audio metadata | The questioned WAV metadata includes Software: Lavf60.16.100 and the comment “Original recording - unedited”. The software field is consistent with FFmpeg-related processing, while the comment is only a claim and does not verify authenticity. |
| Audio spectrogram | The questioned audio spectrogram shows an abrupt rise in broadband background energy near 4.44 s and an apparent frequency ceiling around 16 kHz. The original has a longer, different spectral pattern. These features are consistent with editing or processing; they do not establish the precise prior codec. |
| Noise-level comparison | SoX RMS amplitude for the questioned file was 0.003430 during 0.1–0.7 s and 0.011149 during 6.2–6.9 s. This is about 3.25 times the amplitude, or 10.24 dB higher in the later interval. The result supports a substantial level change, assuming both chosen sections represent comparable background noise. |
| Video stream comparison | FFprobe reported 20.000 s and 600 video frames for the original MOV, versus 16.000 s and 480 frames for the questioned MP4, both 1280 × 720 at 30 fps using H.264 video and AAC audio. The questioned video was four seconds shorter. |
| Video software and timestamps | The original video has a creation-time tag of 2026-09-03T10:22:11Z. The questioned video has a creation-time tag of 2026-09-05T02:41:07Z and encoder Lavf60.16.100. These tags support later processing but can be modified. |
| Visible frame discontinuity | The side-by-side extracted frames show on-screen counters FRAME 0270 and FRAME 0391 and recording clocks 16:22:19 and 16:22:24. The person standing near the closed laboratory door is absent from the later view, where the door is open. This is consistent with a removed section of approximately 120 frames (4 seconds at 30 fps). |
| Video soundtrack | The supplied original and questioned soundtrack spectrograms differ: the original shows a bright sound event near 10–11 s, while the questioned recording shows a shorter timeline and a prominent event near 10 s. The event identities (e.g., card-reader beeps) have not been independently verified by listening in the submitted evidence. |
| Outstanding verification | The submitted screenshots do not show the numerical I-frame counts, framemd5 match count, full video ExifTool data, the final SHA-256 recheck, an Audacity waveform close-up, or a verified word-for-word listening transcript. Those claims remain unconfirmed. |

## **10. Conclusion**

The supplied screenshots show that the questioned audio and video are not unchanged copies of the corresponding originals. The audio is shorter, has different stream properties, carries FFmpeg-related metadata, and shows an abrupt background-level change near 4.44 seconds. The video is four seconds shorter, has 120 fewer video frames, and contains a visible frame-counter jump across the laboratory door scene. Together, these observations support editing or processing. They do not identify the person responsible or prove an intention to mislead. Additional outputs are still needed to confirm final evidence integrity, detailed frame-hash findings and speech/event descriptions.

## **11. Limitations**

Metadata can be changed or removed. Recording conditions and legitimate transcoding can cause differences in format, frequency content, background noise and frame encoding. No single test proves deliberate manipulation. Conclusions should be based on the combination of verified results and should not exceed what the evidence supports.
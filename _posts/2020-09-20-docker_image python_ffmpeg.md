---
title:  "django project docker image에 올리기(3)"
date:   2020-09-20 13:15:50 +0900
categories: 
  - test
tags: [Docker, Image Build, 실습]
toc: true
toc_sticky: true
toc_label: "Python 서버에서 ffmpeg 세팅"
---



# Python + ffmpeg 설치

# ffmpeg 설치

## #1. ffmpeg 설치하기

```
root@98773843bb34:/home/django_playlist# apt-get update
root@98773843bb34:/home/django_playlist# apt-get upgrade

root@98773843bb34:/home/django_playlist# apt-get install ffmpeg

root@98773843bb34:/home/django_playlist# ffmpeg -version
ffmpeg version 4.1.6-1~deb10u1 Copyright (c) 2000-2020 the FFmpeg developers
built with gcc 8 (Debian 8.3.0-6)
configuration: --prefix=/usr --extra-version='1~deb10u1' --toolchain=hardened --libdir=/usr/lib/x86_64-linux-gnu --incdir=/usr/include/x86_64-linux-gnu --arch=amd64 --enable-gpl --disable-stripping --enable-avresample --disable-filter=resample --enable-avisynth --enable-gnutls --enable-ladspa --enable-libaom --enable-libass --enable-libbluray --enable-libbs2b --enable-libcaca --enable-libcdio --enable-libcodec2 --enable-libflite --enable-libfontconfig --enable-libfreetype --enable-libfribidi --enable-libgme --enable-libgsm --enable-libjack --enable-libmp3lame --enable-libmysofa --enable-libopenjpeg --enable-libopenmpt --enable-libopus --enable-libpulse --enable-librsvg --enable-librubberband --enable-libshine --enable-libsnappy --enable-libsoxr --enable-libspeex --enable-libssh --enable-libtheora --enable-libtwolame --enable-libvidstab --enable-libvorbis --enable-libvpx --enable-libwavpack --enable-libwebp --enable-libx265 --enable-libxml2 --enable-libxvid --enable-libzmq --enable-libzvbi --enable-lv2 --enable-omx --enable-openal --enable-opengl --enable-sdl2 --enable-libdc1394 --enable-libdrm --enable-libiec61883 --enable-chromaprint --enable-frei0r --enable-libx264 --enable-shared
libavutil      56. 22.100 / 56. 22.100
libavcodec     58. 35.100 / 58. 35.100
libavformat    58. 20.100 / 58. 20.100
libavdevice    58.  5.100 / 58.  5.100
libavfilter     7. 40.101 /  7. 40.101
libavresample   4.  0.  0 /  4.  0.  0
libswscale      5.  3.100 /  5.  3.100
libswresample   3.  3.100 /  3.  3.100
libpostproc    55.  3.100 / 55.  3.100
```

## #2. 서버 실행해서 ffmpeg 테스트

```
root@98773843bb34:/home/django_playlist# python manage.py runserver 0.0.0.0:8000
			:
			:
[19/Sep/2020 21:48:45] "GET /song/new/ HTTP/1.1" 200 11600
video_url https://www.youtube.com/watch?v=4yKNsJWXwTI
[youtube] 4yKNsJWXwTI: Downloading webpage
[youtube] 4yKNsJWXwTI: Downloading js player 4b1ba5ea
[youtube] 4yKNsJWXwTI: Downloading js player 4b1ba5ea
[download] plist/static/plist/audio/행성 (This Night).webm has already been downloaded
[download] 100% of 3.43MiB
[ffmpeg] Destination: plist/static/plist/audio/행성 (This Night).mp3
Deleting original file plist/static/plist/audio/행성 (This Night).webm (pass -k to keep)
0:0
3:35
0 215000
```

<br>

---

# ffmpeg 에러 발견!

## mp3_slice.py에서 노래를 슬라이스하지 못한다!!

```
error:
Internal Server Error: /song/new/
Traceback (most recent call last):
  File "/usr/local/lib/python3.8/site-packages/urllib3/connection.py", line 159, in _new_conn
    conn = connection.create_connection(
  File "/usr/local/lib/python3.8/site-packages/urllib3/util/connection.py", line 84, in create_connection
    raise err
  File "/usr/local/lib/python3.8/site-packages/urllib3/util/connection.py", line 74, in create_connection
    sock.connect(sa)
ConnectionRefusedError: [Errno 111] Connection refused

During handling of the above exception, another exception occurred:
```

==> 원인 : pydub MemoryError    해결방법 : 찾지못함..

==> 우회로 ffmpeg에서 mp3 슬라이스가 가능하여 여기서 mp3 슬라이스 하기로 함!



## #1. ffmpeg로 mp3 파일 수정하기

> ffmpeg 의 단점 : 
>
> 1) file overwrite가 안된다
>
> ==> 원본 파일이름을  {song_title}_ori.mp3로 저장하고 수정한 파일을 {song_title}.mp3로 저장한다
>
> 2) song_title 에 소괄호 () 가 들어가면 syntax error 발생한다
>
> ===> **추후에 수정하기로!**

### (1) mp3_download.py 수정하기

```
def download_video_and_subtitle(video_url, file_name):
    download_path = os.path.join(VIDEO_DOWNLOAD_PATH, file_name+'.%(ext)s')


====> 원본 파일이름을  {song_title}_ori.mp3로 저장하도록 수정하기 <=====

def download_video_and_subtitle(video_url, file_name):
    download_path = os.path.join(VIDEO_DOWNLOAD_PATH, file_name+'_ori.%(ext)s')

```

### (2) mp3_slice.py 수정하기

- find_sec 함수 수정하기

```
def find_sec(input_time):
		:
	if len(min_sec) ==3 :
        hour = int(min_sec[0])
        min = int(min_sec[1])
        sec = int(min_sec[2])
    else :
        hour = 0
        min = int(min_sec[0])
        sec = int(min_sec[1])

    return int(hour)*60*60*1000 + int(min) * 60 * 1000 + int(sec) * 1000	
    
    
====> 시간을 초단위로 계산 ---> 시:분:초  형태로 변경 <=====
    
def find_sec(input_time):
		:    
    if len(min_sec) ==3 :
        return input_time
    else :
        return '00:'+input_time
```

- song_slice 함수 수정하기

```
def song_slice(song_name,start,end):
 		:
    try:
        # Opening file and extracting segment
        audio_file = STATIC_SONG_PATH+song_name+'.mp3'
        song = AudioSegment.from_mp3(audio_file)
        extract = song[startTime:endTime]

        # Saving
        extract.export(audio_file, format='mp3')
    except Exception as e:
        print('error:',e)


def song_slice(song_name,start,end):
 		:
    try:
        # Opening file and extracting segment
        audio_file = STATIC_SONG_PATH+song_name

        # mp3 file slice   
        os.system(f'ffmpeg -ss {startTime} -to {endTime} -i {audio_file}"_ori.mp3" \
        -c copy {audio_file}".mp3"')
        # STATIC_SONG_PATH 로 경로 이동
        os.system(f'cd {STATIC_SONG_PATH}')
        # 수정되기 전 원본파일 제거
        os.system(f'rm {audio_file}_ori.mp3')
        print("finish mp3 slice!")
    except Exception as e:
        print('error:',e)
```





### 참고 사이트


- ffmpeg로 mp3 파일 수정하기

  1) https://enfanthoon.tistory.com/109

  2) https://www.winhelponline.com/blog/split-video-audio-multiple-parts-equal/

  3) https://www.tecmint.com/install-ffmpeg-in-linux/




### 앞으로 할 일...

1. elastsic search 컨테이너 생성해서 연동하기
2. ffmpeg 노래제목에 소괄호 () 들어가면 예외처리해주기
3. 간혹 서버로그에서 한글들어간 파일 찾으면 304에러가 뜸

## Android-side integration The library code that executes the ffmpeg command is packaged from: https://androidlearnersite.wordpress.com/2017/03/17/ffmpeg-video-editor/
Android application pure java code to edit the functions you need, do not need to write C code and NDK, just pay attention to the logic implementation.

Latest version, 3.22_02 Click to download . Extract password: 30hi extended library download: libx264 extract password: 8u1i

English document
The main function:
File processing

Video compression

  -s 控制视频分辨率（如160X320）
  -b 比特率（例如 150K）
  这两个参数直接决定压缩质量，分辨率高，比特率增加，都可以提升画质，但是同时也会增加文件体积。
  
  
  
   //常规的压缩代码
   ffmpeg -y -i /sdcard/videokit/in.mp4 -strict experimental -s 160x120 -r 25 -vcodec mpeg4 -b 150k -ab 48000 -ac 2 -ar 22050 /sdcard/videokit/out.mp4
   //使用h264编码，需要拓展库
   ffmpeg -y -i /sdcard/videokit/in.mp4 -strict experimental -vcodec libx264 -preset ultrafast -crf 24 -acodec aac -ar 44100 -ac 2 -b:a 96k -s 320x240 -aspect 4:3 /sdcard/videokit/out3.mp4
It is recommended to use the array format command to avoid problems with the checksum. E.g:

String[] complexCommand = {"ffmpeg","-y" ,"-i", "/sdcard/video kit/in.mp4","-strict","experimental","-s", "160x120","-r","25", "-vcodec", "mpeg4", "-b", "150k", "-ab","48000", "-ac", "2", "-ar", "22050", "/sdcard/video kit/out.mp4"};

2. Audio compression

 String commandStr = "ffmpeg -y -i /sdcard/vk2/in.wav -ar 44100 -ac 2 -ab 64k -f mp3 /sdcard/videokit/out.mp3";
Audio cropping

 String commandStr ={"ffmpeg","-y","-i","/storage/emulated/0/vk2/in.mp3","-strict","experimental","-acodec","copy","-ss","00:00:00","-t","00:00:03.000","/storage/emulated/0/videokit/out.mp3"};
3. Video rotation

Rotate 90 degrees

 ffmpeg -y -i /sdcard/videokit/in.mp4 -strict experimental -vf transpose=1 -s 160x120 -r 30 -aspect 4:3 -ab 48000 -ac 2 -ar 22050 -b 2097k /sdcard/video_output/out.mp4
4. Video screen partial cropping

 ffmpeg -y -i /sdcard/videokit/short.mp4 -strict experimental -vf crop=100:100:0:0 -s 320x240 -r 15 -aspect 3:4 -ab 12288 -vcodec mpeg4 -b 2097152 -sample_fmt s16 /sdcard/videokit/out.mp4
5. Extract a frame from the video and save it as a picture.

 ffmpeg -y -i /sdcard/videokit/in.mp4 -strict experimental -an -r 1/2 -ss 00:00:00.000 -t 00:00:03 /sdcard/videokit/filename%03d.jpg
6. Extract audio files from the video

 //示例1
 ffmpeg -y -i /sdcard/videokit/in.avi -strict experimental -acodec copy /sdcard/videokit/out.mp3
 //示例2
 ffmpeg -y -i /sdcard/videokit/in.mp4 -strict experimental -vn -ar 44100 -ac 2 -ab 256k -f mp3 /sdcard/videokit/out.mp3
7. Recode the audio in the video

 ffmpeg -y -i /sdcard/in.mp4 -strict experimental -vcodec copy -acodec libmp3lame -ab 64k -ac 2 -b 1200000 -ar 22050 /sdcard/out.mp4
8. Change the resolution of the video, 4:3 or 16:9

 ffmpeg -y -i /sdcard/in.mp4 -strict experimental -vf transpose=3 -s 320x240 -r 15 -aspect 3:4 -ab 12288 -vcodec mpeg4 -b 2097152 -sample_fmt s16 /sdcard/out.mp4
9. Edit a video from the video for a while

 ffmpeg -ss 00:00:01.000 -y -i /sdcard/videokit/in.mp4 -strict experimental -t 00:00:02.000 -s 320x240 -r 15 -vcodec mpeg4 -b 2097152 -ab 48000 -ac 2 -b 2097152 -ar 22050 /sdcard/videokit/out.mp4
10. Audio transcoding

 ffmpeg -y -i /sdcard/videokit/big.wav /sdcard/videokit/small.mp3
11. Video add watermark

 //  test with watermark.png 128x128, add it to /sdcard/videokit/
 String[] complexCommand = {"ffmpeg","-y" ,"-i", "/sdcard/videokit/in.mp4","-strict","experimental", "-vf", "movie=/sdcard/videokit/watermark.png [watermark]; [in][watermark] overlay=main_w-overlay_w-10:10 [out]","-s", "320x240","-r", "30", "-b", "15496k", "-vcodec", "mpeg4","-ab", "48000", "-ac", "2", "-ar", "22050", "/sdcard/videokit/out.mp4"};
Stream and more scene processing

0. Push from Android device to PC device and play it.

// Use this command in the APP (192.168.1.11 is the IP of the computer) ffmpeg -i /sdcard/videokit/2.mpg -strict experimental -f mpegts udp://192.168.1.11:8090 // (192.168.1.14 is IP for Android devices) ffplay -f mpegts -ast 1 -vst 0 -ar 48000 udp://192.168.1.14:8090

1. Get the video stream in its original format from the camera preview.

   Parameters parameters = camera.getParameters();
   imageFormat = parameters.getPreviewFormat();
   if (imageFormat == ImageFormat.NV21) {
                  Camera.Size previewSize = parameters.getPreviewSize();
                  frameWidth = previewSize.width;
                  frameHeight = previewSize.height;
                  Rect rect = new Rect(0, 0, frameWidth, frameHeight);
                  YuvImage img = new YuvImage(data, ImageFormat.NV21, frameWidth, frameHeight, null);
                  try {
                                  outStream.write(data);
                                  outStream.flush();
                  }
   }
Execute encoding command

    "ffmpeg -f rawvideo -pix_fmt nv21 -s 640x480 -r 15 -i " + Environment.getExternalStorageDirectory().getAbsolutePath().toString() + "/yuv.data rtmp://host/stream.flv"
2. Receive a video stream from another device from one device and keep it as a file. [Remember to add network permissions]

   //第一台设备：
   ffmpeg -i /sdcard/one3.mp4 -f mpegts udp://192.168.0.107:8090
   //
   //第二台设备：   
   String[] complexCommand = {"ffmpeg","-y" ,"-i", "udp://192.168.0.108:8090","-strict","experimental","-crf", "30","-preset", "ultrafast", "-acodec", "aac", "-ar", "44100", "-ac", "2", "-b:a", "96k", "-vcodec", "libx264", "-r", "25", "-b:v", "500k", "-f", "flv", "/sdcard/videokit/t.flv"};
3.H264 encoding (need to expand the library)

   //例子1
   ffmpeg -y -i /sdcard/Video/1.MTS -strict experimental -vcodec libx264 -preset ultrafast -crf 24 /sdcard/videokit/out.mp4
   //例子2
   ffmpeg -y -i /sdcard/videokit/m.mkv -strict experimental -vcodec libx264 -preset ultrafast -crf 24 -sn /sdcard/videokit/m2.mkv
4. Add subtitles

   //例子1
   ffmpeg -y -i /sdcard/videokit/m2.mkv -i /sdcard/videokit/in.srt -strict experimental -vcodec libx264 -preset ultrafast -crf 24 -scodec copy /sdcard/videokit/mo.mkv
   //例子2
   ffmpeg -y -i /sdcard/videokit/m2.mkv -i /sdcard/videokit/in.srt -strict experimental -scodec copy /sdcard/videokit/outm3.mkv
5. Convert an mp3 audio file to m4a format

  ffmpeg -i /sdcard/videokit/in.mp3 /sdcard/videokit/out.m4a
6. Render a video and an audio file as an H.264 encoded video (requires additional encoding libraries)

  ffmpeg -y -i /sdcard/videokit/in.mp4 -strict experimental -vcodec libx264 -crf 24 -acodec aac /sdcard/videokit/out.mkv
7. Add a retro filter

commandStr = "ffmpeg -y -i /sdcard/videokit/in.mp4 -strict experimental -vf curves=vintage -s 640x480 -r 30 -aspect 4:3 -ab 48000 -ac 2 -ar 22050 -b 2097k -vcodec mpeg4 /sdcard/videokit/curve.mp4";
8. Black and white filter

commandStr = "ffmpeg -y -i /sdcard/videokit/in.mp4 -strict experimental -vf hue=s=0 -vcodec mpeg4 -b 2097152 -s 320x240 -r 30 /sdcard/videokit/out.mp4";
9. Color channel filter, similar to the color channel in PS

String[] complexCommand = {"ffmpeg","-y" ,"-i", "/sdcard/videokit/sample.mp4","-strict", "experimental", "-filter_complex",
"[0:v]colorchannelmixer=.393:.769:.189:0:.349:.686:.168:0:.272:.534:.131[colorchannelmixed];[colorchannelmixed]eq=1.0:0:1.3:2.4:1.0:1.0:1.0:1.0[color_effect]",
"-map", "[color_effect]","-map", "0:a", "-vcodec", "mpeg4","-b", "15496k", "-ab", "48000", "-ac", "2", "-ar", "22050","/sdcard/videokit/out.mp4"};
10. Add a filter made with the curve of PS, that is, you can make your own filter in PS, select Image > Adjustment > Curve, call up the curve window, or press the shortcut key Ctrl+N to adjust the curve window will adjust The good effect is exported as an acv file, and its path is added to the ffmpeg command.

String[] complexCommand={"ffmpeg","-y","-i","/storage/emulated/0/vk2/in.mp4","-strict","experimental","-vf","curves=psfile=/storage/emulated/0/videokit/sepia.acv","-b","2097k","-vcodec","mpeg4","-ab","48000","-ac","2","-ar","22050","/storage/emulated/0/videokit/out.mp4"}﻿
11. Add fade in and fade out transitions to the video

String[] complexCommand = {"ffmpeg","-y" ,"-i", "/sdcard/videokit/in.m4v","-acodec", "copy", "-vf", "fade=t=in:st=0:d=5, fade=t=out:st=20:d=5", "/sdcard/videokit/out.mp4"};
12. splicing and outputting two video files of the same size

   String[] complexCommand = {"ffmpeg","-y","-i", "/sdcard/videokit/in1.mp4", "-i", "/sdcard/videokit/in2.mp4", "-strict","experimental", "-filter_complex", "[0:0] [0:1] [1:0] [1:1] concat=n=2:v=1:a=1", "/sdcard/videokit/out.mp4"};
  //
  //拼接不同编码，不同尺寸，不同帧率，不同纵横比的视频
  
   String[] complexCommand = {"ffmpeg","-y","-i","/storage/emulated/0/videokit/sample.mp4",
      "-i","/storage/emulated/0/videokit/in.mp4","-strict","experimental",
      "-filter_complex",
      "[0:v]scale=640x480,setsar=1:1[v0];[1:v]scale=640x480,setsar=1:1[v1];[v0][0:a][v1][1:a] concat=n=2:v=1:a=1",
      "-ab","48000","-ac","2","-ar","22050","-s","640x480","-r","30","-vcodec","mpeg4","-b","2097k","/storage/emulated/0/vk2_out/out.mp4"}
13. Render a set of image sequences into a video file, the images must be ordered in a regular format. This command strictly requires the image sequence size to be uniform, corresponding to the resolution of the output video.

 commandStr = "ffmpeg -y -r 1/5 -i /sdcard/videokit/pic00%d.jpg /sdcard/videokit/out.mp4";
 //添加音频，png也可以，这里的图片尺寸应该为320x240
 ffmpeg -y -r 1 -i /sdcard/videokit/pic00%d.jpg -i /sdcard/videokit/in.mp3 -strict experimental -ar 44100 -ac 2 -ab 256k -b 2097152 -ar 22050 -vcodec mpeg4 -b 2097152 -s 320x240 /sdcard/videokit/out.mp4
14.Advanced filtering (this function is literally I don't understand, and I will explain it after I verify it myself)

  String[] complexCommand = {"ffmpeg","-y" ,"-i", "/sdcard/videokit/in.mp4","-strict","experimental", "-vf", "crop=iw/2:ih:0:0,split[tmp],pad=2*iw[left]; [tmp]hflip[right]; [left][right] overlay=W/2", "-vb", "20M", "-r", "23.956", "/sdcard/videokit/out.mp4"};
15. Speed ​​up the playback of video and audio files, which is time compression.

 String[] complexCommand = {"ffmpeg","-y" ,"-i", "/sdcard/videokit/in.mp4","-strict","experimental", "-filter_complex", "[0:v]setpts=0.5*PTS[v];[0:a]atempo=2.0[a]","-map","[v]","-map","[a]", "-b", "2097k","-r","60", "-vcodec", "mpeg4", "/sdcard/videokit/out.mp4"};
16. Display two videos side by side

   String[] complexCommand = {"ffmpeg","-y" ,"-i", "/sdcard/Movies/sample.mp4","-i", "/sdcard/Movies/sample2.mp4", "-strict","experimental",
                      "-filter_complex", 
                      "[0:v:0]pad=iw*2:ih[bg];" + 
                      "[bg][1:v:0]overlay=w",
                      "-s", "320x240","-r", "30", "-b", "15496k", "-vcodec", "mpeg4","-ab", "48000", "-ac", "2", "-ar", "22050",
                      "/sdcard/videokit/out.mp4"};  
17. Add time watermark

  String[] complexCommand = {"ffmpeg","-y" ,"-i", "/sdcard/Movies/sample.mp4","-strict","experimental",
                  "-vf", 
                  "movie=/sdcard/videokit/watermark002.png [watermark];" + 
                  "[in][watermark] overlay=main_w-overlay_w-10:10 [out_overlay];" +
                  "[out_overlay]curves=vintage[out]",  
                  "-s", "320x240","-r", "30", "-b", "15496k", "-vcodec", "mpeg4","-ab", "48000", "-ac", "2", "-ar", "22050",
                  "/sdcard/videokit/out_water_vinta.mp4"};
18. Watermarking, audio replacement

String commandStr = "ffmpeg -y -loop 1 -i /sdcard/videokit/pic001.jpg -i /sdcard/videokit/in.mp3 -strict experimental -s 1270x720 -r 25 -aspect 16:9 -vcodec mpeg4 -vcodec mpeg4 -ab 48000 -ac 2 -b 2097152 -ar 22050 -shortest /sdcard/videokit/out2.mp4";
19. Replace the video track

  String[] complexCommand = {"ffmpeg","-y" ,"-i", "/sdcard/videokit/sample.mp4","-i", "/sdcard/videokit/in.mp3", "-strict","experimental",
                  "-map", "0:v", "-map", "1:a",
                  "-s", "320x240","-r", "30", "-b", "15496k", "-vcodec", "mpeg4","-ab", "48000", "-ac", "2", "-ar", "22050","-shortest","/sdcard/videokit/out.mp4"};
20. GIF image to video, or video to GIF image

  //Compress animated gif:
  fmpeg -f gif -i /sdcard/videokit/pic1.gif -strict experimental -r 10 /sdcard/videokit/pic1_1.gif
  
  //Convert mp4 to animated gif:
  ffmpeg -y -i /sdcard/videokit/in.mp4 -strict experimental -r 20 /sdcard/videokit/out.gif
  
  //Convert animated gif to mp4:
  ffmpeg -y -f gif -i /sdcard/videokit/infile.gif /sdcard/videokit/outfile.mp4
License
Copyright 2017 Johnny Shieh Open Project

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

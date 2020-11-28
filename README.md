# HdmiPi-Streaming
Streaming using a cheap HDMI capture card and a raspberry Pi to an RTMP Receiver. 

 Raspberry Pi OC settings (/boot/config.txt): 
```
arm_freq=2000
enable_tvout
#hdmi_enable_4kp60=1
core_freq=550
#gpu_freq=800
h264_freq=850
over_voltage=6
disable_splash=1
force_turbo=1 #Voids Warranty! (uncomment to avoid CPU scaling down to 600Mhz)
boot_delay=1 #helps to avoid sdcard corruption when force_turbo is enabled.
#sdram_freq=500 #uncomment to test. Works only with some boards.
```

Code for 720P resolution and Stream to twitch

```
v4l2-ctl --set-fmt-video=width=1280,height=720 && ffmpeg -f v4l2 -thread_queue_size 384 -input_format mjpeg -framerate 30 -i /dev/video0 -f alsa -thread_queue_size 4096 -i plughw:1,0 -acodec pcm_s16le -ac 1 -ar 96000 -copytb 1 -use_wallclock_as_timestamps 1  -c:a aac  -b:a 128k -ar 44100 -b:v 8M -c:v h264_omx -f flv rtmp://live.twitch.tv/app/XXXXXXXXXXXXXXXXXXXXXXX 
```

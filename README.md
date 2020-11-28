# HdmiPi-Streaming
Streaming using a cheap HDMI capture card and a Raspberry Pi 4 to an RTMP Receiver. 



![Dodgy Banner](/images/IMG_20201128_170914.jpg)



After maybe a month of pulling my hair out, I finally got cheap USB HDMI capture cards work well with the Hardware encoder on the Pi and stream it to Twitch. For best performance your Raspberry Pi needs to be overclocked and have adequate cooling. These are the settings that worked for me. 

Here's what you need: 

- OC'ed Raspberry Pi 4 With decent PSU and cooling  (I use a passive metal case, thats sufficient)
- USB HDMI Capture card (more details below)
- (Optional) HDMI Splitter 
- (Optional) Ethernet Connection to the Pi (5Ghz results in occasional dropped frames)

HDMI Splitter isnt mandatory, but there is a 10s delay between the local stream and whats being broadcast to twitch, so I split my HDMI signal from my Video Game consoles to my Screen and to the PI Capture card. 


# Just tell me the code dammit: 

If you're in a rush, here's the code to Stream to twitch. You'll need the RTMP url and you might need to modify the command depending on how the USB device was detected on your system: 

```
v4l2-ctl --set-fmt-video=width=1280,height=720 && ffmpeg -f v4l2 -thread_queue_size 384 -input_format mjpeg -framerate 30 -i /dev/video0 -f alsa -thread_queue_size 4096 -i plughw:1,0 -acodec pcm_s16le -ac 1 -ar 96000 -copytb 1 -use_wallclock_as_timestamps 1  -c:a aac  -b:a 128k -ar 44100 -b:v 4M -c:v h264_omx -f flv rtmp://live.twitch.tv/app/XXXXXXXXXXXXXXXXXXXXXXX 
```


# Find the right Capture card: 
Look up an HDMI Capture card with UVC (Usb Video Class) and UAC (Usb Audio Class). Typing in "USB Hdmi Capture UVC" should return appropriate results. Make sure that the description lists UVC and UAC. For this use case, there isnt much difference between USB 2 and USB 3, the bottleneck is the Raspberry Pi CPU/GPU. 

The description should look similar to below: 

______

![Ebay Description](/images/Screenshot%20from%202020-11-28%2015-52-31.png)




# Raspberry Pi Setup: 


# Raspberry Pi OC settings (/boot/config.txt): 
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

# Software Explaination: 

Set up your Pi, making sure it is up to date, and ensuring that `ffmpeg` and `v4l-utils` is installed. You can do so with the command `sudo apt install ffmpeg v4l-utils`.  Plug in the device, wait a moment and type into the terminal (local or SSH), `v4l2-ctl --list-devices`.  You should get something similar to this: 

```
pi@capturepi:~ $ v4l2-ctl --list-devices
bcm2835-codec-decode (platform:bcm2835-codec):
	/dev/video10
	/dev/video11
	/dev/video12

bcm2835-isp (platform:bcm2835-isp):
	/dev/video13
	/dev/video14
	/dev/video15
	/dev/video16

UVC Camera (534d:2109): USB Vid (usb-0000:01:00.0-1.4):
	/dev/video0
	/dev/video1

```

The last listing is what we're interested in. Make a quick note of where the device is (in my case **/dev/video0**) The next step will indicate how to decide between video0 and video1. Type in `v4l2-ctl -d /dev/video0 --list-formats-ex`

```
pi@capturepi:~ $ v4l2-ctl -d /dev/video0 --list-formats-ex
ioctl: VIDIOC_ENUM_FMT
	Type: Video Capture

	[0]: 'MJPG' (Motion-JPEG, compressed)
		Size: Discrete 1920x1080
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.040s (25.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
			Interval: Discrete 0.200s (5.000 fps)
		Size: Discrete 1600x1200
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.040s (25.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
			Interval: Discrete 0.200s (5.000 fps)
		Size: Discrete 1360x768
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.040s (25.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
			Interval: Discrete 0.200s (5.000 fps)
		Size: Discrete 1280x1024
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.040s (25.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
			Interval: Discrete 0.200s (5.000 fps)
		Size: Discrete 1280x960
			Interval: Discrete 0.020s (50.000 fps)
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
			Interval: Discrete 0.200s (5.000 fps)
		Size: Discrete 1280x720
			Interval: Discrete 0.017s (60.000 fps)
			Interval: Discrete 0.020s (50.000 fps)
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
		Size: Discrete 1024x768
			Interval: Discrete 0.017s (60.000 fps)
			Interval: Discrete 0.020s (50.000 fps)
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
		Size: Discrete 800x600
			Interval: Discrete 0.017s (60.000 fps)
			Interval: Discrete 0.020s (50.000 fps)
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
		Size: Discrete 720x576
			Interval: Discrete 0.017s (60.000 fps)
			Interval: Discrete 0.020s (50.000 fps)
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
		Size: Discrete 720x480
			Interval: Discrete 0.017s (60.000 fps)
			Interval: Discrete 0.020s (50.000 fps)
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
		Size: Discrete 640x480
			Interval: Discrete 0.017s (60.000 fps)
			Interval: Discrete 0.020s (50.000 fps)
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
	[1]: 'YUYV' (YUYV 4:2:2)
		Size: Discrete 1920x1080
			Interval: Discrete 0.200s (5.000 fps)
		Size: Discrete 1600x1200
			Interval: Discrete 0.200s (5.000 fps)
		Size: Discrete 1360x768
			Interval: Discrete 0.125s (8.000 fps)
		Size: Discrete 1280x1024
			Interval: Discrete 0.125s (8.000 fps)
		Size: Discrete 1280x960
			Interval: Discrete 0.125s (8.000 fps)
		Size: Discrete 1280x720
			Interval: Discrete 0.100s (10.000 fps)
		Size: Discrete 1024x768
			Interval: Discrete 0.100s (10.000 fps)
		Size: Discrete 800x600
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
			Interval: Discrete 0.200s (5.000 fps)
		Size: Discrete 720x576
			Interval: Discrete 0.040s (25.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
			Interval: Discrete 0.200s (5.000 fps)
		Size: Discrete 720x480
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
			Interval: Discrete 0.200s (5.000 fps)
		Size: Discrete 640x480
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.050s (20.000 fps)
			Interval: Discrete 0.100s (10.000 fps)
			Interval: Discrete 0.200s (5.000 fps)
pi@capturepi:~ $ v4l2-ctl -d /dev/video1 --list-formats-ex
ioctl: VIDIOC_ENUM_FMT
	Type: Video Capture

``` 

In my Instance **video0** lists a bunch of formats whilst **video1** doesnt. You could try to open **video0** in VLC or ffplay at this moment but you'll notice that the frame rate is very low. Thats because it defaults to the "YUYV" stream instead of "MJPG" one. We will need to use the "MJPG" stream in this instance. Take note of **/dev/video0** or whatever it may be in your case. 


Now, we need to sort out audio. List the audio devices using the command "cat /proc/asound/devices" : 

```
pi@capturepi:~ $ cat /proc/asound/devices
  0: [ 0]   : control
 16: [ 0- 0]: digital audio playback
 32: [ 1]   : control
 33:        : timer
 56: [ 1- 0]: digital audio capture
```

The last one looks like what we're after. Take note of the **"[ 1- 0]"** or what it might be in your setup. In the script, it will be formatted as **1,0**. If it was listed as **"[ 2- 0]"**, it would be formatted as **2,0**. 


Now, we have the video stream **/dev/video0** and the audio stream **1,0**. However, the PI (As of me writing this) cannot convert 1080p footage at anything above 8-10fps. I suspect that this is due to Software conversion happening for the colour values in the MJPG stream. You'll need to reduce the resolution of the capture device. This is temporary, and will reset on device reboot, or if you change it manually. To change the resolution to 720P, you'll need to invoke the following command: 

```v4l2-ctl --set-fmt-video=width=1280,height=720```

# ffmpeg Command explaination

The command to stream the video **/dev/video0** and audio **1,0** to ffmpeg, convert it using the Raspberry Pi inbuilt HW encoder, then stream it to twitch is as followss: 

```
ffmpeg -f v4l2 -thread_queue_size 384 -input_format mjpeg -framerate 30 -i /dev/video0 -f alsa -thread_queue_size 4096 -i plughw:1,0 -acodec pcm_s16le -ac 1 -ar 96000 -copytb 1 -use_wallclock_as_timestamps 1  -c:a aac  -b:a 128k -ar 44100 -b:v 4M -c:v h264_omx -f flv rtmp://live.twitch.tv/app/XXXXXXXXXXXXXXXXXXXXXXX 
```

Lets break it down: 

 - ffmpeg : The program we are using for conversion
 - "-f v4l2 -thread_queue_size 384 -input_format mjpeg -framerate 30 -i /dev/video0" : Use the 30fps MJPEG stream at **/dev/video0** and give it some buffer. 
 - "-f alsa -thread_queue_size 4096 -i plughw:1,0 -acodec pcm_s16le -ac 1 -ar 96000 -copytb 1 -use_wallclock_as_timestamps 1" : Use the audio from **1,0**, mono audio (Capture card limitation), codec pcm_161e, with a sample of 96kHz. 
 - "-c:a aac  -b:a 128k -ar 44100 -b:v 8M -c:v h264_omx" : Convert Audio to Birtate 128K, sample rate of 44.1kHz. Convert Video using Hardware H264 encoder, bitrate of 4Mb/s. 
 
 - "-f flv rtmp://live.twitch.tv/app/XXXXXXXXXXXXXXXXXXXXXXX " Mux the convereted Video and Audio into an FLV and send it to your Stream key. 
 
 
 You should see your stream show up on your platform of choice, however there will be a delay. On twitch, I get a roughly 10s delay between gameplay and whats on twitch. 
 
 
# Benefits : 

- Uses cheap HDMI capture cards over a USB interface
- Raspberry Pi is relatively inexpensive and has little power comsumption compared to dedicated capture PC
- If capturing a PC display, less conversion load on host PC. 
- Minimal software prerequesites, Raspberry Pi OS Lite + `ffmpeg` + `v4l-utils` is basically all you need. 
 

# Caveats

- Audio is only at mono (I think this is a limitation of the capture card themselves)
- Resolution is capped at 720P for 25+ FPS stream, due to software conversion of the MJPEG colourspace (I think, I might be wrong)
- No GUI (at present)
  - Some proficency at command line needed. 
  
  
 ## Acknowledgements  
  
  My knowledge in linux is bare minimal, I take no credit whatsoever apart from documenting what worked in my case. 
  
  [Arch Wiki, Ofcourse](https://wiki.archlinux.org/index.php/Streaming_to_twitch.tv) 
  
  [FFmpeg Documentation](https://trac.ffmpeg.org/wiki/EncodingForStreamingSites) 
  
  [ffmpeg IRC Channel](https://ffmpeg.org/contact.html) (the person who helped me didnt wish to be credited)
   
  https://github.com/pikvm/ustreamer (I used this for LOTS of debugging)

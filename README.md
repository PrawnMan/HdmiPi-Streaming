# HdmiPi-Streaming
Streaming using a cheap HDMI capture card and a Raspberry Pi 4 to an RTMP Receiver. 

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
v4l2-ctl --set-fmt-video=width=1280,height=720 && ffmpeg -f v4l2 -thread_queue_size 384 -input_format mjpeg -framerate 30 -i /dev/video0 -f alsa -thread_queue_size 4096 -i plughw:1,0 -acodec pcm_s16le -ac 1 -ar 96000 -copytb 1 -use_wallclock_as_timestamps 1  -c:a aac  -b:a 128k -ar 44100 -b:v 8M -c:v h264_omx -f flv rtmp://live.twitch.tv/app/XXXXXXXXXXXXXXXXXXXXXXX 
```


# Find the right Capture card: 
Look up an HDMI Capture card with UVC (Usb Video Class) and UAC (Usb Audio Class). Typing in "USB Hdmi Capture UVC" should return appropriate results. Make sure that the description lists UVC and UAC. For this use case, there isnt much difference between USB 2 and USB 3, the bottleneck is the Raspberry Pi CPU/GPU. 



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

Set up your Pi, making sure it is up to date, and ensuring that **ffmpeg** is installed. Plug in the device, wait a moment and type in **v4l2-ctl --list-devices**.  You should get something similar to this: 

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

The last listing is what we're interested in. Make a quick note of where the device is (in my case **/dev/video0**) The next step will indicate how to decide between video0 and video1

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


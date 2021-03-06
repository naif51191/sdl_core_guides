## Initial Configuration

### SDL Core Setup

Before continuing, follow the [Install and Run Guide](../../getting-started/install-and-run/) for SDL Core if you have not already done so.

### HMI Setup

The Generic HMI does not currently support streaming.

If using the [SDL HMI](https://github.com/smartdevicelink/sdl_hmi), you may need to make the following modifications when using socket streaming.

#### VIDEO
In order to stream video, comment out the following line in `app/model/sdl/Abstract/Model.js`:
```
//  SDL.SDLModel.playVideo(appID);
```

#### AUDIO
In order to stream audio, comment out the following lines in `app/model/sdl/Abstract/Model.js`:
```
//  SDL.StreamAudio.play(
//      SDL.SDLController.getApplicationModel(appID).navigationAudioStream
//  );
```

### GSTREAMER Setup

It is easier to determine which gstreamer video sink will work in your environment by testing with a static file. This can be done by downloading [this file](https://support.apple.com/library/APPLE/APPLECARE_ALLGEOS/HT1425/sample_iPod.m4v.zip) and trying the following command.

Common values for sink:

* ximagesink (x visual environment sink)
* xvimagesink (xv visual environment sink)
* cacasink (ascii art sink)

```
gst-launch-1.0 filesrc location=/path/to/h264/file ! decodebin ! videoconvert ! <sink> sync=false
```

If you're streaming video over TCP, you can point gstreamer directly to your phone's stream using
```
gst-launch-1.0 tcpclientsrc host=<Device IP Address> port=3000 ! decodebin ! videoconvert ! <sink> sync=false
```

## Pipe Streaming

### Configuration (smartDeviceLink.ini)
In the Core build folder, open `bin/smartDeviceLink.ini` and ensure the following values are set:
```
VideoStreamConsumer = pipe
AudioStreamConsumer = pipe
```

### GStreamer Commands

After you start SDL Core, cd into the bin/storage directory and there should be a file named "video_stream_pipe". Use the gst-launch command that worked for your environment and set file source to the video_stream_pipe file. You should see “setting pipeline to PAUSED” and “Pipeline is PREROLLING”.

#### Raw H.264 Video
```
gst-launch-1.0 filesrc location=$SDL_BUILD_PATH/bin/storage/video_stream_pipe ! decodebin ! videoconvert ! xvimagesink sync=false
```

#### H.264 Video over RTP
```
gst-launch-1.0 filesrc location=$SDL_BUILD_PATH/bin/storage/video_stream_pipe ! "application/x-rtp-stream" ! rtpstreamdepay ! "application/x-rtp,media=(string)video,clock-rate=90000,encoding-name=(string)H264" ! rtph264depay ! "video/x-h264, stream-format=(string)avc, alignment=(string)au" ! avdec_h264 ! videoconvert ! ximagesink sync=false
```

#### RAW PCM Audio

```
gst-launch-1.0 filesrc location=$SDL_BUILD_PATH/bin/storage/audio_stream_pipe ! audio/x-raw,format=S16LE,rate=16000,channels=1 ! pulsesink
```

## Socket Streaming

### Configuration (smartDeviceLink.ini)
In the Core build folder, open `bin/smartDeviceLink.ini` and ensure the following values are set:
```
; Socket ports for video and audio streaming
VideoStreamingPort = 5050
AudioStreamingPort = 5080
...
VideoStreamConsumer = socket
AudioStreamConsumer = socket
```

### GStreamer Commands

#### Raw H.264 Video
```
gst-launch-1.0 souphttpsrc location=http://127.0.0.1:5050 ! decodebin ! videoconvert ! xvimagesink sync=false
```

#### H.264 Video over RTP
```
gst-launch-1.0 souphttpsrc location=http://127.0.0.1:5050 ! "application/x-rtp-stream" ! rtpstreamdepay ! "application/x-rtp,media=(string)video,clock-rate=90000,encoding-name=(string)H264" ! rtph264depay ! "video/x-h264, stream-format=(string)avc, alignment=(string)au" ! avdec_h264 ! videoconvert ! ximagesink sync=false
```

#### RAW PCM Audio

```
gst-launch-1.0 souphttpsrc location=http://127.0.0.1:5080 ! audio/x-raw,format=S16LE,rate=16000,channels=1 ! pulsesink
```

## Video Streaming States

This section describes how Core manages the streaming states of mobile applications. Only one application may stream video at a time, but audio applications may stream while in the LIMITED state with other applications.

When an app is moved to HMI level `FULL`:
* All non-streaming applications go to HMI level `BACKGROUND`
* All apps with the same App HMI Type go to `BACKGROUND`
* Streaming apps with a different App HMI Type that were in `FULL` go to `LIMITED`

When an app is moved to HMI level `LIMITED`:
* All non-streaming applications keep their HMI level
* All applications with a different App HMI Type keep their HMI level
* Applications with the same App HMI Type go to `BACKGROUND`

### Additional Resources

!!! NOTE
Livio provides an [example video streaming android application](https://github.com/livio/sdl_video_streaming_android_sample).
!!!

[iOS Video Streaming Guide](https://smartdevicelink.com/en/guides/iOS/video-streaming-for-navigation-apps/video-streaming/)

[Android Video Streaming Guide](https://smartdevicelink.com/en/guides/android/video-streaming-for-navigation-apps/video-streaming/)

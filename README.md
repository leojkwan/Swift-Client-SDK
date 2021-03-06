Ziggeo Swift SDK 1.0
=============

Ziggeo API (http://ziggeo.com) allows you to integrate video recording and playback with only two lines of code in your site, service or app. This is the iOS SDK repository. 

## Setup
- Create iOS App
- Add ZiggeoSwiftFramework.framework
- Add bridging header to swift project
  ```
  #import <ZiggeoSwiftFramework/ZiggeoSwiftFramework.h>
  ```
- Make sure ZiggeoSwiftFramework.framework is added to Embedded Binaries and Linked Framework sections in your app target settings

## Building/packaging app
- You need to remove the current framework from your app
- Grab framework you need from Swift-Client-SDK/Output/Release-iphoneos/ directory
  1. `Release-iphoneos` - used for testing on phones / ipads, and to push to store.
  2. `Release-iphonesimulator` - used to build simulator app, nothing else (can not be pushed to store nor used in real device)
  3. `Release-iphoneuniversal` - can be used for both simulator and real device, however can not be pushed to store when this framework is used
- Add same framework into "linked frameworks" and "embedded binaries" at the project build settings
- Clean and rebuild the application

# Basic usage
## Initialize Application

```
import ZiggeoSwiftFramework

var m_ziggeo: Ziggeo! = nil;
m_ziggeo = Ziggeo(token: "YOUR_APP_TOKEN_HERE");
```

## Player

### Initialization
```
func createPlayer()->AVPlayer {
   let player = AVPlayer(URL: NSURL(string: m_ziggeo.videos.GetURLForVideo("YOUR_VIDEO_TOKEN_HERE"))!);
   return player;
}
```

### Fullscreen playback
```
@IBAction func playFullScreen(sender: AnyObject) {
    let playerController: AVPlayerViewController = AVPlayerViewController();
    playerController.player = createPlayer();
    self.presentViewController(playerController, animated: true, completion: nil);
    playerController.player?.play();
}
```

### Embedded playback
```
@IBAction func playEmbedded(sender: AnyObject) {
    //cleanup
    if m_embeddedPlayer != nil {
        m_embeddedPlayer.pause();
        m_embeddedPlayerLayer.removeFromSuperlayer();
        m_embeddedPlayer = nil;
        m_embeddedPlayerLayer = nil;
    }
    m_embeddedPlayer = createPlayer();
    m_embeddedPlayerLayer = AVPlayerLayer(player: m_embeddedPlayer);
    m_embeddedPlayerLayer.frame = videoViewPlaceholder.frame;
    videoViewPlaceholder.layer.addSublayer(m_embeddedPlayerLayer);
    m_embeddedPlayer.play();
}
```

## Recorder

```
@IBAction func record(sender: AnyObject) {
    let recorder = ZiggeoRecorder(application: m_ziggeo);
    self.presentViewController(recorder, animated: true, completion: nil);
}
```

### Enable cover selector dialog
```
recorder.coverSelectorEnabled = true;
```

### Disable camera flip button

```
recorder.cameraFlipButtonVisible = false;
```

### Set active camera device

```
recorder.cameraDevice = UIImagePickerControllerCameraDevice.Rear;
```

# Advanced SDK usage
# Ziggeo API access
You can use the SDK to access Ziggeo Server API methods in the async manner. The SDK provides next functionality:
- Create/remove/index videos
- Custom Ziggeo Embedded Server API requests 

All the API methods are working asynchronously and never blocking the calling thread. You may optionally use custom callbacks (completion blocks) to receive the results.

## Videos API

### Get video accessor object
```
let videos = m_ziggeo.videos;
```

### Get all videos
```
videos.Index(nil) { (jsonArray, error) in
    NSLog("index error: \(error), response: \(jsonArray)");
};
```

### Create new video
#### Basic
```
videos.CreateVideo(nil, file: fileToUpload.path!, cover: nil, callback: nil, progress: nil);
```

#### Advanced
You can add your custom completion/progress callbacks here to make the SDK inform your app about uploading progress and response. Cover image is optional and could be nil, making Ziggeo platform to generate default video cover
```
videos.CreateVideo(nil, file: fileToUpload.path!, cover: UIImage?, callback: { (jsonObject, response, error) in
    //update ui
}, progress: { (totalBytesSent, totalBytesExpectedToSend) in
    //update progress ui
})
```

### Delete video
```
videos.DeleteVideo("YOUR_VIDEO_TOKEN", callback: { (responseData, response, error) in
    //update ui
})
```

### Get video URL to be used in your own custom player
```
let url = videos.GetURLForVideo("YOUR_VIDEO_TOKEN_HERE"))
```

### Get remote video thumb asynchronously
Remote video thumbs are cached on client side, so you can call the method as frequently as you wish without the performance or network load impact
```
videos.GetImageForVideo("YOUR_VIDEO_TOKEN_HERE", callback: { (image, response, error) in
    //update UI with the received image
})
```

### Generate local video thumb asynchronously
Local video thumbs are cached on client side, so you can call the method as frequently as you wish without the performance impact
```
self.application.videos.GetImageForLocalVideo("LOCAL_VIDEO_PATH", callback: { (image, error) in
    //update UI with the received image
})
```

## Custom Ziggeo API requests
The SDK provides an opportunity to make custom requests to Ziggeo Embedded Server API. You can make POST/GET/custom_method requests and receive RAW data, json-dictionary or string as the result.

### Get API accessor object
```
let connection = self.application.connect;
```

### Make POST request and parse JSON response
```
connection.PostJsonWithPath(path, data: NSDictionary?, callback: { (jsonObject, response, error) in
//jsonObject contains parsed json response received from Ziggeo API Server
})
```

### Make POST request and get RAW data response
```
connection.PostWithPath(path: String, data: NSDictionary?, callback: { (data, response, error) in
//data contains RAW data received from Ziggeo API Server
})
```

### Make GET request and get string response
```
connection.GetStringWithPath(path: String, data: NSDictionary?, callback: { (string, response, error) in
    //the string contains stringified response received from Ziggeo API Server
})
```
There are bunch of other methods for different http methods and response types.

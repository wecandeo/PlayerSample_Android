# Sample Player

## **설정**

### 사전에 조회할 값 설명
### Player 공통
WECANDEO 통계를 사용하기 위해서는 활성화 된 WECANDEO 계정이 있어야 합니다.
먼저 [WECANDEO 홈페이지](https://www.wecandeo.com/) 에서 계정을 생성하고 플랜(Trial, Standard, Enterprise)에 [가입](https://www.wecandeo.com/pricing/videopack/edition/)하여 계정을 활성화 합니다.
활성화 된 계정에 이용중인 상품이 *VideoPack*인 경우 VOD Player를, *LivePack*인 경우 Live Player 를 사용할 수 있습니다.

[WECANDEO API](https://support.wecandeo.com/#apis)를 사용하여 필요한 값을 조회합니다.

※ [WECANDEO API](https://support.wecandeo.com/#apis)를 사용하기 위해 필요한 `API Key`는 활성화 된 계정의 CMS에서 확인 가능합니다.[CMS > 계정관리 > 개발자 API]

- VOD Player
  - videoKey : [동영상 배포 코드 조회 API](https://support.wecandeo.com/reference/videopack-api-video-data-retrieve-video-pub-code)를 호출하면 `videoKey` 를 확인할 수 있습니다.
  - DRM 재생을 위한 값
    - gid : [ CMS > 부가서비스 > Wecandeo DRM ] 메뉴에서 `gid`를 확인할 수 있습니다.
    - secretKey : [ CMS > 부가서비스 > Wecandeo DRM ] 메뉴에서 `secretKey`를 확인할 수 있습니다.
    - packageId : [배포 패키지 목록 조회 API](https://support.wecandeo.com/reference/videopack-api-package-package-list)를 호출하면 `packageId`를 확인할 수 있습니다.
    - videoId : [동영상 목록 - 배포 패키지별 조회 API](https://support.wecandeo.com/reference/videopack-api-video-data-retrieve-video-list-package)를 호출하면 `videoId`를 확인할 수 있습니다.

※ DRM 기능을 사용하기 위해서는 사용 가능한 플랜(Enterprise)에 가입되어 있어야 하며, 관리자를 통해 해당 기능이 활성화 되어 있어야 합니다.

- Live Player
  - liveKey : [CMS > 라이브 채널 > 채널 리스트 > 채널 선택 > 배포 코드] 메뉴에서 `liveKey`를 확인할 수 있습니다.

### 1. 라이브러리 추가
    implementation 'com.google.android.exoplayer:exoplayer:2.12.1'
    implementation 'com.android.volley:volley:1.1.1'
    implementation 'com.google.code.gson:gson:2.8.6'
    implementation files('libs/WecandeoPlaySdk.jar')

### - libs 폴더에 WecandeoPlaySdk.jar 파일 추가 

### 2. AndroidManifest.xml 에 permissions 추가
```
<uses-permission android:name="android.permission.INTERNET" />
```
### 3. AndroidManifest.xml application 태그 안에 usesCleartextTraffic 설정
```
<application
android:usesCleartextTraffic="true">
```

### 4. - UrlInfo, RequestSingleton Class 는 수정하지 않고 사용

## VOD
### Player 구성 방법
- 공통
  - 해당 Activity 에 Player.EventListener, SdkInterface.onSdkListener 인터페이스를 구현해야 합니다.
    - ```Activity implements Player.EventListener, SdkInterface.onSdkListener```
  - onStart(), onResume(), onPause(), onStop() 메서드를 각 메서드에 맞게 wecandeoSdk 호출
  ```
  @Override
  public void onStart(){
      super.onStart();
      if(wecandeoSdk != null)
        wecandeoSdk.onStart();
  }
  ``` 
  - 이벤트
    - 재생 : wecandeoSdk.play()
    - 재시작 : wecandeoSdk.retry()
    - 정지 : wecandeoSdk.stop()
    - 일시 정지 : wecandeoSdk.pause()
    - 앞으로 넘기기 : wecandeoSdk.fastForward()
    - 뒤로 돌아가기 : wecandeoSdk.rewind()
    - 종횡비 설정 : wecandeoSdk.setResizeMode(int position)
      - position 값
      - 0 (RESIZE_MODE_FIT) : 원하는 종횡비를 얻기 위해 너비 또는 높이를 줄입니다.
      - 1 (RESIZE_MODE_FIXED_WIDTH) : 너비는 고정되고 높이는 원하는 종횡비를 얻기 위해 증가 또는 감소합니다.
      - 2 (RESIZE_MODE_FIXED_HEIGHT) : 높이는 고정되고 너비는 종횡비를 얻기 위해 늘리거나 줄입니다.
      - 3 (RESIZE_MODE_FILL) : 지정된 종횡비가 무시 됩니다.
      - 4 (RESIZE_MODE_ZOOM) : 원하는 종횡비를 얻기 위해 너비 또는 높이를 늘립니다.
- DRM
  - 발급된 videoId, videoKey, gId, scretKey, packageId 를 통해 Player 구성
 ```
  wecandeoSdk = new WecandeoSdk(this);
  wecandeoSdk.setSdkListener(this);
  wecandeoSdk.addPlayerListener(this);
  wecandeoVideo = new WecandeoVideo();
  wecandeoVideo.setDrm(true);
  wecandeoVideo.setVideoKey("videoKey");
  wecandeoVideo.setgId("gid");
  wecandeoVideo.setPackageId("packageId");
  wecandeoVideo.setVideoId("videoId");
  wecandeoVideo.setSecretKey("secretKey");
  wecandeoSdk.setWecandeoVideo(wecandeoVideo);
  wecandeoSdk.setPlayerView(playerView);
  wecandeoSdk.setUseController(false);
 ```
- Non DRM
  - 발급된 videoKey 로 영상 상세정보 조회를 하여 나온 videoUrl 값을 이용하여 Player 구성
```
  wecandeoSdk = new WecandeoSdk(this);
  wecandeoSdk.setSdkListener(this);
  wecandeoSdk.addPlayerListener(this);
  wecandeoVideo = new WecandeoVideo();
  wecandeoVideo.setDrm(false);
  wecandeoVideo.setVideoKey("videoUrl");
  wecandeoSdk.setWecandeoVideo(wecandeoVideo);
  wecandeoSdk.setPlayerView(playerView);
  wecandeoSdk.setUseController(false);
  wecandeoSdk.onStart();
```

## LIVE
### Player 구성 방법
- 발급된 liveKey 로 영상 상세정보 조회를 하여 나온 videoUrl 값을 이용하여 Player 구성
```
  wecandeoSdk = new WecandeoSdk(this);
  wecandeoSdk.setSdkListener(this);
  wecandeoSdk.addPlayerListener(this);
  wecandeoVideo = new WecandeoVideo();
  wecandeoVideo.setDrm(false);
  wecandeoVideo.setVideoKey("videoUrl");
  wecandeoSdk.setWecandeoVideo(wecandeoVideo);
  wecandeoSdk.setPlayerView(playerView);
  wecandeoSdk.setUseController(false);
  wecandeoSdk.onStart();
```
- 공통
  - 해당 Activity 에 Player.EventListener, SdkInterface.onSdkListener 인터페이스를 구현해야 합니다.
    - ```Activity implements Player.EventListener, SdkInterface.onSdkListener```
  - onStart(), onResume(), onPause(), onStop() 메서드를 각 메서드에 맞게 wecandeoSdk 호출
  ```
  @Override
  public void onStart(){
      super.onStart();
      if(wecandeoSdk != null)
        wecandeoSdk.onStart();
  }
  ``` 
  - 이벤트
    - 재생 : wecandeoSdk.play()
    - 정지 : wecandeoSdk.stop()

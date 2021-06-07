---
layout: default
title: "Xcode의 실행환경에 따른 앱 빌드"
tags: iOS
---

앱을 개발하다 보면 각각의 실행환경(개발, 스테이징(테스트), 배포)에서 참고하는 값이 다를 수 있다.
API 서버 주소가 다를 수 있고, Google Analytics의 UA_CODE가 다를 수 있고, SNS 공유하기 위한 API Key의 값들이 다를 수 있다.
이렇게 실행환경에 따른 다른 값들이 Info.plist에 설정해야 하는 값도 있고, 소스상에서 구별을 해야하는 값들도 존재한다.
<br/>어떻게 하면 좀더 효율적으로 실행환경에 따른 값들을 사용할 수 있을까?

## 1. Configuration Settings File 생성 (개발/스테이징/배포)
![이미지 #1](/images/2021-05-21-xcode-variant/xcode-01.png)<br>
![이미지 #2](/images/2021-05-21-xcode-variant/xcode-02.png)<br>
Configuration Settings File을 선택한다.<br><br>
![이미지 #3](/images/2021-05-21-xcode-variant/xcode-03.png)
DevConfig.xcconfig(개발), StagingConfig.xcconfig(스테이징/테스트), ReleaseConfig.xcconfig(배포) 파일을 생성한다. 실행환경이 더 많이 존재하면 존재하는 만큼 생성해준다.<br>

<pre><code class="swift">// DevConfig.xcconfig

APP_BUNDLE_IDENTIFIER = com.sabisung.MyApp.dev
APP_NAME = MyApp(DEV)

API_KEY = DEV-abcdefghijklmn
</code></pre>
<pre><code class="swift">// StagingConfig.xcconfig

APP_BUNDLE_IDENTIFIER = com.sabisung.MyApp.staging
APP_NAME = MyApp(STA)

API_KEY = STA-abcdefghijklmn
</code></pre>
<pre><code class="swift">// ReleaseConfig.xcconfig

APP_BUNDLE_IDENTIFIER = com.sabisung.MyApp
APP_NAME = MyApp

API_KEY = REL-abcdefghijklmn
</code></pre>

## 2. Build Configuration 설정
![이미지 #4](/images/2021-05-21-xcode-variant/xcode-04.png)<br>
개발(Debug), 배포(Release)가 있으므로 스테이징을 위한 항목을 생성한다.<br><br>
![이미지 #5](/images/2021-05-21-xcode-variant/xcode-05.png)<br>
Staging이 생성된 화면. 각각의 실행환경에 맞도록 Configuration File을 지정한다.

## 3. Build Scheme 생성
![이미지 #6](/images/2021-05-21-xcode-variant/xcode-06.png)<br>
Xcode의 Product - Scheme - Manage Schemes...을 선택한다.<br><br>
![이미지 #7](/images/2021-05-21-xcode-variant/xcode-07.png)<br>
Scheme 이름과 Build Configruation항목을 선택한다. MyAppDev: Debug, MyAppStaging: Staging, MyApp: Release를 선택한다.<br><br>
![이미지 #8](/images/2021-05-21-xcode-variant/xcode-08.png)<br>
Scheme이 추가된 화면

## 4. Target 생성
![이미지 #9](/images/2021-05-21-xcode-variant/xcode-09.png)<br>
기존 MyApp을 복제하여 새로운 Target을 생성한다. MyAppDev, MyAppStaging을 생성한다.<br><br>
![이미지 #10](/images/2021-05-21-xcode-variant/xcode-10.png)<br>
Build Setting의 Packaging 항목의 Product Bundle Identifier의 값을 Configuration Settings File의 APP_BUNDLER_IDENTIFIER를 지정한다. 개발, 스테이징, 배포 앱을 동시에 설치할 수 있도록 하는 설정이다. Info.plist File 항목의 값은 MyAppDev, MyAppStaging, MyAppRelease 모두 동일하게 MyApp/Info.plist 지정한다.<br><br>
MyAppDev, MyAppStaging 항목도 동일한 방법으로 설정한다.<br>
Info.plist File 항목은 MyApp 항목의 값으로 모두 동일하게 지정한다.<br>
MyAppDev copy-Info.plist, MyAppStaging copy-Info.plist 파일은 삭제한다.<br>

## 5. Info.plist 파일 수정
실행환경에 따라서 Info.plist 항목의 값이 변경될 경우에 사용한다.<br>
각각 실행환경에 따른 값을 설정(Configuration Settings File)한 후에 아래와 같이 키 값을 설정하고, 값은 Configuration Settings File에서 지정한 값을 사용하도록 한다.<br>
![이미지 #11](/images/2021-05-21-xcode-variant/xcode-11.png)

## 6. 실행환경에 따른 상수들
위와 같이 Info.plist 파일에 실행환경에 따른 값을 지정을 해도 무방하지만 이러한 항목들이 많아지거나 공개되면 안되는 정보들은 소스 파일에 지정하여 사용할 수 있다. 즉 MyAppConfig.swift 파일을 각각 실행환경별로 생성한 다음 각 실행환경에 참고하여 사용할 수 있다.<br>
![이미지 #12](/images/2021-05-21-xcode-variant/xcode-12.png)<br>
일단 Group을 생성한다.<br><br>
![이미지 #13](/images/2021-05-21-xcode-variant/xcode-13.png)<br>
생성된 Group 폴더 구조<br><br>
![이미지 #14](/images/2021-05-21-xcode-variant/xcode-14.png)<br>
Dev, Staging, Release 각 폴더에 MyAppConfig.swift 파일을 생성한다.<br><br>
![이미지 #15](/images/2021-05-21-xcode-variant/xcode-15.png)<br>
파일 생성시에 생성되는 파일이 사용될 환경(Target)을 정확히 지정해야 한다. 중요한 부분이다.<br><br>
<pre><code class="swift">// Variant/Dev/MyAppConfig.swift

import Foundation

enum MyAppConfig {
    enum Variant {
        case dev
        case staging
        case release
    }
    
    static func getVariant() -> Variant {
        return .dev
    }
    
    static let EXEC_ENV_NAME = "개발"
}
</code></pre>
<pre><code class="swift">// Variant/Staging/MyAppConfig.swift

import Foundation

enum MyAppConfig {
    enum Variant {
        case dev
        case staging
        case release
    }
    
    static func getVariant() -> Variant {
        return .staging
    }
    
    static let EXEC_ENV_NAME = "스테이징"
}
</code></pre>
<pre><code class="swift">// Variant/Release/MyAppConfig.swift

import Foundation

enum MyAppConfig {
    enum Variant {
        case dev
        case staging
        case release
    }
    
    static func getVariant() -> Variant {
        return .release
    }
    
    static let EXEC_ENV_NAME = "배포"
}
</code></pre>
Xcode의 Product - Scheme - Manage Schemes... 선택한다.<br>
![이미지 #16](/images/2021-05-21-xcode-variant/xcode-16.png)<br>
각각의 Scheme의 Build Configuration을 지정한다. MyAppDev: MyAppDev.app, MyAppStaging: MyAppStaging.app, MyApp: MyApp.app 선택

## 7. 테스트 코드
ViewController.swift의 viewDidLoad 함수 내용이다.<br>
설정한 대로 정상적으로 값이 출력되는가?<br>
```swift
override func viewDidLoad() {
    super.viewDidLoad()
    // Do any additional setup after loading the view.
    
    print("실행환경: \(MyAppConfig.getVariant())")
    print(" EXEC_ENV_NAME: \(MyAppConfig.EXEC_ENV_NAME)")
    
    guard let apiKey: String = Bundle.main.infoDictionary?["API_KEY"] as? String else { return }
    print(" apiKey: \(apiKey)")
}
```
![이미지 #17](/images/2021-05-21-xcode-variant/xcode-17.png)<br>
보이는 시뮬레이터와 같이 동일한 앱이 3개가 설치되어 있다. 개발/스테이징/배포<br><br>
<a class="github-button" href="https://github.com/sabisung/sabisung.github.io/raw/master/download/MyApp.zip" data-color-scheme="no-preference: dark; light: dark; dark: dark;" data-icon="octicon-download" data-size="large" aria-label="Download ntkme/github-buttons on GitHub">Sample Source Download (MyApp.zip)</a>

## 8. 참고
<a href="https://medium.com/tunaiku-tech/how-to-create-build-variant-in-ios-application-66dfeb5bd091">https://medium.com/tunaiku-tech/how-to-create-build-variant-in-ios-application-66dfeb5bd091</a>

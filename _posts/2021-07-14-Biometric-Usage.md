---
layout: default
title: "생체인식 사용하기"
tags: iOS Swift
---

## 왜?
요즘 같이 다수의 사이트에 로그인을 할려고 하면 각각 사이트의 아이디와 비밀번호를 기억해야 한다.<br/>
해킹 방지를 위해서 각각 사이트마다 아이디와 비밀번호를 다르게 지정하는 경우도 있고,<br/>
시간이 지남에 따라서 자주 사용하지 않는 사이트인 경우에는 아이디와 비밀번호를 잊어먹기가 빈번하다.<br/>
(얼마나 많은 인증 문자를 받았던가~)<br/>
심지어는 사이트 이용을 하지 않는 경우도 있다.<br/>
다른 대체 사이트를 이용하는 유저도 많다...<br/>
<br/>
이러한 불편함과 해킹으로 인한 정보 유출 사고를 방지하고자 사용자 개별의 고유한 생체 정보를 사용하여 인증을 하여 사용자 정보 보호가 필요한 상태다.

## iPhone의 생체 정보
iPhone에서 사용이 가능한 생체 정보는 지문과 안면 인식 두 가지가 존재한다.<br/>

## Xcode 설정
Info.plist 파일에 아래 내용을 추가해야 한다.<br/>
<pre><code class="swift">&lt;key&gt;NSFaceIDUsageDescription&lt;/key&gt;
&lt;string&gt;안면인식 로그인을 위해 필요합니다.&lt;/string&gt;
</code></pre>

## BiometricAuth.swift
<pre><code class="swift">//
//  BiometricAuth.swift
//  BiometricAuth
//
//  Created by sabisung on 2021/07/14.
//

import Foundation
import LocalAuthentication

/// 생체 인증 유형
enum BiometricType {
  case none     // 없음
  case touchID  // 지문
  case faceID   // 안면
}

@available(iOS 11.0, *)
class BiometricAuth {
    private let localizedReason4TouchID = "Touch ID를 활성화하려면 사용자 암호가 필요합니다."
    private let localizedReason4FaceID = "Face ID를 활성화하려면 사용자 암호가 필요합니다."
    private let passcodeInterval: TimeInterval = 0.3
    
    private var biometricPolicy: LAPolicy
    private let completionHandler: (Bool, Error?) -> Void
    private var isPasscode: Bool = true
    
    private var laContext = LAContext()
    
    /// 생성자
    /// - Parameters:
    ///   - biometricPolicy: 생체인증 정책
    ///   - isPasscode: 생체인증 실패시 passcode 사용여부
    ///   - completionHandler: 핸들러
    init(biometricPolicy: LAPolicy, isPasscode: Bool, completionHandler: @escaping (Bool, Error?) -> Void) {
        self.biometricPolicy = biometricPolicy
        self.isPasscode = isPasscode
        self.completionHandler = completionHandler
    }
    
    /// 생성자
    /// - Parameters:
    ///   - isPasscode: 생체인증 실패시 passcode 사용여부
    ///   - completionHandler: 핸들러
    convenience init(isPasscode: Bool, completionHandler: @escaping (Bool, Error?) -> Void) {
        self.init(biometricPolicy: LAPolicy.deviceOwnerAuthenticationWithBiometrics, isPasscode: isPasscode, completionHandler: completionHandler)
    }
    
    /// 생성자
    /// - Parameter completionHandler: 핸들러
    convenience init(_ completionHandler: @escaping (Bool, Error?) -> Void) {
        self.init(biometricPolicy: LAPolicy.deviceOwnerAuthenticationWithBiometrics, isPasscode: true, completionHandler: completionHandler)
    }
    
    /// 소멸자
    deinit {
        self.laContext.invalidate()
    }
    
    /// 생체인증 가능 여부
    /// - Returns: 가능여부
    func canEvaluatePolicy() -> Bool {
        return laContext.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, error: nil)
    }
    
    /// 기기가 지원하는 생체인증 유형
    /// - Returns: 생체인증 유형
    func biometricType() -> BiometricType {
        if canEvaluatePolicy() {
            switch laContext.biometryType {
            case .none:
                return .none
            case .touchID:
                return .touchID
            case .faceID:
                return .faceID
            default:
                return .none
            }
        }
        return .none
    }
    
    /// 생체인증 진행
    func authenticateBiometric() {
        guard canEvaluatePolicy() else {
            completionHandler(false, NSError(domain: "", code: LAError.biometryNotAvailable.rawValue, userInfo: [NSLocalizedDescriptionKey: "biometry not available"]))
            return
        }
        let bioType = biometricType()
        guard bioType != .none else {
            completionHandler(false, NSError(domain: "", code: LAError.biometryNotAvailable.rawValue, userInfo: [NSLocalizedDescriptionKey: "biometry not available"]))
            return
        }
        
        if !isPasscode {
            laContext.localizedFallbackTitle = "" // 생체 인증 실패시 passcode 입력 불가 처리
        }
        let localizedReason = bioType == .touchID ? localizedReason4TouchID : (bioType == .faceID ? localizedReason4FaceID : " ")
        laContext.evaluatePolicy(self.biometricPolicy, localizedReason: localizedReason) { (success, error) in
            if success {
                self.completionHandler(true, nil)
            } else {
                switch error {
                case LAError.userFallback?: // passcode 입력
                    print("[BIO] userFallback...")
                    self.laContext.invalidate()
                    DispatchQueue.main.asyncAfter(deadline: .now() + self.passcodeInterval) {
                        self.laContext = LAContext()
                        self.biometricPolicy = .deviceOwnerAuthentication
                        self.authenticateBiometric()
                    }
                default:
                    self.completionHandler(false, error)
                }
            }
        }
    }
}
</code></pre>

# ViewController.swift
<pre><code class="swift">//
//  ViewController.swift
//  BiometricAuth
//
//  Created by sabisung on 2021/07/14.
//

import UIKit

class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
    }

    @IBAction func biometricAuthWithPasscodeTapped(_ sender: UIButton) {
        if #available(iOS 11.0, *) {
            // 생체인증 실패시 passcode 사용
            BiometricAuth { (success, error) in
                print("[BIO] success: \(success)")
                print("[BIO] error: \(String(describing: error))")
            }.authenticateBiometric()
        }
    }
    
    @IBAction func biometricAuthTapped(_ sender: UIButton) {
        if #available(iOS 11.0, *) {
            // 생체인증 실패시 passcode 미사용
            BiometricAuth(isPasscode: false) { success, error in
                print("[BIO] success: \(success)")
                print("[BIO] error: \(String(describing: error))")
            }.authenticateBiometric()
        }
    }
}
</code></pre>

## 실행 샘플 이미지
![이미지 #1](/images/2021-07-14-Biometric-Usage/biometric-usage-01.png){: width="100%"}<br>
![이미지 #2](/images/2021-07-14-Biometric-Usage/biometric-usage-02.png){: width="100%"}<br>
![이미지 #3](/images/2021-07-14-Biometric-Usage/biometric-usage-03.png){: width="100%"}<br>
![이미지 #4](/images/2021-07-14-Biometric-Usage/biometric-usage-04.png){: width="100%"}<br>
![이미지 #5](/images/2021-07-14-Biometric-Usage/biometric-usage-05.png){: width="100%"}<br>

## 샘플 소스 다운로드
<a class="github-button" href="https://github.com/sabisung/sabisung.github.io/raw/master/download/BiometricAuth.zip" data-color-scheme="no-preference: dark; light: dark; dark: dark;" data-icon="octicon-download" data-size="large" aria-label="Download ntkme/github-buttons on GitHub">Sample Source Download (BiometricAuth.zip)</a>

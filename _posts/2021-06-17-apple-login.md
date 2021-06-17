---
layout: default
title: "애플 로그인 구현"
tags: iOS Swift
---

## 왜?
다양한 앱들에서 이미 가입된 SNS 계정을 사용하여 회원가입을 유도하고 있다.<br/>
이미 검증된 사용자 정보를 얻기 쉬우니깐... ㅋㅋㅋ<br/>

## Xcode 설정
구현하기 전에 Xcode의 앱 설정에서 애플 로그인 기능을 사용하겠다고 설정한다.<br/>
![이미지 #1](/images/2021-06-17-apple-login/apple-login-01.png)<br>

## 구현
<pre><code class="swift">//
//  ViewController.swift
//  AppleLogin
//
//  Created by sabisung on 2021/06/17.
//

import UIKit
import AuthenticationServices
import JWTDecode

class ViewController: UIViewController {
    
    struct UserInfo: Codable {
        let id: String
        let email: String?
        let name: String?
        let authCode: String
        var dictionary: [String: String] {
            return [
                "id": id,
                "email": email ?? "",
                "name": name ?? "",
                "authCode": authCode,
            ]
        }
    }
    
    /// 로그인 플래그 값
    private let appleLoginFlagKey = "applelogin.user.islogin"

    /// 로그인 사용자 정보
    private let appleLoginUserInfoKey = "applelogin.user.info"

    @IBOutlet weak var appleLoginView: UIView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
        
        //애플 로그인 버튼
        let appleLoginBtn = ASAuthorizationAppleIDButton(authorizationButtonType: .signIn, authorizationButtonStyle: .black)
        appleLoginBtn.addTarget(self, action: #selector(appleLoginBtnHander), for: .touchUpInside)
        appleLoginView.addSubview(appleLoginBtn)
    }
    
    /// 애플 로그인 버튼 핸들러
    /// - Parameter sender: UIButton
    @objc func appleLoginBtnHander(_ sender: UIButton) {
        let request = ASAuthorizationAppleIDProvider().createRequest()
        request.requestedScopes = [.email, .fullName]
        let controller = ASAuthorizationController(authorizationRequests: [request])
        controller.delegate = self
        controller.presentationContextProvider = self
        controller.performRequests()
    }

    /// 로그인 정보를 UserDefaults에 저장
    /// - Parameters:
    ///   - isLogin: 로그인 여부
    ///   - userInfo: UserInfo 구조체
    private func saveUserInfo(isLogin: Bool, userInfo: UserInfo?) {
        let defaults = UserDefaults.standard
        defaults.set(isLogin, forKey: appleLoginFlagKey)
                
        if isLogin {
            if let jsonData = try? JSONEncoder().encode(userInfo), let jsonString = String.init(data: jsonData, encoding: .utf8) {
                defaults.set(jsonString, forKey: appleLoginUserInfoKey)
            }
        } else {
            defaults.removeObject(forKey: appleLoginUserInfoKey)
        }
    }
    
    /// UserDefaults에 저장된 사용자 정보 반환
    /// - Returns: UserInfo 구조체
    private func getUserInfo() -> UserInfo? {
        let userInfo = UserDefaults.standard.string(forKey: appleLoginUserInfoKey)
        guard let userInfoData = userInfo?.data(using: .utf8) else {
            return nil
        }
        return try! JSONDecoder().decode(UserInfo.self, from: userInfoData)
    }
}

extension ViewController: ASAuthorizationControllerDelegate, ASAuthorizationControllerPresentationContextProviding {
    func presentationAnchor(for controller: ASAuthorizationController) -> ASPresentationAnchor {
        //return UIApplication.shared.keyWindow!
        return UIApplication.shared.windows.filter {$0.isKeyWindow}.first!
    }
    
    /// 애플 로그인 실패
    /// - Parameters:
    ///   - controller: ASAuthorizationController 객체
    ///   - error: Error 객체
    func authorizationController(controller: ASAuthorizationController, didCompleteWithError error: Error) {
        print("\(error.localizedDescription)")
    }
    
    /// 애플 로그인 인증 성공
    /// - Parameters:
    ///   - controller: ASAuthorizationController 객체
    ///   - authorization: ASAuthorization 객체
    func authorizationController(controller: ASAuthorizationController, didCompleteWithAuthorization authorization: ASAuthorization) {
        guard let credential = authorization.credential as? ASAuthorizationAppleIDCredential else {
            saveUserInfo(isLogin: false, userInfo: nil)
            return
        }
        let user: String = credential.user
        var name: String = "\(credential.fullName?.familyName ?? "")\(credential.fullName?.givenName ?? "")"
        if name.count == 0, let savedUserInfo = self.getUserInfo() {
            name = savedUserInfo.name ?? ""
        }
        let authCode: String = "\(String(data: credential.authorizationCode ?? Data(), encoding: .utf8) ?? "")"
        let identifyToken: String = "\(String(data: credential.identityToken ?? Data(), encoding: .utf8) ?? "")" //JWT Token. 항상 email 주소 존재
            
        // identifyToken값내의 email을 사용.
        guard let jwtJson = try? JWTDecode.decode(jwt: identifyToken), let email = jwtJson.body["email"] as? String else {
            saveUserInfo(isLogin: false, userInfo: nil)
            return
        }
        saveUserInfo(isLogin: true, userInfo: UserInfo(id: user, email: email, name: name, authCode: authCode))
        print("User Info: \(getUserInfo())")
    }
}
</code></pre>

## 잠깐!!
애플 로그인 인증 후 사용자 정보가 내려오는데 최초 인증시 내려주는 정보와 이후 인증 후 내려주는 정보의 항목이 다르다.<br/>
이름과 이메일 정보는 최초 인증 후에만 내려주고 이후 인증에서는 내려주지 않는다.<br/>
다만, 이메일 정보는 JWT를 Decoding하여 항상 얻을 수 있다.<br/>
따라서, 이름 정보가 필요할 경우에 대비하여 최초 인증 후 이름을 저장하여 사용한다.<br/>

## 샘플 소스 다운로드
<a class="github-button" href="https://github.com/sabisung/sabisung.github.io/raw/master/download/AppleLogin.zip" data-color-scheme="no-preference: dark; light: dark; dark: dark;" data-icon="octicon-download" data-size="large" aria-label="Download ntkme/github-buttons on GitHub">Sample Source Download (AppleLogin.zip)</a>

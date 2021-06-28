---
layout: default
title: "RootViewController 변경하기"
tags: iOS Swift RxSwift
---

## 시나리오
앱 실행시 InitViewController에서 Alamofire를 이용하여 이미지를 랜덤하게 다운로드하여 이미지 뷰에 표시를 한 후<br/>
5초 후에 ViewController(메인)로 전환하여 다운로드한 이미지를 전달하여 이미지 뷰에 표시를 한다.<br/>
요즘 공부중인 RxSwift를 이용하여 InitViewController에서 ViewController로 이미지를 전달한다.<br/>

## InitViewController.swift
<pre><code class="swift">//
//  InitViewController.swift
//  ChangeRootVC
//
//  Created by sabisung on 2021/06/23.
//

import UIKit
import Alamofire
import RxSwift

class InitViewController: UIViewController {

    @IBOutlet weak var initImageView: UIImageView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
        
        let mainViewController = self.storyboard?.instantiateViewController(withIdentifier: "MainViewController") as! ViewController
        
        randomImageDownload("https://picsum.photos/800/600?random", completionHandler: { [weak self] data in
            print("Image Download Done...")
            self?.initImageView.image = UIImage(data:  data)
            
            mainViewController.transImageSubject.onNext(UIImage(data:  data))
        })
        
        DispatchQueue.main.asyncAfter(deadline: .now() + 5, execute: {
            print("Switch to MainViewController...")
            UIApplication.shared.delegate?.window??.rootViewController = mainViewController
            mainViewController.transImageSubject.onCompleted()
        })
    }
    
    /// 이미지 다운로드
    /// - Parameters:
    ///   - urlString: URL
    ///   - completionHandler: 완료 핸들러
    func randomImageDownload(_ urlString: String, completionHandler: @escaping (Data) -> Void) {
        AF.request(
            urlString,
            method: .get
        )
        .response { resp in
            let response: DataResponse<Data?, AFError>? = resp
            
            guard response?.error == nil else {
                print("ERROR: \(String(describing: response?.error.debugDescription))")
                return
            }
            
            guard let data = response?.data else {
                print("ERROR: No Image")
                return
            }
            
            completionHandler(data)
        }
    }
}
</code></pre>

## ViewController.swift
<pre><code class="swift">//
//  ViewController.swift
//  ChangeRootVC
//
//  Created by sabisung on 2021/06/23.
//

import UIKit
import RxSwift

class ViewController: UIViewController {
    
    private var disposeBag = DisposeBag()
    let transImageSubject = BehaviorSubject<UIImage?>(value: nil)

    @IBOutlet weak var mainImageView: UIImageView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
        
        transImageSubject
            .debug("ViewController")
            .subscribe(onNext: {
                self.mainImageView.image = $0
            })
            .disposed(by: disposeBag)
    }
}
</code></pre>

## 스크린 샷
![InitViewController](/images/2021-06-23-Change-Root-ViewController/ChangeRootVC-01.png){: width="100%"}<br/>
![ViewController](/images/2021-06-23-Change-Root-ViewController/ChangeRootVC-02.png){: width="100%"}<br/>

## 샘플 소스 다운로드
<a class="github-button" href="https://github.com/sabisung/sabisung.github.io/raw/master/download/ChangeRootVC.zip" data-color-scheme="no-preference: dark; light: dark; dark: dark;" data-icon="octicon-download" data-size="large" aria-label="Download ntkme/github-buttons on GitHub">Sample Source Download (ChangeRootVC.zip)</a>

---
layout: default
title: "무한 스크롤링"
tags: iOS Swift
---

## 무한 스크롤이란?
우측 스크롤인 경우 마지막 항목에 도달하면 다시 처음 항목으로 돌아가서 스크롤이 되고,<br/>
좌측 스크롤인 경우 처음에 도달하면 다시 마지막 항목부터 스크롤이 되는 것을 말한다.<br/>

## 소개
스크롤 뷰를 이용한 무한 스크롤을 구현한 샘플 코드이다.<br/>
이미지가 1개일 경우에는 스크롤이 되지 않지만 2개 이상일 경우에는 무한으로 스크롤을 한다.<br/>
스크롤이 마지막에 도달하면 다시 첫번째 이미지부터 스크롤이 된다.<br/>
사용자가 직접 스크롤링을 하는 경우도 마찬가지다.<br/>
물론, 좌/우로 스크롤이 가능하다.<br/>

## 트릭
무한 스크롤이라고 해서 스크롤 항목을 무한대로 가지고 있지는 않다.<br/>
무한대면 어쩔....ㅠㅠ<br/>
단지, 트릭을 사용할 뿐이다.<br/>
즉, 스크롤을 하여 마지막 항목에 도달하면 첫번째 항목을 삭제 후 이를 다시 마지막에 추가를 해서 마치 첫번째로 돌아가는 것처럼 하는 것이다.<br/>
첫번째 항목인 경우 좌측으로 스크롤을 하는 경우에는 마지막 항목을 삭제 후 이를 다시 첫번째에 추가해서 마지막으로 돌아가는 것처럼 보이도록 하는 것이다.<br/>

## 구현
<pre><code class="swift">//
//  ViewController.swift
//  ImageRotate
//
//  Created by sabisung on 2021/06/18.
//

import UIKit

class ViewController: UIViewController {
    
    /// 스크롤되는 이미지 리스트
    let images: [UIImage] = [
        UIImage(named: "Image-1")!,
        UIImage(named: "Image-2")!,
        UIImage(named: "Image-3")!,
        UIImage(named: "Image-4")!,
        UIImage(named: "Image-5")!
    ]
    /// ImageView 리스트
    var imageViews: [UIImageView] = []
    /// 자동 스크롤 시간 간격
    let TIMER_INTERVAL: TimeInterval = 3
    /// Timer
    var timer: Timer? = nil
    /// 드래깅 여부
    var isDragging: Bool = false
    
    /// 스크롤 뷰
    @IBOutlet weak var scrollView: UIScrollView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
        
        setup()
    }
    
    /// UI 설정
    func setup() {
        scrollView.isPagingEnabled = true
        scrollView.showsHorizontalScrollIndicator = false
        scrollView.showsVerticalScrollIndicator = false
        scrollView.delegate = self
        
        imageViews = createImageViews()
        if images.count < 3 {
            imageViews.append(contentsOf: createImageViews())
        }
        
        scrollView.isScrollEnabled = images.count > 1
        scrollView.contentSize = CGSize(width: scrollView.frame.width * CGFloat(imageViews.count), height: scrollView.frame.height)
        scrollView.contentOffset = CGPoint(x: 0, y: 0)
        
        layoutImageViews()
        
        startTimer()
    }
    
    /// 스크롤되는 이미지 뷰 생성
    /// - Returns: 이미지 뷰 리스트
    func createImageViews() -> [UIImageView] {
        return images.map { image -> UIImageView in
            let imageView = UIImageView()
            imageView.contentMode = .scaleAspectFill
            imageView.backgroundColor = .clear
            imageView.image = image
            
            scrollView.addSubview(imageView)
            
            return imageView
        }
    }
    
    /// 이미지 뷰 레이아웃 설정
    func layoutImageViews() {
        imageViews.enumerated().forEach { (index, imageView) in
            imageView.frame = CGRect(x: scrollView.frame.width * CGFloat(index),
                                     y: 0,
                                     width: scrollView.frame.width,
                                     height: scrollView.frame.height)
        }
    }
    
    /// 타이머 시작
    func startTimer() {
        guard images.count > 1 else { return }
        stopTimer()
        timer = Timer.scheduledTimer(timeInterval: TIMER_INTERVAL,
                                     target: self,
                                     selector: #selector(timerCallback),
                                     userInfo: nil,
                                     repeats: false
        )
    }
    
    /// 타이머 중지
    func stopTimer() {
        if let t = timer {
            t.invalidate()
        }
        timer = nil
    }
    
    /// 타이머 콜백
    /// - Parameter sender: 타이머
    @objc func timerCallback(_ sender: Timer) {
        var offset = scrollView.contentOffset
        offset.x += scrollView.frame.width
        scrollView.setContentOffset(offset, animated: true)
    }
}

extension ViewController: UIScrollViewDelegate {
    func scrollViewWillBeginDecelerating(_ scrollView: UIScrollView) {
        isDragging = true
        stopTimer()
    }
    
    func scrollViewDidEndDecelerating(_ scrollView: UIScrollView) {
        isDragging = false
        startTimer()
    }
    
    func scrollViewDidEndScrollingAnimation(_ scrollView: UIScrollView) {
        addImageViewAtLast()
        startTimer()
    }
    
    func scrollViewDidScroll(_ scrollView: UIScrollView) {
        if !isDragging {
            return
        }

        let offsetX = scrollView.contentOffset.x
        
        if offsetX > scrollView.frame.size.width * 1.5 {
            addImageViewAtLast()
        }

        if offsetX < scrollView.frame.size.width * 0.5 {
            addImageViewAtFirst()
        }
    }
    
    /// 첫번째 이미지 뷰를 삭제후 마지막에 삽입
    func addImageViewAtLast() {
        if let imageView = self.imageViews.first {
            self.imageViews.remove(at: 0)
            self.imageViews.append(imageView)
            self.layoutImageViews()
            self.scrollView.contentOffset.x -= self.scrollView.frame.width
        }
    }
    
    /// 마지막 이미지 뷰를 삭제후 처음에 삽입
    func addImageViewAtFirst() {
        if let imageView = self.imageViews.last {
            self.imageViews.removeLast()
            self.imageViews.insert(imageView, at: 0)
            self.layoutImageViews()
            self.scrollView.contentOffset.x += self.scrollView.frame.width
        }
    }
}
</code></pre>

## 동작
![이미지 #1](/images/2021-06-18-infinite-scroll/infinite-scroll.gif)<br>

## 샘플 소스 다운로드
<a class="github-button" href="https://github.com/sabisung/sabisung.github.io/raw/master/download/ImageRotate.zip" data-color-scheme="no-preference: dark; light: dark; dark: dark;" data-icon="octicon-download" data-size="large" aria-label="Download ntkme/github-buttons on GitHub">Sample Source Download (ImageRotate.zip)</a>

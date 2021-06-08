---
layout: default
title: "Playground에서 RxSwift 사용하기"
tags: iOS Swift RxSwift
---

Xcode를 사용하여 일반적인 방법으로 프로젝트를 생성하고, Pod에 RxSwift를 추가하면 앱을 실행하여 디바이스나 시뮬레이터 결과를 확인할 수가 있다. 
그런데 단순히 RxSwift를 공부하기 위해서 매번 앱을 실행하여 결과를 보기에는 뭔가(???)가 부담스럽다. 매번 실행하기도 그렇고...<br/>
Playground에서 RxSwift를 코딩하고, 곧바로 실행된 결과를 보면 좋지 않을까?<br/>
근데 요놈스키(??)가 잘 안된다...ㅠㅠ<br/>
<br/>
일단, Xcode를 이용하여 프로젝트를 생성한다.<br/>

![Xcode 프로젝트 생성](/images/2021-04-08-Playground에서-RxSwift-사용하기/rxswift-01.png)<br>
Xcode를 실행하여 프로젝트를 생성한다.<br/>
<br/>
![App 템플릿 선택](/images/2021-04-08-Playground에서-RxSwift-사용하기/rxswift-02.png)<br>
App 선택 후 Next 클릭 (Xcode 버전에 따라서 화면이 상이할 수 있으나 해당 버전에 맞게 프로젝트를 생성하면 된다)<br/>
<br/>
![App 정보 입력](/images/2021-04-08-Playground에서-RxSwift-사용하기/rxswift-03.png)<br>
본인의 입맛에 맞게 프로젝트명을 입력한다.<br/>
프로젝트가 생성되었다면 아무런 작업도 하지 말고, 묻지도 따지지도 말고 Xcode를 종료한다. :-)<br/>
<br/>
그런다음 터미널을 실행하여 프로젝트가 생성된 폴더로 이동한다.<br/>
<mark style='background-color: #24292e'><font color="white">주) 먼저 CocoaPods가 설치가 되어야 한다.</font></mark><br/>
<br/>
![터미널](/images/2021-04-08-Playground에서-RxSwift-사용하기/rxswift-05.png)<br>
터미널을 실행한 후 프로젝트 폴더로 이동한다. 그런다음 pod init 명령어를 입력하게 되면 아무런 메시지도 나오지 않는다. ls 명령어로 폴더의 파일 목록을 보면 Podfile이라는 파일이 생성되어 있다. 이 Podfile을 vi로 오픈한다.<br/>
<br/>
![터미널](/images/2021-04-08-Playground에서-RxSwift-사용하기/rxswift-04.png)<br>
Podfile을 오픈하여 위 이미지와 같이 # Pods for RxSwiftStudy 라인 아래에 pod 'RxSwift'를 입력한 후 저장하고 vi를 종료한다.<br/>
<br/>
![터미널](/images/2021-04-08-Playground에서-RxSwift-사용하기/rxswift-05.png)<br>
<code>vi Podfile</code> 명령어까지 완료되었다면 pod install을 입력하여 RxSwift를 설치한다.<br/>
<br/>
![파인더](/images/2021-04-08-Playground에서-RxSwift-사용하기/rxswift-06.png)<br>
<code>pod install</code>이 정상적으로 실행되었다면 위와 같이 RxSwiftStudy.xcworkspace라는 파일이 생성된다. 이 파일을 더블클릭한다. Xcode가 실행되어 프로젝트가 오픈된다.<br/>
<br/>
![Xcode](/images/2021-04-08-Playground에서-RxSwift-사용하기/rxswift-07.png)<br>
Xcode에서 프로젝트가 오픈된 상태다. 큰 빨간색 사각형의 프로젝트 네비게이션 부분은 절대로 클릭하면 안된다. 이미 클릭하여 폴더나 파일이 선택되었다면 ⌘+Click으로 선택된 항목을 반드시 선택해제 해야 한다. 그래도 불안하면 Xcode를 종료후 다시 오픈한다. (RxSwiftStudy.xcworkspace 이용) 왼쪽 하단의 + 버튼을 클릭한다.<br/>
<br/>
<mark style='background-color: #24292e'><font color="white">빨간색 큰 사각형의 프로젝트 네비게이터 부분의 항목을 절대로 클릭하면 안된다!!!</font></mark><br/>
<br/>
![Xcode](/images/2021-04-08-Playground에서-RxSwift-사용하기/rxswift-08.png)<br>
New File...을 클릭한다.<br/>
<br/>
![Xcode](/images/2021-04-08-Playground에서-RxSwift-사용하기/rxswift-09.png)<br>
Blank Playgroud를 선택 후 Next 버튼 클릭<br/>
<br/>
![Xcode](/images/2021-04-08-Playground에서-RxSwift-사용하기/rxswift-10.png)<br>
파일명을 지정한다.<br/>
<br/>
![Xcode](/images/2021-04-08-Playground에서-RxSwift-사용하기/rxswift-11.png)<br>
마지막으로!!! 이젠 생성된 Playgroud에서 RxSwift를 사용할 수 있다. :-)<br/>
프로젝트 네비게이터 상단에 보면 생성된 Playground 항목을 볼 수 있다.<br/>
<br/>
## 참고
<a href="https://medium.com/@_achou/making-a-playground-using-rxswift-81d8377bd239">https://medium.com/@_achou/making-a-playground-using-rxswift-81d8377bd239</a>

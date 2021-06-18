---
layout: default
title: "이미지 다운로드 (Alamofire)"
tags: iOS Swift
---

Alamofire를 이용하여 이미지를 다운로드하여 저장하는 샘플.

## 구현
<pre><code class="swift">import UIKit
import Alamofire

/// 랜덤 이미지 다운로드
/// - Parameters:
///   - urlString: 이미지 URL
///   - completionHandler: 이미지 다운로드 완료 후 실행되는 함수
func randomImageDownload(_ urlString: String, completionHandler: @escaping (Data) -> Void) {
    AF.request(
        urlString,
        method: .get
    ).response { resp in
        let response: DataResponse<Data?, AFError>? = resp
        
        guard response?.error == nil else {
            print("ERROR: \(String(describing: response?.error.debugDescription))")
            return
        }
        
        guard let data = response?.data else {
            return
        }
        completionHandler(data)
    }
}

/// Document Path 가져오기
/// - Returns: Document Path URL
func getDocumentPath() -> URL {
    let paths = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)
    return paths[0]
}

randomImageDownload("https://picsum.photos/800/600?random") {
    guard let pngData = UIImage(data: $0)?.pngData() else {
        return
    }
    let fileURL = getDocumentPath().appendingPathComponent("download-\(UUID().uuidString).png")
    print("fileURL: \(fileURL.absoluteString)")
    try? pngData.write(to: fileURL)
}
</code></pre>

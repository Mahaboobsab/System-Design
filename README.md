# System-Design
🎯 System Design Interview: File Downloader App (Mobile)  

**✅ Step 1: Clarify Requirements**  

**Interviewer**:  

**"Design a mobile app that downloads files. What are the core features you'd consider?"**  


**You**:
**"Let me clarify a few things first:**  

- What types of files are we downloading (PDFs, videos, images)?  
- Should downloads support pause/resume?  
- Do we need background download support?  
- Is there a limit on file size or number of concurrent downloads?  
- Should we support offline access and caching?"  

**✅ Step 2: Define Functional Requirements**  

**You**:  

**"Based on typical use cases, here are the functional requirements:**  


- Download files from a given URL
- Show download progress
- Support pause/resume/cancel
- Store files locally for offline access
- Retry failed downloads
- Notify user on completion

**✅ Step 3: Define Non-Functional Requirements**  

**You**:  
**"Non-functional requirements might include:**  

- Efficient memory and disk usage  
- Secure file storage (e.g., sandboxed, encrypted if needed)  
- Background download support  
- Minimal impact on battery and performance  
- Support for iOS 13+ for broader device compatibility

**✅ Step 4: High-Level Architecture**  

**You**:  
**"I’d use the following components:**  

- **UI Layer**: SwiftUI or UIKit for download list and progress  
- **Download Manager**: Handles queueing, progress, pause/resume  
- **File Storage**: Uses FileManager to save files in app sandbox  
- **Networking**: URLSessionDownloadTask with background configuration  
- **Persistence**: Core Data or SQLite to track download metadata  
- **Notification System**: Local notifications for completion alerts

**✅ Step 5: Component Design**  

**Interviewer**:
"How would you implement the download manager?"

**You**:
**"I’d create a singleton DownloadManager class:**  

- Uses URLSession(configuration: .background) for background downloads  
- Maintains a queue of DownloadTask objects  
- Each task tracks URL, progress, status, and local file path  
- Implements delegate methods to handle completion and errors  
- Stores resume data for interrupted downloads"

**✅ Step 6: Edge Cases & Trade-offs**  

**You**:  
**"Some edge cases and trade-offs:**  

- **Network failure**: Retry with exponential backoff  
- **App termination**: Use resume data to continue downloads  
- **Storage limits**: Warn user if disk space is low  
- **Concurrency**: Limit to 3–5 parallel downloads to avoid overload"

**✅ Step 7: Security & Privacy**  

**You**:  
**"If files are sensitive:**  

- Use AES encryption for local storage  
- Store keys securely in Keychain  
- Respect user privacy and avoid tracking"

**✅ Step 8: Monitoring & Analytics**  

**You**:  
-   "I’d log download events (start, complete, fail) for analytics.  
-    Use tools like Firebase or custom logging to monitor performance and errors."

![IMG_20250730_222821](https://github.com/user-attachments/assets/2c3bb8d4-c027-4c31-9cd6-e94c7a08653f)


**📦 Swift Code Template: DownloadManager**  
```swift
import Foundation

class DownloadManager: NSObject, URLSessionDownloadDelegate {
    static let shared = DownloadManager()

    private var session: URLSession!
    private var activeDownloads: [URL: URLSessionDownloadTask] = [:]

    override init() {
        super.init()
        let config = URLSessionConfiguration.background(withIdentifier: "com.yourapp.download")
        config.isDiscretionary = true
        config.sessionSendsLaunchEvents = true
        session = URLSession(configuration: config, delegate: self, delegateQueue: nil)
    }

    func startDownload(from url: URL) {
        guard activeDownloads[url] == nil else { return }
        let task = session.downloadTask(with: url)
        activeDownloads[url] = task
        task.resume()
    }

    func pauseDownload(for url: URL) {
        guard let task = activeDownloads[url] else { return }
        task.cancel(byProducingResumeData: { resumeData in
            // Store resumeData if needed
        })
        activeDownloads.removeValue(forKey: url)
    }

    func resumeDownload(with resumeData: Data, for url: URL) {
        let task = session.downloadTask(withResumeData: resumeData)
        activeDownloads[url] = task
        task.resume()
    }

    // MARK: - URLSessionDownloadDelegate

    func urlSession(_ session: URLSession, downloadTask: URLSessionDownloadTask, didFinishDownloadingTo location: URL) {
        let fileManager = FileManager.default
        let documents = fileManager.urls(for: .documentDirectory, in: .userDomainMask).first!
        let destinationURL = documents.appendingPathComponent(downloadTask.originalRequest?.url?.lastPathComponent ?? "file.pdf")

        do {
            if fileManager.fileExists(atPath: destinationURL.path) {
                try fileManager.removeItem(at: destinationURL)
            }
            try fileManager.moveItem(at: location, to: destinationURL)
            print("✅ File saved to: \(destinationURL.path)")
        } catch {
            print("❌ File move error: \(error.localizedDescription)")
        }

        activeDownloads.removeValue(forKey: downloadTask.originalRequest!.url!)
    }

    func urlSession(_ session: URLSession, task: URLSessionTask, didCompleteWithError error: Error?) {
        if let error = error {
            print("❌ Download failed: \(error.localizedDescription)")
        }
    }
}

```
**✅ When to Write Only DownloadManager**  

**You can focus just on DownloadManager if:**  

- You're asked to implement core download logic  
- The interviewer wants to test your networking and concurrency skills  
- You're integrating it into an existing app

**🧱 When to Build the Full System**  

**You should build the full system if:**  

- You're designing a complete file downloader app  
- You need to show UI integration, progress tracking, and file management  
- You're preparing for a system design interview that expects end-to-end thinking 


  



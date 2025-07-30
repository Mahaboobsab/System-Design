# System-Design
🎯 System Design Interview: File Downloader App (Mobile)  

---

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
---
**✅ Step 2: Define Functional Requirements**  

**You**:  

**"Based on typical use cases, here are the functional requirements:**  

- Download files from a given URL
- Show download progress
- Support pause/resume/cancel
- Store files locally for offline access
- Retry failed downloads
- Notify user on completion
---  

**✅ Step 3: Define Non-Functional Requirements**  

**You**:  
**"Non-functional requirements might include:**  

- Efficient memory and disk usage  
- Secure file storage (e.g., sandboxed, encrypted if needed)  
- Background download support  
- Minimal impact on battery and performance  
- Support for iOS 13+ for broader device compatibility
--- 

**✅ Step 4: High-Level Architecture**  

**You**:  
**"I’d use the following components:**  

- **UI Layer**: SwiftUI or UIKit for download list and progress  
- **Download Manager**: Handles queueing, progress, pause/resume  
- **File Storage**: Uses FileManager to save files in app sandbox  
- **Networking**: URLSessionDownloadTask with background configuration  
- **Persistence**: Core Data or SQLite to track download metadata  
- **Notification System**: Local notifications for completion alerts
--- 

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
--- 

**✅ Step 6: Edge Cases & Trade-offs**  

**You**:  
**"Some edge cases and trade-offs:**  

- **Network failure**: Retry with exponential backoff  
- **App termination**: Use resume data to continue downloads  
- **Storage limits**: Warn user if disk space is low  
- **Concurrency**: Limit to 3–5 parallel downloads to avoid overload"
--- 

**✅ Step 7: Security & Privacy**  

**You**:  
**"If files are sensitive:**  

- Use AES encryption for local storage  
- Store keys securely in Keychain  
- Respect user privacy and avoid tracking"
--- 

**✅ Step 8: Monitoring & Analytics**  

**You**:  
-   "I’d log download events (start, complete, fail) for analytics.  
-    Use tools like Firebase or custom logging to monitor performance and errors."

![IMG_20250730_222821](https://github.com/user-attachments/assets/2c3bb8d4-c027-4c31-9cd6-e94c7a08653f)

---  

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

```swift
import Foundation

struct DownloadItem: Identifiable {
    let id = UUID()
    let url: URL
    var progress: Float = 0.0
    var isDownloading: Bool = false
    var isCompleted: Bool = false
}
```
```swift
import Foundation
import Combine

class DownloadViewModel: ObservableObject {
    @Published var items: [DownloadItem] = []

    func addDownload(urlString: String) {
        guard let url = URL(string: urlString) else { return }
        let item = DownloadItem(url: url)
        items.append(item)
        DownloadManager.shared.startDownload(from: url)
    }
}
```

```swift
import SwiftUI

struct DownloadListView: View {
    @StateObject private var viewModel = DownloadViewModel()
    @State private var newURL: String = ""

    var body: some View {
        NavigationView {
            VStack {
                HStack {
                    TextField("Enter file URL", text: $newURL)
                        .textFieldStyle(RoundedBorderTextFieldStyle())
                    Button("Download") {
                        viewModel.addDownload(urlString: newURL)
                        newURL = ""
                    }
                }.padding()

                List(viewModel.items) { item in
                    VStack(alignment: .leading) {
                        Text(item.url.lastPathComponent)
                        ProgressView(value: item.progress)
                            .progressViewStyle(LinearProgressViewStyle())
                    }
                }
            }
            .navigationTitle("File Downloader")
        }
    }
}
```

```swift
import SwiftUI

@main
struct FileDownloaderApp: App {
    var body: some Scene {
        WindowGroup {
            DownloadListView()
        }
    }
}
```
```swift
<key>UIBackgroundModes</key>
<array>
    <string>fetch</string>
    <string>remote-notification</string>
    <string>background-fetch</string>
</array>
```
### Why Use Singleton for DownloadManager in iOS

The `DownloadManager` in iOS is commonly implemented as a singleton to ensure centralized control and efficient resource management. Below are the key reasons and considerations for using a singleton pattern in this context.

---

#### ✅ Centralized Download Control

- A singleton ensures that all downloads are managed from a single, shared instance.
- Prevents multiple instances from creating conflicting or duplicate download tasks.
- Simplifies coordination across different parts of the app.

---

#### 🔄 Background Session Reuse

- `URLSession` with a background configuration must be created once and reused.
- Apple recommends maintaining a single session for background downloads to ensure proper handling when the app is suspended or terminated.
- Singleton helps maintain the lifecycle of the session across app states.

---

#### 🔗 Shared State Across Views

- Multiple views (e.g., download list, detail view) can access the same manager instance.
- Enables consistent tracking of download progress and status.
- Simplifies UI updates and state management.

---

#### 🧠 Memory Efficiency

- Avoids creating multiple `URLSession` instances, which can be resource-intensive.
- Keeps active downloads in memory without duplication.
- Reduces overhead and improves performance.

---

#### 🎯 Interview Best Practices

- Demonstrates understanding of global coordination and resource management.
- Singleton is a common pattern for managers like:
  - `LocationManager`
  - `NotificationCenter`
  - `DownloadManager`
- Shows awareness of system-level design considerations.

---

#### ⚠️ When Not to Use Singleton

- When multiple isolated download contexts are needed (e.g., per user profile).
- In architectures that favor testability and dependency injection.
- When using protocols and mock implementations for unit testing.

---

#### ✅ Alternative Approach

- Use protocols and dependency injection to make `DownloadManager` testable.
- Create instances as needed and inject them into view models or services.

---

**Conclusion**: Singleton is a practical and efficient choice for `DownloadManager` in many iOS apps, especially when centralized control and background session reuse are required. However, developers should evaluate the architecture and testing needs before committing to this pattern.

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


  



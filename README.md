
## System Design  


# Example 1:  🍽 Uber Eats System Design  


### Step 1: Below is the final designs  

| Restaurant List                                                                 | Menu                                                                 | Basket                                                                 |
|----------------------------------------------------------------------------------|----------------------------------------------------------------------|------------------------------------------------------------------------|
| <img width="167" height="341" alt="Restaurant List" src="https://github.com/user-attachments/assets/ad19b971-b373-4297-bd15-996d67fcfc2e" /> | <img width="167" height="341" alt="Menu" src="https://github.com/user-attachments/assets/4a1b2dc2-3928-4224-b342-3eead7f355e5" /> | <img width="167" height="341" alt="Basket" src="https://github.com/user-attachments/assets/13355850-399a-4ae2-8e2e-30f2ac5bbc22" /> |  

### Step 2: Problem Description  

<img width="600" height="300" alt="Screenshot 2025-11-04 at 8 47 08 PM" src="https://github.com/user-attachments/assets/0c1d5a96-1f44-40fd-a15a-103b2a98ce8d" />

###  Step 3: 📋 Requirements

#### ✅ Features
- Choose an address  
- View a restaurant list  
- See a restaurant menu  
- Add dishes to the basket  
- Make an order  
- Track order status  
- Track order on a map  
- Ignore payments  

#### ⚙️ Non-functional
- Reasonable app performance  
- Optimize for slow internet  
- Optimize for limited storage on device  
- Don't overload backend


### Step 4: Data Model  

~~~swift
import Foundation

// MARK: - Dish
struct Dish: Identifiable, Codable {
    let dishID: Int
    let restaurantID: Int
    let name: String
    let price: Double
    let imageURL: String

    var id: Int { dishID }
}

// MARK: - Basket
struct Basket: Identifiable, Codable {
    let basketID: Int
    let userID: Int
    let restaurantID: Int
    let selectedDishes: [SelectedDish]

    var id: Int { basketID }
}

// Helper model for selected dishes
struct SelectedDish: Codable {
    let dishID: Int
    let count: Int
}

// MARK: - Order
struct Order: Identifiable, Codable {
    let orderID: Int
    let status: String
    let basket: Basket
    let courierLatitude: Double
    let courierLongitude: Double

    var id: Int { orderID }
}
~~~

### Step 5: REST API

**Restaurants & Dishes**  

- GET /users/<userID> Returns: User  

- GET /restaurants/<addressID> Returns: [Restaurant]  

- GET /dishes/<restaurantID> Returns: [Dish]  

**Basket**  

- POST /basket/{userID, restaurantID, dishID, count} Returns: Basket  

- PATCH /basket/{basketID, dishID, count} Returns: Basket  

- GET /basket/{basketID} Returns: Basket

**Orders**  

- POST /orders/{userID, basketID} Returns: Order  

- GET /orders/{orderID} Returns: Order  

**SSE (Server Sent Events)**  

- GET /live-order-status/{orderID} Returns: event stream

### Step 6: Storage

Metadata  

Core Data  
Realm DB  
Serialize objects  

### Step 7: High Level Design  



<img width="994" height="647" alt="Screenshot 2025-11-04 at 10 06 08 PM" src="https://github.com/user-attachments/assets/cd22d377-3e55-47ef-933b-57dd5433ca87" />  


### Step 8: Create SOLID Principles  

~~~swift
import Foundation

// MARK: - Error Type
enum UberError: Error {
    case networkError(Error)
    case decodingError(Error)
    case invalidResponse
}

// MARK: - Restaurant Model
struct Restaurant: Codable, Identifiable {
    let restaurantID: Int
    let name: String
    let rating: Int
    let imageURL: String
    let address: Address

    var id: Int { restaurantID }
}

struct Address: Codable {
    let addressID: Int
    let label: String
    let city: String
    let street: String
    let flat: String
    let postcode: String
    let latitude: Double
    let longitude: Double
}

// MARK: - Protocol Definition
protocol RestaurantService {
    func getRestaurants(addressID: Int,
                        completion: @escaping (Result<[Restaurant], UberError>) -> Void)
}

// MARK: - Implementation
class RestaurantServiceImpl: RestaurantService {
    func getRestaurants(addressID: Int,
                        completion: @escaping (Result<[Restaurant], UberError>) -> Void) {
        let urlString = "https://api.example.com/restaurants/\(addressID)"
        guard let url = URL(string: urlString) else {
            completion(.failure(.invalidResponse))
            return
        }

        URLSession.shared.dataTask(with: url) { data, response, error in
            if let error = error {
                completion(.failure(.networkError(error)))
                return
            }

            guard let data = data else {
                completion(.failure(.invalidResponse))
                return
            }

            do {
                let restaurants = try JSONDecoder().decode([Restaurant].self, from: data)
                completion(.success(restaurants))
            } catch {
                completion(.failure(.decodingError(error)))
            }
        }.resume()
    }
}
~~~




--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


# Example 2:  🎯 System Design Interview: File Downloader App (Mobile)  

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

**✅ Step 9: Extensibility & Future Scope**  

**You**:  

**"To make the app scalable and extensible, I’d plan for future enhancements like:**  
- Support for cloud sync (e.g., iCloud, Google Drive)  
- File previews within the app (PDF viewer, image viewer)  
- User authentication for secure downloads  
- Download prioritization and scheduling  
- Integration with share sheet and external apps"  
---  

**✅ Step 10: Testing Strategy**  

You:
**"I’d ensure robustness through:**  


- Unit tests for DownloadManager logic  
- UI tests for progress indicators and user interactions  
- Network mocking for offline and error scenarios  
- Stress testing with large files and concurrent downloads"  
---  

**✅ Step 11: Interviewer Engagement**  

You:
**"Would you like me to walk through a Swift implementation of the DownloadManager class or sketch out the architecture diagram?"**  
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


  



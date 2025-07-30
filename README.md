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



  



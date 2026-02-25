# BÍ KÍP PHỎNG VẤN ANDROID DEVELOPER (Dzung19)

## MẸO HỌC KHÔNG CẦN HỌC THUỘC LÒNG 🧠
1. **Kỹ thuật Feynman (Dạy lại cho người khác):** Thử giải thích một khái niệm (VD: Clean Architecture) thành lời nói mộc mạc nhất như đang giải thích cho một người không biết code. Nếu bị vấp, tức là chỗ đó bạn chưa hiểu sâu.
2. **Gắn với thực tế dự án:** - Khi ôn về **XML & Tối ưu UI**, hãy nhớ lại lúc làm `Samsung Messages`.
   - Khi ôn về **Compose, Clean Arch, BLE**, hãy mở lại source code của `Petstory` trên GitHub để xem luồng đi của data.
3. **Trả lời theo công thức STAR:** **S**ituation (Tình huống) -> **T**ask (Nhiệm vụ) -> **A**ction (Hành động của bạn) -> **R**esult (Kết quả).
4. **Hỏi ngược lại "Tại sao?":** Tại sao lại dùng StateFlow mà không dùng LiveData? Tại sao tách Domain layer? Hiểu "Tại sao" sẽ giúp bạn nhớ "Cái gì" và "Như thế nào".

---

## KIẾN THỨC CỐT LÕI (Q&A NGẮN GỌN) 🚀

### 1. Kiến trúc (Architecture)
* **Clean Architecture là gì?** * Chia app thành 3 tầng: Presentation (UI), Domain (Business logic), Data (API/DB). 
    * *Từ khóa:* Độc lập, dễ test, dễ thay đổi UI/Database mà không ảnh hưởng logic lõi.
* **MVVM & ViewModel:**
    * ViewModel giữ data cho UI, sống sót qua Configuration Changes (xoay màn hình).
    * *Lưu ý:* ViewModel không bao giờ được chứa reference đến View (Context, Activity) để tránh Memory Leak.
* **Dependency Injection (Hilt):** * Cung cấp các object (phụ thuộc) từ bên ngoài vào class thay vì class tự tạo ra. Giúp code lỏng lẻo (loose coupling) và dễ viết Unit Test.

### 2. Giao diện (UI)
* **Jetpack Compose:**
    * *Khai báo (Declarative):* Mô tả UI nên trông thế nào dựa trên State hiện tại.
    * *Recomposition:* Tự động vẽ lại những Composable có State bị thay đổi, bỏ qua những cái không đổi (tiết kiệm tài nguyên).
    * *State:* `remember` (giữ state khi recompose), `rememberSaveable` (giữ state khi xoay màn hình).
* **Android XML:**
    * *RecyclerView tối ưu:* Dùng `ViewHolder` để tái sử dụng view, dùng `DiffUtil` để chỉ cập nhật những item thực sự thay đổi thay vì `notifyDataSetChanged()`.

### 3. Ngôn ngữ & Luồng (Kotlin)
* **Coroutines:** Xử lý bất đồng bộ (gọi API, đọc DB) không chặn Main Thread. Nhẹ hơn Thread rất nhiều.
* **Flow / StateFlow:** * Flow: Luồng dữ liệu lạnh (chỉ phát data khi có người thu thập - collect).
    * StateFlow: Luồng dữ liệu nóng (luôn giữ 1 state cuối cùng, thay thế hoàn hảo cho LiveData).

### 4. Hardware & Kết nối (Điểm nhấn của bạn)
* **BLE (Bluetooth Low Energy):**
    * Tiết kiệm pin. Cần nắm rõ 3 bước: Scan (tìm thiết bị) -> Connect (kết nối GATT) -> Discover Services/Characteristics (trao đổi data).
* **RESTful API & Retrofit:** * Giao tiếp với Server qua HTTP methods (GET, POST, PUT, DELETE). Xử lý Exception bằng Coroutine `try/catch`.
* **FCM (Firebase Cloud Messaging):** * Service nhận push notification. Xử lý trong `FirebaseMessagingService`. 

### 5. Lưu trữ (Local Storage)
* **Room vs SharedPreferences:**
    * Room: Wrapper của SQLite, dùng cho data phức tạp, có quan hệ, cần truy vấn (danh sách thú cưng, lịch sử bệnh...).
    * SharedPreferences: Lưu cặp Key-Value đơn giản (Token, Settings).
* **Content Provider:** * Cửa ngõ chia sẻ dữ liệu an toàn giữa các ứng dụng khác nhau (rất cần cho các app hệ thống như Samsung Messages).

### 6. Khác
* **Flutter (BLoC):** BLoC tách UI và Business Logic qua luồng Event (đầu vào) và State (đầu ra). Khá giống luồng MVI/MVVM + StateFlow bên Android Native.
* **Git:** Phân biệt `merge` (gộp nhánh, tạo commit merge) và `rebase` (bứng các commit đắp lên đầu nhánh khác, lịch sử thẳng tắp).

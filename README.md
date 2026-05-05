***

# BÍ KÍP PHỎNG VẤN ANDROID DEVELOPER (Dzung19)

## 1. MẸO HỌC KHÔNG CẦN HỌC THUỘC LÒNG 🧠
* **Kỹ thuật Feynman (Dạy lại cho người khác):** Thử giải thích một khái niệm (VD: Clean Architecture) thành lời nói mộc mạc nhất. Nếu bị vấp, tức là chỗ đó chưa hiểu sâu.
* **Gắn với thực tế dự án:** * Khi ôn về **XML & Tối ưu UI**, hãy nhớ lại lúc làm `Samsung Messages`.
   * Khi ôn về **Compose, Clean Arch, BLE**, hãy mở lại source code của `Petstory` trên GitHub để xem luồng đi của data.
* **Trả lời theo công thức STAR:** **S**ituation (Tình huống) -> **T**ask (Nhiệm vụ) -> **A**ction (Hành động) -> **R**esult (Kết quả).
* **Hỏi ngược lại "Tại sao?":** Tại sao dùng StateFlow mà không dùng LiveData? Hiểu "Tại sao" sẽ giúp bạn nhớ "Cái gì" và "Như thế nào".

---

## 2. KIẾN TRÚC & QUẢN LÝ VÒNG ĐỜI (ARCHITECTURE & LIFECYCLE) 🏛️

### Clean Architecture & DI
* **Clean Architecture:** Chia app thành 3 tầng: Presentation (UI), Domain (Business logic), Data (API/DB). Độc lập, dễ test, dễ thay đổi UI/Database mà không ảnh hưởng logic lõi.
* **Dependency Injection (Hilt):** "Tiêm" phụ thuộc từ ngoài vào thay vì class tự khởi tạo. Giúp code lỏng lẻo (loose coupling).
   * **@Binds & @Qualifier:** Dùng `@Binds` liên kết Interface với Implementation để tối ưu tốc độ build. Dùng `@Qualifier` (VD: `@WireGuardProtocol`) để chỉ đường cho Hilt khi có nhiều class thực thi (Đa hình).
   * **Cạm bẫy Memory Leak:** Chỉ inject `@ApplicationContext` ở mức SingletonComponent. Dùng `@ActivityContext` sai chỗ sẽ ghim chết Activity trong RAM.

### Vòng đời Activity (Như một vở kịch)
* **`onCreate()`:** Dựng rạp, làm bối cảnh (chỉ chạy 1 lần).
* **`onStart()`:** Kéo rèm. Giao diện hiện lên nhưng chưa tương tác được.
* **`onResume()`:** Vở kịch bắt đầu. Người dùng tương tác (Foreground).
* **`onPause()`:** Tạm dừng kịch (có Dialog đè lên hoặc chia đôi màn hình). Vẫn hiển thị nhưng mất quyền tương tác.
* **`onStop()`:** Kéo rèm đóng lại (thoát ra Home). App rơi vào Background.
* **`onDestroy()`:** Dỡ bỏ rạp (vuốt tắt app hoặc bị hệ thống kill).

### Vòng đời ViewModel (Người Đạo diễn)
* **Bản chất sống dai:** Khi xoay màn hình, Activity bị "đập đi xây lại", nhưng ViewModel được lưu an toàn trong két sắt **`ViewModelStore`**. Hệ thống lấy két sắt này giao lại cho Activity mới, giúp Data được giữ nguyên.
* **`onCleared()`:** Nơi dọn dẹp chiến trường. Nó chỉ được gọi khi bấm nút Back thoát hẳn. Tại đây, `viewModelScope` sẽ tự động hủy mọi Coroutine chạy ngầm, chống Memory Leak triệt để.

---

## 3. GIAO DIỆN (UI & JETPACK COMPOSE) 🎨

### Bản chất Jetpack Compose
* **Declarative (Khai báo):** Mô tả UI nên trông thế nào dựa trên State hiện tại.
* **3 Giai đoạn vẽ UI:**
   1. **Composition:** Xác định CÓ những component nào.
   2. **Layout:** Xác định NẰM Ở ĐÂU và TO BAO NHIÊU.
   3. **Draw:** Đổ màu, vẽ pixel.

### Tối ưu Recomposition (Direct Read vs Deferred Read)
* **Đọc trực tiếp (Direct Read):** `val name by viewModel.name.collectAsState()`. Dùng cho giá trị ít thay đổi (Tên user, Trạng thái đăng nhập).
* **Trì hoãn (Deferred Read - Lambda):** BẮT BUỘC dùng cho giá trị thay đổi liên tục (Tọa độ scroll, Tốc độ mạng). Việc bọc state vào Lambda `{ state.value }` giúp Compose **bỏ qua hoàn toàn Giai đoạn 1 (Composition)** hoặc cô lập vùng vẽ, giữ UI mượt mà ở 120fps.

### XML Cũ
* **RecyclerView tối ưu:** Dùng `ViewHolder` tái sử dụng view, dùng `DiffUtil` cập nhật đúng item thay đổi thay vì `notifyDataSetChanged()`.

---

## 4. XỬ LÝ BẤT ĐỒNG BỘ (KOTLIN COROUTINES) ⚡

### Core Concepts (Bản chất hệ thống)
* **Coroutines vs Threads:** Coroutines là "luồng siêu nhẹ" do Kotlin quản lý, không thay thế Thread mà chạy trên Thread. Nổi bật với tính chất **Non-blocking (Không chặn luồng)**.
* **Suspend Function:** Hàm có khả năng tạm dừng và tiếp tục. Chỉ được gọi trong Coroutine hoặc một suspend function khác.
* **Dispatchers Nâng Cao:**
   * `Dispatchers.Main`: Đẩy tác vụ xếp hàng chờ. Gây trễ 1 frame (flicker).
   * `Dispatchers.Main.immediate`: Nếu đang ở Main Thread thì chạy đồng bộ ngay lập tức. Cực kỳ tối ưu cho Jetpack Compose.
   * `Dispatchers.IO` (Mạng/DB) & `Dispatchers.Default` (Tính toán nặng).

### Builders & Concurrency (Đồng thời)
* **`launch` vs `async`:** `launch` bắn rồi quên, lỗi văng ra ngoài. `async` dùng khi cần trả kết quả, lỗi được gói trong `Deferred` và chỉ nổ khi gọi `.await()`.
* **`withContext`:** Chuyển luồng **Tuần tự (Sequential)**. Dùng để bọc các hàm gọi API/DB ở Repository, ép chạy luồng nền, giúp an toàn tuyệt đối cho Main Thread.
* **Chạy song song:** Để tiết kiệm thời gian, bọc các lệnh `async` vào trong một `coroutineScope`.

### Quản lý lỗi & Hủy bỏ (Cancellation)
* **Cooperative Cancellation (Hủy hợp tác):** Lệnh `job.cancel()` chỉ bật cờ `isActive = false`. Với vòng lặp nặng, phải kiểm tra `while(isActive)` hoặc `yield()` để Coroutine chịu dừng, tránh Memory Leak.
* **Chết chùm vs Độc lập:** * `coroutineScope` (Job): Tính hủy bỏ 2 chiều. Một API lỗi kéo theo cái đang chạy bị hủy ngang.
   * `supervisorScope` (SupervisorJob): Lỗi 1 chiều. Con chết không làm sập cha và anh em. (Ví dụ: `viewModelScope` mặc định dùng SupervisorJob).
* **CoroutineExceptionHandler (CEH):** Bắt lỗi không xác định (chỉ hoạt động với `launch` ở Root Coroutine).
### Cơ chế của lệnh `join()`: Coroutines vs Threads
*   **Java Thread `join()` (Blocking):** Chặn đứng (Block) luồng hiện tại. Luồng phải đứng im chờ đợi đến khi thread kia làm xong, không thể làm việc khác.
*   **Coroutine `join()` (Suspending):** Dùng cơ chế "Ve sầu thoát xác". Nó tạm cất trạng thái hiện tại (Suspend) và **giải phóng Thread**. Thread lập tức được rảnh tay để chạy các Coroutine khác. Khi tác vụ xong, Coroutine được đánh thức (Resume) và chạy tiếp.
---

## 5. LUỒNG DỮ LIỆU THỰC CHIẾN (KOTLIN FLOW) 🌊

### Flow vs StateFlow vs SharedFlow
* **Cold Flow:** Khởi động lại từ đầu với mỗi Collector mới.
* **Hot Flow (StateFlow):** Nắm giữ trạng thái. Luôn giữ 1 giá trị mới nhất, BẮT BUỘC có giá trị khởi tạo. Dùng cho UI State (Danh sách, chữ, loading).
* **Hot Flow (SharedFlow):** Bắn sự kiện 1 lần. KHÔNG lưu trạng thái cũ, KHÔNG cần giá trị khởi tạo. Dùng bắn Toast, Navigation. Tránh lỗi hiển thị lại thông báo (replay) vô lý khi xoay màn hình.

### Vòng đời của StateFlow
Sống dựa vào Scope mà nó được khởi tạo. Để an toàn, chuyển Cold Flow thành StateFlow bằng `.stateIn()`, kết hợp `SharingStarted.WhileSubscribed(5000)` để giữ cache 5s chống gọi lại mạng khi xoay thiết bị, tự động ngủ khi user ẩn app.

### `SharedFlow` & `StateFlow`
`StateFlow` thực chất là một `SharedFlow` được tối ưu hóa. Công thức chế tạo:
**`StateFlow` = `SharedFlow(replay=1, drop_oldest)` + Bắt buộc có Initial Value + Tự động `distinctUntilChanged()` (chống trùng lặp) + Thuộc tính `.value`.**

**Các tham số kiểm soát Buffer của `SharedFlow`:**
1.  `replay`: Số lượng dữ liệu cũ lưu lại để phát ngay cho Collector **mới tham gia**.
2.  `extraBufferCapacity`: Số "ghế chờ" dự phòng để lưu dữ liệu khi Collector đang bận xử lý không kịp. Giúp Emitter không bị block. Tổng bộ đệm = `replay + extraBufferCapacity`.
3.  `onBufferOverflow`: Chiến lược xử lý khi bộ đệm đầy (`SUSPEND` - đứng chờ, `DROP_OLDEST` - xóa cái cũ nhất, `DROP_LATEST` - bỏ cái mới).

### Flow Operators (Toán tử ăn điểm)
* **`debounce(300ms)`:** Trì hoãn gõ phím, chống spam API.
* **`flatMapLatest`:** Hủy request cũ ngay khi có request mới (chống Race Condition).
* **`collectAsStateWithLifecycle()`:** Tự động ngừng collect khi app rơi vào Background để tiết kiệm pin/mạng.
* **`conflate()`:** Xử lý **Backpressure**. Drop các giá trị cũ khi UI đang bận vẽ, chỉ lấy giá trị mới nhất (tốt cho biểu đồ tốc độ mạng).
* **`combine` vs `zip`:** `combine` gộp ngay giá trị mới nhất của luồng này với luồng kia. `zip` bắt phải đứng chờ 1-1 gây mất data.
* **Exception Transparency:** Dùng `flowOn(Dispatchers.IO)` đổi luồng Upstream. Lỗi bắt bằng `.catch { }` nằm dưới cùng, nhường logic xử lý cho `.onEach { }`.

---

## 6. NETWORKING & VPN DEEP DIVE 🌐

* **TCP vs UDP:** TCP tin cậy, chậm, dùng bắt tay 3 bước. UDP tốc độ cao, không bắt tay, phù hợp stream/VPN.
* **TLS & Diffie-Hellman (ECDH):** TLS kết hợp mã hóa bất đối xứng (trao đổi khóa bằng thuật toán toán học Diffie-Hellman mà không gửi chìa khóa qua mạng) và đối xứng (truyền dữ liệu bằng AES).
* **WireGuard & DNS Leak:** WireGuard dùng **Cryptokey Routing** và chạy trên UDP không cần bắt tay. DNS Leak là lỗi rò rỉ truy vấn tên miền ra ngoài đường ống mã hóa.
* **Tối ưu Doze Mode cho VPN:** Kotlin Coroutines ping ngầm sẽ bị OS kill. Giải pháp: Cấu hình `PersistentKeepalive` để tầng Native (C++/Go) tự ping bằng tài nguyên cực thấp.

---

## 7. LƯU TRỮ, BACKGROUND & FCM 💾

* **Room vs SharedPreferences:** Room cho data phức tạp. SharedPrefs cho Key-Value. Room lưu file vật lý tại `/data/data/<package_name>/databases/` (Sandbox).
* **SharedPrefs (`apply` vs `commit`):** Luôn dùng **`apply()`** (ghi bất đồng bộ) để chống ANR. Chỉ dùng `commit()` khi bắt buộc phải chờ kết quả ghi ổ cứng (và phải gọi trên luồng nền).
* **FCM (Firebase Cloud Messaging):** * Bị đổi Token khi: Clear Data, Cài lại app. Cập nhật ở `onNewToken()`.
   * App bị Kill: Hệ thống tự hiện Push. Phải lấy payload data bằng `intent.extras` ở `onCreate()`.
* **Giữ kết nối ngầm (Background/Doze):** BẮT BUỘC dùng **Foreground Service** kèm Notification cứng. Android 14+ phải khai báo type `connectedDevice`.

---

## 8. PHẦN CỨNG (BLE - BLUETOOTH LOW ENERGY) 🔵

* **Kiến trúc GATT:** * **Service:** Ngăn chứa nhóm chức năng.
   * **Characteristic:** Nơi chứa data thực tế để Đọc/Ghi/Lắng nghe (Notify).
* **Cạm bẫy cực hiểm:**
   * **Callback Thread:** Mọi thứ trả về ở `BluetoothGattCallback` đều nằm ở luồng ngầm (Binder Thread), cập nhật UI trực tiếp sẽ gây Crash.
   * **Sequential Write:** HĐH Android không có bộ xếp hàng (Queue). Gọi 2 lệnh Ghi liên tiếp mà chưa chờ xong lệnh 1 sẽ bị drop lệnh 2.
   * **Android 13+:** Bắt buộc dùng cú pháp `writeCharacteristic` mới truyền trực tiếp mảng byte thay vì `characteristic.value`.

---

## 9. KOTLIN NÂNG CAO (UNDER THE HOOD) 🧙

* **Type Erasure (`inline` + `reified`):** Dùng `reified T` để chống lại việc máy ảo Java xóa mất kiểu Generic lúc chạy. Giúp parse JSON sạch đẹp.
* **Value Class (Zero-cost Abstraction):** Dùng `@JvmInline value class`. An toàn kiểu dữ liệu lúc code, nhưng khi build ra mã máy bị xóa lớp vỏ, tối ưu hoàn toàn trên RAM.
* **Generic Covariance (`<out T>`):** Xây dựng "Phễu API" (`Result<out T>`) dùng chung để loại bỏ `try-catch` lặp lại.

## 10. ANDROID SYSTEM APP & AOSP CORE

### 1. Phân loại System App
*   **`/system/app/` (Standard System App):** Ứng dụng hệ thống thường. Không gỡ cài đặt được, quyền hạn ngang app bình thường. Không cần file XML Whitelist.
*   **`/system/priv-app/` (Privileged System App):** Ứng dụng đặc quyền (VD: Danh bạ, Tin nhắn gốc). Có quyền can thiệp phần cứng, hệ thống. **Bắt buộc** phải có file XML Whitelist cấp quyền đặt ở `/etc/permissions/` nếu không sẽ bị bootloop (treo logo).

### 2. Cấu hình `Android.bp` để build System App
**Cờ (Flags) quan trọng:**
*   `certificate: "platform"`: Ký bằng chìa khóa gốc của hệ thống.
*   `privileged: true`: Đẩy app vào `priv-app`. Nếu không có, mặc định vào `app`.
*   `platform_apis: true`: Cho phép ứng dụng dùng các API `@hide` (ẩn) của Framework.
*   `required: ["tên_module_xml"]`: Dùng để đính kèm file XML Whitelist (nếu là priv-app).

### 3. Bài toán bảo mật: Lấy số IMEI
*   **Dưới Android 10:** Chỉ cần xin quyền `READ_PHONE_STATE` và gọi `TelephonyManager`.
*   **Từ Android 10+:** Google cấm app thường lấy IMEI (văng `SecurityException`).
*   **Cách lách luật (Bypass):** Chỉ lấy được khi app là Privileged System App, Device Owner (MDM), hoặc được user set làm App gọi điện mặc định (Default Dialer).
*   **Giải pháp thay thế cho App thường:** Dùng `ANDROID_ID`, Firebase Installation ID, hoặc tự sinh UUID.

### 4. Bí kíp Phỏng vấn AOSP Framework
Nhà tuyển dụng sẽ xoáy sâu vào cách HĐH quản lý ứng dụng:
1.  **Boot Process & Zygote:** Tiến trình `Zygote` dùng cơ chế `fork()` để nhân bản process mới cho App, giúp share bộ nhớ và khởi động nhanh.
2.  **Binder IPC:** Cơ chế giao tiếp liên tiến trình cốt lõi. Ưu việt nhờ cơ chế 1-copy process (truyền data qua Memory Mapping `mmap`). Hiểu AIDL, Proxy (Client) và Stub (Server).
3.  **System Services (Bộ 3 quyền lực):**
    *   *AMS (Activity Manager Service):* Quản lý Lifecycle, cấp phát RAM, OOM Killer.
    *   *WMS (Window Manager Service):* Quản lý tọa độ, Z-index, WindowToken.
    *   *PMS (Package Manager Service):* Quản lý cài đặt APK, cấp quyền Permissions.
4.  **Cơ chế Render UI:** Choreographer, VSYNC, luồng Measure -> Layout -> Draw.

---

## 11. THUẬT TOÁN & BUILD TOOLS 🛠️

* **Thuật toán cốt lõi:**
   * **Tìm kiếm:** Trie (Cây tiền tố, Autocomplete), A* (Đường đi ngắn nhất có heuristic), Hashing (Bảng băm $O(1)$).
   * **Sắp xếp:** Quick Sort (Trung bình $O(n \log n)$), Tim Sort (Lai tạp siêu tối ưu, là mặc định của Java/Kotlin).
* **ProGuard / R8:**
   * Cơ chế: Shrinking (Xóa code thừa), Obfuscation (Đổi tên a,b,c chống dịch ngược), Optimization (Tối ưu bytecode).
   * **Lưu ý:** Gắn `@Keep` cho Data Models để không bị xóa mất biến khi dùng Gson/Moshi parse dữ liệu mạng.

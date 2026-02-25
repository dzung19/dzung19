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

### 3. Luồng dữ liệu (State & Flow)
* **Phân biệt State - StateFlow - SharedFlow:**
    * **State (Compose):** Giữ giá trị tại UI. Khi đổi giá trị -> báo UI vẽ lại (Recomposition). *Ví dụ: Công tắc đèn.*
    * **StateFlow:** Luôn giữ 1 trạng thái mới nhất, BẮT BUỘC có giá trị khởi tạo. Dùng để chứa data hiển thị (Danh sách, chữ, loading). *Ví dụ: Bảng tỷ số bóng đá (luôn có số, ai nhìn lên là thấy tỷ số mới nhất).*
    * **SharedFlow:** Bắn sự kiện 1 lần (One-time event), KHÔNG có giá trị khởi tạo, KHÔNG lưu trạng thái cũ. Dùng để bắn Toast, chuyển màn hình, báo lỗi mạng. *Ví dụ: Tiếng loa thông báo ở sân bay (nghe xong là bay theo gió, người đến sau không nghe được nữa).*
    * *Tip ăn điểm:* Dùng SharedFlow bắn lỗi mạng để tránh xoay màn hình (recompose) bị bắn lỗi 2 lần.

* **Toán tử Flow (Search API thực tế):**
    * **`debounce(300ms)`:** Trì hoãn. Chờ user gõ xong (ngừng gõ 300ms) mới cho luồng chạy tiếp -> Chống spam gọi API rác lên Server.
    * **`flatMapLatest`:** Nếu có request mới (chữ "CHO"), lập tức HỦY request cũ (chữ "C") -> Tránh lỗi Race Condition (mạng lag làm data cũ đè lên data mới).
    * **`collectAsStateWithLifecycle()`:** Dùng trong Compose để hứng Flow. Siêu năng lực của nó là tự động ngừng collect khi app rơi vào Background (chưa tới `STARTED`) để tiết kiệm pin/mạng, không cần tự code `onPause` rườm rà.
 
### 4. Xử lý Bất đồng bộ (Coroutines & withContext)
* **Coroutines vs Threads:**
    * Thread (Luồng) do Hệ điều hành quản lý, rất nặng. Tạo nhiều Thread sẽ tràn RAM.
    * Coroutine cực kỳ nhẹ (Lightweight thread), do Kotlin quản lý. Có thể chạy hàng ngàn Coroutine trên 1 Thread duy nhất mà không tốn tài nguyên. Tính chất quan trọng nhất là **Non-blocking (Không chặn luồng)**.

* **Các loại Dispatchers (Nơi phân công công việc):**
    * `Dispatchers.Main`: Chuyên làm việc với UI (Cập nhật text, list, hiện Toast).
    * `Dispatchers.IO`: Chuyên ra vào ổ cứng và mạng (Gọi API, đọc/ghi Room DB, SharePreferences).
    * `Dispatchers.Default`: Chuyên tính toán nặng tốn CPU (Lọc mảng dữ liệu khổng lồ, xử lý ảnh, mã hóa).

* **Hàm `withContext` (Bộ đàm chuyển luồng thần thánh):**
    * **Bản chất:** Là một `suspend function` (hàm tạm ngưng). Nó giúp ta **chuyển đổi luồng (switch context)** ngay giữa chừng mà không cần dùng callback lồng nhau (Callback Hell).
    * Nó tạm ngưng coroutine hiện tại, vứt cục code bên trong sang một luồng khác chạy, chạy xong thì trả kết quả về đúng cái luồng ban đầu.
    * **Câu trả lời thực chiến (Ăn điểm tối đa):** *"Trong project Petstory của em, khi gọi API hoặc query Room DB ở tầng Data/Repository, em luôn bọc nó trong `withContext(Dispatchers.IO)` để ép nó chạy ở luồng nền. Nhờ vậy, khi kết quả trả về tầng ViewModel (đang chạy ở `viewModelScope` thuộc Main Thread), em có thể lấy data đó gán thẳng vào StateFlow để cập nhật UI ngay lập tức mà không bao giờ sợ ứng dụng bị crash do lỗi 'Chạm vào UI từ luồng nền'."*

### 5. Hardware & Background (Điểm "ăn tiền" của Petstory)
* **BLE (Bluetooth Low Energy):**
    * 3 bước cơ bản: Scan -> Connect (GATT) -> Discover Services/Characteristics.
* **Giữ kết nối ngầm không bị Android "kill" (Doze Mode):**
    * BẮT BUỘC dùng **Foreground Service**.
    * BẮT BUỘC phải đính kèm một **Notification cố định** để báo cho người dùng biết app đang chạy ngầm.
    * *Tip update Android 14+:* Phải khai báo thuộc tính `foregroundServiceType="connectedDevice"` trong `AndroidManifest.xml` để hệ điều hành cấp phép duy trì kết nối với vòng cổ thú cưng.

### 6. Lưu trữ (Local Storage)
* **Room / SQLite lưu data vào đâu?**
    * Lưu thành file vật lý tại thư mục đóng kín (Sandbox): `/data/data/<package_name>/databases/`.
    * Các app khác không thể đọc được. Muốn chia sẻ data an toàn ra ngoài (như app nhắn tin), phải dùng **Content Provider**.
* **Room vs SharedPreferences:**
    * Room: Data phức tạp, có quan hệ, cần truy vấn.
    * SharedPreferences: Cặp Key-Value đơn giản (Token, Settings).
* **SharedPreferences: `apply()` vs `commit()` (Cực kỳ quan trọng)**
    * **`apply()` (Khuyên dùng 99%):** Lưu **BẤT ĐỒNG BỘ** (Asynchronous). Nó cập nhật dữ liệu ngay lập tức vào bộ nhớ RAM, rồi âm thầm ghi xuống ổ cứng ở dưới background. Không trả về kết quả. Rất an toàn, không làm đơ Main Thread (UI).
    * **`commit()`:** Lưu **ĐỒNG BỘ** (Synchronous). Nó ghi thẳng xuống ổ cứng và bắt Main Thread phải đứng chờ ghi xong mới được chạy dòng code tiếp theo. Trả về `true`/`false`. 
    * *Câu trả lời ăn điểm:* "Em luôn dùng `apply()` để tránh lỗi ANR (App Not Responding). Chỉ khi nào em có một logic nghiệp vụ cực kỳ khắt khe, bắt buộc phải biết chắc chắn dữ liệu đã lưu thành công xuống ổ cứng hay chưa thì em mới dùng `commit()`, và khi đó em sẽ ném nó vào luồng Coroutines (Background thread) chứ tuyệt đối không gọi trên Main Thread."

### 7. Kiến trúc (Architecture)
* **Clean Architecture là gì?** Chia app thành 3 tầng: Presentation (UI), Domain (Business logic), Data (API/DB). Giúp code độc lập, dễ test.
* **MVVM & ViewModel:** ViewModel giữ data sống sót qua Configuration Changes (xoay màn hình). Tuyệt đối không chứa reference đến View (Context) để tránh Memory Leak.
* **Dependency Injection (Hilt):** "Tiêm" phụ thuộc từ ngoài vào thay vì class tự khởi tạo. Giúp code lỏng lẻo (loose coupling) và dễ viết Unit Test.
### 8. Vòng đời (Lifecycle) - Activity & ViewModel
* **A. Vòng đời Activity (Giống như một Vở kịch)** 
    * **`onCreate()`:** Dựng rạp, làm bối cảnh (Chỉ chạy 1 lần khi mở app). Chỗ này thường để `setContentView` hoặc gọi Jetpack Compose.
    * **`onStart()`:** Kéo rèm lên. Khán giả (người dùng) bắt đầu nhìn thấy sân khấu, nhưng diễn viên chưa diễn (UI hiện lên nhưng chưa tương tác được).
    * **`onResume()`:** Vở kịch chính thức bắt đầu! Người dùng có thể bấm nút, vuốt màn hình. App đang ở trạng thái **Foreground (hiện diện hoàn toàn)**.
    * **`onPause()`:** Tạm dừng kịch. Xảy ra khi có một Dialog xin quyền (Permission) hiện lên đè một góc màn hình, hoặc chia đôi màn hình. App vẫn *hiển thị một phần* nhưng *mất quyền tương tác*.
    * **`onStop()`:** Kéo rèm đóng lại. Xảy ra khi bạn bấm nút Home thoát ra ngoài hoặc mở app khác. App bị che khuất hoàn toàn (rơi vào **Background**).
    * **`onDestroy()`:** Dỡ bỏ rạp kịch. Xảy ra khi bạn vuốt tắt app từ màn hình đa nhiệm, hoặc hệ thống thiếu RAM nên tự "kill" app.

* **B. Vòng đời ViewModel (Người Đạo diễn)** 
    * **Bản chất:** Đạo diễn thì sống dai hơn bối cảnh sân khấu! 
    * **Tình huống xoay màn hình (Configuration Change):** Khi người dùng xoay ngang điện thoại, Android sẽ "đập đi xây lại" giao diện -> Tức là Activity sẽ chạy `onPause -> onStop -> onDestroy` rồi lập tức chạy lại `onCreate -> onStart -> onResume`. 
    * Nếu bạn lưu dữ liệu ở Activity, lúc xoay màn hình data sẽ biến mất (ví dụ: đang gõ form đăng ký dở bị mất sạch chữ). 
    * **Sức mạnh của ViewModel:** Nó nằm ngoài vòng xoáy sinh tử đó. Xoay màn hình, Activity bị hủy nhưng ViewModel **VẪN SỐNG**. Data giữ nguyên. Nó chỉ thực sự chết (gọi hàm `onCleared()`) khi bạn bấm nút Back để **thoát hẳn** ứng dụng.
    * **Hàm `onCleared()`:** Nơi dọn dẹp "chiến trường". Câu trả lời ghi điểm: *"Trong ViewModel, em thường dùng `viewModelScope`. Khi ViewModel bị hủy (app tắt hẳn), `onCleared()` được gọi, `viewModelScope` tự động hủy mọi Coroutine đang chạy ngầm gọi API, giúp app của em không bao giờ bị rò rỉ bộ nhớ (Memory Leak)."*

### 9. Khác
* **RESTful API:** Giao tiếp qua HTTP methods (GET, POST...). Xử lý lỗi bằng Coroutine `try/catch`.
* **Flutter (BLoC):** BLoC tách UI và Logic qua luồng Event (vào) và State (ra). Rất giống MVVM + StateFlow.
* **Git:** Phân biệt `merge` (tạo commit gộp) và `rebase` (đắp commit lên đầu nhánh, lịch sử thẳng tắp).

### . Vòng đời của StateFlow thực chất ăn theo cái gì?
**Câu trả lời chuẩn 10 điểm:** StateFlow phụ thuộc hoàn toàn vào **`CoroutineScope`** mà nó được khởi tạo hoặc được collect (thu thập). Cụ thể chia làm 3 trường hợp:

* **Trường hợp 1: Sống theo Đạo diễn (ViewModel)**
    * Nếu bạn tạo StateFlow trong ViewModel (dùng `viewModelScope`), nó sẽ sống dai như ViewModel. Xoay màn hình nó không chết. Nó chỉ bị hủy khi ViewModel gọi hàm `onCleared()` (tức là khi thoát hẳn màn hình đó).

* **Trường hợp 2: Sống theo Sân khấu (UI / Jetpack Compose)**
    * Khi bạn mang cái StateFlow đó ra ngoài giao diện để hiển thị (bằng hàm `collectAsStateWithLifecycle()`), thì quá trình "lắng nghe" (collect) lại ăn theo vòng đời của View (Activity/Fragment).
    * Sân khấu kéo rèm (`onStop`) -> Dừng collect để tiết kiệm pin. Sân khấu sáng đèn lại (`onStart`) -> Tiếp tục collect.

* **Trường hợp 3: Sống theo Nhà hát (Toàn bộ App)**
    * Nếu bạn để StateFlow ở một class Singleton (như `UserRepository` hay `BluetoothManager` trong app Petstory) và chạy nó bằng một Scope toàn cục (như `GlobalScope` hoặc scope tự tạo ở tầng Application), thì nó sẽ sống bằng đúng tuổi thọ của ứng dụng. Bấm nút Back thoát app ra ngoài thì nó mới chết.

**💡 Từ khóa ghi điểm (Senior Tip):**
"Dạ, khi em biến một Flow bình thường thành StateFlow bằng hàm `.stateIn()`, hàm này bắt buộc em phải truyền vào một cái `scope`. Em truyền `viewModelScope` vào đó, nên StateFlow của em sẽ ăn theo vòng đời của ViewModel ạ. Ngoài ra em hay dùng thuộc tính `SharingStarted.WhileSubscribed(5000)` để quy định rằng: Nếu người dùng ẩn app đi quá 5 giây (không còn ai collect), StateFlow sẽ tự động ngủ để tiết kiệm RAM."

# NHIỆM VỤ 07: TRA CỨU VÀ THEO DÕI TRẠNG THÁI ĐƠN HÀNG (ORDER TRACKING & AUDIT LOG)

---

## 1. TỔNG QUAN NHIỆM VỤ (OVERVIEW)
- **Mục tiêu**: Xây dựng hệ thống lưu trữ, tra cứu thông tin đơn hàng tức thời dựa vào mã đơn (Order ID), đồng thời ghi vết toàn bộ lịch sử thay đổi trạng thái của đơn hàng theo dòng thời gian (từ lúc tạo đơn, gán tài xế, bốc dỡ lên xe, đang giao, cho đến khi hoàn tất hoặc thất bại).
- **Yêu cầu cốt lõi**:
  - Tự cài đặt hai cấu trúc dữ liệu cốt lõi từ đầu (không dùng cấu trúc dựng sẵn):
    1. **Bảng băm (Hash Table)**: Tra cứu nhanh đơn hàng theo khóa `order_id` với thời gian kỳ vọng $O(1)$. Tự cài đặt hàm băm (Hash function) và cơ chế giải quyết xung đột (Collision resolution: Chaining hoặc Open Addressing).
    2. **Danh sách liên kết (Linked List)**: Lưu chuỗi lịch sử trạng thái của từng đơn hàng theo thứ tự thời gian tuyến tính.
  - Phân tích chi tiết hệ số tải (Load factor), chi phí tái băm (Rehashing) và hiệu quả truy xuất.

---

## 2. VAI TRÒ TRONG DỰ ÁN VÀ BÁO CÁO CHUNG
- **"Cơ sở dữ liệu tại chỗ" (In-Memory Database & Audit Trail)**: Tất cả các module khác trong hệ thống (`order_queue.py`, `shipper_assign.py`, `truck_loading.py`) đều tương tác với module này để cập nhật trạng thái đơn hàng khi quy trình chuyển bước.
- **Trải nghiệm người dùng cuối (Customer & Admin View)**: Cho phép khách hàng hoặc nhân viên tổng đài tra cứu ngay lập tức: "Đơn hàng của tôi đang ở đâu, shipper nào phụ trách, lúc 09:30 đang ở trạng thái nào?".
- **Trong báo cáo LaTeX**:
  - Trình bày tại Chương 2 (Cấu trúc bảng băm và danh sách liên kết).
  - Phân tích toán học về hàm băm đồng đều (Uniform Hashing), xác suất đụng độ và độ phức tạp trung bình $O(1)$ so với $O(n)$ của tìm kiếm tuần tự hoặc $O(\log n)$ của tìm kiếm nhị phân.

---

## 3. CẤU TRÚC THƯ MỤC VÀ LIÊN KẾT FILE
```text
project-root/
│
├── order_tracking.py       # File mã nguồn chính chứa Hash Table & Status Linked List
├── Docs/
│   ├── task.md
│   ├── task_7_order_tracking.md
│   └── ...
└── test.py                 # Benchmark tra cứu 100.000 đơn hàng và kiểm tra va chạm hàm băm
```

---

## 4. CÔNG NGHỆ, PHƯƠNG PHÁP & SO SÁNH ĐÁNH GIÁ

### 4.1. Các phương pháp được lựa chọn
1. **Bảng băm giải quyết xung đột bằng phương pháp nối kết (Separate Chaining)**:
   - Sử dụng một mảng kích thước $M$ gồm các danh sách liên kết (Buckets).
   - Hàm băm: Đổi chuỗi ký tự `order_id` thành số nguyên thông qua thuật toán đa thức (Polynomial Rolling Hash: $hash = \sum s[i] \cdot p^i \pmod M$) rồi lấy phần dư với kích thước bảng $M$.
   - Tự động co giãn (Dynamic Resizing / Rehashing) khi hệ số tải $\alpha = \frac{N}{M} \ge 0.75$.
   - Chi phí tra cứu trung bình: $O(1)$.
2. **Lịch sử trạng thái bằng Danh sách liên kết đơn (Singly Linked List with Tail Pointer)**:
   - Mỗi đơn hàng sở hữu một danh sách các mốc sự kiện (`StatusEventNode`).
   - Thêm trạng thái mới vào cuối: $O(1)$ nhờ con trỏ `tail`.
   - Giữ nguyên thứ tự thời gian bất biến (Immutable timeline).

### 4.2. So sánh với các phương pháp khác & Lý do không chọn
| Cấu trúc dữ liệu | Tra cứu theo ID | Thêm bản ghi mới | Lưu chuỗi sự kiện | Lý do không chọn |
| :--- | :--- | :--- | :--- | :--- |
| **Mảng / Danh sách tuyến tính** | $O(N)$ | $O(1)$ | Khá tốt | Tra cứu quá chậm. Khi có 1 triệu đơn hàng, mỗi lần khách hàng kiểm tra mã vận đơn phải quét qua hàng trăm nghìn phần tử. |
| **Cây tìm kiếm nhị phân (BST / AVL)** | $O(\log N)$ | $O(\log N)$ | Trung bình | $O(\log N)$ vẫn chậm hơn $O(1)$ của Bảng băm; thao tác cân bằng cây phức tạp không cần thiết khi chỉ tra cứu theo khóa đẳng thức (`key == target`). |
| **Bảng băm dò tuyến tính (Linear Probing)** | $O(1)$ | $O(1)$ | Không lưu chuỗi sự kiện | Dễ bị hiện tượng gom cụm chính (Primary Clustering); khi xóa đơn hàng phải đánh dấu lười (Tombstone) rất phức tạp. Separate Chaining trong sáng và dễ mở rộng hơn. |
| **Kiểu `dict` sẵn có của Python** | $O(1)$ | $O(1)$ | - | Vi phạm yêu cầu bắt buộc: tự cài đặt các cấu trúc dữ liệu chính, không dùng thư viện sẵn có của đề cương. |

---

## 5. CÁC THUẬT TOÁN & CẤU TRÚC DỮ LIỆU CẦN CÀI ĐẶT
1. **Hàm băm chuỗi (Polynomial Hash Function)**:
   - Chuyển chuỗi `order_id` (ví dụ `"ORD_98124"`) thành chỉ số bucket an toàn, phân bố đồng đều, tránh đụng độ.
2. **Cơ chế Rehashing tự động**:
   - Khi số phần tử vượt ngưỡng $0.75 \times M$, nhân đôi kích thước mảng ($M' = 2M$) và phân phối lại toàn bộ các phần tử.
3. **Cấu trúc Lịch sử sự kiện (`StatusHistoryLinkedList`)**:
   - Lưu trữ các sự kiện gồm: `(timestamp, status, note, updated_by)`.
   - Các trạng thái tiêu chuẩn: `CREATED` $\rightarrow$ `QUEUED` $\rightarrow$ `ASSIGNED` $\rightarrow$ `LOADED` $\rightarrow$ `IN_TRANSIT` $\rightarrow$ `DELIVERED` / `FAILED`.

---

## 6. CHI TIẾT THIẾT KẾ FILE VÀ CÁC HÀM TRONG `order_tracking.py`

### 6.1. Cấu trúc lưu vết lịch sử
- `class StatusNode`:
  - `timestamp`: Thời điểm ghi nhận.
  - `status`: Tên trạng thái.
  - `note`: Ghi chú kèm theo (ví dụ: "Đã giao cho tài xế Nguyễn Văn A").
  - `next`: Con trỏ tới trạng thái kế tiếp.
- `class StatusHistory`:
  - `append_status(status, note)`: Thêm một trạng thái mới vào đuôi danh sách. $O(1)$.
  - `get_full_history()`: Duyệt từ đầu đến cuối trả về toàn bộ dòng thời gian của đơn hàng. $O(K)$ với $K$ là số lần đổi trạng thái.

### 6.2. Cấu trúc Bảng băm tự cài đặt
- `class HashNode`:
  - `key`: Mã đơn (`order_id`).
  - `value`: Đối tượng Đơn hàng (`Order` chứa thông tin chi tiết và `StatusHistory`).
  - `next`: Con trỏ liên kết xử lý va chạm (Chaining).
- `class CustomHashTable`:
  - `__init__(self, initial_capacity=16)`: Khởi tạo mảng buckets với dung lượng ban đầu.
  - `_hash(self, key)`: Tính toán chỉ số băm: `index = hash_code % capacity`. $O(L)$ với $L$ là độ dài chuỗi khóa.
  - `put(self, key, value)`: Thêm hoặc cập nhật đơn hàng theo khóa. $O(1)$ kỳ vọng.
  - `get(self, key)`: Tra cứu đơn hàng theo khóa. $O(1)$ kỳ vọng, $O(N)$ trường hợp xấu nhất nếu tất cả va chạm vào 1 ô.
  - `remove(self, key)`: Xóa đơn hàng khỏi bảng băm. $O(1)$ kỳ vọng.
  - `_resize(self, new_capacity)`: Tự động dời mảng và tái băm khi hệ số tải vượt ngưỡng. $O(N)$.

### 6.3. API Quản lý theo dõi đơn hàng
- `class OrderTrackingSystem`:
  - `create_order(order_id, customer_info, ...)`: Tạo mới và lưu vào bảng băm.
  - `update_order_status(order_id, new_status, note)`: Tra cứu đơn $O(1)$ rồi thêm trạng thái vào danh sách liên kết $O(1)$.
  - `track_order(order_id)`: Trả về trạng thái hiện tại và toàn bộ lịch sử di chuyển của đơn hàng.

---

## 7. KẾT QUẢ ĐẦU RA CẦN ĐẠT (DELIVERABLES)
1. **Mã nguồn `order_tracking.py`**:
   - Cài đặt hoàn chỉnh Bảng băm và Danh sách liên kết không dùng thư viện ngoài.
   - Hàm in lịch sử trạng thái trực quan dạng Timeline console.
2. **Kiểm thử và đánh giá hiệu năng**:
   - Thử nghiệm băm với $100.000$ mã đơn ngẫu nhiên: thống kê số lượng va chạm (Collisions) ở mỗi bucket để chứng minh tính phân bố đều của hàm băm.
   - So sánh thời gian tìm kiếm giữa `CustomHashTable` và mảng duyệt tuyến tính.
3. **Báo cáo LaTeX**:
   - Phân tích toán học về va chạm băm và hệ số tải $\alpha$.
   - Sơ đồ trực quan cấu trúc dữ liệu kết hợp giữa Bảng băm và Danh sách liên kết.

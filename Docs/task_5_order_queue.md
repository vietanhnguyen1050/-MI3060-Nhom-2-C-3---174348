# NHIỆM VỤ 05: HÀNG ĐỢI ĐƠN HÀNG CHỜ XỬ LÝ (ORDER DISPATCH QUEUE & PRIORITY QUEUE)

---

## 1. TỔNG QUAN NHIỆM VỤ (OVERVIEW)
- **Mục tiêu**: Quản lý dòng đơn hàng phát sinh liên tục trong ngày, phân loại và đưa vào hàng chờ xử lý điều phối dựa trên mức độ ưu tiên của dịch vụ (ví dụ: Hỏa tốc - Express vs. Tiêu chuẩn - Standard).
- **Yêu cầu cốt lõi**:
  - Tự cài đặt hai cấu trúc dữ liệu nền tảng từ đầu:
    1. **Hàng đợi thông thường (FIFO Queue)**: Dành cho đơn hàng giao tiêu chuẩn (đến trước xử lý trước).
    2. **Hàng đợi ưu tiên (Priority Queue dựa trên Max-Heap / Min-Heap)**: Dành cho đơn hỏa tốc, đơn hẹn giờ hoặc đơn có trọng số ưu tiên cao.
  - Hỗ trợ thêm đơn mới (`push`), lấy đơn kế tiếp cần giao (`pop`), xem đơn đầu (`peek`).
  - Phân tích độ phức tạp thời gian và không gian chi tiết.

---

## 2. VAI TRÒ TRONG DỰ ÁN VÀ BÁO CÁO CHUNG
- **"Cổng tiếp nhận" (Ingestion Gateway) của hệ thống**: Mọi đơn hàng từ khách hàng sau khi tạo phải đi qua module này trước khi được phân công shipper hay đóng hàng lên xe.
- **Đảm bảo Cam kết chất lượng dịch vụ (SLA - Service Level Agreement)**: Giúp công ty logistics ưu tiên giải quyết các đơn hàng giá trị cao hoặc đơn giao gấp trong 1 giờ.
- **Trong báo cáo LaTeX**:
  - Trình bày tại Chương 2 (Cấu trúc dữ liệu tuyến tính và cây phân cấp).
  - Phân tích chi tiết tại sao mảng thông thường không đáp ứng được yêu cầu về thời gian khi khối lượng đơn hàng lên đến hàng triệu đơn mỗi ngày.

---

## 3. CẤU TRÚC THƯ MỤC VÀ LIÊN KẾT FILE
```text
project-root/
│
├── order_queue.py          # File mã nguồn chính chứa FIFO Queue & Priority Queue
├── order_tracking.py       # Cập nhật trạng thái đơn (PENDING -> QUEUED)
├── Docs/
│   ├── task.md
│   ├── task_5_order_queue.md
│   └── ...
└── test.py                 # Kiểm thử hàng đợi với 10.000+ đơn hàng mô phỏng
```

---

## 4. CÔNG NGHỆ, PHƯƠNG PHÁP & SO SÁNH ĐÁNH GIÁ

### 4.1. Các phương pháp được lựa chọn
1. **FIFO Queue bằng Danh sách liên kết đơn (Singly Linked List with Head & Tail)**:
   - Thêm vào đuôi (`tail`): $O(1)$.
   - Lấy ra ở đầu (`head`): $O(1)$.
   - Không bị giới hạn kích thước cố định như mảng tĩnh (Static Array).
2. **Priority Queue bằng Binary Heap (Max-Heap / Min-Heap tự cài đặt)**:
   - Thêm đơn hàng: $O(\log n)$.
   - Lấy đơn có mức ưu tiên cao nhất: $O(\log n)$.
   - Xem đơn ưu tiên nhất: $O(1)$.

### 4.2. So sánh với các cấu trúc dữ liệu khác & Lý do không chọn
| Cấu trúc dữ liệu | Thao tác Thêm (Insert) | Thao tác Lấy ra (Extract-Max/Min) | Lý do không chọn |
| :--- | :--- | :--- | :--- |
| **Mảng Python thông thường (`list.pop(0)`)** | $O(1)$ (`append`) | **$O(n)$** | Khi gọi `pop(0)`, Python phải dồn toàn bộ $n-1$ phần tử còn lại về phía trước. Với $100.000$ đơn hàng, hệ thống sẽ bị nghẽn nghiêm trọng. |
| **Mảng có thứ tự (Sorted Array)** | **$O(n)$** | $O(1)$ | Mỗi khi chèn một đơn hàng mới phải tìm vị trí và dịch chuyển các phần tử phía sau, chi phí $O(n)$ quá lớn khi đơn hàng vào liên tục. |
| **Thư viện có sẵn (`queue.Queue`, `heapq`)** | $O(\log n)$ | $O(\log n)$ | Vi phạm quy định không dùng thư viện ngoài của môn học. |
| **Cây tìm kiếm nhị phân cân bằng (AVL / Red-Black Tree)** | $O(\log n)$ | $O(\log n)$ | Cài đặt cực kỳ phức tạp (phải xoay cây để cân bằng) trong khi chỉ cần tìm phần tử lớn nhất/nhỏ nhất thì Binary Heap đơn giản, tốn ít bộ nhớ và bộ đệm cache tốt hơn nhiều. |

---

## 5. CÁC THUẬT TOÁN & CẤU TRÚC DỮ LIỆU CẦN CÀI ĐẶT
1. **Lớp đối tượng Đơn hàng (`Order`)**:
   - `order_id`: Mã đơn (ví dụ: `ORD_001`).
   - `created_time`: Thời điểm tạo đơn.
   - `priority_level`: Mức độ ưu tiên (Số nguyên, ví dụ: 3 = Hỏa tốc, 2 = Nhanh, 1 = Tiêu chuẩn).
   - `delivery_deadline`: Hạn chót giao hàng (Timestamp).
   - `weight`, `volume`: Khối lượng và kích thước.
   - `destination`: Địa điểm giao hàng.
2. **Hàng đợi FIFO (`StandardOrderQueue`)**:
   - `enqueue(order)`: Thêm đơn vào cuối.
   - `dequeue()`: Lấy đơn lâu nhất ra.
3. **Hàng đợi Ưu tiên (`PriorityOrderQueue`)**:
   - Tiêu chí ưu tiên: So sánh theo cặp `(priority_level DESC, created_time ASC)` (Ưu tiên hạng dịch vụ trước; nếu cùng hạng thì ai đến trước xử lý trước).
   - Các thao tác: `push(order)`, `pop()`, `peek()`, `_sift_up()`, `_sift_down()`.

---

## 6. CHI TIẾT THIẾT KẾ FILE VÀ CÁC HÀM TRONG `order_queue.py`

### 6.1. Cấu trúc Node và Queue liên kết
- `class QueueNode`:
  - `data`: Lưu đối tượng `Order`.
  - `next`: Con trỏ trỏ tới `QueueNode` tiếp theo.
- `class StandardOrderQueue`:
  - `__init__(self)`: `self.head = None`, `self.tail = None`, `self.size = 0`.
  - `enqueue(self, order)`: Thêm đơn hàng vào đuôi danh sách. $O(1)$.
  - `dequeue(self)`: Rút đơn hàng ở đầu danh sách. $O(1)$.
  - `is_empty(self)`: $O(1)$.

### 6.2. Cấu trúc Hàng đợi ưu tiên
- `class PriorityOrderQueue`:
  - `__init__(self)`: `self.heap = []`, `self.size = 0`.
  - `push(self, order)`: Thêm đơn vào heap, kích hoạt sift-up. $O(\log n)$.
  - `pop(self)`: Rút đơn có mức ưu tiên cao nhất ra khỏi heap, kích hoạt sift-down. $O(\log n)$.
  - `peek(self)`: Trả về đơn ưu tiên nhất nhưng không xóa khỏi hàng đợi. $O(1)$.
  - `_compare(self, order_a, order_b)`: Hàm so sánh đa tiêu chí (độ ưu tiên $\rightarrow$ thời gian chờ).

### 6.3. Bộ điều phối hàng đợi tổng hợp
- `class OrderDispatcher`:
  - Quản lý đồng thời `express_queue` (Priority Queue) và `standard_queue` (FIFO Queue).
  - `add_order(self, order)`: Đơn hỏa tốc tự động vào Priority Queue, đơn thường vào FIFO Queue.
  - `get_next_order(self)`: Ưu tiên rút cạn các đơn hỏa tốc trước; khi hết đơn hỏa tốc mới rút đơn thường.

---

## 7. KẾT QUẢ ĐẦU RA CẦN ĐẠT (DELIVERABLES)
1. **Mã nguồn `order_queue.py`**:
   - Cài đặt đầy đủ, độc lập, không dùng thư viện ngoài.
   - Chạy thử nghiệm mô phỏng phát sinh ngẫu nhiên 10.000 đơn và rút ra đúng thứ tự ưu tiên.
2. **Kiểm thử hiệu năng (Benchmarking)**:
   - Đo thời gian xử lý khi so sánh `list.pop(0)` của Python với `CustomQueue.dequeue()`.
   - Lập bảng số liệu chứng minh tốc độ vượt trội $O(1)$ so với $O(n)$.
3. **Báo cáo LaTeX**:
   - Trình bày cấu trúc mảng của Binary Heap và thuật toán sàng (Sift/Heapify).
   - Biểu đồ thời gian thực thi giữa các phương pháp.

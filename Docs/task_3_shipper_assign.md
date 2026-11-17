# NHIỆM VỤ 03: PHÂN CÔNG SHIPPER GẦN NHẤT (SHIPPER ASSIGNMENT)

---

## 1. TỔNG QUAN NHIỆM VỤ (OVERVIEW)
- **Mục tiêu**: Khi phát sinh một đơn hàng mới (hoặc cần lấy hàng từ một vị trí cụ thể), hệ thống cần quét các tài xế (shipper) đang rảnh rỗi trên bản đồ và phân công ngay cho shipper có vị trí gần nhất hoặc thời gian tiếp cận nhanh nhất.
- **Yêu cầu cốt lõi**:
  - Tự cài đặt thuật toán tìm kiếm trên đồ thị:
    - **BFS (Breadth-First Search)**: Trong trường hợp đồ thị không có trọng số (khoảng cách tính theo số chặng / ngã tư).
    - **Dijkstra đa nguồn / đảo ngược (Multi-source / Reverse Dijkstra)**: Khi đồ thị có trọng số khoảng cách thực tế.
  - Tự cài đặt cấu trúc dữ liệu **Hàng đợi chuẩn (Queue)** từ đầu (không dùng `collections.deque`).
  - Hỗ trợ quản lý trạng thái tài xế: vị trí hiện tại, trạng thái (sẵn sàng/bận).

---

## 2. VAI TRÒ TRONG DỰ ÁN VÀ BÁO CÁO CHUNG
- **Kết nối nhu cầu khách hàng với đội ngũ vận hành**: Đây là mắt xích quyết định tốc độ đáp ứng dịch vụ (Response Time / Dispatch Time) trong logistics công nghệ tương tự GrabFood, ShopeeFood hay AhaMove.
- **Tiền đề cho quy trình đóng gói & giao hàng**: Sau khi shipper được gán đơn, đơn hàng mới được xếp vào danh sách giao của shipper đó (`order_queue.py` và `truck_loading.py`).
- **Trong báo cáo LaTeX**:
  - Trình bày tại mục Điều phối nguồn lực (Resource Allocation) trong Chương 3.
  - Phân tích kỹ thuật giải bài toán "1 điểm lấy hàng - $K$ shipper đang di chuyển": Tại sao chạy Dijkstra từ vị trí đơn hàng lại tối ưu hơn chạy $K$ lần Dijkstra từ từng shipper.

---

## 3. CẤU TRÚC THƯ MỤC VÀ LIÊN KẾT FILE
```text
project-root/
│
├── shipper_assign.py       # File mã nguồn chính của nhiệm vụ này
├── graph.py                # Lấy đồ thị bản đồ
├── shortest_path.py        # Tận dụng thuật toán Dijkstra hoặc Min-Heap
├── Docs/
│   ├── task.md
│   ├── task_3_shipper_assign.md
│   └── ...
└── test.py                 # Kiểm thử phân công shipper với nhiều kịch bản
```

---

## 4. CÔNG NGHỆ, PHƯƠNG PHÁP & SO SÁNH ĐÁNH GIÁ

### 4.1. Phương pháp được chọn
Tùy vào tính chất đồ thị giao thông:
1. **BFS (Breadth-First Search) kết hợp Queue tự cài đặt (Custom FIFO Queue)**:
   - Dùng khi đồ thị xấp xỉ không trọng số (mỗi cạnh là 1 khối phố, tìm shipper ít bước chuyển nhất).
   - Độ phức tạp: $O(|V| + |E|)$.
2. **Reverse/Single-Source Dijkstra từ vị trí đơn hàng**:
   - Dùng khi cạnh có trọng số thực tế (km/phút). Bắt đầu Dijkstra từ đỉnh lấy hàng (Pickup Location), lan truyền ra ngoài cho đến khi chạm tới shipper rảnh rỗi gần nhất.
   - Độ phức tạp: $O((|V'| + |E'|) \log |V|)$ với $|V'| \ll |V|$ vì có thể dừng sớm (Early Stopping) ngay khi gặp shipper đầu tiên.

### 4.2. So sánh với các phương pháp khác & Lý do không chọn
| Phương pháp | Ý tưởng | Độ phức tạp | Lý do không chọn / Nhược điểm |
| :--- | :--- | :--- | :--- |
| **Dijkstra $K$ lần từ từng shipper** | Chạy Dijkstra từ shipper 1, 2, ..., $K$ đến vị trí đơn hàng | $K \times O((|V| + \|E\|) \log \|V\|)$ | Rất lãng phí tài nguyên tính toán nếu có 100 shipper rảnh rỗi trên bản đồ (phải chạy 100 lần thuật toán). |
| **Khoảng cách Euclidean (Đường chim bay)** | Tính khoảng cách hình học $\sqrt{\Delta x^2 + \Delta y^2}$ giữa tọa độ shipper và đơn | $O(K)$ | **Sai lệch nghiêm trọng trong thực tế** do sông ngòi, cầu vượt, đường một chiều, ngõ cụt. Shipper nhìn rất gần nhưng phải đi vòng 5 km qua cầu. |
| **DFS (Depth-First Search)** | Duyệt theo chiều sâu | $O(\|V\| + \|E\|)$ | DFS **không** đảm bảo tìm thấy đường ngắn nhất, có thể đi lạc vào nhánh rất sâu trước khi tìm thấy shipper. |

---

## 5. CÁC THUẬT TOÁN & CẤU TRÚC DỮ LIỆU CẦN CÀI ĐẶT
1. **Cấu trúc Hàng đợi FIFO tự viết (`CustomQueue`)**:
   - Dùng Danh sách liên kết đơn (Singly Linked List) với con trỏ `head` (lấy ra) và `tail` (thêm vào) để đạt thời gian $O(1)$ cho cả `enqueue` và `dequeue`.
2. **Thuật toán BFS dừng sớm (`bfs_nearest_shipper`)**:
   - Duyệt theo từng lớp khoảng cách cho đến khi chạm đỉnh chứa shipper có trạng thái `AVAILABLE`.
3. **Thuật toán Dijkstra dừng sớm (`dijkstra_nearest_shipper`)**:
   - Min-heap xuất phát từ điểm lấy hàng. Khi phần tử lấy ra từ heap trùng với vị trí shipper rảnh rỗi, kết thúc ngay lập tức.

---

## 6. CHI TIẾT THIẾT KẾ FILE VÀ CÁC HÀM TRONG `shipper_assign.py`

### 6.1. Các lớp (Classes)
- `class Shipper`:
  - `shipper_id`: Mã định danh shipper (ví dụ: `SHP_001`).
  - `current_location`: Đỉnh đồ thị hiện tại shipper đang đứng.
  - `is_available`: Boolean (`True` nếu đang rảnh, `False` nếu đang bận giao).
- `class Node` & `class CustomQueue`:
  - `enqueue(item)`: Thêm vào cuối hàng đợi $O(1)$.
  - `dequeue()`: Lấy khỏi đầu hàng đợi $O(1)$.
  - `is_empty()`: Kiểm tra rỗng $O(1)$.

### 6.2. Các hàm giải thuật chính
- `find_nearest_shipper_bfs(graph, pickup_location, shippers_list)`:
  - Dùng cho đồ thị không trọng số.
  - Trả về: `(best_shipper, path, step_count)`.
- `find_nearest_shipper_dijkstra(graph, pickup_location, shippers_list)`:
  - Dùng cho đồ thị có trọng số thực tế.
  - Khởi tạo khoảng cách, lan truyền từ `pickup_location`.
  - Kiểm tra nếu đỉnh đang xét có shipper rảnh rỗi thì lập tức ngắt vòng lặp (`Early Exit`).
  - Trả về: `(best_shipper, path, total_distance)`.
- `assign_order_to_shipper(order, shipper)`:
  - Cập nhật trạng thái shipper thành bận (`is_available = False`), gán mã đơn hàng cho shipper.

---

## 7. KẾT QUẢ ĐẦU RA CẦN ĐẠT (DELIVERABLES)
1. **Mã nguồn `shipper_assign.py`**:
   - Cài đặt đầy đủ `CustomQueue` và thuật toán tìm kiếm.
   - Có hàm mô phỏng kịch bản: phân công 10 đơn hàng cho 5 shipper rải rác trên bản đồ.
2. **Kiểm thử và đánh giá**:
   - So sánh thời gian thực thi giữa phương án "Dijkstra 1 lần từ vị trí đơn" và "Dijkstra $K$ lần từ từng shipper".
3. **Báo cáo LaTeX**:
   - Mô tả chiến lược điều phối đơn hàng tự động.
   - Bảng phân tích chi phí tính toán lý thuyết và thực nghiệm.

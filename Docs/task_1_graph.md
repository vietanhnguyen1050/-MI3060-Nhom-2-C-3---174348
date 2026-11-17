# NHIỆM VỤ 01: XÂY DỰNG CẤU TRÚC ĐỒ THỊ BẢN ĐỒ GIAO HÀNG (GRAPH MODEL)

---

## 1. TỔNG QUAN NHIỆM VỤ (OVERVIEW)
- **Mục tiêu**: Xây dựng mô hình biểu diễn mạng lưới giao thông nội thành dưới dạng đồ thị có trọng số (khoảng cách hoặc thời gian di chuyển giữa các nút giao, kho hàng, điểm nhận/trả).
- **Yêu cầu cốt lõi**:
  - Tự cài đặt cấu trúc dữ liệu đồ thị từ đầu bằng Python (thuần hướng đối tượng, **không** dùng thư viện như `networkx`).
  - Hỗ trợ thêm/xóa đỉnh, cạnh, cập nhật trọng số động (ví dụ: đường tắc, mưa ngập).
  - Đọc/ghi đồ thị từ file cấu hình dữ liệu (`.txt`, `.csv` hoặc `.json`).
  - Cung cấp API trực quan, ổn định cho các module thuật toán khác truy vấn thông tin láng giềng, trọng số.

---

## 2. VAI TRÒ TRONG DỰ ÁN VÀ BÁO CÁO CHUNG
- **"Xương sống" (Backbone) của toàn bộ hệ thống logistics**: Mọi bài toán sau đó như Tìm đường ngắn nhất (`shortest_path.py`), Phân công shipper (`shipper_assign.py`), Tối ưu hóa lộ trình đa điểm (`route_planner.py`) đều phụ thuộc trực tiếp vào module này để lấy thông tin địa hình mạng lưới.
- **Trong báo cáo LaTeX**:
  - Xuất hiện ở Chương 2 (Mô hình hóa bài toán & Thiết kế cấu trúc dữ liệu nền tảng).
  - Cung cấp mô hình toán học $G = (V, E, W)$, định nghĩa ma trận/danh sách kề, phân tích không gian bộ nhớ lưu trữ $O(|V| + |E|)$.

---

## 3. CẤU TRÚC THƯ MỤC VÀ LIÊN KẾT FILE
```text
project-root/
│
├── graph.py                # File mã nguồn chính của nhiệm vụ này
├── Docs/
│   ├── task.md
│   ├── task_1_graph.md     # Tài liệu đặc tả nhiệm vụ này
│   └── ...
├── data/
│   ├── map_small.txt       # Dữ liệu kiểm thử nhỏ (unit test)
│   └── map_large.txt       # Dữ liệu mô phỏng khu vực nội thành quy mô lớn
└── test.py                 # File chạy test kiểm thử hiệu năng
```

---

## 4. CÔNG NGHỆ, PHƯƠNG PHÁP & SO SÁNH ĐÁNH GIÁ

### 4.1. Phương pháp được chọn: **Danh sách kề tự cài đặt (Adjacency List with Custom Linked List / Dictionary)**
- **Mô tả**: Mỗi đỉnh lưu một danh sách liên kết (hoặc bảng băm tự cài đặt) chứa các đỉnh kề và trọng số tương ứng $w(u, v)$.
- **Lý do lựa chọn**:
  - Bản đồ giao thông thực tế là **đồ thị thưa** (Sparse Graph): mỗi nút giao thông thông thường chỉ kết nối với 3 đến 5 nhánh đường (bậc của đỉnh $deg(v) \le 5$, số cạnh $|E| \approx 3|V|$ đến $4|V| \ll |V|^2$).
  - Tiết kiệm bộ nhớ vượt trội: độ phức tạp không gian chỉ là $O(|V| + |E|)$.
  - Phù hợp hoàn hảo cho thuật toán Dijkstra và BFS: thao tác duyệt qua tất cả các đỉnh kề của đỉnh $u$ tốn đúng $O(deg(u))$.

### 4.2. So sánh với các phương pháp khác & Lý do không chọn
| Phương pháp | Độ phức tạp bộ nhớ | Duyệt đỉnh kề $O(...)$ | Kiểm tra cạnh $(u,v)$ | Lý do không chọn |
| :--- | :--- | :--- | :--- | :--- |
| **Ma trận kề (Adjacency Matrix)** | $O(\|V\|^2)$ | $O(\|V\|)$ | $O(1)$ | Lãng phí bộ nhớ khủng khiếp với đồ thị lớn (ví dụ bản đồ 10.000 nút giao cần mảng $10^8$ phần tử); duyệt đỉnh kề tốn $O(\|V\|)$ làm chậm Dijkstra. |
| **Danh sách cạnh (Edge List)** | $O(\|E\|)$ | $O(\|E\|)$ | $O(\|E\|)$ | Muốn tìm láng giềng phải quét toàn bộ tập cạnh, cực kỳ kém hiệu quả cho việc tìm đường. |
| **Thư viện có sẵn (NetworkX)** | Tối ưu | $O(deg(u))$ | $O(1)$ | Vi phạm yêu cầu tiên quyết của đề tài môn học (tự cài đặt cấu trúc dữ liệu). |

---

## 5. CÁC THUẬT TOÁN CẦN SỬ DỤNG
1. **Thuật toán nạp đồ thị (Graph Parsing Algorithm)**: Đọc file văn bản theo cấu trúc đỉnh - cạnh - trọng số và dựng cấu trúc danh sách kề trong thời gian $O(|V| + |E|)$.
2. **Thuật toán cập nhật trọng số động (Dynamic Weight Update)**: Hỗ trợ sửa đổi trọng số cạnh $O(deg(u))$ khi có thông tin kẹt xe.
3. **Thuật toán kiểm tra tính liên thông**: Duyệt đồ thị để cảnh báo nếu bản đồ bị cô lập.

---

## 6. CHI TIẾT THIẾT KẾ FILE VÀ CÁC HÀM TRONG `graph.py`

### 6.1. Các lớp (Classes)
- `class Edge`:
  - `target`: Đỉnh đích.
  - `weight`: Trọng số (khoảng cách km hoặc thời gian phút).
  - `road_name`: Tên đường / thông tin phụ.
  - `next`: Con trỏ trỏ tới cạnh tiếp theo nếu dùng Danh sách liên kết đơn tự cài đặt.
- `class Graph`:
  - Đại diện cho toàn bộ đồ thị mạng lưới giao thông.

### 6.2. Các hàm cốt lõi trong `Graph`
- `__init__(self, directed=False)`: Khởi tạo đồ thị rỗng (vô hướng hoặc có hướng cho đường một chiều).
- `add_vertex(self, vertex_id)`: Thêm một địa điểm/nút giao mới. $O(1)$.
- `add_edge(self, u, v, weight, road_name="")`: Thêm cạnh nối 2 nút giao với trọng số tương ứng. $O(1)$.
- `get_neighbors(self, u)`: Trả về danh sách/con trỏ các đỉnh kề của đỉnh $u$. $O(1)$ trả về đầu danh sách, duyệt hết trong $O(deg(u))$.
- `get_weight(self, u, v)`: Lấy trọng số giữa 2 đỉnh kề nhau. $O(deg(u))$.
- `update_edge_weight(self, u, v, new_weight)`: Cập nhật trọng số khi đường bị tắc hoặc thông thoáng. $O(deg(u))$.
- `load_from_file(self, filepath)`: Đọc và dựng đồ thị từ file dữ liệu. $O(|V| + |E|)$.
- `display(self)`: In trực quan đồ thị phục vụ gỡ lỗi.

---

## 7. KẾT QUẢ ĐẦU RA CẦN ĐẠT (DELIVERABLES)
1. **Mã nguồn `graph.py`**:
   - Chạy độc lập không lỗi, có docstring giải thích chi tiết.
   - Không import các thư viện cấu trúc dữ liệu có sẵn (`heapq`, `collections`, `networkx`).
2. **Bộ dữ liệu kiểm thử**: File `data/map_small.txt` (5-10 đỉnh) và `data/map_large.txt` (1.000+ đỉnh, 3.000+ cạnh).
3. **Báo cáo phân tích**: Bảng phân tích chi tiết độ phức tạp không gian & thời gian của từng thao tác để đưa vào tài liệu LaTeX.

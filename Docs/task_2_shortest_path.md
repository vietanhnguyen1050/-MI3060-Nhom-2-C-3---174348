# NHIỆM VỤ 02: TÌM ĐƯỜNG NGẮN NHẤT TỪ KHO ĐẾN KHÁCH HÀNG (DIJKSTRA ALGORITHM)

---

## 1. TỔNG QUAN NHIỆM VỤ (OVERVIEW)
- **Mục tiêu**: Tìm đường đi ngắn nhất (về khoảng cách địa lý hoặc thời gian di chuyển) giữa hai điểm bất kỳ trên bản đồ giao thông (từ kho trung tâm đến điểm giao hàng của khách).
- **Yêu cầu cốt lõi**:
  - Tự cài đặt thuật toán Dijkstra tìm đường ngắn nhất.
  - Tự cài đặt cấu trúc dữ liệu **Hàng đợi ưu tiên (Min-Heap / Binary Heap)** từ đầu, không dùng thư viện `heapq` của Python.
  - Truy vết đường đi chi tiết (danh sách các nút giao liên tiếp) và tính tổng chi phí (km hoặc phút).
  - Đo đạc thời gian thực thi trên đồ thị quy mô lớn để đánh giá hiệu năng thuật toán.

---

## 2. VAI TRÒ TRONG DỰ ÁN VÀ BÁO CÁO CHUNG
- **Cốt lõi định tuyến đơn lẻ**: Đây là chức năng cơ bản nhất của bất kỳ hệ thống bản đồ hoặc ứng dụng logistics nào (tương tự Google Maps / Grab Maps).
- **Phục vụ các module cấp cao hơn**:
  - `route_planner.py` cần gọi Dijkstra để xây dựng ma trận khoảng cách giữa các điểm giao hàng trước khi chạy thuật toán TSP.
  - `shipper_assign.py` có thể dùng Dijkstra để tìm shipper có thời gian di chuyển ngắn nhất đến điểm lấy hàng.
- **Trong báo cáo LaTeX**:
  - Trọng tâm tại Chương 3 (Các thuật toán tối ưu hóa định tuyến).
  - Trình bày chứng minh tính đúng đắn của Dijkstra, phân tích độ phức tạp thời gian khi dùng Min-Heap tự cài đặt $O((|V| + |E|) \log |V|)$ so với duyệt tuyến tính $O(|V|^2)$.

---

## 3. CẤU TRÚC THƯ MỤC VÀ LIÊN KẾT FILE
```text
project-root/
│
├── shortest_path.py        # File mã nguồn chính chứa Min-Heap & Dijkstra
├── graph.py                # Import cấu trúc Graph đã làm ở Nhiệm vụ 01
├── Docs/
│   ├── task.md
│   ├── task_2_shortest_path.md
│   └── ...
└── test.py                 # Chạy benchmark hiệu năng thuật toán Dijkstra
```

---

## 4. CÔNG NGHỆ, PHƯƠNG PHÁP & SO SÁNH ĐÁNH GIÁ

### 4.1. Phương pháp được chọn: **Thuật toán Dijkstra kết hợp Min-Heap tự cài đặt (Custom Binary Min-Heap)**
- **Lý do lựa chọn**:
  - Bản đồ giao thông thực tế có trọng số cạnh $w(u,v) \ge 0$ (khoảng cách hay thời gian không thể âm), đây là điều kiện lý tưởng cho Dijkstra.
  - Sử dụng Min-Heap giúp thao tác lấy đỉnh có khoảng cách nhỏ nhất (`extract_min`) chỉ tốn $O(\log |V|)$, và thao tác thêm/cập nhật đỉnh tốn $O(\log |V|)$.
  - Tổng thời gian thực thi: $O((|V| + |E|) \log |V|)$, cực nhanh trên đồ thị thưa giao thông.

### 4.2. So sánh với các thuật toán khác & Lý do không chọn
| Thuật toán | Độ phức tạp thời gian | Trọng số âm | Mục tiêu | Lý do không chọn |
| :--- | :--- | :--- | :--- | :--- |
| **Dijkstra duyệt mảng tuyến tính** | $O(\|V\|^2)$ | Không | Đơn nguồn | Quá chậm khi đồ thị có hàng nghìn đỉnh ($|V| = 10.000 \implies 10^8$ phép tính). |
| **Bellman-Ford** | $O(\|V\| \cdot \|E\|)$ | Có | Đơn nguồn | Rất chậm; bản đồ giao thông logistics không có khoảng cách âm nên tính năng xử lý trọng số âm của Bellman-Ford là dư thừa và gây lãng phí tài nguyên tính toán. |
| **Floyd-Warshall** | $O(\|V\|^3)$ | Có | Mọi cặp đỉnh | Tiêu tốn $O(\|V\|^3)$ thời gian và $O(\|V\|^2)$ bộ nhớ; hoàn toàn bất khả thi cho đồ thị bản đồ quy mô lớn (ví dụ $|V| = 5.000$). |
| **A* (A-star)** | Nhanh hơn Dijkstra (dựa vào Heuristic) | Không | Đích cụ thể | Đòi hỏi hàm ước lượng Heuristic chính xác (tọa độ GPS $x, y$ thực tế theo đường chim bay). Với bài toán tổng quát chỉ có trọng số đồ thị, Dijkstra là lựa chọn chuẩn tắc và bắt buộc theo đề cương. |

---

## 5. CÁC THUẬT TOÁN & CẤU TRÚC DỮ LIỆU CẦN CÀI ĐẶT
1. **Cấu trúc Min-Heap tự viết (`CustomMinHeap`)**:
   - Mảng biểu diễn cây nhị phân: Nút cha tại $i$, con trái tại $2i+1$, con phải tại $2i+2$.
   - Các thao tác: `push` (thêm phần tử và heapify-up), `pop` (lấy phần tử nhỏ nhất và heapify-down).
2. **Thuật toán Dijkstra (`dijkstra_shortest_path`)**:
   - Khởi tạo khoảng cách `dist[source] = 0`, các đỉnh khác bằng $\infty$.
   - Mảng `parent` để truy vết đường đi.
   - Vòng lặp thư giãn cạnh (edge relaxation) thông qua Min-Heap.
3. **Thuật toán truy vết đường đi (`reconstruct_path`)**:
   - Đi ngược từ đỉnh đích `target` về đỉnh nguồn `source` qua mảng `parent`, sau đó đảo ngược danh sách.

---

## 6. CHI TIẾT THIẾT KẾ FILE VÀ CÁC HÀM TRONG `shortest_path.py`

### 6.1. Lớp `CustomMinHeap`
- `__init__(self)`: Khởi tạo mảng rỗng `heap = []` và biến đếm số lượng `size = 0`.
- `push(self, priority, item)`: Thêm một cặp `(priority, item)` vào cuối mảng rồi thực hiện `_sift_up`. Độ phức tạp $O(\log n)$.
- `pop(self)`: Đổi chỗ phần tử đầu và cuối, xóa phần tử nhỏ nhất, rồi thực hiện `_sift_down`. Độ phức tạp $O(\log n)$.
- `is_empty(self)`: Kiểm tra heap còn phần tử không. $O(1)$.
- `_sift_up(self, idx)`: Đẩy nút con lên nếu nhỏ hơn cha. $O(\log n)$.
- `_sift_down(self, idx)`: Đẩy nút cha xuống nếu lớn hơn con nhỏ nhất. $O(\log n)$.

### 6.2. Các hàm giải thuật chính
- `dijkstra(graph, start_vertex)`:
  - Đầu vào: Đồ thị `graph` và đỉnh xuất phát `start_vertex`.
  - Đầu ra: Bảng khoảng cách `distances` và bảng truy vết `previous_nodes` tới tất cả các đỉnh khác.
  - Độ phức tạp: $O((|V| + |E|) \log |V|)$.
- `find_shortest_path(graph, start_vertex, end_vertex)`:
  - Gọi `dijkstra`, sau đó gọi hàm truy vết để trả về tuple: `(path_list, total_distance)`.
  - Trả về `([], inf)` nếu không có đường đi kết nối.
- `print_path(path_list, total_distance)`:
  - In ra định dạng dễ đọc: `Kho A -> Ngã tư B -> Khách hàng C (Tổng: 5.2 km)`.

---

## 7. KẾT QUẢ ĐẦU RA CẦN ĐẠT (DELIVERABLES)
1. **Mã nguồn `shortest_path.py`**: Cài đặt độc lập hoàn toàn, Min-Heap tự dựng từ cấu trúc danh sách cơ bản, code sạch, có chú thích chi tiết.
2. **Kết quả kiểm thử**:
   - Kiểm thử tính đúng đắn trên đồ thị nhỏ có thể kiểm tra bằng tay.
   - Benchmark thời gian chạy trên đồ thị $1.000$ đến $10.000$ đỉnh để vẽ biểu đồ cho báo cáo.
3. **Nội dung viết báo cáo**:
   - Mã giả (Pseudocode) của Dijkstra và Min-Heap.
   - So sánh thời gian chạy thực nghiệm và lý thuyết.

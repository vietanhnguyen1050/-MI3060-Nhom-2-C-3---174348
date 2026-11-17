# NHIỆM VỤ 04: LẬP LỘ TRÌNH GIAO NHIỀU ĐIỂM TRONG MỘT CHUYẾN (MULTI-STOP ROUTE PLANNER)

---

## 1. TỔNG QUAN NHIỆM VỤ (OVERVIEW)
- **Mục tiêu**: Khi một shipper nhận đồng thời $N$ đơn hàng khác nhau trong cùng một ca làm việc, hệ thống cần tìm thứ tự ghé thăm các điểm giao hàng sao cho tổng quãng đường hoặc tổng thời gian di chuyển từ kho (hoặc điểm xuất phát) qua tất cả các điểm rồi kết thúc là nhỏ nhất.
- **Bản chất toán học**: Đây là bài toán Người du lịch (**Traveling Salesperson Problem - TSP**) nổi tiếng, thuộc lớp bài toán NP-khó (NP-hard).
- **Yêu cầu cốt lõi**:
  - Tự cài đặt hai phương pháp tiếp cận:
    1. **Phương pháp chính xác**: Quy hoạch động có trạng thái bitmask (Held-Karp) hoặc Nhánh cận / Quay lui (Branch and Bound / Backtracking) khi số điểm $N$ nhỏ ($N \le 15$).
    2. **Phương pháp xấp xỉ (Heuristic)**: Thuật toán Tham lam láng giềng gần nhất (Nearest Neighbor) khi số điểm $N$ lớn ($N > 15$).
  - Tự tính ma trận khoảng cách giữa các điểm giao bằng cách gọi hàm `dijkstra` từ module `shortest_path.py`.

---

## 2. VAI TRÒ TRONG DỰ ÁN VÀ BÁO CÁO CHUNG
- **Đỉnh cao tối ưu hóa chi phí vận hành**: Trong thực tế, chi phí logistics tiết kiệm được nhiều nhất ở khâu gom đơn và tối ưu đường đi giao chặng cuối (Last-mile delivery). Đi sai thứ tự có thể làm quãng đường tăng gấp đôi.
- **Hoàn thiện chu trình điều phối**: Nhận danh sách đơn từ Hàng đợi (`order_queue.py`), xác định thứ tự giao tối ưu, từ đó cung cấp thứ tự này cho module Xếp dỡ thùng xe (`truck_loading.py`) theo nguyên tắc LIFO.
- **Trong báo cáo LaTeX**:
  - Chương 3 - Phần "Bài toán tối ưu hóa tổ hợp (Combinatorial Optimization)".
  - Trình bày công thức quy hoạch động Held-Karp, chứng minh độ phức tạp tính toán và so sánh nghiệm giữa thuật toán chính xác và tham lam.

---

## 3. CẤU TRÚC THƯ MỤC VÀ LIÊN KẾT FILE
```text
project-root/
│
├── route_planner.py        # File mã nguồn chính chứa TSP (Greedy & DP/Backtracking)
├── shortest_path.py        # Gọi hàm dijkstra để xây dựng ma trận khoảng cách
├── graph.py                # Truy cập cấu trúc đồ thị
├── Docs/
│   ├── task.md
│   ├── task_4_route_planner.md
│   └── ...
└── test.py                 # So sánh chất lượng nghiệm và thời gian giữa các thuật toán
```

---

## 4. CÔNG NGHỆ, PHƯƠNG PHÁP & SO SÁNH ĐÁNH GIÁ

### 4.1. Các phương pháp được lựa chọn
1. **Thuật toán Tham lam láng giềng gần nhất (Nearest Neighbor - Greedy)**:
   - Từ điểm hiện tại, luôn chọn đi tới điểm chưa ghé thăm gần nhất.
   - Độ phức tạp: $O(N^2)$.
   - Ưu điểm: Cực nhanh, tính toán tức thời cho cả $N = 100$ hoặc $1.000$ điểm.
2. **Quy hoạch động trạng thái Bitmask (Held-Karp Dynamic Programming)**:
   - Dùng khi $N \le 15$ để tìm được nghiệm tối ưu toàn cục tuyệt đối.
   - Trạng thái $dp(mask, u)$: chi phí nhỏ nhất để thăm tập đỉnh biểu diễn bởi bitmask `mask` và kết thúc tại đỉnh $u$.
   - Độ phức tạp: $O(N^2 \cdot 2^N)$.

### 4.2. So sánh với các phương pháp khác & Lý do không chọn
| Phương pháp | Độ phức tạp thời gian | Tính tối ưu | Lý do không chọn làm phương pháp duy nhất |
| :--- | :--- | :--- | :--- |
| **Vét cạn / Hoán vị toàn bộ ($N!$)** | $O(N!)$ | Tối ưu tuyệt đối | Bùng nổ tổ hợp quá nhanh ($10! \approx 3.6 \times 10^6$, nhưng $15! \approx 1.3 \times 10^{12}$ phép tính $\implies$ treo máy). |
| **Chỉ dùng Quy hoạch động (Held-Karp)** | $O(N^2 \cdot 2^N)$ | Tối ưu tuyệt đối | Chỉ chạy được với $N \le 18$. Nếu một shipper gom 30 bưu kiện trong một ngày thì Held-Karp tràn bộ nhớ và chạy mất hàng giờ. |
| **Chỉ dùng Tham lam (Nearest Neighbor)** | $O(N^2)$ | Gần đúng (xấp xỉ) | Có thể rơi vào bẫy cục bộ (Local Optima), chặng cuối cùng có thể phải đi một quãng đường cực xa trở về kho. Cần thuật toán chính xác để đối chiếu chất lượng. |
| **Giải thuật Di truyền (Genetic Algorithm) / Ant Colony** | Heuristic ngẫu nhiên | Khá tốt | Cấu trúc phức tạp, khó kiểm soát thời gian hội tụ và vi phạm tính chất tự cài đặt cấu trúc dữ liệu cơ bản theo yêu cầu đề cương môn học. |

$\implies$ **Quyết định thiết kế**: Kết hợp chiến lược Hybrid — Hệ thống tự động kiểm tra: nếu $N \le 12$ gọi Quy hoạch động/Nhánh cận; nếu $N > 12$ tự động chuyển sang Tham lam Láng giềng gần nhất.

---

## 5. CÁC THUẬT TOÁN & BƯỚC THỰC HIỆN CỤ THỂ
1. **Xây dựng ma trận khoảng cách đầy đủ (Distance Matrix Generation)**:
   - Với $N$ điểm giao hàng $\{p_1, p_2, ..., p_N\}$ và kho $S$, chạy Dijkstra $N+1$ lần để tạo ma trận kích thước $(N+1) \times (N+1)$ trong thời gian $(N+1) \times O((|V| + |E|) \log |V|)$.
2. **Thuật toán Nearest Neighbor**:
   - Mảng đánh dấu `visited = [False] * (N+1)`.
   - Vòng lặp $N$ bước chọn `argmin(dist[curr][next])`.
3. **Thuật toán Quy hoạch động Held-Karp / Nhánh cận**:
   - Lưu trữ bảng nhớ và truy vết lại hành trình tối ưu.

---

## 6. CHI TIẾT THIẾT KẾ FILE VÀ CÁC HÀM TRONG `route_planner.py`

### 6.1. Các hàm xử lý ma trận chi phí
- `build_cost_matrix(graph, delivery_points)`:
  - Gọi `dijkstra` từ `shortest_path.py` cho từng điểm trong danh sách.
  - Trả về ma trận 2 chiều `matrix[i][j]` là khoảng cách thực tế giữa điểm $i$ và điểm $j$.

### 6.2. Các hàm giải bài toán TSP
- `tsp_nearest_neighbor(cost_matrix, start_index=0)`:
  - Tìm đường đi xấp xỉ bằng thuật toán tham lam.
  - Trả về: `(best_route, total_cost)`.
  - Độ phức tạp: $O(N^2)$.
- `tsp_dynamic_programming(cost_matrix, start_index=0)`:
  - Giải chính xác bằng Held-Karp (Bitmask DP).
  - Trả về: `(optimal_route, min_cost)`.
  - Độ phức tạp: $O(N^2 \cdot 2^N)$.
- `plan_multi_stop_route(graph, warehouse, customer_stops)`:
  - Hàm wrapper thông minh: tự động phân nhánh chọn phương pháp tối ưu dựa vào số lượng điểm `len(customer_stops)`.
  - Nối chi tiết từng chặng đường con giữa các điểm thành chuỗi đường đi hoàn chỉnh trên bản đồ giao thông.

---

## 7. KẾT QUẢ ĐẦU RA CẦN ĐẠT (DELIVERABLES)
1. **Mã nguồn `route_planner.py`**:
   - Cài đặt đầy đủ cả 2 thuật toán Tham lam và DP.
   - Cơ chế tự động lựa chọn thuật toán phù hợp.
2. **Thực nghiệm so sánh**:
   - So sánh độ lệch (Error Gap) giữa nghiệm tham lam và nghiệm tối ưu của DP trên tập điểm $N = 4$ đến $N = 12$.
   - Đo thời gian chạy khi $N$ tăng dần.
3. **Báo cáo LaTeX**:
   - Trình bày bảng so sánh hiệu quả giữa quy hoạch động và tham lam.
   - Minh họa đường đi của một lộ trình giao hàng mẫu qua bản đồ.

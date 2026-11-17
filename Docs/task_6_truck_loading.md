# NHIỆM VỤ 06: XẾP HÀNG LÊN XE VÀ TỐI ƯU TẢI TRỌNG (TRUCK LOADING & KNAPSACK OPTIMIZATION)

---

## 1. TỔNG QUAN NHIỆM VỤ (OVERVIEW)
- **Mục tiêu**: Giải quyết trọn vẹn quy trình bốc xếp hàng hóa lên phương tiện vận chuyển (xe tải hoặc thùng hàng xe máy của shipper) thỏa mãn 2 điều kiện:
  1. **Giới hạn tải trọng (Capacity Constraint)**: Phương tiện chỉ chở được tối đa khối lượng $W_{max}$. Cần chọn tập đơn hàng có tổng giá trị giao dịch hoặc mức độ cấp thiết lớn nhất sao cho tổng khối lượng không vượt quá $W_{max}$ (Bài toán Cái túi 0/1 - 0/1 Knapsack Problem).
  2. **Thứ tự xếp hàng vật lý (Physical Loading Order - LIFO)**: Kiện hàng giao sau phải xếp vào sâu trong thùng xe trước; kiện hàng giao đầu tiên phải nằm ngay cửa thùng xe để bốc dỡ thuận tiện nhất mà không cần đảo lộn toàn bộ hàng trong thùng (Ngăn xếp - Stack).
- **Yêu cầu cốt lõi**:
  - Tự cài đặt thuật toán **Quy hoạch động giải bài toán Cái túi 0/1** (0/1 Knapsack DP).
  - Tự cài đặt cấu trúc dữ liệu **Ngăn xếp (Stack - LIFO)** từ đầu.
  - Kết nối thứ tự giao hàng từ module Lập lộ trình (`route_planner.py`) để xếp hàng vào thùng xe theo đúng thứ tự đảo ngược.

---

## 2. VAI TRÒ TRONG DỰ ÁN VÀ BÁO CÁO CHUNG
- **Đảm bảo tính khả thi thực tế và an toàn vận chuyển**: Một thuật toán tìm đường dù tối ưu đến mấy cũng trở nên vô nghĩa nếu xe bị chở quá tải trọng cho phép (vi phạm luật giao thông hoặc hỏng xe) hoặc shipper phải mất 30 phút dỡ hết thùng hàng ra lề đường chỉ để lấy một bưu kiện nằm dưới đáy.
- **Cầu nối giữa Kế hoạch lộ trình và Thực thi thực tế**: Nhận kết quả lộ trình từ `route_planner.py` và đưa vào ngăn xếp xếp dỡ.
- **Trong báo cáo LaTeX**:
  - Trình bày tại Chương 2 (Ngăn xếp Stack) và Chương 3 (Quy hoạch động giải Knapsack).
  - Phân tích chi tiết công thức truy hồi quy hoạch động và bảng phương án.

---

## 3. CẤU TRÚC THƯ MỤC VÀ LIÊN KẾT FILE
```text
project-root/
│
├── truck_loading.py        # File mã nguồn chính chứa Stack & Knapsack DP
├── route_planner.py        # Nhận thứ tự các điểm dừng giao hàng
├── order_queue.py          # Lấy danh sách đơn hàng cần bốc xếp
├── Docs/
│   ├── task.md
│   ├── task_6_truck_loading.md
│   └── ...
└── test.py                 # Kiểm thử bài toán cái túi và quy trình bốc dỡ
```

---

## 4. CÔNG NGHỆ, PHƯƠNG PHÁP & SO SÁNH ĐÁNH GIÁ

### 4.1. Các phương pháp được lựa chọn
1. **Bài toán chọn hàng - Quy hoạch động Cái túi 0/1 (0/1 Knapsack DP)**:
   - Đặt $DP[i][w]$ là giá trị lớn nhất có thể đạt được khi xét $i$ đơn hàng đầu tiên với tải trọng tối đa $w$.
   - Công thức truy hồi:
     $$DP[i][w] = \max(DP[i-1][w], DP[i-1][w - w_i] + v_i) \quad \text{với } w \ge w_i$$
   - Độ phức tạp thời gian: $O(N \cdot W_{max})$.
   - Độ phức tạp không gian: $O(W_{max})$ (tối ưu hóa mảng 1 chiều).
2. **Cấu trúc xếp dỡ vật lý - Ngăn xếp tự cài đặt (Custom Stack - LIFO)**:
   - Các kiện hàng được đẩy (`push`) vào thùng xe theo thứ tự ngược lại của lộ trình giao. Khi đến điểm giao, chỉ cần lấy (`pop`) đúng kiện hàng trên cùng ra.
   - Thao tác `push`, `pop`, `peek`: đều đạt $O(1)$.

### 4.2. So sánh với các phương pháp khác & Lý do không chọn
| Bài toán | Phương pháp so sánh | Đánh giá | Lý do không chọn |
| :--- | :--- | :--- | :--- |
| **Chọn tải trọng** | **Vét cạn ($2^N$)** | Kiểm tra toàn bộ $2^N$ tập con | Quá chậm, với 50 đơn hàng tiềm năng thì $2^{50} \approx 1.12 \times 10^{15}$ phép tính $\implies$ không thể thực thi. |
| **Chọn tải trọng** | **Tham lam theo tỷ lệ giá/khối lượng ($v_i / w_i$)** | Nhanh ($O(N \log N)$) | Chỉ giải quyết tối ưu cho bài toán Cái túi dạng phân số (Fractional Knapsack). Đối với kiện hàng nguyên vẹn (0/1 Knapsack), tham lam có thể cho kết quả sai lệch rất xa nghiệm tối ưu. |
| **Quản lý bốc xếp** | **Hàng đợi FIFO** | $O(1)$ | Sai lệch quy luật vật lý: thùng xe tải chỉ có 1 cửa sau. Nếu dùng FIFO (vào trước ra trước) thì kiện cần lấy lại nằm sâu tận đầu xe, bắt buộc phải bốc toàn bộ hàng ra ngoài. |

---

## 5. CÁC THUẬT TOÁN & CẤU TRÚC DỮ LIỆU CẦN CÀI ĐẶT
1. **Cấu trúc Ngăn xếp (`CustomStack`)**:
   - Tự xây dựng bằng Danh sách liên kết đơn hoặc Mảng động tự co giãn.
   - Các phương thức: `push(item)`, `pop()`, `peek()`, `is_empty()`, `size()`.
2. **Thuật toán Quy hoạch động Knapsack 0/1 (`solve_knapsack_01`)**:
   - Nhận vào danh sách đơn hàng (mỗi đơn có khối lượng `weight` và giá trị/độ ưu tiên `value`) và giới hạn tải trọng xe `max_capacity`.
   - Tính toán bảng phương án và truy vết (Backtracking) để tìm chính xác danh sách các đơn hàng được chọn.
3. **Quy trình phối hợp bốc dỡ (`load_truck_for_route`)**:
   - Nhận danh sách đơn đã được sắp xếp theo lộ trình giao từ `route_planner.py`: $[D_1, D_2, \dots, D_k]$ ($D_1$ giao trước, $D_k$ giao sau cùng).
   - Đảo ngược thứ tự và lần lượt `push` vào thùng xe: `push(D_k) -> ... -> push(D_1)`.
   - Tại điểm đến, tài xế chỉ việc gọi `truck.pop()` để giao đúng đơn của khách hàng.

---

## 6. CHI TIẾT THIẾT KẾ FILE VÀ CÁC HÀM TRONG `truck_loading.py`

### 6.1. Cấu trúc Ngăn xếp
- `class StackNode`:
  - `data`: Kiện hàng / Đơn hàng.
  - `next`: Con trỏ tới phần tử bên dưới trong ngăn xếp.
- `class CustomStack`:
  - `push(self, order)`: Đặt kiện hàng lên đỉnh ngăn xếp. $O(1)$.
  - `pop(self)`: Lấy kiện hàng trên cùng ra giao khách. $O(1)$.
  - `peek(self)`: Xem kiện hàng trên cùng hiện tại là đơn nào. $O(1)$.
  - `is_empty(self)`: Kiểm tra thùng xe rỗng. $O(1)$.

### 6.2. Các hàm giải thuật
- `knapsack_01(orders, max_capacity)`:
  - Đầu vào: Danh sách đơn hàng `orders`, giới hạn tải trọng `max_capacity`.
  - Đầu ra: `(selected_orders, total_value, total_weight)`.
  - Độ phức tạp: Thời gian $O(N \cdot W)$, Bộ nhớ $O(W)$.
- `load_truck(truck_stack, ordered_stops)`:
  - Đẩy các đơn hàng vào thùng xe theo thứ tự nghịch đảo của lộ trình.
- `deliver_next_stop(truck_stack)`:
  - Rút đơn hàng trên cùng ra khỏi thùng xe khi đến điểm giao hàng.

---

## 7. KẾT QUẢ ĐẦU RA CẦN ĐẠT (DELIVERABLES)
1. **Mã nguồn `truck_loading.py`**:
   - Cài đặt hoàn chỉnh thuật toán Quy hoạch động Cái túi 0/1 (có truy vết nghiệm).
   - Cài đặt cấu trúc Stack thuần túy không dùng thư viện.
2. **Kiểm thử mô phỏng**:
   - Thử nghiệm với xe tải có tải trọng 500 kg và danh sách 30 đơn hàng với trọng lượng và giá trị khác nhau.
   - Mô phỏng quá trình shipper lần lượt ghé từng điểm và lấy hàng ra khỏi thùng xe theo cơ chế LIFO.
3. **Báo cáo LaTeX**:
   - Giải thích bảng quy hoạch động và sơ đồ minh họa cơ chế bốc dỡ hàng theo ngăn xếp (Stack).

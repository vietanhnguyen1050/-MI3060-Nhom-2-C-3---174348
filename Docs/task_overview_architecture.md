# KIẾN TRÚC TỔNG THỂ DỰ ÁN VÀ QUY TRÌNH TÍCH HỢP HỆ THỐNG
## HỆ THỐNG ĐIỀU PHỐI GIAO HÀNG (LOGISTICS) NỘI THÀNH

---

## 1. TỔNG QUAN HỆ THỐNG (SYSTEM OVERVIEW)
Dự án mô phỏng và giải quyết bài toán cốt lõi trong ngành Logistics đô thị: từ khâu tiếp nhận đơn hàng, quản lý hàng chờ, phân công tài xế, tối ưu hóa cung đường giao đa điểm, cho đến xếp dỡ vật lý lên phương tiện và theo dõi trạng thái đơn hàng thời gian thực.

Toàn bộ hệ thống được xây dựng trên nguyên tắc: **Tự thiết kế và cài đặt tất cả các Cấu trúc Dữ liệu & Giải thuật nền tảng bằng ngôn ngữ Python thuần, tuyệt đối không phụ thuộc vào các thư viện cấu trúc dữ liệu có sẵn.**

---

## 2. BẢN ĐỒ PHÂN CHIA NHIỆM VỤ (TASK ROADMAP)

Hệ thống được chia thành 7 nhiệm vụ chuyên biệt, tương ứng với 7 file đặc tả chi tiết trong thư mục `Docs/`:

| STT | File Tài Liệu | File Mã Nguồn | Vai Trò Chính | Cấu Trúc Dữ Liệu & Giải Thuật Cốt Lõi |
| :---: | :--- | :--- | :--- | :--- |
| **1** | [task_1_graph.md](file:///d:/OneDrive/Tài%20liệu/GitHub/-MI3060-Nhom-2-C-3---174348/Docs/task_1_graph.md) | `graph.py` | Mô hình bản đồ giao thông | Danh sách kề tự cài đặt, Đồ thị có trọng số |
| **2** | [task_2_shortest_path.md](file:///d:/OneDrive/Tài%20liệu/GitHub/-MI3060-Nhom-2-C-3---174348/Docs/task_2_shortest_path.md) | `shortest_path.py` | Tìm đường ngắn nhất kho - khách | Thuật toán Dijkstra, Min-Heap tự cài đặt |
| **3** | [task_3_shipper_assign.md](file:///d:/OneDrive/Tài%20liệu/GitHub/-MI3060-Nhom-2-C-3---174348/Docs/task_3_shipper_assign.md) | `shipper_assign.py` | Phân công tài xế gần nhất | Thuật toán BFS / Dijkstra dừng sớm, Queue tự cài đặt |
| **4** | [task_4_route_planner.md](file:///d:/OneDrive/Tài%20liệu/GitHub/-MI3060-Nhom-2-C-3---174348/Docs/task_4_route_planner.md) | `route_planner.py` | Lộ trình giao đa điểm (TSP) | Tham lam Nearest Neighbor, Quy hoạch động Held-Karp |
| **5** | [task_5_order_queue.md](file:///d:/OneDrive/Tài%20liệu/GitHub/-MI3060-Nhom-2-C-3---174348/Docs/task_5_order_queue.md) | `order_queue.py` | Hàng đợi đơn hỏa tốc / thường | FIFO Queue liên kết, Priority Queue (Max-Heap) |
| **6** | [task_6_truck_loading.md](file:///d:/OneDrive/Tài%20liệu/GitHub/-MI3060-Nhom-2-C-3---174348/Docs/task_6_truck_loading.md) | `truck_loading.py` | Tải trọng xe & Xếp dỡ LIFO | Bài toán Cái túi 0/1 (DP), Ngăn xếp Stack tự cài đặt |
| **7** | [task_7_order_tracking.md](file:///d:/OneDrive/Tài%20liệu/GitHub/-MI3060-Nhom-2-C-3---174348/Docs/task_7_order_tracking.md) | `order_tracking.py` | Tra cứu $O(1)$ & Ghi vết lịch sử | Bảng băm (Separate Chaining), Danh sách liên kết |

---

## 3. SƠ ĐỒ LUỒNG HOẠT ĐỘNG TOÀN HỆ THỐNG (END-TO-END WORKFLOW)

```
[1. Khách đặt hàng] 
        │
        ▼
[order_tracking.py] ── Tạo mã đơn & ghi nhận trạng thái CREATED vào Bảng Băm
        │
        ▼
[order_queue.py] ──── Đưa vào Priority Queue (Hỏa tốc) hoặc FIFO Queue (Tiêu chuẩn)
        │
        ▼
[shipper_assign.py] ─ Quét tìm tài xế rảnh gần nhất bằng BFS / Dijkstra từ vị trí kho/lấy hàng
        │
        ▼
[route_planner.py] ── Gom các đơn cho tài xế, chạy TSP (Tham lam / QHĐ) ra lộ trình tối ưu: D1 -> D2 -> D3
        │
        ▼
[truck_loading.py] ── Kiểm tra tải trọng xe (Knapsack DP), xếp hàng vào Thùng xe Stack (LIFO: push D3 -> D2 -> D1)
        │
        ▼
[Thực hiện giao hàng]─ Đến điểm D1: Rút đỉnh Stack (pop D1), cập nhật trạng thái DELIVERED vào Bảng băm
```

---

## 4. TỔ CHỨC FILE `main.py` VÀ `test.py`
- **`main.py`**: Điểm khởi động toàn bộ kịch bản mô phỏng giao hàng tích hợp (End-to-End Pipeline), nhận dữ liệu đầu vào và kết nối lần lượt tất cả 7 module.
- **`test.py`**: Chứa toàn bộ các bài benchmark hiệu năng:
  - So sánh tốc độ Min-Heap tự viết với mảng tuyến tính.
  - So sánh thời gian chạy và sai số giữa TSP Tham lam và TSP Quy hoạch động.
  - Thống kê tỷ lệ va chạm của Bảng băm khi lưu $100.000$ đơn hàng.
  - Xuất số liệu và biểu đồ để phục vụ viết Báo cáo LaTeX.

Giao hàng (logistics) nội thành
Đồ thị/Dữ liệu
1. Bản đồ khu vực giao hàng: đồ thị có trọng số (khoảng cách hoặc thời gian).

Tìm đường thuật toán Dijkstra
2. Đường ngắn nhất từ kho đến khách: Dijkstra.
3. Phân công shipper gần nhất: BFS hoặc Dijkstra từ vị trí đơn hàng.

4. Lộ trình giao nhiều điểm trong một chuyến: tham lam (láng giềng gần nhất), hoặc quay lui/quy hoạch động khi số điểm nhỏ. 
5. Hàng đợi đơn chờ xử lý: queue; đơn hỏa tốc dùng priority queue (heap). 
6. Xếp hàng lên xe: stack (giao sau xếp trước); bài toán cái túi cho giới hạn tải trọng. 
7. Tra cứu và theo dõi trạng thái đơn: bảng băm theo mã đơn; danh sách liên kết lưu lịch sử trạng thái.

Yêu cầu chung:
● Tự cài đặt các cấu trúc dữ liệu chính, không dùng thư viện sẵn có cũng như các thuật toán sắp xếp, tìm kiếm, … 
● Phân tích độ phức tạp của các thao tác chính và giải thích lý do chọn cấu trúc đó. 
● Đọc/ghi dữ liệu từ file hoặc CSDL và có bộ dữ liệu kiểm thử đủ lớn để so sánh hiệu năng.
Task: Báo cáo Latex, code thuật toán bằng ngôn ngữ Python
# HƯỚNG DẪN SỬ DỤNG GIT & GITHUB
## Quy tắc version control cho nhóm 4 thành viên

> Mục tiêu: giúp cả nhóm làm việc trên cùng một repository, hạn chế ghi đè code của nhau và lưu lại lịch sử thay đổi rõ ràng.
>
> Quy ước của nhóm: **1 Owner quản lý repository, 3 thành viên còn lại là Collaborators. Mọi thay đổi đều được thực hiện trên branch riêng và gửi Pull Request để Owner review, merge vào `main`.**

---

## 1. Vai trò và trách nhiệm

| Vai trò | Trách nhiệm |
|---|---|
| Owner (1 người) | Tạo và quản lý repository; thêm Collaborators; thống nhất cấu trúc thư mục; review Pull Request; xử lý hoặc hỗ trợ xử lý conflict; merge code vào `main`. |
| Collaborators (3 người) | Clone repository; tạo branch riêng cho từng công việc; code và commit thường xuyên; push branch lên GitHub; tạo Pull Request để đề nghị merge. |

**Quy tắc chung:** Không ai push trực tiếp lên branch `main`. Kể cả Owner, nên dùng Pull Request để mọi thay đổi đều được kiểm tra và có lịch sử rõ ràng.

---

## 2. Cấu trúc thư mục đề xuất

Cấu trúc dưới đây chỉ là mẫu ban đầu. Nhóm có thể điều chỉnh theo yêu cầu của đề tài và các thuật toán cần triển khai.

```text
algorithm-report/
│
├── README.md
├── .gitignore
│
├── docs/                   # Báo cáo, tài liệu, hình ảnh minh họa
│   ├── report/
│   └── references/
│
├── src/                    # Mã nguồn
│   ├── algorithm_1/
│   ├── algorithm_2/
│   └── algorithm_3/
│
├── tests/                  # Test case và dữ liệu kiểm thử
│
└── presentation/            # Slide thuyết trình (nếu có)
```

### Quy tắc tổ chức file

- Mỗi thuật toán nên có thư mục riêng nếu phần cài đặt tương đối độc lập.
- Đặt tên file và thư mục rõ nghĩa, thống nhất cách viết; ví dụ: `dijkstra.ts`, `binary_search.py`.
- Không tạo nhiều bản sao như `final`, `final_v2`, `final_really_final`.
- Không commit file sinh ra tự động, file tạm, thư mục build hoặc thông tin cá nhân/mật khẩu.
- File báo cáo nên có một bản chính thức duy nhất để tránh nhiều người sửa các bản khác nhau.

---

## 3. Các khái niệm Git cần biết

| Thuật ngữ | Ý nghĩa đơn giản |
|---|---|
| Repository (Repo) | Kho lưu trữ toàn bộ mã nguồn và lịch sử thay đổi của dự án. |
| Clone | Tải repository từ GitHub về máy lần đầu. |
| Branch | Nhánh làm việc riêng, giúp phát triển tính năng mà không ảnh hưởng trực tiếp đến `main`. |
| Commit | Lưu lại một nhóm thay đổi vào lịch sử Git trên máy. |
| Push | Đẩy commit từ máy lên GitHub. |
| Pull | Lấy thay đổi mới từ GitHub về máy và tích hợp vào branch hiện tại. |
| Pull Request (PR) | Yêu cầu đưa thay đổi từ branch này vào branch khác, thường là `main`. |
| Merge | Gộp thay đổi từ một branch vào branch khác. |
| Conflict | Xung đột khi Git không thể tự xác định cách kết hợp các thay đổi. |

**Phân biệt nhanh:**
- `commit`: lưu thay đổi ở máy local.
- `push`: gửi commit lên GitHub.
- `pull`: lấy thay đổi từ GitHub về.
- `merge`: gộp nội dung giữa hai branch.

---

## 4. Quy trình làm việc tiêu chuẩn

Nhóm sử dụng quy trình sau cho mỗi công việc hoặc thuật toán:

```text
                 GitHub
                   |
             branch: main
                   |
          Thành viên tạo branch
                   |
          Code -> Commit -> Push
                   |
             Pull Request
                   |
           Owner review và kiểm tra
                   |
              Merge vào main
                   |
          Các thành viên Pull bản mới
```

### Bước 1: Clone repository (chỉ thực hiện lần đầu)

Owner gửi đường dẫn repository GitHub cho các thành viên.

Mở Terminal tại thư mục muốn lưu dự án và chạy:

```bash
git clone <repository-url>
```

Ví dụ:

```bash
git clone https://github.com/username/algorithm-report.git
```

Di chuyển vào thư mục dự án:

```bash
cd algorithm-report
```

### Bước 2: Cập nhật `main` trước khi bắt đầu công việc

Trước khi tạo branch mới, lấy phiên bản mới nhất từ GitHub:

```bash
git switch main
git pull origin main
```

Việc này giúp branch mới được tạo từ phiên bản `main` mới nhất.

### Bước 3: Tạo branch riêng

Mỗi thành viên làm việc trên branch riêng, đặt tên theo nội dung công việc.

Cú pháp:

```bash
git switch -c <branch-name>
```

Ví dụ:

```bash
git switch -c feature/dijkstra
git switch -c feature/bfs
git switch -c docs/report-introduction
git switch -c fix/dijkstra-test
```

Quy ước đặt tên branch:

| Loại công việc | Tiền tố | Ví dụ |
|---|---|---|
| Phát triển chức năng/thuật toán | `feature/` | `feature/dijkstra` |
| Sửa lỗi | `fix/` | `fix/bfs-error` |
| Tài liệu/báo cáo | `docs/` | `docs/report-introduction` |
| Cải thiện, chỉnh sửa code | `refactor/` | `refactor/graph-structure` |

**Lưu ý:** Tạo branch mới từ `main` đã được cập nhật. Không dùng chung một branch cho nhiều công việc không liên quan.

### Bước 4: Code và kiểm tra thay đổi

Sau khi chỉnh sửa file, kiểm tra trạng thái:

```bash
git status
```

Xem những dòng đã thay đổi:

```bash
git diff
```

Chạy chương trình và kiểm tra phần mình vừa làm trước khi commit.

### Bước 5: Commit thay đổi

Đưa các file muốn lưu vào staging area:

```bash
git add <file-name>
```

Hoặc thêm tất cả file đã thay đổi:

```bash
git add .
```

Tạo commit:

```bash
git commit -m "feat: implement Dijkstra algorithm"
```

Một số quy ước commit message:

| Tiền tố | Sử dụng khi | Ví dụ |
|---|---|---|
| `feat:` | Thêm chức năng hoặc thuật toán | `feat: implement Dijkstra algorithm` |
| `fix:` | Sửa lỗi | `fix: correct BFS traversal` |
| `docs:` | Cập nhật tài liệu | `docs: add algorithm explanation` |
| `refactor:` | Cải thiện cấu trúc code | `refactor: simplify graph structure` |
| `test:` | Thêm hoặc sửa kiểm thử | `test: add Dijkstra test cases` |

Một commit nên tập trung vào **một nội dung thay đổi có liên quan**. Tránh commit chung chung như `update`, `fix`, `abc`, `final`.

### Bước 6: Push branch lên GitHub

```bash
git push -u origin <branch-name>
```

Ví dụ:

```bash
git push -u origin feature/dijkstra
```

Lần push đầu tiên của branch dùng `-u` để thiết lập nhánh theo dõi. Những lần sau trên cùng branch chỉ cần:

```bash
git push
```

### Bước 7: Tạo Pull Request (PR)

Sau khi push, vào repository trên GitHub:

1. Mở tab **Pull requests**.
2. Chọn **New pull request** (hoặc chọn nút tạo PR GitHub gợi ý).
3. Chọn:
   - **Base:** `main` — branch nhận thay đổi.
   - **Compare:** branch của bạn, ví dụ `feature/dijkstra`.
4. Đặt tiêu đề rõ ràng, ví dụ: `feat: implement Dijkstra algorithm`.
5. Mô tả ngắn gọn những gì đã làm, file/thư mục bị ảnh hưởng và cách kiểm tra.
6. Chọn Owner làm reviewer và tạo Pull Request.

Owner sẽ kiểm tra code, yêu cầu chỉnh sửa nếu cần và merge PR khi đạt yêu cầu.

**Không tự merge PR của mình.** Nếu Owner yêu cầu chỉnh sửa, tiếp tục sửa trên chính branch đó, sau đó commit và push. Pull Request sẽ tự cập nhật.

### Bước 8: Cập nhật `main` sau khi PR được merge

Sau khi PR được Owner merge, mọi thành viên cập nhật bản mới nhất:

```bash
git switch main
git pull origin main
```

Nếu muốn tiếp tục công việc mới, tạo branch mới từ `main` vừa cập nhật.

---

## 5. Khi đang làm mà `main` có thay đổi mới

Nếu một thành viên đang làm trên branch riêng và `main` đã có commit mới, nên cập nhật branch của mình trước khi tạo PR hoặc trước khi tiếp tục công việc.

Ví dụ đang ở branch `feature/dijkstra`:

```bash
git switch main
git pull origin main

git switch feature/dijkstra
git merge main
```

Lệnh cuối đưa các thay đổi mới từ `main` vào branch đang làm.

Nếu có conflict, xem hướng dẫn ở mục 7. Sau khi xử lý và kiểm tra lại, push branch:

```bash
git push
```

**Không chạy `git pull` một cách máy móc khi đang ở nhầm branch.** Hãy kiểm tra branch hiện tại bằng `git branch` hoặc `git status` trước.

---

## 6. Các lệnh Git thường dùng

| Lệnh | Tác dụng |
|---|---|
| `git status` | Kiểm tra branch hiện tại và trạng thái file. |
| `git branch` | Liệt kê các branch local. |
| `git branch -a` | Liệt kê branch local và remote. |
| `git switch main` | Chuyển sang branch `main`. |
| `git switch <branch>` | Chuyển sang branch khác. |
| `git pull origin main` | Lấy và tích hợp thay đổi từ `main` trên GitHub. |
| `git add .` | Đưa các thay đổi vào staging area. |
| `git commit -m "message"` | Tạo commit. |
| `git push` | Đẩy commit lên remote branch đã liên kết. |
| `git fetch origin` | Lấy thông tin cập nhật từ GitHub nhưng chưa tích hợp vào branch hiện tại. |
| `git merge main` | Gộp `main` vào branch hiện tại. |
| `git log --oneline` | Xem lịch sử commit dạng ngắn. |
| `git diff` | Xem nội dung thay đổi chưa được đưa vào staging area. |

---

## 7. Xử lý Conflict (xung đột)

Conflict thường xảy ra khi hai người cùng chỉnh sửa một phần của cùng một file và Git không thể tự kết hợp thay đổi.

Ví dụ Git thông báo conflict trong `src/dijkstra/index.ts`.

### Cách xử lý

1. Chạy `git status` để xem file đang bị conflict.
2. Mở file được thông báo. Git có thể hiển thị:

```text
<<<<<<< HEAD
// Nội dung hiện tại trên branch của bạn
=======
// Nội dung được merge từ branch khác
>>>>>>> main
```

3. Trao đổi với người viết phần code còn lại nếu chưa rõ nên giữ logic nào.
4. Chỉnh sửa file để tạo ra nội dung cuối cùng đúng. Xóa các dấu đánh dấu conflict (`<<<<<<<`, `=======`, `>>>>>>>`).
5. Kiểm tra lại code và chạy chương trình/test.
6. Đánh dấu file đã xử lý:

```bash
git add src/dijkstra/index.ts
```

7. Hoàn tất merge:

```bash
git commit
```

Nếu Git mở trình soạn thảo để nhập nội dung merge commit, lưu và đóng trình soạn thảo để hoàn tất.

**Quan trọng:** Không chọn giữ toàn bộ nội dung của một bên chỉ để hết conflict. Cần đọc và kết hợp logic đúng. Nếu không chắc chắn, hỏi người phụ trách phần code đó hoặc Owner.

Nếu muốn hủy một lần merge đang xử lý và quay lại trạng thái trước khi merge:

```bash
git merge --abort
```

Chỉ dùng lệnh này khi merge vẫn đang diễn ra và bạn thực sự muốn hủy thao tác.

---

## 8. Quy tắc làm việc nhóm

### 8.1. Quy tắc bắt buộc

1. **Không push trực tiếp lên `main`.** Mọi thay đổi phải thông qua Pull Request.
2. **Mỗi người làm trên branch riêng.** Không code trực tiếp trên `main`.
3. **Pull `main` trước khi bắt đầu công việc mới** để tránh làm trên phiên bản cũ.
4. **Commit thường xuyên**, mỗi commit nên tương ứng với một phần công việc có ý nghĩa.
5. **Push branch lên GitHub** để lưu thay đổi và giúp các thành viên khác theo dõi.
6. **Không tự ý sửa code của người khác** khi chưa trao đổi, đặc biệt nếu đang làm trên cùng một file.
7. **Kiểm tra chương trình trước khi tạo PR.** Không đưa code chưa chạy được hoặc đang lỗi vào `main`.
8. **Đọc kỹ thay đổi trước khi merge.** Owner cần kiểm tra nội dung PR, không merge chỉ vì thành viên đã gửi yêu cầu.
9. **Không commit thông tin nhạy cảm** như mật khẩu, API key, token hoặc file chứa thông tin cá nhân.
10. **Không dùng `git push --force` hoặc `git reset --hard` nếu chưa hiểu rõ hậu quả.** Các lệnh này có thể làm mất hoặc ghi đè công việc.

### 8.2. Quy tắc phối hợp

- Trước khi bắt đầu, thống nhất ai phụ trách thuật toán nào, ai phụ trách báo cáo và ai phụ trách kiểm thử.
- Nếu cần sửa một file mà người khác đang phụ trách, trao đổi trước để tránh ghi đè hoặc conflict.
- Khi hoàn thành một phần việc, thông báo cho nhóm và gửi PR sớm; không đợi đến sát hạn mới push toàn bộ.
- Nếu gặp lỗi Git hoặc conflict không hiểu cách xử lý, dừng lại và hỏi Owner. Không thử các lệnh xóa/reset một cách ngẫu nhiên.
- Khi PR được merge, các thành viên cập nhật `main` trước khi tạo branch cho công việc tiếp theo.

### 8.3. Những việc không nên làm

| Không nên | Vì sao |
|---|---|
| Code trực tiếp trên `main` | Có thể làm ảnh hưởng phiên bản chung của nhóm. |
| Nhiều người cùng sửa một file mà không trao đổi | Dễ xảy ra conflict hoặc ghi đè nội dung. |
| Commit tất cả vào cuối dự án | Khó theo dõi lịch sử và khó khôi phục khi có lỗi. |
| Đặt commit message như `update` hoặc `abc` | Không biết commit đã thay đổi nội dung gì. |
| Push file `.env`, mật khẩu hoặc token | Có nguy cơ lộ thông tin nhạy cảm. |
| Tự ý force push | Có thể ghi đè lịch sử và commit của người khác. |
| Merge PR khi chưa kiểm tra | Có thể đưa code lỗi vào `main`. |

---

## 9. Checklist trước khi tạo Pull Request

Trước khi gửi PR, mỗi thành viên tự kiểm tra:

- [ ] Đang làm trên đúng branch, không phải `main`.
- [ ] Đã cập nhật branch với `main` mới nhất khi cần.
- [ ] Code đã được chạy thử.
- [ ] Không đưa file tạm, file build hoặc thông tin nhạy cảm vào commit.
- [ ] Commit message mô tả đúng nội dung thay đổi.
- [ ] Đã push branch lên GitHub.
- [ ] Pull Request có tiêu đề và mô tả rõ ràng.

Owner kiểm tra trước khi merge:

- [ ] PR có đúng base branch là `main`.
- [ ] Nội dung thay đổi đúng với yêu cầu.
- [ ] Không có thay đổi ngoài phạm vi hoặc file không cần thiết.
- [ ] Code chạy được và không gây lỗi rõ ràng.
- [ ] Conflict đã được xử lý.
- [ ] Đã trao đổi với người thực hiện nếu cần chỉnh sửa.

---

## 10. Quy trình mẫu cần nhớ

Mỗi khi bắt đầu một công việc mới, thành viên có thể dùng chuỗi lệnh sau:

```bash
# 1. Chuyển về main và cập nhật
git switch main
git pull origin main

# 2. Tạo branch mới
git switch -c feature/ten-cong-viec

# 3. Kiểm tra thay đổi
git status

# 4. Đưa thay đổi vào staging
git add .

# 5. Commit
git commit -m "feat: mô tả công việc"

# 6. Push branch lên GitHub
git push -u origin feature/ten-cong-viec
```

Sau đó tạo Pull Request trên GitHub và chờ Owner review, merge.

Khi PR đã được merge:

```bash
# Cập nhật main mới nhất
git switch main
git pull origin main
```

Từ đây, thành viên có thể bắt đầu công việc tiếp theo bằng cách tạo branch mới.

---

## TÓM TẮT QUY TẮC CỦA NHÓM

```text
1. Luôn cập nhật main trước khi bắt đầu.
2. Tạo branch riêng cho từng công việc.
3. Code -> git add -> git commit.
4. Push branch lên GitHub.
5. Tạo Pull Request vào main.
6. Owner review và merge.
7. Mọi người pull main mới nhất.
8. Lặp lại quy trình cho công việc tiếp theo.
```

**Nguyên tắc quan trọng nhất:** `main` là phiên bản chung của cả nhóm. Mỗi thành viên phát triển trên branch riêng; chỉ đưa code vào `main` thông qua Pull Request đã được kiểm tra.

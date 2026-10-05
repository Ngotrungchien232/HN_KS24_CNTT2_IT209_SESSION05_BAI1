# BÁO CÁO BÀI TẬP 1: KHÔI PHỤC COMMIT ĐÃ MẤT BẰNG GIT REFLOG

- **Họ và tên học viên:** [Điền họ và tên của bạn]
- **Lớp / Khóa học:** DevOps - Session 05
- **Bài tập:** Bài 1 - Khôi phục commit bằng Git Reflog

---

## 1. Mục tiêu bài tập
- Hiểu được cơ chế lưu trữ lịch sử hoạt động cục bộ của Git thông qua **Reflog** (Reference Logs).
- Sử dụng thành thạo lệnh `git reflog` để truy vết các commit hoặc con trỏ `HEAD` cũ.
- Khôi phục thành công commit và mã nguồn đã bị mất khỏi nhánh làm việc sau khi thực hiện lệnh `git reset --hard HEAD~1` mà không cần viết lại mã thủ công.

---

## 2. Bối cảnh và Kịch bản giả lập sự cố
- Trong quá trình phát triển, học viên đã hoàn thành tính năng quan trọng và commit vào kho lưu trữ cục bộ.
- Do nhầm lẫn, học viên thực thi lệnh:
  ```bash
  git reset --hard HEAD~1
  ```
- **Hậu quả:** 
  - Con trỏ `HEAD` và nhánh hiện tại bị lùi lại 1 commit.
  - Commit chứa tính năng mới biến mất hoàn toàn khỏi danh sách `git log`.
  - Toàn bộ thay đổi trong working directory (tệp mã nguồn vừa tạo) bị xóa sạch.

---

## 3. Các bước thực hiện và kết quả

### Bước 1: Khởi tạo repository và tạo commit ban đầu
```bash
git init
echo "Phien ban khoi tao du an" > init.txt
git add .
git commit -m "Khoi tao du an"
```

### Bước 2: Thêm tính năng quan trọng và commit
Tạo tệp `feature.txt` có nội dung `"Day la tinh nang quan trong"`:
```bash
echo "Day la tinh nang quan trong" > feature.txt
git add .
git commit -m "Them tinh nang quan trong"
```

Kiểm tra lịch sử commit trước sự cố:
```bash
git log --oneline
```
*Kết quả ghi nhận:*
```text
4339f86 (HEAD -> master) Them tinh nang quan trong
99d953b Khoi tao du an
```

---

### Bước 3: Giả lập sự cố mất commit
Thực hiện lùi con trỏ và xóa working tree:
```bash
git reset --hard HEAD~1
```
*Thông báo:*
```text
HEAD is now at 99d953b Khoi tao du an
```

Kiểm tra lại `git log`:
```bash
git log --oneline
```
*Kết quả:*
```text
99d953b (HEAD -> master) Khoi tao du an
```
> **Nhận xét:** Commit `"Them tinh nang quan trong"` đã không còn hiển thị trong `git log`. Tệp `feature.txt` đã bị xóa khỏi thư mục làm việc.

---

### Bước 4: Truy vết commit bằng Git Reflog
Thực hiện tra cứu lịch sử thay đổi tham chiếu:
```bash
git reflog
```
*Kết quả hiển thị:*
```text
99d953b (HEAD -> master) HEAD@{0}: reset: moving to HEAD~1
4339f86 HEAD@{1}: commit: Them tinh nang quan trong
99d953b HEAD@{2}: commit (initial): Khoi tao du an
```
> **Phát hiện:** Tại con trỏ `HEAD@{1}`, mã hash `4339f86` chính là commit `"Them tinh nang quan trong"` trước khi lệnh `reset` được thực thi.

---

### Bước 5: Khôi phục commit đã mất
Sử dụng mã hash tìm được từ Reflog để đưa con trỏ `HEAD` và working directory trở lại trạng thái của commit đó:
```bash
git reset --hard 4339f86
```
*(Hoặc có thể sử dụng: `git reset --hard HEAD@{1}`)*

*Thông báo:*
```text
HEAD is now at 4339f86 Them tinh nang quan trong
```

---

### Bước 6: Kiểm tra kết quả sau khi khôi phục
1. Kiểm tra lại lịch sử commit:
```bash
git log --oneline
```
*Kết quả:*
```text
4339f86 (HEAD -> master) Them tinh nang quan trong
99d953b Khoi tao du an
```
Commit quan trọng đã quay trở lại đầu nhánh.

2. Kiểm tra tệp tin và nội dung:
```bash
cat feature.txt
# Hoặc trên PowerShell: Get-Content feature.txt
```
*Nội dung:*
```text
Day la tinh nang quan trong
```
Dữ liệu đã được phục hồi đầy đủ 100%.

---

## 4. Giải thích cơ chế kỹ thuật
1. **Tại sao `git log` không thấy nhưng `git reflog` lại thấy?**
   - `git log` chỉ duyệt qua các commit có thể đi tới được (reachable commits) từ con trỏ `HEAD` hoặc nhánh hiện tại đi ngược về quá khứ theo đồ thị DAG. Khi ta reset về `HEAD~1`, commit sau nó trở thành "dangling commit" (commit mồ côi) nên `git log` bỏ qua.
   - `git reflog` theo dõi mọi hành động làm dịch chuyển con trỏ `HEAD` trên máy cục bộ (như `commit`, `checkout`, `reset`, `merge`, `rebase`). Dù commit bị tách khỏi nhánh, reflog vẫn giữ lại lịch sử tham chiếu trong một khoảng thời gian (mặc định 30 - 90 ngày) trước khi Garbage Collection (`git gc`) thu dọn.

2. **Bài học kinh nghiệm:**
   - Trong Git cục bộ, gần như mọi commit đã được tạo đều có thể cứu được nhờ `git reflog`.
   - Cần cẩn trọng khi sử dụng cờ `--hard` vì nó thay đổi trực tiếp cả `working directory`.

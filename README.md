# Bài 2 — Quản lý nhánh và Giải quyết xung đột (Merge Conflict)

> **Session 04 — Exercise 2**
> Nhánh phụ: `feature-update` · Nhánh chính: `main` · File xảy ra xung đột: `README.md`

---

## 1. Bối cảnh bài toán

Trong lúc phát triển tính năng mới trên nhánh phụ `feature-update`, nhánh chính `main`
cũng được cập nhật. Cả hai nhánh cùng chỉnh sửa **cùng một dòng** trong `README.md`,
dẫn đến **xung đột gộp nhánh (merge conflict)** khi gộp lại. Bài yêu cầu **không dùng
công cụ tự động gộp**, mà phải xử lý thủ công các ký hiệu đánh dấu xung đột.

---

## 2. Các lệnh đã thực hiện

```bash
# 1) Khởi tạo dự án và commit gốc trên main
git init -b main
git add .
git commit -m "chore: khoi tao du an"

# 2) Tạo nhánh phụ và phát triển tính năng
git checkout -b feature-update
#    -> sửa README.md (dòng tính năng)
git add README.md
git commit -m "feat: them tinh nang dang nhap bang email"

# 3) Quay về main và cập nhật cùng dòng đó theo hướng khác
git checkout main
#    -> sửa README.md (cùng dòng, nội dung khác)
git add README.md
git commit -m "feat: them tinh nang quen mat khau tren main"

# 4) Gộp nhánh -> phát sinh xung đột
git merge feature-update
#    CONFLICT (content): Merge conflict in README.md

# 5) Giải quyết thủ công (mục 3), sau đó hoàn tất commit gộp
git add README.md
git commit -m "merge: gop feature-update vao main, giai quyet xung dot README.md"

# 6) Kiểm tra lịch sử dạng đồ thị nhánh
git log --graph --oneline
```

---

## 3. Giải quyết xung đột thủ công

Sau lệnh `git merge feature-update`, Git đánh dấu file `README.md` ở trạng thái **`UU` (both modified)**
và chèn các ký hiệu xung đột vào file:

```text
# Du an hoc Git - Session 04

## Tinh nang dang phat trien
<<<<<<< HEAD
- Tinh nang: quen mat khau.
=======
- Tinh nang: dang nhap bang email.
>>>>>>> feature-update
```

**Ý nghĩa các ký hiệu:**

| Ký hiệu | Ý nghĩa |
| --- | --- |
| `<<<<<<< HEAD` | Bắt đầu vùng nội dung của nhánh hiện tại (`main`) — phiên bản "của mình" |
| `=======` | Vạch phân cách giữa hai phiên bản đang mâu thuẫn |
| `>>>>>>> feature-update` | Kết thúc vùng nội dung của nhánh đang được gộp vào (`feature-update`) |

**Các bước xử lý thủ công:**

1. **Mở file** `README.md` và xác định vùng bị đánh dấu xung đột.
2. **Quyết định nội dung cuối cùng**: cả hai tính năng đều cần giữ, nên tôi gộp cả hai dòng
   và đặt `dang nhap bang email` trước cho hợp lý về mặt nghiệp vụ.
3. **Xóa toàn bộ các dòng đánh dấu** `<<<<<<<`, `=======`, `>>>>>>>` — chỉ giữ lại nội dung đã chọn.
4. **Kiểm tra lại** không còn sót ký hiệu xung đột nào trong file.
5. **Đánh dấu đã giải quyết** bằng `git add README.md` (đưa file ra khỏi trạng thái `UU`).
6. **Hoàn tất commit gộp** bằng `git commit` — Git tự sinh merge commit với 2 nhánh tổ tiên.

**Nội dung `README.md` sau khi giải quyết xung đột:**

```markdown
# Du an hoc Git - Session 04

## Tinh nang dang phat trien
- Tinh nang: dang nhap bang email.
- Tinh nang: quen mat khau.
```

---

## 4. Cơ chế gộp 3 vùng (3-Way Merge)

Git không so sánh trực tiếp hai phiên bản, mà dùng **ba vùng** để quyết định gộp:

```
                (base) — tổ tiên chung lần gộp gần nhất
                - Trang chu co ban.
                   /            \
        (ours) main            (theirs) feature-update
   - quen mat khau.        - dang nhap bang email.
                   \            /
                (result) — nội dung sau khi gộp
```

- **Base (tổ tiên chung):** commit gần nhất mà cả hai nhánh cùng chia sẻ — ở đây là commit
  khởi tạo với dòng `- Trang chu co ban.`
- **Ours (HEAD / `main`):** phiên bản hiện tại — `- quen mat khau.`
- **Theirs (`feature-update`):** phiên bản đang gộp vào — `- dang nhap bang email.`

Git so sánh `base ↔ ours` và `base ↔ theirs` trên từng dòng:

- Nếu **chỉ một bên** thay đổi so với base → Git tự động lấy thay đổi đó (auto-merge).
- Nếu **cả hai bên** cùng thay đổi **trên cùng dòng** theo cách khác nhau → Git **không thể
  quyết định** nên phát sinh **xung đột** và đánh dấu để người dùng tự xử lý.

Trong bài này, cả `main` và `feature-update` đều sửa đúng dòng `- Trang chu co ban.` nên
rơi vào trường hợp xung đột — buộc phải giải quyết thủ công.

---

## 5. Kiểm tra — Lịch sử commit dạng đồ thị nhánh

Kết quả chạy `git log --graph --oneline --all`:

```text
*   0de3ac6 merge: gop feature-update vao main, giai quyet xung dot README.md
|\
| * af4e004 feat: them tinh nang dang nhap bang email
* | 1fd2a89 feat: them tinh nang quen mat khau tren main
|/
* 6d8119a chore: khoi tao du an
```

**Đọc đồ thị:**

- `6d8119a` là **commit gốc** (tổ tiên chung của cả hai nhánh).
- Nhánh `feature-update` **tách ra từ `main`** tại commit gốc (`af4e004`).
- `main` tiếp tục có commit riêng (`1fd2a89`) trong khi `feature-update` có commit (`af4e004`)
  — hai nhánh phát triển song song.
- `0de3ac6` là **merge commit** có **2 nhánh tổ tiên** (thể hiện qua ký hiệu `|\` và `|/`),
  đánh dấu thời điểm gộp `feature-update` trở lại `main` sau khi giải quyết xung đột.

Trạng thái nhánh:

```text
  feature-update
* main
```

### 📸 Hình minh chứng

> (Dán ảnh chụp màn hình kết quả `git log --graph --oneline` chạy trên máy của bạn vào đây.)

---

## 6. Kết luận

- Tạo và di chuyển linh hoạt giữa `main` và `feature-update` bằng `git checkout`/`git switch`.
- Cố ý tạo xung đột gộp nhánh trên cùng một dòng của `README.md`.
- Xử lý thủ công bằng cách xóa các ký hiệu `<<<<<<<`, `=======`, `>>>>>>>` và giữ nội dung đúng.
- Hoàn tất **merge commit** với 2 nhánh tổ tiên, hiểu rõ cơ chế gộp 3 vùng (base / ours / theirs).

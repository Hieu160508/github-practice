# Tổng hợp thực hành GitHub — Từ đầu đến cuối

Tài liệu này ghi lại toàn bộ flow bạn đã thực hành: tạo repo, push file, tạo nhánh, tạo Pull Request, merge, và dọn dẹp. Mỗi lệnh đều kèm giải thích để dùng làm tài liệu tra cứu lại sau này.

---

## Khái niệm nền tảng (đọc trước khi vào lệnh)

- **Local**: máy tính của bạn (Ubuntu). **Remote**: server lưu code, ở đây là GitHub, mặc định được Git đặt tên là `origin`.
- **Repository (repo)**: một thư mục project được Git theo dõi lịch sử thay đổi.
- **Commit**: một "điểm lưu" trong lịch sử — chụp lại trạng thái các file tại một thời điểm, kèm message mô tả.
- **Branch (nhánh)**: một đường phát triển độc lập. `main` là nhánh chính; nhánh phụ (ví dụ `feature/add-hello`) dùng để làm việc mà không ảnh hưởng trực tiếp tới `main`.
- **Staging area**: vùng "chờ" — file phải được `add` vào đây trước khi `commit`.
- **Pull Request (PR)**: yêu cầu merge nội dung từ 1 nhánh vào nhánh khác, kèm cơ chế review trước khi merge thật.

---

## Phần 0 — Chuẩn bị môi trường (làm 1 lần duy nhất)

```bash
git --version
```
Kiểm tra Git đã cài chưa. Nếu chưa:
```bash
sudo apt update && sudo apt install git -y
```

```bash
git config --global user.name "Tên của bạn"
git config --global user.email "email@bạn.com"
git config --global init.defaultBranch main
```
- `git config --global`: thiết lập cấu hình áp dụng cho **mọi repo** trên máy (không phải chỉ 1 repo).
- `user.name` / `user.email`: danh tính gắn vào mỗi commit bạn tạo ra — để biết ai đã sửa gì.
- `init.defaultBranch main`: khi tạo repo mới bằng `git init`, đặt tên nhánh mặc định là `main` (thay vì `master` kiểu cũ).

```bash
ssh-keygen -t ed25519 -C "email@bạn.com"
cat ~/.ssh/id_ed25519.pub
```
- `ssh-keygen`: tạo một cặp khóa SSH (1 khóa riêng giữ trên máy, 1 khóa công khai đưa cho GitHub) để xác thực khi push/clone qua SSH mà không cần nhập mật khẩu mỗi lần.
- `cat ...pub`: in nội dung khóa công khai ra màn hình để copy.
- Sau khi copy, vào **GitHub → Settings → SSH and GPG keys → New SSH key**, dán vào, lưu lại.

```bash
ssh -T git@github.com
```
Kiểm tra kết nối SSH tới GitHub đã hoạt động chưa — thấy dòng `Hi <username>!` là thành công.

---

## Phần 1 — Tạo repo và clone về máy

**Trên web GitHub** (github.com/new):
- Đặt tên repo (ví dụ `github-practice`)
- Tick **"Add a README file"**
- Bấm **Create repository**

**Trên terminal:**
```bash
cd ~/Projects
git clone git@github.com:<username-thật>/github-practice.git
```
- `git clone <url>`: tải toàn bộ repo (kèm lịch sử commit) từ remote về máy local, tạo thành một thư mục mới cùng tên repo.
- Lưu ý: `<username-thật>` phải được **thay bằng username GitHub thật của bạn**, không gõ nguyên chữ `<username>` — vì dấu `<` `>` trong Bash là ký tự điều hướng input/output, sẽ gây lỗi `No such file or directory` (đây chính là lỗi bạn đã gặp lúc đầu).

```bash
cd github-practice
```
Di chuyển vào thư mục repo vừa clone.

---

## Phần 2 — Tập push file trực tiếp vào `main`

```bash
echo "# Ghi chú thực hành GitHub" > notes.md
```
- `echo "..."`: in ra chuỗi text.
- `>`: ghi (đè) output đó vào file `notes.md` — tạo file mới nếu chưa có.

```bash
git add notes.md
```
Đưa `notes.md` vào staging area — báo cho Git biết file này sẽ nằm trong commit tiếp theo.

```bash
git commit -m "Add notes.md"
```
- `git commit`: chốt một snapshot từ những gì đang ở staging area.
- `-m "..."`: message mô tả commit, gõ thẳng trên dòng lệnh.

```bash
git push origin main
```
- `git push`: đẩy các commit từ local lên remote.
- `origin main`: đẩy lên remote tên `origin`, vào nhánh `main`.

Kết quả: file `notes.md` xuất hiện trên GitHub.

---

## Phần 3 — Tạo nhánh riêng + Pull Request

```bash
git checkout -b feature/add-hello
```
- Tạo nhánh mới tên `feature/add-hello` và chuyển sang làm việc trên đó ngay (`-b` = tạo + nhảy sang).
- Nhánh này là bản sao độc lập của `main` tại thời điểm tạo — sửa gì ở đây chưa ảnh hưởng `main`.

```bash
echo "console.log('hello github');" > hello.js
git add hello.js
git commit -m "Add hello.js"
```
Giống Phần 2: tạo file → đưa vào staging → chốt commit.

```bash
git push -u origin feature/add-hello
```
- Đẩy nhánh `feature/add-hello` (đang đứng) lên remote — vì trên GitHub chưa có nhánh này nên lệnh này sẽ **tạo mới** nó trên remote.
- `-u` (`--set-upstream`): liên kết nhánh local với nhánh remote tương ứng, để các lần `push`/`pull` sau chỉ cần gõ suông không cần chỉ định lại `origin feature/add-hello`.

**Tạo PR trên web GitHub:**
1. Bấm vào link GitHub tự in ra sau lệnh push (dạng `.../pull/new/feature/add-hello`), hoặc bấm nút **"Compare & pull request"** trên banner vàng của repo.
2. Kiểm tra 2 ô: **base** = `main` (nhánh nhận thay đổi), **compare** = `feature/add-hello` (nhánh chứa thay đổi).
3. Điền **title** (GitHub tự gợi ý từ commit message) và **description** (có thể để trống khi thực hành).
4. Bấm **Create pull request**.

---

## Phần 4 — Review & Merge PR

Các thành phần chính trên màn hình PR:
- **Open**: trạng thái PR đang mở, chưa merge.
- **Conversation / Commits / Checks / Files changed**: các tab — *Files changed* cho xem diff (phần code thay đổi); *Checks* là kiểm tra tự động (CI), repo chưa cấu hình thì hiện 0.
- **"No conflicts with base branch"**: Git xác nhận nhánh của bạn không đụng độ với `main`, có thể merge tự động.

**Thao tác merge:**
1. Bấm **Merge pull request**.
2. Bấm tiếp **Confirm merge** để hoàn tất.
3. Bấm **Delete branch** (nút hiện ra sau khi merge) để xoá nhánh `feature/add-hello` **trên remote (GitHub)**.

---

## Phần 5 — Đồng bộ lại máy local

```bash
git checkout main
```
Chuyển từ `feature/add-hello` về `main`. Lưu ý: `main` ở local lúc này **chưa biết** về việc merge vừa xảy ra trên GitHub — dòng `Your branch is up to date with 'origin/main'` sau lệnh này chỉ là thông tin cũ, chưa phải vừa kiểm tra lại remote.

```bash
git pull origin main
```
- `git pull`: ra remote kiểm tra + tải commit mới nhất về + tự gộp vào nhánh hiện tại.
- Sau lệnh này, `hello.js` (được thêm qua PR) sẽ xuất hiện trong `main` ở local.

```bash
git branch -d feature/add-hello
```
- Xoá nhánh `feature/add-hello` **ở local** (khác với nút Delete branch trên GitHub — đó là xoá bên remote, đây là xoá bên máy bạn).
- `-d`: chỉ cho xoá nếu nhánh đã được merge rồi (an toàn, tránh mất code chưa lưu ở đâu khác).

```bash
git fetch --prune
```
- `git fetch`: lấy thông tin mới nhất từ remote về nhưng **không** tự gộp vào nhánh đang đứng (khác `pull`).
- `--prune`: dọn các nhánh remote đã bị xoá (ở đây là `feature/add-hello` đã xoá trên GitHub) khỏi bộ nhớ cache local, để `git branch -a` không còn hiển thị nhánh "ma" đó nữa.

```bash
git branch -a
```
Kiểm tra lại toàn bộ nhánh (local + remote). Nếu mọi thứ đúng, chỉ còn thấy `main` và `remotes/origin/main` — không còn dấu vết của `feature/add-hello` ở đâu cả.

---

## Sơ đồ tổng quát cả flow

```
[Local: feature/add-hello] --git push--> [Remote: feature/add-hello]
                                                  |
                                          tạo Pull Request
                                                  |
                                          review + merge
                                                  v
                                          [Remote: main] (đã có thêm thay đổi)
                                                  |
[Local: main] <--------------- git pull ---------+
        |
        + git branch -d feature/add-hello   (dọn local)
        + git fetch --prune                 (dọn cache nhánh remote đã xoá)
```

---

## Vài lỗi đã gặp trong buổi thực hành (để tra cứu lại)

| Lỗi | Nguyên nhân | Cách xử lý |
|---|---|---|
| `No such file or directory` khi `cd /mnt/Projects` | Thư mục/ổ đĩa chưa tồn tại hoặc chưa mount | Tạo thư mục bằng `mkdir -p`, hoặc dùng tạm `~/Projects` |
| `No such file or directory` khi `git clone git@github.com:<username>/...` | Gõ nguyên `<username>` (kèm dấu `<` `>`) — Bash hiểu nhầm thành ký tự điều hướng input | Thay bằng username thật, bỏ dấu `<` `>` |
| `Permission denied (publickey)` | SSH key chưa thêm vào GitHub, hoặc chưa được nạp | Kiểm tra bằng `ssh -T git@github.com`, thêm key nếu cần |

---

*Tài liệu này ghi lại 1 vòng đầy đủ: tạo repo → clone → push trực tiếp → tạo nhánh → PR → merge → dọn dẹp. Đây chính là flow làm việc phổ biến nhất khi làm việc nhóm trên GitHub.*

## Upload file từ máy tính lên Github
Bước 1: Nếu đang đứng trong repo github-practice. Kiểm tra đường dãn: 
```bash
pwd
```
  -> Nó sẽ ra đường dẫn `/home/ubuntu/github-practice`
Bước 2: Di chuyển file cần up vào `/home/ubuntu/github-practice` (trên máy tính)
Bước 3: Chạy lần lượt:
```bash
git status
git add <tên-file>
git commit -m "..."
```

Bước 4: Đẩy các commit vừa thay đổi lên main
```bash
git push origin main
```

## Xóa file tren GitHub
Chạy lần lượt:
```bash
git status 'kiểm tra'
git rm tên_file
git commit -m "..."
git push origin main
```

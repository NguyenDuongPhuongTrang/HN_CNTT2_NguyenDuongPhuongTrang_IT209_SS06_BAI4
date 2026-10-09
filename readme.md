# Bài 4: Quản lý tiến trình nền với `nohup` và tín hiệu `kill`

## 1. Mục tiêu

* Khởi chạy shell script chạy nền bằng `nohup`.
* Đảm bảo tiến trình vẫn hoạt động khi đóng Terminal hoặc ngắt kết nối SSH.
* Kiểm tra tiến trình bằng `ps` và `pgrep`.
* Kiểm tra file log được ghi liên tục.
* Tìm PID của tiến trình.
* Dừng tiến trình an toàn bằng tín hiệu `SIGTERM (15)`.
* Chỉ sử dụng `SIGKILL (9)` nếu `SIGTERM` không thể dừng tiến trình.

---

## 2. Tạo shell script

Tạo file `loop-monitor.sh`:

```bash
nano loop-monitor.sh
```

Nội dung:

```bash
#!/bin/bash

while true; do
    echo "System time: $(date)" >> /tmp/monitor.log
    sleep 5
done
```

Kiểm tra nội dung file:

```bash
cat loop-monitor.sh
```

---

## 3. Cấp quyền thực thi

Cấp quyền thực thi cho script:

```bash
chmod +x loop-monitor.sh
```

Kiểm tra:

```bash
ls -l loop-monitor.sh
```

Kết quả:

```text
-rwxr-xr-x ... loop-monitor.sh
```

File đã có quyền thực thi.

---

## 4. Khởi chạy script bằng `nohup`

Khởi chạy script ở chế độ nền:

```bash
nohup ./loop-monitor.sh > /dev/null 2>&1 &
```

Trong đó:

* `nohup`: giúp tiến trình tiếp tục chạy khi đóng Terminal hoặc ngắt SSH.
* `> /dev/null`: bỏ qua output thông thường.
* `2>&1`: chuyển cả thông báo lỗi vào `/dev/null`.
* `&`: chạy tiến trình dưới nền.

---

## 5. Tìm PID của tiến trình

Sử dụng:

```bash
pgrep -af loop-monitor.sh
```

Ví dụ:

```text
12345 /bin/bash ./loop-monitor.sh
```

PID của tiến trình trong ví dụ là:

```text
12345
```

**PID thực tế:** `PID_CUA_BAN`

---

## 6. Kiểm tra tiến trình đang chạy

Sử dụng:

```bash
ps aux | grep loop-monitor.sh
```

Kết quả mong đợi có tiến trình:

```text
... /bin/bash ./loop-monitor.sh
```

Điều này chứng minh script đang chạy dưới nền.

---

## 7. Kiểm tra file log

Script ghi thời gian hiện tại vào:

```text
/tmp/monitor.log
```

Kiểm tra 10 dòng cuối:

```bash
tail -n 10 /tmp/monitor.log
```

Ví dụ:

```text
System time: Tue Oct  6 20:10:01 +07 2026
System time: Tue Oct  6 20:10:06 +07 2026
System time: Tue Oct  6 20:10:11 +07 2026
System time: Tue Oct  6 20:10:16 +07 2026
```

Các dòng log cách nhau khoảng 5 giây, chứng minh script đang hoạt động liên tục.

Có thể theo dõi trực tiếp bằng:

```bash
tail -f /tmp/monitor.log
```

Nhấn `Ctrl + C` để thoát.

---

## 8. Kiểm tra sau khi đăng nhập SSH lại

Sau khi khởi chạy bằng `nohup`, thoát khỏi SSH:

```bash
exit
```

Sau đó đăng nhập lại VPS.

Kiểm tra tiến trình:

```bash
pgrep -af loop-monitor.sh
```

Kiểm tra log:

```bash
tail -n 10 /tmp/monitor.log
```

Nếu tiến trình vẫn tồn tại và file log tiếp tục có dữ liệu mới thì chứng minh `nohup` đã giúp tiến trình hoạt động độc lập với phiên Terminal/SSH.

---

## 9. Dừng tiến trình bằng SIGTERM

Sử dụng PID đã tìm được ở bước trên.

Ví dụ PID là `12345`:

```bash
kill -15 12345
```

Hoặc:

```bash
kill -SIGTERM 12345
```

Tín hiệu `15` tương ứng với `SIGTERM`, yêu cầu tiến trình kết thúc một cách an toàn.

---

## 10. Kiểm tra tiến trình sau khi kill

Kiểm tra:

```bash
pgrep -af loop-monitor.sh
```

Nếu không có kết quả trả về thì tiến trình đã được dừng thành công.

Có thể kiểm tra thêm:

```bash
ps aux | grep loop-monitor.sh
```

Nếu chỉ còn dòng chứa:

```text
grep --color=auto loop-monitor.sh
```

thì script `loop-monitor.sh` đã không còn chạy.

---

## 11. SIGKILL

Theo yêu cầu bài tập, chỉ sử dụng `SIGKILL (9)` nếu `SIGTERM (15)` không thể dừng tiến trình.

Lệnh:

```bash
kill -9 PID_CUA_BAN
```

Sau đó kiểm tra:

```bash
pgrep -af loop-monitor.sh
```

Trong bài thực hành này, `SIGTERM` đã đủ để dừng tiến trình nên không cần sử dụng `SIGKILL`.

---

## 12. Tổng hợp các lệnh đã sử dụng

```bash
# Tạo thư mục bài tập
mkdir -p ~/homework/session_06/ex4
cd ~/homework/session_06/ex4

# Tạo script
nano loop-monitor.sh

# Cấp quyền thực thi
chmod +x loop-monitor.sh

# Kiểm tra quyền
ls -l loop-monitor.sh

# Chạy script dưới nền bằng nohup
nohup ./loop-monitor.sh > /dev/null 2>&1 &

# Tìm PID
pgrep -af loop-monitor.sh

# Kiểm tra tiến trình
ps aux | grep loop-monitor.sh

# Kiểm tra log
tail -n 10 /tmp/monitor.log

# Theo dõi log liên tục
tail -f /tmp/monitor.log

# Dừng tiến trình bằng SIGTERM
kill -15 PID_CUA_BAN

# Kiểm tra tiến trình đã dừng
pgrep -af loop-monitor.sh

# SIGKILL nếu SIGTERM không hoạt động
kill -9 PID_CUA_BAN
```

---

## 13. Kết quả

Đã hoàn thành các yêu cầu:

* [x] Tạo file `loop-monitor.sh`.
* [x] Sử dụng vòng lặp `while true`.
* [x] Ghi thời gian hiện tại vào `/tmp/monitor.log`.
* [x] Ghi log mỗi 5 giây.
* [x] Cấp quyền thực thi cho script.
* [x] Khởi chạy script bằng `nohup` và `&`.
* [x] Tìm PID bằng `pgrep`.
* [x] Kiểm tra tiến trình bằng `ps`.
* [x] Kiểm tra log bằng `tail`.
* [x] Kiểm tra tiến trình vẫn hoạt động sau khi đăng nhập SSH lại.
* [x] Dừng tiến trình bằng `kill -15`.
* [x] Xác nhận tiến trình đã biến mất khỏi danh sách process.

---

## 14. Minh chứng

### 14.1. Nội dung `loop-monitor.sh`

![Nội dung loop-monitor.sh](images/01-script.png)

### 14.2. Cấp quyền thực thi

![Cấp quyền thực thi](images/02-chmod.png)

### 14.3. Khởi chạy bằng nohup và tìm PID

![Nohup và PID](images/03-nohup-pid.png)

### 14.4. Kiểm tra tiến trình

![Kiểm tra process](images/04-process.png)

### 14.5. Kiểm tra file log

![Kiểm tra log](images/05-log.png)

### 14.6. Kiểm tra sau khi SSH đăng nhập lại

![Kiểm tra sau khi đăng nhập lại](images/06-reconnect.png)

### 14.7. Kill tiến trình bằng SIGTERM

![Kill process](images/07-kill.png)

---

## 15. Thông tin bài nộp

**Đường dẫn:**

```text
homework/session_06/ex4/
```

**Các file:**

```text
ex4/
├── loop-monitor.sh
└── README.md
```


root@vps-2:~# pgrep -af loop-monitor.sh
1736 /bin/bash ./loop-monitor.sh
1795 /bin/bash ./loop-monitor.sh
1810 /bin/bash ./loop-monitor.sh
2080 /bin/bash ./loop-monitor.sh
root@vps-2:~# kill -15 1736
root@vps-2:~# pgrep -af loop-monitor.sh
1795 /bin/bash ./loop-monitor.sh
1810 /bin/bash ./loop-monitor.sh
2080 /bin/bash ./loop-monitor.sh
root@vps-2:~#
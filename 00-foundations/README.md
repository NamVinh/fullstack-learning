# 00 — Foundations

> Nền tảng chung bắt buộc cho **mọi path** (Backend, Frontend, DevOps).
> Học một lần, dùng suốt sự nghiệp. Không hiểu phần này → học gì cũng sẽ có lỗ hổng.

---

## Mục lục

**Junior**
1. [Internet & Web hoạt động như thế nào](#1-internet--web-hoạt-động-như-thế-nào)
2. [HTTP cơ bản](#2-http-cơ-bản)
3. [Terminal / CLI](#3-terminal--cli)
4. [SSH](#4-ssh)
5. [Git cơ bản](#5-git-cơ-bản)
6. [Package Managers](#6-package-managers)

**Mid**
7. [HTTP nâng cao](#7-http-nâng-cao)
8. [TCP/IP & OSI Model](#8-tcpip--osi-model)
9. [DNS](#9-dns)
10. [TLS / HTTPS](#10-tls--https)
11. [Git nâng cao](#11-git-nâng-cao)

---

# JUNIOR

---

## 1. Internet & Web hoạt động như thế nào

> **Tại sao quan trọng**: Mọi thứ bạn build đều chạy trên internet. Không hiểu cơ chế → debug network issue sẽ mò mẫm mãi.

### Khái niệm cốt lõi

**Internet là gì?**
Mạng lưới toàn cầu kết nối hàng tỷ thiết bị với nhau qua giao thức TCP/IP. Không phải 1 thứ duy nhất — là tập hợp của nhiều mạng nhỏ.

**Web là gì?**
Một dịch vụ chạy *trên* internet (giống như email, FTP cũng chạy trên internet). Web dùng giao thức HTTP/HTTPS để trao đổi dữ liệu.

### Hành trình của 1 request: gõ URL đến khi thấy trang

```
Bạn gõ: https://example.com/products

1. Browser kiểm tra cache DNS local
   └─ Nếu không có → hỏi DNS Resolver (thường của ISP hoặc 8.8.8.8)

2. DNS Resolver tìm IP của example.com
   └─ Root DNS → TLD DNS (.com) → Authoritative DNS của example.com
   └─ Trả về: 93.184.216.34

3. Browser tạo kết nối TCP đến 93.184.216.34:443
   └─ 3-way handshake: SYN → SYN-ACK → ACK

4. TLS handshake (vì HTTPS)
   └─ Trao đổi certificate, thống nhất encryption key

5. Browser gửi HTTP request:
   GET /products HTTP/1.1
   Host: example.com

6. Server xử lý → trả về HTTP response
   └─ HTML, JSON, hoặc file

7. Browser render trang
```

### Tự kiểm tra
- Giải thích được hành trình trên không cần nhìn tài liệu
- Biết sự khác nhau giữa Internet và Web
- Biết IP address là gì, tại sao cần DNS

---

## 2. HTTP cơ bản

> **Tại sao quan trọng**: HTTP là ngôn ngữ của web. FE gọi API → HTTP. BE nhận request → HTTP. Debug mọi thứ đều qua HTTP.

### HTTP là gì?

HyperText Transfer Protocol — giao thức **stateless** (mỗi request độc lập, server không nhớ request trước) theo mô hình **request-response**.

### HTTP Methods

| Method | Dùng để | Idempotent? | Body? |
|--------|---------|-------------|-------|
| `GET` | Lấy dữ liệu | Có | Không |
| `POST` | Tạo mới | Không | Có |
| `PUT` | Thay thế toàn bộ | Có | Có |
| `PATCH` | Cập nhật một phần | Không | Có |
| `DELETE` | Xóa | Có | Thường không |

**Idempotent** = gọi nhiều lần → cùng kết quả. `DELETE /users/1` gọi lần 2 vẫn trả về "đã xóa" (không tạo thêm side effect).

### HTTP Status Codes

```
1xx — Informational (ít dùng)
  100 Continue

2xx — Success
  200 OK              → request thành công
  201 Created         → tạo resource mới thành công (dùng sau POST)
  204 No Content      → thành công nhưng không có body (dùng sau DELETE)

3xx — Redirect
  301 Moved Permanently  → chuyển hướng vĩnh viễn (browser cache)
  302 Found              → chuyển hướng tạm thời
  304 Not Modified       → dùng cache đi, server không thay đổi gì

4xx — Client Error (lỗi từ phía người dùng/client)
  400 Bad Request     → request sai format
  401 Unauthorized    → chưa đăng nhập
  403 Forbidden       → đã đăng nhập nhưng không có quyền
  404 Not Found       → resource không tồn tại
  409 Conflict        → xung đột (vd: email đã tồn tại)
  422 Unprocessable   → validation failed
  429 Too Many Requests → bị rate limit

5xx — Server Error (lỗi từ phía server)
  500 Internal Server Error → server bị lỗi
  502 Bad Gateway           → server upstream lỗi
  503 Service Unavailable   → server quá tải hoặc maintenance
```

### HTTP Headers quan trọng

**Request headers:**
```
Content-Type: application/json      → body tôi gửi là JSON
Accept: application/json            → tôi muốn nhận JSON
Authorization: Bearer <token>       → xác thực
```

**Response headers:**
```
Content-Type: application/json      → body tôi trả về là JSON
Cache-Control: no-cache             → đừng cache
Set-Cookie: session=abc; HttpOnly   → set cookie
```

### Cấu trúc HTTP Request/Response

```
# Request
POST /api/users HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bearer token123

{"name": "Vinh", "email": "vinh@example.com"}

# Response
HTTP/1.1 201 Created
Content-Type: application/json

{"id": 42, "name": "Vinh"}
```

### Tự kiểm tra
- Khi nào dùng POST, khi nào dùng PUT, khi nào dùng PATCH?
- 401 vs 403 khác nhau chỗ nào?
- Tại sao HTTP là stateless? Điều đó có nghĩa gì với auth?

---

## 3. Terminal / CLI

> **Tại sao quan trọng**: Server không có GUI. DevOps, SSH, Docker, Git — tất cả đều qua terminal. Không thạo CLI → không làm được gì trên server.

### Navigation

```bash
pwd                    # in ra thư mục hiện tại (Print Working Directory)
ls                     # liệt kê file/folder
ls -la                 # -l: dạng list, -a: gồm cả file ẩn (bắt đầu bằng .)
cd /path/to/folder     # di chuyển vào folder
cd ..                  # lên 1 cấp
cd ~                   # về home directory
cd -                   # quay lại thư mục trước
```

### Quản lý file/folder

```bash
mkdir my-folder            # tạo folder
mkdir -p a/b/c             # tạo folder lồng nhau (không báo lỗi nếu đã tồn tại)
touch file.txt             # tạo file rỗng
cp file.txt copy.txt       # copy file
cp -r folder/ backup/      # copy folder (-r: recursive)
mv file.txt newname.txt    # đổi tên hoặc di chuyển
rm file.txt                # xóa file
rm -rf folder/             # xóa folder và toàn bộ nội dung (-r: recursive, -f: force)
                           # ⚠️ NGUY HIỂM — không có trash, không undo được
```

### Đọc nội dung file

```bash
cat file.txt               # in toàn bộ nội dung (dùng cho file nhỏ)
less file.txt              # xem từng trang (q để thoát, / để search)
head -n 20 file.txt        # 20 dòng đầu
tail -n 20 file.txt        # 20 dòng cuối
tail -f app.log            # xem log real-time (f: follow)
grep "error" app.log       # tìm dòng chứa "error"
grep -r "TODO" ./src       # tìm trong toàn bộ folder
grep -i "error" app.log    # case-insensitive
```

### Permissions

```bash
# Đọc permission: rwxr-xr-x
# r=read(4), w=write(2), x=execute(1)
# [owner][group][others]
# rwx r-x r-x = 755

chmod 755 script.sh        # owner: rwx, group: r-x, others: r-x
chmod +x script.sh         # thêm execute permission cho tất cả
chmod -w file.txt          # bỏ write permission
chown user:group file.txt  # đổi owner và group
```

### Pipes & Redirects

```bash
command1 | command2        # pipe: output của cmd1 → input của cmd2
ls -la | grep ".txt"       # tìm file .txt trong danh sách

echo "hello" > file.txt    # ghi vào file (ghi đè)
echo "world" >> file.txt   # ghi vào file (thêm vào cuối)
command 2> error.log       # redirect stderr vào file
command > out.log 2>&1     # redirect cả stdout và stderr vào file
command > /dev/null        # bỏ qua output (không hiển thị)
```

### Process

```bash
ps aux                     # liệt kê tất cả processes đang chạy
kill 1234                  # gửi SIGTERM (yêu cầu kết thúc) đến process 1234
kill -9 1234               # gửi SIGKILL (buộc kết thúc ngay)
top                        # xem processes theo real-time
htop                       # top đẹp hơn (cần cài)
```

### Tự kiểm tra
- Tìm tất cả file `.js` trong thư mục hiện tại chứa chữ "TODO"
- Xem 50 dòng cuối của file log và filter chỉ dòng có "ERROR"
- Giải thích `chmod 644` có nghĩa là gì

---

## 4. SSH

> **Tại sao quan trọng**: Mọi server đều được truy cập qua SSH. Không biết SSH → không vào được server.

### SSH là gì?

Secure Shell — giao thức kết nối và điều khiển máy tính từ xa qua terminal, được **mã hóa** (khác Telnet — plaintext).

### Kết nối cơ bản

```bash
ssh username@hostname          # kết nối với password
ssh username@192.168.1.100     # kết nối bằng IP
ssh -p 2222 user@host          # dùng port khác (mặc định 22)
ssh -i ~/.ssh/my_key user@host # dùng private key cụ thể
```

### Key-based Authentication (an toàn hơn password)

```bash
# Bước 1: Tạo key pair trên máy local
ssh-keygen -t ed25519 -C "your-email@example.com"
# → tạo 2 file:
#   ~/.ssh/id_ed25519      (private key — KHÔNG chia sẻ cho ai)
#   ~/.ssh/id_ed25519.pub  (public key — copy lên server)

# Bước 2: Copy public key lên server
ssh-copy-id username@hostname
# hoặc thủ công:
cat ~/.ssh/id_ed25519.pub | ssh user@host "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

# Bước 3: Kết nối (không cần nhập password nữa)
ssh username@hostname
```

**Cơ chế hoạt động:**
- Server có **public key** của bạn
- Bạn có **private key** trên máy local
- Khi connect: server gửi challenge, bạn ký bằng private key, server verify bằng public key
- Chỉ người có private key mới xác thực được

### SSH Config file (`~/.ssh/config`)

```
# Thay vì gõ: ssh -i ~/.ssh/my_key -p 2222 ubuntu@192.168.1.100
# Chỉ cần gõ: ssh myserver

Host myserver
  HostName 192.168.1.100
  User ubuntu
  Port 2222
  IdentityFile ~/.ssh/my_key
```

### Copy file qua SSH

```bash
scp file.txt user@host:/remote/path/      # upload
scp user@host:/remote/file.txt ./local/   # download
scp -r ./folder user@host:/remote/path/  # copy cả folder
```

### Tự kiểm tra
- Tạo key pair, giải thích tại sao private key không được chia sẻ
- Sự khác nhau giữa authentication bằng password và bằng key là gì?

---

## 5. Git cơ bản

> **Tại sao quan trọng**: Git là công cụ bắt buộc trong mọi team. Dùng sai → mất code, conflict loạn, không rollback được.

### Git là gì?

Distributed Version Control System — lưu lịch sử thay đổi của code. Mỗi developer có bản copy đầy đủ của repository trên máy local.

### Cấu trúc của Git

```
Working Directory  →  Staging Area (Index)  →  Local Repository  →  Remote Repository
    (code đang sửa)    (git add)                (git commit)          (git push)
```

### Workflow cơ bản

```bash
# Khởi tạo
git init                          # tạo repo mới
git clone https://github.com/...  # copy repo về local

# Xem trạng thái
git status                        # xem file nào đã thay đổi
git diff                          # xem chi tiết thay đổi (chưa staged)
git diff --staged                 # xem thay đổi đã staged

# Commit
git add file.txt                  # stage 1 file
git add .                         # stage tất cả thay đổi
git commit -m "feat: add login"   # tạo commit
git commit -am "fix: typo"        # add + commit (chỉ với file đã tracked)

# Remote
git push origin main              # đẩy lên remote
git pull origin main              # kéo về + merge
git fetch origin                  # kéo về nhưng KHÔNG merge
```

### Branch

```bash
git branch                        # liệt kê branches
git branch feature/login          # tạo branch mới
git checkout feature/login        # chuyển sang branch
git checkout -b feature/login     # tạo + chuyển sang branch (gộp 2 lệnh)
git switch feature/login          # cách mới (Git 2.23+)
git switch -c feature/login       # tạo + chuyển (cách mới)

# Merge
git checkout main
git merge feature/login           # merge feature branch vào main
git branch -d feature/login       # xóa branch sau khi merge
```

### Undo — quan trọng, phải biết cái nào nguy hiểm

```bash
# Chưa staged — bỏ thay đổi trong file
git restore file.txt              # ⚠️ KHÔNG undo được

# Đã staged — bỏ khỏi staging area (giữ thay đổi)
git restore --staged file.txt     # an toàn

# Đã commit — tạo commit mới "đảo ngược" commit đó
git revert <commit-hash>          # an toàn, không xóa history

# Đã commit — di chuyển HEAD về trước (nguy hiểm hơn)
git reset --soft HEAD~1           # xóa commit, giữ staged changes
git reset --mixed HEAD~1          # xóa commit, giữ unstaged changes (default)
git reset --hard HEAD~1           # ⚠️ xóa commit + xóa luôn thay đổi

# Xem lịch sử
git log                           # danh sách commits
git log --oneline                 # compact hơn
git log --oneline --graph         # thêm graph nhánh
```

### `.gitignore`

```
# Không commit những thứ này
node_modules/
.env
.env.local
dist/
*.log
.DS_Store
```

### Tự kiểm tra
- Sự khác nhau giữa `git pull` và `git fetch`?
- `git reset --hard` vs `git revert` — khi nào dùng cái nào?
- Tại sao không commit `node_modules`?

---

## 6. Package Managers

> **Tại sao quan trọng**: Node.js ecosystem sống bằng npm. Hiểu cách hoạt động → tránh dependency hell, biết khi nào có vấn đề.

### npm là gì?

Node Package Manager — quản lý thư viện (packages) trong dự án Node.js/JavaScript.

### `package.json` — trái tim của dự án

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "test": "vitest"
  },
  "dependencies": {
    "express": "^4.18.0"
  },
  "devDependencies": {
    "typescript": "^5.0.0",
    "vitest": "^1.0.0"
  }
}
```

### dependencies vs devDependencies

| | dependencies | devDependencies |
|-|-------------|----------------|
| **Là gì** | Cần lúc runtime (production) | Chỉ cần lúc phát triển |
| **Ví dụ** | `express`, `react`, `prisma` | `typescript`, `eslint`, `vitest` |
| **Install** | `npm install express` | `npm install -D typescript` |
| **Khi deploy** | Phải có | Có thể bỏ qua (`npm ci --production`) |

### Version và Semver

```
"express": "^4.18.2"
             │  │ │
             │  │ └─ patch: bug fixes (tự động update)
             │  └─── minor: new features, backward compatible (tự động update)
             └────── major: breaking changes (KHÔNG tự động update)

^4.18.2  → cho phép update đến <5.0.0
~4.18.2  → cho phép update đến <4.19.0
4.18.2   → chính xác version này, không update
```

### Lock file — tại sao phải commit

```
package-lock.json (npm) / yarn.lock / pnpm-lock.yaml

Lưu CHÍNH XÁC version của mọi dependency (kể cả dependency của dependency).
→ Đảm bảo mọi người trong team, và CI/CD đều cài đúng version giống nhau.
→ PHẢI commit vào git.
```

### Các lệnh hay dùng

```bash
npm install                    # cài tất cả từ package.json
npm install express            # cài package + thêm vào dependencies
npm install -D typescript      # cài package + thêm vào devDependencies
npm uninstall express          # gỡ package
npm run dev                    # chạy script "dev" trong package.json
npm run build                  # chạy script "build"
npx create-next-app            # chạy package mà không cài global
npm ci                         # cài chính xác theo lock file (dùng trong CI)
npm outdated                   # xem packages nào có version mới
```

### npm vs yarn vs pnpm

| | npm | yarn | pnpm |
|-|-----|------|------|
| **Speed** | Chậm nhất | Nhanh | Nhanh nhất |
| **Disk space** | Nhiều | Nhiều | Ít nhất (symlinks) |
| **Lock file** | package-lock.json | yarn.lock | pnpm-lock.yaml |
| **Khuyên dùng** | Default | Ổn | Tốt nhất cho monorepo |

### Tự kiểm tra
- Tại sao `npm ci` khác `npm install`?
- `^` và `~` trong version khác nhau như thế nào?
- Tại sao không nên cài package global (`npm install -g`) cho project?

---

# MID

---

## 7. HTTP nâng cao

> **Tại sao quan trọng**: Caching sai → app chậm. CORS sai → frontend không gọi được API. CSP sai → bị XSS. Phải hiểu để debug và cấu hình đúng.

### Caching

Browser và server dùng HTTP headers để thống nhất về caching — tránh download lại file không thay đổi.

```
# Server response:
Cache-Control: max-age=3600          → cache 1 giờ
Cache-Control: no-cache              → hỏi server trước khi dùng cache
Cache-Control: no-store              → không cache gì cả
Cache-Control: public                → ai cũng cache được (CDN, browser)
Cache-Control: private               → chỉ browser cache (không phải CDN)

ETag: "abc123"                       → fingerprint của file
Last-Modified: Wed, 01 Jan 2025 ...  → lần cuối file thay đổi

# Khi browser có cache và hỏi lại:
If-None-Match: "abc123"              → "tôi đang có version này, còn mới không?"
If-Modified-Since: Wed, 01 Jan...   → "từ ngày này đến nay có thay đổi không?"

# Server trả về:
304 Not Modified                     → "không thay đổi, dùng cache đi"
200 OK + body mới                    → "đã thay đổi, đây là version mới"
```

### Cookies

```
# Server set cookie:
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Lax; Max-Age=3600; Path=/

HttpOnly   → JavaScript KHÔNG đọc được cookie này (bảo vệ khỏi XSS đánh cắp cookie)
Secure     → chỉ gửi qua HTTPS, không gửi qua HTTP
SameSite   → Strict: không gửi cross-site | Lax: gửi khi navigate | None: luôn gửi
Max-Age    → thời gian sống (giây), nếu không có → session cookie (mất khi đóng browser)
Path       → cookie chỉ gửi cho request đến path này
```

### CORS (Cross-Origin Resource Sharing)

**Vấn đề**: Browser chặn request từ `frontend.com` đến `api.com` để bảo vệ user.

**Origin** = scheme + hostname + port. Ví dụ: `https://app.com:443`

```
# Browser gửi request từ https://app.com đến https://api.com:

1. Simple request (GET/POST + basic headers):
   Browser tự động thêm: Origin: https://app.com
   Server phải trả về:   Access-Control-Allow-Origin: https://app.com

2. Preflight request (PUT/DELETE/custom headers):
   Browser gửi trước:    OPTIONS /api/users
                         Origin: https://app.com
                         Access-Control-Request-Method: DELETE
   
   Server phải trả về:   Access-Control-Allow-Origin: https://app.com
                         Access-Control-Allow-Methods: GET, POST, DELETE
                         Access-Control-Allow-Headers: Authorization, Content-Type

3. Sau khi preflight OK → browser mới gửi request thật
```

**Cấu hình đúng trong Express:**
```js
// KHÔNG dùng * ở production (kém bảo mật, không support cookie)
app.use(cors({
  origin: ['https://app.com', 'https://www.app.com'],
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Authorization', 'Content-Type'],
  credentials: true,  // cần khi dùng cookie
}))
```

### Content Security Policy (CSP)

Header bảo vệ khỏi XSS bằng cách kiểm soát browser chỉ load resource từ nguồn được phép.

```
Content-Security-Policy: default-src 'self'; script-src 'self' https://cdn.example.com; style-src 'self' 'unsafe-inline'

# Giải thích:
default-src 'self'           → mặc định chỉ load từ cùng origin
script-src 'self' https://cdn.example.com  → JS chỉ từ self hoặc cdn này
style-src 'self' 'unsafe-inline'  → CSS từ self + inline styles
```

### HTTP/1.1 vs HTTP/2 vs HTTP/3

| | HTTP/1.1 | HTTP/2 | HTTP/3 |
|-|----------|--------|--------|
| **Protocol** | TCP | TCP | UDP (QUIC) |
| **Multiplexing** | Không (1 request/connection) | Có (nhiều request/connection) | Có |
| **Header compression** | Không | Có (HPACK) | Có (QPACK) |
| **Head-of-line blocking** | Có | Có (ở TCP level) | Không |
| **Tốc độ** | Chậm nhất | Nhanh hơn | Nhanh nhất |

**HTTP/2 multiplexing** = nhiều request/response chạy song song trên 1 TCP connection → không cần mở nhiều connection.

---

## 8. TCP/IP & OSI Model

> **Tại sao quan trọng**: Khi debug network (tại sao request timeout? port bị block? server không nghe?), bạn cần biết đang nói chuyện ở tầng nào.

### OSI Model — 7 tầng

```
7. Application   → HTTP, HTTPS, FTP, SMTP, DNS        (dữ liệu)
6. Presentation  → Encryption, encoding, compression  (format dữ liệu)
5. Session       → Quản lý session giữa 2 máy
4. Transport     → TCP, UDP                            (port, flow control)
3. Network       → IP, routing                         (địa chỉ IP)
2. Data Link     → MAC address, Ethernet, WiFi         (frame)
1. Physical      → Cáp, sóng radio, bit               (bit)
```

**Thực tế**: Làm web thì chủ yếu quan tâm tầng 4 (TCP/UDP), 3 (IP), và 7 (HTTP).

### TCP vs UDP

| | TCP | UDP |
|-|-----|-----|
| **Connection** | Có (3-way handshake) | Không |
| **Reliability** | Đảm bảo nhận đủ, đúng thứ tự | Không đảm bảo |
| **Speed** | Chậm hơn | Nhanh hơn |
| **Dùng cho** | HTTP, SSH, file transfer | Video streaming, gaming, DNS, VoIP |

**TCP 3-way handshake:**
```
Client          Server
  │──── SYN ────▶│    "Tôi muốn kết nối"
  │◀── SYN-ACK ──│    "OK, tôi sẵn sàng"
  │──── ACK ────▶│    "Đã nhận, bắt đầu nhé"
  │                    → Connection established
```

### IP Address & Ports

```
IP Address: 192.168.1.100        → xác định máy tính nào
Port: :3000                      → xác định process nào trên máy đó

Một kết nối TCP được xác định bởi: (src IP, src port, dst IP, dst port)

Well-known ports:
  80   → HTTP
  443  → HTTPS
  22   → SSH
  5432 → PostgreSQL
  6379 → Redis
  3306 → MySQL
```

---

## 9. DNS

> **Tại sao quan trọng**: Deploy app → cần trỏ domain. Debug "không vào được web" → thường là DNS.

### DNS là gì?

Domain Name System — "danh bạ điện thoại" của internet. Dịch tên miền (`example.com`) thành địa chỉ IP (`93.184.216.34`).

### DNS Resolution — từng bước

```
Bạn gõ: example.com

1. Browser cache       → kiểm tra đã có chưa
2. OS cache            → /etc/hosts, system DNS cache
3. DNS Resolver        → ISP hoặc 8.8.8.8 (Google)
4. Root DNS server     → "tôi không biết, nhưng hỏi .com server"
5. TLD server (.com)   → "tôi không biết, nhưng hỏi ns1.example.com"
6. Authoritative DNS   → "IP là 93.184.216.34" ← đây là DNS của chủ domain
7. Kết quả cache lại với TTL (thời gian sống)
```

### DNS Record Types

| Record | Dùng để | Ví dụ |
|--------|---------|-------|
| `A` | Trỏ domain → IPv4 | `example.com → 93.184.216.34` |
| `AAAA` | Trỏ domain → IPv6 | `example.com → 2606:2800::1` |
| `CNAME` | Trỏ domain → domain khác | `www.example.com → example.com` |
| `MX` | Chỉ định mail server | `example.com → mail.example.com` |
| `TXT` | Dữ liệu text tùy ý | SPF, DMARC, domain verification |
| `NS` | Chỉ định nameserver | `example.com → ns1.namecheap.com` |

**TTL (Time To Live)**: bao lâu bản ghi được cache. TTL thấp (60s) → thay đổi lan nhanh. TTL cao (86400s) → ít traffic hơn đến DNS server.

### Kiểm tra DNS

```bash
nslookup example.com            # query DNS
dig example.com                 # chi tiết hơn
dig example.com +short          # chỉ in IP
dig MX example.com              # xem MX records
dig @8.8.8.8 example.com        # query DNS cụ thể (8.8.8.8 = Google)
```

---

## 10. TLS / HTTPS

> **Tại sao quan trọng**: Mọi app production đều phải dùng HTTPS. Hiểu TLS → setup đúng, không bị warning, biết tại sao cert bị lỗi.

### TLS là gì?

Transport Layer Security — mã hóa dữ liệu giữa client và server. HTTPS = HTTP + TLS.

**Tại sao cần?** HTTP plaintext → bất kỳ ai ở giữa (ISP, router, attacker) đều đọc được. TLS mã hóa → chỉ client và server đọc được.

### TLS Handshake (đơn giản hóa)

```
Client                          Server
  │                                │
  │──── ClientHello ──────────────▶│  "Tôi support TLS 1.3, các cipher này"
  │                                │
  │◀─── ServerHello + Certificate ─│  "Dùng cipher này, đây là cert của tôi"
  │                                │
  │  [Client verify certificate]   │
  │  Certificate hợp lệ?           │
  │  → Do CA (Cert Authority) ký?  │
  │  → Domain khớp không?          │
  │  → Còn hạn không?              │
  │                                │
  │──── Finished (encrypted) ─────▶│  "Session key đã trao đổi, bắt đầu encrypt"
  │                                │
  │◀═══ Encrypted HTTP ════════════│  Mọi thứ từ đây đều được mã hóa
```

### Certificate

```
Certificate chứa:
- Domain name (Common Name): example.com
- Public key của server
- Chữ ký của CA (Certificate Authority): Let's Encrypt, DigiCert...
- Thời hạn (Not Before / Not After)

Certificate Chain:
  Your cert (ký bởi Intermediate CA)
    └─ Intermediate CA (ký bởi Root CA)
         └─ Root CA (self-signed, browser tin tưởng sẵn)
```

### Let's Encrypt — free certificate

```bash
# Certbot tự động xin + gia hạn cert miễn phí
sudo certbot --nginx -d example.com -d www.example.com

# Gia hạn tự động (thêm vào cron hoặc systemd timer)
certbot renew
```

### Tự kiểm tra
- Giải thích tại sao HTTPS không thể bị "nghe lén" dù traffic đi qua nhiều router
- Certificate Expired error xảy ra khi nào và fix thế nào?
- Tại sao `localhost` không cần HTTPS nhưng production thì bắt buộc?

---

## 11. Git nâng cao

> **Tại sao quan trọng**: Senior dev phải biết git tốt hơn chỉ add/commit/push. History sạch, rebase đúng, conflict giải quyết nhanh.

### Merge vs Rebase

```
# Merge — tạo "merge commit", giữ nguyên history thật
main:    A─B─C─────G  (G là merge commit)
              ╲ ╱
feature:      D─E─F

git checkout main
git merge feature

# Rebase — "replay" commits của feature lên đầu main, history linear
main:    A─B─C─D'─E'─F'
feature:       D─E─F

git checkout feature
git rebase main
git checkout main
git merge feature  (fast-forward, không tạo merge commit)
```

**Khi nào dùng merge?** History thật, preserve context, public branches.
**Khi nào dùng rebase?** History sạch, feature branches local, trước khi tạo PR.

⚠️ **Không rebase branch đã push lên remote** → người khác dùng branch đó sẽ bị conflict.

### Interactive Rebase

```bash
git rebase -i HEAD~3   # sửa 3 commits gần nhất

# Mở editor với:
pick a1b2c3 feat: add login
pick d4e5f6 fix: typo in login
pick g7h8i9 feat: add logout

# Thay đổi thành:
pick a1b2c3 feat: add login
squash d4e5f6 fix: typo in login   # gộp vào commit trước
pick g7h8i9 feat: add logout

# Kết quả: 2 commits thay vì 3
```

**Các lệnh trong interactive rebase:**
- `pick` — giữ nguyên
- `squash` / `s` — gộp vào commit trước
- `fixup` / `f` — gộp, bỏ commit message
- `reword` / `r` — giữ changes, sửa message
- `drop` / `d` — xóa commit

### Cherry-pick

```bash
# Lấy 1 commit cụ thể từ branch khác
git cherry-pick a1b2c3

# Lấy nhiều commits
git cherry-pick a1b2c3 d4e5f6

# Lấy range
git cherry-pick a1b2c3..g7h8i9
```

**Dùng khi**: Hotfix đã commit vào feature branch nhưng cần apply vào main ngay.

### Bisect — tìm commit gây ra bug

```bash
git bisect start
git bisect bad                   # commit hiện tại bị bug
git bisect good v1.0.0           # version này còn OK

# Git tự checkout commit giữa chừng để test
# Bạn test, rồi đánh dấu:
git bisect good                  # commit này OK
git bisect bad                   # commit này bị bug

# Git tiếp tục binary search đến khi tìm ra commit gây lỗi
git bisect reset                 # kết thúc, trở về HEAD
```

### Stash

```bash
git stash                        # lưu changes tạm, working dir sạch
git stash push -m "work in progress on feature X"
git stash list                   # danh sách stashes
git stash pop                    # apply stash gần nhất + xóa khỏi stash list
git stash apply stash@{2}        # apply stash cụ thể (giữ lại trong list)
git stash drop stash@{0}         # xóa stash
```

### Conventional Commits

Chuẩn commit message để automation (changelog, semantic versioning) và dễ đọc history.

```
<type>[optional scope]: <description>

type:
  feat     → tính năng mới          → minor version bump
  fix      → sửa bug                → patch version bump
  chore    → maintenance (deps, config)
  docs     → chỉ thay đổi docs
  style    → format, không thay đổi logic
  refactor → refactor, không phải feat hay fix
  test     → thêm/sửa tests
  perf     → cải thiện performance
  ci       → CI/CD config
  BREAKING CHANGE → phá vỡ backward compatibility → major version bump

Ví dụ:
  feat(auth): add Google OAuth login
  fix(api): handle null response from payment service
  feat!: remove deprecated /v1 endpoints  (! = breaking change)
```

### Branch Strategy

**Git Flow** (phức tạp, phù hợp release cycle dài):
```
main ─────────────────────────────── production
develop ──────────────────────────── integration
  └─ feature/login ────────────────── feature
  └─ release/v2.0 ─────────────────── release prep
hotfix/critical-bug ─────────────── hotfix từ main
```

**Trunk-based Development** (đơn giản, phù hợp CI/CD nhanh):
```
main ────────────────────────── luôn deployable
  └─ feature/login (1-2 ngày) ─ tạo PR nhanh, merge sớm
```

**Khuyên dùng cho team nhỏ-vừa**: Trunk-based + feature flags.

### Tự kiểm tra
- Khi nào sẽ bị conflict khi rebase? Giải quyết thế nào?
- Tại sao `git push --force` nguy hiểm? Khi nào được phép dùng?
- Viết commit message cho: "sửa bug khi user nhập email có chữ hoa không login được"

---

## Resources

| Topic | Link |
|-------|------|
| Internet / Web | [MDN — How the Internet works](https://developer.mozilla.org/en-US/docs/Learn/Common_questions/Web_mechanics/How_does_the_Internet_work) |
| HTTP | [MDN — HTTP overview](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview) |
| HTTP/2 | [http2.github.io](https://http2.github.io) |
| Terminal | [The Missing Semester (MIT)](https://missing.csail.mit.edu) |
| Git | [Pro Git Book (free)](https://git-scm.com/book/en/v2) |
| Git interactive | [Learn Git Branching](https://learngitbranching.js.org) |
| Conventional Commits | [conventionalcommits.org](https://www.conventionalcommits.org) |
| TLS | [How HTTPS works (comic)](https://howhttps.works) |
| DNS | [How DNS works (comic)](https://howdns.works) |

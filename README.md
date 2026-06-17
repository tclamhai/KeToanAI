# Kế Toán SYT

**Phần mềm Kế toán Sở Y Tế** theo Thông tư 24/2024/TT-BTC

> Ứng dụng desktop quản lý kế toán cho các đơn vị hành chính sự nghiệp thuộc Sở Y tế, xây dựng trên nền tảng Electron + NestJS + SQLite.

---

## Mục lục

- [Tổng quan](#tổng-quan)
- [Kiến trúc hệ thống](#kiến-trúc-hệ-thống)
- [Cài đặt](#cài-đặt)
- [Chạy ứng dụng](#chạy-ứng-dụng)
- [Triển khai Production (Docker Swarm)](#triển-khai-production-docker-swarm)
- [Local Print Agent (QZ Tray Style)](#local-print-agent-qz-tray-style)
- [Cấu trúc thư mục](#cấu-trúc-thư-mục)
- [Các chức năng chính](#các-chức-năng-chính)
- [Mô hình dữ liệu](#mô-hình-dữ-liệu)
- [API Endpoints](#api-endpoints)

---

## Tổng quan

Kế Toán SYT là phần mềm kế toán dành cho đơn vị hành chính sự nghiệp, tuân thủ **Thông tư 24/2024/TT-BTC** của Bộ Tài chính. Phần mềm hỗ trợ:

- Quản lý hệ thống tài khoản kế toán (theo TT24)
- Lập phiếu thu, phiếu chi với in ấn
- **Thiết lập mẫu đánh số tự tăng** — cho phép cấu hình mẫu đánh số (pattern) và chu kỳ reset theo tháng hoặc năm cho phiếu thu/chi và chứng từ ghi sổ.
- **Đồng bộ danh mục theo kỳ kế toán** — tự động nhân bản (cloning) hoặc nạp dữ liệu mẫu (seeding) khi chuyển sang năm làm việc mới.
- **Phiếu kế toán** — trung tâm nhập liệu định khoản theo nhóm phiếu, tháng/năm
- **Chứng từ ghi sổ** — khai báo Số CT, Ngày CT và định khoản
- Sổ nhật ký chung, Sổ cái
- **Phân hệ Kho (Vật tư - Hàng hóa)** — Hỗ trợ quản lý độc lập đa phân hệ kho (Kho Dược, Kho Hành chánh, Kho Nhà thuốc, Kho Vật tư - Y cụ), chứng từ Nhập/Xuất/Chuyển kho, 8 danh mục quản lý trực tiếp và báo cáo kho chi tiết.
- **Báo cáo tài chính động** (B01–B05 BCTC) với cấu hình công thức linh hoạt và chạy thử in-memory
- **Trình xây dựng Báo cáo tùy chỉnh** (Report Builder)
- Quản lý nguồn kinh phí, mục lục ngân sách, dự toán ngân sách
- Quản lý đối tượng theo dõi (khách hàng, nhà cung cấp, nhân viên...)
- Ghi sổ / bỏ ghi sổ chứng từ
- Số dư đầu kỳ
- **Bút toán đồng thời** & **Từ điển định khoản AI**
- **Trợ lý AI Đa phương thức** (Gemini Cloud + Chrome Built-in AI)
- **Sao lưu & Phục hồi dữ liệu** và nhập dữ liệu từ Sở Y tế (DBF/SYT)
- **Kiểm toán & Truy vết** (Audit Trail)
- Dashboard tùy biến kéo thả (Drag & Drop)
- Quản lý phòng ban, người dùng, phân quyền menu, và kiểm soát đăng nhập chặt chẽ theo đơn vị
- Cấu hình thông tin đơn vị linh hoạt (mở rộng quyền khai báo cho mọi người dùng)
- Lịch sử thao tác (audit log)

---

## Kiến trúc hệ thống

```
┌─────────────────────────────────────────────┐
│               Electron Shell                │
│  ┌───────────────────────────────────────┐  │
│  │     Frontend (Vanilla HTML/CSS/JS)    │  │
│  │         public/js/pages/*.js          │  │
│  └──────────────┬────────────────────────┘  │
│                 │ REST API (port 3456)       │
│  ┌──────────────▼────────────────────────┐  │
│  │   Backend (NestJS + Express Routes)   │  │
│  │    backend/src/ + server/routes/      │  │
│  │    server/services/ (Engine Layer)    │  │
│  └──────────────┬────────────────────────┘  │
│                 │ Sequelize ORM              │
│  ┌──────────────▼────────────────────────┐  │
│  │       SQLite (data/ketoan_syt.db)     │  │
│  └───────────────────────────────────────┘  │
└─────────────────────────────────────────────┘
```

| Thành phần     | Công nghệ                                    |
| -------------- | --------------------------------------------- |
| Desktop Shell  | Electron                                      |
| Backend        | NestJS (TypeScript) + Express routes (JS)     |
| ORM            | Sequelize v6                                  |
| Database       | SQLite (file-based, không cần cài đặt)        |
| Frontend       | Vanilla HTML/CSS/JS (SPA, client-side routing) |
| Icons          | Font Awesome 6                                |
| AI Engine      | Google Gemini API + Chrome Built-in AI (Gemini Nano) |
| Automation     | Playwright + Stealth Plugin                   |

---

## Cài đặt

### Cách 1: Dành cho Người dùng (File cài đặt)

> **Chỉ cần 1 file — không cần cài thêm bất kỳ phần mềm nào.**

#### Yêu cầu hệ thống

| Yêu cầu | Chi tiết |
| -------- | -------- |
| Hệ điều hành | Windows 10 trở lên (64-bit) |
| RAM | ≥ 4 GB |
| Dung lượng đĩa | ≥ 500 MB |

#### Các bước cài đặt

1. **Tải file** `KeToanSYT Setup 1.0.5.exe` (~111 MB)
2. **Chạy file cài đặt**
   - Windows SmartScreen có thể cảnh báo (do chưa có chứng chỉ ký số)
   - Bấm **"More info"** → **"Run anyway"**
3. **Chọn thư mục cài đặt** (mặc định: `C:\Users\<tên>\AppData\Local\Programs\KeToanSYT`)
4. **Hoàn tất** — Shortcut tự động tạo trên **Desktop** và **Start Menu**
5. **Mở ứng dụng lần đầu** — Hiện hộp thoại **chọn thư mục lưu database**:
   - **"Dùng thư mục mặc định"** → lưu tại `%AppData%\ke-toan-syt\data\`
   - **"Chọn thư mục khác..."** → chọn ổ D, USB, thư mục mạng, v.v.
   - Đường dẫn đã chọn được lưu lại, các lần sau không hỏi lại
6. Hệ thống **tự động khởi tạo dữ liệu**:
   - Tạo database SQLite mới
   - Seed hệ thống tài khoản kế toán (Thông tư 24/2024/TT-BTC)
   - Seed loại chứng từ, nhóm phiếu, nguồn kinh phí, mục lục ngân sách
   - Tạo tài khoản admin mặc định
7. **Đăng nhập**: `admin` / `admin123`

> **Lưu ý:** Lần đầu mở ứng dụng có thể mất 5-10 giây để khởi tạo dữ liệu. Các lần sau mở nhanh hơn.

#### Dữ liệu người dùng

- Database SQLite lưu tại thư mục **người dùng tự chọn** khi cài đặt lần đầu
- Mặc định: `%AppData%\ke-toan-syt\data\ketoan_syt.db`
- Cấu hình đường dẫn lưu tại: `%AppData%\ke-toan-syt\ketoan_config.json`
- **Gỡ cài đặt KHÔNG xóa dữ liệu** — đảm bảo an toàn dữ liệu kế toán
- Để xóa hoàn toàn: gỡ cài đặt + xóa thư mục database đã chọn + xóa `%AppData%\ke-toan-syt\`

#### Không cần cài đặt thêm


- ❌ Node.js (đã nhúng trong Electron)
- ❌ Python, Visual Studio Build Tools
- ❌ SQLite (đã nhúng)
- ❌ Bất kỳ runtime/framework nào khác

---

### Cách 2: Dành cho Nhà phát triển (Source code)

#### Yêu cầu

- **Node.js** >= 18.x
- **npm** >= 9.x
- **Visual Studio Build Tools 2022** (để rebuild native module `sqlite3`)

#### Bước cài đặt

```bash
# Clone repository
git clone <repo-url> KeToanAI
cd KeToanAI

# Cài dependencies gốc
npm install

# Cài dependencies backend (NestJS)
cd backend && npm install && cd ..
```

#### Khởi tạo dữ liệu mẫu

```bash
npm run seed
```

Lệnh này tạo dữ liệu ban đầu: hệ thống tài khoản, loại chứng từ, nhóm phiếu, tài khoản admin mặc định, cấu hình báo cáo tài chính B01–B05.

---

## Chạy ứng dụng

### Chế độ phát triển (Development)

```bash
npm run dev
```

Lệnh này khởi chạy đồng thời:
- **NestJS backend** ở `http://localhost:3456` (watch mode)
- **Electron** window kết nối tới backend

### Chỉ chạy backend (không Electron)

```bash
npm run dev:server
```

Truy cập `http://localhost:3456` trên trình duyệt.

### Đóng gói thành file cài đặt

```bash
# Build backend TypeScript
npm run build:backend

# Rebuild sqlite3 cho Electron (cần VS Build Tools)
npx @electron/rebuild --force

# Tạo file cài đặt NSIS (.exe)
npm run build
# → Output: dist/installer/KeToanSYT Setup X.X.X.exe

# Hoặc chỉ tạo thư mục unpacked (để test nhanh)
npm run build:dir
# → Output: dist/installer/win-unpacked/
```

---

## Triển khai Production (Docker Swarm)

Dự án hỗ trợ môi trường triển khai Production đạt độ sẵn sàng cao (High Availability), cập nhật không gián đoạn (Zero-Downtime Rolling Update) và bảo mật vững chắc thông qua công nghệ **Docker Swarm Mode** kết hợp tường lửa **iptables** và proxy **Nginx** trên VPS Linux.

### 1. Kiến trúc Triển khai
* **2 Replicas Web App (`ketoanai-web`)**: Đảm bảo phân tải và chịu lỗi. Nếu một container gặp sự cố hoặc khởi động lại, container còn lại vẫn gánh tải bình thường.
* **Rolling Update (`start-first`)**: Trong quá trình cập nhật phiên bản mới, Swarm sẽ khởi chạy container mới mang code mới trước. Sau khi container mới vượt qua các bài kiểm tra sức khỏe (`healthcheck` gọi vào `/api/license/status`), Swarm mới tiến hành tắt các container cũ.
* **Bảo mật cổng nội bộ (Port Security)**:
  * Swarm Ingress được cấu hình lại để chỉ lắng nghe cổng `3456` tại địa chỉ loopback `127.0.0.1`.
  * Thiết lập quy tắc tường lửa `iptables` để chặn hoàn toàn truy cập trực tiếp từ môi trường internet ngoài vào cổng `3456`. Chỉ duy nhất proxy Nginx cục bộ chạy trên máy chủ được phép chuyển tiếp (proxy_pass) traffic từ HTTPS (`https://ketoanai.id.vn`) vào container.

### 2. Thiết lập trên VPS
1. **Kích hoạt Swarm Mode**:
   ```bash
   docker swarm init
   ```
2. **Cấu hình lại mạng Swarm Ingress để giới hạn binding IP nội bộ**:
   ```bash
   docker network rm ingress
   docker network create --driver overlay --ingress --opt com.docker.network.bridge.host_binding_ipv4=127.0.0.1 ingress
   ```
3. **Cài đặt quy tắc bảo mật tường lửa `iptables`**:
   ```bash
   iptables -I DOCKER-USER -p tcp --dport 3456 -i eth0 -j DROP
   ```

### 3. Quy trình Triển khai tự động (Git-Based Deploy)
Mã nguồn được cập nhật lên máy chủ thông qua Git hook (`post-receive`). 
Khi nhà phát triển thực hiện đẩy code lên server:
```bash
git push vps main
```

Quy trình tự động hóa kích hoạt bởi server (`/home/git/ketoanai.git/hooks/post-receive`):
1. Checkout phiên bản code mới nhất ra thư mục làm việc `/root/KeToanAI`.
2. Thực hiện rebuild Docker image mới cho Web App:
   ```bash
   docker compose build ketoanai-web
   ```
3. Deploy / Cập nhật Stack Swarm:
   ```bash
   docker stack deploy -c docker-compose.yml ketoanai
   ```
4. Thực hiện Rolling Update cuốn chiếu và kiểm tra sức khỏe dịch vụ:
   ```bash
   docker service update --force --image ketoanai-web:latest ketoanai_ketoanai-web
   ```

### 4. Quản lý container bằng Portainer CE
Để theo dõi trực quan tài nguyên hệ thống, trạng thái dịch vụ và kiểm tra logs, Portainer CE được cài đặt trực tiếp trên VPS:
* Truy cập qua cổng mặc định: HTTPS `9443` hoặc HTTP `9000` của VPS IP.
* **Tài khoản đăng nhập**: `admin` / `KetoanAI2026!`
```bash
docker volume create portainer_data
docker run -d -p 9443:9443 -p 9000:9000 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```

---

## Local Print Agent (QZ Tray Style)

Ứng dụng hỗ trợ in ấn trực tiếp không qua hộp thoại xem trước (Direct Silent Printing) tương tự như QZ Tray dành cho các máy trạm Windows kết nối máy in vật lý. 

Khi truy cập phiên bản phần mềm chạy trên Web (VPS), người dùng có thể kích hoạt tính năng **"In bỏ qua Preview"** và sử dụng Local Print Agent chạy ở máy trạm cục bộ để thực hiện lệnh in trực tiếp và nhanh chóng.

### 1. Cơ chế hoạt động
* **HTTP API Server**: Chạy ngầm trên cổng `18182` tại `127.0.0.1`. Nhận lệnh in thông qua yêu cầu `POST /print` chứa mã HTML từ Web App.
* **Biên dịch Headless**: Tự động sử dụng trình duyệt Chromium có sẵn trên máy trạm (**Microsoft Edge** hoặc **Google Chrome** tùy cấu hình) để dựng HTML thành file PDF chất lượng cao ở chế độ ngầm (`--headless`).
* **In ấn trực tiếp**: Sử dụng công cụ portable **SumatraPDF** (tự động tải bản portable khi chạy lần đầu) để đẩy file PDF ra máy in đã chọn mà không hiển thị hộp thoại Preview.
* **Giao diện khay hệ thống (System Tray)**: Chạy ẩn hoàn toàn dưới khay hệ thống, cung cấp menu chuột phải để mở nhanh trang Dashboard cấu hình máy in, khởi động lại đại lý hoặc thoát.

### 2. Giao diện Dashboard (Web Console)
Truy cập tại địa chỉ: `http://127.0.0.1:18182/`
* **Trạng thái kết nối**: Hiển thị trạng thái hoạt động của Agent, SumatraPDF và đường dẫn trình duyệt Chromium.
* **Cấu hình máy in**: Cho phép chọn máy in mặc định để nhận lệnh in, lưu cấu hình vào `agent-config.json`.
* **Trình duyệt Render**: Tùy chọn chuyển đổi linh hoạt giữa **Microsoft Edge** và **Google Chrome** làm động cơ render PDF.
* **Khởi động cùng Windows**: Nút bật/tắt tự động khởi chạy cùng Windows (thông qua thư mục Startup của người dùng).
* **Nhật ký hoạt động (Logs)**: Hiển thị trực tiếp các sự kiện in ấn và biên dịch PDF thời gian thực.
* **In thử nghiệm (Test Print)**: Nút in thử mẫu Hóa đơn A5 và Phiếu thu/chi để kiểm tra nhanh máy in.

### 3. Hướng dẫn sử dụng trên máy trạm

#### Yêu cầu hệ thống máy trạm
* Hệ điều hành: Windows 10 trở lên.
* Đã cài đặt **Microsoft Edge** hoặc **Google Chrome**.

#### Các file kịch bản điều khiển (nằm trong thư mục `scripts/print-agent/`)
* **`run-agent.bat`**: Khởi chạy Print Agent, hiển thị icon khay hệ thống và tự động mở trang Dashboard cấu hình trên trình duyệt.
* **`run-agent-silent.vbs`**: Khởi chạy Print Agent hoàn toàn im lặng, chỉ xuất hiện dưới dạng biểu tượng khay hệ thống.
* **`stop-agent.bat`**: Dừng hoàn toàn khay hệ thống và các tiến trình in ấn đang chạy ngầm của Agent.
* **`build-agent.bat`**: Biên dịch mã nguồn thành file thực thi duy nhất `ketoanai-print-agent.exe` độc lập không phụ thuộc môi trường Node.js.

#### Cách chạy ứng dụng trên máy trạm:
1. Double-click vào file [run-agent.bat](file:///d:/KeToanAI/scripts/print-agent/run-agent.bat) để khởi chạy.
2. Trên khay hệ thống sẽ hiển thị biểu tượng máy in. Click đúp vào biểu tượng để mở Dashboard điều khiển hoặc click chuột phải chọn **Open Dashboard** / **Restart Agent** / **Exit**.
3. Khi sử dụng Web App, bạn tích chọn checkbox **"In bỏ qua Preview"**. Hệ thống sẽ tự động chuyển tiếp các chứng từ in tới máy in thông qua Đại lý.

---

## Cấu trúc thư mục

```
KeToanAI/
├── backend/                  # NestJS backend (TypeScript)
│   └── src/
│       ├── main.ts           # Entry point — mount Express routes
│       ├── app.module.ts     # Root module
│       └── objects/          # NestJS modules (đối tượng)
├── database/
│   ├── models/
│   │   └── index.js          # Sequelize models + associations (30+ models)
│   └── seeders/
│       └── seed.js           # Dữ liệu mẫu ban đầu
├── electron/
│   ├── main.js               # Electron main process
│   └── preload.js            # Preload script
├── public/                   # Frontend tĩnh (SPA)
│   ├── index.html            # Entry HTML
│   ├── css/
│   │   └── app.css           # Toàn bộ styles (hỗ trợ Light/Dark/Eye Protection)
│   └── js/
│       ├── api.js            # API client, showToast, showConfirm, Voice, Grid Utils
│       ├── app.js            # SPA router + page loader
│       ├── sidebar.js        # Sidebar navigation
│       ├── login.js          # Login page
│       └── pages/
│           ├── dashboard.js           # Dashboard tùy biến kéo thả & biểu đồ
│           ├── documents.js           # Chứng từ ghi sổ
│           ├── vouchers.js            # Phiếu Thu/Chi
│           ├── accounting_vouchers.js # Phiếu kế toán (trung tâm định khoản)
│           ├── accounts.js            # Hệ thống tài khoản
│           ├── journals.js            # Sổ nhật ký chung
│           ├── opening_balances.js    # Số dư đầu kỳ
│           ├── others.js              # Nguồn KP, MLNS, Đối tượng, Nhóm phiếu, Báo cáo TC...
│           ├── reports_view.js        # Xem báo cáo
│           ├── report_builder.js      # Trình xây dựng báo cáo tùy chỉnh
│           ├── simultaneous_rules.js  # Bút toán đồng thời
│           ├── entry_templates.js     # Từ điển định khoản AI
│           ├── users.js               # Quản lý người dùng & phòng ban
│           ├── audit.js               # Kiểm toán & truy vết
│           └── backup_restore.js      # Sao lưu & phục hồi dữ liệu
├── server/
│   ├── index.js              # Express server (standalone)
│   ├── middleware/
│   │   └── auth.js           # JWT authentication middleware
│   ├── routes/               # Express API routes (28 route files)
│   │   ├── auth.js
│   │   ├── accounts.js
│   │   ├── documents.js
│   │   ├── vouchers.js
│   │   ├── accounting-vouchers.js
│   │   ├── journals.js
│   │   ├── ledgers.js
│   │   ├── opening-balances.js
│   │   ├── funding-sources.js
│   │   ├── budget-items.js
│   │   ├── budget.js
│   │   ├── objects.js
│   │   ├── dashboard.js
│   │   ├── reports.js          # Báo cáo TC + Dynamic Formula API
│   │   ├── report-templates.js
│   │   ├── user-logs.js
│   │   ├── users.js
│   │   ├── departments.js
│   │   ├── menus.js
│   │   ├── assets.js
│   │   ├── inventory.js
│   │   ├── simultaneous-rules.js
│   │   ├── entry-templates.js
│   │   ├── ai.js
│   │   ├── promptTemplates.js
│   │   ├── audit.js
│   │   ├── backup-restore.js
│   │   └── settings.js
│   └── services/             # Business logic layer
│       ├── reportFormulaEngine.js  # Công cụ tính toán công thức báo cáo
│       ├── reportGenerator.js      # Sinh báo cáo tài chính
│       ├── ledgerService.js        # Sổ cái & tính toán số dư
│       ├── sytImportService.js     # Nhập dữ liệu từ Sở Y tế (DBF)
│       ├── geminiWeb.service.js    # Stealth Playwright + Gemini Web AI
│       └── promptDictionary.js     # Từ điển prompt AI
├── data/                     # SQLite database file (auto-created)
└── package.json
```

---

## Các chức năng chính

### 1. Nghiệp Vụ

#### Phiếu kế toán ⭐ (Trung tâm nhập liệu)

Màn hình chính để nhập và quản lý toàn bộ định khoản kế toán.

- **Lọc theo**: Nhóm phiếu + Tháng + Năm → nhấn "Mở bảng"
- **Lưới Excel-like**: Nhập trực tiếp trên lưới, tự động lưu khi rời ô (Auto-save) và đồng bộ State-DOM cực kỳ chặt chẽ (chống mất chữ khi cuộn).
- **Cột dữ liệu**: STT, Mã nguồn, Nội dung, TK Nợ, ĐT Nợ, TK Có, ĐT Có, Số tiền, Số phiếu, Ngày lập, Số CT, Ngày CT
- **AI Tự động học tập (Behavior Learning)**:
  - Hệ thống tự động ghi nhận thói quen nhập liệu của kế toán viên dưới nền (Background Learning).
  - Tự động sinh `EntryTemplate` (Bút toán mẫu) sau 3 lần nhập lặp lại.
  - Tự động điền các ô tiếp theo (Smart Autocomplete & Contextual Fill) dựa trên AI.
  - Xử lý bút toán đồng thời linh hoạt.
- **Giao diện thông minh**:
  - Hỗ trợ cuộn ngang (horizontal scroll), thay đổi kích thước cột (resize) và kéo thả đổi vị trí cột (drag & drop).
  - Các cột danh mục (TK, Đối tượng, Loại phiếu, Mã nguồn) sử dụng Input + Datalist cho phép gõ tìm kiếm thông minh thay vì thẻ `<select>` truyền thống.
- **Thao tác nhanh**:
  - Cơ chế **Continuous Enter Navigation**: `Enter` lướt qua mọi ô mượt mà. Hệ thống dùng `readOnly` thông minh để tự động vượt qua các ô không khả dụng mà không làm rơi Focus.
  - Chèn dòng (tại vị trí bất kỳ), Xóa dòng (kiểm tra trạng thái), tự động đánh lại STT.
  - Bộ công cụ **Tìm kiếm & Thay thế** nâng cao ngay trên lưới.
- **Cảnh báo lỗi trực quan**:
  - Auto-validation: Chữ đỏ báo lỗi sai mã, highlight vàng toàn dòng nếu thiếu thông tin thiết yếu. Tự động chặn xuống dòng nếu dữ liệu đang sai sót.
  - Nút `[+]` tự động xuất hiện kế bên Đối tượng không hợp lệ để tạo mới cực nhanh.
  - Tính năng **Lọc CT sai** giúp tập trung sửa chữa các dòng báo lỗi.

#### Chứng từ ghi sổ

Quản lý chứng từ ghi sổ với khai báo Số CT và Ngày CT ở phần header.

- Khai báo: Loại CT, Nhóm phiếu, **Số CT**, **Ngày CT**, Diễn giải, Số tiền
- Lưới định khoản: Mã nguồn, Nội dung, TK Nợ, ĐT Nợ, TK Có, ĐT Có, Số tiền
- Hỗ trợ: Ghi sổ / Bỏ ghi sổ / Xóa chứng từ
- Checkbox batch: Ghi sổ / Bỏ ghi sổ hàng loạt nhiều chứng từ cùng lúc

#### Phiếu Thu / Phiếu Chi

- Lập phiếu thu (PT), phiếu chi (PC) theo biểu mẫu chuẩn
- In phiếu trực tiếp từ giao diện
- Tự động ghi sổ sau khi lập (tùy chọn)
- **Mặc định**: Số CT = Số phiếu, Ngày CT = Ngày lập
- Nhập liệu giọng nói thông minh (Smart Voice Template)

#### Quy tắc đánh số chứng từ tự tăng

Hệ thống cho phép cấu hình linh hoạt cách thức đánh số chứng từ tự động cho cả **Phiếu Thu - Phiếu Chi** và **Chứng từ ghi sổ / Chứng từ khác** trong trang Cài đặt:
- **Chu kỳ reset số thứ tự:** Cho phép tùy chọn reset số thứ tự tự tăng (`{num}`) theo **Tháng** (MONTH) hoặc **Năm** (YEAR). Khi chuyển sang tháng mới hoặc năm mới, số thứ tự sẽ tự động quay về `00001` (hoặc `001`).
- **Mẫu định dạng (Pattern):** Cho phép người dùng tự định nghĩa cấu trúc số chứng từ bằng các token động:
  - `{type}`: Mã loại chứng từ (PT, PC, CTGS...).
  - `{yyyy}`: Năm 4 số (ví dụ: `2026`).
  - `{yy}`: Năm 2 số (ví dụ: `26`).
  - `{mm}`: Tháng 2 số (ví dụ: `05`).
  - `{num}`: Số tự tăng tự động đệm số 0 (độ dài 5 số với Phiếu thu/chi, 3 số với Chứng từ ghi sổ).

#### Số dư đầu kỳ

- Nhập số dư đầu kỳ theo tháng/năm cho từng tài khoản
- Ghép mục lục ngân sách tự động cho tài khoản có `use_budget_item`
- Hỗ trợ đối tượng chi tiết theo tài khoản

#### Bút toán đồng thời

- Cấu hình quy tắc: Khi ghi nhận TK Nợ + TK Có → Tự động phát sinh bút toán đồng thời
- Ví dụ: Khi ghi Nợ 611 / Có 111 → Đồng thời ghi Có 008
- Quản lý CRUD các quy tắc, bật/tắt linh hoạt

#### Từ điển định khoản AI

- Bút toán mẫu dựa trên mô hình keyword → TK Nợ/TK Có
- AI tự động học từ hành vi nhập liệu thực tế (`UserBehavior`)
- Gợi ý thông minh khi nhập nội dung trùng pattern

### 2. Trợ lý AI Đa phương thức (AI Chatbox)

Hệ thống tích hợp Trợ lý Kế toán AI thông minh ngay trên giao diện (Floating Sidebar), hỗ trợ kế toán viên xử lý nghiệp vụ, phân tích số liệu và giải đáp thắc mắc.

- **Tương tác Nổi Cao Cấp (Interactive Floating Panel):**
  - **Drag & Move (Kéo thả tự do):** Kéo thả thanh tiêu đề màu xanh để di chuyển khung chat tới bất kỳ đâu trên màn hình. Nút tròn màu tím mở chat sẽ tự động bám đuổi sát mép trái. Vị trí tự động reset về mặc định khi đóng panel.
  - **Drag & Drop File (Đính kèm file):** Kéo thả trực tiếp tệp hình ảnh vào khung chat. Viền khung đổi sang dạng nét đứt màu xanh dương để báo hiệu và hiển thị hình ảnh xem trước kèm nút xóa phía trên ô nhập liệu.
  - **Taskbar Clearance Protection (Chống che khuất):**
    - Chế độ thu gọn: Khoảng cách đáy an toàn `64px` (khoảng cách đỉnh `16px`), giúp khung chat nổi hoàn hảo trên thanh Taskbar Windows. Nút kích hoạt mở chat có khoảng cách đáy là `72px`.
    - Chế độ mở rộng (Fullscreen): Đáy thụt vào `48px` tránh đè dưới thanh Taskbar.
- **Đa phương thức nhập liệu (Multimodal Input):**
  - Text: Nhắn tin thông thường.
  - Voice: Nhập liệu bằng giọng nói (Speech-to-Text).
  - Vision: Chụp ảnh màn hình ứng dụng hoặc tải lên file ảnh đính kèm (Hóa đơn, báo cáo) để AI phân tích nội dung trực quan.
- **Tích hợp Nghiệp vụ & In ấn:**
  - AI tự động nhận diện mẫu lệnh để xuất báo cáo nhanh hoặc lập phiếu thu/chi nháp và phản hồi kèm nút in trực tiếp.
  - Tích hợp style `@media print` ẩn hoàn toàn chatbot khi in chứng từ.
- **Hỗ trợ 2 chế độ suy luận (Inference Modes):**
  - **Cloud AI (Gemini):** Kết nối API Gemini của Google để sử dụng các mô hình mạnh mẽ như `gemini-1.5-flash-latest`. Danh sách model được tự động cập nhật dựa trên API Key.
  - **Local AI (Chrome Built-in AI):** Chạy trực tiếp AI trên máy tính không cần Internet thông qua `Prompt API` (`window.ai.languageModel`) kết hợp mô hình Gemini Nano tích hợp sẵn trong Chrome 127+.
- **Quản lý dữ liệu thông minh:**
  - **Cá nhân hóa Cài đặt:** API Key và cấu hình Model được lưu trữ và cô lập độc lập cho từng người dùng (User Profile).
  - **Lịch sử trò chuyện:** Toàn bộ nội dung chat và hình ảnh (Base64) được lưu trữ vĩnh viễn trên trình duyệt bằng `IndexedDB` (thông qua `localforage`) nhằm tránh giới hạn Quota của `localStorage`.
  - **Tải lười (Infinite Scroll):** Hiển thị 10 tin nhắn gần nhất khi mở và tự động tải thêm khi cuộn lên trên (Pagination).
  - Nút "Clear Chat" giúp làm sạch hội thoại nhanh chóng.

### 3. Báo Cáo

| Chức năng                    | Mô tả                                                                     |
| ---------------------------- | -------------------------------------------------------------------------- |
| Sổ nhật ký                   | Sổ nhật ký chung theo kỳ                                                  |
| Sổ cái                       | Sổ cái tài khoản, chi tiết phát sinh theo tháng, có thể xem chi tiết từng TK |
| Bảng cân đối tài khoản       | Cân đối phát sinh theo kỳ (tháng/quý/năm)                                 |
| Báo cáo tài chính (B01–B05)  | Bảng CĐKT, Kết quả HĐKD, Lưu chuyển tiền tệ, Thuyết minh BCTC, Quyết toán |
| Cấu hình công thức báo cáo   | Tùy chỉnh công thức tính từng chỉ tiêu, ánh xạ tài khoản, chạy thử in-memory |
| Report Builder               | Trình xây dựng báo cáo tùy chỉnh linh hoạt                               |

#### Cấu hình Công thức Báo cáo Động

Tính năng nổi bật cho phép kế toán viên **tùy chỉnh hoàn toàn** công thức tính toán cho từng chỉ tiêu trên báo cáo tài chính:

- **Layout Split-Column**: Modal cấu hình rộng rãi chia làm hai vùng:
  - **Cột trái**: Danh sách chỉ tiêu thụt lề theo cấp bậc, tích hợp thanh tìm kiếm
  - **Cột phải**: Form chỉnh sửa thuộc tính chỉ tiêu + bảng ánh xạ tài khoản
- **Ánh xạ tài khoản chi tiết**: Tài khoản chính, tài khoản đối ứng, phương thức tính (`PSNO`, `PSCO`, `DNDK`, `PSDU`...), dấu hạch toán (+/−)
- **Công thức tổng hợp chéo**: Ví dụ `01 + 02 - 03` để tổng hợp giữa các chỉ tiêu
- **Chạy thử in-memory (Dry Run)**: Tính toán tạm thời dựa trên Sổ cái hiện tại mà không ghi đè DB
- **Đối chiếu tự động**: Cho báo cáo B03 (Lưu chuyển tiền tệ), tự động đối chiếu số dư tiền cuối kỳ với TK 111 + 112
- **Hỗ trợ đa theme**: Light, Dark, Eye Protection — tất cả kế thừa CSS Variables

### 4. Dashboard Tùy biến

- **Widget kéo thả (Drag & Drop)**: Sắp xếp lại thứ tự các widget thống kê trên dashboard
- **Tùy chỉnh hiển thị**: Ẩn/hiện widget, thay đổi layout
- **Biểu đồ trực quan**: Phát sinh theo tháng, cơ cấu tài khoản, so sánh thu chi
- **Thống kê tổng hợp**: Tổng thu, tổng chi, số lượng chứng từ, trạng thái ghi sổ

### 5. Quản Lý

| Chức năng          | Mô tả                                                  |
| ------------------ | ------------------------------------------------------- |
| Tài khoản          | Hệ thống tài khoản kế toán TT24, gán đối tượng, MLNS   |
| Loại đối tượng     | Khách hàng, Nhà cung cấp, Nhân viên...                  |
| Đối tượng          | Danh mục đối tượng theo dõi chi tiết                    |
| Nguồn kinh phí     | Mã nguồn tài trợ / ngân sách                            |
| Mục lục NS         | Mục lục ngân sách nhà nước                              |
| Dự toán ngân sách  | Quản lý dự toán theo mục lục + nguồn kinh phí            |
| Nhóm phiếu         | TM (Tiền mặt), NH (Ngân hàng), TH (Tổng hợp), KD (Kho dược)... |
| Phòng ban          | Quản lý đơn vị, phòng ban (hỗ trợ đa cấp)              |
| Người dùng         | Quản lý tài khoản, phân quyền ADMIN/ACCOUNTANT/APPROVER/VIEWER |
| Phân quyền menu    | Gán menu hiển thị theo vai trò người dùng               |
| Cài đặt            | Hiển thị tài khoản, đối tượng, tự động ghi sổ           |

### 6. Kiểm toán & Sao lưu

| Chức năng          | Mô tả                                                  |
| ------------------ | ------------------------------------------------------- |
| Kiểm toán (Audit)  | Truy vết toàn bộ thay đổi dữ liệu: old values ↔ new values |
| Lịch sử thao tác   | Ghi nhận mọi thao tác người dùng (CREATE, UPDATE, DELETE) |
| Sao lưu dữ liệu    | Xuất backup toàn bộ database SQLite                      |
| Phục hồi dữ liệu   | Khôi phục từ file backup                                 |
| Nhập dữ liệu SYT   | Import dữ liệu từ Sở Y tế (định dạng DBF) qua dịch vụ chuyên biệt |

### 7. Phân hệ Kho (Vật tư - Hàng hóa) 📦

Hỗ trợ các cơ sở y tế quản lý chuyên sâu và độc lập các kho vật tư, hóa chất, dược phẩm, thiết bị y tế.

- **Đa phân hệ kho độc lập**:
  - Hỗ trợ phân vùng dữ liệu và nghiệp vụ riêng biệt cho từng loại kho: **Kho Dược**, **Kho Hành chánh**, **Kho Nhà thuốc**, và **Kho Vật tư - Y cụ**.
  - Tự động thay đổi ngữ cảnh giao diện, nhãn thương hiệu (logo "KH") và tiêu đề trên Sidebar chính (ví dụ: `PHÂN HỆ KHO DƯỢC`).
  - Hỗ trợ nút **Thoát** nhanh để quay về giao diện menu tổng hợp của hệ thống.
- **Nghiệp vụ Nhập - Xuất - Chuyển kho**:
  - **Chứng từ Nhập kho**: Nhập kho từ nhà cung cấp (NHAP_NCC), Nhập trả từ khoa phòng (NHAP_TRA), và Nhập kho khác (NHAP_KHAC).
  - **Chứng từ Xuất kho**: Xuất chuyển kho nội bộ (XUAT_CK - chỉ cho phép chuyển đổi giữa các kho cùng phân hệ), Xuất bán hàng (XUAT_BAN), Xuất trả nhà cung cấp (XUAT_TRA), và Xuất khác (XUAT_KHAC).
  - **Lưới nhập liệu dòng chứng từ**: Thiết kế Excel-like liên tục, tự động đồng bộ State-DOM, hỗ trợ phím điều hướng nhanh. Các cột thông tin chi tiết: Hàng hóa, ĐVT, Số lượng, Đơn giá, Thành tiền, Số lô, Hạn dùng.
  - **Lô & Hạn dùng (Batch & Expiry Date tracking)**: Theo dõi chặt chẽ số lô, hạn sử dụng của từng mặt hàng khi nhập/xuất kho. Khi xuất kho, hệ thống tự động gợi ý danh sách các lô còn tồn kèm đơn giá tương ứng để thủ kho chọn nhanh.
  - **Ghi sổ & Bỏ ghi sổ (Post/Unpost)**: Kiểm soát trạng thái ghi sổ của chứng từ kho. Chỉ chứng từ đã ghi sổ mới cập nhật số lượng tồn kho thực tế (`quantity_on_hand`) và sinh giao dịch hạch toán (`InventoryTransaction`).
  - **Tác vụ hàng loạt (Batch Actions)**: Hỗ trợ chọn nhiều chứng từ qua checkbox để Ghi sổ hoặc Bỏ ghi sổ hàng loạt nhanh chóng từ thanh công cụ bottom toolbar.
- **Quản lý danh mục trực tiếp (Exposed Catalogs)**:
  - Giao diện danh mục được thiết kế full-width rộng rãi, loại bỏ hoàn toàn bố cục tab lồng phụ phức tạp trước đây, mang lại trải nghiệm làm việc trực quan, tối ưu không gian hiển thị.
  - Hỗ trợ quản lý trực tiếp 8 danh mục cốt lõi của phân hệ vật tư:
    1. **Vật tư - Hàng hóa**: Khai báo danh mục hàng hóa chi tiết (mã, tên, ĐVT, nhóm, loại, nước sản xuất, hãng sản xuất, giá bán, cấu hình định mức...).
    2. **Tồn kho đầu kỳ**: Khai báo số dư tồn kho đầu kỳ chi tiết theo từng kho hạch toán. Tự động đồng bộ số dư tồn kho tổng hợp của vật tư hàng hóa.
    3. **Danh mục Kho**: Quản lý danh sách các kho vật lý thuộc từng phân hệ tương ứng.
    4. **Phân hệ kho**: Quản lý và kích hoạt danh mục các phân hệ kho trong hệ thống (Kho Dược, Kho Hành chánh...).
    5. **Nhóm hàng hóa**: Phân nhóm vật tư phục vụ thống kê báo cáo.
    6. **Loại hàng hóa**: Phân loại chi tiết vật tư, hóa chất, dược phẩm.
    7. **Khoa phòng**: Quản lý danh mục các khoa phòng, bộ phận phục vụ nghiệp vụ cấp phát xuất kho hoặc nhập trả.
    8. **Phân loại Nhập xuất**: Cấu hình các tính chất, lý do nhập/xuất kho tương ứng.
- **Báo cáo Kho chuyên sâu**:
  - **Báo cáo thẻ kho**: Sổ chi tiết vật tư, hàng hóa theo từng kho, lô và khoảng thời gian hạch toán.
  - **Bảng tổng hợp Nhập - Xuất - Tồn**: Thống kê số lượng, giá trị nhập xuất tồn của tất cả mặt hàng trong kỳ.
  - **Báo cáo kho động (Dynamic reports)**: Tích hợp trình xây dựng báo cáo động để kết xuất các chỉ tiêu phân tích linh hoạt.
  - Hỗ trợ biểu mẫu in A4 Landscape tiêu chuẩn, tự động ẩn chatbot và menu điều hướng khi in.
- **Cơ chế chống tranh chấp DOM bất đồng bộ**:
  - Tích hợp kiểm tra an toàn trạng thái trang hiện tại (`localStorage.getItem('activePage')`) trước khi render Dashboard Kho nhằm ngăn chặn triệt để hiện tượng dữ liệu tải bất đồng bộ chậm ghi đè lên giao diện khi người dùng click chuyển menu nhanh.

---

## Mô hình dữ liệu

### Các model chính

```
Department            — Phòng ban / Đơn vị (hỗ trợ đa cấp parent/children)
User                  — Người dùng (ADMIN, ACCOUNTANT, APPROVER, VIEWER)
SystemMenu            — Menu hệ thống (phân quyền theo vai trò)
Account               — Tài khoản kế toán (hỗ trợ use_budget_item, object_type)
ObjectType            — Loại đối tượng theo dõi
ObjectEntity          — Đối tượng chi tiết
DocumentType          — Loại chứng từ (PT, PC, PKT, CTGS...)
DocumentGroup         — Nhóm phiếu (TM, NH, TH, KD...)
Document              — Chứng từ kế toán (header)
DocumentLine          — Chi tiết định khoản (TK Nợ/Có, ĐT, Số tiền, Mã nguồn, Số phiếu, Ngày lập)
Journal               — Sổ nhật ký chung
Ledger                — Sổ cái (số dư đầu kỳ, phát sinh, cuối kỳ)
OpeningBalance        — Số dư đầu kỳ
FundingSource         — Nguồn kinh phí
BudgetItem            — Mục lục ngân sách (đa cấp)
BudgetEstimate        — Dự toán ngân sách
FixedAsset            — Tài sản cố định
DepreciationEntry     — Hao mòn tài sản
InventoryItem         — Vật tư hàng hóa
InventoryTransaction  — Nhập xuất kho
AuditLog              — Nhật ký kiểm toán (old_values ↔ new_values JSON)
UserLog               — Lịch sử thao tác
EntryTemplate         — Từ điển định khoản (Bút toán mẫu)
SimultaneousRule      — Bút toán đồng thời
UserBehavior          — Học máy hành vi người dùng
ReportTemplate        — Báo cáo tùy chỉnh (Report Builder)
ReportTemplateConfig  — Cấu hình báo cáo động (B01-B05)
ReportTemplateLine    — Dòng chỉ tiêu báo cáo động
ReportAccountFormula  — Công thức tài khoản (ánh xạ TK → chỉ tiêu)
```

### Quan hệ chính

```
Department ──> Department (parent/children — đa cấp)
User ──> Department

Document ──> DocumentType, DocumentGroup, User (creator/approver)
Document ──< DocumentLine (1:N)
DocumentLine ──> Account (debit/credit), FundingSource, BudgetItem

Journal ──> Document, Account
Account ──> ObjectType
ObjectEntity ──> ObjectType

BudgetEstimate ──> BudgetItem, FundingSource
BudgetItem ──> BudgetItem (parent/children)

FixedAsset ──> Department
DepreciationEntry ──> FixedAsset, Document
InventoryTransaction ──> InventoryItem, Document

ReportTemplateLine ──> ReportTemplateConfig (via report_code)
ReportAccountFormula ──> ReportTemplateLine (via report_code + item_code)
```

---

## API Endpoints

### Authentication
| Method | Endpoint         | Mô tả              |
| ------ | ---------------- | ------------------- |
| POST   | `/api/auth/login` | Đăng nhập (JWT)    |

### Phiếu kế toán (Định khoản)
| Method | Endpoint                           | Mô tả                         |
| ------ | ---------------------------------- | ------------------------------ |
| GET    | `/api/accounting-vouchers`         | Lưới định khoản (lọc tháng/năm/nhóm) |
| POST   | `/api/accounting-vouchers/lines`   | Chèn dòng định khoản           |
| PUT    | `/api/accounting-vouchers/lines/:id` | Cập nhật dòng                |
| DELETE | `/api/accounting-vouchers/lines/:id` | Xóa dòng                    |
| POST   | `/api/accounting-vouchers/reindex` | Đánh lại STT                   |

### Chứng từ ghi sổ
| Method | Endpoint                    | Mô tả                    |
| ------ | --------------------------- | ------------------------- |
| GET    | `/api/documents`            | Danh sách chứng từ        |
| GET    | `/api/documents/:id`        | Chi tiết chứng từ         |
| POST   | `/api/documents`            | Tạo chứng từ + dòng       |
| PUT    | `/api/documents/:id`        | Cập nhật chứng từ          |
| DELETE | `/api/documents/:id`        | Xóa chứng từ              |
| POST   | `/api/documents/:id/post`   | Ghi sổ                    |
| POST   | `/api/documents/:id/unpost` | Bỏ ghi sổ                 |

### Phiếu Thu/Chi
| Method | Endpoint                    | Mô tả                    |
| ------ | --------------------------- | ------------------------- |
| GET    | `/api/vouchers`             | Danh sách phiếu           |
| POST   | `/api/vouchers`             | Tạo phiếu                |
| PUT    | `/api/vouchers/:id`         | Cập nhật phiếu            |
| POST   | `/api/vouchers/:id/post`    | Ghi sổ phiếu              |
| POST   | `/api/vouchers/:id/unpost`  | Bỏ ghi sổ                 |

### Danh mục
| Method | Endpoint                | Mô tả              |
| ------ | ----------------------- | ------------------- |
| GET    | `/api/accounts`         | Hệ thống tài khoản  |
| GET    | `/api/funding-sources`  | Nguồn kinh phí      |
| GET    | `/api/budget-items`     | Mục lục ngân sách    |
| GET    | `/api/budget`           | Dự toán ngân sách    |
| GET    | `/api/objects/types`    | Loại đối tượng       |
| GET    | `/api/objects/entities` | Đối tượng chi tiết   |
| GET    | `/api/objects/entities/by-account/:id` | Đối tượng theo tài khoản |
| GET    | `/api/documents/groups` | Nhóm phiếu          |
| GET    | `/api/documents/types`  | Loại chứng từ        |

### Số dư đầu kỳ
| Method | Endpoint                          | Mô tả                    |
| ------ | --------------------------------- | ------------------------- |
| GET    | `/api/opening-balances?month=&year=` | Lấy số dư đầu kỳ      |
| POST   | `/api/opening-balances/save`      | Lưu số dư đầu kỳ         |

### Báo cáo
| Method | Endpoint                                    | Mô tả                              |
| ------ | ------------------------------------------- | ----------------------------------- |
| GET    | `/api/journals`                             | Sổ nhật ký                          |
| GET    | `/api/ledgers`                              | Sổ cái                              |
| GET    | `/api/reports`                              | Báo cáo tài chính                   |
| GET    | `/api/reports/dynamic/formulas/:reportCode` | Công thức báo cáo động              |
| POST   | `/api/reports/dynamic/dry-run`              | Chạy thử công thức (in-memory)      |
| POST   | `/api/reports/dynamic/formulas/save-all/:reportCode` | Lưu toàn bộ công thức   |
| GET    | `/api/report-templates`                     | Báo cáo tùy chỉnh (Report Builder) |
| GET    | `/api/dashboard`                            | Dữ liệu dashboard                  |

### Quản trị
| Method | Endpoint                        | Mô tả                    |
| ------ | ------------------------------- | ------------------------- |
| GET    | `/api/users`                    | Danh sách người dùng      |
| GET    | `/api/departments`              | Danh sách phòng ban        |
| GET    | `/api/menus`                    | Menu hệ thống             |
| GET    | `/api/entry-templates`          | Từ điển định khoản         |
| GET    | `/api/simultaneous-rules`       | Bút toán đồng thời        |
| GET    | `/api/user-logs`                | Lịch sử thao tác          |
| GET    | `/api/audit`                    | Nhật ký kiểm toán          |

### Tài sản & Kho
| Method | Endpoint | Mô tả |
| ------ | ---------------------------------- | ------------------------------------- |
| GET | `/api/assets` | Tài sản cố định |
| GET | `/api/inventory` | Vật tư hàng hóa (danh sách & lọc) |
| POST | `/api/inventory` | Tạo vật tư hàng hóa mới |
| PUT | `/api/inventory/:id` | Cập nhật vật tư hàng hóa |
| DELETE | `/api/inventory/:id` | Xóa vật tư hàng hóa |
| GET | `/api/inventory/sub-systems` | Danh sách phân hệ kho |
| POST | `/api/inventory/sub-systems` | Tạo phân hệ kho mới |
| PUT | `/api/inventory/sub-systems/:id` | Cập nhật phân hệ kho |
| DELETE | `/api/inventory/sub-systems/:id` | Xóa phân hệ kho |
| GET | `/api/inventory/warehouses` | Danh mục kho theo phân hệ |
| POST | `/api/inventory/warehouses` | Tạo kho mới |
| PUT | `/api/inventory/warehouses/:id` | Cập nhật kho |
| DELETE | `/api/inventory/warehouses/:id` | Xóa kho |
| GET | `/api/inventory/classifications` | Danh mục phân loại Nhập/Xuất |
| POST | `/api/inventory/classifications` | Tạo phân loại nhập xuất mới |
| PUT | `/api/inventory/classifications/:id`| Cập nhật phân loại nhập xuất |
| DELETE | `/api/inventory/classifications/:id`| Xóa phân loại nhập xuất |
| GET | `/api/inventory/groups` | Danh mục nhóm hàng hóa |
| POST | `/api/inventory/groups` | Tạo nhóm hàng hóa mới |
| PUT | `/api/inventory/groups/:id` | Cập nhật nhóm hàng hóa |
| DELETE | `/api/inventory/groups/:id` | Xóa nhóm hàng hóa |
| GET | `/api/inventory/types` | Danh mục loại hàng hóa |
| POST | `/api/inventory/types` | Tạo loại hàng hóa mới |
| PUT | `/api/inventory/types/:id` | Cập nhật loại hàng hóa |
| DELETE | `/api/inventory/types/:id` | Xóa loại hàng hóa |
| GET | `/api/inventory/opening-balances` | Lấy số dư tồn kho đầu kỳ theo kho |
| POST | `/api/inventory/opening-balances/save`| Lưu số dư tồn kho đầu kỳ |
| GET | `/api/inventory/vouchers` | Danh sách chứng từ kho theo bộ lọc |
| GET | `/api/inventory/vouchers/:id` | Chi tiết chứng từ kho và các dòng |
| POST | `/api/inventory/vouchers` | Lập chứng từ kho mới |
| PUT | `/api/inventory/vouchers/:id` | Cập nhật chứng từ kho |
| DELETE | `/api/inventory/vouchers/:id` | Xóa chứng từ kho |
| POST | `/api/inventory/vouchers/:id/post` | Ghi sổ chứng từ kho |
| POST | `/api/inventory/vouchers/:id/unpost`| Bỏ ghi sổ chứng từ kho |
| GET | `/api/inventory/items/:id/batches` | Lấy danh sách số lô & hạn dùng còn tồn |
| GET | `/api/inventory/reports/summary` | Kết xuất báo cáo tổng hợp Nhập-Xuất-Tồn|

### Sao lưu & Phục hồi
| Method | Endpoint                        | Mô tả                    |
| ------ | ------------------------------- | ------------------------- |
| GET    | `/api/backup-restore/backup`    | Tạo bản sao lưu           |
| POST   | `/api/backup-restore/restore`   | Phục hồi dữ liệu          |
| POST   | `/api/backup-restore/import-syt`| Nhập dữ liệu từ Sở Y tế  |

### AI
| Method | Endpoint                        | Mô tả                    |
| ------ | ------------------------------- | ------------------------- |
| POST   | `/api/ai/chat`                  | Gửi tin nhắn AI           |
| GET    | `/api/ai/prompt-templates`      | Quản lý prompt templates   |

---

## Giao diện đa Theme

Phần mềm hỗ trợ 3 chế độ giao diện, chuyển đổi tức thời:

| Theme            | Mô tả                                    |
| ---------------- | ----------------------------------------- |
| **Light**        | Giao diện sáng mặc định, sạch sẽ         |
| **Dark**         | Chế độ tối, giảm mỏi mắt ban đêm         |
| **Eye Protection**| Tông vàng ấm, bảo vệ mắt khi làm việc lâu |

Tất cả component đều kế thừa CSS Variables (`var(--bg)`, `var(--bg-card)`, `var(--text)`, `var(--primary)`, `var(--border)`, ...) đảm bảo đồng bộ toàn hệ thống.

---

## Đăng nhập mặc định

| Tài khoản | Mật khẩu    |
| --------- | ----------- |
| admin     | admin123    |

---
## Lưu ý
Trình duyệt Chrome của bạn là phiên bản mới (từ 127 trở lên) và đã bật cờ Enables optimization guide on device và Prompt API for Gemini Nano trong chrome://flags.

## Cơ chế Bản quyền & Kích hoạt (Licensing & Activation)

Hệ thống sử dụng cơ chế xác thực bản quyền bằng chữ ký điện tử **RSA-2048** bảo mật cao:
- **Khóa công khai (Public Key)**: Được nhúng trực tiếp trong mã nguồn ứng dụng để xác thực tính hợp lệ của bản quyền.
- **Khóa bí mật (Private Key)**: Được lưu giữ riêng tư và dùng để ký (sign) file bản quyền thông qua script sinh license.

### 1. Định danh phần cứng (Machine ID)
- **Chạy trực tiếp (Local Windows)**: Machine ID được tính toán tự động dựa trên UUID phần cứng của thiết bị (truy vấn qua công cụ `wmic` của Windows).
- **Chạy trong container (Docker/VPS)**: Do chạy trong môi trường ảo hóa Docker, công cụ `wmic` không khả dụng. Ứng dụng sẽ tự động tạo một mã UUID ngẫu nhiên duy nhất cho máy đó và lưu lại bền vững tại `/data/.machine_id`. Nhờ việc mount thư mục `/data` ra volume ngoài (`ketoanai-data`), Machine ID trên VPS vẫn được giữ nguyên vẹn kể cả khi recreate hay build lại container.

### 2. Hướng dẫn sinh file Bản quyền (`.lic`)
Để sinh file bản quyền, chạy script `generate_license.js` tại thư mục gốc dự án trên máy cá nhân hoặc VPS:

```bash
# Cấu trúc câu lệnh:
node generate_license.js <Machine_ID> "<Tên_Đơn_Vị>" [Ngày_Hết_Hạn: YYYY-MM-DD] [Mã_Đơn_Vị]

# Ví dụ thực tế:
node generate_license.js 4BB9BE12665E0359D70774B489129AC0389D72119E2C3FF3EAA259CA "BỆNH VIỆN RĂNG HÀM MẶT" 2029-12-31
```

**Kết quả đầu ra**:
- Một file bản quyền định dạng `ketoanai_<8_ký_tự_đầu_machine_id>.lic` (ví dụ: `ketoanai_4BB9BE12.lic`) sẽ được tạo ra tại thư mục chạy lệnh.
- File này chứa thông tin đơn vị được cấp phép, thời hạn sử dụng và chữ ký điện tử tương ứng.

### 3. Các bước Kích hoạt phần mềm
1. Truy cập vào giao diện phần mềm KeToanAI trên trình duyệt hoặc Electron.
2. Điều hướng tới mục **Cài đặt / Kích hoạt bản quyền**.
3. Chọn tab **Kích hoạt Offline (.lic)**.
4. Tải (Upload) file `.lic` vừa sinh lên hệ thống.
5. Phần mềm sẽ tự động kiểm tra chữ ký số và mở khóa đầy đủ các tính năng kế toán.

---

## Tối ưu hóa Hiệu năng (Performance Optimization)

Để đạt được hiệu năng cao nhất trên cả môi trường cục bộ (Tauri/Electron) và môi trường máy chủ đám mây (VPS/Docker Swarm), hệ thống đã được triển khai các kiến trúc tối ưu hóa nâng cao:

### 1. Tối ưu hóa quy trình Khởi động & Tải lại trang (F5)
* **Song song hóa Script (`async = false`)**: Thay thế cơ chế tải tuần tự chặn luồng (blocking) bằng cách tải song song tất cả 21 file script nghiệp vụ, đồng thời duy trì đúng thứ tự thực thi bằng cờ `async = false`.
* **Browser Caching**: Sử dụng mã phiên bản tĩnh `?v=X.XX` thay vì dùng timestamp động, giúp trình duyệt lưu đệm (cache) hoàn toàn mã nguồn nghiệp vụ. Tải lại trang sau lần đầu tiên diễn ra tức thì.
* **Gom nhóm API (`Promise.all`)**: Nhập chung các truy vấn hệ thống lúc mở app (`/api/system/status`, `/api/license/status`, `/api/system/config`) thành một phiên làm việc duy nhất để giảm RTT. Loại bỏ hoàn toàn độ trễ chào mừng 1 giây.
* **Kết quả**: Thời gian tải lại trang (F5) trên VPS giảm từ **60 giây+** xuống dưới **1.2 giây**.

### 2. Tối ưu hóa Bản quyền & Caching mã máy
* **RAM Caching**: Mã phần cứng (`Machine ID`) của thiết bị được tính toán một lần duy nhất tại thời điểm khởi động máy chủ và được lưu vào RAM (`cachedMachineId`). Toàn bộ các API nghiệp vụ khi kiểm tra chéo bản quyền sẽ đọc trực tiếp từ RAM trong **0.003ms** thay vì chạy lại lệnh shell.
* **Bỏ qua gọi lệnh hệ thống trên Linux**: Nếu phát hiện chạy trên Linux/macOS/Docker, backend bỏ qua hoàn toàn lệnh shell `wmic` của Windows, truy xuất trực tiếp file UUID ảo. Loại bỏ hoàn toàn tình trạng nghẽn Event Loop và hết tiến trình con (child process exhaustion) của Node.js.

### 3. Tối ưu hóa Công thức Báo cáo & Đồng bộ Sổ cái
* **Thuật toán Đồng bộ Lũy kế $O(N)$**: Thay vì quét toàn bộ dữ liệu từ đầu năm đến tháng hiện tại (YTD table scan) gây ra độ phức tạp lũy kế $O(N^2)$ khi chạy các bút toán đồng bộ, hệ thống áp dụng cơ chế đồng bộ lũy kế tịnh tiến từng tháng liền kề. Dữ liệu tháng hiện tại kế thừa trực tiếp số dư cuối kỳ của tháng trước đó.
* **Ngắt vòng lặp kiểm tra sớm (Early Break)**: Khi phát hiện bất kỳ tháng nào bị lệch số dư và được sửa đổi (healing), hệ thống sẽ chạy cập nhật lan tỏa đến hết tháng 12 và kết thúc vòng lặp ngay lập tức. Bỏ qua các bước kiểm tra trùng lặp trên các tháng tiếp theo đã được đồng bộ.
* **Kết quả**: Thời gian kiểm tra và in toàn bộ Báo cáo tài chính trên VPS giảm từ **12 giây** xuống chỉ còn **10ms - 30ms** (warm run).

---

## License

MIT © KeToanAI


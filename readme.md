# 🚀 Private Workspace & AI Hub (Zero-Trust Architecture)

Hệ thống làm việc chung khép kín (All-in-One Private Workspace) được thiết kế tối ưu cho nhóm nhỏ (5–10 người), chạy hoàn toàn trên máy chủ riêng, bảo mật tuyệt đối qua mạng riêng ảo **Tailscale** và xác thực một chạm qua **Authentik (SSO)**.

## 📦 Các thành phần trong hệ thống

1. **Tailscale Sidecar:** Cổng vào duy nhất (Zero-Trust Gateway), tự động cấp chứng chỉ HTTPS nội bộ (`.ts.net`) mà không cần mở port ra Internet hay cài Reverse Proxy thủ công.
2. **Authentik (SSO):** Hệ thống quản lý danh tính tập trung — 1 tài khoản duy nhất cho mỗi nhân viên để đăng nhập toàn bộ hệ thống.
3. **Nextcloud & OnlyOffice:** Lưu trữ file, ghi chú, lịch, họp nhóm (*Nextcloud Talk*) và chỉnh sửa tài liệu văn bản (`.docx`, `.xlsx`, `.pdf`) trực tiếp theo thời gian thực.
4. **Planka:** Bảng phân công công việc trực quan (Kanban Board) dạng Trello, đồng bộ tài khoản qua OIDC.
5. **Llama.cpp & NextChat:** Trợ lý AI nội bộ chạy model mã nguồn mở (như Qwen2.5 / Llama 3) qua giao diện chat riêng tư, không lộ dữ liệu ra ngoài.
6. **SurfSense:** AI agent xử lý tài liệu giống NotebookLM, có thể truy cập file từ NextCloud qua WebDAV và xác thực qua Authentik OIDC.
7. **n8n:** Hệ thống tự động hóa trung tâm (Workflow Automation) dùng để đồng bộ dữ liệu công việc, nhắc lịch và đẩy thông báo xuống các ứng dụng nhắn tin.

---

## 🛠️ Yêu cầu trước khi cài đặt

- Máy chủ đã cài đặt **Docker** và **Docker Compose**.
- Tài khoản **Tailscale** cá nhân.
- Một tệp mô hình AI định dạng `.gguf` (ví dụ: `qwen2.5-7b-instruct-q4_k_m.gguf`) đặt trong thư mục `workspace/models/`.

---

## ⚙️ Hướng dẫn cài đặt nhanh

### Bước 1: Tạo Auth Key trên Tailscale
1. Truy cập [Tailscale Admin Console](https://login.tailscale.com/admin/settings/keys).
2. Tạo một **Auth Key** mới:
   - Bật tùy chọn **Reusable** (Tái sử dụng).
   - Gán tag: `tag:workspace-server`.
3. Sao chép lại chuỗi khóa (`tskey-auth-...`).

### Bước 2: Cấu hình tệp `.env`
Mở tệp `env` điền thông tin Tailscale của bạn và chuyển nó thành `.env`:
```env
TS_HOSTNAME=workspace-hub
TS_DOMAIN=workspace-hub.ten-tailnet.ts.net
TS_AUTHKEY=tskey-auth-khoa-cua-ban-o-day
LLAMA_MODEL_FILE=qwen2.5-7b-instruct-q4_k_m.gguf

```

### Bước 3: Khởi chạy hệ thống

Chạy lệnh sau tại thư mục chứa `docker-compose.yml`:

```bash
docker compose up -d

```

---

## 🌐 Các đường dẫn truy cập qua Tailscale

Sau khi container khởi động và kết nối thành công với Tailscale, bạn có thể truy cập qua các tên miền nội bộ `.ts.net`:

* **Nextcloud:** `https://workspace-hub.ten-tailnet.ts.net`
* **OnlyOffice:** `https://workspace-hub.ten-tailnet.ts.net/onlyoffice`
* **Authentik (SSO):** `https://workspace-hub.ten-tailnet.ts.net/authentik`
* **Planka (Kanban):** `https://workspace-hub.ten-tailnet.ts.net/planka`
* **NextChat (AI):** `https://workspace-hub.ten-tailnet.ts.net/nextchat`
* **SurfSense (AI Document Agent):** `https://workspace-hub.ten-tailnet.ts.net/surfsense`
* **n8n (Automation):** `http://<IP-Tailscale>:5678` *(Chỉ dành cho Admin theo phân quyền ACL - truy cập trực tiếp qua IP Tailscale)*

---

## 🔧 Cấu hình Authentik cho SurfSense

Sau khi hệ thống khởi động, bạn cần cấu hình Authentik để SurfSense có thể sử dụng OIDC:

1. Đăng nhập vào Authentik tại `https://workspace-hub.ten-tailnet.ts.net/authentik`
2. Tạo một Provider mới:
   - **Type:** OpenID Connect
   - **Name:** SurfSense
   - **Client ID:** `surfsense-client-id`
   - **Client Secret:** Lấy từ biến môi trường `SURFSENSE_OIDC_CLIENT_SECRET`
   - **Redirect URIs:** `https://workspace-hub.ten-tailnet.ts.net/surfsense/api/auth/callback/oidc`

3. Tạo một Application mới:
   - **Name:** SurfSense
   - **Slug:** `surfsense`
   - **Provider:** Chọn provider SurfSense vừa tạo

4. Gán quyền truy cập cho người dùng cần thiết

---

## 📁 Truy cập file từ NextCloud trong SurfSense

SurfSense đã được cấu hình để truy cập file từ NextCloud qua WebDAV:
- **WebDAV URL:** `https://workspace-hub.ten-tailnet.ts.net/remote.php/dav/files/{username}`
- **Username:** `admin` (hoặc username của bạn trong NextCloud)
- **Password:** Mật khẩu NextCloud của bạn

Trong SurfSense, bạn có thể:
1. Thêm nguồn từ NextCloud WebDAV
2. Upload file trực tiếp
3. Sử dụng AI để nghiên cứu, chuyển đổi và chỉnh sửa tài liệu
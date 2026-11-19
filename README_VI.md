# 🚀 Google AI Trial Tool

[English](README.md) | [Tiếng Việt](README_VI.md)

> ⚠️ **CẢNH BÁO:** Công cụ này sử dụng kỹ thuật can thiệp tiến trình (Frida injection) và **có nguy cơ cao bị khóa tài khoản (ban)**. Hãy cân nhắc kỹ trước khi sử dụng và không dùng trên tài khoản Google chính.

Công cụ sử dụng **Frida** để tiêm (inject) vào tiến trình Google One, gọi trực tiếp `BillingClient` API nhằm phát hiện và kích hoạt các gói dùng thử miễn phí **$0** bị ẩn.

---

## 📋 Yêu cầu chuẩn bị

| Thành phần | Mô tả |
| :--- | :--- |
| **Giả lập MuMu** | Đã khởi động, đăng nhập tài khoản Google và đã liên kết phương thức thanh toán |
| **adb** | Đã thêm vào `PATH`, hoặc thiết lập qua biến môi trường `ADB_PATH` |
| **frida (máy tính)** | Cài bằng `pip install frida-tools`, phiên bản `>= 16` (khuyên dùng **16.7.19**) |
| **frida-server (thiết bị)** | Đẩy vào `/data/local/tmp/frida-server` và cấp quyền thực thi |
| **module.apk (thiết bị)** | File APK chứa thư viện `BillingClient`, đẩy vào `/data/local/tmp/module.apk` |
| **Google One** | Đã cài đặt trên giả lập (`com.google.android.apps.subscriptions.red`) |

> **Lưu ý:** Frida `>= 17` có thay đổi lớn gây lỗi tương thích (Java bridge cần load tường minh). Khuyến nghị sử dụng **Frida 16.7.19**.

---

## 🛠️ Thiết lập trên thiết bị

```bash
# Kết nối với giả lập MuMu
adb connect 127.0.0.1:16384

# Đẩy frida-server lên thiết bị (tải đúng kiến trúc CPU tại https://github.com/frida/frida/releases)
adb push frida-server /data/local/tmp/frida-server
adb shell "chmod 755 /data/local/tmp/frida-server"

# Đẩy module.apk (file APK có chứa com.android.billingclient)
adb push module.apk /data/local/tmp/module.apk
```

---

## 💻 Cách sử dụng

```bash
# Liệt kê tất cả các gói dùng thử khả dụng của tài khoản hiện tại
./auto-trial.sh --list

# Kích hoạt gói Pro (5TB, $19.99/tháng) dùng thử 3 tháng miễn phí
./auto-trial.sh --product pro --dur 3mo

# Kích hoạt gói Pro biến thể (5TB, $19.99/tháng) dùng thử 12 tháng miễn phí ★ Khuyên dùng
./auto-trial.sh --product pro12 --dur 12mo

# Kích hoạt gói Ultra 20x (30TB, $199.99/tháng) dùng thử 12 tháng miễn phí
./auto-trial.sh --product 20x --dur 12mo

# Kích hoạt gói Ultra 5x (20TB, $99.99/tháng) dùng thử 6 tháng miễn phí
./auto-trial.sh --product 5x --dur 6mo

# Tự động chọn gói dùng thử dài nhất (mặc định chọn gói cao nhất)
./auto-trial.sh

# Chỉ định cổng ADB của giả lập MuMu
./auto-trial.sh --port 16416 --product pro --dur 3mo
```

---

## ⚙️ Tham số & Biến môi trường

### Tham số dòng lệnh
| Tham số | Mô tả | Mặc định |
| :--- | :--- | :--- |
| `--list` | Chỉ liệt kê gói dùng thử, không kích hoạt | *(Chế độ kích hoạt)* |
| `--product` | `pro` / `pro12` / `5x` / `20x` / `auto` / ID đầy đủ `g1.*` | `auto` |
| `--dur` | `1mo` / `3mo` / `6mo` / `12mo` / `longest` | `longest` |
| `--port` | Cổng ADB của giả lập MuMu | Tự động quét |
| `--adb` | Đường dẫn đến file thực thi `adb` | Tự động tìm |

### Biến môi trường
| Biến | Mô tả |
| :--- | :--- |
| `ADB_PATH` | Đường dẫn file thực thi `adb` |
| `FRIDA_SCRIPT` | Đường dẫn tới `frida-auto-trial.js` (mặc định cùng thư mục) |
| `FRIDA_SERVER_PATH` | Đường dẫn `frida-server` trên thiết bị (mặc định: `/data/local/tmp/frida-server`) |
| `MODULE_APK_DEVICE` | Đường dẫn `module.apk` trên thiết bị (mặc định: `/data/local/tmp/module.apk`) |

---

## 📦 Bảng đối chiếu Product ID

| Tên rút gọn | Product ID | Dung lượng | Giá/tháng | Thời gian dùng thử |
| :--- | :--- | :--- | :--- | :--- |
| `pro12` ★ | `g1.2tb.ai.subs_variant` | 5TB | $19.99 | 6 / 12 tháng |
| `pro` | `g1.play.2tb.ai` | 5TB | $19.99 | 3 tháng |
| `5x` | `g1.play.20tb` | 20TB | $99.99 | 1 / 3 / 6 / 12 tháng |
| `20x` | `g1.play.ai.pro` | 30TB | $199.99 | 1 / 3 / 6 / 12 tháng |
| — | `g1.2tb.ai` | 5TB | $19.99 | Không có |
| — | `g1.ai.pro` | 30TB | $199.99 | Không có |
| — | `g1.30tb` | 30TB | $149.99 | Không có |

> **Cập nhật (19/11/2026):** Nút "Nâng cấp" mặc định trong app trỏ tới các ID không có dùng thử (`g1.2tb.ai`, v.v.). Các gói dùng thử Ultra `5x`/`20x` hiện đã bị gỡ bỏ. Hiện tại gói Pro còn 2 ID khả dụng:
> - `g1.2tb.ai.subs_variant` (`pro12`): Tối đa **12 tháng miễn phí** (ưu tiên dùng).
> - `g1.play.2tb.ai` (`pro`): **3 tháng miễn phí**.

---

## ❌ Tại sao cách làm trên trình duyệt (Web) không khả thi?

Qua thử nghiệm, các phương pháp trên web đều không thể vượt qua bước xác thực phía máy chủ của Google Play:
- **URL tĩnh:** Các tham số `rpctoken`, `_gfid` và payments blob bị khóa theo phiên (session) và sẽ hết hạn ngay lập tức.
- **Sửa tham số `ear` bằng Console:** Chuỗi `ear` (25 byte) lưu mã sản phẩm tại `ear[13]` và chữ ký toàn vẹn tại `ear[24]`. Khi đổi sản phẩm, chữ ký không khớp khiến máy chủ trả về lỗi `400`.
- **Chrome Extension RPC Hook:** Các giải pháp can thiệp payload `Ngbm` RPC trong cộng đồng hiện đã bị xóa (lỗi 404).

**Kết luận:** Các gói dùng thử ẩn (như `g1.play.2tb.ai`) không hiển thị trên giao diện web nên không thể tạo phiên thanh toán hợp lệ. Cách duy nhất đáng tin cậy là tiêm Frida để gọi `BillingClient` API trực tiếp trên thiết bị.

---

## ⚠️ Lưu ý quan trọng

- Script chỉ bật hộp thoại thanh toán của Google Play, bạn cần **xác nhận thủ công** (nhập mật khẩu).
- Gói sẽ **tự động gia hạn** sau khi hết hạn dùng thử — hãy nhớ hủy trước ngày gia hạn.
- Tài khoản bắt buộc phải **đã liên kết thẻ/phương thức thanh toán** và **không có gói đăng ký nào đang hoạt động**.
- Nếu `launchBillingFlow` trả về `code=0`, lượt dùng thử có thể đã bị tính là sử dụng ngay cả khi popup thanh toán không hiện lên.
- Toàn bộ thao tác thực hiện trên giả lập; nên liên kết thẻ có số dư nhỏ (hoặc thẻ ảo) để đảm bảo an toàn.

---

## 📁 Danh sách tệp tin

- `auto-trial.sh` — Script Shell khởi chạy, tự động phát hiện thiết bị, quản lý `frida-server`, tiêm script và theo dõi kết quả.
- `frida-auto-trial.js` — Script Frida dùng `DexClassLoader` để nạp `BillingClient`, truy vấn và kích hoạt gói dùng thử.
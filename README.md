# B0DY — web (b0dy.studio, một trang duy nhất)

MỘT trang tĩnh: landing + card đặt lịch (kính mờ) đè lên nhau. Không framework, không build.
Backend là một dự án Apps Script **riêng**, ghi vào sheet `BOOKING` của file `[B0DY Studio] BA`.

```
index.html              → b0dy.studio  (landing + card đặt lịch #booking)
booking/index.html      → trang chuyển hướng: /booking → /#booking
assets/fonts/           → chữ tự lưu, không gọi CDN ngoài
assets/favicon.svg
media/hero.mp4|.webm    → video nền  ⚠ ĐANG LÀ BẢN TẠM (chờ file "Video demo")
media/hero-poster.jpg   → khung hình đầu, hiện ngay khi chưa tải xong video
Code.gs                 → dán vào Apps Script (KHÔNG đưa lên GitHub cũng được)
test.js                 → harness test Playwright
```

---

## Bốn việc cần làm để lên sóng

### 1. Tạo sheet BOOKING + backend

1. Mở file **[B0DY Studio] BA** ▸ `Extensions` ▸ `Apps Script`.
   (Script gắn kèm file — không phải chép ID bảng tính, tránh sai một ký tự.)
2. Đổi tên dự án thành **B0DY Booking API**.
3. Xoá `myFunction`, dán toàn bộ `Code.gs`, Ctrl+S. Chờ thấy **"Saved to Drive"**.
4. Chọn hàm `setupBooking` trong dropdown cạnh nút Run ▸ **Run** ▸ Allow quyền.
   → Sheet `BOOKING` xuất hiện, đã có sẵn cấu hình 4 ca **mẫu**.
   ⚠ Trước khi bấm Run, bấm **OK** trên hộp thoại tài khoản Google nếu nó hiện ra, và
   **nhìn kỹ tên hàm trong dropdown** — chạy nhầm hàm đã từng tạo tab rác một lần.
5. `Deploy` ▸ `New deployment` ▸ type **Web app** ▸ tên `Booking API v1`
   ▸ Execute as **Me** ▸ Who has access **Anyone** ▸ Deploy.
6. Copy URL **/exec**.

### 2. Cắm URL vào 2 trang

Trong `index.html`, tìm dòng:

```js
var API = 'PASTE_EXEC_URL_HERE';
```

thay bằng URL `/exec` vừa copy. **Không gõ tay** — copy dán.

> Chưa cắm URL thì hai trang vẫn chạy ở **chế độ xem trước** với lịch giả — tiện để duyệt giao diện trước.

### 3. Đưa lên GitHub Pages

1. Tạo repo mới `b0dy-web` (public).
2. Kéo thả toàn bộ thư mục vào trang upload của GitHub. **Không cần đưa `Code.gs` và `test.js`.**
3. Bấm **Commit changes** → **chụp lại trang repo, thấy hash + "now" mới coi là xong.**
   Nút này đã từng im lặng không ăn hai lần.
4. `Settings` ▸ `Pages` ▸ Source **Deploy from a branch** ▸ `main` / `root`.
5. Custom domain: `b0dy.studio`. Tạo file `CNAME` nội dung `b0dy.studio`.

Đặt lịch nằm ngay tại `b0dy.studio/booking` (chuyển về `/#booking`) — KHÔNG cần repo thứ hai, không cần subdomain.

| Bản ghi | Trỏ tới |
|---|---|
| `A` @ | 185.199.108.153 · 185.199.109.153 · 185.199.110.153 · 185.199.111.153 |

---

## Cấu hình ca — sheet BOOKING, cột K trở sang phải

Sửa trực tiếp trên sheet, web tự cập nhật trong vòng ~1 phút. Không cần đụng code.

```
      K     L        M      N      O      P      Q      R     S      T
 4   CA    GIỜ      T2     T3     T4     T5     T6     T7    CN     CHỖ
 5   01   07:00   Quyết   Mây   Quyết   Mây   Quyết  Định           1
 6   02   09:00   Mây     Định  Mây     Định  Mây    Khải           1
 7   03   18:00   Quyết   Quyết Quyết   Quyết Quyết                 1
 8   04   20:00   Khải    Lâm   Khải    Lâm   Khải                  1

      V                X                            Y
 4   NGÀY ĐÓNG        THAM SỐ                       GIÁ TRỊ
 5   9/2/2026         Số ngày mở đặt                10
 6                    Đặt trước tối thiểu (giờ)     2
 7                    Thời lượng (phút)             60
```

- **Ô trống = ca đó không mở** ngày hôm đó.
- Thêm ca thứ 5 = thêm một dòng dưới dòng 8.
- `CHỖ` để trống hiểu là 1.
- `NGÀY ĐÓNG` ghi kiểu `M/d/yyyy` — đóng nguyên ngày (nghỉ lễ, coach bận).

Dữ liệu khách đặt nằm ở cột **A–J**: `# · Ngày · Thứ · Khung giờ · Coach · Tên khách ·
Số điện thoại · Meta handle · Trạng thái · Ghi chú`.
Trạng thái mặc định là `Chờ xác nhận`; đổi thành `Huỷ` sẽ **trả lại chỗ trống** cho web.

---

## Vì sao trang không phải chờ loading

Sàn tốc độ của Apps Script là **~1,3s mỗi lượt gọi** — không tối ưu code nào gỡ được.
Cách chữa là bỏ lượt gọi ra khỏi đường chờ của khách:

1. **Landing hâm nóng + lấy trước.** Ngay khi khách mở b0dy.studio, trang bắn `ping`
   (đánh thức instance, tránh cold start 20–60s) và lấy luôn lịch trống. Khách xem video
   vài giây là dữ liệu đã về.
2. **Card cùng trang.** Đặt lịch là card mở đè lên landing — dữ liệu đã nằm sẵn trong bộ nhớ,
   không còn bước bàn giao giữa hai trang.
3. **Bản cũ trong máy.** Khách quay lại thì vẽ lưới từ `localStorage` trước, cập nhật ngầm sau.
4. **Ghi lạc quan có đường lùi.** Bấm Xác nhận là sang màn kết quả sau ~60ms, trạng thái ghi
   đúng là *"Đang giữ chỗ…"* rồi mới chốt *"Đã giữ chỗ"*. Nếu ca vừa hết hoặc mất mạng,
   màn hình nói thẳng là chưa đặt được và mời chọn giờ khác — **không bao giờ báo thành công
   khi chưa thành công.**

---

## Hai việc còn treo

1. **Video nền đang là bản tạm** (nhiễu tối, 6 giây) — chỉ để kiểm tra cơ chế và xem bố cục.
   Khi có footage thật: nén **≤ 8MB**, 1080p, ~12–15s, loop liền mạch, không tiếng, rồi thay
   `media/hero.mp4` + `media/hero.webm` + cắt lại `media/hero-poster.jpg`.
   (Giao diện web GitHub chỉ cho upload file ≤ 25MB.)
2. **Chữ.** Switzer bản gốc của Fontshare là bộ Latin 385 ký tự — **không có dấu tiếng Việt**.
   Trang đang tạm dùng **Be Vietnam Pro Medium 500** (đủ dấu, cùng tinh thần grotesque).
   Khi có file **SVN-Switzer Medium** (bản Việt hoá đang dùng trong Figma), xuất ra `.woff2`,
   để vào `assets/fonts/SVN-Switzer-Medium.woff2` và bỏ chú thích khối `@font-face` ở đầu
   `<style>` của cả hai trang — không phải sửa gì thêm.

---

## Chạy test trước mỗi lần deploy

```bash
node test.js
```

Mock toàn bộ mạng ngoài, chạy 8 kịch bản (lưới, bàn giao dữ liệu, đặt thành công, ca vừa hết,
mất mạng, sai số điện thoại, hết ca, dấu chồng tiếng Việt) và chụp ảnh vào `shots/`.
**Còn PAGEERR là không deploy.**

## Sửa Apps Script

Sửa `Code.gs` **chưa đủ để live**. Bắt buộc:
`Deploy` ▸ `Manage deployments` ▸ ✏️ ▸ Version **New version** ▸ `Deploy`.
URL `/exec` không đổi.

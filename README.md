# B0DY — web (b0dy.studio, một trang duy nhất)

MỘT trang tĩnh: landing + card đặt lịch (kính mờ) đè lên nhau. Không framework, không build.
Backend là dự án Apps Script **độc lập** tên **B0DY Booking API**, ghi vào sheet `BOOKING`
của file `[B0DY Studio] BA`.

```
index.html              → b0dy.studio  (landing + card đặt lịch #booking)
booking/index.html      → trang chuyển hướng: /booking → /#booking
assets/fonts/           → chữ tự lưu (SVN-Switzer Medium + JetBrains Mono), không gọi CDN ngoài
assets/favicon.svg
media/hero.mp4          → video nền (từ "Video demo", faststart, không tiếng)
media/hero-poster.jpg   → khung hình đầu, hiện ngay khi chưa tải xong video
CNAME                   → b0dy.studio (GitHub Pages custom domain)
Code.gs                 → bản sao code backend (KHÔNG đưa lên GitHub public)
test.js                 → harness test Playwright
```

---

## HIỆN TRẠNG (14/08/2026) — ĐÃ LÊN SÓNG

- **Backend**: Apps Script độc lập "B0DY Booking API", Web App **Version 6**, Execute as Me /
  access Anyone. URL `/exec` đã cắm sẵn trong `index.html`. Đã test thật trên production:
  schedule · book · het_cho · da_dat đều đúng, dòng test đã dọn.
- **Frontend**: repo GitHub `quangvinh376/b0dy-web`, GitHub Pages bật từ `main` / root,
  custom domain `b0dy.studio` (file CNAME).
- **DNS**: tên miền đang nằm ở **Squarespace** — cần thay 4 bản ghi A của @ bằng
  185.199.108.153 · 185.199.109.153 · 185.199.110.153 · 185.199.111.153
  và CNAME `www` → `quangvinh376.github.io`, rồi bật **Enforce HTTPS** trong Settings ▸ Pages.

## Backend — cách nó chạy (đã dựng xong, chỉ đọc để hiểu)

1. Dự án Apps Script **độc lập** (script.google.com), KHÔNG gắn kèm file. File BA được tìm một
   lần bằng tên `[B0DY Studio] BA` qua DriveApp khi chạy `setupBooking`, rồi lưu ID vào
   Script Properties (`BA_ID`) — không chép tay ID.
2. Múi giờ luôn ép `Asia/Ho_Chi_Minh` trong code (múi giờ file là US Pacific — không tin nó).
3. Sửa `Code.gs` **chưa đủ để live**: phải `Deploy` ▸ `Manage deployments` ▸ ✏️ ▸
   Version **New version** ▸ `Deploy`. URL `/exec` không đổi.

---

## Cấu hình ca — sheet BOOKING, cột K trở sang phải (hàng 4–8)

Sửa trực tiếp trên sheet, web tự cập nhật trong vòng ~1 phút. Không cần đụng code.
⚠ Cấu hình hiện tại là **dữ liệu mẫu** do máy ghi — sửa lại tên coach + giờ cho đúng lịch thật.

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

Dữ liệu khách đặt: **giữ nguyên khung sẵn có của sheet** — tiêu đề hàng 4, dữ liệu từ hàng 5,
cột **A–H**: `# · Ngày hẹn · Giờ hẹn · Tên · Handle · Số điện thoại · Trạng thái · Ghi chú`.
Web ghi Trạng thái = `Hẹn tập thử` (đúng danh sách data validation của cột G).
Đổi thành `Huỷ` hoặc `Loại` sẽ **trả lại chỗ trống** cho web.
Coach + thứ nằm trong Ghi chú, dạng `Coach X · T2 · web 14/8 09:00`.

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
   đúng là *"ĐANG ĐẶT LỊCH…"* rồi mới chốt *"ĐẶT LỊCH THÀNH CÔNG"*. Nếu ca vừa hết hoặc
   mất mạng, màn hình nói thẳng là chưa đặt được và mời chọn giờ khác — **không bao giờ báo
   thành công khi chưa thành công.**

---

## Việc còn treo

1. **Zalo ZNS (giai đoạn 2)** — chưa có Zalo OA. Hiện coach nhắn tay theo sheet BOOKING.
2. **Social links là placeholder** — footer đang trỏ `instagram.com/b0dy.studio` và
   `facebook.com/b0dy.studio`; thay bằng handle thật trong `index.html`.
3. **Cấu hình ca trong sheet là dữ liệu mẫu** — sửa lại theo lịch coach thật.

---

## Chạy test trước mỗi lần deploy

```bash
node test.js
```

Mock toàn bộ mạng ngoài, chạy đủ kịch bản (lưới 10 ngày, 3 bước + coach, đặt thành công với
server chậm 1.3s, ca vừa hết, mất mạng, sai số điện thoại, EN/VN + nhớ lựa chọn, mono viết hoa)
và chụp ảnh vào `shots/`. **Còn PAGEERR là không deploy.**

## Sửa Apps Script

Sửa `Code.gs` **chưa đủ để live**. Bắt buộc:
`Deploy` ▸ `Manage deployments` ▸ ✏️ ▸ Version **New version** ▸ `Deploy`.
URL `/exec` không đổi.

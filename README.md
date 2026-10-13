# gfw-links-dnk

Pipeline tự động tạo và duy trì link bản đồ Global Forest Watch (GFW) cho ~340 lô cao su vườn DNK (Campuchia), phục vụ truy xuất nguồn gốc EUDR. Cùng cơ chế với repo `gfw-links-donaruco`.

## Sản phẩm bàn giao

`docs/links.json` — danh sách link GFW theo mã lô (`ProductionPlace`, vd `5.08NK.D3.11.082`), tự cập nhật hằng đêm.

Quy tắc đọc cho hệ thống truy xuất: lấy `long_url` theo `id_lo`; chỉ hiển thị khi `status` là `alive`/`healed`; `last_checked` thể hiện độ tươi dữ liệu.

## Cơ chế

- **SYNC** (01:30 VN hằng đêm): query toàn bộ lô từ AGOL Feature Layer `DNK_2026`, POST geostore GFW cho lô mới/đổi ranh, cập nhật `docs/links.json`.
- **HEAL** (08h30/16h30/00h30 VN): kiểm tra từng geostore còn sống; nếu bị xóa thì POST lại đúng GeoJSON — nhờ tính determinism của geostore v1, ID (và URL) không đổi.
- Biến động (lô mới, đổi ranh, lô bị xóa, link chết) báo qua Telegram, tin nhắn có tiền tố `[DNK]`.
- Lịch chạy lệch 30 phút so với repo Donaruco để hai pipeline không gọi GFW cùng lúc.
- Layer DNK không có field hiện trạng (`hTrang`), nên khi trùng mã lô sẽ lấy bản ghi đầu và cảnh báo.

## Cấu hình

Secrets cần khai báo trong Settings → Secrets and variables → Actions:
`AGOL_USERNAME`, `AGOL_PASSWORD`, `AGOL_LAYER_URL`, `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID`.

`AGOL_LAYER_URL` = `https://services-ap1.arcgis.com/<org>/arcgis/rest/services/DNK_2026/FeatureServer/0`.

`map_template.txt` chứa phần query string cấu hình layer (Tree cover loss 2021→nay + Tree plantations). Khi GFW ra dữ liệu TCL năm mới (~tháng 4): cập nhật template (ở cả repo Donaruco), xóa `docs/links.json`, chạy tay SYNC để rebuild toàn bộ URL.

# Báo cáo Day 13 — Nguyễn Thành Thân, 2A202603202

## 1. Kết luận trước
Mã số `2A202603202`, task `D13-2A202603202` (CVAT task 29, job 29), 10 frame: s01f00, s01f40, s02f06, s02f26, s03f07, s03f32, s04f00, s04f29, s05f10, s05f34.

Hệ điều hành Windows, CVAT server 2.82.1 (cvat-sdk 2.83.0, Python 3.13). Trình duyệt: Chrome. Không ngồi cạnh bạn nào (làm riêng).

Tổng 109 cuboid (88 Car, 8 Bus, 3 Truck, 7 Cyclist, 3 Pedestrian). Frame nhiều nhất là s01f00 (23 cuboid, giao lộ đông xe); frame ít nhất là s03f32 (5 cuboid) vì đoạn đường thưa xe và phần lớn vật nằm xa 25–38 m. `selfcheck` cờ 38/109 cuboid (chủ yếu kích thước ngoài khoảng tham chiếu và đáy lệch mặt đường > 0,3 m). Tôi tin nhất phần xe con gần (< 20 m): hướng và đáy (ví dụ s01f00 idx 1, s03f32 idx 2 không bị cờ); yếu nhất là vật xa 30–60 m ở s01f00 (chỉ 59–111 điểm) và kích thước xe buýt/SUV (bề rộng hay vượt bảng 4.3).

Đổi góc nhìn: kéo chuột trái để xoay, chuột phải để dời, lăn để phóng to; xoay khung 3D tới khi nhìn thẳng xuống để có góc nhìn từ trên. Vẽ: chọn label → Shape → nháy đúp lên cụm điểm, rồi chỉnh tâm, dài, rộng, yaw ở Top và đáy, chiều cao ở Side/Front. Hướng: bật Appearance → Cuboid orientation; xem cuboid trên 7 ảnh camera bằng `python scripts/lab.py preview`.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ. Ảnh: `t1-s01f00-3d`, `t1-s01f00-top`, `t1-s01f40-3d`, `t1-s01f40-top` (ảnh top có đánh dấu phía trước, +x, +y).

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f00 | Phía trước (−y) ở cạnh phải ảnh; +x (trái xe) chỉ lên cạnh trên, +y (sau xe) chỉ sang cạnh trái | Giao lộ, gần phẳng, hơi thấp dần về phía trước; thấp hơn LiDAR ~1,8 m quanh xe, ~2,2 m ở 40 m phía trước | ~61 m (Bus idx 16, bên trái phía trước) | Car, Bus, Truck, Cyclist, Pedestrian | Sau: 5 xe con (idx 1–5, 10–34 m). Trái: xe con sát bên 3,4 m (idx 18), cyclist idx 13, 17. Phải: Bus idx 8 (32 m), Car idx 7, 2 cyclist, 2 người đi bộ (idx 11, 12) | Cụm điểm thấp lẻ tẻ ngoài 50 m phía trước, không rõ là xe hay cột/dải phân cách |
| s01f40 | Ảnh top xoay khác s01f00: phía trước (−y) ở cạnh trên ảnh (hơi nghiêng trái); +x chỉ sang cạnh trái (trái xe), +y chỉ xuống cạnh dưới (sau xe) | Phẳng; thấp hơn LiDAR ~1,8 m | ~33 m (Car idx 6 phía sau, Car idx 3 phía trước 32 m) | Chỉ Car (6 cuboid), tất cả cùng chiều (yaw 263–269°) | Sau: Car idx 5 (15 m), idx 6 (33 m). Trái: Car idx 4 sát bên ~3 m | Xe xa > 35 m phía trước thấy trên cam03 nhưng chỉ còn vài điểm, chưa gán |

Chỉ camera cho biết: màu xe (xe con trắng sát bên trái trên cam05 ở cả s01f00 và s01f40), trạng thái đèn tín hiệu và có biển chỉ đường phía trên giao lộ (s01f00, cam03), loại xe cụ thể (Bus idx 8 ở s01f00 là xe buýt, xe idx 6 ở s03f07 là xe tải nhỏ — chỉ chắc nhờ ảnh). Chỉ LiDAR cho biết: khoảng cách theo mét (xe phía trước s01f40 idx 2 cách 23,8 m), kích thước thật (Bus s01f00 idx 8: 9,84 × 3,31 × 3,62 m), và vật phía sau/hai bên ngoài góc nhìn cam03 (5 xe phía sau ở s01f00, xe idx 6 phía sau 33 m ở s01f40).

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s02f06 | cạnh phải | Bus idx 9 (phía sau 29 m) đáy cao hơn mặt đường 0,41 m; hai xe buýt phía trước-trái gây vùng bóng che (ảnh `t4-1-bong-che`) |
| s02f26 | cạnh phải | Có điểm lẻ loi không thuộc vật nào gần xe (ảnh `t4-4-diem-la`) |
| s03f07 | cạnh phải | Kẹt xe, nhiều xe sát nhau; `selfcheck` báo hộp 1–9 và 2–10 chồng nhau |
| s03f32 | cạnh phải | Ít vật (5), có 1 người đi bộ xa 38,7 m chỉ 22 điểm |
| s04f00 | cạnh phải | Car idx 5 xa 43 m có kích thước 7,70 × 3,88 m, quá lớn cho Car |
| s04f29 | cạnh phải | không |
| s05f10 | cạnh phải | Car idx 4 (32 m phía trước) đáy cao hơn mặt đường 0,48 m |
| s05f34 | cạnh phải | Mặt đường phía bên trái thấp hơn; vùng xa phía trước rất thưa điểm (ảnh `t4-3-phan-xa-thap`) |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car`, `t2-b-bus-truck`, `t2-c-ped-cyclist`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | s01f40, idx 5 | Car | có (yaw 269,4°, xe phía sau cùng chiều) | có (z_bottom −2,078 m; lệch mặt đường −0,25 m) | có | không (l 5,17 m, w 2,59 m, h 1,68 m; rộng vượt 2,10 m của SUV) |
| (b) xe buýt hoặc xe tải | s03f07, idx 6 | Truck | có (yaw 274,9°, xe tải phía trước cùng chiều) | có (z_bottom −2,013 m; lệch −0,23 m) | có | gần đúng (l 5,96 m, w 1,93 m, h 2,33 m; giữa bán tải và xe tải nhẹ: cao hơn bán tải, hẹp hơn xe tải nhẹ) |
| (c) người đi bộ hoặc xe hai bánh | s03f32, idx 5 | Pedestrian | chưa chắc (yaw 84,5°; người ở xa 38,7 m, chỉ 22 điểm) | có (z_bottom −1,954 m; lệch −0,12 m) | có | không (l 0,86 m, w 1,00 m, h 1,99 m; rộng > 0,9 m, cao > 1,95 m vì vài điểm thưa nên hộp hơi rộng) |

Không tham gia vòng hiệu chuẩn trong task chung, nên không có so sánh với các bạn. Vật bỏ qua vì không đủ cơ sở: s01f40, các xe > 35 m phía trước (LOW_VISIBILITY, chỉ vài điểm); s01f00, cụm điểm thấp ngoài 50 m phía trước (UNCERTAIN_CLASS).

## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân ghi là **giả thuyết**. Cả 4 hiện tượng lấy từ frame của tôi, không mượn.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | s02f06, ảnh `t4-1-bong-che` | Giả thuyết: xe buýt idx 1 (~25,5 m phía trước-trái, cao 3,78 m) chắn tia LiDAR nên tia không tới vùng phía sau | Sau xe buýt idx 1 là một dải gần như không có điểm, trong khi hai bên vẫn có điểm; xe buýt idx 3 phía sau nó (43,4 m) chỉ còn 278 điểm so với 1281 điểm của idx 1 | Vùng trống là chưa quan sát được, không phải đường trống; vật bị che một phần thì bật occluded và cuboid bao phần suy luận được |
| 2. Thưa ở xa | s01f00, ảnh `t4-2-thua-o-xa` | Giả thuyết: các tia LiDAR toả ra xa nhau theo khoảng cách nên vật xa nhận ít tia | Xe ở ~39,5 m (idx 22) chỉ còn 59 điểm; xe ~34 m (idx 5) còn 97 điểm, so với 1707 điểm của xe ở 9,7 m | Thấy khoảng 15–20% thể tích hoặc suy luận được thì vẽ; không thì bỏ qua và ghi LOW_VISIBILITY |
| 3. Phản xạ thấp / ít điểm bất thường | s05f34, ảnh `t4-3-phan-xa-thap` | Giả thuyết: bề mặt tối màu hoặc kính hấp thụ/phản xạ lệch tia laser nên ít điểm quay về | Vùng khoanh đỏ phía trước xa chỉ có vài điểm mờ dù cam03 thấy xe trên làn; Car idx 9 (53,6 m) chỉ 24 điểm, ít hơn hẳn Car idx 5 xa hơn (56,4 m, 68 điểm) | Chỉ ghi giả thuyết, không khẳng định nguyên nhân; vẫn vẽ theo quy tắc vật thưa điểm |
| 4. Điểm lạ | s02f26, ảnh `t4-4-diem-la` | Giả thuyết: phản xạ nhiều lần (multipath) hoặc nhiễu cảm biến; nguyên nhân chưa rõ | Một vài điểm lẻ loi (khoanh đỏ) gần xe, tách hẳn khỏi các xe và mặt đường, không khớp vật nào trên camera | Không gán thành vật |

## 5. Số từ cuboid của tôi (Bước 5)
Lấy từ `submission/D13-2A202603202-bangA.csv`; số điểm lấy từ cột `points_in` của `-tukiem.csv`. Mặt đường khoảng −1,7 m: z đáy lệch hơn 0,3 m là lơ lửng hoặc chìm.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| s01f00, 1, Car | −1,051 m | 1,581 m | −1,842 m | 9,73 m | 9,79 m | 270,8° | chạm đường (lệch mặt đường cục bộ −0,07 m) |
| s05f34, 6, Car | −0,884 m | 1,941 m | −1,855 m | 23,39 m | 23,40 m | 87,1° | lơ lửng: mặt đường chỗ này thấp hơn, đáy cao hơn mặt đường 0,56 m |
| s01f40, 3, Car | −1,452 m | 2,172 m | −2,538 m | 32,21 m | 32,24 m | 268,7° | chìm: đáy thấp hơn mặt đường 0,60 m |

(a) Gần nhất: frame s01f40, idx 4, `dist_m` = 3,17 m, r = 3,41 m. Xa nhất: frame s01f00, idx 16 (Bus), `dist_m` = 60,62 m, r = 60,63 m. (b) Người đi bộ s01f00 idx 12: l = 1,13 m, w = 0,88 m, h = 1,90 m, so với bảng 4.3 (người lớn 0,4–0,9 × 0,5–0,9 × 1,50–1,95 m): rộng và cao trong khoảng, dài vượt 0,23 m. Xe con s01f00 idx 1: l = 4,93 m, w = 2,16 m, h = 1,58 m, so với bảng 4.3 (SUV 4,4–5,3 × 1,80–2,10 × 1,65–2,10 m; sedan cao 1,40–1,60 m): dài trong khoảng, rộng vượt 0,06 m vì hộp bao cả gương chiếu hậu và điểm nhiễu ở mép. (c) Xe cùng chiều phía trước: s01f40 idx 2, `yaw_deg` = 263,3° (gần 270°). Xe ngược chiều: s01f00 idx 22, `yaw_deg` = 92,9° (gần 90°).

Mật độ theo khoảng cách (4 cuboid ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| s04f00, 7, Car | ~5 m: 4,61 m | 5828 |
| s01f40, 5, Car | ~15 m: 14,84 m | 830 |
| s04f00, 10, Car | ~25 m: 24,48 m | 243 |
| s01f00, 22, Car | ~40 m: 39,46 m | 59 |

Số điểm giảm nhanh theo khoảng cách: xa gấp ~3 lần (4,6 → 14,8 m) thì số điểm còn khoảng 1/7; xa gấp ~8,6 lần (4,6 → 39,5 m) thì còn khoảng 1/100, gần với quy luật tỉ lệ nghịch bình phương khoảng cách. So với đường cong slide (1968 → 718 → 214 → 72 → 19 điểm) thì cùng xu hướng; số của tôi ở 5 m cao hơn vì xe idx 7 sát bên, bề mặt hông xe hướng thẳng vào LiDAR. Sau `selfcheck` còn 38 cờ: giữ nguyên cờ kích thước ở các Bus (rộng 3,3–3,9 m) và SUV (rộng 2,4–2,7 m) vì cụm điểm thực sự rộng như vậy, có thể do gương và nhiễu mép; các cờ chìm/lơ lửng (s01f40 idx 3, 4; s02f06 idx 1; s05f10 idx 4; s05f34 idx 6) và chồng hộp ở s03f07 (1–9, 2–10) chưa sửa kịp trước khi xuất bảng A.

Không kịp đổi kiểm với bạn bên cạnh (làm bài riêng).

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
(a) Model học trên nhãn này để dự đoán vị trí và kích thước thật của vật quanh xe; nhãn đo bằng mét giữ nguyên ý nghĩa ở mọi khoảng cách và góc nhìn, còn pixel thay đổi theo camera và độ xa nên không dùng trực tiếp để tính khoảng cách, tránh va chạm. Vật thưa điểm cần quy tắc riêng (15–20% thể tích, LOW_VISIBILITY) vì nhãn đoán mò trên vài điểm sẽ dạy model sai vị trí, kích thước, tạo nhãn nhiễu.

(b) Dùng AI: Claude (Cowork). Câu hỏi chính: lệnh tạo .venv và cài thư viện, xử lý lỗi cấu hình/cảnh báo phiên bản cvat-sdk, tóm tắt guideline, gợi ý khung câu cho báo cáo; AI giúp đối chiếu bảng A/phiếu tự kiểm với job CVAT để điền số vào báo cáo. Tôi tự vẽ cuboid và tự chụp ảnh; số liệu lấy từ bảng A và phiếu tự kiểm.

(c) Điều còn chưa chắc: hướng của hai người đi bộ đứng gần nhau ở s01f00 (idx 11, 12; yaw 84–88°); kích thước vật xa 40–60 m ở s01f00 (Bus idx 16 chỉ 77 điểm); class của s04f00 idx 5 (dài 7,70 m, rộng 3,88 m, có thể là Truck/Bus chứ không phải Car); chiều cao s03f07 idx 4 (1,08 m, thấp bất thường) và s03f32 idx 3 (1,00 m).

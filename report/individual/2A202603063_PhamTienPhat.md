# Báo cáo Day 13 — Phạm Tiến Phát, 2A202603063

## 1. Kết luận trước
Mã số 2A202603063, task `D13-2A202603063`. 10 frame: s01f08, s01f30, s02f02, s02f37, s03f10, s03f52, s04f02, s04f50, s05f24, s05f48. Máy Windows 11 Pro, trình duyệt Microsoft Edge, CVAT **v2.74.1** chạy local (bản cài từ Ngày 12; lớp dùng 2.82/2.83, `cvat-sdk` 2.83 chỉ báo cảnh báo phiên bản, tạo task và xuất bảng A vẫn chạy).

Bảng A có **93 cuboid** (69 Car, 11 Cyclist, 8 Pedestrian, 3 Truck, 2 Bus), 83 cuboid có `dist_m` ≤ 30 m. Frame ít nhất là s01f30 và s05f48 (5 cuboid mỗi frame): hai frame này ít xe gần, các cụm còn lại là cây, dải phân cách, hoặc vật quá thưa để vẽ có cơ sở. `selfcheck` cờ **1/93** cuboid (s02f02 #6, giữ nguyên, lý do ở mục 5). Tôi tin nhất phần **đáy** (khoảng hở với mặt đường 0,00–0,27 m ở mọi hộp) và **xe gần cùng chiều** (hướng đối chiếu được trên cam01/cam09). Yếu nhất là **hướng của xe ở đường bên trái bị che** và **kích thước xe chỉ thấy một mặt** (dài, rộng phải suy theo bảng 4.3).

Cách dùng CVAT:
- Đổi góc nhìn: kéo chuột trái để xoay, chuột phải để dời, con lăn để phóng; `Shift` + mũi tên để nghiêng camera về nhìn thẳng từ trên xuống hoặc nhìn ngang; nháy đúp chỗ trống để về góc mặc định.
- Vẽ và chỉnh cuboid: nút cuboid → label → **Shape**, nháy đúp vào cụm điểm để đặt; chọn hộp (bấm thẻ trong danh sách hoặc nháy đúp) rồi chỉnh tâm, dài, rộng, yaw ở **Top**, đáy và chiều cao ở **Side/Front**.
- Hướng và ảnh camera: bật **Appearance → Cuboid orientation** (vạch đỏ `+X` là đầu xe); xem cuboid trên 7 ảnh camera bằng `python scripts/lab.py preview` (ảnh trong `work/xem/`).

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ. Ảnh: `t1-s01f08-3d.png`, `t1-s01f08-top.png`, `t1-s01f30-3d.png`, `t1-s01f30-top.png`.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f08 | Phía trước (−y) ở **cạnh trái** ảnh; +y chỉ sang phải (ra sau xe), +x chỉ xuống dưới ảnh (bên trái xe) | Phẳng; đáy các xe trên làn mình ở z ≈ −1,76 đến −1,79 m, tức thấp hơn LiDAR ~1,75 m; đường bên kia dải phân cách thấp hơn thêm ~0,3 m (đáy xe ở đó ≈ −2,0 đến −2,3 m) | ~35 m (xe con phía sau, idx 9, `dist_m` 35,2 m) | Xe con, xe van chở người, một người ngồi trên xe đạp, dải phân cách có hàng rào cây, cây ven đường | Xe xanh sát bên trái (~4 m); xe con phía sau cùng làn (19,7 m); xe van và xe con trên đường bên trái, ngược chiều; người đi xe đạp trên vỉa hè bên phải (~10 m) | Một cụm dài ~11 m, thấp ~1,3 m ở bên trái phía trước (~24 m sang trái, ~17 m phía trước): có thể là một hàng xe đỗ liền nhau hoặc hàng rào cây, camera bị xe xanh che nên không phân biệt được |
| s01f30 | Phía trước (−y) ở **cạnh trái** ảnh; +y chỉ sang phải, +x chỉ xuống dưới ảnh | Phẳng; đáy xe gần ở z ≈ −1,74 đến −1,77 m (thấp hơn LiDAR ~1,75 m) | ~31 m (xe tải nhẹ phía sau bên trái, idx 5, `dist_m` 30,7 m) | Xe con, xe tải nhẹ, dải phân cách, cây; phía trước bên phải có nhóm người và xe máy đỗ | Xe trắng phía sau cùng làn (8,1 m), xe xanh phía sau (16,9 m); sedan tối và xe tải nhẹ trên đường bên trái phía sau | Cụm điểm dày ở vỉa hè phía trước bên phải (~30 m): người ngồi lẫn xe máy đỗ, không tách được từng người |

Chỉ **camera** cho biết: (1) loại xe chi tiết: xe van chở người hay van chở hàng, xe tải có thùng hở; (2) đầu hay đuôi xe nhờ đèn pha, đèn hậu, biển số trước/sau, nhất là xe đứng gần như vuông góc; (3) người đang ngồi trên xe đạp hay chỉ đứng cạnh xe dựng.
Chỉ **LiDAR** cho biết: (1) khoảng cách và kích thước theo mét (xe xanh sát bên cách ~4 m, dài thấy được ~4 m); (2) độ cao mặt đường ở từng vùng (đường bên trái thấp hơn làn mình ~0,3 m); (3) vật trong vùng camera bị che hoặc không nhìn tới: các xe ở đường bên trái bị xe xanh che trên cam05/cam06 nhưng vẫn có cụm điểm trên point cloud.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s02f02 | Cạnh trái | Xe buýt đỏ chạy sát bên trái (~3,4 m) che một góc lớn phía sau nó |
| s02f37 | Cạnh trái | Hàng rào giữa đường chia hai chiều; nhiều xe máy điện và xe đạp dựng trên làn xe đạp bên phải |
| s03f10 | Cạnh trái | Kẹt xe, các xe cách nhau 1–2 m; tường quây công trình bên trái và bên phải |
| s03f52 | Cạnh trái | Xe buýt trên đường bên trái phía trước; nhiều cột và cây ở vỉa hè phải |
| s04f02 | Cạnh trái | Ngã tư cạnh cây xăng, nhiều người đi bộ và công nhân ở cây xăng bên phải |
| s04f50 | Cạnh trái | Hàng rào vàng giữa đường cong; đường bên trái là chiều ngược lại |
| s05f24 | Cạnh trái | Ngã tư, bãi xe đạp công cộng dài bên phải; nhiều người đi xe máy điện băng qua |
| s05f48 | Cạnh trái | Xe van xanh sát bên phải (~4,8 m) che gần hết phía phải |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | s01f08, 1 | Car | Có: yaw 270,0°, xe đi cùng chiều, đuôi xe thấy trên cam06 | Có: z đáy −1,763 m, khoảng hở 0,01 m | Có | Có: 4,50 × 1,82 × 1,45 m (sedan 3,8–5,0 × 1,70–1,90 × 1,40–1,60) |
| (b) xe buýt hoặc xe tải | s01f30, 5 (frame 1 không có xe buýt hay xe tải) | Truck | Có theo chiều đường: yaw 90,0°, xe trên đường bên trái, ngược chiều | Có: z đáy −1,777 m | Có | Một phần: dài 5,75 m, rộng 2,13 m nằm trong khoảng xe tải nhẹ; cao 1,92 m thấp hơn khoảng 2,40–3,50 m vì xe thùng hở thấp và chỉ thấy một mặt |
| (c) người đi bộ hoặc xe hai bánh | s01f08, 13 | Cyclist | Có: yaw 289,0°, người ngồi trên xe đạp quay về phía trước | Đáy z −1,419 m, khoảng hở 0,20 m so với vỉa hè (dưới ngưỡng 0,3 m) | Có | Dài 1,90 m, cao 1,76 m nằm trong khoảng xe đạp có người; rộng 0,80 m hơi lớn hơn 0,50–0,70 vì hộp ôm cả túi đỏ người đó đeo |

Vòng hiệu chuẩn: **không thực hiện**. Task chung `D13-HC485087` đã tạo nhưng tôi không vẽ ba vật theo chỉ định của coach, nên không có file để so với lớp.

Vật bỏ qua vì không đủ cơ sở:
- s01f08, bên trái phía trước ~24 m: cụm dài ~11 m, thấp ~1,3 m (LOW_VISIBILITY, UNCERTAIN_CLASS: hàng xe đỗ hay hàng rào cây).
- s01f30, vỉa hè phía trước bên phải ~30 m: nhóm người ngồi lẫn xe máy đỗ (LOW_VISIBILITY, không tách được từng người).
- s02f37, s05f24: xe đạp và xe máy dựng bên đường không có người, không vẽ theo guideline.

Vật đã vẽ nhưng còn nghi vấn: s03f10 idx 6 (van có chữ quảng cáo chở hàng, gán Truck, UNCERTAIN_CLASS); s05f48 idx 1 (van xanh có cửa kính dọc thân, gán Car, có thể là minibus → Bus, UNCERTAIN_CLASS).

## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân ghi là **giả thuyết**.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | s02f02 / `t4-1-bong-che.png` / frame của tôi | Xe buýt đỏ cách ~3,4 m chắn tia LiDAR, tia không đi tới vùng phía sau xe buýt | Phía sau xe buýt (xa hơn, về bên trái) là một vùng hình quạt gần như không có điểm, trong khi hai bên vùng đó vẫn dày điểm | Vùng trống là vùng **chưa quan sát được**, không phải đường trống; vật nào thấy một phần sau vật che thì bật occluded và bao phần suy luận được, không thấy gì thì không vẽ |
| 2. Thưa ở xa | s02f37 idx 5 / `t4-2-thua-o-xa.png` / frame của tôi | Ở 33 m khoảng cách giữa các tia của LiDAR 40 tia đã lớn, chỉ vài vòng quét chạm vào xe | Xe con phía sau ở `dist_m` 33,4 m chỉ có 29 điểm trong hộp; khung Top, Side, Front gần như trống | Vẽ vì ảnh cam09 thấy rõ xe và cụm điểm đủ hình dạng đầu xe (đủ khoảng 15–20% thể tích); kích thước lấy theo bảng 4.3; nếu ít hơn thì bỏ qua và ghi LOW_VISIBILITY |
| 3. Phản xạ thấp / ít điểm bất thường | s02f02 idx 1 / `t4-3-phan-xa-thap.png` / frame của tôi | Kính cửa sổ xe buýt cho tia đi xuyên qua hoặc phản xạ đi hướng khác nên ít điểm quay về | Ở khung Side, dải cửa kính của thân xe ít điểm, nhiều điểm nằm sâu bên trong thân xe (ghế, khung bên kia) | Không thu nhỏ hộp theo chỗ thiếu điểm; hộp bao cả thân xe theo các mặt còn điểm (nóc, đầu, đuôi, phần sơn dưới) và ảnh cam05/cam06; nguyên nhân chỉ là giả thuyết |
| 4. Điểm lạ | Task chung `D13-HC485087`, frame s02f35 / `t4-4-diem-la.png` / **mượn từ task chung** | Có thể là phản xạ nhiều lần (tia dội vào bề mặt bóng rồi mới quay về), nên máy tính ra một vị trí ảo | Cụm khoảng 46 điểm ở z ≈ −13 m, thấp hơn mặt đường quanh đó khoảng 11 m, phía sau bên trái ~28 m; nhìn ngang thấy rõ một vệt điểm lơ lửng dưới mặt đất. Frame của tôi cũng có cụm nhỏ (5 điểm, z ≈ −4,2 m ở s03f10, phía trước bên trái ~9 m) nhưng quá nhỏ để chụp rõ | Không gán thành vật; không để cụm này kéo đáy hộp gần đó xuống |

## 5. Số từ cuboid của tôi (Bước 5)
Lấy từ `submission/D13-2A202603063-bangA.csv`; số điểm từ cột `points_in` của `submission/D13-2A202603063-tukiem.csv`.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| s01f08, 1, Car | −1,037 m | 1,452 m | −1,763 m | 3,97 m | 4,11 m | 270,0° | Chạm đường (khoảng hở 0,01 m) |
| s02f02, 1, Bus | −0,300 m | 3,000 m | −1,800 m | 3,38 m | 3,39 m | 270,0° | Chạm đường (khoảng hở 0,03 m) |
| s01f30, 5, Truck | −0,815 m | 1,923 m | −1,777 m | 30,73 m | 30,74 m | 90,0° | Chạm đường: lệch −0,08 m so với −1,7 m, dưới ngưỡng 0,3 m |

(a) Gần nhất: s02f02 idx 1 (Bus), `dist_m` 3,38 m, r = 3,39 m. Xa nhất: s05f48 idx 5 (Car), `dist_m` 35,66 m, r = 35,69 m (z −1,40 m nên r chỉ lớn hơn `dist_m` 0,03 m).

(b) Người đi bộ s04f02 idx 12: 0,60 × 0,62 × 1,60 m, cả ba chiều trong khoảng người lớn (0,4–0,9 × 0,5–0,9 × 1,50–1,95 m). Xe con s01f08 idx 1: 4,50 × 1,82 × 1,45 m, trong khoảng sedan. Chiều ngoài khoảng gặp ở chỗ khác: xe tải s01f30 idx 5 cao 1,92 m (< 2,40 m) vì chỉ thấy thùng thấp ở một mặt; cyclist s01f08 idx 13 rộng 0,80 m (> 0,70 m) vì hộp ôm cả túi người đó đeo.

(c) Xe cùng chiều phía trước s01f08 idx 2: yaw 269,0°, gần 270°. Xe ngược chiều s01f08 idx 4 (van trắng đường bên trái): yaw 90,0°, đúng 90°.

Mật độ theo khoảng cách (4 cuboid ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| s04f50, 1, Car | 7,35 m | 1866 |
| s04f02, 2, Car | 15,36 m | 692 |
| s04f02, 8, Car | 25,01 m | 241 |
| s04f50, 8, Car | 32,77 m | 131 |

Quy luật: xa gấp đôi thì số điểm còn khoảng 1/3 đến 1/4 (7,35 m → 15,36 m: 1866 → 692 điểm; 15,36 m → 32,77 m: 692 → 131 điểm), gần với tỉ lệ nghịch bình phương khoảng cách. Số của tôi khớp đường cong ở slide (xe con khoảng 1968 điểm ở 0–10 m, 718 ở 10–20 m, 214 ở 20–30 m, 72 ở 30–40 m); xe ở 32,77 m có 131 điểm, nhiều hơn mức trung vị 72 vì xe đó ở ngay phía sau, quay đầu thẳng về LiDAR và không bị che.

Tự kiểm: sau `preview` và `selfcheck` tôi sửa hai cuboid trên CVAT rồi chạy lại `sync`:
- s02f02 idx 6 (Pedestrian): hộp cũ đặt lệch vào hàng rào, chỉ có 5 điểm và đáy thấp hơn mặt đất 0,19 m. Tôi dời hộp sang chỗ người đứng (khoảng 14,0 m bên phải, 18,3 m phía trước), đáy ở mặt vỉa hè. Hộp mới có 22 điểm, khoảng hở 0,03 m.
- s03f10 idx 12 (Cyclist): hộp cũ ôm vào tường quây công trình, bị cờ "73 điểm sát ngay ngoài mép". Tôi dời hộp vào đúng người đi xe máy, cách tường ~1,7 m về phía đường, đáy ở mặt đường. Hộp mới có 102 điểm, hết cờ.

Cảnh báo giữ nguyên: s02f02 idx 6 "11 điểm sát ngay ngoài mép". Tôi giữ vì 11 điểm đó thuộc hàng rào ngay cạnh người đi bộ, không phải phần thân bị cắt; nới hộp ra sẽ ôm cả hàng rào.

Đổi kiểm: **chưa thực hiện**, vì không có bạn ngồi cạnh khi làm bài.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
(a) Model 3D phải xuất ra vị trí và kích thước thật của vật theo mét để xe tính khoảng cách phanh và đường đi. Kích thước theo pixel thay đổi theo khoảng cách và camera, nên nhãn theo pixel không dạy được model điều đó. Vật thưa điểm cần quy tắc riêng vì model chỉ học được từ những gì cảm biến thực sự thấy. Nếu người gán đoán hộp cho vật gần như không có điểm, model sẽ học cách "tưởng tượng" vật ở chỗ trống và báo nhầm. Nếu mỗi người lại bỏ một kiểu, model nhận nhãn không nhất quán ở vùng xa.

(b) Không dùng AI.

(c) Điều còn chưa chắc:
- Hướng (90° hay 270°) của các xe ở đường bên trái s01f08 bị xe xanh che trên camera: tôi suy theo chiều làn đường.
- Hướng của s04f02 idx 4, s04f50 idx 5, s03f10 idx 12.
- Class của hai xe van (s03f10 idx 6, s05f48 idx 1).
- Dài và rộng của các xe chỉ thấy một mặt: đã kéo theo bảng 4.3 về phía khuất, có thể lệch tâm.
- Bài nộp sau hạn 23:59 ngày 25/11/2026.

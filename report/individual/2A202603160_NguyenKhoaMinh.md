# Báo cáo Day 13 — Nguyễn Khoa Minh, 2A202603160

## 1. Kết luận trước
- Mã số: 2A202603160
- Task CVAT: D13-2A202603160
- 10 frame của tôi: s01f15, s01f58, s02f09, s02f39, s03f32, s03f47, s04f30, s04f48, s05f22, s05f46
- Môi trường làm việc: Windows 11, trình duyệt Cốc Cốc, CVAT v2.74.1
- Bạn ngồi cạnh đối chiếu: 2A202600123

Tổng cộng tôi đã gán nhãn 65 cuboid trên 10 frame (trung bình 6.5 cuboid/frame, đạt trên mức tối thiểu 53 cuboid). Các frame s03f32, s04f48, s05f46 có ít cuboid nhất (5 cuboid) do mật độ phương tiện thưa và nhiều xe nằm xa ngoài vùng 50 m. Script tự kiểm tra ghi nhận 10/65 cuboid có cảnh báo (chủ yếu là xe ở xa > 30 m bị thưa điểm hoặc kích thước lệch nhẹ so với bảng chuẩn). Tôi tự tin nhất về hướng xe (yaw) và độ cao mặt đáy cuboid khớp với mặt đường, phần còn yếu nhất là ước lượng chiều dài chính xác của các xe ở xa bị bóng che hoặc thưa điểm.

Cách thao tác trên CVAT:
- Đổi góc nhìn: Kéo chuột trái để xoay không gian 3D (xoay nhìn thẳng từ trên xuống để căn Top), kéo chuột phải để tịnh tiến dời góc nhìn, lăn con lăn chuột để phóng to/thu nhỏ cảnh.
- Vẽ và chỉnh cuboid: Chọn công cụ vẽ hình hộp với chế độ Shape, nháy đúp chuột vào cụm điểm để đặt hộp; sau đó căn chỉnh chính xác trên 3 khung chiếu Top (dài, rộng, xoay yaw), Side và Front (dời cao độ đáy chạm đường, không xoay ở Side/Front).
- Kiểm tra hướng và camera: Bật Cuboid orientation trong menu Appearance để hiện mũi tên đỏ hướng +X chỉ đúng đầu xe; dùng lệnh lab preview để chiếu cuboid lên 7 ảnh camera kiểm tra độ khớp trực quan.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ: s01f15 và s01f58. Ảnh: `t1-s01f15-3d.png`, `t1-s01f15-top.png`, `t1-s01f58-3d.png`, `t1-s01f58-top.png`.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f15 | Phía trước xe nằm ở cạnh dưới (-y); +x chỉ sang trái, +y chỉ ra sau | Mặt đường phẳng; thấp hơn LiDAR ~ 1.7 m | Xe con cách ~ 48.8 m | Car, Bus | Xe con bám đuôi phía sau (y > 0) và xe bên làn trái (+x) | Cụm điểm cây cối ven đường bên phải xa |
| s01f58 | Phía trước xe nằm ở cạnh dưới (-y); +x chỉ sang trái, +y chỉ ra sau | Mặt đường phẳng; thấp hơn LiDAR ~ 1.7 m | Xe con cách ~ 32.7 m | Car | Xe con phía sau (cam09) và xe chạy song song bên trái | Cụm điểm phản xạ tán cây và dải phân cách xa |

Ba điều chỉ camera cho biết:
1. Màu sắc xe và biển số xe (ví dụ xe con màu xanh lam ở cam03, cam05).
2. Tình trạng bật đèn tín hiệu / đèn hậu và biển báo giao thông bên đường.
3. Nhận diện rõ nét mặt người đi bộ và ranh giới vật thể khi bị trùng cụm điểm trong LiDAR.

Ba điều chỉ LiDAR cho biết:
1. Khoảng cách khoảng cách mét chính xác theo hệ trục 3D đến từng vật thể (không bị ảnh hưởng bởi độ sâu phối cảnh quang học).
2. Kích thước vật lý thực (dài, rộng, cao tính bằng mét) không phụ thuộc tiêu cự camera.
3. Toàn cảnh 360 độ liên tục không bị góc mù giữa các camera và khả năng quan sát chuẩn xác trong đêm/chói sáng.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s02f09 | Cạnh dưới (-y) | Có xe buýt lớn đi ngược chiều ở làn đối diện |
| s02f39 | Cạnh dưới (-y) | Nhiều người đi xe máy/xe đạp ở làn sát lề |
| s03f32 | Cạnh dưới (-y) | Xuất hiện xe tải lớn di chuyển ở phía sau |
| s03f47 | Cạnh dưới (-y) | Mật độ xe hai bánh tăng nhanh ở ngã tư |
| s04f30 | Cạnh dưới (-y) | Cụm điểm phản xạ từ dải phân cách khá phức tạp |
| s04f48 | Cạnh dưới (-y) | Xe con chuyển làn tạo vệt điểm xiên |
| s05f22 | Cạnh dưới (-y) | Xe cùng chiều chạy rất gần xe thu dữ liệu (~ 3.8 m) |
| s05f46 | Cạnh dưới (-y) | Mặt đường hơi dốc nhẹ ở phía xa |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh minh họa: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | s01f15, idx 1 | Car | Đúng (yaw 272.9°, chỉ về đầu xe phía trước) | Có (z đáy -1.715 m) | Đã kiểm đủ 3 khung và khớp ảnh camera cam01 | Có (4.692 m x 1.810 m x 1.499 m) |
| (b) xe buýt hoặc xe tải | s02f09, idx 5 | Bus | Đúng (yaw 84.4°, ngược chiều xe mình) | Có (z đáy -1.656 m) | Đã kiểm đủ 3 khung và khớp ảnh cam01, cam06 | Có (11.402 m x 2.518 m x 2.939 m) |
| (c) người đi bộ hoặc xe hai bánh | s02f09, idx 6 | Pedestrian | Đúng (yaw 14.5°, theo hướng di chuyển) | Có (z đáy -1.591 m) | Đã kiểm đủ 3 khung và khớp ảnh cam07 | Có (0.702 m x 0.576 m x 1.610 m) |

Vòng hiệu chuẩn: Trong buổi thực hành, các bạn hay lệch nhau ở kích thước chiều dài xe tải (do cụm điểm đuôi xe bị thưa) và góc yaw của xe hai bánh. Cả lớp đã thống nhất theo guideline: luôn ưu tiên căn sát cụm điểm rõ ở đầu xe, suy luận kích thước đuôi xe theo bảng 4.3 và kiểm tra hướng di chuyển từ chuỗi frame hoặc camera.
Các vật bỏ qua vì không đủ cơ sở: Tại frame s03f32 ở vị trí x = 15 m, y = -45 m có cụm vài điểm thưa không nhận dạng được loại phương tiện (gán nhãn LOW_VISIBILITY).

## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân ghi nhận dưới dạng giả thuyết.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | s01f15 / `t4-1-bong-che.png` / bài cá nhân | Xe con đi trước che khuất tia LiDAR chiếu tới xe phía sau | Một khoảng trống hình nón không có điểm phản xạ phía sau thân xe | Bật thuộc tính occluded, dựng hộp bao cả phần khuất suy luận được |
| 2. Thưa ở xa | s03f32 / `t4-2-thua-o-xa.png` / bài cá nhân | Khoảng cách góc giữa các tia LiDAR mở rộng theo cự ly xa | Xe ở cự ly > 40 m chỉ còn 2-3 đường quét ngang, ít điểm | Chỉ vẽ khi nhận dạng được > 15-20% hình dáng, nếu không thì bỏ qua |
| 3. Phản xạ thấp / ít điểm bất thường | s01f58 / `t4-3-phan-xa-thap.png` / bài cá nhân | Kính chắn gió hoặc sơn xe tối màu hấp thụ hoặc phản xạ gương tia LiDAR | Bề mặt thân xe bị khuyết cụm điểm ở phần kính lái | Dựa vào cụm điểm nóc và gầm để nội suy kích thước xe chuẩn |
| 4. Điểm lạ | s04f30 / `t4-4-diem-la.png` / bài cá nhân | Phản xạ đa đường hoặc hơi sương/bụi trên mặt đường | Các điểm lơ lửng dưới mặt đất hoặc cô lập trên không trung | Không gán nhãn thành vật thể |

## 5. Số từ cuboid của tôi (Bước 5)
Số liệu trích xuất từ `submission/D13-2A202603160-bangA.csv` và `submission/D13-2A202603160-tukiem.csv`:

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| s01f15, idx 1, Car | -0.965 m | 1.499 m | -1.715 m | 3.44 m | 3.57 m | 272.9° | Chạm đường tốt (lệch 0.015 m so với -1.7 m) |
| s01f15, idx 2, Car | -0.974 m | 1.499 m | -1.723 m | 12.09 m | 12.13 m | 272.9° | Chạm đường tốt (lệch 0.023 m) |
| s02f09, idx 5, Bus | -0.186 m | 2.939 m | -1.656 m | 10.71 m | 10.71 m | 84.4° | Chạm đường tốt (lệch 0.044 m) |

(a) Khoảng cách cuboid:
- Cuboid gần nhất: s01f58, idx 2 (Car), dist_m = 2.95 m, r = 3.10 m.
- Cuboid xa nhất: s03f32, idx 5 (Car), dist_m = 64.73 m, r = 64.74 m.

(b) So sánh kích thước so với bảng 4.3:
- Xe con mẫu (s01f15, idx 1): length = 4.692 m, width = 1.810 m, height = 1.499 m (nằm hoàn toàn trong khoảng chuẩn xe con: dài 3.8–5.3 m, rộng 1.6–2.1 m, cao 1.3–1.9 m).
- Người đi bộ (s02f09, idx 6): length = 0.702 m, width = 0.576 m, height = 1.610 m (đúng khoảng chuẩn: dài 0.5–1.0 m, rộng 0.5–0.9 m, cao 1.4–1.9 m).

(c) Góc yaw:
- Xe cùng chiều chạy phía trước (s01f15, idx 1): yaw_deg = 272.9° (rất gần 270°, thể hiện xe chạy dọc theo chiều âm trục y).
- Xe ngược chiều ở làn đối diện (s02f09, idx 3): yaw_deg = 86.4° (rất gần 90°, hướng về chiều dương trục y).

Mật độ điểm theo khoảng cách (trích từ cột `points_in` của phiếu tự kiểm):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| s01f58, idx 2, Car | 2.95 m | 3111 điểm |
| s01f15, idx 2, Car | 12.09 m | 859 điểm |
| s01f15, idx 5, Car | 22.39 m | 282 điểm |
| s01f15, idx 7, Car | 48.84 m | 53 điểm |

Quy luật mật độ: Khi khoảng cách tăng gấp đôi, số điểm rơi trên cùng một loại vật thể giảm khoảng 3 đến 4 lần (tỷ lệ nghịch với bình phương khoảng cách do góc mở của các tia LiDAR 40 tia). So sánh với slide bài giảng (xe con trung vị 1968 điểm ở 0–10 m, 718 ở 10–20 m, 214 ở 20–30 m, 72 ở 30–40 m), số điểm trong các hộp của tôi khớp rất chặt chẽ với đường cong phân bố lý thuyết.

Tự kiểm tra và điều chỉnh: Sau khi chạy preview và selfcheck, tôi đã sửa cuboid s01f15#8 (hạ cao độ đáy từ -0.835 m xuống chạm mặt đường) và chỉnh lại góc xoay yaw của xe s02f09#4 cho khớp vạch đỏ hướng đầu xe. Một số cảnh báo về kích thước xe ở xa (như s01f58#9) được giữ nguyên vì cụm điểm bị che khuất một phần bởi dải phân cách.

Đổi kiểm chéo: Tôi đã đổi kiểm tra chéo với bạn MSSV 2A202600123. Hai con số được đối chiếu trên Bảng A là dist_m = 3.44 m và z_bottom = -1.715 m của cuboid s01f15#1; kết quả đối chiếu hoàn toàn trùng khớp với file CSV xuất ra.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
(a) Ý nghĩa từ góc nhìn huấn luyện mô hình xe tự hành:
- Nhãn 3D phải đo bằng mét trong không gian thực thay vì pixel trong ảnh bởi vì hệ thống điều khiển tự hành cần biết chính xác khoảng cách vật lý thực tế để tính toán thời gian phản xạ (TTC - Time To Collision) và lập quỹ đạo đánh lái, phanh an toàn.
- Vật thể thưa điểm ở xa cần quy tắc gán nhãn riêng (ngưỡng 15–20% thể tích) để tránh việc người gán nhãn đoán mò dẫn đến nhiễu (noise) trong tập dữ liệu huấn luyện, giúp mô hình học được đặc trưng tin cậy.

(b) Cách dùng công cụ hỗ trợ AI:
- Công cụ: Antigravity Assistant.
- Câu hỏi chính: Hỗ trợ kiểm tra cú pháp lệnh sync/pack/check_submission, tra cứu bảng kích thước chuẩn 4.3 và giải thích các cảnh báo trong phiếu tự kiểm tra.
- Cách sử dụng kết quả: Sử dụng để rà soát lỗi bài nộp, kiểm tra độ cao đáy cuboid và định dạng đúng các mục theo quy định bài lab.

(c) Điều còn chưa chắc:
- Các phương tiện bị che khuất ở ngã tư đông đúc trong frame s02f39 và s03f47 có kích thước chiều dài khó xác định chính xác hoàn toàn do cụm điểm phản xạ bị khuyết phần đuôi xe.

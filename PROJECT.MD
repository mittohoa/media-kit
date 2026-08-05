# Media KIT — Trợ lý dọn dẹp & nâng chất lượng thư viện ảnh

## 1. Bối cảnh & Vấn đề

Thư viện ảnh trên thiết bị di động phình to nhanh nhưng gần như không được dọn dẹp,
vì việc rà soát thủ công hàng nghìn ảnh quá tốn công. Bốn vấn đề chính:

1. **Đầy bộ nhớ** — ảnh chiếm phần lớn dung lượng, người dùng không biết bắt đầu xoá từ đâu.
2. **Ảnh trùng hoặc gần giống** — chụp liên tiếp (burst), ảnh gửi qua chat bị lưu lại nhiều lần.
3. **Ảnh mờ** — rung tay, lấy nét sai, chụp thiếu sáng.
4. **Ảnh bố cục kém** — chụp vội, chủ thể lệch/bị cắt, đường chân trời nghiêng, dính UI của app khác.

## 2. Mục tiêu

Sản phẩm hướng tới **hai mục tiêu cuối cùng**, có giá trị ngang nhau:

> **A. Giải phóng bộ nhớ** — chỉ ra chính xác nên xoá gì, và xoá một cách an toàn.
>
> **B. Gợi ý chỉnh sửa ảnh đẹp hơn, chuẩn bố cục hơn** — với những ảnh đáng giữ,
> chỉ ra ảnh đang sai ở đâu và đề xuất crop/xoay cụ thể.

Hai mục tiêu dùng chung một lần quét và một chỉ mục, nhưng **khác hẳn nhau về tương tác**:
A là thao tác hàng loạt, lướt nhanh, quyết định theo nhóm. B là thao tác trên **từng ảnh một**,
cần xem kỹ và so sánh trước/sau. Vì vậy chúng phải là **hai luồng riêng biệt trong giao diện**,
không trộn vào cùng một danh sách.

**Tiêu chí thành công (KPI):**

*Mục tiêu A — giải phóng bộ nhớ*

| Chỉ tiêu | Mục tiêu |
|---|---|
| Dung lượng giải phóng được trong lần dọn đầu tiên | ≥ 20% tổng dung lượng ảnh |
| Tỉ lệ đề xuất xoá bị người dùng từ chối | < 10% |
| Tỉ lệ xoá nhầm ảnh quan trọng (không khôi phục được) | 0% — luôn có thùng rác |

*Mục tiêu B — gợi ý chỉnh sửa*

| Chỉ tiêu | Mục tiêu |
|---|---|
| Tỉ lệ gợi ý chỉnh sửa được người dùng chấp nhận | ≥ 40% |
| Tỉ lệ ảnh đã chỉnh theo gợi ý bị hoàn tác (undo) | < 15% |
| Mỗi gợi ý đều nêu được **lý do** người dùng hiểu được | 100% |
| Thời gian phân tích bố cục cho 1 ảnh (khi mở ảnh) | ≤ 500 ms |

*Chung — hiệu năng*

| Chỉ tiêu | Mục tiêu |
|---|---|
| Thời gian quét lần đầu (thư viện 5.000 ảnh) | ≤ 3 phút |
| Thời gian quét lại (chỉ ảnh mới) | ≤ 10 giây |
| Tiêu thụ pin cho một lần quét đầy đủ | < 3% |

## 3. Đối tượng người dùng

- **Người dùng phổ thông**: điện thoại báo đầy bộ nhớ, muốn dọn nhanh mà không phải suy nghĩ.
- **Người chụp ảnh nghiệp dư**: muốn lọc ra ảnh tốt nhất trong loạt ảnh và biết cách crop/xoay đẹp hơn.

## 4. Phạm vi

### 4.1. MVP (Phase 1) — hoàn thiện mục tiêu A

- Quét thư viện ảnh, xây dựng chỉ mục và cache kết quả.
- **Đọc metadata (EXIF/XMP/IPTC)** và xếp mỗi ảnh vào một trong ba mức: đầy đủ / một phần / bị strip.
- **Cảnh báo ảnh thiếu thông tin gốc** do tải về hoặc chia sẻ qua app chat, kèm đối chiếu với
  bản gốc trong thư viện nếu có.
- Phát hiện **ảnh trùng tuyệt đối** và **ảnh gần giống**, gom nhóm, tự chọn sẵn ảnh tốt nhất.
- Phát hiện **ảnh mờ**.
- Phân loại nhanh: **ảnh chụp màn hình**, **ảnh tải về / chuyển tiếp qua chat**, **ảnh tài liệu/hoá đơn**.
- Màn hình dọn dẹp: xem trước theo nhóm, chọn/bỏ chọn, xoá vào thùng rác, hiển thị dung lượng tiết kiệm được.
- Thùng rác có thời hạn khôi phục 30 ngày.

> Ưu tiên nhóm "ảnh chụp màn hình + ảnh tải về" trước: dễ nhận diện chính xác nhất
> và thường là nguồn giải phóng dung lượng lớn nhất.

### 4.2. Phase 2 — hoàn thiện mục tiêu B

- **Phân tích chất lượng ảnh**: điểm chất lượng tổng hợp, cảnh báo ảnh độ phân giải thấp,
  ảnh nén lại nhiều lần, ảnh cháy sáng/quá tối, ảnh nhiễu.
- **Phân tích bố cục** (chi tiết ở §6.5): tách chủ thể, phát hiện đường chân trời nghiêng,
  chủ thể lệch khỏi điểm mạnh, chủ thể bị cắt cụt, headroom/looking room sai.
- **Gợi ý chỉnh sửa**: đề xuất góc xoay và khung crop cụ thể, kèm preview trước/sau và
  **lý do bằng ngôn ngữ thường** ("đường chân trời nghiêng 2.3° — xoay lại cho thẳng").
- **Áp dụng chỉnh sửa**: xoay + crop, ghi ra **file mới**, giữ nguyên ảnh gốc, có undo.

> Phân tích bố cục **chạy theo yêu cầu khi người dùng mở ảnh**, không nằm trong lần quét hàng loạt.
> Đây là điều kiện để giữ được KPI quét 3 phút ở mục tiêu A — xem §7.

### 4.3. Phase 3 — mở rộng

- **Phân loại ảnh theo chủ đề** (người, đồ ăn, phong cảnh, thú cưng, sự kiện…) và gợi ý theo nhóm.
- Gom nhóm theo sự kiện (thời gian + vị trí), gợi ý album.
- Nhắc dọn dẹp định kỳ, thống kê xu hướng dung lượng.

### 4.4. Non-goals

- **Không tự động xoá** bất kỳ ảnh nào — app chỉ đề xuất, người dùng quyết định.
- **Không upload ảnh lên server** — toàn bộ phân tích chạy trên thiết bị. App có thể **đọc**
  từ cloud của người dùng (§6.6) nhưng không bao giờ **gửi** ảnh đi đâu.
- **Không tự đồng bộ hay sao lưu** — app không thay thế Google Photos/OneDrive, chỉ đọc để đối chiếu.
- Không làm trình chỉnh sửa ảnh đầy đủ (filter, retouch, layer); chỉ áp dụng **crop và xoay** theo gợi ý.
- **Không ghi đè ảnh gốc** khi áp dụng chỉnh sửa — luôn tạo file mới.
- Không cố chấm bố cục cho ảnh chụp màn hình, ảnh tài liệu, meme — các loại này bị loại khỏi mục tiêu B.
- Không xử lý video ở giai đoạn đầu.
- Không hỗ trợ iCloud Photos (không có API công khai) và không quét toàn bộ Google Photos
  (API đã bị giới hạn — xem §6.6).

## 5. User stories

- Là người dùng, tôi muốn **thấy ngay app có thể giải phóng bao nhiêu GB** trước khi phải thao tác gì.
- Là người dùng, tôi muốn ảnh trùng được **gom thành nhóm và tự chọn sẵn ảnh đẹp nhất**, tôi chỉ cần xác nhận.
- Là người dùng, tôi muốn **xem ảnh cỡ lớn trước khi xoá**, vì ảnh nhìn nhỏ rất dễ đánh giá sai.
- Là người dùng, tôi muốn **khôi phục được ảnh đã xoá nhầm**.
- Là người dùng, tôi muốn **đánh dấu album/ảnh cần bảo vệ** để app không bao giờ đề xuất xoá.
- Là người chụp ảnh, tôi muốn biết **vì sao một ảnh bị chấm điểm thấp** và **nên sửa thế nào**.
- Là người dùng, tôi muốn biết ảnh nào **chỉ là bản chia sẻ chất lượng thấp** đã mất ngày chụp và
  vị trí, để xoá đi hoặc đi xin lại bản gốc.
- Là người dùng, tôi muốn được **cảnh báo khi ảnh sắp gửi có chứa vị trí GPS**, và xoá được nó trước khi gửi.

## 6. Hướng giải pháp kỹ thuật

### 6.1. Đọc & phân tích metadata

Metadata là **nền tảng cho tất cả các phần còn lại**: gom nhóm burst cần `DateTimeOriginal` + GPS,
chấm chất lượng cần thông số chụp, phân loại cần biết ảnh đến từ camera hay từ nơi khác.
Đây cũng là bước rẻ nhất trong toàn bộ pipeline nên luôn chạy đầu tiên.

**Các trường cần đọc:**

| Nhóm | Trường | Dùng để |
|---|---|---|
| Thời gian | `DateTimeOriginal`, `CreateDate`, `OffsetTime` | Gom nhóm burst, gom sự kiện, sắp xếp |
| Thiết bị | `Make`, `Model`, `LensModel`, `Software` | Xác định ảnh gốc hay ảnh đã qua xử lý/chia sẻ |
| Thông số chụp | `ISO`, `ExposureTime`, `FNumber`, `FocalLength`, `Flash` | Diễn giải nguyên nhân ảnh mờ/nhiễu |
| Vị trí | `GPSLatitude`, `GPSLongitude`, `GPSAltitude` | Gom nhóm theo địa điểm, xác minh trùng lặp |
| Hiển thị | `Orientation`, ICC profile, không gian màu | Xoay đúng chiều, đánh giá màu chính xác |
| Mở rộng | XMP, IPTC, MakerNotes (HDR, chân dung, burst id) | Nhận diện ảnh burst/HDR do máy tạo ra |

#### Phát hiện ảnh thiếu metadata do tải về hoặc chia sẻ

Ảnh đi qua ứng dụng nhắn tin và mạng xã hội (Zalo, Messenger, Facebook, WhatsApp) gần như luôn
**bị xoá sạch EXIF và nén lại**. Kết quả là người dùng giữ một bản sao chất lượng thấp, mất ngày chụp
và vị trí, trong khi bản gốc có thể vẫn còn ở thiết bị khác. App cần nhận ra và cảnh báo việc này.

Tín hiệu nhận diện (kết hợp nhiều tín hiệu, không dựa vào một tín hiệu đơn lẻ):

- **Thiếu `DateTimeOriginal`** trong khi file vẫn có ngày tạo → không phải ảnh gốc từ camera.
- **Thiếu `Make`/`Model`** → ảnh không do camera của thiết bị này chụp.
- **Thiếu ICC profile / thiếu thumbnail EXIF** → dấu hiệu đã bị strip.
- **Kích thước bị chuẩn hoá**: cạnh dài đúng bằng 1280 / 1600 / 2048 px — mức resize mặc định của các app chat.
- **Bytes-per-pixel thấp bất thường** → đã bị nén lại nhiều lần.
- **Mẫu tên file**: `IMG-YYYYMMDD-WA####`, `FB_IMG_*`, `received_*`, `Screenshot_*`.
- **Đường dẫn thư mục**: `/Download`, `/Pictures/Messenger`, `/Pictures/Zalo`, `/DCIM/Screenshots`.

Phân thành ba mức để hiển thị:

| Mức | Nghĩa | Hành xử của app |
|---|---|---|
| **Đầy đủ** | Có thời gian + thiết bị + (thường có) GPS | Ảnh gốc, độ tin cậy cao cho mọi phân tích |
| **Một phần** | Có thời gian nhưng thiếu GPS/thông số chụp | Vẫn dùng được, nhưng hạ độ tin cậy khi gom nhóm |
| **Bị strip** | Không còn EXIF camera | Cảnh báo "ảnh chia sẻ / tải về", ứng viên dọn dẹp ưu tiên |

#### Hành động gợi ý cho người dùng

- **Cảnh báo rõ hệ quả**: "Ảnh này là bản chia sẻ, đã mất ngày chụp và vị trí gốc, dung lượng chỉ bằng
  ~15% ảnh gốc." — nói bằng ngôn ngữ người dùng hiểu, không nói "thiếu EXIF".
- **Đối chiếu với bản gốc**: nếu tìm được ảnh trùng nội dung có metadata đầy đủ và độ phân giải cao hơn
  trong cùng thư viện → đề xuất giữ bản gốc, xoá bản chia sẻ. Đây là trường hợp dọn dẹp **an toàn và rõ ràng nhất**.
- **Khôi phục thời gian** từ tên file (`IMG-20240115-WA0007` → 15/01/2024) khi EXIF không còn,
  để ảnh không bị dồn sai vị trí trên dòng thời gian.
- **Nhắc lấy bản gốc**: nếu ảnh quan trọng nhưng chất lượng thấp, gợi ý xin lại bằng cách gửi
  dưới dạng file gốc (AirDrop / Quick Share / "gửi tệp" thay vì "gửi ảnh").
- **Riêng tư ngược lại**: cảnh báo khi ảnh sắp chia sẻ **có chứa GPS**, kèm tuỳ chọn xoá vị trí trước khi gửi.

### 6.2. Trùng lặp & trùng nội dung

Trùng lặp không chỉ là trùng file mà còn là **trùng nội dung** (cùng cảnh nhưng khác góc chụp,
đã crop, đã chỉnh màu, đã gửi qua chat). Vì vậy dùng pipeline 4 mức, rẻ trước — đắt sau,
mỗi mức chỉ chạy trên các cặp ứng viên mà mức trước lọc ra:

| Mức | Loại trùng phát hiện được | Kỹ thuật |
|---|---|---|
| 1 | Trùng byte tuyệt đối | Hash nội dung file (xxHash/MD5) |
| 2 | Trùng pixel: nén lại, resize, crop nhẹ | Perceptual hash 64-bit (dHash/pHash) trên thumbnail, Hamming ≤ 8–10, index bằng BK-tree |
| 3 | **Trùng nội dung**: khác góc, khác màu, đã chỉnh sửa | Embedding ảnh (CNN/CLIP) + cosine similarity ≥ ~0.90 |
| 4 | Xác nhận là **cùng một cảnh vật lý** | Keypoint matching ORB/AKAZE + RANSAC homography |

- **Nhóm burst**: `DateTimeOriginal` chênh < 3 giây và toạ độ GPS gần nhau.
- **Chọn ảnh đại diện**: ưu tiên điểm sharpness cao nhất, sau đó đến độ phân giải và
  ảnh có khuôn mặt mở mắt.
- **So khớp vector**: với quy mô ~10.000 ảnh, brute-force cosine trên `Float32List` là đủ nhanh
  (vài chục ms), chưa cần vector index. Khi vượt ~50.000 ảnh mới cân nhắc HNSW hoặc
  binarize embedding thành bit để so bằng Hamming.

#### Phân biệt "trùng" và "tương tự" — ràng buộc bắt buộc

Embedding sẽ chấm **hai ảnh khác nhau của cùng một địa điểm hoặc cùng một người** là rất giống nhau,
nhưng đó **không phải ảnh trùng**. Gộp chung sẽ dẫn đến xoá nhầm. Do đó tách thành hai khái niệm
riêng biệt trong cả dữ liệu lẫn giao diện:

- **Ảnh trùng** — đạt mức 1–2, hoặc đạt mức 3 *và* cùng thời điểm/vị trí, *và* qua được xác minh mức 4.
- **Ảnh tương tự** — chỉ đạt mức 3. → chỉ gom nhóm cho người dùng tự xem.

#### Tự tick chọn xoá: theo bằng chứng của từng nhóm, không theo loại nhóm

Kiểm thử trên thiết bị thật (Pixel 6 Pro, 254 ảnh) cho thấy ngưỡng Hamming ≤ 4 **không đủ**
để kết luận hai ảnh là một: hai bức chụp cùng một sân khấu chỉ khác nhau ở một người đứng
phía sau, và ba ảnh chụp màn hình cùng giao diện chỉ khác một con số, đều lọt xuống dưới ngưỡng.
Với bản dựng đầu, chúng bị **tự tick chọn xoá** — đúng kiểu lỗi mà §9 xếp là rủi ro nặng nhất.

Vì vậy quyền tự tick gắn với **bằng chứng của từng nhóm**, không gắn với category:

| Nhóm | Tự tick | Vì sao |
|---|---|---|
| Cùng MD5 | ✅ | Giống nhau từng byte, ảnh giữ lại vẫn còn |
| Bản chia sẻ kèm bản gốc | ✅ | Bản gốc metadata đầy đủ, độ phân giải cao hơn vẫn được giữ |
| Đã có bản sao lưu trên cloud | ✅ | Xoá bản máy vẫn còn bản cloud |
| Chỉ giống theo perceptual hash | ❌ | Hash chỉ nói "trông giống", không nói "là một" |
| Ảnh chụp màn hình, ảnh mờ | ❌ | Là phán đoán về giá trị ảnh, không phải về tính trùng lặp |
| Ảnh tương tự | ❌ | Gồm cả ảnh chụp liên tiếp |

Hệ quả đo được: trên cùng bộ dữ liệu, số ảnh tick sẵn giảm từ **13 ảnh (18.0 MB)** xuống
**1 ảnh (139 KB)**. Người dùng vẫn dùng được "Chọn tất cả" khi muốn.

#### Xem trước trước khi tick

Người dùng phải **nhìn được ảnh cỡ lớn** trước khi quyết định, và quyết định được ngay trong
lúc đang xem. Hai quy tắc rút ra từ thử nghiệm thật:

- Nút xem trước phải **luôn hiện trên mỗi ô ảnh**. Long-press là thao tác ẩn, không ai đoán ra.
- Ô ảnh đã chọn đánh dấu bằng **viền**, không phủ màu lên ảnh — người dùng đang phải so sánh
  các ảnh gần giống nhau, một lớp màu phủ lên phá hỏng đúng việc đó.

### 6.3. Ảnh mờ

- Điểm sharpness = phương sai của Laplacian (hoặc Tenengrad), chuẩn hoá theo độ phân giải.
- **Tránh chấm nhầm ảnh xoá phông**: đo sharpness trong **vùng chủ thể**
  (face detection hoặc saliency) thay vì trên toàn ảnh — ảnh bokeh có nền mờ nhưng chủ thể nét.
- Ngưỡng cảnh báo nên hiệu chỉnh theo phân phối của chính thư viện người dùng, không cứng nhắc.

### 6.4. Chất lượng ảnh

Điểm chất lượng tổng hợp từ các thành phần:

- Độ phân giải và tỉ lệ **bytes-per-pixel** (phát hiện ảnh đã bị nén lại nhiều lần qua chat).
- Histogram phơi sáng: tỉ lệ pixel cháy sáng / tối đen.
- Mức nhiễu ước lượng, độ tương phản, độ bão hoà màu.
- Nguồn ảnh từ EXIF: ảnh chụp bằng camera gốc hay ảnh tải về/screenshot.

### 6.5. Bố cục & gợi ý chỉnh sửa (mục tiêu B)

Đây là phần khó nhất của dự án và cần đầu tư nghiên cứu nhiều nhất.
Không thể chấm bố cục trực tiếp từ pixel — phải qua **ba tầng**:

```
Tầng 1: TRÍCH XUẤT      → chủ thể ở đâu, đường nét ở đâu, người đứng thế nào
Tầng 2: ĐỐI CHIẾU QUY TẮC → so với các quy tắc bố cục, tìm ra ảnh đang sai gì
Tầng 3: SINH GỢI Ý       → tìm khung crop + góc xoay khắc phục được lỗi đó
```

#### Tầng 1 — Trích xuất đặc trưng

| Đặc trưng | Công cụ | Phục vụ quy tắc nào |
|---|---|---|
| **Saliency map** (vùng thu hút mắt) | U²-Net bản nhẹ (`u2netp`, ~4.7 MB) qua TFLite/ONNX | Điểm mạnh, cân bằng, khoảng trống |
| **Mặt nạ chủ thể** | ML Kit Subject Segmentation, hoặc saliency nhị phân hoá | Vị trí & kích thước chủ thể |
| **Khuôn mặt + hướng nhìn** | `google_mlkit_face_detection` (có góc Euler đầu) | Headroom, looking room |
| **Khung xương người** | ML Kit Pose Detection / MoveNet Lightning (~3 MB) | Cắt phạm khớp, tư thế |
| **Đoạn thẳng** | OpenCV `HoughLinesP` hoặc LSD | Chân trời nghiêng, đường dẫn hướng, hội tụ dọc |
| **Vật thể chính** | ML Kit Object Detection | Chủ thể khi ảnh không có người |

Với ảnh không có chủ thể rõ (phong cảnh, hoa văn), saliency map là đầu vào duy nhất đáng tin —
mọi quy tắc dựa vào "chủ thể" phải tự động tắt thay vì đoán bừa.

#### Tầng 2 — Danh mục quy tắc bố cục & khả năng phát hiện

Đánh giá thật về việc quy tắc nào máy kiểm được, quy tắc nào không:

| Quy tắc | Cách phát hiện | Độ tin cậy | Đưa vào phase |
|---|---|---|---|
| **Đường chân trời nghiêng** | Hough → gom các đoạn gần ngang, lấy góc trung vị | ★★★ Rất cao | 2 |
| **Chủ thể bị cắt cụt ở mép** | Bbox/mask chạm biên ảnh | ★★★ Rất cao | 2 |
| **Cắt phạm khớp** (cổ tay, đầu gối, mắt cá) | Pose → khớp nằm sát đường cắt | ★★★ Rất cao | 2 |
| **Quy tắc 1/3** | Khoảng cách trọng tâm saliency tới 4 giao điểm | ★★☆ Cao | 2 |
| **Headroom** | Đỉnh đầu (pose/face) tới mép trên | ★★☆ Cao | 2 |
| **Looking room / lead room** | Góc Euler Y của mặt → khoảng trống phía trước mặt | ★★☆ Cao | 2 |
| **Lấp đầy khung hình** | Tỉ lệ diện tích chủ thể / toàn ảnh | ★★☆ Cao | 2 |
| **Đường dọc hội tụ** (méo phối cảnh) | Độ hội tụ của các đoạn gần dọc | ★★☆ Cao | 2 |
| **Đối xứng** | Lật ảnh theo trục, so khớp hai nửa | ★★☆ Cao | 2 |
| **Chân trời đặt giữa ảnh** | Vị trí đường chân trời so với 1/3 trên–dưới | ★★☆ Cao | 2 |
| **Khoảng trống (negative space)** | Tỉ lệ và phân bố vùng không saliency | ★☆☆ Trung bình | 3 |
| **Cân bằng thị giác** | Mô-men khối lượng saliency trái/phải | ★☆☆ Trung bình | 3 |
| **Đường dẫn hướng** | Đoạn thẳng hướng về phía chủ thể | ★☆☆ Trung bình | 3 |
| **Tỉ lệ vàng / xoắn ốc Fibonacci** | Như 1/3 nhưng dùng lưới phi (0.382/0.618) | ★☆☆ Trung bình | 3 |
| **Vật thể mọc ra từ đầu** (merger) | Đoạn thẳng/vật thể cắt ngay vùng đầu | ★☆☆ Trung bình | 3 |
| **Khung trong khung** (framing) | Rất khó — cần hiểu ngữ cảnh | ✗ Không đáng tin | — |
| **Lớp lang chiều sâu** | Cần ước lượng độ sâu, khó trên ảnh thường | ✗ Không đáng tin | — |
| **Quy tắc số lẻ, "kể chuyện"** | Mang tính chủ quan/nội dung | ✗ Không làm | — |

**Nguyên tắc quan trọng:** chỉ hiện gợi ý cho các quy tắc ★★★ và ★★☆. Nhóm ★☆☆ để sau và
mặc định tắt. Nhóm ✗ ghi ra đây để **chốt là không làm**, tránh sa đà.

Ngoài ra cần một danh sách **ngoại lệ** để không gợi ý bậy:

- Ảnh chân dung cận cảnh → bỏ qua quy tắc 1/3 (đặt giữa là đúng).
- Ảnh đối xứng có chủ đích (kiến trúc, phản chiếu mặt nước) → bỏ qua quy tắc 1/3.
- Ảnh chụp macro/không có đường chân trời → tắt kiểm tra nghiêng.
- Ảnh chụp màn hình, ảnh tài liệu, meme → **tắt toàn bộ phân tích bố cục**.

#### Tầng 3 — Sinh gợi ý chỉnh sửa

Đây là bước biến "phát hiện lỗi" thành "hành động", làm theo hướng **tối ưu khung crop**:

1. Nếu phát hiện nghiêng → tính góc xoay, xoay ảnh trước.
2. Sinh tập khung crop ứng viên: nhiều tỉ lệ (giữ nguyên, 1:1, 4:5, 16:9) × nhiều vị trí/kích thước.
3. Chấm điểm mỗi khung bằng tổng có trọng số của các quy tắc ★★★/★★☆ ở tầng 2,
   **trừ điểm** nếu cắt mất vùng saliency hoặc cắt phạm khớp người.
4. Chọn khung điểm cao nhất, và chỉ đề xuất khi **điểm cao hơn ảnh gốc một khoảng đủ lớn** —
   nếu ảnh gốc đã ổn thì im lặng, đó cũng là một kết quả tốt.
5. Kiểm tra độ phân giải còn lại sau crop; nếu tụt dưới ngưỡng thì bỏ gợi ý.

Đầu ra bắt buộc gồm ba thứ: **hành động cụ thể** ("xoay −2.3°, crop theo khung này"),
**preview trước/sau**, và **lý do bằng ngôn ngữ thường**. Không bao giờ chỉ đưa ra một con số điểm.

Có thể dùng thêm model chấm điểm thẩm mỹ (**NIMA**, dựa trên MobileNet, có bản TFLite)
để so sánh chéo giữa các khung ứng viên. Nhưng **không dùng NIMA làm nguồn giải thích** —
nó là hộp đen, không nói được "sai ở đâu", trong khi mục tiêu B bắt buộc phải giải thích được.

#### Đã làm được gì (lát cắt đầu tiên của Phase 2)

Lát cắt đầu **cố ý không dùng model học máy nào**. Saliency theo phương pháp frequency-tuned
(Achanta 2009) cộng với histogram hướng cạnh bằng Sobel đủ để phủ nhóm quy tắc dưới đây,
mà không phải nhét 30 MB model vào app, không cần delegate của nền tảng, và test được bằng
ảnh dựng sẵn:

| Quy tắc | Trạng thái |
|---|---|
| Đường chân trời nghiêng | ✅ Đo chính xác tới ~1° (kiểm bằng ảnh dựng ở các góc đã biết) |
| Chủ thể bị cắt ở mép | ✅ |
| Quy tắc 1/3 | ✅ kèm ngoại lệ cận cảnh và ảnh đối xứng |
| Chủ thể quá nhỏ | ✅ |
| Cân bằng trái/phải | ✅ |
| Cắt phạm khớp, headroom, looking room | ⏳ cần pose/face detection |

**Một chi tiết kỹ thuật quan trọng:** phải làm mượt ảnh **trước khi** lấy gradient, giống bước
đầu của Canny. Với một cạnh sắc nghiêng 3°, độ dốc chỉ là 0,05 pixel mỗi cột nên trong cửa sổ
3×3 cạnh đó trông y hệt nằm ngang và góc đo được luôn ra 0. Đây là lỗi thật đã gặp khi viết
test, không phải lo xa.

**Kết quả trên 40 ảnh chụp thật (Pixel 6 Pro):** 4 ảnh có gợi ý (10%) — 2 nghiêng chân trời,
2 chủ thể bị cắt. Tỉ lệ này nằm đúng vùng mong muốn: không im lặng tới mức vô dụng, cũng không
báo bừa. Có một test corpus (`test/composition_corpus_test.dart`) canh cho tỉ lệ này khi
hiệu chỉnh ngưỡng.

**Hai lỗi tìm được khi hiệu chỉnh bộ sinh khung crop.** Ban đầu nó không đề xuất gì, kể cả
với cảnh dựng sẵn có chủ thể nằm hẳn ở góc — trường hợp mà crop chắc chắn giúp ích:

1. **Nền áp đảo bản đồ saliency.** Nền chuyển sắc hay có vân cũng lệch khỏi màu trung bình
   của ảnh, và vì nền chiếm gần hết diện tích nên nó áp đảo tổng saliency, kéo trọng tâm về
   giữa khung bất kể chủ thể nằm đâu. Test cũ không bắt được vì dùng nền phẳng lì — không
   giống ảnh thật. Sửa bằng cách chỉ giữ phần saliency vượt ngưỡng.
2. **Công thức chấm điểm trộn lẫn hai thứ khác loại.** Diện tích và phần chủ thể giữ lại
   chiếm tới 45% điểm số, nên mọi khung crop đều bị phạt nặng hơn phần thưởng bố cục mà nó
   mang lại → không khung nào vượt nổi ảnh gốc. Sửa bằng cách tách bạch: **điểm số chỉ đo
   chất lượng bố cục**, còn diện tích và phần giữ lại thành **ràng buộc cứng**, cộng một
   trọng số rất nhỏ để phân định khi hai khung ngang điểm.

Sau khi sửa, mức cải thiện của khung tốt nhất trên 40 ảnh thật trải từ −0.02 đến 0.095, nên
ngưỡng chốt ở 0.08. Kết quả cuối: **6/40 ảnh có gợi ý (15%), trong đó 3 kèm khung crop**.
Ảnh mà crop không giúp gì (mức cải thiện âm) vẫn im lặng đúng như mong muốn.

#### Từ gợi ý sang chỉnh sửa thật

Phát hiện lỗi mà không sửa giúp thì lời khuyên chỉ là lời nói suông — và với người không
rành chỉnh ảnh, một câu như "ảnh bị ám vàng, hãy chỉnh cân bằng trắng" gần như vô dụng.
Vì vậy mỗi lỗi phát hiện được đều đi kèm **một bộ lọc sửa được nó**:

| Lỗi | Bộ lọc |
|---|---|
| Đường chân trời nghiêng | Xoay + cắt phần viền trống |
| Chủ thể lệch điểm mạnh | Cắt lại khung |
| Ám màu | Cân bằng trắng theo giả định thế giới xám |
| Ảnh phẳng | Kéo giãn dải sáng (bỏ 0,5% hai đầu) |
| Ảnh tối | Nâng vùng tối bằng đường cong mũ |
| Cháy sáng | Hạ vùng sáng bằng đường cong mũ |

Các bộ lọc sửa lỗi **tính tham số từ chính ảnh đó**, không phải hằng số cố định, nên mỗi ảnh
được chỉnh đúng mức nó cần. Ngoài ra có các bộ lọc thẩm mỹ (ấm, lạnh, rực rỡ, tông phim,
đen trắng) luôn dùng được.

**Cách trình bày quyết định giá trị của tính năng này.** Không dùng thanh trượt hay tên kỹ
thuật, mà là **một dải ảnh xem trước thật**: mỗi phương án hiện kết quả của chính nó, phương
án app đề xuất được đánh dấu và chọn sẵn. Người dùng nhìn rồi chạm một lần là xong — không
cần biết "cân bằng trắng" nghĩa là gì.

> Điều này mở rộng non-goal ở §4.4: ngoài crop và xoay, app nay áp dụng cả bộ lọc màu.
> Vẫn giữ nguyên hai ràng buộc gốc — **không ghi đè ảnh gốc**, và **giữ EXIF** khi lưu.

#### Loại ảnh không phải ảnh chụp, càng sớm càng tốt

Chấm bố cục cho ảnh chụp màn hình chỉ tạo ra gợi ý vô nghĩa kiểu "đường chân trời nghiêng 3°"
trên một cửa sổ phần mềm. Mỗi ảnh tốn khoảng nửa giây và một lượt giải mã, nên phải loại
**trước khi phân tích**, không phải phân tích xong rồi bỏ. Ba tín hiệu, theo thứ tự chắc chắn:

1. **Kích thước trùng khít màn hình thiết bị** *và* không có EXIF máy ảnh. Chắc chắn nhất, và
   bắt được cả ảnh chụp giao diện tối — thứ mà hai tín hiệu dưới đều bó tay.
2. **Tên file và thư mục** (`Screenshot_*`, `Pictures/Screenshots`).
3. **Tỉ lệ cạnh ngang–dọc cao** — bắt ảnh *chụp lại* màn hình hoặc giấy tờ bằng camera, vốn
   vẫn là ảnh camera nên hai tín hiệu trên không tố cáo gì.

Kết quả phân loại được lưu vào chỉ mục, và truy vấn hàng đợi phân tích lọc thẳng bằng SQL.

#### Cần nghiên cứu thêm

- Khảo sát và chuẩn hoá bộ quy tắc bố cục từ các nguồn nhiếp ảnh phổ biến — hiện các nguồn
  đặt tên và diễn giải không thống nhất, cần chốt một danh sách chuẩn cho dự án.
- Thử nghiệm và so sánh model saliency nhẹ chạy được on-device (u2netp, BASNet rút gọn,
  ML Kit Subject Segmentation) về độ chính xác và tốc độ.
- Tham khảo các nghiên cứu về **auto-cropping thẩm mỹ** (GAIC — Grid Anchor based Image Cropping,
  VFN, VPN) để lấy cách sinh và chấm khung ứng viên; đánh giá khả năng chuyển sang TFLite.
- Xác định **trọng số** giữa các quy tắc và **ngưỡng cải thiện tối thiểu** để đề xuất —
  chỉ có thể chốt bằng thử nghiệm trên ảnh thật, không suy luận suông được.
- Xây một bộ ảnh kiểm thử có gán nhãn tay (~100–200 ảnh: nghiêng, lệch chủ thể, cắt phạm khớp,
  và ảnh bố cục tốt) để đo tỉ lệ báo sai trước khi phát hành.

### 6.6. Nguồn ảnh: thiết bị, thẻ nhớ và cloud

Thư viện ảnh thực tế nằm rải ở nhiều nơi và **thường đang được đồng bộ tự động**.
Bỏ qua điều này sẽ dẫn tới hai hậu quả nghiêm trọng: đếm sai dung lượng, và tệ hơn là
**xoá lan sang cloud** ngoài ý muốn của người dùng.

#### Trừu tượng hoá nguồn

Mọi nguồn đều đi qua một interface chung `MediaSource`, để phần phân tích không cần biết
ảnh đến từ đâu:

| Nguồn | Trạng thái | Ghi chú kỹ thuật |
|---|---|---|
| Bộ nhớ trong | Phase 1 | `photo_manager` qua MediaStore / PHPhotoLibrary |
| Thẻ nhớ ngoài (SD) | Phase 1 | Cùng API, nhưng **xoá cần SAF** trên Android scoped storage |
| **Thư mục trên đĩa** | ✅ Xong | `dart:io`, giải mã trong isolate. Đường duy nhất chạm được toàn bộ thư viện Google Photos — qua bản xuất Takeout |
| Google Drive | Phase 3 | Có sẵn `md5Checksum` trong metadata file |
| OneDrive | Phase 3 | Microsoft Graph, có sẵn `file.hashes` |
| Dropbox | Phase 3 | Có `content_hash` riêng |
| **Google Photos** | **Hạn chế** | Từ 31/03/2025 chỉ còn **Picker API** — ứng dụng bên thứ ba **không quét được toàn thư viện** |
| iCloud Photos | Không hỗ trợ | Không có API công khai; chỉ thấy được phần đã tải về máy qua thư viện hệ thống |

**Tối ưu quan trọng:** không cần tải ảnh cloud về để so trùng. Drive/OneDrive/Dropbox đều
trả sẵn **checksum** trong metadata → đủ để phát hiện trùng tuyệt đối. Trùng pixel thì
hash trên **thumbnail** mà API trả về. Chỉ tải ảnh gốc khi người dùng chủ động yêu cầu.

#### Cách đối chiếu: lọc rẻ trước, xác nhận sau

1. Lập chỉ mục cloud: chỉ lấy **tên + dung lượng + checksum**, không tải ảnh.
2. Lọc vòng đầu bằng khoá `tên|dung lượng` — một ảnh được sao lưu giữ nguyên cả hai.
3. Chỉ những ảnh lọt vòng đầu **mà bản trên máy chưa có MD5** mới phải đọc nội dung để tính.
4. Chỉ kết luận **đã sao lưu** khi checksum khớp.

Bước 4 là ràng buộc an toàn: ảnh mang nhãn "đã sao lưu" sẽ được app tự tick chọn xoá, nên
trùng tên và dung lượng thôi là chưa đủ. Hệ quả thực tế: **OneDrive thường không kết luận
được**, vì Microsoft Graph mặc định trả `quickXorHash` chứ không phải MD5 — khi đó ảnh giữ
trạng thái "không rõ" và không bao giờ bị tự tick.

**Trạng thái hiện tại:** tầng dữ liệu, luồng OAuth (Authorization Code + PKCE), client
Google Drive và OneDrive, bộ đối chiếu và giao diện kết nối đã xong. Client id nhập được
ngay trong Cài đặt (kèm hướng dẫn từng bước và link mở thẳng trang cấu hình), lưu vào
secure storage — không cần build lại. Chưa chạy thử được đăng nhập thật vì cần client id
do chủ dự án tự tạo.

#### Đường vòng cho Google Photos: bản xuất Takeout

Vì API đã đóng, cách duy nhất chạm được toàn bộ thư viện Google Photos là **Google Takeout**.
Điều quan trọng mà nhiều người hiểu nhầm: **Takeout không xoá metadata**, nó chuyển ngày chụp,
toạ độ và các thông tin khác sang một file `.json` nằm cạnh mỗi ảnh. Cảm giác "tải về là mất
metadata" đến từ việc chỉ copy các file ảnh và bỏ lại đống sidecar.

Luồng đầy đủ:

1. Người dùng xuất bằng Takeout, giải nén ra một thư mục.
2. Trỏ Media KIT vào thư mục đó (`FolderMediaSource`).
3. App đọc EXIF trong ảnh; **thiếu gì thì lấy từ sidecar** (`TakeoutSidecar`).
4. Toàn bộ Phase 1 và Phase 2 chạy nguyên vẹn trên nguồn này.
5. Khi lưu ảnh đã chỉnh, app **ghi metadata ngược vào EXIF** (`MetadataWriter`) — nhờ đó
   tải lên lại vẫn còn ngày chụp, toạ độ và thông tin máy.

Bước 5 là mấu chốt và cũng là chỗ từng có lỗi: `encodeJpg` chỉ ghi ra những gì có sẵn trong
`image.exif`, mà các phép xoay/crop của package `image` tạo ra ảnh mới **không mang EXIF sang**.
Không chép lại thì mọi ảnh đã chỉnh đều mất sạch metadata.

#### Đồng bộ tự động — ràng buộc an toàn quan trọng nhất của dự án

Khi thiết bị đang bật sao lưu tự động (Google Photos, OneDrive, Samsung Cloud…),
**xoá ảnh trên máy có thể xoá luôn bản trên cloud**. Thùng rác 30 ngày của app không cứu được
trường hợp này vì bản cloud đi vào thùng rác của chính dịch vụ đó, với thời hạn khác
(Google Photos 60 ngày, OneDrive 30 ngày).

Vì vậy mỗi ảnh cần thêm một trạng thái sao lưu:

| Trạng thái | Nghĩa | Hành xử |
|---|---|---|
| **Đã sao lưu** | Tìm được bản trùng trên cloud đã kết nối | Xoá bản máy là **an toàn nhất** — vẫn còn bản cloud |
| **Chưa sao lưu** | Không tìm thấy trên cloud nào | Cảnh báo: xoá là mất vĩnh viễn |
| **Không rõ** | Chưa kết nối cloud, hoặc đang bật đồng bộ mà app không kiểm chứng được | Cảnh báo chung về nguy cơ xoá lan; mặc định thận trọng |

Từ đây sinh ra nhóm đề xuất giá trị nhất cho mục tiêu A: **"ảnh đã có bản sao lưu trên cloud"** —
xoá bản máy giải phóng dung lượng mà không mất gì. Đây chính là cơ chế "giải phóng dung lượng"
mà Google Photos làm, nhưng ở đây hoạt động được với nhiều dịch vụ cùng lúc.

Nguyên tắc bắt buộc: **không bao giờ tự tick chọn xoá ảnh ở trạng thái "chưa sao lưu" hoặc
"không rõ"** khi phát hiện thiết bị đang bật đồng bộ.

### 6.7. Phân loại

- On-device image labeling (ML Kit / MobileNet, TFLite) cho nhãn chủ đề chung.
- Rule bổ sung cho các nhóm đặc thù dễ nhận diện: screenshot (tỉ lệ khớp màn hình + không có EXIF camera),
  ảnh tài liệu (nhiều text, nền trắng), ảnh tải về (đường dẫn thư mục + thiếu EXIF).

## 7. Kiến trúc & Công nghệ

- **Nền tảng**: Flutter (Android + iOS).
- **Truy cập thư viện ảnh**: `photo_manager`.

| Nhu cầu | Thư viện | Ghi chú |
|---|---|---|
| Đọc EXIF | `exif` (Dart thuần, chỉ đọc) | Chạy được trong isolate, không cần platform channel |
| Đọc/ghi EXIF (xoá GPS trước khi chia sẻ) | `native_exif` | Ghi phải qua API nền tảng; `exif` không ghi được |
| Thông tin cơ bản của asset | `photo_manager` (`AssetEntity`) | Sẵn có ngày tạo, toạ độ, kích thước, orientation — dùng trước, chỉ đọc EXIF đầy đủ khi cần |
| XMP / IPTC | Tự parse segment APP1/APP13 | Hỗ trợ trong hệ sinh thái Dart còn hạn chế; chỉ làm nếu thực sự cần |
| Decode / resize / thumbnail | `image` (Dart thuần) | Đủ cho tầng hash; chậm, không dùng cho phân tích nặng |
| Laplacian, Hough, ORB/AKAZE, RANSAC | OpenCV qua FFI (`opencv_dart`) | Dùng cho độ mờ, đường chân trời nghiêng, xác minh trùng mức 4 |
| Embedding ảnh (trùng nội dung) | `tflite_flutter` + MobileNetV3-Large | ~5–10 MB, vector ~1280 chiều — lựa chọn cho MVP |
| Embedding ngữ nghĩa + tìm kiếm bằng văn bản | `tflite_flutter` / `onnxruntime` + MobileCLIP-S0 | Nâng cấp ở Phase 3; một model gánh cả trùng nội dung, phân loại và tìm kiếm |
| Gán nhãn chủ đề, phát hiện khuôn mặt | `google_mlkit_image_labeling`, `google_mlkit_face_detection` | Dùng cho phân loại và cho việc chọn ảnh đại diện |
| Lưu chỉ mục (id ảnh, hash, embedding, điểm số) | SQLite (drift/sqflite) | Embedding lưu dạng BLOB `Float32List` |
| **Saliency map** (mục tiêu B) | U²-Net bản nhẹ `u2netp` (~4.7 MB) qua `tflite_flutter`/`onnxruntime` | Đầu vào cốt lõi cho mọi quy tắc bố cục |
| **Tách chủ thể** (mục tiêu B) | `google_mlkit_subject_segmentation` | Chỉ có trên Android — cần đường lui bằng saliency cho iOS |
| **Khung xương người** (mục tiêu B) | `google_mlkit_pose_detection` hoặc MoveNet Lightning | Phát hiện cắt phạm khớp, headroom |
| **Phát hiện đoạn thẳng** (mục tiêu B) | OpenCV `HoughLinesP` / LSD qua `opencv_dart` | Chân trời nghiêng, đường hội tụ dọc |
| **Chấm điểm thẩm mỹ** (tuỳ chọn) | NIMA (MobileNet) qua TFLite | Chỉ để so sánh chéo các khung crop — không dùng để giải thích |

> Tên và phiên bản package trên pub.dev thay đổi khá nhanh — kiểm tra lại tại thời điểm thêm dependency.
> Riêng `google_mlkit_subject_segmentation` chỉ hỗ trợ Android, cần phương án thay thế cho iOS ngay từ khâu thiết kế.

**Chiến lược model:** MVP dùng MobileNetV3 (nhẹ, rủi ro thấp). Đến Phase 3, khi cần phân loại
chủ đề và tìm kiếm, cân nhắc thay bằng MobileCLIP để dùng chung một embedding cho cả ba tính năng
thay vì chạy nhiều model song song.

**Hai chế độ xử lý — quyết định quan trọng nhất về hiệu năng:**

| | Mục tiêu A — dọn dẹp | Mục tiêu B — bố cục |
|---|---|---|
| Khi nào chạy | Quét hàng loạt, nền | Theo yêu cầu, khi người dùng mở một ảnh |
| Phạm vi | Toàn bộ thư viện | Một ảnh |
| Model dùng | Hash, sharpness, embedding nhẹ | Saliency + pose + line detection (nặng hơn nhiều) |
| Ngân sách thời gian | ≤ 3 phút / 5.000 ảnh | ≤ 500 ms / ảnh |
| Lưu kết quả | Có, cache lâu dài | Có, cache theo ảnh sau lần phân tích đầu |

Chạy saliency + pose cho toàn bộ thư viện trong lần quét là **không khả thi** về thời gian và pin.
Đây là lý do mục tiêu B phải là luồng theo yêu cầu, không phải một cột trong bảng kết quả quét.

### 7.1. Chiến lược cache cho thư viện lớn

Thư viện vài chục nghìn ảnh là bình thường. Nguyên tắc xuyên suốt: **mở app không được
chạm vào thư viện ảnh**, và **không bao giờ nạp toàn bộ danh sách vào RAM**.

| Vấn đề | Cách xử lý |
|---|---|
| Mở app bị đơ vì quét lại | Kết quả gom nhóm **lưu xuống SQLite**; mở app chỉ đọc số liệu tổng hợp. Quét chỉ chạy khi người dùng bấm, hoặc chạy nền tăng dần |
| Màn hình tổng quan phải đếm hàng nghìn ảnh | Đếm bằng **SQL `GROUP BY`**, không nạp bản ghi ảnh nào |
| Danh sách ảnh chiếm RAM | Nạp **theo trang** (`getAssetListPaged` + `LIMIT`/`OFFSET`), cuộn tới đâu nạp tới đó |
| Decode thumbnail lặp lại khi cuộn | **Cache thumbnail trên đĩa**, khoá theo `assetId + kích thước`, giới hạn tổng dung lượng và loại bỏ theo LRU |
| `ImageCache` của Flutter phình to | Đặt trần `maximumSizeBytes` ngay khi khởi động |
| Đọc toàn bộ file để tính MD5 quá tốn | Trùng tuyệt đối bắt buộc **cùng dung lượng file** → chỉ tính MD5 trong các nhóm trùng `byte_size`. Thư viện thường có rất ít nhóm như vậy |
| Lấy dung lượng file bằng `AssetEntity.file` | **Không dùng.** Trên Android 10+ scoped storage, API này sao chép nguyên file vào cache app. Dùng `AssetEntity.fileSize` (đọc cột SIZE của MediaStore) |
| Quét lại từ đầu mỗi lần | Chỉ xử lý ảnh có `modified_at` mới hơn bản ghi trong chỉ mục |
| Chờ tuần tự từng ảnh | **Nút thắt thật sự.** Sinh thumbnail là lượt gọi nền tảng, mỗi ảnh phải giải mã một JPEG chục megapixel — chờ lần lượt thì CPU nằm không. Chạy song song 6 ảnh một lúc |
| Đọc EXIF cho mọi ảnh | Quét hai lượt: lượt một chỉ dùng dữ liệu rẻ, lượt hai chỉ đọc EXIF cho ảnh mà điểm còn lưng chừng |

#### Số đo thật (Pixel 6 Pro, 254 ảnh, bản release)

| Phiên bản | Thời gian | Mỗi ảnh |
|---|---|---|
| Ban đầu (tuần tự, đọc EXIF mọi ảnh) | 56 s | 220 ms |
| Thêm quét hai lượt | 56 s | 220 ms |
| Thêm chạy song song 6 luồng | **13 s** | **51 ms** |

Hai bài học từ đợt đo này:

1. **Quét hai lượt gần như không cải thiện thời gian.** Giả thuyết ban đầu — chi phí nằm ở việc
   Android sao chép nguyên file để đọc EXIF — là **sai**. Vẫn giữ vì nó giảm I/O đĩa và pin,
   nhưng nó không phải thứ làm lần quét chậm.
2. **Chạy song song mới là thứ tạo khác biệt**, giảm 4,3 lần. Chi phí thật nằm ở việc chờ
   nền tảng giải mã ảnh, không phải ở tính toán trong Dart — bản debug (62 s) và bản release
   (56 s) gần bằng nhau đã cho thấy điều đó.

Ngoại suy: 4.000 ảnh ≈ **3,5 phút** cho lần quét đầu; các lần sau chỉ vài giây nhờ quét tăng dần.

### 7.1.1. Quét thư mục trên desktop

Trên di động, hệ điều hành giải mã ảnh bằng mã native. Trên desktop chính Dart phải làm việc
đó, nên chi phí khác hẳn. Đo trên máy 56 nhân, ảnh 12 MP:

| Bước | Chi phí |
|---|---|
| Đọc file từ đĩa | 7 ms/ảnh |
| Sinh isolate | 6 ms |
| **Giải mã JPEG đầy đủ** | **~1.050 ms** |
| Giải mã đầy đủ, 12 ảnh song song | 165 ms/ảnh |
| **Giải mã bản thu nhỏ nhúng trong EXIF** | **~9 ms** |

Ba việc đã làm, theo thứ tự hiệu quả:

1. **Dùng bản thu nhỏ nhúng sẵn trong EXIF** khi có và đủ lớn. Máy ảnh nhúng sẵn một bản
   ~384×510; giải mã nó nhanh hơn ảnh gốc **hơn 100 lần**. Đo trên bộ ảnh thật: 25/40 ảnh có.
2. **Đưa toàn bộ phân tích pixel ra khỏi isolate chính.** Trước đó `ImageAnalyzer.analyze`
   chạy trên isolate chính nên mọi luồng song song đều dồn về đó chờ — tăng số luồng cũng vô ích.
3. **Số luồng theo số nhân CPU** trên desktop (di động vẫn giữ 6, vì ở đó Dart chỉ ngồi chờ
   hệ điều hành).

Kết quả: **285 → 151 ms/ảnh**. Ngoại suy 4.000 ảnh: khoảng 10 phút cho lần quét đầu.

> **Hệ quả phải xử lý:** bản thu nhỏ do máy ảnh nhúng cho điểm độ nét cao hơn khoảng 10% so
> với tự thu nhỏ từ ảnh gốc, vì hai chuỗi lọc khác nhau. Với một ngưỡng cứng, cùng một tấm
> ảnh sẽ lúc bị coi là mờ lúc không, tuỳ đường nào được dùng. Đó là lý do ngưỡng độ mờ
> chuyển sang tính theo **trung vị của chính thư viện** (§6.3) — mọi phép co giãn toàn cục
> bị triệt tiêu.

### 7.2. Hiệu năng chung

- Toàn bộ phân tích chạy trong **isolate**, không chặn UI.
- Xử lý theo **batch**, có thể tạm dừng / tiếp tục.
- Phân tích trên **thumbnail** trước, chỉ đọc ảnh gốc khi thật cần.
- **Quét tăng dần**: lần sau chỉ xử lý ảnh mới hoặc đã thay đổi.
- Tự giảm tốc khi pin yếu hoặc máy nóng.

## 7.3. Nhận diện & giao diện

**Tên app:** Media KIT. **Icon:** hai khung ảnh chồng nhau + tia sparkle, trên nền gradient
indigo → violet. Icon được **sinh bằng code** (`tool/generate_icon.dart`) chứ không phải file
thiết kế rời, nên sửa màu hay bố cục là sửa code rồi chạy lại.

Nguyên tắc giao diện:

- **Không bám khuôn mặc định**: thay `BottomNavigationBar` bằng thanh pill nổi, thay AppBar
  phẳng bằng khối hero gradient. Nội dung tràn xuống dưới thanh nav.
- **Trả lời câu hỏi chính ngay màn hình đầu**: "dọn được bao nhiêu GB" nằm ở vị trí lớn nhất.
- **Mỗi nhóm một màu nhận diện** để phân biệt bằng mắt, không cần đọc nhãn.
- **Dark mode** đầy đủ, theo hệ thống hoặc do người dùng chọn.
- **Tuỳ chỉnh chữ**: chọn phông và cỡ chữ (85%–140%), có xem trước tại chỗ. Cỡ chữ trong app
  nhân với cỡ chữ hệ thống nên không phá thiết lập trợ năng của người dùng.
- **Đa ngôn ngữ**: Tiếng Việt (gốc) và English, chọn được thủ công hoặc theo hệ thống.

> Phông chữ hiện dùng các họ có sẵn trên máy, không tải từ mạng — app cam kết chạy offline.
> Muốn thêm phông riêng thì bỏ `.ttf` vào `assets/fonts`, khai báo trong `pubspec.yaml` và thêm
> một giá trị vào enum `AppFont`.

## 8. Nguyên tắc an toàn & quyền riêng tư

- Mọi phân tích diễn ra **trên thiết bị**; không có ảnh nào rời khỏi máy.
- **Không bao giờ xoá tự động.** Mọi thao tác xoá đều do người dùng xác nhận.
- **Thùng rác 30 ngày** với chức năng khôi phục; hiển thị rõ ảnh sẽ bị xoá vĩnh viễn khi nào.
  Từ Android 11, ảnh nằm trong thùng rác của **hệ thống** chứ không được app chép đi đâu
  (§14.1) — hết 30 ngày app thôi theo dõi, còn file thì hệ thống dọn theo hạn của nó và
  tới lúc đó vẫn khôi phục được từ thùng rác của máy.
- Cho phép **đánh dấu album/ảnh được bảo vệ**, app không đề xuất xoá chúng.
- Hỗ trợ đúng mô hình quyền: Android 13+ (`READ_MEDIA_IMAGES`) và iOS **Limited Photo Access**
  — app phải hoạt động được kể cả khi người dùng chỉ cấp quyền một phần.

## 9. Rủi ro & Giảm thiểu

| Rủi ro | Ảnh hưởng | Giảm thiểu |
|---|---|---|
| Đề xuất xoá nhầm ảnh quan trọng | Mất niềm tin, gỡ app | Thùng rác, xem ảnh lớn trước khi xoá, album bảo vệ, ngưỡng thiên về thận trọng |
| Chấm nhầm ảnh xoá phông là ảnh mờ | Đề xuất sai hàng loạt | Đo sharpness theo vùng chủ thể |
| Embedding gộp nhầm ảnh khác nhau cùng địa điểm/người thành "trùng" | Xoá mất ảnh người dùng | Tách "trùng" và "tương tự"; nhóm "tương tự" không bao giờ tự tick; bắt buộc xác minh mức 4 trước khi tự chọn |
| Chạy embedding làm chậm lần quét đầu | Vượt KPI 3 phút / hao pin | Chỉ chạy mức 3 trên ứng viên đã lọc qua mức 1–2; embedding trên thumbnail 224px; batch trong isolate, có thể tạm dừng |
| Quét chậm / nóng máy trên thư viện lớn | Người dùng bỏ giữa chừng | Xử lý theo batch, quét tăng dần, hiển thị tiến độ, cho tạm dừng |
| Gợi ý bố cục mang tính chủ quan | Người dùng thấy vô dụng | Chỉ dùng quy tắc ★★★/★★☆, đòi hỏi mức cải thiện tối thiểu mới đề xuất, luôn kèm preview + lý do, cho phép tắt |
| Gợi ý bố cục sai cho ảnh có chủ đích (chân dung giữa khung, ảnh đối xứng) | Mất uy tín ngay lần dùng đầu | Danh sách ngoại lệ ở §6.5; im lặng khi không chắc thay vì gợi ý bừa |
| Saliency/pose quá nặng nếu chạy cho toàn thư viện | Vượt KPI quét, nóng máy | Mục tiêu B là luồng theo yêu cầu từng ảnh, không nằm trong quét hàng loạt |
| `subject_segmentation` chỉ có trên Android | Mục tiêu B lệch chất lượng giữa hai nền tảng | Dùng saliency (u2netp) làm đường chung cho cả hai; ML Kit chỉ là tối ưu thêm trên Android |
| Hạn chế quyền ảnh trên iOS/Android mới | Không quét được đầy đủ | Thiết kế luồng hoạt động với quyền một phần ngay từ đầu |
| Ảnh bị strip EXIF làm hỏng việc gom nhóm burst và gom sự kiện | Nhóm sai, dòng thời gian lệch | Hạ độ tin cậy thay vì bỏ qua; khôi phục thời gian từ tên file; dựa nhiều hơn vào embedding khi thiếu metadata |
| Nhận diện "ảnh chia sẻ" chỉ dựa vào một tín hiệu (vd. tên file) | Gắn nhãn sai cho ảnh gốc | Chấm điểm tổng hợp từ nhiều tín hiệu; chỉ cảnh báo khi vượt ngưỡng tin cậy |
| Khác biệt API xoá ảnh giữa Android/iOS | Lỗi nền tảng | Trừu tượng hoá lớp thao tác file, kiểm thử riêng từng nền tảng |
| **Xoá lan sang cloud khi đang bật đồng bộ tự động** | Mất ảnh vĩnh viễn ngoài ý muốn — rủi ro nặng nhất của dự án | Trạng thái sao lưu cho từng ảnh (§6.6); không tự tick ảnh "chưa sao lưu"/"không rõ"; cảnh báo rõ trước khi xoá |
| Xoá ảnh trên thẻ nhớ ngoài thất bại do scoped storage | Thao tác im lặng không có tác dụng | Dùng SAF khi cần, báo lỗi rõ thay vì nuốt exception |
| Google Photos API không cho quét toàn thư viện | Tính năng cloud hụt kỳ vọng người dùng | Nói rõ giới hạn ngay trong app; tập trung vào Drive/OneDrive nơi API còn đầy đủ |
| Tải ảnh cloud về để phân tích làm tốn dung lượng mạng | Người dùng bị hao 4G | Dùng checksum và thumbnail do API cung cấp; chỉ tải bản gốc khi người dùng yêu cầu; mặc định chỉ chạy trên Wi‑Fi |
| Thư viện rất lớn làm mở app chậm | Người dùng bỏ app ngay lần đầu | Toàn bộ chiến lược cache ở §7.1 |
| Quét tuần tự làm thời gian vượt KPI | Người dùng chờ quá lâu ở lần dùng đầu | Đã xử lý: chạy song song 6 luồng, giảm từ 220 ms xuống 51 ms mỗi ảnh (§7.1) |
| **Phần lớn ảnh của người dùng không nằm trên máy** | App báo "chỉ có 254 ảnh" trong khi người dùng nghĩ mình có 4.000 → tưởng app hỏng | Đo trên máy thật cho thấy MediaStore chỉ có 254 ảnh, phần còn lại nằm trên Google Photos. Cần **nói rõ trong app** rằng con số này là ảnh trên máy, và đây là lý do mạnh nhất để làm phần cloud ở §6.6 |

## 10. Lộ trình

- **Phase 1 — MVP (mục tiêu A)**: quét & chỉ mục → metadata → trùng lặp → ảnh mờ →
  screenshot/ảnh tải về → luồng dọn dẹp + thùng rác.
- **Phase 2 — mục tiêu B**: điểm chất lượng ảnh → tầng trích xuất (saliency, pose, line) →
  các quy tắc ★★★/★★☆ → bộ sinh khung crop → màn hình gợi ý chỉnh sửa + áp dụng.
- **Phase 3 — mở rộng**: các quy tắc bố cục ★☆☆ → phân loại chủ đề → gom nhóm sự kiện →
  nhắc dọn dẹp định kỳ & thống kê.

Song song với Phase 1, cần bắt đầu sớm phần **nghiên cứu cho mục tiêu B** (danh sách ở cuối §6.5):
chuẩn hoá bộ quy tắc bố cục, thử nghiệm model saliency, và dựng bộ ảnh kiểm thử có gán nhãn tay.
Đây là phần tốn thời gian nhất và không thể rút ngắn bằng cách code nhanh hơn.

**Tình trạng tại v1.0.7**: Phase 1 và Phase 2 đã xong và đang chạy trên máy thật.
Những gì còn treo — kể cả vài thứ lẽ ra thuộc Phase 1, như video — nằm ở §14.

## 11. Màn Giới thiệu trong app

Một app có tính năng nhạy cảm (đọc toàn bộ thư viện ảnh, đề xuất xoá) thì phần
giới thiệu **không phải trang trí**: nó là chỗ duy nhất trả lời được ba câu hỏi
mà người dùng mới cài sẽ hỏi ngay — app này làm gì, có tự xoá ảnh không, và ảnh
của tôi có bị gửi đi đâu không.

Tách hẳn khỏi Cài đặt vì hai thứ khác mục đích: Cài đặt là chỗ **đổi** thứ gì
đó, Giới thiệu là chỗ **đọc**. Trộn chung thì phần đọc bị đẩy xuống cuối một
danh sách công tắc, đúng lúc người cần nó nhất là người vừa mở app lần đầu.

Nội dung bắt buộc, theo đúng thứ tự đó:

1. **Nhận diện** — tên app, phiên bản (kèm số dựng), nút kiểm tra bản cập nhật.
2. **Làm được gì** — các chip nhiều màu, mỗi chip một tính năng. Màu lấy đúng
   bảng màu của các mục dọn dẹp, để chip vừa là giới thiệu vừa là chỉ dẫn:
   người dùng gặp lại đúng màu đó ở màn tương ứng.
3. **Bắt đầu nhanh** — bốn bước đánh số, từ lúc quét tới lúc lưu ảnh đã chỉnh.
4. **Câu hỏi thường gặp** — mở ra được, trả lời thẳng năm câu nặng nhất: app có
   tự xoá không, chỉnh sửa có hỏng ảnh gốc không, ảnh có bị gửi đi không, sao
   đếm ít ảnh hơn tưởng, điểm số dựa trên cái gì.
5. **Quyền riêng tư** — nói rõ ranh giới: phân tích chạy trên máy; không quảng
   cáo, không theo dõi, không tài khoản; kết nối cloud chỉ đọc tên và mã băm
   file chứ không tải/gửi ảnh.
6. **Góp ý và trợ giúp** — nút gửi thư tới `mittohoa@gmail.com`, **điền sẵn
   phiên bản vào tiêu đề**: câu hỏi đầu tiên khi nhận báo lỗi luôn là "bạn đang
   dùng bản nào", hỏi lại là mất một vòng thư.
7. **Giấy phép mã nguồn mở** — mở trang giấy phép sẵn có của Flutter.

### 11.1 Cập nhật trong app

Bản đầu chỉ kiểm tra **khi người dùng tự bấm**. Về nguyên tắc thì sạch, nhưng
đo trên thực tế thì hỏng: một máy chạy 1.0.1 nằm im vô thời hạn dù 1.0.2 đã
phát hành, vì chẳng có gì báo cho biết mà bấm. Không ai vào màn Giới thiệu để
kiểm tra cập nhật.

Nên đổi thành **tự hỏi, nhưng hỏi lặng**: chỉ hỏi lúc mở app, không thông báo hệ
thống, không tác vụ nền, không xin thêm quyền chạy nền nào.

Khoảng chờ giữa hai lần hỏi lúc đầu để một ngày, với lý do "khỏi gọi mạng mỗi
lần mở app". Dùng thật mới thấy sai hướng: phát hành một bản vá buổi sáng thì
máy đã hỏi hôm đó nằm im tới hôm sau, kể cả khi người dùng mở app năm lần. Mà
thứ gọi đi chỉ là một file JSON vài trăm byte trên CDN. Rút xuống nửa giờ —
thực tế là mỗi lần mở app đều hỏi, chỉ trừ trường hợp thoát ra vào lại liên tục. Câu "chỉ chạm mạng khi bạn tự bấm" trong README và màn
Giới thiệu đã được sửa lại cho đúng thay vì để nguyên — nói sai về việc mình
chạm mạng lúc nào thì tệ hơn hẳn việc chạm mạng.

#### Vì sao là `version.json` chứ không phải API của GitHub

Bản đầu hỏi `api.github.com/repos/.../releases/latest`. Ba vấn đề:

1. API đó giới hạn 60 lượt mỗi giờ cho mỗi địa chỉ IP không đăng nhập. Một mạng
   công ty hay quán cà phê dùng chung IP là đủ chạm trần.
2. Nó trả về một cấu trúc rất lớn mà app chỉ dùng bốn trường.
3. Quan trọng nhất: **không có chỗ nào đặt sha256 của file APK**. Không có
   sha256 thì app không thể biết file vừa tải có đúng là file đã phát hành hay
   không trước khi mở trình cài đặt.

Nên chuyển sang một file JSON tĩnh do chính mình phát hành, đặt cạnh trang giới
thiệu tại `https://mittohoa.github.io/media-kit/version.json`:

```json
{
  "versionCode": 4,
  "versionName": "1.0.3",
  "releasedAt": "2026-08-03",
  "apkUrl": "https://github.com/.../MediaKIT-1.0.3-arm64-v8a.apk",
  "sizeBytes": 33787716,
  "sha256": "…64 ký tự hexa…",
  "notes": "Tóm tắt bản này, hiện nguyên văn trong hộp mời cập nhật."
}
```

So bằng `versionCode` (số nguyên) chứ không phải `versionName` (chuỗi): số
nguyên tăng dần thì không có chuyện `1.10.0` bị coi là cũ hơn `1.9.0`, cũng
không phải đoán ý nghĩa của hậu tố `-beta`.

#### Luồng

1. Mở app → nếu quá nửa giờ kể từ lần hỏi trước thì đọc `version.json`.
2. Có bản mới → hiện tấm thẻ trượt lên, kèm **nguyên văn `notes`**. Gạt đi thì
   nhớ đúng bản đó và không hỏi lại; bản mới hơn nữa vẫn hiện.
3. Bấm tải → tải nền, có thanh tiến độ, **băm SHA-256 ngay trong lúc ghi file**
   (không đọc lại lần hai chỉ để băm).
4. Băm khớp → mở trình cài đặt của hệ thống. Băm lệch → xoá file, báo thẳng là
   thứ nhận được không phải bản đã phát hành, **không tự thử lại** — thử lại chỉ
   che mất chuyện đó.
5. Gạt thẻ đi rồi thì còn chấm đỏ trên tab Cài đặt. Không có nó thì "để sau"
   thành "không bao giờ".

#### Không có chuyện tự cài im lặng

Android không cho app thường cài app khác mà không hỏi. Chỉ chủ sở hữu thiết bị
hoặc app ký bằng khoá hệ thống mới làm được. Thứ làm được là chuẩn bị sẵn file
rồi mở đúng màn xác nhận của hệ thống — còn lại một cú bấm. Trong mã và trong
giao diện gọi đúng tên việc đó là *mời cài*, không phải *tự cài*.

Hai chỗ dễ vấp phía Android:

- Gửi thẳng `file://` qua Intent thì từ Android 7.0 hệ thống ném
  `FileUriExposedException`. Phải đi qua `FileProvider`, khai báo phạm vi hẹp
  đúng thư mục chứa APK vừa tải.
- Từ Android 8.0, quyền cài đặt cấp cho **từng app một** chứ không phải một công
  tắc chung của máy. Chưa được cấp mà cứ gửi Intent thì hệ thống lặng lẽ không
  làm gì — nhìn hệt như nút hỏng. Nên `MainActivity.kt` kiểm tra
  `canRequestPackageInstalls()` trước và đưa người dùng thẳng tới màn cấp quyền
  của chính app này.

#### Phát hành

`version.json` là thứ **duy nhất** quyết định thiết bị có thấy bản mới hay
không. Phát hành mà quên cập nhật nó thì không máy nào được báo, dù file APK đã
nằm sẵn trên GitHub. Nên nó được sinh tự động từ chính file vừa dựng:

```bash
flutter build apk --release --split-per-abi
dart run tool/prepare_release.dart "Ghi chú phát hành…"
gh release create v1.0.3 build/release-1.0.3/*.apk --title "Media KIT v1.0.3"
cp build/release-1.0.3/version.json <repo-công-khai>/docs/ && git -C … push
```

Chép tay thì chỉ cần một lần quên đổi `sha256`, hoặc đổi `versionName` mà quên
`versionCode`, là mọi máy hoặc không được báo gì, hoặc tải về rồi bị chặn ở bước
đối chiếu băm.

## 12. Trạng thái từng nền tảng

| Nền tảng | Đã kiểm tới đâu |
|---|---|
| Android | Chạy và đo trên Pixel 6 Pro và Galaxy S24 FE thật, mọi tính năng kể cả video |
| Windows | Dựng và chạy được; đã đo lượt quét trên thư viện 4.000 ảnh |
| iOS | **Chưa dựng thử lần nào** |

Hai nền tảng đang được nhắm tới là **Android và Windows**; iOS để lại cho đợt
nâng cấp sau. Việc đó định hình cả những phần mới: video vào chỉ mục được làm
cho hai nguồn đó (`DeviceMediaSource` qua MediaStore, `FolderMediaSource` qua
đuôi file), còn phía iOS thì đường đọc thư viện dùng chung `photo_manager` nên
*về nguyên tắc* chạy được — nhưng chưa ai chạy, và `NSPhotoLibraryUsageDescription`
mô tả việc đọc ảnh chứ chưa nhắc tới video.

Về iOS: không có macOS thì `flutter build ios` không chạy được, nên phần này chưa
có bằng chứng nào ngoài việc mã biên dịch qua `flutter analyze`. Cái đã làm được
là rà cấu hình để lần dựng đầu tiên không vỡ ngay ở bước chuẩn bị:

- `IPHONEOS_DEPLOYMENT_TARGET` nâng từ 12.0 lên **15.5**, và thêm `ios/Podfile`
  đặt `platform :ios, '15.5'`. ML Kit Face Detection đòi 15.5; để mặc định 12.0
  thì `pod install` gãy ngay, và hai con số này lệch nhau cũng gãy.
- `NSPhotoLibraryUsageDescription` và `NSPhotoLibraryAddUsageDescription` đã có.
- `PRODUCT_BUNDLE_IDENTIFIER` đã đổi theo định danh mới.
- Phân vùng chủ thể không có trên iOS. `SubjectSegmenterService.isSupported` chỉ
  đúng trên Android nên nó không bao giờ được gọi; bản thân plugin cũng chỉ có
  lớp rỗng ở phía iOS.

Những thứ **chưa** kiểm được và cần làm khi có máy Mac: `pod install` có chạy trót
lọt với hai plugin ML Kit không, ảnh lưu ra có vào đúng thư viện ảnh không (iOS bỏ
qua tham số `relativePath` nên ảnh vào Recents chứ không vào album riêng), và
luồng OAuth cloud có quay lại app đúng không.

## 13. Quy tắc về ảnh chụp màn hình

Ảnh chụp màn hình đưa ra bất kỳ nơi công khai nào — README, trang phát hành, bài
giới thiệu — **không được chứa ảnh thật của bất kỳ ai**, kể cả của chính tác giả.
Lý do đơn giản: đăng lên Internet là không thu hồi được, và người trong ảnh
thường không có mặt lúc quyết định đăng.

Ba cách xử lý, theo thứ tự ưu tiên:

1. **Chụp những màn không có ảnh nào** — trường bong bóng ở màn Dọn dẹp, màn Giới
   thiệu, màn Cài đặt. Đủ để giới thiệu phần lớn app.
2. **Dựng thư viện ảnh minh hoạ rồi cho app chạy thật trên đó.** Đây là cách duy
   nhất khoe được các màn chi tiết nhóm, gợi ý bố cục và chấm điểm — vì chúng
   toàn là ảnh. Script `tool/generate_demo_photos.dart` sinh sẵn một bộ có đủ
   thứ để mọi tính năng đều có cái mà bắt: cụm ảnh gần giống nhau, bản sao y
   hệt, ảnh mờ, đường chân trời nghiêng, chủ thể đặt lệch.

   **Chạy trên máy ảo, không phải máy thật.** Đẩy ảnh minh hoạ vào máy thật thì
   chúng chỉ *thêm vào* thư viện chứ không thay thế: mở màn chi tiết nhóm ra
   vẫn là ảnh thật xếp trước, vì nhóm sắp theo dung lượng chứ không theo ngày.
   Máy ảo mới trắng thư viện, và đó là điều kiện duy nhất chụp được an toàn.
3. **Che kín** — chỉ dùng khi hai cách trên không được. Làm mờ thường vẫn để lộ
   dáng người và bối cảnh, nên phải che hẳn chứ không làm mờ.

Dựng bộ ảnh minh hoạ và đưa lên máy ảo:

```bash
dart run tool/generate_demo_photos.dart build/demo-photos
emulator -avd Pixel_6_Pro -no-snapshot-load
adb -s emulator-5554 shell mkdir -p /sdcard/Pictures/MediaKitDemo
adb -s emulator-5554 push build/demo-photos/. /sdcard/Pictures/MediaKitDemo/

# Từng file một, KHÔNG dùng scan_volume — xem giải thích bên dưới.
for f in build/demo-photos/*.jpg; do
  adb -s emulator-5554 shell content call \
    --uri content://media/external/images/media \
    --method scan_file --arg /sdcard/Pictures/MediaKitDemo/$(basename "$f")
done
```

`adb push` tạo hàng trong MediaStore với `is_pending=1` và `width`/`height` bằng
NULL — tức là file đã nằm đó nhưng chưa ai đọc nội dung. `photo_manager` lọc bỏ
hàng đang pending, nên app quét ra đúng số ảnh cũ và trông y như push hỏng.
`scan_volume` **không** gỡ trạng thái này (nó chỉ dò file mới); phải gọi
`scan_file` cho từng đường dẫn thì MediaStore mới thực sự mở file, điền kích
thước và bỏ cờ pending. Kiểm tra bằng:

```bash
adb shell content query --uri content://media/external/images/media \
  --projection is_pending --where "_data LIKE '%MediaKitDemo%'"
```

### 13.1 Đừng chạm mù trên máy thật

Ngày 3–4/8/2026 tôi thử giao diện chọn ảnh bằng cách gửi hàng loạt
`adb shell input tap <x> <y>` theo toạ độ đoán từ ảnh chụp màn hình trước đó.
Bố cục thật lệch so với dự đoán, nên vài cú chạm rơi vào luồng xoá và **7 ảnh
thật của người dùng bị chuyển vào thùng rác**. Phát hiện được nhờ số ảnh tụt
259 → 252 khi đang kiểm một con số khác. Khôi phục đủ cả 7 vì thùng rác giữ
30 ngày.

Chạm theo toạ độ cố định là thao tác mù: không có gì bảo đảm nút nằm đúng chỗ
mình tưởng, mà ở màn dọn dẹp thì nút nằm sai chỗ đồng nghĩa với xoá ảnh của
người ta.

**Mọi thử nghiệm có tương tác — chọn, xoá, kéo thả — làm trên máy ảo.** Máy thật
chỉ dùng để cài APK, chụp màn hình đọc, và các thao tác đọc thuần. Nếu buộc phải
chạm trên máy thật thì chụp màn hình ngay trước mỗi cú chạm và xác nhận toạ độ,
không chuỗi nhiều cú chạm liên tiếp.

## 14. Việc còn treo

Ghi lại vào lúc phát hành v1.0.7. Mỗi mục kèm chỗ đã kiểm được trong mã, để lần
sau khỏi phải dò lại từ đầu.

### 14.1 Hai nhánh chưa có hướng: video và file khác

Đây là khoảng cách lớn nhất giữa lời hứa và thứ app thực sự làm: khẩu hiệu là
**giải phóng dung lượng**, mà trên điện thoại thật video gần như luôn chiếm phần
lớn. Đo trên máy thật 259 ảnh: app báo giải phóng được 146 MB, trong khi thư mục
video của cùng máy đó có thể đang giữ vài GB.

#### Video đã vào chỉ mục — **xong**, và chỉ mới tới đó

`RequestType.common` thay cho `RequestType.image`, nguồn thư mục nhận thêm bảy
đuôi file video, và `PhotoKind.video` (giá trị **cuối** enum — `kind` lưu bằng
chỉ số, chèn vào giữa là mọi ảnh đã có đổi loại sau khi nâng cấp) đánh dấu chúng
trong cùng một bảng với ảnh. Cơ sở dữ liệu lên v6 với cột `duration_ms`.

Việc khó không nằm ở chỗ nhận chúng vào mà ở chỗ **giữ chúng khỏi mọi đường đã
xây cho ảnh**. Ba đường đều đọc trọn file vào RAM, và mỗi đường đều đủ để văng
app ngay giữa lần quét đầu:

- **sinh thumbnail để phân tích** — bỏ hẳn cho video (`_videoRecord`), nên không
  băm, không đo nét, không đoán giấy tờ. Đằng nào cũng vô nghĩa: dHash của khung
  hình đầu không nói được hai video có giống nhau không;
- **băm MD5 cho ứng viên trùng dung lượng** — `sizeCollisionCandidates` loại
  video ra bằng SQL. Hai video cùng dung lượng là chuyện rất thường;
- **đọc EXIF** — `needingExif` loại video ra.

Thêm hai chỗ nữa: phân tích bố cục loại video qua `_nonPhotographicKinds` (chấm
điểm một khung hình tĩnh không nói được gì về video), và `AndroidManifest` phải
khai `READ_MEDIA_VIDEO` — thiếu nó thì Android 13+ từ chối quyền video **không
báo gì**, thư viện hiện ra chỉ có ảnh, đúng như thể tính năng chưa được làm.

Vì chưa có băm nào, video không lọt vào nhóm dọn dẹp nào — mọi nhóm chỉ gom
những dòng *có* băm. Đó là trạng thái đúng cho bước này: app **thấy** và **đếm**
đúng dung lượng video, còn việc chỉ ra video nào đáng xoá là bước sau.

Nguồn thư mục (bản Windows) để trống thời lượng: muốn biết thì phải giải mã file.
`null` ở đó không được hiểu là 0 giây — bộ lọc thô của bước sau là cặp (dung
lượng, thời lượng), coi mọi video không rõ thời lượng là cùng 0 giây thì chúng
dính hết vào nhau.

#### Chặn cứng: thùng rác hiện tại không dùng được cho video — **đã gỡ**

Bản cũ gọi `originalBytes()` — **nạp trọn file vào RAM** rồi ghi ra một bản sao,
xong mới xoá bản gốc. Với ảnh 3 MB thì không sao. Với video 1 GB thì hai chuyện
xảy ra: nạp 1 GB vào RAM là văng app, chắc chắn chứ không phải "có thể"; và kể
cả stream được thì nó vẫn **nhân đôi dung lượng tạm thời** — trên đúng cái máy
mà người ta đang dọn *vì nó đầy*.

Đã sửa làm hai bước, cả hai đều xong:

1. `MediaSourcePort.copyOriginalTo` chép theo dòng, không qua RAM. Đỉnh bộ nhớ
   không còn phụ thuộc cỡ file.
2. `TrashHolder` — thùng rác có **hai lối giữ file**, chọn theo thứ nguồn đỡ
   được. Android 11+ đi `MediaStore` (`moveToTrash`/`restoreFromTrash`): file
   **nằm nguyên tại chỗ**, không chép một byte nào, nên cỡ file hết là vấn đề.
   Android 10 trở xuống và desktop vẫn chép sang thư mục riêng như cũ.

Ba chỗ lệch giữa hai lối, đã xử lý và có test (`trash_system_test.dart`):

- **Ảnh xem trước phải lưu trước.** Vào thùng rác của hệ thống rồi thì ảnh biến
  khỏi mọi truy vấn thông thường, hỏi thumbnail lúc vẽ lưới là muộn. App giữ sẵn
  một bản 720px (~100 KB) — vẫn rẻ hơn hàng nghìn lần so với chép cả file.
- **Xoá hẳn phải nhờ nguồn**, vì file vẫn nằm trong thư viện chứ không nằm trong
  thư mục của app.
- **Hết hạn thì thôi theo dõi, không gọi xoá.** `purgeExpired` chạy lúc mở app,
  mà xoá trên Android 11+ phải qua hộp thoại xác nhận của hệ thống — gọi ở đó là
  một hộp thoại bật lên chẳng vì thao tác nào người dùng vừa làm.

Còn lại: hộp thoại xác nhận của hệ thống hiện ở mỗi lượt xoá và mỗi lượt khôi
phục. `MANAGE_MEDIA` bỏ được nó, nhưng đó là quyền phải xin riêng trong Cài đặt
và nó chạm thẳng vào §8 — chưa quyết.

Việc thêm video vào danh sách quét (`RequestType.image` ở `device_media_source`)
thì vẫn chưa làm; giờ nó không còn xây trên nền lún nữa.

#### Đo trên máy thật: Galaxy S24 FE, 3.583 mục, 31,7 GB

Lần đầu toàn bộ phần video chạy đầu-cuối trên máy người dùng. Số đo, cùng một
thư viện, cùng một máy:

| Bước | Trước | Sau |
|---|---|---|
| Phân tích (mọi mục đã có chỉ mục) | 27–39 giây | 4,8 giây |
| Xem video, lượt đầu | 61 giây, **chạm trần, chưa xong** | 44 giây, **34/34 xong** |
| Xem video, lượt sau | — | **8 ms** |
| Tổng một lượt quét lại | ~102 giây | **~7 giây** |

Hai chỗ sửa, cả hai chỉ lộ ra khi cắm cáp vào máy thật:

- **`asset.fileSize` là một lời gọi qua kênh platform cho từng mục**, không phải
  một trường có sẵn. Với 3.583 mục đã nằm sẵn trong chỉ mục, bước "phân tích"
  mất 27–39 giây **để không làm gì cả**. Tầng trên vốn đã đưa `knownModified`
  xuống nguồn đúng để tránh chuyện đó; nguồn thư mục dùng nó từ lâu, nguồn thiết
  bị thì bỏ quên.
- **`OPTION_CLOSEST` phải giải mã từ keyframe trước đó tới đúng mốc.** Với bản
  quay màn hình 2,79 GB (keyframe rất thưa) một khung mất hàng giây. Đổi sang
  `OPTION_CLOSEST_SYNC` và bỏ qua video trên 300 MB thì hàng đợi chạy hết trong
  ngân sách.

**Cỡ trang: đo lại thì giả thuyết chỉ đúng một nửa.** Nâng cỡ trang trên di
động từ 60 lên 300 (60 lượt qua kênh → 12) cắt bước phân tích từ 6,2 xuống
**4,5 giây** — nhanh thêm 27%, không phải hai phần ba như dự đoán ghi ở đây
trước đó. Nghĩa là chi phí **không nằm ở số lượt gọi mà ở lượng dữ liệu chuyển
qua kênh**: 3.586 bản ghi asset vẫn phải đi qua đúng một lần, gộp thành ít gói
hơn không làm chúng nhỏ đi.

Cỡ trang để khác nhau theo nền tảng, cùng lý lẽ với `defaultConcurrency()`:
trên desktop mỗi trang là một lượt `Future.wait` **mở từng file** đọc header,
nên trang lớn chỉ đổi chỗ nghẽn này lấy chỗ nghẽn khác.

Muốn cắt tiếp thì phải đổi cách hỏi chứ không phải đổi cỡ trang: chỉ hỏi những
mục sửa sau lần quét trước (`updateTimeCond`). Nhưng khi đó app không còn thấy
toàn bộ danh sách id, mà `pruneMissing` cần đúng thứ đó để dọn ảnh đã bị xoá
bằng app khác — nhanh thêm vài giây, đổi lấy việc ảnh đã xoá nằm lại trong chỉ
mục. **Không làm**: 4,5 giây cho thư viện 31,7 GB là con số dùng được, còn đánh
đổi kia mua sự phức tạp bằng một chỗ sai âm thầm.

Không có khung hình nào bị bỏ trong suốt lượt quét. Chỗ duy nhất trong log có
dấu hiệu nghẽn là lúc mở app: `Skipped 46 frames` (~0,8 giây), tức là `init()`
dọn thùng rác, dọn cache và đọc lại số liệu trên luồng chính.

#### Tiêu chí "trùng" của video: ba mức, ba giá khác nhau

Đường của ảnh là MD5 → dHash → embedding. Với video mức nào cũng phải sửa, vì
đọc file đắt gấp hàng trăm lần.

**Mức 1 — trùng file. Đã làm.** Hai chặng. Chặng một là cặp **(dung lượng, thời
lượng)** lọc bằng SQL, **không đọc byte nào** — `videoHashCandidates`. Video lẻ
loi không bao giờ đi tiếp: không có cái thứ hai cùng cặp số thì nó không thể là
bản sao của ai cả. Chặng hai băm ba đoạn 1 MB (đầu/giữa/cuối) của phần còn đụng
nhau — `VideoSampleHash`.

Ba đoạn chứ không một: nhiều định dạng để phần mô tả ở đầu file, nên hai video
khác nhau vẫn có thể giống nhau ở megabyte đầu. Đoạn giữa rơi vào giữa dòng hình
ảnh; đoạn cuối bắt file cắt ngắn hoặc tải dở. Dung lượng được trộn vào dấu, nên
hai file khác cỡ không bao giờ ra cùng một dấu.

Đọc đúng ba đoạn đó mà không chép file là chỗ phải xuống tới `MainActivity`: từ
Android 10, lấy `File` của một asset khiến hệ thống chép nguyên file vào cache
app trước — 1 GB chép ra chỉ để đọc 1 MB ở giữa. `openFileDescriptor` đọc thẳng
từ chỗ file đang nằm.

**Bằng chứng ở đây yếu hơn ảnh một bậc**, và điều đó phải hiện ra ở giao diện:
nhóm dựng từ dấu lấy mẫu **không tự tick chọn xoá**. Mỗi mục nặng hàng trăm MB
và thường không có bản nào khác ở đâu cả.

**Chặng nâng bằng chứng — đã làm.** `VideoFullHash` băm **trọn file** cho đúng
những video đã lọt lưới lấy mẫu — lúc đó chỉ còn vài cặp — đọc theo khúc 4 MB
qua `byteRange`, không qua `originalBytes`, nên đỉnh bộ nhớ là một khúc bất kể
cỡ file. Khi **mọi** bản trong nhóm đã xác nhận cùng MD5 thì nhóm được tự tick:
cùng từng byte là mức bằng chứng duy nhất §6.2 cho phép, và quyền gắn với từng
nhóm chứ không với category. Ba ràng buộc giữ cho chặng này không phá lượt quét:

- chạy **cuối** ngân sách video — nó chỉ nâng bằng chứng cho nhóm đã có, hết
  giờ thì nhóm vẫn hiện, chỉ chưa tự tick, lượt sau băm tiếp;
- file trên 1 GB (`VideoFullHash.maxBytes`) không băm, và chặn ngay trong SQL
  của `videoFullHashCandidates` — không chặn thì video quá cỡ quay lại hàng đợi
  ở mọi lượt quét, tính rồi trả null, một hàng đợi không bao giờ cạn;
- MD5 lưu ngay từng cái một: mỗi mã là hàng chục giây đọc đĩa, mất vì thoát app
  giữa chừng thì đắt hơn hẳn một dấu lấy mẫu.

Video có MD5 vẫn đi đường video (`_exactDuplicates` bỏ qua video): nhãn "Ảnh
trùng", cách bày xấp ảnh và cách chọn bản giữ theo độ nét đều là của ảnh, sai
hết với video. MD5 ở đây là MD5 thuần của file — trùng khớp định nghĩa
`md5Checksum` của Drive, nên về sau đối chiếu sao lưu cloud dùng lại được.

**Mức 2 — cùng nội dung, khác file. Đã làm.** Clip gửi qua Zalo/Messenger bị nén
lại: khác byte, khác dung lượng, thời lượng lệch vài chục mili giây. Lấy 5 khung
hình ở **mốc tương đối** (10/30/50/70/90% thời lượng) rồi chạy đúng dHash sẵn có
— `VideoFingerprint`. dHash bền với việc nén lại, đó chính là lý do nó được chọn
cho ảnh.

Bốn quyết định đáng ghi lại:

- **Mốc tương đối, không tuyệt đối.** Hai bản của cùng một clip lệch nhau vài
  chục mili giây thì mốc tương đối vẫn rơi vào cùng một cảnh. Đổi lại, bản **cắt
  ngắn** không khớp bản gốc — đúng như mong muốn, hai thứ đó không phải bản sao
  của nhau.
- **Không lấy ở 0% và 100%.** Đầu video hay là cảnh tối hoặc logo, cuối hay là
  màn hình đen; cả hai giống nhau ở *mọi* video và sẽ kéo hai clip chẳng liên
  quan lại gần nhau.
- **`OPTION_CLOSEST` chứ không `OPTION_CLOSEST_SYNC`.** Bản sync nhảy tới
  keyframe gần nhất, mà hai lần mã hoá đặt keyframe ở những chỗ khác nhau — hai
  khung lấy về có thể cách nhau cả giây.
- **Cho phép đúng một khung trượt.** Cảnh chuyển động nhanh thì một mốc lệch vài
  khung hình cũng đủ đổi hẳn dHash. Nới hơn nữa thì hai clip quay cùng một chỗ
  vào hai lúc khác nhau bắt đầu dính vào nhau.

Lưới lọc trước khi giải mã là **thời lượng** (±1 giây), không phải dung lượng —
nén lại đổi hẳn dung lượng nhưng gần như không đổi thời lượng. Video đã biết là
trùng file thì bỏ qua, tránh giải mã hai lần và tránh một video nằm hai nhóm.

**Windows không có bộ giải mã nào**, nên `FolderMediaSource.videoFrame` trả về
null và bản Windows dừng ở mức 1. Không hỏng gì — mục "video trùng nội dung"
rỗng thay vì sai. Kèm ffmpeg vào để gỡ chỗ này là một quyết định khác hẳn.

**Mức 3 — embedding video.** Bỏ. Không xứng chi phí trên máy người dùng.

#### Mục đáng tiền khác của video

**Clip lỡ tay — đã làm.** Video 1–2 giây (`DuplicateGrouper.accidentalClipMs`),
quay nhầm trong túi hoặc lỡ tay chạm nút. Nhận ra chỉ bằng thời lượng, không đọc
byte nào, tỉ lệ trúng rất cao. Ngưỡng thấp là cố ý: clip 3–4 giây có thể là một
khoảnh khắc thật. Trên Windows mục này luôn rỗng vì nguồn thư mục không biết
thời lượng — và đó là hành vi đúng: coi "không biết" là 0 giây thì **mọi** video
đều bị gọi là clip lỡ tay.

**Video quay màn hình — đã làm.** Nhận bằng tên file/thư mục
(`VideoRules.isScreenRecording`), y như dự tính: đó là bằng chứng duy nhất —
video không có EXIF camera để đối chiếu, kích thước khung hình thì trùng khít
video thường — và may là bằng chứng rất chắc, vì tên do chính app quay đặt.
Mẫu phủ Samsung (`Screen_Recording_*`, `DCIM/Screen recordings`), Android gốc
(`screen-YYYYMMDD-*`), iOS ReplayKit (`RPReplay_*`) và thư mục của các app quay
bên thứ ba. Nhận diện chạy ở **bước gom nhóm** trên `file_name`/`relative_path`
đã có sẵn trong chỉ mục — không migration, logic thuần, test được
(`screen_recording_test.dart`). Xếp **trước** ba mục xem lại: bản quay màn hình
15 phút mà rơi vào "video quá dài" là mất đúng thông tin đáng giá nhất.
Không tự tick — tên file nói "đây là bản quay màn hình", không nói "bản này
thừa"; buổi họp quan trọng cũng là một bản quay màn hình.

> `CleanupCategory.videoScreenRecording` nằm **cuối** enum dù thứ tự khai báo
> là thứ tự hiển thị: `category` lưu xuống DB theo chỉ số, chèn vào giữa là mọi
> nhóm đã lưu đổi loại sau khi nâng cấp — cùng ràng buộc với `PhotoKind.video`.

**Chưa làm:** video nặng lâu không mở.

Bốn nhóm video hiện có, xếp theo mức bằng chứng giảm dần. Duy nhất nhóm video
trùng **đã xác nhận trọn file** được tự tick; còn lại đều chỉ hỏi:

| Nhóm | Bằng chứng | Chi phí |
|---|---|---|
| Video trùng, đã xác nhận | cùng MD5 của trọn file | đọc trọn các bản đã lọt lưới, một lần trong đời |
| Video trùng | cùng dung lượng + thời lượng + ba đoạn byte lấy mẫu | 3 MB đọc |
| Video trùng nội dung | 5 khung hình, dHash khớp gần hết | 5 lần giải mã khung |
| Clip lỡ tay | thời lượng ≤ 2 giây | không đọc gì |
| Video quay màn hình | tên file/thư mục do app quay đặt | không đọc gì |

#### Còn phần biên tập video thì sao

Cắt, ghép, crop, gắn logo, lồng tiếng, khử nhiễu, tăng độ nét — đã cân nhắc và
**chưa làm**, vì chúng chia làm hai loại rất khác nhau:

- **Cắt và tách — đã làm.** `VideoTrimmer.kt`: `MediaExtractor` đọc gói dữ liệu
  đã nén ra, `MediaMuxer` ghi thẳng sang file mới. Không giải mã, không mã hoá
  lại, không ffmpeg. Ba chỗ phải nói ra chứ không được im lặng:
  - **Điểm bắt đầu lùi về keyframe gần nhất** (`SEEK_TO_PREVIOUS_SYNC`), thường
    sớm hơn dưới hai giây. Đó là bản chất của cắt lossless: video nén liên
    khung, cắt giữa chừng thì phần đầu không có gì để dựa vào. Hàm trả về mốc
    **thật** và giao diện hiện đúng nó — hứa cắt ở giây 10 rồi giao file bắt đầu
    ở giây 8,4 là nói sai về thứ mình vừa làm.
  - **Chép mọi track**, không riêng video. Bỏ track âm thanh là cắt xong video
    câm, mà người dùng chỉ phát hiện sau khi bản gốc đã bị xoá.
  - **Góc xoay nằm ở metadata**, không ở pixel (`setOrientationHint`). Quên nó
    là video dọc thành quay ngang.
  Bản gốc không bị đụng tới, nên cắt xong dung lượng *tăng* cho tới khi người
  dùng tự xoá bản gốc — màn cắt nói thẳng điều đó.

  **Đã kiểm trên máy thật** (Galaxy S24 FE, clip 1080×1920 quay bằng chính máy):
  cắt một đoạn và tách làm hai đều ra file đúng, cả ba đều `1080×1920` — góc xoay
  giữ nguyên — và log `MPEG4Writer` ghi `817 frames - Audio`, tức là track âm
  thanh được chép sang. Điểm bắt đầu lùi 1,1–1,2 giây so với mốc đã chọn, đúng
  khoảng cách keyframe của video quay bằng điện thoại.

  Lần thử đầu tiên lộ ra một lỗi mà test không bắt được: `saveEdited` gọi
  `saveImageWithPath` — **API của ảnh** — nên video cắt xong ghi ra file sạch sẽ
  rồi không bao giờ vào được thư viện. Người dùng bấm Cắt, app báo xong, mà chẳng
  thấy video mới đâu. MediaStore cũng đòi video nằm dưới `Movies/` chứ không phải
  `Pictures/`. Đây là kiểu lỗi chỉ máy thật mới nói ra: mọi thứ *trước* bước lưu
  đều đúng, và test thì dừng ở ranh giới đó.
- **Nén lại** (4K→1080p, H.264→HEVC) — món giải phóng dung lượng lớn nhất, nhưng
  phải mã hoá lại: vài phút CPU, đúng thứ §14.3 vừa đi chặn. Chưa làm.
- **Biến app thành trình biên tập:** phần còn lại. Đều cần bộ mã hoá, tức là
  ffmpeg: +20–40 MB mỗi ABI (app đang 32 MB) kèm chuyện giấy phép, và gói
  `ffmpeg_kit_flutter` chính thức đã ngừng bảo trì — chỗ này cần kiểm lại trước
  khi đặt cược. Nặng hơn: §8 hứa không ghi đè bản gốc, nên mọi bản sửa là file
  mới, tức là **thêm** dung lượng — một app dọn dẹp mà mỗi thao tác lại sinh
  thêm file thì hai nửa của nó chống nhau. Riêng khử nhiễu và tăng độ nét là mô
  hình chạy trên từng khung hình: một phút video là 1.800 lần suy luận, trên
  điện thoại là hàng giờ.

Nếu làm phần biên tập thì nên là một **quyết định pivot có ý thức** — đổi cả
định vị, dung lượng cài đặt và đối thủ — chứ không bò vào từng tính năng một.

#### Video khác ảnh ở ba chỗ

**Đọc file đắt gấp hàng trăm lần.** MD5 một video 1 GB mất vài giây; vài chục
video là vài phút. Không thể băm hết. Lọc thô bằng **(dung lượng, thời lượng)**
trước — khác cặp số đó thì chắc chắn không trùng — rồi mới băm những cặp còn
đụng nhau. Đây cũng chính là cách `content_hash` hiện tại né việc băm mọi ảnh.

**"Tương tự" phải lấy mẫu khung hình.** Rẻ nhất: thumbnail ở giây 0 và giữa
clip, rồi chạy đúng dHash + BK-tree đã có. Không cần thuật toán mới.

**Mục đáng tiền lại khác ảnh.** Với ảnh là trùng và mờ. Với video:

- **video trùng** — cực phổ biến, một clip chuyển qua ứng dụng nhắn tin vài lần
  là nằm lại vài bản;
- **clip lỡ tay** — 1–2 giây, quay nhầm trong túi. Nhận ra chỉ bằng thời lượng,
  tỉ lệ trúng rất cao;
- **video dài và nặng đã lâu không mở**.

Chấm điểm và bố cục không áp dụng được cho video, phải loại khỏi mục tiêu B
giống cách ảnh chụp màn hình đang bị loại (§6.5).

#### Thư mục do người dùng chọn — **đã làm**, và nó cộng thêm chứ không thay thế

Người dùng chỉ một thư mục qua bộ chọn của hệ thống (`ACTION_OPEN_DOCUMENT_TREE`),
app quét nó **cùng lúc** với thư viện của máy. Cách dễ hơn là đổi qua lại — một
lúc một nguồn, đúng như bản Windows đang làm — nhưng nó bỏ mất đúng thứ đáng
tiền nhất: **ảnh trùng giữa hai nơi**. Một tấm tải về nằm cả trong `Download`
lẫn trong thư viện là dạng trùng hay gặp bậc nhất trên máy thật, và không nguồn
đơn lẻ nào nhìn thấy cả hai.

`CompositeSource` gộp nhiều nguồn thành một, và ba chỗ phải xử lý riêng:

- **Định tuyến theo id.** Mỗi thao tác đến với một id và phải về đúng nguồn giữ
  nó; nguồn nào biết id của mình thì khai qua `IdScoped` (SAF nhận mọi id bắt
  đầu bằng `content://`). **Không** thêm tiền tố vào id: chỉ mục đang lưu id
  thật, đổi chúng là mọi dòng cũ thành mồ côi và người dùng mất cả chỉ mục lẫn
  thùng rác chỉ vì nâng cấp app.
- **Phân trang nối tiếp.** Đi hết trang của nguồn này rồi mới sang nguồn kia;
  trang cuối của mỗi nguồn có thể ngắn. Không sao — tầng trên chỉ dừng ở trang
  **rỗng**, không dừng ở trang ngắn.
- **Thùng rác hỏi theo từng ảnh.** Thư viện máy đi `MediaStore` (Android 11+),
  thư mục SAF thì không có thùng rác nào nên app tự chép. Hỏi một lần cho cả
  lượt là một nửa số ảnh đi nhầm đường: gửi ảnh trong thư mục sang `MediaStore`
  thì hệ thống không biết nó là gì — ảnh vẫn nằm nguyên đó trong khi app đã ghi
  rằng đã xoá. `supportsSystemTrash` vì thế nhận `id`.

Phía Android, `SafFolder.kt` làm mọi việc với `content://` document URI: liệt kê
đệ quy qua `DocumentsContract`, đọc theo khoảng và sinh thumbnail qua
`openFileDescriptor` (không chép file), xoá qua `deleteDocument`, và khôi phục
từ thùng rác bằng `createDocument` + chảy theo dòng. Quyền được giữ qua các lần
mở app bằng `takePersistableUriPermission` — thiếu bước đó thì lần mở sau app
mất quyền đọc chính thư mục nó đang hiển thị, và mọi ảnh trong đó thành ô xám
không lý do. Bỏ thư mục thì **trả quyền lại** (`releasePersistableUriPermission`)
chứ không giữ im lặng.

Hai chỗ đã biết là hụt, ghi lại để khỏi tưởng là lỗi:

- Ảnh trong thư mục SAF **không có EXIF được đọc**: `needingExif` đi qua đường
  của nguồn thiết bị. Thời điểm sửa file thay cho ngày chụp.
- Khôi phục từ thùng rác ghi vào **gốc** thư mục đã chọn chứ không phải đúng thư
  mục con cũ: SAF không cho tạo document ở một đường dẫn tuỳ ý.

#### File khác: đi đường hẹp, đừng xin quyền toàn bộ bộ nhớ

Vấn đề không phải kỹ thuật mà là **quyền**. Từ Android 11, đọc file bất kỳ cần
`MANAGE_EXTERNAL_STORAGE` — quyền "Truy cập tất cả file". App này không lên Play
nên xin được, nhưng nó phá đúng thứ đang là điểm mạnh: một app hứa "chỉ đụng vào
ảnh của bạn" mà lại xin quyền đọc toàn bộ bộ nhớ thì lời hứa đó không còn ai
tin, và §8 không còn kiểm chứng được nữa.

Đường hẹp đúng hơn: **để người dùng tự chọn một thư mục** qua bộ chọn của hệ
thống (SAF, `ACTION_OPEN_DOCUMENT_TREE`). Họ chỉ đường tới `Download` hay thư
mục của ứng dụng nhắn tin, app chỉ thấy đúng chỗ đó, không quyền nào đáng sợ.

Cái hay là **app đã có sẵn hình dạng này**: `FolderMediaSource` đang phục vụ bản
Windows chính là "đọc ảnh từ một thư mục do người dùng chỉ" (§6.6). Thêm nhánh
Android cho nó là mở rộng một trừu tượng đã có, không phải dựng cái mới.

#### Thứ tự đề xuất

1. ~~Sửa thùng rác: chuyển file thay vì sao chép.~~ Xong.
2. ~~Đưa video vào danh sách quét.~~ Xong — app thấy và đếm đúng dung lượng
   video, nhưng chưa chỉ ra được video nào đáng xoá.
3. ~~Video trùng file và clip lỡ tay.~~ Xong — lọc thô bằng cặp (dung lượng,
   thời lượng) rồi băm ba đoạn lấy mẫu.
4. ~~Video trùng **nội dung** (bản đã bị nén lại).~~ Xong trên Android; Windows
   dừng ở mức 1 vì không có bộ giải mã.
5. ~~Thư mục do người dùng chọn qua SAF.~~ Xong trên Android — và nó **cộng
   thêm** vào thư viện chứ không thay thế, nên tìm được cả ảnh trùng giữa thư
   mục đó với thư viện.

### 14.2 Cảnh báo đồng bộ hiện ở mọi lần xoá nên mất tác dụng — **đã sửa**

`group_detail_screen.dart` hiện khối cảnh báo đỏ khi `backup != backedUp`. Mà
`BackupStatus.backedUp` chỉ được đặt sau khi đối chiếu với cloud
(`library_scanner.dart`), và cloud cần client id — chưa cấu hình thì mọi ảnh
vĩnh viễn ở `unknown`.

Kết quả: hộp đỏ "Xoá ảnh trên máy có thể xoá luôn bản trên cloud" xuất hiện
trong **từng lần xoá**, kể cả trên máy không bật đồng bộ gì cả. Cảnh báo lúc nào
cũng bật thì người ta ngừng đọc, và đúng ngày nó thật sự đúng thì không ai để ý.
Một cảnh báo mất tác dụng còn tệ hơn không có cảnh báo, vì nó tạo cảm giác đã
được nhắc.

Đã chọn đường thứ hai — chỉ cảnh báo khi máy có app đồng bộ ảnh. Đường thứ nhất
(nối cloud để `backedUp` chạy thật) không giải quyết được chính chỗ này: người
không nối cloud vẫn bị cảnh báo y như cũ.

**Tín hiệu dùng được, và giới hạn của nó.** Android không có API công khai nào
cho biết máy đang *bật* sao lưu ảnh. Thứ hỏi được là app đó **có được cài hay
không** (`SyncApps`, `MainActivity.installedSyncApps`) — thô hơn hẳn, nhưng cắt
đúng trường hợp cảnh báo chắc chắn là nhiễu: không có app nào thì không có gì để
lan sang. Nên lời cảnh báo cũng đổi theo cho khớp với thứ app thật sự biết: "Máy
này có Google Photos. Nếu đang bật sao lưu tự động, …" thay vì phán rằng đồng bộ
đang bật.

Hai chỗ dễ vấp:

- Từ Android 11, hỏi về một gói khác chỉ trả lời được khi gói đó đã khai trong
  `<queries>` của manifest. Thiếu khai báo thì câu hỏi luôn nhận về "không có" —
  tức là cảnh báo **tắt hẳn trên đúng những máy cần nó**, im lặng, không lỗi nào.
- Quyết định "có cảnh báo không" nằm ở `SyncApps.shouldWarn`, tách khỏi phần gọi
  kênh platform để test được (`sync_warning_test.dart`). Cả hai hướng sai đều
  tốn: hiện thừa thì cảnh báo thành nhiễu như cũ, hiện thiếu thì mất một cảnh
  báo mà §9 xếp là rủi ro nặng nhất của dự án.

Việc app nhìn xem máy có app nào đã được ghi vào phần Quyền riêng tư trong màn
Giới thiệu — nói sai về thứ mình có nhìn tới thì tệ hơn hẳn việc nhìn tới nó.

### 14.3 Tự phân tích bố cục ở thư viện lớn — **đã chặn, còn phải đo**

Từ v1.0.7, phân tích bố cục tự chạy sau khi quét. Đo trên 259 ảnh: khoảng 90
giây, chấp nhận được.

Thư viện điện thoại thường 10–30 nghìn ảnh, cùng nhịp đó là **một tới ba tiếng**
CPU chạy liên tục ngay sau lần quét đầu — máy nóng, tụt pin, mà người dùng không
hiểu vì sao. Trước v1.0.7 chuyện này không xảy ra vì phải tự bấm, tức là người
bấm biết mình đang yêu cầu gì.

**Đã làm: ngân sách cho lượt tự chạy.** `CompositionIndexer.run(budget:)` dừng
khi hết giờ và để phần còn lại cho lần mở app sau; `SuggestController` truyền
`autoBudget` = 90 giây cho lượt tự chạy và **không** truyền gì cho nút bấm tay.
90 giây vì đó là nhịp duy nhất dự án đã đo trên máy thật và đã chấp nhận — không
phải một con số bịa ra cho tròn.

Ba chỗ dễ sai, đều im lặng, đều có test giữ (`composition_budget_test.dart`):

- kiểm ngân sách **trước** khi làm gì thì lượt nào cũng không làm được ảnh nào,
  và phần còn dở đứng nguyên mãi mãi. Nên chỗ kiểm nằm *sau* khi đã lưu một ảnh;
- nhiều lượt ngắn nối nhau phải phân tích hết thư viện — chỉ cần một chỗ quên
  đánh dấu là vòng lặp đó không bao giờ kết thúc;
- hết ngân sách không phải là người dùng bấm dừng, nên nó không được đụng vào
  `cancel()`.

Ảnh mới nhất được xử lý trước (`needingComposition` sắp theo thời gian chụp giảm
dần), nên phần làm được trong ngân sách cũng là phần người dùng hay xem nhất.
Lúc dừng giữa chừng, thẻ ở tab Gợi ý nói thẳng còn bao nhiêu ảnh và bấm đâu để
chạy tiếp — không thì người dùng chỉ thấy một thanh tiến độ đứng nửa chừng.

**Còn phải đo.** Ngân sách chặn được phần tệ nhất, nhưng con số 350 ms mỗi ảnh
vẫn là suy ra từ đúng một lượt đo 259 ảnh. `composition_bench_test.dart` đo nhịp
thật rồi chiếu lên 10k/30k ảnh và in ra một lượt tự chạy làm được bao nhiêu:

```bash
MEDIA_AI_BIG_CORPUS=<thư mục ảnh> flutter test test/composition_bench_test.dart --timeout none
```

Chạy trên máy tính thì nhanh hơn điện thoại nhiều, nên nó để so tương đối chứ
không thay được một lượt đo trên máy thật. Nếu đo xong vẫn thấy 90 giây mỗi lần
mở app là quá nhiều thì bước tiếp theo là **chỉ chạy tiếp khi đang sạc** — cần
thêm một plugin đọc trạng thái pin, nên chưa làm khi chưa có số liệu đòi.

### 14.4 iOS chưa dựng thử lần nào

Xem §12. Cấu hình đã rà (deployment target 15.5, quyền, Podfile) nhưng cần macOS
+ Xcode mới dựng được. Không có đường vòng nào.

### 14.5 Sáu mục phân loại mới, và cờ "bỏ qua"

Thêm một lượt sáu mục, tất cả theo cùng nguyên tắc đã dùng cho video quay màn
hình: **bằng chứng rẻ và chắc, lấy từ thứ chỉ mục đã biết sẵn**, không thêm
model nào, không đọc thêm byte nào lúc gom nhóm.

| Mục | Bằng chứng | Ghi chú |
|---|---|---|
| Ảnh tài liệu, hoá đơn | `PhotoKind.document` — đã tính sẵn từ lúc quét | Chỉ là đưa kết quả có sẵn ra thành nhóm |
| Ảnh chụp lỡ trong túi | Độ sáng trung bình ≤ 8 hoặc ≥ 250 (`mean_luma`, cột mới v9) | Ngưỡng rất chặt, cố ý — bắt nhầm ảnh đêm nghệ thuật tệ hơn bỏ sót; ảnh cũ chưa đo (`NULL`) thì không đoán |
| File hỏng | 0 byte, hoặc **đọc được byte mà không giải mã nổi** (`PhotoKind.broken`) | Thumbnail null đơn thuần KHÔNG đủ — có thể là trục trặc thoáng qua, mà nhãn dính rồi thì lần quét sau không gỡ (ảnh không đổi thì không phân tích lại). Trước đây các file này bị bỏ qua im lặng và làm số đếm lệch |
| Video tải về / qua chat | Tên `VID-*-WA*`, `received_*`; thư mục WhatsApp/Messenger/Zalo/Telegram/Download | Bản video của "bản chia sẻ" bên ảnh |
| Video cũ và nặng | Quay trước 2 năm + từ 100 MB | Thay thế khả thi cho "video nặng lâu không mở": Android không cho biết lần mở cuối, ngày quay thì biết sẵn. Xem-lại |
| Ảnh nặng bất thường | ≥ max(20 MB, 8× trung vị ảnh của thư viện) | Cùng lý lẽ ngưỡng thích ứng với ảnh mờ (§6.3). Xem-lại |

Thứ tự nhận có chủ ý: file hỏng trước mọi mục phán đoán (file không đọc được
thì "mờ" hay "tài liệu" đều vô nghĩa); ảnh chụp lỡ **trước** ảnh mờ (tấm đen
kịt chắc chắn cũng "mờ", nhưng "chụp lỡ trong túi" mới nói đúng chuyện gì xảy
ra); quay màn hình trước video-qua-chat (bản quay màn hình gửi qua WhatsApp
vẫn là bản quay màn hình).

Một lỗi test đáng ghi lại: fixture ban đầu dùng **cùng một mẫu nhiễu** cho mọi
ảnh, chỉ khác độ sáng — mà dHash so tương quan giữa các ô chứ không so độ sáng
tuyệt đối, nên cả bộ bị gom thành một cụm "ảnh tương tự" trước khi mục cần test
kịp chạy. Đúng bài học nền-phẳng-lì của §6.5, lần này ở phía test.

#### Cờ "bỏ qua, không đề xuất lại"

Người dùng đánh dấu được ảnh/video là **bỏ qua**: không vào bất kỳ nhóm dọn
dẹp nào nữa, ở mọi lần quét sau. Đây là một nửa của user story "album/ảnh cần
bảo vệ" (§5), làm ở mức từng ảnh.

- `suggest_ignored` neo vào **id ảnh** (cột mới v9), cùng lý lẽ với
  `keeper_pinned`: nhóm dựng lại sau mỗi lần quét, quyết định của người dùng
  phải sống lâu hơn thế. Bộ gom nhận các id này vào `claimed` ngay từ đầu —
  mọi bước sau tự tôn trọng. Bước trùng-tuyệt-đối xưa nay chạy đầu tiên nên
  không nhìn `claimed`; giờ phải nhìn, và có test giữ.
- Nút "Bỏ qua" đứng cạnh nút xoá trong màn chi tiết nhóm — nó là câu trả lời
  thứ hai cho cùng một câu hỏi. Không hỏi xác nhận: thao tác không mất gì.
- **Bỏ qua được thì phải xem lại được**, không thì không ai dám dùng: danh
  sách "Đã bỏ qua" nằm trong Cài đặt, "Xem xét lại" gỡ cờ và mục quay về các
  nhóm từ lần quét sau (không dựng lại nhóm ngay — gom nhóm cần nhìn toàn thư
  viện, và người dùng đang đứng trong Cài đặt chứ không chờ một lượt quét).
- Cùng vòng đời với `keeper_pinned`: file bị sửa đổi (modified) thì bị phân
  tích lại và cờ mất theo — chấp nhận được, bản đã sửa đáng được xem xét lại.

#### Bản gốc đã có bản chỉnh — đường giải phóng khép vòng chỉnh sửa

§8 cấm ghi đè bản gốc nên mỗi lần chỉnh sửa là **thêm** một file — phần phình
ra đó phải có đường thu lại, không thì mục tiêu A và B chống nhau. Liên kết
bản chỉnh ↔ bản gốc vốn đã nằm trong bảng `edits` từ lâu (đủ ba cách nhận
diện: id, tên, MD5) nhưng không màn hình nào dùng nó cả.

Mục `editedOriginal` dựng từ chính bảng đó: mỗi bản gốc một nhóm kiểu chòm
sao, **bản chỉnh là tấm giữ, bản gốc là ứng viên giải phóng**, dung lượng thu
hồi là của bản gốc. Bốn quyết định:

- đứng **sau** các nhóm trùng (bản gốc có bản sao từng byte thì nhóm trùng
  đáng giá hơn — nó được tự tick) nhưng **trước** cụm perceptual — không thì
  cặp gốc↔chỉnh rơi vào "ảnh tương tự", đúng mà vô dụng;
- nhiều bản chỉnh thì ghép với bản **mới nhất** (thứ tự `EditDao.links()`),
  bản cũ đứng ngoài như ảnh thường;
- **không tự tick**: bản chỉnh có thể đã crop mất một phần khung, chỉ người
  dùng biết phần đó có đáng giữ không. Nhưng cũng **không phải xem-lại** —
  giải phóng bản gốc là thu hồi thật, được cộng vào lời hứa;
- áp dụng cho cả video đã cắt/tách — cùng bảng `edits`, không phân biệt.

### 14.6 Nâng cấp màn gợi ý chỉnh sửa: sờ vào ảnh, và chấm lại điểm

Bốn lời chê thật từ chủ dự án, và cách xử lý:

- **"Hai ảnh so sánh quá nhỏ"** → một ảnh to hết bề ngang, **giữ-để-xem bản
  gốc** (nhấp nháy tại chỗ nhạy hơn hẳn hai ảnh cạnh nhau), chạm mở toàn màn
  hình phóng đại 8× vẫn giữ-để-so được. Nhãn Trước/Sau đổi theo trạng thái.
- **"Crop/xoay khó thao tác, không tương tác trên ảnh được"** → khung crop
  trực tiếp (`interactive_crop.dart`): khung đứng yên — ảnh trượt dưới ngón,
  véo để phóng, phần ngoài khung tối đi, thước xoay sát mép khung có hít về
  0°/góc gợi ý kèm rung. Cố ý không dùng cử chỉ xoay hai ngón (kích hoạt nhầm
  khi véo, không chỉnh nổi phần mười độ). Hình học tách thuần
  (`crop_math.dart`, test riêng) — nhờ mô hình xoay→nội-tiếp→crop của
  `EditApplier` nên tầng chạm chỉ thao tác NormRect trong [0,1].
- **"Mẫu sẵn còn ít"** → khung gợi ý nắn theo tỉ lệ 1:1/4:5/16:9 quanh đúng
  tâm chủ thể; tổ hợp "sửa hết lỗi + từng tông màu" — hai thứ vốn không loại
  trừ nhau, bắt chọn một là dải mẫu nói dối về lựa chọn.
- **"Chỉnh xong không biết tốt lên hay xấu đi"** → chấm lại điểm bản đang
  chỉnh (debounce 250 ms), hiện "Kỹ thuật a → b · Thẩm mỹ c → d" xanh/đỏ ngay
  dưới ảnh. Hai ràng buộc: đi qua đúng MỘT lối vào `analyze()`
  (`MemoryMediaSource` bọc bytes thành nguồn, không mở lối tắt từng gây lỗi
  hai-kết-quả), và độ phân giải mang đi chấm là của ảnh THẬT sau crop — cắt
  40% ảnh là mất 40% điểm ảnh, điểm phải nói ra cái giá đó.

Còn lại của mạch này (đã bàn, chưa làm): nhân bản công thức từ ảnh đã chỉnh +
lưu preset đặt tên (EditPlan là dữ liệu thuần, bảng `edits` đã lưu vết; chỉ
chuyển bộ lọc màu, không chuyển crop/xoay); import LUT `.cube` — cửa hợp pháp
duy nhất mang tông màu app khác vào; wipe slider cho chỉnh cùng khung hình và
khung-vẽ-đè cho gợi ý crop.


### 14.7 Presets: dùng lại công thức, LUT ngoài, và bộ tông dựng sẵn

Bốn tầng, từ trong ra ngoài — DB v10 (edits lưu `plan_json`) và v11 (bảng
`presets`):

1. **Công thức gần đây** — mỗi lần Áp dụng, công thức lưu kèm liên kết; ảnh
   sau tự thấy 5 công thức mới nhất trong dải mẫu. Chỉ mang bộ lọc màu sang,
   không mang crop/xoay — khung hình thuộc về từng ảnh.
2. **Preset đặt tên** — nút "Lưu công thức", hiện trong dải mẫu mọi ảnh.
3. **Nhập LUT `.cube`** (`cube_lut.dart`) — cửa hợp pháp duy nhất mang tông
   màu app khác vào: parser + nội suy tam tuyến tính thuần Dart, file chép
   vào thư mục app, tên lấy từ TITLE, lỗi báo đúng bệnh. LUT áp SAU mọi bộ
   lọc; file mất/hỏng thì bỏ qua êm.
4. **Bộ tông dựng sẵn** — 29 preset từ đặc tả `docs/LUT.md` (6 signature +
   23 theo thể loại). Engine tham số kiểu Lightroom rút gọn nằm ở
   `lut_presets.dart` (lib, không phải tool): app dựng bảng 33³ **trong RAM**
   qua `lutPath: "builtin:<id>"` — không đóng 28 MB file vào gói cài;
   `tool/generate_luts.dart` chỉ là vỏ xuất `.cube` để chia sẻ. Nút "Bộ tông
   có sẵn" nạp một lần theo yêu cầu, không tự nạp lúc mở màn (29 preview là
   cái giá ai muốn mới trả).

Kèm trong đợt này: sửa mượt phần chỉnh tay (kéo/véo giữ trạng thái cục bộ,
báo cha một lần lúc thả; render + chấm điểm sang isolate — bài học "closure
inline trong State kéo cả cây widget sang isolate" đã thành hàm static nhận
tham số tường minh; `_EagerScale` thắng ListView trong khung; phóng quanh
điểm đặt ngón `scaledAt`).

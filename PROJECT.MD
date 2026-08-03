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

Chỉ kiểm tra **khi người dùng tự bấm**, không tự chạy nền. Đây là ràng buộc
thiết kế chứ không phải tiết kiệm pin: app không gửi gì ra mạng trong lúc làm
việc với ảnh, và giữ đúng như vậy thì lời hứa "mọi phân tích chạy trên máy bạn"
mới kiểm chứng được bằng cách ngắt mạng rồi dùng thử.

Cơ chế: hỏi GitHub Releases của repo public, so phiên bản **theo từng số** chứ
không so chuỗi (so chuỗi thì `1.10.0` nhỏ hơn `1.9.0`, và người dùng sẽ không
bao giờ được báo có bản mới sau bản 1.9). Có bản mới thì mở link tải trong
trình duyệt — không tự tải và tự cài, nên không cần quyền cài đặt gói, và người
dùng luôn thấy rõ mình đang tải gì.

Repo chưa có bản phát hành nào thì GitHub trả 404; coi đó là "đang dùng bản mới
nhất" chứ không phải lỗi.

## 12. Trạng thái từng nền tảng

| Nền tảng | Đã kiểm tới đâu |
|---|---|
| Android | Chạy và đo trên Pixel 6 Pro thật, mọi tính năng |
| Windows | Dựng và chạy được; đã đo lượt quét trên thư viện 4.000 ảnh |
| iOS | **Chưa dựng thử lần nào** |

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

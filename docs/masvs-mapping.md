# Mapping MASVS Secure Vault

nối mã test R6 theo mục tiêu. Mapping là kế hoạch xác minh: implementation Unknown, test Not run, approval Pending.

## 1 Cách đọc

Chuỗi truy vết: yêu cầu → control → test R6 → actual/evidence → review. Bảng sau có 14 control trong năm nhóm được giao; việc có test liên quan không có nghĩa test bao phủ toàn control. Mã dạng FR-AUTH-01 thuộc namespace R7, không đồng nghĩa FR-01 trong checklist cũ của R6. “Cần bổ sung” là khoảng trống kế hoạch, không là test Fail đã quan sát.

| Control MASVS | Yêu cầu R7 liên quan | Test R6 và giới hạn |
| --- | --- | --- |
| STORAGE-1 | NFR-02/07; FR-CORE-02/03/05; FR-DOC-04; FR-BAK-01/03 | ST-01–06, ST-08; CR-02/06. Áp dụng; file giấy tờ/backup cần case bổ sung. Đọc dữ liệu lưu và metadata, không chỉ DB. |
| STORAGE-2 | NFR-03/05/06/07/11; FR-PWD-09; FR-CORE-06/07 | ST-07/09; PL-01–03/08; PV-01/03/04. Áp dụng; log, UI, clipboard, thông báo và file tạm là các kênh khác nhau. |
| CRYPTO-1 | NFR-09; FR-CORE-01/03/04/05; FR-PWD-10 | CR-01/04/05/06/07/09. Áp dụng; test API/nonce/tag/AAD đúng format. R6 rà status MASTG từng mã trước dùng. |
| CRYPTO-2 | NFR-10/11; FR-CORE-01/02/06; FR-AUTH-06/07/08 | ST-02/03; CR-02/03/05/08; AU-06. Áp dụng; cần bổ sung test đổi password và portable recovery khi có thiết kế. |
| AUTH-1 | NFR-01 | Chưa có case cần chạy; review scope. Đề xuất N/A nếu không remote auth; chưa duyệt. Local auth vẫn thuộc AUTH-2. |
| AUTH-2 | FR-AUTH-01–05/07–09; FR-AF-05/08 | AU-01–08; PL-04. Áp dụng; R6 V1 chưa bao phủ Autofill/race V5; timeout/password còn cần chốt. |
| AUTH-3 | FR-AUTH-06/09; FR-BAK-01/03 | Chưa có case đủ trong checklist R6. Áp dụng, còn gap step-up cho thao tác nhạy cảm. Không coi session mở là xác thực bổ sung. |
| PLATFORM-1 | NFR-08; FR-DOC-02; FR-AF-01–08 | PL-05; AU-02; bổ sung SAF/Autofill. Áp dụng; launcher/service Autofill có thể cần export đúng permission; không đặt luật mọi component đều không export. |
| PLATFORM-2 | FR-DOC-05; NFR-08/19 | Chưa có; cần inventory viewer/WebView. Có điều kiện. R6 nêu không WebView nhưng chưa xem code; chỉ N/A sau xác minh. |
| PLATFORM-3 | NFR-05/06/16; FR-PWD-08/09; FR-DOC-08/09; FR-AF-06 | PL-01–03/08; PV-04. Áp dụng; che mặc định, bảo vệ UI/Recents/clipboard/thông báo và không auto submit. |
| PRIVACY-1 | NFR-01/04/08; FR-DOC-02/03; FR-AF-01/07 | PL-06; PV-02. Áp dụng; minimal permission và inventory dữ liệu, offline chưa đủ chứng minh privacy. |
| PRIVACY-2 | NFR-01/03/04; FR-DOC-04 | PV-02; PV-01 chỉ hỗ trợ chống rò. Áp dụng; cần inventory identifier/tracking. Log sạch một mình không chứng minh unlinkability. |
| PRIVACY-3 | NFR-04/24; FR-AF-01/07; FR-BAK-01 | Chưa có case đủ trong checklist R6. Áp dụng; cần nội dung giải thích thu thập/sử dụng/Autofill/backup tại nơi quyết định. |
| PRIVACY-4 | FR-PWD-04/05; FR-DOC-06/07; FR-AF-01/07; FR-BAK-01/03/04 | PV-03; bổ sung consent/delete/backup. Áp dụng; xóa logic khác secure erase; không hứa thu hồi backup ngoài app. |

## 2 Ngoài năm nhóm và giới hạn

NETWORK-1/2: đề xuất N/A theo offline, chờ xác minh build/data flow. CODE-1–4 và RESILIENCE-1–4 chưa đánh giá đầy đủ trong phạm vi nhiệm vụ này; dependency/input validation/build vẫn cần rà theo yêu cầu dự án. PL-07 là kiểm tra release/debug, không phải bằng chứng đạt toàn RESILIENCE. Không công bố đạt toàn MASVS.

Checklist R6 hiện có 38 mã ST/CR/AU/PL/PV. Dùng làm đầu mối test, không tạo bộ song song. 28 SV-TC v0.1 của R7 chỉ là kịch bản thiết kế tham khảo lưu riêng, được đánh dấu cần đối chiếu/bổ sung vào checklist R6, chưa phải test đã thực hiện. Các phần V3/V5/backup/step-up/privacy notice chưa được checklist V1 bao phủ đủ.

## 3 Các điểm phải review

Ưu tiên D1 thống nhất mã yêu cầu/threat, D2 background/idle/restart, D3 password/delay, D4 file format/viewer, D5 V1 prototype và trạng thái, D6 nguồn MASTG/đường dẫn, D7 step-up/privacy/backup. Bảng đối chiếu chi tiết 38 test do R7 lưu để review; xem danh sách D1–D7 trong [devlog An](devlog-an.md) để chốt các điểm ảnh hưởng trước. Không tự sửa T-06 trong file nhóm hoặc đóng N/A theo giả định.

Test chưa có chức năng ở một mốc nên ghi chưa triển khai/hoãn kiểm tra theo mốc. Chỉ gọi N/A khi thực sự ngoài phạm vi đánh giá đã xác nhận; không dùng N/A cho control cuối cùng vẫn phải đáp ứng. “Dự kiến FAIL” trong tài liệu nguồn không phải actual Fail, và Accepted Risk cần người duyệt/rủi ro/ngày.

## 4 Bằng chứng và nguồn

Để ghi Pass: phải có build/hash, thiết bị/API, bước chạy, expected/actual, bằng chứng dữ liệu giả, ngày và người test; một test Pass không tự đổi cả control sang Pass. R6 thực hiện/review phương pháp, owner sửa, R1 review rủi ro, R7 cập nhật liên kết. Hiện chưa có bằng chứng thực thi hoặc approval.

Nguồn nhóm ghim: main 85f2c17b0bc1653ac9acbf5b60ddd01def8b6629, docs/crypto-design.md và docs/threat-model.md; manh3 92e067a18c66443d40fb4866116f42702c5a782c, docs/secure-testing.md và docs/secure-test-v1.md. Các đường dẫn checklist/templates ghi trong nội dung R6 chưa khớp file thực tế nên cần xác nhận.

Nguồn chuẩn: https://github.com/OWASP/masvs/releases/tag/v2.1.0 ; MASVS commit 8e133d09f4140518ed04cc254b18be9ff4990ffc. MASTG tham khảo kế thừa snapshot 2eb0f7097e5fc702df2f043fb34079fa42956548; không khẳng định test status hôm nay từ snapshot cũ. Trước thực thi R6 xác minh version, deprecated/placeholder và API phù hợp.

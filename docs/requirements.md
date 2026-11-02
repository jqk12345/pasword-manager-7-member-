# Yêu cầu phần mềm Secure Vault

SV-REQ-001 v0.2 • 01/11/2026 • An R7 • BẢN CÔ ĐỌNG CHỜ REVIEW. Phạm vi/74 mã yêu cầu v0.1 được giữ; điều kiện nghiệm thu dưới đây được tóm lược để nhóm đọc nhanh. 222 AC chi tiết v0.1 được R7 lưu làm hồ sơ truy vết riêng, chưa đưa vào PR này; bản nộp tóm lược bên dưới chưa phải toàn bộ AC nguyên văn. Không tự thêm chính sách đã duyệt hoặc kết quả test.

## 1 Mục tiêu và nguyên tắc

Ứng dụng Android Kotlin/Compose quản lý mật khẩu, giấy tờ, backup mã hóa và Autofill cục bộ. Không backend/cloud/telemetry/INTERNET; chỉ dữ liệu giả trong phát triển/demo. Không lưu master password/khóa thô; dữ liệu nhạy cảm được bảo vệ trước ghi đĩa. Không giới thiệu prototype chưa mã hóa là sản phẩm an toàn.

## 2 Chức năng và điều kiện nghiệm thu chính

| Mã giữ nguyên | Yêu cầu | Nghiệm thu tóm lược |
| --- | --- | --- |
| FR-AUTH-01–09 | Tạo/mở/khóa vault; tự khóa, delay thử sai, đổi password; biometric tùy chọn, invalidation và session. | Password đúng mở được; sai/hủy không cấp secret. Khóa chặn cả repository/file/Autofill. Đổi password giữ dữ liệu, mật khẩu cũ mất hiệu lực; lỗi ghi không làm hỏng bộ khóa. Biometric invalid phải fallback an toàn. |
| FR-PWD-01–11 | Thêm/xem/sửa/xóa credential; tìm kiếm/phân loại; che/hiện, clipboard, generator và strength. Trường: dịch vụ, username, password, website/domain, ghi chú. | CRUD đúng, hủy không ghi/xóa nhầm. Mật khẩu che mặc định, copy theo policy và không xóa clipboard mới của người dùng. Generator dùng nguồn ngẫu nhiên an toàn; không coi estimator là bảo đảm mật khẩu an toàn. |
| FR-DOC-01–09 | Giấy tờ giả lập: loại/tên/số/họ tên/ngày cấp/nơi cấp/hết hạn/ghi chú và ảnh/PDF; SAF, validate, nhập/xem/sửa/xóa, che số, nhắc hạn. | Chỉ đọc tệp được chọn và hợp lệ. File lưu mã hóa, tên không lộ định danh, không plaintext temp. Hủy/lỗi không có DB-file nửa hoàn tất. Xóa xử lý file/lịch nhắc; thông báo không lộ dữ liệu. |
| FR-AF-01–08 | Bật/tắt Autofill; nhận diện trường/app demo; xác minh app/cert hoặc website/browser; mở khóa, chọn điền, save có đồng ý và xử lý hủy/race. | Đích không xác minh được hoặc vault khóa thì không cấp credential. Không chọn app theo tên gần giống; không tự submit form. Từ chối lưu không ghi secret; request cũ/đổi đích không được nhận kết quả. |
| FR-BAK-01–05 | Xuất backup mã hóa; parser/header bounds; import xác thực; restore nguyên tử; version và tương thích. Owner/mốc còn cần xác nhận. | Backup hợp lệ khôi phục đúng phạm vi; sai password/tamper/cắt file bị từ chối trước chấp nhận dữ liệu. Hủy/lỗi giữ vault hợp lệ; policy merge/replace rõ. Không tự hứa portable Keystore hoặc xóa bản sao ngoài app. |
| FR-CORE-01–08 | KDF phiên bản, sinh/bọc khóa dữ liệu, AEAD, nonce/AAD, vòng đời secret, lỗi an toàn, format/migration. | Khóa ngẫu nhiên 256 bit; không dùng lại nonce cùng khóa. Sửa ciphertext/tag/AAD phải thất bại, không trả plaintext chưa xác thực. Xóa secret theo khả năng runtime; format/migration và rewrap giữ nhất quán. |

## 3 Ràng buộc phi chức năng

| Mã giữ nguyên | Ràng buộc | Kiểm chứng chính |
| --- | --- | --- |
| NFR-01–04 | Offline; bảo vệ dữ liệu lưu; log/lỗi không chứa secret; tối thiểu hóa quyền/thu thập và chỉ dữ liệu giả. | Rà manifest/dependency; kiểm DB/WAL/file/cache/log với marker giả và luồng lỗi/hủy. |
| NFR-05–08 | UI/Recents, clipboard, backup Android và exported/intent/URI được kiểm soát. | Danh mục màn hình/component có policy; thử lock/back/restart, clipboard mới, URI và quyền không hợp lệ. |
| NFR-09–12 | Mật mã/thư viện có căn cứ; Keystore theo capability; quản lý secret thực tế; công bố giới hạn root/hook. | Review API/format/key lifecycle và thiết bị; không tuyên bố hardware/zeroization/chống root tuyệt đối. |
| NFR-13–16 | Transaction/migration, fail-closed, hiệu năng đo được, UX/accessibility. | Test lỗi/hủy/biên/khôi phục; budget hiệu năng/policy UX phải chốt, không tự đặt con số. |
| NFR-17–20 | Android/browser matrix; phân lớp MVVM; dependency/build kiểm soát; test và truy vết. | UI không gọi DAO; crypto độc lập UI; đúng build/API, phiên bản nguồn và ID req/test được ghi. |
| NFR-21–24 | Bằng chứng trung thực, release gate, tài liệu/phối hợp và thông điệp sản phẩm. | Chỉ ghi Pass khi có actual/evidence; không tag/duyệt thay nhóm, không cam kết đạt toàn chuẩn hoặc giá trị pháp lý. |

## 4 Điều cần xác nhận trước baseline

Mã MK/KEK và VK/DEK đã được mô tả tương ứng trong crypto main; vẫn cần owner review thống nhất implementation/format. Các giá trị dưới đây là thông tin tài liệu nhóm, chưa được R7 tự đóng OQ.

| Chủ đề | Thông tin cần chốt |
| --- | --- |
| Bộ mã yêu cầu và threat | Giữ mã R7 đang có để không đứt truy vết; dùng bảng đối chiếu theo từng test. R6 dùng FR-01/NFR-03 khác nghĩa namespace R7; main có T-01..T-11 nhưng checklist còn trỏ T-12..T-24. |
| Ba tình huống tự khóa | Tách background, idle foreground và process restart; chưa gộp thành một timeout. Crypto main ghi nền 60 giây, idle tối đa 5 phút; threat ghi khóa khi xuống nền hoặc idle 5 phút; R6 dùng mục tiêu 5 phút. |
| Password và delay thử sai | Chính sách cần quyết định trước nghiệm thu; chưa tự áp con số. R6 thử min 12 ký tự; crypto main có dãy 5 lần/30s, 6/1m, 7/5m, 8+/15m; vẫn cần xác nhận password/clock/restart. |
| Mã hóa file và format | Chọn một format/library cùng policy giới hạn/viewer; không tự ghép chunk-GCM mới. Nguồn nhắc Tink và chunk AES-GCM/4KB; chưa đủ format cuối, final chunk/truncation, nonce và API đã xác minh. |
| V1 prototype và trạng thái test | Thiếu control ở V1 ghi chưa triển khai/hoãn mốc, không biến thành không áp dụng toàn sản phẩm. R6 dự kiến V1 plaintext; chưa thấy source/app nên chưa xác nhận. Dự kiến FAIL khác FAIL đã test. Accepted Risk phải có người duyệt. |
| Privacy step up backup | Giữ GAP cần policy; bổ sung case vào checklist R6 khi chức năng có thiết kế. Chưa có test đủ cho AUTH-3, PRIVACY-3; WebView có điều kiện; backup owner/mốc chưa xác nhận. |

## 5 Phạm vi phiên bản và bàn giao

V1 là mốc chức năng theo lộ trình; nếu chưa có crypto chỉ dùng dữ liệu giả và ghi rõ prototype. V2 crypto/session, V3 giấy tờ, V4 hardening, V5 Autofill, V6 đánh giá; thiết kế bảo mật bắt đầu từ G1. Chưa thấy code/build trong các nhánh đã đọc không có nghĩa thành viên chưa có code ở máy riêng.

R7: quản lý yêu cầu/mapping và câu hỏi. R1 review thiết kế/rủi ro, R2 architecture/schema/V1/V4, R3 giấy tờ, R4 Autofill, R5 crypto/Keystore, R6 review/chạy test. Chưa có review/approval hoặc kết quả sản phẩm mới trong bản này.

Nguồn: hồ sơ yêu cầu gốc và v0.1 local; main commit 85f2c17b0bc1653ac9acbf5b60ddd01def8b6629 (crypto/threat); R6 commit 92e067a18c66443d40fb4866116f42702c5a782c (checklist). Nội dung GitHub là thiết kế/kế hoạch, không tự chứng minh triển khai. Danh mục đầy đủ 74 mã và AC giữ trong phụ lục cá nhân; mã trên mỗi dải bao gồm tất cả số liên tiếp. Khi bổ sung/chốt policy phải nâng version và cập nhật test/mapping.

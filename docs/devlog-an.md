# Nhật ký An R7

## 02/11/2026 Chuẩn bị bản nháp G1 để review

- **Đã soạn:** [requirements](requirements.md) và [MASVS mapping](masvs-mapping.md) v0.2, chuyển từ hai bản Word cô đọng của R7. Giữ 74 mã theo dải; 222 AC v0.1 chi tiết và bảng đối chiếu 38 test được lưu riêng, không nằm trong PR này. Chưa coi đây là baseline đã được nhóm duyệt.
- **Đã đối chiếu:** main `84fdbae4c308a4575910f084d47e078062ee502c` (crypto/threat) và nhánh manh3 `92e067a18c66443d40fb4866116f42702c5a782c` (checklist 38 mã ST/CR/AU/PL/PV). Mã yêu cầu R7 và R6 khác namespace nên nối theo nội dung, chưa đổi tham chiếu trong file R6.
- **Kiểm tra tài liệu:** chỉ ba file Markdown R7; bảng/đường dẫn nội bộ hợp lệ, 14 control chính, tham chiếu mã test tồn tại trong snapshot R6. Không chạy build, app, MobSF hoặc test bảo mật.
- **Trạng thái:** Draft for review; implementation Unknown; test Not run; approval Pending. Nội dung có AI hỗ trợ biên tập; An/R7 và người review cần xác nhận trước baseline.

### Điểm cần R1 R6 và owner xác nhận

| Mã | Cần thống nhất | Người review đề xuất |
| --- | --- | --- |
| D1 | Chọn baseline mã FR/NFR và threat. Ví dụ R6 AU-05 gọi brute-force là T-06, nhưng main T-02 là brute-force, T-06 là clipboard/log. Không đổi theo số máy móc. | R1, R6, R7 |
| D2 | Tách tự khóa khi xuống nền, không thao tác foreground và restart. Nguồn đang có nền 60s, idle 5 phút và diễn đạt khóa khi xuống nền. | R1, R2, R5, R6 |
| D3 | Chốt password policy, dãy delay thử sai và cách xử lý clock/restart; các con số trong tài liệu nguồn chưa được R7 tự phê duyệt. | R1, R2, R5 |
| D4 | Chốt format mã hóa file, thư viện, nonce/AAD, kiểm cắt file, giới hạn và viewer. Không tự thiết kế chunk-GCM mới. | R1, R3, R5 |
| D5 | V1 prototype nếu chưa mã hóa chỉ dùng dữ liệu giả. Chưa triển khai ở V1 khác N/A toàn sản phẩm; dự kiến FAIL khác kết quả Fail. Accepted Risk cần người duyệt. | R1, R2, R6 |
| D6 | Rà MASTG status/version trước chạy; xác nhận đường dẫn thật `secure-testing.md`, `secure-test-v1.md`, `vulnerability.md` thay các đường dẫn mẫu chưa có. | R6, R7 |
| D7 | Bổ sung kế hoạch step-up, privacy notice/consent/delete và backup owner/mốc; WebView cần inventory trước kết luận N/A. | R1, R6 và owner |

### Bước tiếp theo

Thu review D1–D7, cập nhật yêu cầu/mapping và tham chiếu test theo quyết định. Không đổi trạng thái sang Approved/Pass chỉ vì tài liệu được đưa lên GitHub. Chưa chỉnh README, devlog thành viên khác hoặc tài liệu của R1/R5/R6.

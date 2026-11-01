# Kiểm thử bảo mật — Secure Vault Android (V1 / v0.1)

| Mục | Nội dung |
|---|---|
| Trạng thái | **Bản nháp v0.1, 31/10/2026. Chưa có test nào được thực hiện; không có nội dung nào trong tài liệu này là kết quả test.** |
| Người phụ trách / review | R6 (Mạnh3) soạn và chạy; R1 (Mạnh4) review |
| Tài liệu đi kèm | `docs/checklists/security-testing-v1.md` (test case), `docs/templates/vulnerability-report-template.md` |
| Cơ sở | `requirements.md`, `architecture.md`, `database-schema.md`, `crypto-design.md`, `threat-model.md`, `masvs-mapping.md` (bản nháp G1) và lộ trình advisor. **Tác giả không có mã nguồn**; tình trạng triển khai dưới đây lấy từ lộ trình và người test phải xác nhận lại. |

> Bản tiếng Việt dịch từ bản tiếng Anh; thuật ngữ kỹ thuật, ID, lệnh giữ nguyên. Khi hai bản khác nhau, bản tiếng Anh là bản gốc.

## 1. Giới thiệu

**Mục đích.** Xác định cách kiểm thử bảo mật bản dựng đầu tiên (v0.1) của Secure Vault, bằng chứng cần thu thập, và cách ghi nhận kết quả cùng finding.

**Tổng quan dự án.** Ứng dụng quản lý mật khẩu Android hoạt động offline (Kotlin, Compose, MVVM, Room). Theo lộ trình advisor, **v0.1 là vault cơ bản với dữ liệu chưa mã hóa**: màn hình master password (tạm), CRUD credential, tìm kiếm, ẩn/hiện, copy, unit test cho repository. Mã hóa, Keystore, sinh trắc học và lockout đến ở V2; vault giấy tờ ở V3; hardening (secure flag, auto-lock, xóa clipboard, tắt backup) ở V4; Autofill ở V5.

**Mục tiêu.** (1) Thiết lập baseline "trước" có tài liệu cho bản chưa mã hóa để chứng minh cải tiến của V2 bằng bằng chứng. (2) Kiểm chứng những gì dự kiến đã đúng ở v0.1 (không có bí mật trong log/prefs/bộ nhớ ngoài, không hard-code khóa, quyền tối thiểu, không có component bị export, logic đăng nhập đúng). (3) Chuẩn bị quy trình lặp lại được, dùng lại ở các phiên bản sau và đợt đánh giá V6.

**Phạm vi.** Trong phạm vi: các nhóm MASVS STORAGE, CRYPTO, AUTH, PLATFORM, PRIVACY ở mức có thể thực hiện ở v0.1 (xem §3). Ngoài phạm vi: Autofill, vault giấy tờ, nhắc hạn bằng WorkManager, mạng (không có `INTERNET`), máy root và khả năng chống dịch ngược, pentest chính thức, quét MobSF (dự kiến V6).

**Giới hạn.** v0.1 lưu dữ liệu chưa mã hóa theo thiết kế, nên nhiều test *dự kiến FAIL* và được xếp loại Baseline. Qua được checklist này không có nghĩa ứng dụng an toàn.

## 2. Chuẩn và tài liệu tham chiếu

Đã xác minh ngày **31/10/2026** từ nguồn OWASP chính thức:

| Mục | Tình trạng đã xác minh | Nguồn |
|---|---|---|
| OWASP MASVS | **v2.1.0**, được đánh dấu "Latest" trên trang release (ngày hiển thị là 18/1; trang không ghi năm, nguồn thứ cấp ghi tháng 1/2024). Bổ sung MASVS-PRIVACY. v2 không còn mức L1/L2/R | github.com/OWASP/masvs/releases; mas.owasp.org/MASVS/ |
| OWASP MASTG | **v2.0.0**, bản v2 ổn định đầu tiên (bao gồm tháng 1 đến tháng 6/2026) | github.com/OWASP/owasp-mastg/releases/tag/v2.0.0 |
| MAS Checklists | `mas.owasp.org/checklists/` hiện chuyển hướng tới bài tin "MASTG v2.0.0 Removes MAS Checklists" (đường dẫn đề ngày 2026/07/14). Chưa đọc nội dung bài. **Vì vậy dự án dùng checklist riêng, ánh xạ tới control MASVS và ID test MASTG v2.** | mas.owasp.org/news/2026/07/14/checklists-removal/ |
| ID test MASTG | Lấy từ chỉ mục MASTG v2 trên mas.owasp.org cùng ngày. Chỉ trích dẫn ID nếu nó xuất hiện ở đó; nếu không có test phù hợp, checklist ghi rõ và nêu weakness MASWE liên quan | mas.owasp.org/MASVS/ (mục điều hướng) |

**MASVS và MASTG.** MASVS là chuẩn xác minh (điều gì phải đúng). MASTG là phương pháp kiểm thử (kiểm tra thế nào). Một test MASTG hỗ trợ một control MASVS; qua một test không có nghĩa control đã được xác minh.

Nhóm MASVS liên quan: STORAGE (1, 2), CRYPTO (1, 2), AUTH (1, 2, 3), PLATFORM (1, 3), PRIVACY (1, 2, 4). Không dùng: NETWORK (không có mạng), PLATFORM-2 (không có WebView), RESILIENCE (không tuyên bố đảm bảo). Một kiểm tra (PL-07, cờ debuggable) được MASTG xếp vào MASVS-RESILIENCE và chỉ dùng làm kiểm tra vệ sinh build.

**Chênh lệch ngày cần lưu ý.** `masvs-mapping.md` của G1 ghi lần kiểm tra ngày 29/10/2026; tài liệu này kiểm tra lại ngày 31/10/2026 với cùng kết quả.

## 3. Phạm vi kiểm thử bảo mật

Mỗi mục dưới đây được chi tiết hóa thành test case (ID, quy trình, kết quả mong đợi, bằng chứng, mapping) trong `checklists/security-testing-v1.md`.

### 3.1 Bảo mật lưu trữ (MASVS-STORAGE)

Mục tiêu: dữ liệu nhạy cảm không bị để ở nơi kẻ tấn công hoặc app khác đọc được. Mối đe dọa: T-01, T-10, T-11, T-12. Test: ST-01 đến ST-09 (nội dung database, cách lưu master password, SharedPreferences/DataStore, file thừa, bộ nhớ ngoài, backup, log, cache, keyboard cache). Dự kiến ở v0.1: ST-01, ST-02, ST-06 FAIL ở mức baseline; các test còn lại nên pass.

### 3.2 Mật mã (MASVS-CRYPTO)

Mục tiêu: mật mã khi được thêm vào là đúng, và v0.1 không đưa vào vật liệu mật mã yếu hay hard-code. Mối đe dọa: T-02, T-03, T-17. Test: CR-03, CR-07, CR-09 chạy được ngay (khóa hard-code, cách dùng thuật toán, nguồn ngẫu nhiên). CR-01, CR-02, CR-04, CR-05, CR-06, CR-08 (AES-GCM, hệ khóa VK/MK, nonce, Argon2id, chống sửa dữ liệu, Keystore) là NOT APPLICABLE đến V2 và được viết sẵn để V2 có test sẵn sàng.

### 3.3 Xác thực (MASVS-AUTH)

Mục tiêu: không truy cập được vault nếu thiếu master password và trạng thái xác thực theo đúng vòng đời. Mối đe dọa: T-05, T-06, T-13, T-15, T-22. Test: AU-01, AU-02, AU-07, AU-08 ngay bây giờ; AU-03 nếu có thao tác khóa; AU-04 baseline; AU-05 (chống brute-force) và AU-06 (sinh trắc học) N/A đến V2.

### 3.4 Bảo mật nền tảng (MASVS-PLATFORM)

Mục tiêu: app khác và hệ điều hành không lấy được bí mật qua chụp UI, clipboard, intent hay cấu hình sai. Mối đe dọa: T-08, T-09, T-13, T-24. Test: PL-05, PL-06, PL-07, PL-08 chạy được ngay; PL-01 (chụp màn hình), PL-02 (recents), PL-03 (clipboard), PL-04 (auto-lock) là baseline cho V4.

### 3.5 Quyền riêng tư (MASVS-PRIVACY)

Mục tiêu: không thu thập thừa, không rò rỉ qua chẩn đoán, và xóa là xóa thật. Mối đe dọa: T-10, T-17, T-19, T-23. Test: PV-01, PV-02, PV-04 chạy được; PV-03 (phần dư sau khi xóa) là baseline.

### 3.6 Truy vết: control MASVS - yêu cầu - test

| Control MASVS | Yêu cầu (G1) | Test |
|---|---|---|
| STORAGE-1 | FR-07, FR-13, NFR-03 | ST-01, ST-02, ST-03, ST-04, ST-05, ST-08, CR-02, CR-06 |
| STORAGE-2 | NFR-02, NFR-05, FR-10 | ST-06, ST-07, ST-09, PV-01, PV-03 |
| CRYPTO-1 | NFR-03, NFR-04, NFR-07 | CR-01, CR-04, CR-05, CR-06, CR-07, CR-09 |
| CRYPTO-2 | NFR-06, FR-04, FR-05 | CR-02, CR-03, CR-05, CR-08, ST-02 |
| AUTH-2 | FR-02, FR-03, FR-04, FR-06, FR-12, FR-25 | AU-01 đến AU-08, PL-04 |
| PLATFORM-1 | NFR-10, NFR-11 | PL-05, AU-02 |
| PLATFORM-3 | FR-09, FR-10, FR-16, FR-24 | PL-01, PL-02, PL-03, PL-08, PV-04 |
| PRIVACY-1 | NFR-01, NFR-12 | PL-06, PV-02 |
| PRIVACY-2 | NFR-12 | PV-01, PV-02 |
| PRIVACY-4 | FR-18, FR-07 | PV-03 |
| AUTH-1, AUTH-3 | FR-02, FR-05 | Không test ở v0.1 (AUTH-1 ít liên quan vì không có xác thực từ xa; AUTH-3 step-up đến cùng với đổi mật khẩu và đăng ký sinh trắc học ở V2) |

## 4. Môi trường kiểm thử

| Hạng mục | Bắt buộc | Ghi chú |
|---|---|---|
| Android Studio và ADB | **Bắt buộc** | Build variant, `run-as`, `logcat`, `dumpsys`, `am` |
| Emulator Medium Phone API 37 (Android 17), x86_64 | **Bắt buộc** | Nhóm đã tạo (Device Manager); cần Android 13+ cho hành vi cờ nhạy cảm của clipboard |
| Điện thoại Android thật | Tùy chọn cho v0.1 | Cần từ V2 để đo thời gian Argon2id và sinh trắc học thật |
| Build variant | **Bắt buộc** | Debug để kiểm tra lưu trữ; release (unsigned hoặc ký debug) cho PL-07 |
| Dữ liệu test | **Bắt buộc** | Marker giả định nghĩa trong checklist; không bao giờ dùng dữ liệu thật |
| DB Browser for SQLite hoặc `sqlite3` | **Bắt buộc** (một trong hai) | Xem DB đã sao chép |
| `apkanalyzer` / APK Analyzer, `jadx` | Tùy chọn | Manifest đã merge và chuỗi; Android Studio có APK Analyzer |
| MobSF, Frida, semgrep | Tùy chọn / sau này | Dự kiến V6; không cần cho v0.1 |
| Ghi chú hệ điều hành máy host | — | Nhóm dùng Windows. Trong Windows PowerShell 5.1, dùng `cmd /c "adb exec-out ... > file"` cho đầu ra nhị phân |

## 5. Phương pháp kiểm thử

1. **Chuẩn bị:** build bản debug và release, ghi commit hash, thiết bị, API level, cài mới; tạo thư mục bằng chứng `evidence/<Test ID>/`.
2. **Phân tích tĩnh:** xem manifest đã merge, tìm trong source các chỗ ghi log, API crypto, API random, bí mật hard-code, kiểu nhập, `FLAG_SECURE`; danh sách dependency.
3. **Phân tích động:** chạy các luồng bằng dữ liệu giả trên emulator trong khi bắt Logcat.
4. **Kiểm tra dữ liệu lưu trữ:** sao chép DB/WAL/SHM và thư mục app qua `run-as`; tìm marker; so sánh snapshot file.
5. **Kiểm thử xác thực:** mật khẩu sai/biên, bypass bằng điều hướng, intent, process death, chuyển nền.
6. **Kiểm thử mật mã:** v0.1 giới hạn ở kiểm tra tĩnh; unit và integration test từ V2.
7. **Kiểm thử bảo mật nền tảng:** chụp màn hình, recents, clipboard, component bị export, quyền, cờ debuggable.
8. **Thu thập bằng chứng:** ảnh chụp, đầu ra lệnh, `log.txt`, kết quả truy vấn DB, ghi chú ngắn; mỗi mục gắn Test ID, ngày, commit.
9. **Xác minh kết quả:** một thành viên khác kiểm tra lại mọi FAIL và một mẫu các kết quả PASS dựa trên bằng chứng trước khi chốt trạng thái.

## 6. Phân loại kết quả test

| Trạng thái | Dùng khi |
|---|---|
| PASS | Đã thực hiện quy trình, quan sát đúng kết quả mong đợi, đã lưu bằng chứng |
| FAIL | Đã thực hiện quy trình, không quan sát được kết quả mong đợi; mở finding |
| NOT TESTED | Có áp dụng nhưng chưa thực hiện |
| NOT APPLICABLE | Chức năng hoặc control không tồn tại ở phiên bản này (ghi lý do và phiên bản chạy lại) |
| BLOCKED | Không chạy được vì trở ngại bên ngoài (công cụ, thiết bị, thiếu phụ thuộc); ghi lý do |

Quy tắc: **chưa test không phải PASS**; **chưa triển khai không phải PASS**; test chỉ PASS khi có bằng chứng phù hợp. Trạng thái thiết kế được theo dõi riêng:

| Mức | Định nghĩa | Bằng chứng |
|---|---|---|
| Designed | Được mô tả trong tài liệu G1 | Tài liệu |
| Implemented | Có trong code của một commit nhất định | Tham chiếu code |
| Tested | Đã thực hiện một test case | Kết quả đã ghi |
| Verified | Đã test, bằng chứng đã được review, và control đáp ứng control MASVS trong phạm vi nêu | Bằng chứng đã review |

**Trạng thái control ở v0.1 (cần xác nhận với code):** mọi control bảo mật trong `crypto-design.md` và V4 mới ở mức *Designed*, theo lộ trình thì chưa *Implemented* ở v0.1, chưa *Tested*, chưa *Verified*.

## 7. Security findings

Dùng `templates/vulnerability-report-template.md`. Định dạng ID `SV-F-nnn`. Mỗi finding một nguyên nhân gốc; liên kết mọi test FAIL với một finding. Mức độ: Critical, High, Medium, Low, Informational, được giải thích bằng tác động và khả năng khai thác, ghi chú thêm confidentiality, integrity, availability; có thể thêm CVSS v4.0 kèm vector đầy đủ (không bao giờ tự gán CVE). Luồng trạng thái: Open, In Progress, Fixed, Retest Required, Closed, Accepted Risk. **Finding Baseline** (control được lên lịch cho phiên bản sau) được ghi với trạng thái *Accepted Risk (v0.1 baseline)*, kèm phiên bản đích và bắt buộc retest ở phiên bản đó. Credential thật không bao giờ được xuất hiện trong bằng chứng.

Ưu tiên: P0 (vault, master password, khóa, credential plaintext), P1 (xác thực, quản lý khóa, backup, rò rỉ, vòng đời), P2 (tăng cường quyền riêng tư, bề mặt thứ cấp). Số lượng: 38 test, P0 10, P1 21, P2 7.

## 8. Giới hạn và rủi ro còn lại (v0.1)

- Dữ liệu và (có thể cả) master password được lưu chưa mã hóa; ai có file là đọc được. Điều này chỉ được chấp nhận **vì** v0.1 dùng dữ liệu giả và không phát hành như một password manager.
- Chưa có auto-lock, secure flag, xóa clipboard hay loại trừ backup đến V4; chưa có lockout hay sinh trắc học đến V2.
- Kiểm tra tĩnh phụ thuộc vào việc người test đọc code thực tế; kế hoạch này được viết khi chưa có code.
- Emulator không tái hiện mọi hành vi phần cứng (Keystore, sinh trắc học); kiểm thử trên thiết bị thật bắt đầu từ V2.
- Tấn công trên máy root, kiểm tra bộ nhớ và dịch ngược nằm ngoài kiểm thử v0.1 (xem `threat-model.md` T-04, T-20).
- Qua checklist không phải bằng chứng của an toàn, và một test không chứng minh được khả năng chống mọi cách tấn công.

## 9. Tài liệu tham khảo

OWASP MASVS (mas.owasp.org/MASVS/) và các bản phát hành (github.com/OWASP/masvs/releases), truy cập 31/10/2026; ghi chú phát hành OWASP MASTG v2.0.0 (github.com/OWASP/owasp-mastg/releases/tag/v2.0.0); bài tin MAS về việc gỡ checklist (mas.owasp.org/news/2026/07/14/checklists-removal/); các weakness MASWE được tham chiếu theo ID trong checklist; các tài liệu dự án nêu ở phần đầu. Tài liệu Android developer (backup, Keystore, BiometricPrompt, công cụ `dumpsys`/`am`) cần được người test trích dẫn kèm URL chính xác khi quy trình dựa vào chúng.

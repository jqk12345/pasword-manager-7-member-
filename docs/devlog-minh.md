# 📝 DevLog - Nhật Ký Phát Triển (Minh R5 - Crypto & Keystore)

---

### [2026-10-29] Tìm hiểu kiến trúc phân cấp khóa
- **Đã làm:** Khảo sát nguyên lý phân tách Master Key (KEK) và Vault Key (DEK), cơ chế Double Key Wrapping (Password + Keystore).
- **Khái niệm then chốt:** Key separation, KEK wrap DEK, Biometric Keystore wrap DEK, Zeroization.
- **Kế hoạch tiếp theo:** Phân tích 3 ranh giới bảo mật (Security Boundaries) và chuẩn bị dựng sơ đồ.

### [2026-10-30] Thiết kế ranh giới bảo mật & khung sơ đồ kiến trúc
- **Đã làm:** Xác định 3 phân vùng bảo mật: Untrusted Zone (Storage/Input), In-Memory Safe Zone (RAM), và Hardware Security Zone (Android Keystore / TEE).
- **Khái niệm then chốt:** Security Boundaries, BiometricPrompt CryptoObject, Memory Zeroization, Window of Vulnerability.
- **Kế hoạch tiếp theo:** Dựng sơ đồ phân tầng trên draw.io và khảo sát thư viện KDF argon2kt.

### [2026-10-31] Hoàn thành sơ đồ kiến trúc & khảo sát KDF argon2kt
- **Đã làm:** Vẽ và xuất sơ đồ kiến trúc 3 vùng bảo mật (`architecture.drawio`, `architecture.png`). Khảo sát thư viện `lambdapioneer/argon2kt` (v1.6.0), đề xuất tham số KDF theo RFC 9106, soạn thảo bản nháp `crypto-design.md`.
- **Khái niệm then chốt:** Argon2id (Data-dependent vs Data-independent memory access, chống Side-channel và GPU/ASIC), Key Wrapping AES-256-GCM, Memory dump attack.
- **Kế hoạch tiếp theo:** Phối hợp Trưởng nhóm Mạnh4 (R1) duyệt thiết kế G1; chuẩn bị bước vào giai đoạn V2 (Argon2kt implementation & benchmark).

### [2026-11-01] Duyệt thiết kế G1 & Đồng bộ hóa cùng R1 Mạnh4
- **Đã làm:** Thống nhất sơ đồ phân cấp khóa và tham số KDF Argon2id với Trưởng nhóm Mạnh4 (R1). Kiểm tra tài liệu tích hợp trên main và chuẩn bị bước vào giai đoạn V1.
- **Khái niệm then chốt:** Key hierarchy synchronization, Dual wrapping review, JNI boundary.
- **Kế hoạch tiếp theo:** Hỗ trợ Nam (R2) cấu hình metadata schema ở V1; nghiên cứu sâu cấu trúc thư viện argon2kt cho V2.

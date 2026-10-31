# 📝 DevLog - Nhật Ký Phát Triển / Development Log (Mạnh4 - R1 Security Architect)

---

### [2026-10-29] Phân tích tài sản, bề mặt tấn công & mô hình mối đe dọa (Threat Model)
- **Đã làm / Completed:** Xác định danh mục Assets (`A-01` đến `A-07`), phân loại 4 nhóm Attackers (`ATK-1` đến `ATK-4`), phân tích Attack Surface và 5 ranh giới tin cậy (Trust Boundaries `TB-01` đến `TB-05`).
- **Khái niệm then chốt / Key Concepts:** Threat Modeling (STRIDE), Trust Boundaries, Residual Risks trên thiết bị đã bị root.
- **Kế hoạch tiếp theo / Next Steps:** Đối chiếu thiết kế với OWASP Mobile Top 10 (2024) và OWASP MASVS.

### [2026-10-30] Thiết kế kiến trúc mật mã & quy tắc AEAD (Crypto Design)
- **Đã làm / Completed:** Thiết kế sơ đồ phân cấp khóa (`Master Password` -> `Argon2id` -> `Master Key` + `Android Keystore` -> bọc `Vault Key` -> `AES-256-GCM`). Quy định quản lý Nonce 12 byte và cấu trúc Associated Data (`AAD = <record_type>:<record_id>:<field_name>:v1`) chống tráo đổi bản ghi.
- **Khái niệm then chốt / Key Concepts:** Key Separation, Double Key Wrapping, AES-256-GCM Context Binding (AAD), $O(1)$ Master Password Rotation (`Rewrap VK`).
- **Kế hoạch tiếp theo / Next Steps:** Bổ sung xử lý lỗi, chính sách Lockout lũy tiến và hoàn thiện bản song ngữ Việt - Anh.

### [2026-10-31] Hoàn thiện `docs/threat-model.md` & `docs/crypto-design.md` (Song ngữ VI/EN)
- **Đã làm / Completed:** Hoàn thành `docs/threat-model.md` (gồm 11 kịch bản tấn công `T-01`..`T-11` và bảng chốt chặn đầy đủ `M1`..`M10` của OWASP Mobile Top 10 2024) và `docs/crypto-design.md` (song ngữ Việt - Anh).
- **Khái niệm then chốt / Key Concepts:** OWASP Mobile Top 10 (2024), `KeyPermanentlyInvalidatedException`, Memory Zeroization (`fill(0)`).
- **Kế hoạch tiếp theo / Next Steps:** Họp duyệt thiết kế mốc G1 ngày 01/11 và hỗ trợ nhóm bước sang giai đoạn V1.

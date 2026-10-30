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

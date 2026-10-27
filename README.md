# pasword-manager-7-member
SECURE VAULT · LUỒNG DỮ LIỆU
│
├── [UNTRUSTED ZONE] Ngoài app + ổ đĩa
│   ├── Master Password ── gõ từ bàn phím ──▶ CharArray (CẤM String)
│   ├── SAF: content://… (CCCD, hộ chiếu, bằng lái) ──▶ InputStream
│   ├── AutofillService nhận FillRequest (packageName, webDomain)
│   ├── Ổ đĩa (chỉ ciphertext):
│   │   ├── vault_meta      : salt 16B · KdfParams · wrappedDEK · wrappedDEK_bio
│   │   ├── Room            : EncryptedEntryRecord(nonce 12B ‖ ciphertext ‖ tag 16B)
│   │   ├── filesDir/docs/  : <uuid>.bin (mã hoá theo khối)
│   │   └── Backup .svbk    : header(magic, version, KdfParams) làm AAD + payload
│   └── CẤM: clipboard không tự xoá · screenshot · Recents · Log · allowBackup
│
├── [IN-MEMORY SAFE ZONE] RAM của process, chỉ khi Unlocked
│   ├── M1  Argon2id(password, salt, m=64 MiB, t=3, p=1) ──▶ KEK 32B   (benchmark lại trên máy thật)
│   ├── M1  AES-256-GCM.open(wrappedDEK, KEK) ──▶ DEK 32B ; KEK.fill(0)
│   ├── M3  VaultSession giữ DEK
│   │   ├── lock khi: người dùng bấm · xuống nền 60s · tắt màn hình
│   │   └── lock = DEK.fill(0) + xoá state UI (best-effort, JVM không bảo đảm 100%)
│   ├── M4  VaultRepository: seal/open từng entry, AAD = entryId
│   ├── M5  DocumentStore: stream mã hoá, không bao giờ ghi file tạm bản rõ
│   └── M8  CredentialMatcher: khớp packageName + SHA-256 chứng chỉ ký / webDomain
│
└── [HARDWARE SECURITY] Android Keystore (TEE / StrongBox)
    ├── M2  vault_wrap_key : AES-256-GCM, non-exportable
    ├── setUserAuthenticationRequired(true) + setInvalidatedByBiometricEnrollment(true)
    └── BiometricPrompt(CryptoObject(cipher)) ──▶ giải bọc wrappedDEK_bio ──▶ DEK vào M3

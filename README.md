# 🔐 Secure Vault

**Secure Vault** là ứng dụng Android được thiết kế để lưu trữ an toàn các tài liệu nhạy cảm và thông tin đăng nhập. Ứng dụng sử dụng cơ chế dẫn xuất khóa từ mật khẩu, mã hóa có xác thực, Android Keystore và xác thực sinh trắc học để bảo vệ dữ liệu người dùng.

Kiến trúc bảo mật được chia thành ba khu vực chính: **Vùng không đáng tin cậy (Untrusted Zone)**, **Vùng an toàn trong bộ nhớ (In-Memory Safe Zone)** và **Vùng bảo mật phần cứng (Hardware Security)**.

## 1. Kiến trúc bảo mật

### 1.1. Vùng không đáng tin cậy (Untrusted Zone)

Khu vực này bao gồm dữ liệu đầu vào từ bên ngoài ứng dụng và dữ liệu được lưu trữ lâu dài trên thiết bị. Tất cả dữ liệu đi vào ứng dụng đều phải được xử lý cẩn thận.

- **Mật khẩu chính (Master Password):** Được nhập từ bàn phím và lưu trong `CharArray` thay vì `String` nhằm hạn chế thời gian tồn tại của dữ liệu nhạy cảm trong bộ nhớ.
- **Nhập tài liệu:** Các tài liệu nhạy cảm như căn cước công dân, hộ chiếu và bằng lái xe được nhập thông qua Storage Access Framework (SAF) của Android, sử dụng URI dạng `content://` và `InputStream`.
- **Yêu cầu Autofill:** `AutofillService` nhận thông tin từ `FillRequest`, bao gồm tên gói ứng dụng và tên miền website, để xác định ứng dụng hoặc trang web đang yêu cầu thông tin đăng nhập.
- **Lưu trữ dữ liệu:** Chỉ lưu dữ liệu đã được mã hóa xuống ổ đĩa.

Các vị trí lưu trữ được sử dụng trong kiến trúc gồm:

| Vị trí lưu trữ | Nội dung |
|---|---|
| `vault_meta` | Salt 16 byte, tham số KDF, `wrappedDEK` và `wrappedDEK_bio` |
| Cơ sở dữ liệu Room | Các bản ghi dữ liệu đã mã hóa |
| `filesDir/docs/` | Tài liệu đã mã hóa, lưu dưới dạng `<uuid>.bin` |
| Bản sao lưu `.svbk` | Phần header và dữ liệu được mã hóa |

Ứng dụng cần ngăn thông tin nhạy cảm xuất hiện trong nhật ký hệ thống, ảnh chụp màn hình, màn hình Recents và các bản sao lưu ứng dụng không được bảo vệ. Dữ liệu trong clipboard cũng không được để tồn tại vô thời hạn.

### 1.2. Vùng an toàn trong bộ nhớ (In-Memory Safe Zone)

Các thao tác mã hóa và giải mã nhạy cảm được thực hiện trong bộ nhớ của tiến trình ứng dụng khi kho dữ liệu đang ở trạng thái mở khóa.

#### Dẫn xuất khóa và mở khóa

Thuật toán Argon2id được sử dụng để tạo khóa mã hóa KEK dài 32 byte từ mật khẩu chính và một salt ngẫu nhiên.

Các tham số ban đầu:

- **Thuật toán:** Argon2id
- **Dung lượng bộ nhớ:** 64 MiB
- **Số vòng lặp:** 3
- **Mức độ song song:** 1
- **Độ dài khóa đầu ra:** 32 byte

KEK được dùng để giải bọc khóa mã hóa dữ liệu DEK thông qua AES-256-GCM. Sau khi khôi phục DEK, ứng dụng xóa mảng KEK bằng `KEK.fill(0)` theo cơ chế xử lý bộ nhớ tốt nhất có thể.

Các tham số Argon2id cần được kiểm tra hiệu năng trên thiết bị Android thực tế trước khi chốt cấu hình chính thức.

#### Quản lý phiên Vault

`VaultSession` quản lý DEK trong thời gian kho dữ liệu được mở khóa.

Vault sẽ chuyển sang trạng thái khóa khi:

- Người dùng chủ động khóa Vault.
- Ứng dụng chạy ở chế độ nền trong 60 giây.
- Màn hình thiết bị tắt.

Khi khóa, ứng dụng xóa DEK bằng `DEK.fill(0)` và loại bỏ trạng thái giao diện người dùng có chứa thông tin nhạy cảm.

**Lưu ý bảo mật:** Việc ghi đè mảng trong môi trường quản lý bộ nhớ là một biện pháp giảm thiểu rủi ro, không phải bảo đảm tuyệt đối. JVM và Android Runtime không bảo đảm mọi bản sao của dữ liệu nhạy cảm sẽ được xóa hoàn toàn ngay lập tức khỏi bộ nhớ.

### 1.3. Vùng bảo mật phần cứng (Hardware Security)

Android Keystore cung cấp khả năng bảo vệ khóa, bao gồm hỗ trợ phần cứng khi thiết bị đáp ứng yêu cầu.

Kiến trúc sử dụng khóa không thể xuất trực tiếp có tên `vault_wrap_key`, áp dụng AES-256-GCM để bảo vệ các thao tác bọc khóa.

Các cấu hình bảo mật liên quan gồm:

- `setUserAuthenticationRequired(true)`
- `setInvalidatedByBiometricEnrollment(true)`

Để mở khóa bằng sinh trắc học, `BiometricPrompt` sử dụng một `CryptoObject` chứa đối tượng mã hóa. Sau khi xác thực thành công, đối tượng này được sử dụng để giải bọc `wrappedDEK_bio`, cho phép khôi phục DEK vào `VaultSession` đang hoạt động.

Khả năng bảo vệ bằng phần cứng phụ thuộc vào thiết bị. Việc sử dụng TEE hoặc StrongBox cần được kiểm tra trên thiết bị mục tiêu.

## 2. Các thành phần chính

| Thành phần | Chức năng |
|---|---|
| **M1 — Quản lý khóa** | Dẫn xuất KEK bằng Argon2id, giải bọc DEK và xóa KEK sau khi sử dụng. |
| **M2 — Bảo vệ khóa bằng phần cứng** | Sử dụng Android Keystore để bảo vệ các thao tác bọc khóa và hỗ trợ mở khóa bằng sinh trắc học. |
| **M3 — VaultSession** | Quản lý DEK đang hoạt động và trạng thái phiên đã mở khóa. |
| **M4 — VaultRepository** | Mã hóa và giải mã từng mục dữ liệu trong Vault bằng AES-256-GCM. |
| **M5 — DocumentStore** | Mã hóa tài liệu theo luồng dữ liệu và tránh tạo tệp tạm chứa nội dung chưa mã hóa. |
| **M8 — CredentialMatcher** | Đối chiếu ứng dụng Android bằng tên gói và dấu vân tay SHA-256 của chứng chỉ ký; hoặc đối chiếu website bằng tên miền. |

## 3. Cơ chế mã hóa và định dạng dữ liệu

### Dữ liệu trong Vault

Mỗi bản ghi trong cơ sở dữ liệu được lưu dưới dạng dữ liệu đã mã hóa:

`nonce (12 byte) || ciphertext || authentication tag (16 byte)`

AES-256-GCM cung cấp tính bảo mật và tính toàn vẹn cho dữ liệu. ID của bản ghi (`entryId`) được sử dụng làm Dữ liệu bổ sung có xác thực (Additional Authenticated Data — AAD), giúp ràng buộc bản ghi đã mã hóa với đúng mục dữ liệu của nó.

### Lưu trữ tài liệu

Tài liệu được mã hóa theo luồng dữ liệu và lưu tại:

`filesDir/docs/<uuid>.bin`

Thiết kế này tránh việc ghi các bản sao tạm thời chưa mã hóa của tài liệu xuống ổ đĩa. Định dạng mã hóa tài liệu cần xác định rõ cấu trúc từng khối, cách quản lý nonce, thẻ xác thực và quy trình kiểm tra tính toàn vẹn.

### Metadata của Vault

Dữ liệu `vault_meta` bao gồm:

- Salt ngẫu nhiên dài 16 byte.
- Các tham số dẫn xuất khóa.
- `wrappedDEK`.
- `wrappedDEK_bio`.

DEK được bảo vệ bằng cơ chế bọc khóa, thay vì lưu trực tiếp dưới dạng văn bản thuần.

### Sao lưu đã mã hóa

Tệp sao lưu sử dụng phần mở rộng `.svbk`.

Cấu trúc gồm:

- **Header:** Giá trị magic, phiên bản định dạng và các tham số KDF.
- **Payload:** Dữ liệu sao lưu đã mã hóa.

Header được xác thực dưới dạng Dữ liệu bổ sung có xác thực (AAD) nhằm phát hiện các thay đổi trái phép đối với những trường được xác thực trong header.

Phần triển khai sao lưu cần kiểm tra phiên bản định dạng, từ chối dữ liệu đầu vào không hợp lệ và xác minh tính xác thực trước khi chấp nhận dữ liệu khôi phục.

## 4. Các nguyên tắc bảo mật

Secure Vault được thiết kế dựa trên những nguyên tắc sau:

- **Bảo vệ dữ liệu lưu trữ:** Ưu tiên lưu các mục Vault và tài liệu ở dạng mã hóa thay vì văn bản thuần.
- **Mã hóa có xác thực:** Sử dụng AES-256-GCM để bảo vệ tính bí mật và phát hiện các thay đổi trái phép.
- **Bảo vệ bằng mật khẩu:** Dẫn xuất KEK bằng Argon2id, sử dụng salt ngẫu nhiên và các tham số chi phí có thể cấu hình.
- **Phân tách khóa:** Sử dụng KEK để bảo vệ DEK thay vì mã hóa trực tiếp từng mục dữ liệu bằng mật khẩu chính.
- **Bảo vệ bằng phần cứng:** Sử dụng Android Keystore và xác thực sinh trắc học khi thiết bị hỗ trợ.
- **Quản lý dữ liệu trong bộ nhớ:** Xóa các mảng byte và ký tự chứa dữ liệu nhạy cảm khi không còn cần thiết, đồng thời nhận thức được giới hạn của môi trường runtime.
- **Xử lý tài liệu an toàn:** Xử lý tài liệu nhập vào bằng luồng dữ liệu, tránh tạo tệp tạm chứa nội dung chưa mã hóa.
- **Bảo vệ quyền riêng tư:** Hạn chế thông tin nhạy cảm xuất hiện trong ảnh chụp màn hình, Recents, nhật ký và các bản sao lưu không được bảo vệ.
- **Đối chiếu thông tin đăng nhập:** Xác minh danh tính ứng dụng thông qua tên gói và dấu vân tay chứng chỉ ký; hoặc đối chiếu website bằng tên miền.

## 5. Luồng mở khóa và khóa Vault

### Mở khóa bằng mật khẩu

1. Người dùng nhập mật khẩu chính.
2. Mật khẩu được xử lý bằng `CharArray`.
3. Argon2id dẫn xuất KEK dài 32 byte.
4. AES-256-GCM giải bọc DEK.
5. KEK được xóa theo cơ chế xử lý tốt nhất có thể.
6. DEK được `VaultSession` quản lý trong thời gian Vault mở khóa.
7. `VaultRepository` và `DocumentStore` sử dụng phiên đang hoạt động để xử lý dữ liệu được bảo vệ.

### Mở khóa bằng sinh trắc học

1. Người dùng yêu cầu xác thực sinh trắc học.
2. `BiometricPrompt` xác thực người dùng thông qua `CryptoObject`.
3. Cipher đã được xác thực dùng để giải bọc `wrappedDEK_bio`.
4. DEK được khôi phục vào `VaultSession`.

### Tự động khóa

Khi một trong các điều kiện khóa được kích hoạt, ứng dụng xóa DEK đang hoạt động theo cơ chế xử lý tốt nhất có thể và loại bỏ trạng thái giao diện có chứa thông tin nhạy cảm.

## 6. Danh sách kiểm tra khi triển khai

Các hạng mục sau cần được kiểm tra trong quá trình phát triển:

- [ ] Tạo salt và nonce mã hóa bằng bộ sinh số ngẫu nhiên an toàn.
- [ ] Kiểm tra tham số Argon2id trên các thiết bị Android thực tế.
- [ ] Xác minh thẻ xác thực AES-GCM trước khi chấp nhận dữ liệu đã giải mã.
- [ ] Đảm bảo `entryId` được sử dụng nhất quán làm AAD.
- [ ] Thiết kế cơ chế quản lý nonce an toàn cho mã hóa tài liệu theo khối.
- [ ] Xác thực header bản sao lưu và kiểm tra phiên bản định dạng.
- [ ] Kiểm tra hành vi xác thực Keystore và việc vô hiệu hóa khóa khi có thay đổi đăng ký sinh trắc học.
- [ ] Triển khai cơ chế khóa phù hợp khi ứng dụng xuống nền, màn hình tắt hoặc người dùng khóa thủ công.
- [ ] Ngăn thông tin nhạy cảm xuất hiện trong ảnh chụp màn hình, Recents và nhật ký.
- [ ] Tắt các cơ chế sao lưu dữ liệu ứng dụng không được phép.
- [ ] Kiểm thử tệp không hợp lệ, mật khẩu sai, ciphertext bị sửa đổi và quá trình nhập tài liệu bị gián đoạn.
- [ ] Kiểm tra quy tắc đối chiếu thông tin đăng nhập trước nguy cơ giả mạo danh tính ứng dụng hoặc tên miền website.

## 7. Các lưu ý bảo mật quan trọng

README này mô tả kiến trúc bảo mật dự kiến, không tự chứng minh rằng mọi cơ chế đã được triển khai đầy đủ hoặc đã trải qua kiểm toán bảo mật độc lập.

Trước khi đưa vào sử dụng thực tế, cần kiểm thử toàn bộ vòng đời của khóa, định dạng mã hóa, cơ chế sao lưu và khôi phục, xử lý tài liệu theo luồng, quản lý phiên và quy trình khôi phục thông qua sinh trắc học.

Việc xóa mảng dữ liệu trong môi trường runtime có cơ chế quản lý bộ nhớ không bảo đảm mọi bản sao của dữ liệu nhạy cảm đều bị xóa hoàn toàn.

Ngoài ra, hệ thống phải bảo đảm nonce mã hóa không bị sử dụng lại với cùng một khóa AES-GCM. Các thao tác bọc khóa cũng phải xác thực chính xác metadata liên quan.

## 8. Mục tiêu dự án

Mục tiêu của Secure Vault là xây dựng kiến trúc lưu trữ cục bộ an toàn cho tài liệu nhạy cảm và thông tin đăng nhập trên thiết bị Android, kết hợp mã hóa dựa trên mật khẩu, bảo vệ khóa bằng phần cứng, mở khóa sinh trắc học, lưu trữ tài liệu đã mã hóa và kiểm soát việc tự động điền thông tin đăng nhập.


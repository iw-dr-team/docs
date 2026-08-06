# Tích hợp Firebase MCP với Codex

> Hướng dẫn ngắn gọn cho Windows/PowerShell. Các khối lệnh có thể sao chép trực tiếp.

## 1. Thiết lập đường dẫn project

Chỉ cần sửa đường dẫn này:

```powershell
$ProjectDir = "D:\Projects\dancing-road\DancingRoadUnity"
Set-Location $ProjectDir
```

## 2. Kiểm tra và cài công cụ

```powershell
node --version
npm --version
codex --version
```

Nếu `node` hoặc `npm` chưa có, cài Node.js LTS bằng `winget`:

```powershell
Get-Command node -ErrorAction SilentlyContinue
Get-Command npm -ErrorAction SilentlyContinue

winget install --id OpenJS.NodeJS.LTS -e
```

Sau khi cài xong, đóng PowerShell hiện tại, mở lại PowerShell mới rồi kiểm tra lại:

```powershell
node --version
npm --version
```

Tiếp tục cài Firebase CLI:

```powershell
npm install -g firebase-tools@latest
firebase login
firebase projects:list
```

Firebase MCP dùng tài khoản đã đăng nhập bằng Firebase CLI.

## 3. Khởi tạo thư mục Firebase

Kiểm tra project đã có cấu hình Firebase CLI chưa:

```powershell
Test-Path "$ProjectDir\firebase.json"
Test-Path "$ProjectDir\.firebaserc"
```

Nếu chưa có `firebase.json`, chạy:

```powershell
Set-Location $ProjectDir
firebase init
```

Trong trình hướng dẫn:

1. Chọn **Use an existing project**.
2. Chọn đúng Firebase project.
3. Chỉ chọn các dịch vụ thực sự cần dùng.
4. Không chọn **Data Connect** nếu project Unity không sử dụng Cloud SQL/Data Connect.

Kiểm tra project đang liên kết:

```powershell
firebase use
```

Nếu chưa đúng project:

```powershell
firebase use --add
```

> `google-services.json` là cấu hình Firebase SDK cho Android. Firebase MCP sử dụng project context từ `firebase.json` và `.firebaserc`.

## 4. Kiểm tra Firebase MCP độc lập

Chạy trước khi thêm MCP vào Codex:

```powershell
npx --yes --prefer-online firebase-tools@latest mcp --generate-tool-list
```

Nếu thành công, bạn sẽ thấy các tool như:

```text
firebase_get_environment
firebase_get_project
firebase_list_projects
crashlytics_get_issue
crashlytics_list_events
```

Các cảnh báo `npm warn deprecated` có thể bỏ qua nếu bảng tool vẫn xuất hiện.

## 5. Thêm Firebase MCP vào Codex

Xóa cấu hình cũ để tránh trùng server:

```powershell
codex mcp remove firebase
codex mcp remove firebase-console
```

Thêm lại Firebase MCP:

```powershell
codex mcp add firebase -- npx --yes --prefer-online firebase-tools@latest mcp --dir "$ProjectDir"
```

Kiểm tra cấu hình:

```powershell
codex mcp get firebase
codex mcp list
```

Mở một phiên Codex mới:

```powershell
Set-Location $ProjectDir
codex
```

Trong Codex:

```text
/mcp
```

Kết quả mong đợi:

```text
firebase
  Auth: Unsupported
  Tools:
    firebase_get_environment
    firebase_get_project
    crashlytics_get_issue
    ...
```

`Auth: Unsupported` không phải lỗi với MCP chạy bằng STDIO. Điều quan trọng là có danh sách tool và không còn startup failure.

## 6. Kiểm thử trong Codex

Kiểm tra môi trường và project:

```text
Sử dụng firebase_get_environment và firebase_get_project để xác nhận môi trường Firebase hiện tại. Chỉ đọc dữ liệu.
```

Phân tích Crashlytics:

```text
Sử dụng Firebase MCP để lấy các issue Crashlytics gần đây của project hiện tại.
Lấy event và stack trace liên quan, xác định exception gốc và đề xuất hướng kiểm tra trong Unity.
Chỉ đọc dữ liệu, không đổi trạng thái issue và không tạo note.
```

## 7. Xử lý lỗi thường gặp

### PowerShell lỗi tại `--dir`

PowerShell không dùng `\` để nối dòng. Cách an toàn nhất là chạy trên một dòng:

```powershell
codex mcp add firebase -- npx --yes --prefer-online firebase-tools@latest mcp --dir "$ProjectDir"
```

### Lỗi quyền khi chạy `firebase init`

Nếu thấy:

```text
Permissions denied enabling firebasedataconnect.googleapis.com
```

Bạn đã chọn **Data Connect**. Nhấn `Ctrl+C`, chạy lại `firebase init` và chỉ chọn dịch vụ cần thiết.

### Codex hiển thị `Tools: (none)` hoặc handshake thất bại

Lỗi thường có dạng:

```text
connection closed: initialize response
```

Kiểm tra MCP độc lập:

```powershell
npx --yes --prefer-online firebase-tools@latest mcp --generate-tool-list
```

- Nếu lệnh này có bảng tool: mở lại Codex và kiểm tra cấu hình MCP.
- Nếu lệnh này lỗi: xử lý lỗi npm/npx trước.

### Lỗi thiếu dependency trong cache npx

Lỗi đã gặp:

```text
Cannot find module './internal/streams/stream'
```

Xóa cache tạm của `npx`:

```powershell
$NpmCache = npm config get cache
Remove-Item -Recurse -Force (Join-Path $NpmCache "_npx") -ErrorAction SilentlyContinue
npm cache verify
```

Tải lại package và kiểm tra:

```powershell
npx --yes --prefer-online firebase-tools@latest mcp --generate-tool-list
```

Sau khi bảng tool xuất hiện, xóa và thêm lại Firebase MCP trong Codex.

## 8. Khối lệnh hoàn chỉnh

```powershell
# Chỉ sửa dòng này
$ProjectDir = "D:\Projects\dancing-road\DancingRoadUnity"

# Kiểm tra môi trường
if (-not (Get-Command node -ErrorAction SilentlyContinue)) {
    winget install --id OpenJS.NodeJS.LTS -e
    Write-Host "Da cai Node.js LTS. Hay dong PowerShell, mo lai roi chay tiep khoi lenh nay."
    exit
}
if (-not (Get-Command npm -ErrorAction SilentlyContinue)) {
    Write-Host "Khong tim thay npm. Hay dong PowerShell, mo lai roi kiem tra lai Node.js."
    exit
}

node --version
npm --version
codex --version

# Firebase CLI
npm install -g firebase-tools@latest
firebase login
firebase projects:list

# Project context
Set-Location $ProjectDir
if (-not (Test-Path "$ProjectDir\firebase.json")) {
    firebase init
}
firebase use

# Kiểm tra MCP
npx --yes --prefer-online firebase-tools@latest mcp --generate-tool-list

# Thêm MCP vào Codex
codex mcp remove firebase
codex mcp remove firebase-console
codex mcp add firebase -- npx --yes --prefer-online firebase-tools@latest mcp --dir "$ProjectDir"

# Kiểm tra và mở Codex
codex mcp get firebase
codex mcp list
codex
```

Trong Codex:

```text
/mcp
```

## 9. Lưu ý an toàn

Firebase MCP có cả tool đọc và ghi. Với project production, hãy ghi rõ **chỉ đọc dữ liệu**, kiểm tra project bằng `firebase_get_environment` trước khi thay đổi và không deploy/cập nhật dữ liệu nếu chưa xác nhận project ID.

## 10. Test

Sau khi kết nối Firebase MCP với Codex, hãy thử prompt để thực hiện một vài yêu cầu:

Ví dụ 1:
```text
dùng firebase mcp truy cập vào Firebase Crashlytic để lấy tất cả các event có số lượng lớn hơn 50 của bản build 3.2.2 (26072414). kết quả trả về docs/CrashlyticReport.md
```
Thu được kết quả [CrashlyticReport.md](https://github.com/iw-dr-team/docs/blob/main/FirebaseMcpWithCodex/CrashlyticReport.md)

Ví dụ 2:
```text
phân tích draft DRi_Communication_System_Update giúp tôi, kết quả thu được trả về file docs/ReportVarTest.md
```

Thu được kết quả [ReportVarTest.md](https://github.com/iw-dr-team/docs/blob/main/FirebaseMcpWithCodex/ReportVarTest.md)

Lưu ý: đọc qua tài liệu [Firebase MCP](https://firebase.google.com/docs/ai-assistance/mcp-server) để biết hiện tại đã hỗ trợ những tool nào. Ví dụ như hiện tại chưa có tool hỗ trợ tạo var test trên firebase remote config

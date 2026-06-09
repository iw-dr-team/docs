## Checklist Release DR1

### 1. Xử lý với nhánh build
- [] 1.1 Tạo một branch mới từ main và đặt tên theo format sprint/DR-SprintXX (ví dụ sprint/DR-Sprint85)
- [] 1.2 Merge tất cả các branch feature đã hoàn thành trong giai đoạn này vào sprint/DR-SprintXX. Lúc này sprint/DR-SprintXX chính là branch build release android
- [] 1.3 Từ sprint/DR-SprintXX hãy tạo một branch mới đặt tên theo format sprint/DR-SprintXX-iOS
- [] 1.4 Merge branch main_iOS_v3 vào sprint/DR-SprintXX-iOS. Lúc này sprint/DR-SprintXX-iOS chính là branch build release ios

### 2. Setup + Build

- [] 2.1 Đăng nhập Unity Pro
- [] 2.2 Tắt `Show Unity Logo`
- [] 2.3 Check version và bundle version code (version code theo quy luật: năm/tháng/ngày/giờ - ví dụ: 25112714)
Hiện tại có thể bấm `Iwave` > `Generate Bundle Version` để tự động generate bundle version code + điền pass build
- [] 2.4 Chọn compress method LZ4 cho build android và LZ4HC cho build ios
- [] 2.5 Thực hiện build

### 3. Kiểm tra sau build và distribute

- [] 3.1 [Check Link SDK version tracking, không được để cho version các sdk thay đổi khi không có task update sdk ⇒ không tùy tiện update sdk](https://docs.google.com/spreadsheets/d/1Cs3YKDnOBMwlAKxLjKN1Jd84F3whkMAbRg8xUrStsj0/edit?gid=1655533654#gid=1655533654)
- [] 3.2 So sánh size build với các bản trước đó, cần kiểm soát được size build và nắm rõ lý do nếu bản build bị tăng size.
- [] 3.3 Đẩy bản build android lên Firebase Distribution (Đẩy lên testflight đối với bản build ios)

### 4. Xử lý sau khi đã release

- [] 4.1 Tạo tag github và đặt tên theo format DRa_x.x.x / DRi_x.x.x (ví dụ DRa_3.1.2)
- [] 4.2 Merge branch sprint/DR-SprintXX về main và merge branch sprint/DR-SprintXX-iOS về main_iOS_v3

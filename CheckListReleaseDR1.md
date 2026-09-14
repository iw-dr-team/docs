## Checklist Release DR1

### 1. Xử lý với nhánh build
- [ ] 1.1 Tạo một branch mới từ `main` và đặt tên theo format `sprint/DR-SprintXX` (ví dụ `sprint/DR-Sprint85`)
- [ ] 1.2 Merge tất cả các branch feature đã hoàn thành trong giai đoạn này vào `sprint/DR-SprintXX`. Lúc này `sprint/DR-SprintXX` chính là branch build release android
- [ ] 1.3 Từ `sprint/DR-SprintXX` hãy tạo một branch mới đặt tên theo format `sprint/DR-SprintXX-iOS`
- [ ] 1.4 Merge branch main_iOS_v3 vào `sprint/DR-SprintXX-iOS`. Lúc này `sprint/DR-SprintXX-iOS` chính là branch build release ios

### 2. Setup + Build

- [ ] 2.1 Đăng nhập Unity Pro
- [ ] 2.2 Tắt `Show Unity Logo`
- [ ] 2.3 Check version và bundle version code (version code theo quy luật: năm/tháng/ngày/giờ - ví dụ: 25112714)
Hiện tại có thể bấm `Iwave` > `Generate Bundle Version` để tự động generate bundle version code + điền pass build
- [ ] 2.4 Chọn compress method LZ4 cho build android và LZ4HC cho build ios
- [ ] 2.5 Thực hiện build

### 3. Kiểm tra sau build và distribute

- [ ] 3.1 [Check Link SDK version tracking, không được để cho version các sdk thay đổi khi không có task update sdk ⇒ không tùy tiện update sdk](https://docs.google.com/spreadsheets/d/1Cs3YKDnOBMwlAKxLjKN1Jd84F3whkMAbRg8xUrStsj0/edit?gid=1655533654#gid=1655533654)
- [ ] 3.2 So sánh size build với các bản trước đó, cần kiểm soát được size build và nắm rõ lý do nếu bản build bị tăng size.
- [ ] 3.3 Đẩy bản build android lên Firebase Distribution (Đẩy lên testflight đối với bản build ios)
- [ ] 3.4 Đối với bản build ios, yêu cầu test thử trên màn hình ipad (thiết bị thật) trong trường hợp làm các feature liên quan tới UI/UX (tránh các lỗi hiển thị)

### 4. Xử lý sau khi đã release

- [ ] 4.1 Tạo tag github và đặt tên theo format `DRa_x.x.x` / `DRi_x.x.x` (ví dụ `DRa_3.1.2`)
- [ ] 4.2 Merge branch `sprint/DR-SprintXX` về `main` và merge branch `sprint/DR-SprintXX-iOS` về `main_iOS_v3`

### 5. Đẩy debugging symbols lên firebase crashlytic (bước này có thể làm ngay sau khi build aab, không cần đợi đến khi release xong)

- [ ] Trong build settings của Unity chọn option ***Debugging***

<img width="653" height="608" alt="Screenshot 2026-08-11 150420" src="https://github.com/user-attachments/assets/61082c7d-2d60-4038-a4f8-4504256c9393" />

- [ ] Build như bình thường sẽ nhận được file zip debug

<img width="757" height="165" alt="Screenshot 2026-08-11 150126" src="https://github.com/user-attachments/assets/35480f3c-bee0-4599-912f-34df2db17e1d" />

- [ ] Run trên terminal lệnh

```
firebase crashlytics:symbols:upload --app=1:528923424982:android:46195f809d709f94 "path symbols.zip"
```
Ví dụ: 
```
firebase crashlytics:symbols:upload --app=1:528923424982:android:46195f809d709f94 "D:\Build_DR1\dr1_323-3.2.3-v26081114-IL2CPP.symbols.zip"
```
trong đó ***D:\Build_DR1\dr1_323-3.2.3-v26081114-IL2CPP.symbols.zip*** là path của file symbols.zip

Đợi đến khi trên terminal báo ***i  Successfully uploaded all symbols*** là thành công

<img width="1132" height="637" alt="Screenshot 2026-08-11 145642" src="https://github.com/user-attachments/assets/044dc375-492f-4d96-b995-f4ba89b5dff6" />
<img width="1148" height="323" alt="Screenshot 2026-08-11 145826" src="https://github.com/user-attachments/assets/fc999c03-210f-4a16-a0ec-d3f2e71a27aa" />

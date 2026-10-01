# Hướng dẫn thêm Ball mới vào Shop

Tài liệu mô tả các bước setup một ball mới, dựa trên cách ball 73/74 (EDM Festival) đã được thêm.

## 1. Tổng quan

Một ball được ghép từ 3 nguồn, liên kết với nhau qua **số thứ tự N**:

| Thành phần | Vị trí | Quy tắc tên |
|---|---|---|
| Dòng trong shop | `Assets/Resources/shopconfig.csv` | `Id = 100 + N`, `Earn = N` |
| Config gameplay | `Assets/Resources/ScriptableObjects/Balls/Ball_{N-1}.asset` | `_id = N` |
| Icon trong shop | `Assets/Textures/ItemIcon/v2_ball_{N-1}.png` (nằm trong `ItemIcon.spriteatlas`) | `v2_ball_{N-1}` |

Code liên quan:

- `InGameAssets.LoadBallConfig()` (`Assets/Scripts/Handle/InGameAssets.cs`): load `ScriptableObjects/Balls/Ball_{CurrBall - 1}` và prefab `Prefabs/Balls/{_prefName}`. Nếu thiếu asset, game **âm thầm quay về BallBasic** (DR-6340), chỉ để lại một warning `Missing ball assets for ball N`.
- `ShopConfig.GetIconShop()` (`Assets/Features/Shop/Scripts/ShopConfig.cs`): lấy sprite `v2_ball_{id - 1}` từ `UISupport.instance.commonAtlas`.
- `ShopItem(string[])` (`Assets/Features/Shop/Scripts/ShopItem.cs`): parse từng dòng CSV.
- `ShopData.LoadOnlineData()` (`Assets/Features/Shop/Scripts/ShopData.cs`): tải CSV online và ghi đè bản local.

> Ví dụ trong tài liệu: ball cuối hiện tại là N = 74, nên ball mới là **N = 75**, tức Id `175`, asset `Ball_74.asset`, icon `v2_ball_74`. Luôn kiểm tra lại dòng `BALL` cuối cùng trong `shopconfig.csv` trước khi chọn N.

## 2. Các bước thực hiện

### Bước 1: Model 3D, texture, material

| Asset | Vị trí | Ghi chú |
|---|---|---|
| Mesh | `Assets/Resources/Models/Ball_{N}.mesh` | Tên file tự đặt, `_meshName` trong BallConfig trỏ tới tên này |
| Texture màu | `Assets/Resources/Textures/Ball/ball_{N-1}.png` | Code load theo `ball_{_id - 1}` |
| Texture base | `Assets/Resources/Textures/Ball/ball_{N-1}_Base.png` | Chỉ cần khi bật `isUseBaseTex` |
| Material | `Assets/Materials/Ball/` | Thường dùng lại `BallTintMasked.mat` như ball 74. Tạo material riêng nếu cần shader khác |
| VFX (nếu có) | `Assets/Resources/Prefabs/Vfxs/Vfx_{meshName}.prefab` | Bật `isHasVfx` trong BallConfig |
| Sprite fake specular | `Assets/Resources/Textures/BallSprite/` | Mặc định `ball_transparent` |

### Bước 2: Tạo BallConfig

1. Duplicate một ball tương tự (ví dụ `Ball_73.asset`), rồi đổi tên thành `Ball_{N-1}.asset`.
2. Sửa các field:
   - `_id`: **N**
   - `_meshName`: tên mesh ở bước 1
   - `_prefName`: giữ `BallCustom`, trừ khi ball cần prefab riêng trong `Resources/Prefabs/Balls/`
   - `_prefAppear`: prefab hiệu ứng xuất hiện (nếu có)
   - `_gameplayMaterials` / `_shopMaterials`: material dùng trong gameplay và trong preview ở shop
   - `vfxMaterials`, `vfxGradientColors`, `vfxColors`: chỉ dùng khi ball có VFX
   - Settings: `isHasMesh`, `isHasVfx`, `isHasUvFake`, `isUseBaseTex`, `isVfxInBall`, `timeAppear`
   - Colors: `_colorsChannel1/2/3`, `_colorsRim` (mỗi mảng 4 phần tử)
   - Rim light: `_rimPower`, `_rimThreshold`
   - Rotation: `_parentRotation`, `_childRotation`, `_mainChildRotation`, `_moveRotation`, `_rotateType`, `_axisType`, `_speedRotation`
   - Scale: `_shopVfxChildScale`

> Nếu `_prefName` trỏ tới một prefab không tồn tại, game không báo lỗi mà chỉ quay về BallBasic.

### Bước 3: Icon trong shop

1. Thêm `Assets/Textures/ItemIcon/v2_ball_{N-1}.png`, import setting giống `v2_ball_73.png` (Sprite, cùng max size và compression).
2. Thêm sprite vào `Assets/Atlas/Resources/ItemIcon.spriteatlas` (mục *Objects for Packing*).
3. Nếu quên bước 2, ô trong shop sẽ không có ảnh.

### Bước 4: Thêm dòng vào shopconfig

Header: `Id,Name,EarnType,Earn,PriceType,Price,ShopType,Category`

```csv
175,Ball 75,BALL,75,GOLD,150,BALL,HALLOWEEN
```

| Cột | Giá trị | Ghi chú |
|---|---|---|
| Id | `100 + N` | |
| Name | `Ball N` | |
| EarnType | `BALL` | Enum `Currency` |
| Earn | `N` | Phải khớp `_id` trong BallConfig |
| PriceType | `GOLD` / `ADS` / ... | Có thể ghi 2 loại, phân tách bằng `;`, ví dụ `ADS;GOLD` |
| Price | `150` | Ghi 2 giá tương ứng nếu có, ví dụ `1;100`. Giá thứ 2 chỉ có hiệu lực khi `RemoteData.ShopUI_CtaRevamp = true` |
| ShopType | `BALL` | Enum `ShopType` |
| Category | `NONE` / `HALLOWEEN` / ... | Enum `CategoryType`. Giá trị sai sẽ bị parse thành `NONE` |

File cần cập nhật:

- `Assets/Resources/shopconfig.csv` (file mặc định, `AppData.SHOP_CONFIG_FILE`)
- `Assets/Resources/shopconfig_thy1.csv` (nếu bản build/test đó vẫn được dùng)

### Bước 5: Category / sự kiện (chỉ khi ball thuộc sự kiện mới)

- **Dùng enum có sẵn** (ví dụ `HALLOWEEN`, `EDMFESTIVAL`): không cần sửa code.
- **Thêm sự kiện mới**:
  1. Thêm giá trị vào `CategoryType` trong `Assets/Scripts/Enums.cs`. Thêm vào **cuối enum**, không chèn giữa, để không làm lệch giá trị đã lưu.
  2. Thêm case sắp xếp trong `ShopItemSortService.cs` (xem cách làm với case `HALLOWEEN` / `EDMFESTIVAL`).
  3. Thêm badge và object hiển thị theo trạng thái sự kiện trong `ShopObjectV3.cs` (xem `objHalloween` / `objEDM2026`), rồi gán object tương ứng trong prefab ô shop.

### Bước 6: Remote config / CSV online

- `ShopData.LoadOnlineData()` tải CSV từ `RemoteData.ShopUI_ShopConfigUrl`, ghi đè file local và dùng bản này thay cho CSV trong build. Vì vậy **phải cập nhật cả file CSV online**, nếu không người chơi thật sẽ không thấy ball.
- Thứ tự an toàn:
  1. Phát hành bản build có đủ asset (BallConfig, mesh, icon).
  2. Sau đó mới thêm dòng vào CSV online.
- Nếu làm ngược lại, người dùng bản cũ sẽ thấy item không có icon, và khi chọn ball thì game quay về BallBasic.
- Dùng `RemoteData.ShopConfig_RemoveID` (danh sách Id, phân tách bằng dấu phẩy) để ẩn item khi cần.

### Bước 7: Kiểm tra

- [ ] Shop: icon hiển thị đúng, giá và loại tiền đúng, đúng tab Ball, đúng category/badge sự kiện.
- [ ] Mua/unlock ball bằng mọi `PriceType` đã cấu hình.
- [ ] Equip, rồi vào gameplay: mesh, material, màu, rim light, rotation đều đúng.
- [ ] Console không có log `[InGameAssets] Missing ball assets for ball N`.
- [ ] Preview 3D trong shop (`_shopMaterials`, `_shopVfxChildScale`).
- [ ] Các màn khác có hiển thị ball: EndGame, ColorGate (`ColorGateBallAssets`).
- [ ] Ball có VFX: thử trên máy cấu hình thấp (`AppData.IsLowDevice()`).
- [ ] Thử với CSV local (`AppData.isUsingLocalCSV`) và với CSV online.

## 3. Lưu ý về code được bảo vệ

Thêm ball theo các bước trên **không cần sửa file nào trong `docs/ProtectedScripts.md`**. Hỏi ý kiến trước khi:

- Sửa `ShopTrackingEvent.cs`, `ShopFreeReward.cs`, `CurrencyShopV2.cs`, `PremiumShopV2.cs` hoặc bất kỳ file nào trong `Features/ShopIAP/`.
- Cho ball unlock bằng IAP, gói bundle, hoặc reward ads mới.
- Thêm hoặc đổi event, parameter tracking cho ball.

## 4. Checklist nhanh

| # | Việc | File |
|---|---|---|
| 1 | Mesh | `Resources/Models/Ball_{N}.mesh` |
| 2 | Texture (nếu có) | `Resources/Textures/Ball/ball_{N-1}.png` |
| 3 | Material | `Materials/Ball/` |
| 4 | VFX (nếu có) | `Resources/Prefabs/Vfxs/Vfx_{meshName}.prefab` |
| 5 | BallConfig | `Resources/ScriptableObjects/Balls/Ball_{N-1}.asset` (`_id = N`) |
| 6 | Icon | `Textures/ItemIcon/v2_ball_{N-1}.png` |
| 7 | Atlas | `Atlas/Resources/ItemIcon.spriteatlas` |
| 8 | CSV local | `Resources/shopconfig.csv` (+ `shopconfig_thy1.csv`) |
| 9 | Category mới (nếu có) | `Enums.cs`, `ShopItemSortService.cs`, `ShopObjectV3.cs` |
| 10 | CSV online | `RemoteData.ShopUI_ShopConfigUrl`, cập nhật **sau** khi build đã phát hành |

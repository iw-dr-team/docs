# Hướng dẫn thêm Character mới vào game

Tài liệu mô tả quy trình setup một character mới, dựa trên tool `DancingRoad > Character Import Tool` (`Assets/Scripts/Editor/CharacterImportTool.cs`) và cách các character 23 → 26 (SummerBoy, SummerGirl, EDMFestival1/2) đã được thêm.

## 1. Tổng quan

Một character được ghép từ 4 nguồn, liên kết với nhau qua **index K** (= số đầu tên folder model). Trong shop/save data dùng **Earn = K + 1**.

| Thành phần | Vị trí | Quy tắc tên |
|---|---|---|
| Model + animation gốc | `Assets/Models/Characters/{K}.{Name}/` | Folder bắt đầu bằng số `K`, ví dụ `27.HalloweenWitch` |
| Prefab | `Assets/Resources/Prefabs/Characters/{K}.prefab` | Load bằng `Resources.Load("Prefabs/Characters/" + (Earn - 1))` |
| Icon trong shop | `Assets/Textures/ItemIcon/v2_character_{K}.png` | `ShopConfig.GetIconShop()` lấy sprite `v2_character_{Earn - 1}` |
| Dòng trong shop | `Assets/Resources/shopconfig.csv` | `Earn = K + 1` |

Code liên quan:

- `BallController.CreateCharacter()` (`Assets/Scripts/Handle/Map/BallController.cs`): load prefab trong gameplay, gán `Animator`, scale theo `RemoteData.Character_Scale` (mặc định `1.2`). Nếu thiếu prefab chỉ log `[DR-6688] Character prefab not found` rồi **chơi không có character**.
- `InGameSupportV3.LoadCharacter()` (`Assets/Scripts/V3/Handle/InGameSupportV3.cs`): preview ở Home/Shop. Trigger `shop` nếu có state `Shop`, nếu không thì trigger `run`. Tốc độ animation = `RemoteData.Character_RunSpeed` (mặc định `2.5`).
- `ShopConfig.GetIconShop()` (`Assets/Features/Shop/Scripts/ShopConfig.cs`): lấy icon từ `UISupport.instance.commonAtlas`.
- `MainShopUIV3` (`Assets/Features/Shop/Scripts/MainShopUIV3.cs`): dựng list character từ `AppData.ListShopItem` (`EarnType == CHARACTER`).

> Ví dụ trong tài liệu: index lớn nhất hiện tại là **K = 26** (`26.EDMFestival2`), nên character mới là **K = 27**, tức prefab `27.prefab`, icon `v2_character_27`, `Earn = 28`. Luôn kiểm tra lại `Assets/Models/Characters/` và `Assets/Resources/Prefabs/Characters/` trước khi chọn K. Các index 5, 13–16 đang bị bỏ trống, **không dùng lại** (có thể đã tồn tại trong save data hoặc CSV online cũ).

## 2. Các bước thực hiện

### Bước 1: Chuẩn bị asset từ artist

Tạo folder `Assets/Models/Characters/{K}.{Name}/` (regex của tool: `^(\d+)[.\s]+(.+)$`, nên `27.Witch` hay `27. Witch` đều được) và bỏ vào:

| File | Bắt buộc | Ghi chú |
|---|---|---|
| FBX base (mesh + rig) | ✅ | Tên file phải chứa `base`, `rig` hoặc `mesh` để tool nhận là model gốc. Nếu folder chỉ có 1 FBX thì FBX đó được dùng làm base |
| FBX animation | ✅ | Mỗi FBX 1 hoặc nhiều clip. Tên file/clip nên chứa keyword ở bảng dưới để Auto-Map |
| Texture (`.png/.jpg/.tga/.psd`) | ✅ | File ảnh **đầu tiên** trong folder được dùng làm main texture của material và làm icon tạm |
| Material (`.mat`) | Tuỳ chọn | Nếu có sẵn, tool dùng lại. Nếu không, tool tạo `{Name}Material.mat` với shader `Standard` |

**Keyword để Auto-Map animation** (không phân biệt hoa thường):

| State | Keyword nhận | Keyword loại trừ | Fallback nếu thiếu |
|---|---|---|---|
| `Intro` | `idle`, `intro` | `idle_2`, `idle2`, `run2`, `run_2` | `idle_2` → `run` |
| `idle_2` | `idle_2`, `idle2`, `run2`, `run_2` | | |
| `run` | `run`, `walk` | `run2`, `run_2` | |
| `die` | `die`, `dead`, `death` | | |
| `fly` | `fly`, `flying`, `float` | | `idle_2` → `run` |
| `break_road_start` | `breakroad_start`, `break_road_start`, `breakstart`, `brs` | `end` | |
| `break_road_end` | `breakroad_end`, `break_road_end`, `breakend`, `bre` | | |
| `jump_break_start` | `jumpbreak_start`, `jumpbreakstart`, `jump_break_start`, `jumpbreak` | `end` | |
| `jump_break_end` | `jumpbreak_end`, `jumpbreakend`, `jump_break_end` | | |
| `Shop` | `shop`, `display` | | `idle_2` → `run` |

> Đặt tên file FBX animation theo dạng `{Name}_run.fbx`, `{Name}_die.fbx`… để Auto-Map đúng ngay lần đầu, đỡ phải chọn tay.

**Yêu cầu với model:**
- Pivot ở chân character, mặt hướng theo trục +Z như các character có sẵn.
- Không dùng root motion (tool tắt `applyRootMotion`).
- Tool scale model `0.65` trong prefab, sau đó game nhân thêm `Character_Scale`. Nên đặt **Scale Factor** trong Import Settings sao cho chiều cao khớp character tham chiếu (ví dụ `24.prefab`), không bù bằng Transform.

### Bước 2: Chạy Character Import Tool

1. Mở `DancingRoad > Character Import Tool`, bấm **Refresh**.
2. Chọn character mới (chấm màu: 🔴 thiếu animation, 🟡 sẵn sàng, 🟠 đã có prefab nhưng thiếu CSV/icon, 🟢 xong). Cột `P` / `C` / `I` = Prefab / CSV / Icon.
3. Mở panel chi tiết (`▼`):
   1. **Section 1 – Prefab & Animations**
      - Bấm **Extract Clips**: tool bật `Import Animation` (Generic) cho mọi FBX, rồi tách clip từ các FBX animation thành file `.anim` trong cùng folder.
      - Bấm **Auto-Map**, kiểm tra từng dòng state. Dòng **đỏ** là state chưa có clip → chọn tay trong dropdown.
      - Bấm **Build Prefab**.
   2. **Section 2 – shopconfig.csv**: nhập `Display Name`, chọn `Category`, `Price Type`, `Price` (xem Bước 4), bấm **Add to shopconfig.csv**.
   3. **Section 3 – Item Icon**: bấm **Copy Icon to ItemIcon folder**.
4. Hoặc bấm **▶ Setup All Steps** để chạy cả 7 bước một lần: FBX → Extract clips → Material → Animator → Prefab → CSV → Icon.

Tool sinh ra:

| Asset | Vị trí |
|---|---|
| Clip `.anim` | `Assets/Models/Characters/{K}.{Name}/*.anim` |
| Material | `Assets/Models/Characters/{K}.{Name}/{Name}Material.mat` (nếu chưa có) |
| Animator Controller | `Assets/Models/Characters/{K}.{Name}/{Name}Animator.controller` (**luôn bị xoá và tạo lại** mỗi lần build) |
| Prefab | `Assets/Resources/Prefabs/Characters/{K}.prefab` |
| Dòng CSV | cuối `Assets/Resources/shopconfig.csv` |
| Icon (tạm) | `Assets/Textures/ItemIcon/v2_character_{K}.png` |

> ⚠️ **Re-Build Prefab / Re-Setup ghi đè** prefab và animator controller. Mọi chỉnh tay trên prefab (thêm VFX, đổi transition, offset…) sẽ mất. Chỉnh xong thì không bấm lại tool nữa, hoặc ghi chú lại để làm lại sau khi rebuild.

**Animator controller do tool tạo:**

- Trigger: `idle`, `run`, `die`, `fly`, `break_road_start`, `break_road_end`, `jump_break_start`, `jump_break_end`, `turn_back`, `shop`.
- Default state: `Intro` → tự sang `idle_2` (exit time 0.9375).
- `Shop` chạy speed `0.5`, tự sang `run` khi hết (exit time 0.90625).
- Transition giữa các state đều dùng duration `0.25s`. **Không đổi tên state/trigger**, vì gameplay gọi `SetTrigger` theo đúng các tên này.

### Bước 3: Icon trong shop

Tool chỉ copy **texture của model** làm icon tạm, nên cần thay bằng ảnh portrait thật:

1. Thay file `Assets/Textures/ItemIcon/v2_character_{K}.png` bằng ảnh do artist cung cấp, giữ nguyên tên file (để giữ GUID trong `.meta`).
2. Import setting giống `v2_character_24.png` (Texture Type = Sprite, cùng max size và compression).
3. Atlas `Assets/Atlas/Resources/ItemIcon.spriteatlas` pack **cả folder** `Assets/Textures/ItemIcon`, nên không cần thêm sprite thủ công. Chỉ cần kiểm tra ở Inspector của atlas (bấm *Pack Preview*) là sprite mới đã có.

### Bước 4: Dòng trong shopconfig

Header: `Id,Name,EarnType,Earn,PriceType,Price,ShopType,Category`

```csv
486,Witch,CHARACTER,28,ADS;GOLD,2;300,CHARACTER,HALLOWEEN
```

| Cột | Giá trị | Ghi chú |
|---|---|---|
| Id | Id lớn nhất trong file + 1 | Tool tự tính (`NextCsvId`). Id phải **duy nhất trên toàn file** (ball, road, trail… dùng chung) |
| Name | Tên hiển thị | |
| EarnType | `CHARACTER` | Enum `Currency` |
| Earn | **`K + 1`** | Phải khớp prefab `{Earn - 1}.prefab` và icon `v2_character_{Earn - 1}` |
| PriceType | `ADS;GOLD`, `ADS`, `GOLD`, `DIAMOND`, `USD`, `NONE`, hoặc tên sự kiện | Character sự kiện thường để PriceType = Category (ví dụ `THANKSGIVING2025`) |
| Price | `2;300`, `1`… | Ghi 2 giá nếu PriceType có 2 loại. Giá thứ 2 chỉ có hiệu lực khi `RemoteData.ShopUI_CtaRevamp = true` |
| ShopType | `CHARACTER` | Enum `ShopType` |
| Category | `NONE` / `HALLOWEEN` / `SUMMER2026`… | Enum `CategoryType` (`Assets/Scripts/Enums.cs`). Giá trị sai sẽ bị parse thành `NONE` |

**Lưu ý khi dùng tool cho bước này:**

- Category mặc định của tool là `SUMMER2026`. **Nhớ đổi lại** cho đúng sự kiện.
- Dropdown `Category` của tool **chưa có** `EDMFESTIVAL`, `HALLOWEEN2026`. Nếu cần các giá trị này: hoặc sửa tay dòng CSV sau khi tool thêm, hoặc bổ sung vào `CategoryOptions` trong `CharacterImportTool.cs`.
- Tool chỉ thêm vào `shopconfig.csv`. Nếu bản build/test còn dùng `Assets/Resources/shopconfig_thy1.csv` thì thêm tay vào file đó.
- Tool nhận biết "đã có dòng CSV" bằng cách tìm `Earn = K + 1`. Nếu dòng có Earn sai, tool sẽ báo thiếu CSV và thêm một dòng trùng.

> ⚠️ Hiện tại 2 dòng EDM (`484 EDM DJ`, `485 EDM Dancer`) đều có `Earn = 73`, trong khi prefab là `25.prefab` / `26.prefab` (tức Earn đúng phải là `26` / `27`). Cần kiểm tra lại trước khi copy các dòng này làm mẫu.

### Bước 5: Category / sự kiện (chỉ khi character thuộc sự kiện mới)

- **Dùng enum có sẵn** (`HALLOWEEN`, `XMAS2025`, `SUMMER2026`, `EDMFESTIVAL`…): không cần sửa code.
- **Thêm sự kiện mới**:
  1. Thêm giá trị vào `CategoryType` trong `Assets/Scripts/Enums.cs`. Thêm vào **cuối enum**, không chèn giữa.
  2. Thêm case sắp xếp trong `ShopItemSortService.cs`.
  3. Thêm badge/object theo sự kiện trong `ShopObjectV3.cs`, gán object tương ứng trong prefab ô shop.
  4. Thêm giá trị vào `CategoryOptions` (và `DefaultPriceForCategory` nếu cần) trong `CharacterImportTool.cs`.
- Nếu character được bán/tặng trong gói sự kiện (IAP, popup pack, reward), xem mục 3 — đó là code được bảo vệ.

### Bước 6: Remote config / CSV online

- `ShopData.LoadOnlineData()` tải CSV từ `RemoteData.ShopUI_ShopConfigUrl` và ghi đè bản local, nên **phải cập nhật cả CSV online**, nếu không người chơi thật sẽ không thấy character.
- Thứ tự an toàn:
  1. Phát hành bản build có đủ prefab + icon.
  2. Sau đó mới thêm dòng vào CSV online.
- Nếu làm ngược lại, người dùng bản cũ thấy ô shop không có icon, và khi equip thì gameplay không có character (lỗi `[DR-6688]`).
- Dùng `RemoteData.ShopConfig_RemoveID` để ẩn item khi cần.
- `Character_Scale` và `Character_RunSpeed` là remote config **dùng chung cho mọi character**. Không chỉnh để sửa một character riêng lẻ; sửa scale/speed trong prefab/import setting thay vì vậy.

### Bước 7: Test bằng shopconfig local

Giống quy trình của ball (xem `docs/AddNewBall.md` Bước 7):

- Tạm chặn remote: thêm `OnGetListItemDone(); return;` đầu `ShopData.LoadOnlineData()`, **hoặc** tick `Is Using Local CSV` trong `SplashController` (scene `Assets/Scenes/Splash.unity`).
- Xoá cache `%USERPROFILE%\AppData\LocalLow\Inwave\Dancing Road\shopconfig.csv` (Editor) hoặc xoá data app (thiết bị).
- ⚠️ **Discard các thay đổi test trước khi commit** (`git checkout -- Assets/Features/Shop/Scripts/ShopData.cs Assets/Scenes/Splash.unity`).

### Bước 8: Kiểm tra

- [ ] Tool hiển thị chấm 🟢 và đủ `P` / `C` / `I`.
- [ ] Mở `{K}.prefab`: model đúng kích thước so với character tham chiếu, material/texture đúng, `Animator` đã gán controller.
- [ ] Shop: icon đúng (không phải texture tạm), giá và loại tiền đúng, đúng tab Character, đúng category/badge.
- [ ] Preview ở Home/Shop chạy animation `Shop` (hoặc `run`), không bị T-pose.
- [ ] Mua/unlock bằng mọi `PriceType` đã cấu hình, rồi equip.
- [ ] Gameplay: `Intro` → `idle_2` → `run`, chết (`die`), phá đường (`break_road_*`), nhảy (`jump_break_*`), bay (`fly`) đều có animation, không đứng im.
- [ ] Console không có `[DR-6688] Character prefab not found` hoặc `[LoadCharacter] ...`.
- [ ] Các màn khác có hiển thị icon character: Daily Check-in, Unlock Skin, Mission Progress, Receive Reward.
- [ ] Thử với CSV local và CSV online.

## 3. Lưu ý về code được bảo vệ

Thêm character theo các bước trên **không cần sửa file nào trong `docs/ProtectedScripts.md`**. Hỏi ý kiến trước khi:

- Sửa `ShopTrackingEvent.cs`, `ShopFreeReward.cs`, `CurrencyShopV2.cs`, `PremiumShopV2.cs` hoặc bất kỳ file nào trong `Features/ShopIAP/`.
- Sửa các `HandlePurchaseCallback*.cs` của sự kiện (ví dụ `HandlePurchaseCallbackXmas2025.cs`, `HandlePurchaseCallbackThanksGiving.cs`) để đưa character vào gói IAP.
- Cho character unlock bằng IAP, gói bundle, hoặc reward ads mới.
- Thêm hoặc đổi event, parameter tracking cho character.

## 4. Checklist nhanh

| # | Việc | File |
|---|---|---|
| 1 | Folder model (FBX base + FBX anim + texture) | `Models/Characters/{K}.{Name}/` |
| 2 | Extract Clips → Auto-Map → chọn tay state đỏ | Character Import Tool, Section 1 |
| 3 | Build Prefab | `Resources/Prefabs/Characters/{K}.prefab` |
| 4 | Dòng CSV (`Earn = K + 1`, đổi Category khỏi mặc định `SUMMER2026`) | `Resources/shopconfig.csv` (+ `shopconfig_thy1.csv`) |
| 5 | Icon thật (thay icon tạm) | `Textures/ItemIcon/v2_character_{K}.png` |
| 6 | Category mới (nếu có) | `Enums.cs`, `ShopItemSortService.cs`, `ShopObjectV3.cs`, `CharacterImportTool.cs` |
| 7 | Test local, rồi discard thay đổi test | `ShopData.cs` / `Splash.unity` |
| 8 | CSV online, cập nhật **sau** khi build đã phát hành | `RemoteData.ShopUI_ShopConfigUrl` |

# Advertising Deep Dive

Tài liệu này đi sâu hơn vào cách quảng cáo được load, phân phối qua mediation và vì sao thường xảy ra conflict dependency khi tích hợp nhiều ad network trong Unity.

## 1. Những khái niệm cần phân biệt

### 1.1 Mediation SDK
Mediation SDK là lớp API chính mà game gọi trong code Unity. Ví dụ:

- Google Mobile Ads SDK khi dùng AdMob mediation.
- AppLovin MAX SDK khi dùng MAX mediation.
- Unity LevelPlay SDK khi dùng LevelPlay mediation.

Game thường chỉ gọi các API như initialize, load interstitial, show rewarded, destroy banner... từ mediation SDK. Game không gọi trực tiếp API của từng ad network nếu đang đi theo mô hình mediation.

### 1.2 Adapter
Adapter là lớp cầu nối giữa mediation SDK và SDK gốc của từng ad network.

Ví dụ khi dùng AdMob mediation nhưng muốn lấy ads từ AppLovin, project cần:

- Google Mobile Ads Unity plugin.
- AppLovin mediation adapter tương ứng với AdMob.
- AppLovin SDK gốc.

Mediation SDK biết khi nào cần gọi adapter. Adapter biết cách chuyển request/callback giữa mediation SDK và SDK gốc của ad network.

### 1.3 Ad network SDK
Đây là SDK gốc của network thực sự có inventory quảng cáo. Ví dụ AppLovin SDK, Meta Audience Network SDK, Unity Ads SDK, Mintegral SDK, InMobi SDK...

Ad network SDK chịu trách nhiệm giao tiếp với server của network đó, tải creative, xử lý cache, render quảng cáo và trả callback về adapter.

### 1.4 Dashboard config
Phần lớn logic phân phối không nằm trong code Unity mà nằm trên dashboard của mediation:

- App id / SDK key.
- Ad unit id / placement id.
- Mediation group.
- Country, platform, app version, user segment.
- Bidding source.
- Waterfall instance, eCPM floor, priority.
- Frequency cap, pacing, capping theo placement.

Vì vậy, cùng một build app nhưng thay đổi dashboard có thể làm hành vi phân phối ads thay đổi mà không cần update binary.

## 2. File dependencies .xml thực sự làm gì?

Trong Unity, các plugin quảng cáo thường tạo file `Dependencies.xml` hoặc các file tương tự trong `Assets/ExternalDependencyManager`.

File này không trực tiếp load quảng cáo. Nó chỉ khai báo app cần native library nào trên Android/iOS.

Ví dụ ở Android, file dependency có thể khai báo Maven artifact:

```xml
<androidPackage spec="com.applovin:applovin-sdk:13.6.2" />
<androidPackage spec="com.google.ads.mediation:applovin:13.6.2.0" />
```

Khi chạy resolve, EDM4U sẽ đưa các khai báo này vào Gradle. Sau đó Gradle tải `.aar`/`.jar` từ Maven repository về Gradle cache và đóng gói chúng vào app.

Ở iOS, file dependency thường khai báo CocoaPods pod:

```xml
<iosPod name="AppLovinSDK" version="13.6.2" />
<iosPod name="GoogleMobileAdsMediationAppLovin" version="13.6.2.0" />
```

Khi build iOS, EDM4U sinh hoặc cập nhật `Podfile`, sau đó `pod install` tải `.xcframework`/framework về thư mục `Pods` và tạo `.xcworkspace`.

## 3. Luồng build-time

Luồng build-time là giai đoạn app chuẩn bị đủ native SDK để có thể gọi quảng cáo sau này.

```text
Unity plugin
  -> Dependencies.xml
  -> EDM4U resolve
  -> Gradle/CocoaPods
  -> Maven/Pods repository
  -> Gradle cache/Pods folder
  -> Native libraries packaged into APK/AAB/IPA
```

Các lỗi thường xảy ra ở giai đoạn này:

- Không tải được artifact vì mất mạng, repository lỗi hoặc version đã bị remove.
- Hai plugin yêu cầu cùng một dependency nhưng khác version.
- Android manifest merge fail.
- Duplicate class trên Android.
- CocoaPods không resolve được version chung.
- iOS duplicate symbol hoặc framework không tương thích kiến trúc.
- Không sinh được `.xcworkspace` sau `pod install`.

## 4. Luồng runtime khi gọi load ad

Luồng runtime là giai đoạn app đã chạy trên thiết bị và bắt đầu request quảng cáo.

```text
Game code
  -> Mediation SDK load(adUnitId/placement)
  -> Mediation SDK lấy config từ dashboard/cache
  -> Chọn bidding/waterfall/hybrid
  -> Gọi adapter phù hợp
  -> Adapter gọi ad network SDK
  -> Ad network SDK gọi server
  -> Server trả fill/no fill/error
  -> SDK cache creative nếu có fill
  -> Callback load success/fail về game
  -> Game gọi show
  -> Impression/revenue callback
```

Một số điểm quan trọng:

- `load success` nghĩa là SDK đã có quảng cáo sẵn sàng để show, không đảm bảo user sẽ xem impression.
- `show` mới là lúc quảng cáo render lên màn hình.
- `paid/revenue/impression callback` thường đến gần thời điểm impression được ghi nhận.
- Rewarded ads có callback riêng cho việc user đủ điều kiện nhận reward; không nên chỉ dựa vào callback close.

## 5. Mediation phân phối ads như thế nào?

### 5.1 Waterfall
Waterfall là mô hình thử lần lượt các ad source theo thứ tự ưu tiên hoặc eCPM floor.

Ví dụ:

```text
Ad request
  -> Network A instance, floor $20
  -> Nếu no fill thì thử Network B, floor $15
  -> Nếu no fill thì thử Network C, floor $10
  -> Nếu có fill thì cache/show ad
```

Ưu điểm:

- Dễ hiểu, dễ kiểm soát.
- Phù hợp khi team Monet muốn ép ưu tiên theo network/country/placement.

Nhược điểm:

- Có thể bỏ lỡ network trả giá cao hơn nếu nó nằm thấp trong waterfall.
- Cần tối ưu instance/floor thủ công.
- Dễ tăng latency nếu waterfall dài.

### 5.2 Bidding
Bidding là mô hình nhiều ad source trả giá gần như cùng lúc cho một request. Mediation chọn bid tốt nhất theo rule của platform.

```text
Ad request
  -> Gửi bid request tới nhiều bidding source
  -> Nhận bid response
  -> Chọn source thắng
  -> Load/show ad
```

Ưu điểm:

- Cạnh tranh theo từng impression.
- Ít cần tạo nhiều waterfall instance thủ công.
- Thường giảm latency so với waterfall dài.

Nhược điểm:

- Cần network hỗ trợ bidding.
- Debug có thể khó hơn vì quyết định phân phối thay đổi theo từng request.

### 5.3 Hybrid
Nhiều setup thực tế dùng hybrid: bidding chạy cùng waterfall. Bidding source cạnh tranh với các waterfall line item theo giá trị eCPM/bid.

Khi debug, cần hỏi rõ team Monet:

- Ad unit này đang dùng bidding, waterfall hay hybrid?
- Network nào đang enable cho country/platform đang test?
- Có segment, frequency cap, pacing hoặc app version rule nào không?
- eCPM floor có quá cao nên no fill không?

## 6. Vì sao conflict dependency xảy ra?

Conflict xảy ra khi dependency graph không tìm được một bộ version native SDK tương thích cho toàn bộ project.

Các pattern thường gặp:

### 6.1 Hai mediation cùng kéo một ad network SDK khác version
Ví dụ:

```text
GoogleMobileAdsMediationMoloco -> MolocoSDKiOS 4.6.0
IronSourceMolocoAdapter        -> MolocoSDKiOS 4.5.0
```

Cách xử lý thường là nâng adapter thấp hơn lên version tương thích với SDK cao hơn, sau khi đọc changelog và compatibility table.

### 6.2 Hai ad network cùng phụ thuộc một SDK nền khác version
Ví dụ một adapter phụ thuộc `GoogleAppMeasurement` version A, adapter khác phụ thuộc version B. Android/Gradle đôi khi tự chọn được version, nhưng iOS/CocoaPods có thể fail nếu constraints không giao nhau.

### 6.3 Duplicate mediation SDK
Không nên để cùng lúc nhiều bản SDK chính hoặc nhiều adapter cùng chức năng bị import bằng nhiều đường:

- `.unitypackage` cũ vẫn còn trong `Assets`.
- UPM package mới đã được cài thêm.
- Native dependency vừa nằm trong `Assets/Plugins/Android`, vừa được Gradle kéo từ Maven.
- iOS framework vừa embed thủ công, vừa được CocoaPods tải.

### 6.4 Adapter và SDK gốc không cùng compatibility matrix
Adapter thường chỉ support một range version nhất định của SDK gốc. Nếu ép version SDK gốc quá mới hoặc quá cũ, có thể build pass nhưng runtime crash/no fill/callback sai.

## 7. Quy trình xử lý conflict nên dùng

### 7.1 Xác định dependency đang đến từ đâu
Android:

- Xem các file `Dependencies.xml`.
- Xem `mainTemplate.gradle`, `launcherTemplate.gradle`, `baseProjectTemplate.gradle` nếu project có custom Gradle template.
- Xem Gradle dependency tree nếu có thể chạy Gradle trong exported Android project.
- Kiểm tra Gradle cache để biết artifact thực tế đã tải.

iOS:

- Xem các file `Dependencies.xml`.
- Xem `Podfile`.
- Xem `Podfile.lock` để biết version thực tế CocoaPods đã chọn.
- Xem lỗi cụ thể từ `pod install`.

### 7.2 Tìm compatibility table/changelog chính thức
Không nên sửa version theo cảm tính. Hãy kiểm tra:

- Adapter version yêu cầu SDK gốc version nào.
- Mediation SDK version hiện tại support adapter version nào.
- Network SDK có breaking change không.
- Version mới có yêu cầu min SDK, min iOS, Android Gradle Plugin, Kotlin, Swift hoặc privacy manifest mới không.

### 7.3 Chọn một nguồn sự thật cho version
Với mỗi network, nên có một nơi kiểm soát version chính:

- Dùng dependency file do plugin/adapter cung cấp nếu mọi thứ tương thích.
- Nếu cần override, ghi rõ lý do, ngày sửa và link changelog.
- Tránh sửa cả `Dependencies.xml`, `Podfile` và template Gradle theo nhiều hướng khác nhau mà không có ghi chú.

### 7.4 Clean resolve có kiểm soát
Khi đã chỉnh version:

- Resolve lại dependency.
- Kiểm tra artifact thực tế được tải.
- Build Android/iOS.
- Mở debug tool của mediation.
- Test load/show/impression/reward callback trên thiết bị thật.

## 8. Checklist debug no fill hoặc không load được ads

- App id, SDK key, ad unit id, bundle id/package name đã đúng dashboard chưa?
- App đang bật test mode hoặc test device chưa?
- Consent/GDPR/ATT có chặn load ads không?
- Mediation SDK đã initialize thành công chưa?
- Adapter có được cài và initialize thành công không?
- Debugger/Test Suite có báo missing SDK, missing adapter hoặc version mismatch không?
- Country đang test có được enable trong mediation group không?
- Placement/ad unit có bị frequency cap hoặc pacing không?
- Waterfall floor có quá cao không?
- Network dashboard có approve app chưa?
- Device/log có báo no fill thật hay lỗi network/dependency/consent?

## 9. Tài liệu chính thức nên đọc

- Google AdMob Unity quick start: https://developers.google.com/admob/unity/quick-start
- Google AdMob mediation: https://developers.google.com/admob/unity/mediate
- Google AdMob response info: https://developers.google.com/admob/unity/response-info
- Google AdMob Ad Inspector: https://developers.google.com/admob/unity/ad-inspector
- External Dependency Manager for Unity: https://github.com/googlesamples/unity-jar-resolver
- AppLovin MAX Unity integration: https://developers.applovin.com/en/max/unity/overview/integration/
- AppLovin MAX mediation debugger: https://developers.applovin.com/en/max/unity/testing-networks/mediation-debugger/
- AppLovin MAX mediated network guides: https://developers.applovin.com/en/max/unity/mediation/
- Unity LevelPlay Unity integration: https://docs.unity.com/en-us/grow/levelplay/sdk/unity/integration
- Unity LevelPlay Test Suite: https://docs.unity.com/en-us/grow/levelplay/sdk/unity/integration-test-suite
- Unity LevelPlay mediation adapters: https://docs.unity.com/en-us/grow/levelplay/sdk/unity/network-manager
- Unity waterfall placements: https://docs.unity.com/en-us/grow/dashboard/waterfalls
- Google AdMob Unity Ads mediation: https://developers.google.com/admob/unity/mediation/unity
- CocoaPods dependency syntax: https://guides.cocoapods.org/using/the-podfile.html
- Gradle dependency management: https://docs.gradle.org/current/userguide/declaring_dependencies.html

## 10. Ghi chú về Unity waterfall placements

Unity đã thông báo waterfall placements đang được phase out. Bắt đầu từ ngày 23/07/2026, bạn không thể tạo waterfall placements mới; các placement chưa chuyển sang bidding hoặc archived sẽ không còn edit được nhưng vẫn tiếp tục serve ads theo thông báo hiện tại của Unity. Tại thời điểm viết tài liệu này là 20/07/2026, mốc này chỉ còn rất gần. Nếu project vẫn phụ thuộc vào Unity waterfall placements, cần trao đổi sớm với Monet để có kế hoạch chuyển sang in-app bidding hoặc cấu hình mediation phù hợp theo hướng dẫn mới nhất.

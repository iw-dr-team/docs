# Advertising Note
## Lời nói đầu
Đây không phải tài liệu dạy bạn cách tích hợp quảng cáo vào Unity, vì các mediation SDK đều đã có tài liệu hướng dẫn đầy đủ.
Tài liệu này chỉ nhằm mục đích cung cấp những lưu ý quan trọng khi bạn làm việc với quảng cáo trong Unity, giúp bạn đi nhanh hơn và tránh những sai lầm phổ biến.

## 1. Tài liệu về quảng cáo trong Unity

### 1.1 Google mobile ads
Mọi người còn hay gọi là Admob.
- [Repository](https://github.com/googleads/googleads-mobile-unity/)
- [Release](https://github.com/googleads/googleads-mobile-unity/releases)
- [Issues](https://github.com/googleads/googleads-mobile-unity/issues)

Để tích hợp admob vào unity bạn có thể dùng các các cách sau:
- Cách 1: Dùng Unity Package Manager (UPM) để cài đặt
Đăng ký scope cho UPM:
```json
{
  "scopedRegistries": [
    {
      "name": "OpenUPM",
      "url": "https://package.openupm.com",
      "scopes": [
        "com.google"
      ]
    }
  ]
}
```
sau đó vào Package Manager, chọn My Registries, tìm Google Mobile Ads for Unity và install.
- Cách 2: Download file .unitypackage từ release và import vào project của bạn.
- Cách 3: mở manifest.json của project và thêm dòng sau vào dependencies:
```json
"com.google.ads.mobile": "https://github.com/googleads/googleads-mobile-unity.git?path=packages/com.google.ads.mobile#v11.0.0",      
```

Sau khi install xong thì bạn có thể xem [docs](https://developers.google.com/admob/unity/quick-start?hl=vi) và [sample code](https://github.com/googleads/googleads-mobile-unity/tree/main/samples/HelloWorld/Assets/Scripts) 
để tham khảo và tích hợp vào project của bạn.

### 1.2 AppLovin Max

- [Repository](https://github.com/AppLovin/AppLovin-MAX-Unity-Plugin)
- [Release](https://github.com/AppLovin/AppLovin-MAX-Unity-Plugin/releases)
- [Issues](https://github.com/AppLovin/AppLovin-MAX-Unity-Plugin/issues)

Để tích hợp AppLovin Max vào Unity bạn có thể download file .unitypackage từ release và import vào project của bạn.
Sau khi import xong thì bạn có thể xem [docs](https://support.applovin.com/en/max/unity/overview/integration) và [sample code](https://github.com/AppLovin/AppLovin-MAX-Unity-Plugin/tree/master/DemoApp)
để tham khảo và tích hợp vào project của bạn.

### 1.3 LevelPlay

- [Repository](https://github.com/ironsource-mobile/Unity-sdk)
- [Release](https://github.com/ironsource-mobile/Unity-sdk/releases)
- [Changelog](https://docs.unity.com/en-us/grow/levelplay/sdk/android/changelog)

Để tích hợp LevelPlay vào Unity bạn có thể download file .unitypackage từ release và import vào project của bạn hoặc dùng Unity Package Manager (UPM) để cài đặt AdsMediation (com.unity.services.levelplay).

## 2. Lưu ý khi làm việc với quảng cáo trong Unity

### 2.1 Lưu ý trước khi build test
- Trước khi thực hiện build test hãy lưu ý các vấn đề sau:

***Đối với Admob***
- [ ] Admob có hỗ trợ [ad unit id test](https://developers.google.com/admob/unity/test-ads?hl=en)
- [ ] Admob hỗ trợ tính năng [Ad Inspector](https://developers.google.com/admob/unity/ad-inspector?hl=en) để debug quảng cáo trong quá trình test.
- [ ] Nếu sử dụng GDPR của admob (thường trong logic code sẽ có điều kiện show GDPR consent thỏa mãn thì mới load ads) thì bạn cần bật GDPR message trong Admob dashboard để test, nếu không bật thì sẽ không show GDPR => không load được ads.

***Đối với Applovin Max***
- [ ] Applovin Max không có hỗ trợ ad unit id test giống Admob
- [ ] Applovin Max có hỗ trợ tính năng [Mediation Debugger](https://support.applovin.com/en/max/unity/testing-networks/mediation-debugger) để debug quảng cáo trong quá trình test.
- [ ] Sẽ không có ads trả về nếu bundle id trong project không khớp với bundle id trong dashboard của Applovin Max, hãy kiểm tra kỹ trước khi build test.

***Đối với LevelPlay**



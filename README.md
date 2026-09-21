# Unity Mobile Production Playbook (Dancing Road documentation)

|STT|Docs|
|---|----|
|1  |[Release DR1](CheckListReleaseDR1.md)|
|2  |[Advertising Note](AdvertisingNote.md)|
|3  |[Advertising Deep Dive](AdvertisingDeepDive.md)|
|4  |[FirebaseMCP with Codex documentation and guide](https://github.com/iw-dr-team/docs/blob/main/FirebaseMcpWithCodex/FirebaseMCP-with-Codex-documentation-and-guide.md)|
|5  |[Firebase debug view on device ios](https://github.com/unity-package/firebase-debugview-ios)|
|6  |[GuideToIntegratingCodexWithLLMProxy](https://github.com/iw-dr-team/docs/blob/main/GuideToIntegratingCodexWithLLMProxy)|

# From Personal Experience to Team Knowledge

Chào tất cả anh em, tôi là Đức (developer của team Dancing Road). Thời điểm bạn đọc tài liệu này có thể tôi vẫn còn làm việc tại Dancing Road hoặc đã rời đi.
Nhưng tôi vẫn muốn chia sẻ những kinh nghiệm của mình trong quá trình làm việc tại đây. Hãy đọc tiếp để khám phá nhé!

## 1. WHY - Tại sao tôi viết tài liệu này?

Trong quá trình làm việc tại Dancing Road, tôi nhận thấy team thường xuyên gặp những vẫn đề lặp đi lặp lại trong quá trình vận hành và phát triển game.
Những vấn đề này thường được giải quyết bằng cách trao đổi trực tiếp giữa các thành viên trong team và sau một thời gian sẽ bị lãng quên. Tất nhiên, những vấn đề này sẽ lại
xuất hiện trong tương lai và team lại gặp khó khăn trong quá trình xử lý một lần nữa.

Với triết lý:
> Không tập trung vào câu hỏi:
>
> **"Tôi học được gì?"**

> Mà tập trung vào:
>
> **"Làm sao để người khác không phải gặp lại vấn đề mà mình đã gặp?"**

Mục tiêu của Unity Mobile Production Playbook không đơn thuần là tài liệu, nó còn là ***chuyển đổi kinh nghiệm cá nhân thành tài sản chung của team***.
Khi một thành viên rời đi, những kinh nghiệm của họ sẽ không bị mất đi mà sẽ lưu trữ trong tài liệu này. Những thành viên mới sẽ không phải mất thời gian để tìm hiểu lại những vấn đề mà team đã gặp phải.

Với tinh thần tốt đẹp này, tôi hi vọng tài liệu sẽ được contribute bởi tất cả các thành viên trong team, để mọi người cùng nhau xây dựng một tài sản chung cho team.

## 2. HOW - Tôi đã thực hiện thế nào?

Tôi bắt đầu với một số vấn đề mà tôi gặp phải trong quá trình làm việc:

### Checklist Release DR1

Khi tôi nhận nhiệm vụ build release, tôi nhận thấy có nhiều bước cần thực hiện và nhiều vấn đề cần kiểm tra.
Và thật dễ dàng để bỏ sót một bước bất kỳ nào đó. Vì thế tôi đã tạo ra một checklist để đảm bảo rằng mọi bước đều được thực hiện đầy đủ và chính xác.

- [ ] Xử lý branch github theo yêu cầu
- [ ] Setup + build trên Unity
- [ ] Kiểm tra sau khi build và distribute
- [ ] Xử lý sau khi release
- [ ] Đẩy debugging symbols lên firebase crashlytic

Sau khi tạo ra checklist này, tôi nhận thấy rằng việc bị miss các bước không còn xảy ra nữa. Và tôi cũng gửi checklist này đến các thành viên khác trong team và các dev khác trong team
đều có thể build đúng chuẩn yêu cầu.

### Advertising Note

Với kinh nghiệm làm việc với SDK quảng cáo, tôi nhận thấy rằng việc tích hợp SDK quảng cáo vào game không khó khăn, nhưng việc hiểu và xử lý lỗi liên quan đến SDK thì lại không đơn giản.
Vì thế tôi tạo ra một tài liệu với khẩu hiệu:
> Đây không phải tài liệu dạy bạn cách tích hợp quảng cáo vào Unity, vì các mediation SDK đều đã có tài liệu hướng dẫn đầy đủ.
> 
> Tài liệu này chỉ nhằm mục đích cung cấp những lưu ý quan trọng khi bạn làm việc với quảng cáo trong Unity, giúp bạn đi nhanh hơn và tránh những sai lầm phổ biến.

Trong tài liệu nói về:
- Tài liệu về các mediation lấy ở đâu, khi gặp bug thì cần tìm hiểu tài liệu nào
- Install thế nào để tiện update
- Các lưu ý trước khi build test
- Các lưu ý trước khi update version SDK
- Lưu ý về callback
- Kiểm tra tracking ad_impression
- Cách quảng cáo được load về và phân phối từ ad network
- Nguyên nhân conflict giữa các ad network và cách giải quyết

Tôi đã gặp nhiều dev trong công ty và cộng đồng, riêng việc xử lý conflict giữa các ad network luôn là vấn đề khó khăn và tốn thời gian giải quyết mỗi khi update version sdk.
Và tôi muốn chia sẻ về gốc rễ vấn đề này, để các dev có thể hiểu và fix một cách chính xác hơn, nhanh chóng hơn.

### Firebase

Với các dev ở level junior cũng đã có những kinh nghiệm làm việc với các dịch vụ firebase cơ bản. Vì thế tôi không viết lại tài liệu hướng dẫn sử dụng firebase cơ bản, 
mà tôi tập trung vào những thứ mang lại hiệu suất cao hơn cho dev, giúp tiết kiệm thời gian.

- FirebaseMCP with Codex documentation and guide: Tài liệu hướng dẫn sử dụng FirebaseMCP với Codex, giúp dev tiết kiệm thời gian khi làm việc với firebase.
- Ứng dụng:
  - Lấy ra danh sách 200 parameter từ firebase remote config bị outdate.
  - Lấy kết quả A/B test từ firebase remote config và phân tích kết quả.
  - Lấy ra danh sách crash từ firebase crashlytics và phân tích nguyên nhân.

- Tài liệu hướng dẫn enable debug view trên thiết bị iOS, giúp dev và QC tiện lợi hơn trong quá trình setup, không cần phải setup từ xcode trên máy Mac nữa.

## 3. WHAT - Tôi đã tạo ra được gì?

Kết quả sau một thời gian tìm hiểu và xây dựng tài liệu. Hiện tại, tôi đã tạo ra một bộ [tài liệu](https://github.com/iw-dr-team/docs)

Mặc dù khởi đầu còn sơ khai và chưa đầy đủ, nhưng tầm nhìn của ý tưởng này rất tốt.
Để nối tiếp ý tưởng này, tôi đã có lời mời các thành viên khác trong team cùng nhau đóng góp vào bộ tài liệu này và đã nhận được sự ủng hộ từ họ. Sau một thời gian tôi và các thành viên khác sẽ
tiếp tục contribute để hoàn thiện hơn. 
Không chỉ là kiến thức về kỹ thuật, mà còn là những kinh nghiệm trong quá trình vận hành game, những vấn đề mà team đã gặp phải và cách giải quyết của chúng.
Ví dụ:
- Cách tiếp cận hệ thông remote config trong project: cách đặt tên file và biến -> những yêu cầu bắt buộc để tránh lỗi -> setup trên firebase console -> cách tạo var test.
- Quy trình làm việc với git theo nội quy của team: mã task -> tên branch -> message -> push.

Tôi hi vọng sẽ tạo nên một hiệu ứng dây chuyền về việc viết tài liệu và chia sẻ kiến thức trong team, đồng thời mang lại giá trị lớn về sau cho team.






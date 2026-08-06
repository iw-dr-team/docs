# Report Var Test: DRi_Communication_System_Update

## Tong quan

- Firebase project: `dancing-road`
- Experiment ID: `567`
- Display name: `DRi_Communication_System_Update`
- Trang thai: `PENDING`
- Lan cap nhat gan nhat: `2026-08-05 09:11:16 UTC`
- Service: `EXPERIMENT_SERVICE_REMOTE_CONFIG`

Draft nay mirror dung cau truc cua ban Android `DRa_Communication_System_Update`: cung objective, cung danh sach variant, khac nhau o platform target iOS.

## Objectives

- Primary objective: `retention_1`
- Secondary objective: `retention_7`
- Safety/crash metric: `app_exception` voi `NO_EVENT_USERS`
- Activation event: de trong, tuc viec nhan experiment duoc tinh la activation.

## Variants

Tat ca variant co weight `1`, tuong duong moi variant khoang `12.5%` traffic neu chia deu.

| Variant | Ghi chu |
| --- | --- |
| `Baseline` | Can kiem tra value trong Firebase Console de dam bao baseline khop config live hien tai. |
| `Case 1 - SM = campaign_order` | Test serving mode theo thu tu campaign. |
| `Case 2 - SM = popup_rotation` | Test serving mode xoay vong popup. |
| `US2 - song_start` | Test cooldown theo so lan `song_start`. |
| `US2 - time` | Test cooldown theo thoi gian. |
| `US2 - Both` | Test dong thoi cooldown theo song_start va time. |
| `max_popups_per_sequence` | Test so popup toi da trong mot Home entry. RUI RO UX cao nhat. |
| `US5 - required song` | Test targeting dua tren lich su da choi bai hat. |

## Nhan xet chinh

1. Draft dang tach dung cho iOS (`DRi`) va tuong dong Android (`DRa_Communication_System_Update`), phu hop neu muc tieu la parity giua hai platform.

2. So luong 8 variants la nhieu. Moi nhanh chi nhan khoang `12.5%` traffic, nen can traffic/thoi gian du lon de doc duoc retention. Neu iOS traffic khong lon, nen can nhac tach thanh cac test nho hon.

3. Baseline can duoc kiem tra truc tiep trong Firebase Console. Default cua key `CommunicationSystem_Campaign` trong Remote Config la chuoi rong, trong khi live iOS hien co config duoi condition `iOS since 3.1.0 exclude SA RU FR GB`. Neu Baseline de `no change`/empty sai cach, baseline co the thanh "khong popup", lam ket qua bi lech.

4. Objective `retention_1` hop ly de do tac dong dai han, nhung chua truc tiep do funnel popup. Code hien co tracking `igpopup_eligible`, `igpopup_imp`, `igpopup_click`, `igpopup_click_success`, `igpopup_close`. Nen them `igpopup_imp` hoac `igpopup_click_success` lam secondary objective neu Firebase con slot.

5. Cac variant `US2 - time`, `US2 - song_start`, `US2 - Both` tac dong global frequency gate: `globalCooldownTimeSec`, `globalCooldownSongStartCount`, `dailyPopupCap`, `weeklyPopupCap`. Rui ro chinh la giam impression qua manh, lam retention kho doc neu khong theo doi `igpopup_imp`.

6. `max_popups_per_sequence` la variant rui ro UX cao nhat vi co the hien nhieu popup lien tiep trong cung mot Home entry. Neu tang len > 1, can cap ngay/tuan chat va theo doi close rate.

7. `US5 - required song` phu thuoc song history local qua `requiredPlayedSongId`, `requiredPlayedSongRecentDays`, `requiredPlayedSongMinCount`. Can dam bao song id trong draft la ACM id dung, khong phai local numeric id.

## Kiem tra truoc khi Start

- Mo draft trong Firebase Console va kiem tra value cua parameter `CommunicationSystem_Campaign` cho tung variant, vi Firebase API khong expose draft values.
- Dam bao target condition dung iOS app `1:528923424982:ios:46195f809d709f94`, version `>= 3.1.0`, va country exclusion la chu dich.
- Khong chay dong thoi `DRi_Communication_System_Update` va `DRi_Communication_System_Update_2` neu ca hai cung test `CommunicationSystem_Campaign` tren cung audience.
- Them objective funnel (`igpopup_imp` hoac `igpopup_click_success`) neu con slot.


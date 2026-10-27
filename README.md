# FlymeDuo

Gộp **Wi-Fi, tín hiệu di động và pin** trên thanh trạng thái Flyme thành một biểu tượng ba trong một, theo phong cách “Duo” của iPhone.

FlymeDuo là **bản thích ứng cho Flyme** của [HyperDuo](https://github.com/yixing233/HyperDuo): phần lõi kết xuất (hình học, phối màu, sắp xếp thành phần) giống bản gốc, còn tất cả các điểm chèn phía giao diện hệ thống đều được thay bằng bản triển khai của Flyme: `FlymeBatteryMeterView`, `QSStatusBar`, `KeyguardStatusBarView`, `NotificationPanelView`.

## Những gì đã được thích ứng

### Ba vị trí, một chiếc vòng

- **Thanh trạng thái thông thường**: vòng được vẽ trong ô pin của `system_icons`;
- **Trung tâm điều khiển**: thanh trên cùng của `QSStatusBar` (có thể loại trừ trong phần phạm vi áp dụng);
- **Trung tâm thông báo / phần đầu màn hình khoá**: `keyguard_status_bar` (trong Flyme, nó được include vào bảng thông báo và là cùng một view, nên không bị nhận nhầm là thanh trạng thái).

Các ô pin, Wi-Fi và tín hiệu ở cả ba vị trí đều do mô-đun tiếp quản: khi vẽ vòng, biểu tượng gốc nhường chỗ; khi bị loại trừ, toàn bộ được trả lại cho hệ thống gốc.

### Thu phóng chung: lấy chiều cao thanh trạng thái làm chuẩn

Phép đo của Flyme khoá view pin trong hộp nội tại của biểu tượng pin (khoảng 10dp), làm không gian thiết kế 120 đơn vị của vòng bị ép dẹt. Bản thích ứng viết lại `onMeasure`: **cả ba vị trí đều lấy chiều cao thanh trạng thái (140px ≈ 43dp) làm 100%**. Hàng ở thanh trạng thái thông thường chỉ cao 65px, không đủ chỗ, nên mô-đun mở khoá việc cắt xén đối với chuỗi view cha và cho phép vẽ tràn vào phần dư của cửa sổ. Thanh trượt “Kích thước chung của biểu tượng” (60–100%) trong cài đặt có hiệu lực đồng nhất ở cả ba vị trí.

### Phạm vi áp dụng

**Tất cả / Loại trừ Trung tâm điều khiển / Loại trừ Trung tâm thông báo / Loại trừ cả hai.** Vị trí bị loại trừ giữ nguyên pin và biểu tượng gốc của Flyme, cách hiển thị do bản gốc quyết định. Khu vực được phân loại theo chuỗi view cha thực tế và được xác định lại sau khi gắn vào: Flyme đo view trước khi nó được gắn, nên phân loại quá sớm sẽ khiến mọi vùng bị nhận nhầm là thanh trạng thái (đây là nguyên nhân gốc khiến việc loại trừ không hiệu lực trước bản .33).

### Các tính năng còn lại

Giống bản gốc: số phần trăm pin, cung Wi-Fi, chấm tín hiệu, loại mạng và tia sét sạc trong vòng đều có thể bật/tắt từng mục hoặc trả lại cho hệ thống; bố cục viên nang; thẻ tín hiệu ngoài vòng; nhãn 5G (cỡ chữ tự thích ứng theo độ rộng ô chứa); công suất sạc; nhiệt độ pin. Chi tiết về kiến trúc và triển khai xem [docs/DEVELOPMENT.md](docs/DEVELOPMENT.md) (tài liệu của bản gốc; các điểm chèn của Flyme nằm trong `FlymeHooks.java`).

## Cài đặt

| Hạng mục | Yêu cầu |
| --- | --- |
| Hệ thống | Flyme AIOS 2 (Android 16, API 36); các Flyme cùng thế hệ về lý thuyết cũng dùng được |
| Khung | LSPosed (libxposed API 102+), đã kiểm chứng trên 2.2.1 |

1. Tải `FlymeDuo-*.apk` mới nhất từ [Releases](https://github.com/leyodr9000/FlymeDuo/releases) và cài đặt.
2. LSPosed → Mô-đun → bật **FlymeDuo**, trong phạm vi chọn **Giao diện hệ thống (`com.android.systemui`)**.
3. Khởi động lại Giao diện hệ thống:

```sh
adb shell "su -c 'pkill -f com.android.systemui'"
```

Nếu sau khi `pkill` thanh trạng thái trở về như gốc: LSPosed tính **mỗi lần tiến trình Giao diện hệ thống khởi động lại** là một lần “crash”, tích luỹ 4 lần sẽ vào chế độ an toàn của mô-đun. Chỉ cần vào LSPosed chọn lại mô-đun, không phải mô-đun bị lỗi.

## Gỡ lỗi

```powershell
adb logcat -v time -s LSPosedFramework:* | Select-String HyperDuo
```

Mọi đầu ra của mô-đun trong Giao diện hệ thống đều do khung ghi hộ (tag `LSPosedFramework`). Dòng khởi động `flyme adapter build x.y.z, hooks=N` xác nhận lớp thích ứng đã được nạp; mỗi máy chủ pin sẽ in `battery region=…` (thuộc vùng nào) và `battery box …` (chiều cao hàng, độ dài cạnh, kết quả xét loại trừ). Hai dòng này giúp xác định ngay lỗi “không có hiệu lực / sai kích thước” thuộc loại nào.

## Phát triển

```sh
gradle assembleRelease -PhyperduoVersionName=1.7.5-flyme.38 -PhyperduoVersionCode=10742
```

Nhánh `flyme-adapter`, đồng bộ với bản gốc bằng rebase. Phần lõi kết xuất (`TrioRenderer` / `TrioGeometry` / `TrioConfig`) cố gắng không đụng tới; mọi khác biệt của Flyme đều gom vào `FlymeHooks.java`.

## Lời cảm ơn

- **[HyperDuo](https://github.com/yixing233/HyperDuo)**: nền tảng cho toàn bộ bản fork này
- **知秋, thành viên nhóm FlymeBarSizer**: tham khảo thích ứng cho Flyme
- Giao diện cài đặt dựa trên Compose + [Miuix](https://github.com/miuix-kotlin-multiplatform/miuix)

## Giấy phép

**GNU General Public License v3.0 (GPL-3.0)**, toàn văn xem [LICENSE](LICENSE).

Copyright (C) 2026 yixing233 và những người đóng góp cho HyperDuo; phần FlymeDuo sửa đổi cũng được phát hành theo GPL-3.0. Khi phân phối APK đã biên dịch, cần kèm theo mã nguồn tương ứng, trỏ tới tag của phiên bản đó trong kho này là đủ. Không có bảo hành.

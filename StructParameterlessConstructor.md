# Vì sao struct trong C# < 10 không có constructor không tham số?

## Lời nói đầu
Khi viết struct trong Unity, chắc hẳn nhiều bạn đã từng gặp lỗi:

```
error CS0568: Structs cannot contain explicit parameterless constructors
```

Tài liệu này giải thích ngắn gọn **vì sao C# cấm điều đó** và **nên làm gì thay thế**.

## 1. Vấn đề bắt đầu từ đâu?

Giả sử C# cho phép viết như sau:

```csharp
struct Health
{
    public int Value;

    public Health()      // giả sử được phép
    {
        Value = 100;     // mong muốn: máu mặc định là 100
    }
}
```

Trông rất hợp lý. Nhưng hãy xem điều gì xảy ra:

```csharp
var a = new Health();          // Value = 100  -> gọi constructor
var b = default(Health);       // Value = 0    -> KHÔNG gọi constructor
var arr = new Health[3];       // [0, 0, 0]    -> KHÔNG gọi constructor

class Enemy
{
    public Health hp;          // hp.Value = 0 -> KHÔNG gọi constructor
}
```

Cùng là "một con Health mới", nhưng có chỗ thì máu là **100**, có chỗ thì máu là **0**. Enemy vừa spawn đã chết ngay 💀.

## 2. Vì sao lại có giá trị 0 mà không qua constructor?

Nguyên nhân gốc: **struct là kiểu tham trị (value type)**.

| | class (tham chiếu) | struct (tham trị) |
|---|---|---|
| Biến chứa gì? | Con trỏ tới object | Chính dữ liệu |
| `new T[3]` cho ra | `[null, null, null]` | 3 giá trị thật |
| Giá trị "trống" | `null` | Toàn bộ field = 0 |

Với class, phần tử mảng chỉ là `null`, nên không cần tạo object nào cả.

Với struct, ngay khi cấp phát vùng nhớ là **phải có sẵn giá trị thật**. Runtime tạo các giá trị này bằng cách **xóa vùng nhớ về 0** (giống `memset`), và **không gọi constructor**. Cách này cực nhanh: tạo mảng 1 triệu phần tử chỉ là một lần xóa bộ nhớ, thay vì 1 triệu lần gọi hàm.

> Hình dung: struct giống một **khay đựng đồ**. Mỗi lần có khay mới, nhà máy luôn giao **khay rỗng** (giá trị 0). Constructor giống việc **bạn tự tay xếp đồ vào khay**, nhưng nhà máy không bao giờ chờ bạn làm việc đó.

## 3. Quyết định của C#

Nếu cho phép constructor không tham số, sẽ tồn tại **hai giá trị "mặc định" khác nhau**:
- `new Health()` → 100
- `default(Health)`, mảng, field → 0

Hai giá trị mâu thuẫn này là nguồn bug rất khó phát hiện. Vì vậy C# < 10 chọn cách **cấm luôn**, để đảm bảo:

```csharp
new Health() == default(Health)   // luôn đúng, mọi field = 0
```

> Lưu ý: đây là **quyết định thiết kế của ngôn ngữ C#**, không phải giới hạn của runtime. CLR vẫn hỗ trợ (C++/CLI dùng được), và từ **C# 10** C# đã cho phép. Tuy nhiên `default(T)`, mảng và field **vẫn là toàn 0**, vấn đề ở mục 1 vẫn còn nguyên.

## 4. Unity thì sao?

Unity hiện chỉ hỗ trợ **C# 9**, nên vẫn chưa viết được constructor không tham số cho struct.

## 5. Nên làm gì thay thế?

### Cách 1 (khuyến nghị): Thiết kế để giá trị 0 là hợp lệ

Thay vì lưu "máu còn lại", hãy lưu "máu đã mất":

```csharp
struct Health
{
    public const int Max = 100;
    public int Damage;                   // mặc định 0 = chưa mất máu

    public int Value => Max - Damage;    // mặc định = 100
}

var arr = new Health[3];  // tất cả đều có Value = 100 ✅
```

Như vậy `default`, mảng, field đều cho ra trạng thái đúng mà không cần constructor.

### Cách 2: Dùng static factory

```csharp
struct Health
{
    public int Value;

    public Health(int value) { Value = value; }

    public static Health Default => new Health(100);
}

var hp = Health.Default;  // Value = 100
```

Nhược điểm: mảng và field chưa gán vẫn là 0, người dùng phải nhớ gọi `Health.Default`.

### Cách 3: Dùng class nếu thực sự cần constructor

Nếu object có invariant phức tạp và bắt buộc phải khởi tạo, có thể struct không phải lựa chọn phù hợp.

## 6. Vì sao C# >= 10 lại cho phép?

Vấn đề `default` vẫn còn, nhưng các lý do khiến C# phải cấm đã không còn đủ mạnh:

- **Bug runtime đã được sửa:** lý do khiến C# 6 phải gỡ tính năng này là `Activator.CreateInstance<T>()` (và `new T()` trong generic) trên .NET Framework không gọi constructor. .NET Core/.NET 5+ đã sửa lỗi này.
- **Record struct ra đời (C# 10):** record struct cần khởi tạo field giống class. Nếu vẫn cấm constructor không tham số thì record struct sẽ rất khó dùng.
- **Cho phép khởi tạo field trực tiếp:** C# 10 cũng cho viết `public int Value = 100;` trong struct. Tính năng này cần một constructor để chạy phần khởi tạo đó.
- **Dev thực sự cần:** nhiều struct muốn `new S()` cho ra trạng thái hợp lệ. Đội C# chấp nhận đánh đổi: **cho phép, nhưng ghi rõ quy tắc**.

Quy tắc trong C# 10:

```csharp
struct Health
{
    public int Value = 100;   // C# 10: khởi tạo field trực tiếp
    public Health() { }       // C# 10: constructor không tham số
}

var a = new Health();        // Value = 100 -> gọi constructor
var b = default(Health);     // Value = 0   -> vẫn KHÔNG gọi constructor
var arr = new Health[3];     // [0, 0, 0]   -> vẫn KHÔNG gọi constructor
```

Tức là C# 10 **không giải quyết** vấn đề ở mục 1. C# 10 chỉ chuyển trách nhiệm cho dev: **`new S()` và `default(S)` giờ có thể khác nhau, bạn phải tự hiểu và tự chịu.** Lời khuyên ở mục 5 vẫn giữ nguyên giá trị.

## Tóm tắt

- Struct là **kiểu tham trị**, nên luôn có sẵn giá trị mặc định **toàn 0**, được tạo **không qua constructor**.
- C# < 10 cấm constructor không tham số để `new S()` **không thể khác** `default(S)`.
- C# >= 10 cho phép vì bug runtime đã được sửa và để hỗ trợ record struct cùng việc khởi tạo field, nhưng `default(S)` **vẫn là toàn 0**.
- Khi viết struct, hãy thiết kế để **trạng thái toàn 0 là trạng thái hợp lệ**.

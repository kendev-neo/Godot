# PHẦN 3. NÂNG CAO VÀ KIẾN TRÚC

## Chương 18. Cấu trúc mã nguồn, kiểm thử và công cụ \[Nâng cao\]

> [!NOTE]
> **Mục tiêu:** tổ chức project để mở rộng được, viết được kiểm thử cho logic thuần, dùng công cụ để bắt lỗi sớm.

### 18.1 Tách logic thuần khỏi node

Nguyên tắc quan trọng nhất cho khả năng bảo trì và kiểm thử: **đặt luật chơi và tính toán vào class C# thuần** (không kế thừa Node, không gọi API Godot), và để node chỉ làm nhiệm vụ **nối** logic với engine (nhận input, hiển thị, phát âm thanh). Ví dụ: tính sát thương, bảng tương khắc, kinh tế, sinh bản đồ, thuật toán tìm đường nằm trong class thuần; node `Soldier` chỉ gọi chúng.

Lợi ích: kiểm thử được **không cần mở Godot**, chạy nhanh, tái sử dụng, và có thể chạy hàng trăm trận mô phỏng không cần vẽ.

### 18.2 Cấu trúc và quy ước

```
MyGame/
  scripts/
    Core/        logic thuần (không dùng Godot)
    Entities/    node: Player, Enemy...
    UI/
    Autoload/
  scenes/  resources/  assets/
```

- **Namespace** theo tính năng (`MyGame.Combat`, `MyGame.Inventory`), tránh trùng tên.
- **Quy ước đặt tên** của C#: `PascalCase` cho class/method/property/enum, `camelCase` cho biến cục bộ và tham số, `_camelCase` cho field private, tiền tố `I` cho interface (`IDamageable`). Giữ **nhất quán** quan trọng hơn chọn quy ước nào. Dùng file `.editorconfig` để Rider/VS Code tự áp dụng.
- Mỗi file một class chính; tên file bằng tên class (bắt buộc với script Godot).
- Khi dự án lớn, tách phần thuần thành **class library** riêng được tham chiếu từ project Godot, để project kiểm thử cũng tham chiếu được mà không cần Godot.

### 18.3 Kiểm thử

Kim tự tháp kiểm thử: **nhiều** test nhỏ cho logic thuần, **ít hơn** test tích hợp node, **rất ít** test chơi thật thủ công. Với logic thuần dùng **xUnit** hoặc **NUnit**:

```csharp
public static class DamageCalc
{
    public static float Compute(float baseDamage, float multiplier, float armor)
        => MathF.Max(1f, baseDamage * multiplier - armor);   // luôn gây ít nhất 1 sát thương
}

public class DamageCalcTests
{
    [Fact]
    public void Never_below_one()
        => Assert.Equal(1f, DamageCalc.Compute(5f, 1f, 100f));

    [Theory]
    [InlineData(10, 1.5f, 0, 15)]
    [InlineData(10, 0.5f, 2, 3)]
    public void Applies_multiplier_then_armor(float b, float m, float a, float expected)
        => Assert.Equal(expected, DamageCalc.Compute(b, m, a));
}
```

Thiết lập nhanh: `dotnet new xunit -n Tests`, thêm tham chiếu tới project chứa logic thuần, chạy `dotnet test`. Mẫu viết test: **Arrange** (chuẩn bị), **Act** (thực hiện), **Assert** (kiểm tra).

**Làm code dễ kiểm thử:** truyền phụ thuộc thay vì dùng singleton; **tiêm nguồn ngẫu nhiên và thời gian** (nhận `RandomNumberGenerator`/giá trị `dt` làm tham số) để kết quả xác định; tránh logic nằm sâu trong callback của engine.

Với test cần node và engine, dùng framework như **GdUnit4** (hỗ trợ C#) để chạy cảnh trong Godot. Chỉ viết loại test này cho những chỗ thực sự cần.

### 18.4 Công cụ chất lượng

- **Trình biên dịch:** bật nullable (chương 6), coi cảnh báo là lỗi cho các cảnh báo quan trọng; đọc cảnh báo thay vì bỏ qua.
- **Analyzers:** Roslyn analyzers trong Rider/VS Code chỉ ra lỗi phong cách và lỗi tiềm ẩn.
- **Định dạng:** `dotnet format` giữ phong cách nhất quán; đặt trong quy trình trước khi commit.
- **Git** (tài liệu chính, Phụ lục D.5): commit nhỏ, thông điệp rõ, nhánh cho thử nghiệm.
- **Review checklist tự dùng:** có null chưa được kiểm tra? có đăng ký event chưa huỷ? có cấp phát trong `_Process`? có node đã xoá còn được giữ? có magic number nên đưa vào `[Export]`/hằng?

### 18.5 Gỡ lỗi và ghi log

- **Debugger gắn vào Godot** (Rider/VS Code): đặt breakpoint, xem biến, xem call stack; hiệu quả hơn nhiều so với `GD.Print`.
- Log có mức: `GD.Print` (thông tin), `GD.PushWarning`, `GD.PushError`. Thêm tiền tố hệ thống (`[Combat]`) để lọc.
- **Biên dịch có điều kiện:** `#if DEBUG ... #endif`, `[Conditional("DEBUG")]`, hoặc `OS.IsDebugBuild()` để chỉ chạy code gỡ lỗi (vẽ hitbox, log chi tiết) ở bản debug.
- Công cụ trong game: bật **Visible Collision Shapes**, **Visible Navigation**, và vẽ debug bằng `_Draw()` + `QueueRedraw()`.

### 18.6 Ghi chép để dự án sống lâu

`README.md` (cách chạy, cấu trúc, quy ước), danh sách **quyết định kiến trúc** (vì sao chọn cách này) và danh sách lỗi đã biết. Chỉ vài dòng mỗi thứ, nhưng cứu bạn khi quay lại dự án sau vài tuần.

### 18.7 Lỗi thường gặp

- Trộn logic luật chơi vào node, không thể test.
- Test dùng ngẫu nhiên/thời gian thật nên kết quả chập chờn.
- Bỏ qua cảnh báo trình biên dịch.
- Dùng `GD.Print` thay cho debugger với lỗi khó.
- Không có README, quay lại sau vài tuần không còn nhớ gì.

### 18.8 Tự kiểm tra

1. Vì sao nên tách logic thuần khỏi node? Lợi ích cụ thể?
2. Quy ước đặt tên của C# cho class, field private, interface, tham số?
3. Kim tự tháp kiểm thử gồm những tầng nào? Test nào nhiều nhất?
4. Làm sao để một hàm có ngẫu nhiên vẫn test được chắc chắn?
5. Biên dịch có điều kiện dùng để làm gì?
6. Danh sách review tự dùng của bạn gồm những mục nào?

### 18.9 Bài tập

1. Tạo project `dotnet new xunit`, test hai hàm thuần (tính sát thương, tính phần trăm máu) bằng `[Fact]` và `[Theory]`.
2. Lấy một script có logic trộn lẫn, tách phần tính toán thành class thuần và viết test cho nó.
3. Thêm `.editorconfig` với quy ước đặt tên, kiểm tra Rider cảnh báo khi vi phạm.
4. Gắn debugger vào Godot, đặt breakpoint trong `_PhysicsProcess` và theo dõi `Velocity`.

---

**Tiếp theo:** [Question](question.md)

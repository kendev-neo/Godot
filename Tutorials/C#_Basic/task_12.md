# PHẦN 2. C# TRONG GODOT

## Chương 12. Signal và event trong Godot \[Trung bình\]

> [!NOTE]
> **Mục tiêu:** khai báo, phát, kết nối, ngắt kết nối signal đúng; hiểu thứ tự gọi và các bẫy; biết khi nào dùng signal, khi nào dùng event C#.

### 12.1 Signal là gì

Signal là cơ chế **quan sát (observer)** của engine: một node **phát** tín hiệu khi có chuyện xảy ra, các node khác **đã nối** sẽ được gọi. Bên phát **không cần biết** bên nghe là ai. Signal hiện trong editor (tab Node > Signals) và nối được với cả GDScript.

### 12.2 Khai báo, phát, nối, ngắt

```csharp
// Khai báo: delegate PHẢI kết thúc bằng EventHandler; tham số phải là kiểu tương thích Variant
[Signal] public delegate void HealthChangedEventHandler(int current, int max);
[Signal] public delegate void DiedEventHandler();

// Phát
EmitSignal(SignalName.HealthChanged, _hp, MaxHp);
EmitSignal(SignalName.Died);

// Nối
health.HealthChanged += OnHealthChanged;            // signal khai báo bằng C#, hoặc có sẵn của node
button.Pressed += OnPressed;

// Nối dạng tổng quát (cần khi nối tới GDScript hoặc cần cờ)
health.Connect(Health.SignalName.Died, Callable.From(OnDied),
               (uint)GodotObject.ConnectFlags.OneShot);   // chỉ gọi một lần

// Ngắt
health.HealthChanged -= OnHealthChanged;
bool linked = health.IsConnected(Health.SignalName.Died, Callable.From(OnDied));
```

Cờ hay dùng: `OneShot` (tự ngắt sau lần gọi đầu), `Deferred` (gọi vào cuối frame, xem 12.4).

### 12.3 Signal Godot và event C#

|                                   | Signal (`[Signal]`)                      | `event` C#                                    |
| --------------------------------- | ---------------------------------------- | --------------------------------------------- |
| Hiện trong editor, nối bằng chuột | **có**                                   | không                                         |
| Nối với GDScript                  | **có**                                   | không                                         |
| Kiểu tham số                      | giới hạn kiểu tương thích Variant        | **bất kỳ** kiểu C#                            |
| Tốc độ                            | chậm hơn (qua engine)                    | nhanh                                         |
| Tự ngắt khi node bị xoá           | **có** (Godot lo)                        | **không**, bạn phải `-=`                      |
| Nên dùng khi                      | giao tiếp giữa node, cần editor/GDScript | logic thuần C#, truyền kiểu tuỳ ý, đường nóng |

Quy tắc thực hành: **mặc định dùng signal cho giao tiếp giữa các node** (Godot lo vòng đời giúp); dùng `event`/`Action` cho **logic thuần C#** (hệ thống không cần là node, ví dụ `Inventory` thuần) hoặc khi cần truyền kiểu C# tự định nghĩa.

### 12.4 Thứ tự, đồng bộ và bẫy

- Khi bạn `EmitSignal`, các hàm đã nối được gọi **ngay lập tức, tuần tự, theo thứ tự nối**, rồi mới trở về dòng sau `EmitSignal` (**đồng bộ**). Handler chạy lâu sẽ làm chậm nơi phát.
- **Tái nhập (re-entrancy):** handler có thể phát tiếp signal hoặc sửa chính đối tượng đang phát, gây trạng thái khó đoán. Giữ handler ngắn và ít tác dụng phụ.
- **Đổi trạng thái vật lý trong callback va chạm** (`BodyEntered`...) có thể báo lỗi vì engine đang xử lý physics; dùng `CallDeferred`, `SetDeferred`, hoặc nối với cờ `Deferred` để thực hiện vào cuối frame.
- Signal chỉ **báo** điều đã xảy ra, **không đảm bảo** ai đang nghe; đừng dựa vào giá trị trả về.

### 12.5 await signal

```csharp
await ToSignal(this, SignalName.Died);                                              // chờ signal của chính mình
await ToSignal(GetTree().CreateTimer(1.0), SceneTreeTimer.SignalName.Timeout);      // chờ timer
```

Các điều cần nhớ: nếu signal **không bao giờ phát** (hoặc node bị xoá), hàm chờ sẽ **treo mãi** (không chạy tiếp); sau `await`, node có thể đã chết, hãy kiểm tra `IsInstanceValid(this)` (chương 8).

### 12.6 Thiết kế signal tốt

- Đặt tên **thì quá khứ** mô tả điều đã xảy ra: `Died`, `HealthChanged`, `CoinCollected`; không đặt tên theo hành động mong muốn (`ShowGameOver`).
- **Phát điều đã xảy ra**, đừng ra lệnh cho người nghe: `Died` thay vì `PleaseShowGameOver`. Người nghe tự quyết phản ứng.
- Truyền **đủ dữ liệu** để người nghe không phải gọi ngược (`HealthChanged(current, max)`).
- Với hệ thống nhiều bên cùng nghe, dùng **event bus**: một autoload chỉ chứa signal toàn cục (`CoinCollected`, `LevelCompleted`) để các hệ thống không phụ thuộc trực tiếp nhau. Đừng lạm dụng: signal toàn cục quá nhiều làm khó lần theo luồng chạy.
- **Gọi xuống, báo lên** (chương 10).

### 12.7 Callable và lambda

`Callable` là "tham chiếu đến một hàm của một đối tượng" mà engine dùng để gọi lại. `Callable.From(...)` bọc một delegate C#; `new Callable(this, MethodName.OnX)` trỏ tới hàm theo tên. Lambda **ngắt kết nối được** chỉ khi bạn **giữ lại cùng một delegate** đã nối; lambda mới viết lại dù cùng nội dung là một delegate khác nên `-=` không ngắt được:

```csharp
Action handler = () => GD.Print("hi");
button.Pressed += handler;
button.Pressed -= handler;               // ngắt được vì cùng một delegate
// button.Pressed -= () => GD.Print("hi");   // KHÔNG ngắt được, đây là lambda khác
```

### 12.8 Lỗi thường gặp

- Delegate signal không kết thúc bằng `EventHandler`.
- Truyền tham số signal kiểu C# không tương thích Variant.
- Nối cùng một handler hai lần (được gọi hai lần).
- Không ngắt `event` C# khi publisher sống lâu hơn subscriber.
- Đổi trạng thái physics ngay trong handler va chạm (cần `Deferred`).
- Chờ signal không bao giờ phát nên hàm `async` treo.

### 12.9 Tự kiểm tra

1. Signal khác gọi hàm trực tiếp ở điểm nào? Vì sao bên phát không cần biết bên nghe?
2. Ba quy tắc khi khai báo `[Signal]` trong C#?
3. Signal Godot và `event` C# khác nhau ở những điểm chính nào? Khi nào chọn cái nào?
4. `EmitSignal` đồng bộ nghĩa là gì và hệ quả?
5. Khi nào cần cờ `Deferred` hoặc `CallDeferred`?
6. Vì sao `-= () => ...` không ngắt được kết nối?
7. Đặt tên và thiết kế signal tốt thế nào? Event bus dùng khi nào?

### 12.10 Bài tập

1. Viết node `Health` phát `HealthChanged`/`Died`; viết hai node nghe độc lập (một hiện thanh máu, một in log).
2. Nối một signal bằng `Connect` với `OneShot` và xác nhận chỉ gọi một lần.
3. Viết hàm `async` chờ `Died` của một node rồi in; thử trường hợp node bị xoá trước khi phát và xem điều gì xảy ra.
4. Tạo autoload `Events` với hai signal; hai node không biết nhau nhưng giao tiếp qua nó.
5. Cố tình đổi `Monitoring` của một `Area2D` trong handler `BodyEntered`, đọc lỗi, rồi sửa bằng `SetDeferred`.

---

**Tiếp theo:** [Task 13](task_13.md)

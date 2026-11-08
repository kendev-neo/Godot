# PHẦN 1. NGÔN NGỮ C#

## Chương 7. Delegate, lambda và event \[Trung bình\]

> [!NOTE]
> **Mục tiêu:** hiểu "hàm như dữ liệu", viết lambda, hiểu closure và bẫy của nó, dùng `event` an toàn, tránh rò rỉ đăng ký.

### 7.1 Delegate: hàm là giá trị

Delegate là **kiểu tham chiếu đến một hàm**. Bạn có thể lưu hàm vào biến, truyền đi và gọi sau. .NET có sẵn: `Action` (không trả về), `Func<..., R>` (có trả về), `Predicate<T>` (trả `bool`).

```csharp
Action<int> onHit = dmg => GD.Print($"Nhận {dmg}");
Func<int, int, int> add = (a, b) => a + b;
Predicate<Enemy> isAlive = e => e.Hp > 0;

onHit(5);
int s = add(2, 3);
var alive = enemies.FindAll(isAlive);
```

Delegate cho phép "truyền hành vi" vào hàm: `Sort`, `Where`, callback hoàn thành, tween callback đều dựa vào đó.

### 7.2 Lambda và closure

Lambda là cách viết ngắn một hàm vô danh: `(a, b) => a + b`. Lambda có thể **bắt (capture)** biến của hàm bao quanh; đó là **closure**.

```csharp
int bonus = 10;
Func<int, int> withBonus = x => x + bonus;   // bắt biến bonus
bonus = 20;
GD.Print(withBonus(1));                      // 21: lambda thấy GIÁ TRỊ MỚI của bonus
```

**Bẫy 1: closure bắt biến, không bắt giá trị.** Trong vòng `for`, mọi lambda dùng chung một `i`:

```csharp
var actions = new List<Action>();
for (int i = 0; i < 3; i++) actions.Add(() => GD.Print(i));   // in 3, 3, 3
for (int i = 0; i < 3; i++) { int copy = i; actions.Add(() => GD.Print(copy)); }   // in 0, 1, 2
```

(`foreach` an toàn hơn: mỗi vòng có biến riêng.)

**Bẫy 2: closure cấp phát bộ nhớ** (tạo đối tượng chứa biến bị bắt). Lambda không bắt biến thì rẻ hơn; có thể khai báo `static` (`static x => x * 2`) để chắc chắn không bắt gì. Tránh tạo lambda mới mỗi frame trong `_Process`.

**Bẫy 3: lambda giữ tham chiếu** đến đối tượng bị bắt, nên có thể giữ nó sống lâu hơn dự kiến (rò rỉ, hoặc giữ node đã bị xoá).

### 7.3 event: thông báo theo mẫu publisher/subscriber

`event` là delegate được bọc để **bên ngoài chỉ được đăng ký (`+=`) và huỷ (`-=`)**, không được tự gọi hay ghi đè. Chỉ class chứa event mới "phát" được.

```csharp
public class Health
{
    public event Action<int, int>? Changed;          // (hiện tại, tối đa)
    public event Action? Died;

    private int _hp = 100;
    public void Take(int dmg)
    {
        _hp = Math.Max(0, _hp - dmg);
        Changed?.Invoke(_hp, 100);                   // ?.Invoke an toàn khi chưa ai đăng ký
        if (_hp == 0) Died?.Invoke();
    }
}

var h = new Health();
h.Changed += (cur, max) => GD.Print($"{cur}/{max}");    // đăng ký
h.Take(30);
```

**Rò rỉ đăng ký:** publisher giữ danh sách tham chiếu đến subscriber. Nếu subscriber bị xoá (ví dụ `QueueFree`) mà **không huỷ đăng ký (`-=`)** và publisher còn sống, thì subscriber không được dọn bộ nhớ và event vẫn gọi vào đối tượng đã chết, gây lỗi. Quy tắc: **đăng ký ở `_Ready`/`_EnterTree`, huỷ ở `_ExitTree`**, đặc biệt khi publisher sống lâu hơn (autoload, singleton).

### 7.4 Kết nối với Godot

- Godot có hệ thống **signal** riêng (chương 12). Signal khai báo bằng `[Signal]` được Godot quản lý, hiện trong editor, nối được với GDScript; signal có sẵn của node nối bằng `+=` giống event C#.
- `Callable.From(...)` bọc một delegate thành `Callable` để đưa vào API Godot (`Tween.TweenCallback`, `Connect`).
- Với logic **thuần C#** (không cần editor, không cần GDScript thấy), `event` hoặc `Action` là đủ và nhanh hơn; với thứ cần kết nối trong editor hoặc xuyên ngôn ngữ, dùng signal.

### 7.5 Lỗi thường gặp

- Không huỷ đăng ký trong `_ExitTree` (rò rỉ, gọi vào đối tượng đã xoá).
- Bắt biến vòng lặp `for` trong lambda và nhận kết quả bất ngờ.
- Gọi `Changed.Invoke()` khi event là `null` (dùng `?.Invoke`).
- Tạo lambda mới liên tục mỗi frame.
- Đăng ký cùng một handler hai lần (được gọi hai lần).

### 7.6 Tự kiểm tra

1. `Action`, `Func`, `Predicate` khác nhau thế nào?
2. Closure là gì? Vì sao đoạn `for` ở 7.2 in ra `3, 3, 3`?
3. `event` khác delegate thường ở điểm nào?
4. Vì sao phải huỷ đăng ký, và ở đâu trong Godot?
5. Khi nào dùng event C# thay vì signal Godot?

### 7.7 Bài tập

1. Viết hàm `Apply(List<int> list, Func<int,int> f)` trả danh sách đã biến đổi; gọi với vài lambda.
2. Sửa đoạn `for` bắt biến để in `0, 1, 2`.
3. Viết class `Timer2` có `event Action Ticked` và đăng ký hai handler; sau đó huỷ một.
4. Viết class phát `event Action<int>` và subscriber có `Dispose()` huỷ đăng ký; kiểm chứng không còn được gọi.

---

**Tiếp theo:** [Task 8](task_8.md)

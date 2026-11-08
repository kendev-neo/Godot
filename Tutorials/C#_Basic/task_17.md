# PHẦN 3. NÂNG CAO VÀ KIẾN TRÚC

## Chương 17. Các mẫu thiết kế (pattern) trong Godot \[Nâng cao\]

> [!NOTE]
> **Mục tiêu:** biết các mẫu thiết kế hay dùng, vì sao chúng tồn tại, và chọn đúng mẫu cho vấn đề thay vì áp dụng máy móc.

### 17.1 Năm nguyên tắc nền (SOLID, rút gọn theo Godot)

| Nguyên tắc                | Ý nghĩa gọn                             | Trong Godot                                                     |
| ------------------------- | --------------------------------------- | --------------------------------------------------------------- |
| **S**ingle Responsibility | một class một lý do để đổi              | node `Health` chỉ lo máu; `Mover` chỉ lo di chuyển              |
| **O**pen/Closed           | mở rộng được mà không sửa code cũ       | thêm loại quân bằng Resource/class mới, không sửa `if` khắp nơi |
| **L**iskov Substitution   | lớp con thay được lớp cha mà không hỏng | mọi `Unit` đều dùng được qua `List<Unit>`                       |
| **I**nterface Segregation | interface nhỏ, đúng việc                | `IDamageable`, `IInteractable` riêng, không một `IEverything`   |
| **D**ependency Inversion  | phụ thuộc vào trừu tượng                | nhận `IDamageable` thay vì `Enemy` cụ thể                       |

Đây là **kim chỉ nam**, không phải luật tuyệt đối. Mục tiêu chung: code **dễ đổi, dễ hiểu, ít chỗ vỡ dây chuyền**.

### 17.2 Composition / Component

Ghép các node nhỏ, mỗi node một nhiệm vụ (`Health`, `Hitbox`, `Mover`, `Inventory`), phát signal riêng. Nhờ vậy cùng một `Health` dùng cho Player, Enemy, thùng gỗ. Đây là mẫu **tự nhiên nhất của Godot** (xem chương 4.6 và 10).

### 17.3 State machine (máy trạng thái)

Dùng khi một thực thể có **vài trạng thái rõ ràng** và quy tắc chuyển giữa chúng (Idle/Run/Jump/Attack, Patrol/Chase/Dead). Có hai kiểu:

**Kiểu enum + switch** (đơn giản, hợp khi ít trạng thái):

```csharp
public enum State { Idle, Run, Jump, Hurt }
private State _state = State.Idle;

private void ChangeState(State next)
{
    if (next == _state) return;
    ExitState(_state);          // dọn dẹp trạng thái cũ
    _state = next;
    EnterState(next);           // khởi tạo trạng thái mới (đổi animation, bật hitbox...)
}

public override void _PhysicsProcess(double delta)
{
    switch (_state)
    {
        case State.Idle: UpdateIdle(delta); break;
        case State.Run:  UpdateRun(delta);  break;
        // ...
    }
}
```

**Kiểu node/class cho mỗi trạng thái** (mỗi trạng thái một class có `Enter/Exit/Update`, xem mục 13.2 Phần I tài liệu chính): tốt khi trạng thái phức tạp và nhiều. Quy tắc thiết kế: **mọi chuyển trạng thái đi qua một hàm duy nhất** (`ChangeState`), để có chỗ đặt log, `Enter/Exit`, và tránh trạng thái "lai".

### 17.4 Observer: signal, event và event bus

Đã học ở chương 7 và 12. Tóm tắt lựa chọn: **signal** cho giao tiếp giữa node; **event C#** cho logic thuần; **event bus** (autoload chứa signal toàn cục) khi nhiều hệ thống độc lập cùng quan tâm một sự kiện. Rủi ro: quá nhiều signal toàn cục làm khó theo dõi "ai phản ứng với cái gì".

### 17.5 Command (lệnh)

Bọc một **hành động** thành đối tượng để có thể xếp hàng, ghi lại, hoàn tác hoặc phát lại.

```csharp
public interface ICommand { void Execute(); void Undo(); }

public class MoveCommand : ICommand
{
    private readonly Node2D _node; private readonly Vector2 _from, _to;
    public MoveCommand(Node2D node, Vector2 to) { _node = node; _from = node.Position; _to = to; }
    public void Execute() => _node.Position = _to;
    public void Undo()    => _node.Position = _from;
}

var history = new Stack<ICommand>();
var cmd = new MoveCommand(unit, target);
cmd.Execute(); history.Push(cmd);
// hoàn tác: history.Pop().Undo();
```

Hợp với: hoàn tác trong trình chỉnh sửa/xây dựng, **hệ thống lệnh cho đơn vị** (người chơi ra lệnh "tấn công/thủ/rút lui"), phát lại trận (ghi lại chuỗi lệnh theo thời gian), mạng.

### 17.6 Object pool

Tái sử dụng đối tượng đắt tiền/thường sinh (đạn, hiệu ứng, kẻ địch) thay vì `Instantiate`/`QueueFree` liên tục, nhằm **giảm cấp phát và khựng hình** (chương 9). Pool cần: `Get()` (lấy hoặc tạo), `Release()` (trả về và **đặt lại trạng thái**), và quy tắc tắt xử lý/ẩn node khi nằm trong pool (`SetProcess(false)`, `Visible = false`, tắt va chạm). Lỗi thường gặp: quên reset trạng thái nên đối tượng tái dùng mang "ký ức" cũ.

### 17.7 Singleton, service locator và truyền phụ thuộc

Có ba cách để node A lấy được dịch vụ B:

| Cách                             | Ví dụ                                      | Ưu                        | Nhược                                                    |
| -------------------------------- | ------------------------------------------ | ------------------------- | -------------------------------------------------------- |
| **Singleton/autoload**           | `GameManager.Instance`                     | tiện, ngắn gọn            | phụ thuộc ngầm, khó test, vòng đời phức tạp (chương 4.3) |
| **Service locator**              | `Services.Get<IAudio>()`                   | tách khỏi class cụ thể    | vẫn phụ thuộc ngầm                                       |
| **Truyền phụ thuộc (injection)** | `[Export] Inventory inv;` hoặc `Init(inv)` | rõ ràng, dễ test, dễ thay | phải nối dây (làm trong scene hoặc code khởi tạo)        |

Quy tắc thực hành: dùng autoload cho thứ **thật sự toàn cục** (trạng thái game, âm thanh, event bus, save); các hệ thống còn lại **truyền phụ thuộc rõ ràng** qua `[Export]` hoặc hàm `Init`.

### 17.8 Thiết kế hướng dữ liệu và Strategy

Đưa **khác biệt** giữa các biến thể vào **Resource** (dữ liệu) thay vì vào `if` rải rác: `UnitData`, `ItemData`, `WaveData`. Với hành vi khác nhau (AI hung hăng/phòng thủ), dùng **Strategy**: một interface hoặc Resource mô tả hành vi, đối tượng giữ tham chiếu và uỷ quyền cho nó; đổi hành vi chỉ cần đổi tham chiếu.

### 17.9 Factory và khởi tạo đối tượng từ scene

Node tạo từ scene dùng constructor không tham số, nên **không truyền dữ liệu qua constructor**. Mẫu quen thuộc: tạo, **gọi `Init(dữ liệu)`**, rồi mới thêm vào cây:

```csharp
var e = scene.Instantiate<Enemy>();
e.Init(data);                    // gán dữ liệu TRƯỚC khi vào cây
parent.AddChild(e);              // sau đó _Ready của e chạy và dùng data đã có
```

Trong `_Ready` của `Enemy`, dùng dữ liệu đã được `Init` đặt. Đặt `Init` **trước** `AddChild` đảm bảo `_Ready` thấy dữ liệu đúng.

### 17.10 Dữ liệu tập trung (data-oriented) cho đám đông

Khi có **hàng nghìn thực thể giống nhau** (lính, đạn, hạt), mỗi thực thể một node là quá nặng. Cách khác: lưu trạng thái trong **mảng struct** (vị trí, máu, loại...) và một **manager** duyệt qua mảng mỗi frame, vẽ bằng `MultiMeshInstance2D`. Đổi lại code khó viết/đọc hơn và mất sự tiện lợi của node. Chỉ dùng khi **đo thấy** cách node-per-entity không đủ (chương 16).

### 17.11 Phản mẫu (anti-pattern) cần tránh

- **God object:** một `GameManager` biết và làm mọi thứ. Tách theo trách nhiệm.
- **Kế thừa sâu** chỉ để dùng lại code (dùng composition).
- **Chuỗi `GetNode("../../../X")`** gắn chặt cấu trúc cây.
- **Singleton ở mọi nơi**, mọi class phụ thuộc ngầm vào nhau.
- **Signal rối (spaghetti):** signal toàn cục chằng chịt, không thể lần theo luồng chạy.
- **Tối ưu/trừu tượng hoá quá sớm:** thêm interface, pattern khi chưa có vấn đề thật. Quy tắc: **làm cho chạy, rồi làm cho đúng, rồi mới làm cho nhanh/đẹp**; chỉ trừu tượng hoá khi thấy mình lặp lại lần thứ ba.
- **Lưu trạng thái động trong Resource dùng chung.**

### 17.12 Tự kiểm tra

1. Nêu ngắn gọn từng chữ trong SOLID và ví dụ trong Godot.
2. Khi nào dùng state machine? Vì sao mọi chuyển trạng thái nên đi qua một hàm?
3. Command pattern giải quyết bài toán nào? Cho ví dụ hợp với game chiến thuật.
4. Object pool giảm vấn đề gì? Lỗi thường gặp khi dùng pool?
5. So sánh singleton, service locator và truyền phụ thuộc.
6. Vì sao khởi tạo node bằng `Init(data)` trước `AddChild`?
7. Kể 4 phản mẫu và cách tránh.

### 17.13 Bài tập

1. Viết state machine kiểu enum cho kẻ địch (Patrol, Chase, Attack, Dead) với `ChangeState`, `EnterState`, `ExitState`.
2. Viết `ICommand` và hệ thống hoàn tác/làm lại với hai `Stack<ICommand>`.
3. Viết `Pool<T>` cho một scene đạn: `Get`, `Release`, reset trạng thái, bật/tắt xử lý.
4. Viết một interface `IAttackBehavior` với hai cài đặt (cận chiến/tầm xa) và đổi được khi chạy.
5. Tách một script "làm mọi thứ" thành 3 component nhỏ bằng composition.

---

**Tiếp theo:** [Task 18](task_18.md)

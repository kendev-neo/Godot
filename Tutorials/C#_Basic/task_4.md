# PHẦN 1. NGÔN NGỮ C#

## Chương 4. Lập trình hướng đối tượng (OOP) \[Cơ bản → Trung bình\]

> [!NOTE]
> **Mục tiêu:** thiết kế class có dữ liệu và hành vi hợp lý; phân biệt class, struct, record, interface; biết khi nào dùng kế thừa và khi nào dùng ghép (composition); hiểu `static` nguy hiểm thế nào trong Godot.

### 4.1 Class và đối tượng

Class là **bản thiết kế**, đối tượng (object, instance) là **sản phẩm** tạo ra từ bản thiết kế đó. Class gom **dữ liệu** (field, property) và **hành vi** (method) vào một chỗ, và che giấu chi tiết bên trong (**đóng gói**, encapsulation).

```csharp
public class Weapon
{
    private float _cooldown;                          // field: dữ liệu riêng, bên ngoài không thấy

    public string Name { get; }                       // property chỉ đọc (gán trong constructor)
    public int Damage { get; private set; }           // đọc công khai, chỉ ghi nội bộ
    public bool Ready => _cooldown <= 0f;             // property tính toán, không lưu

    public Weapon(string name, int damage)            // constructor: chạy khi tạo đối tượng
    {
        Name = name;
        Damage = damage;
    }

    public void Tick(float dt) => _cooldown = Mathf.Max(0f, _cooldown - dt);
    public void Upgrade(int bonus) => Damage += bonus;
}

var sword = new Weapon("Kiếm", 10);                  // tạo đối tượng
var bow = new Weapon("Cung", 6) ;
sword.Upgrade(2);
```

**Field và property khác nhau thế nào?** Field là biến thô. Property là "cặp hàm đọc/ghi" trông như biến, cho phép kiểm soát (kiểm tra giá trị, phát signal khi đổi, chỉ đọc từ ngoài). Quy tắc: **dữ liệu public nên là property, không phải field public**. (Ngoại lệ trong Godot: biến `[Export]` thường là field public hoặc property có `[Export]`; cả hai đều chạy.)

**Khởi tạo gọn:** `new Label { Text = "Hi" }` (object initializer) gán thuộc tính ngay khi tạo.

**`this`:** tham chiếu đến đối tượng hiện tại, dùng khi tên tham số trùng tên field (`this.name = name`) hoặc để truyền chính mình đi (`other.Register(this)`).

### 4.2 Mức truy cập

| Từ khoá     | Ai truy cập được                                   |
| ----------- | -------------------------------------------------- |
| `public`    | mọi nơi                                            |
| `private`   | chỉ trong chính class (**mặc định** nếu không ghi) |
| `protected` | class và các class **con**                         |
| `internal`  | cùng assembly (cùng project)                       |

Nguyên tắc: **mặc định đóng, mở khi cần**. Càng ít thứ `public`, càng ít chỗ có thể làm hỏng trạng thái class, và càng dễ sửa sau này.

### 4.3 static: thuộc về class, không thuộc về đối tượng

```csharp
public class Enemy
{
    public static int TotalKilled;       // MỘT biến dùng chung cho mọi Enemy
    public int Hp;                       // mỗi Enemy có Hp riêng

    public static void ResetStats() => TotalKilled = 0;
}

Enemy.TotalKilled++;                     // gọi qua tên class, không cần new
```

**Cảnh báo quan trọng trong Godot:** biến `static` **sống suốt đời chương trình**, không bị xoá khi bạn đổi hoặc tải lại scene. Nếu một biến static giữ tham chiếu đến Node, sau khi scene đổi node đó đã bị xoá nhưng tham chiếu vẫn còn, và dùng nó gây `ObjectDisposedException`. Quy tắc:

- Dùng `static` cho **hằng, hàm tiện ích không có trạng thái** (như `Mathf`), hoặc dữ liệu thật sự toàn cục.
- Singleton kiểu `public static GameManager Instance` phải được gán lại trong `_EnterTree` và đặt `null` trong `_ExitTree` nếu node có thể bị huỷ.
- Đừng lưu node vào static trừ khi bạn quản lý rõ vòng đời của nó.

### 4.4 Kế thừa, đa hình, abstract

**Kế thừa** (`:`) cho class con dùng lại và mở rộng class cha. Quan hệ phải là **"là một" (is-a)**: Kỵ binh _là một_ đơn vị.

```csharp
public abstract class Unit                       // abstract: không tạo trực tiếp, chỉ để kế thừa
{
    public int Hp { get; protected set; } = 100;

    public virtual void Attack(Unit target)      // virtual: cho phép lớp con ghi đè
    {
        target.TakeDamage(10);
    }

    public void TakeDamage(int amount) => Hp -= amount;

    public abstract string Describe();            // abstract: lớp con BẮT BUỘC cài đặt
}

public class Cavalry : Unit
{
    public override void Attack(Unit target)     // override: ghi đè hành vi
    {
        base.Attack(target);                     // gọi bản của lớp cha
        target.TakeDamage(5);                    // cộng thêm sát thương xung phong
    }
    public override string Describe() => "Kỵ binh";
}

Unit u = new Cavalry();                          // biến kiểu cha, đối tượng kiểu con
u.Attack(enemy);                                 // chạy Cavalry.Attack: ĐA HÌNH
```

**Đa hình (polymorphism)** là khả năng gọi cùng một hàm trên biến kiểu cha mà đối tượng thật quyết định hành vi. Nhờ đó một danh sách `List<Unit>` chứa nhiều loại quân và `foreach (var u in units) u.Attack(t)` chạy đúng hành vi từng loại.

`sealed` ngăn class bị kế thừa tiếp. C# chỉ cho phép **kế thừa một class cha** (nhưng nhiều interface).

**Trong Godot:** mọi script C# gắn vào node đều kế thừa một kiểu node (`CharacterBody2D`, `Node`, `Resource`...). Bạn kế thừa engine, rồi thêm hành vi của mình.

### 4.5 Interface (hợp đồng hành vi)

Interface chỉ nói **"phải làm được gì"**, không nói làm thế nào. Một class có thể cài đặt **nhiều** interface.

```csharp
public interface IDamageable { void TakeDamage(int amount); }
public interface IInteractable { void Interact(Node2D who); }

public partial class Crate : StaticBody2D, IDamageable, IInteractable
{
    public void TakeDamage(int amount) { /* vỡ */ }
    public void Interact(Node2D who) { /* mở */ }
}

if (body is IDamageable d) d.TakeDamage(5);     // không cần biết cụ thể là Crate hay Enemy
```

|                    | Abstract class                    | Interface                                        |
| ------------------ | --------------------------------- | ------------------------------------------------ |
| Có code dùng chung | có                                | không (thường)                                   |
| Số lượng kế thừa   | 1                                 | nhiều                                            |
| Dùng khi           | các lớp có chung bản chất và code | các lớp không liên quan nhưng chung một khả năng |

### 4.6 Ghép (composition) thay vì kế thừa sâu

Kế thừa sâu (`Entity → Character → Enemy → FlyingEnemy → BossFlyingEnemy`) cứng nhắc và khó đổi. Thay vào đó **ghép các thành phần nhỏ**: một `Enemy` _có_ `Health`, `Mover`, `Hitbox`. Đây cũng chính là cách Godot làm việc: scene là cây các node nhỏ, mỗi node một nhiệm vụ. Quy tắc: **"ưu tiên has-a hơn is-a"**; chỉ kế thừa khi quan hệ thật sự là is-a và lớp cha ổn định.

### 4.7 struct, class và record

|              | `class`                                   | `struct`                                          | `record`                                             |
| ------------ | ----------------------------------------- | ------------------------------------------------- | ---------------------------------------------------- |
| Kiểu         | **tham chiếu**                            | **giá trị**                                       | tham chiếu (có thể `record struct`)                  |
| Gán `b = a`  | hai biến cùng trỏ một đối tượng           | **copy** toàn bộ                                  | cùng trỏ một đối tượng                               |
| So sánh `==` | so con trỏ (cùng đối tượng?)              | so từng trường (tuỳ)                              | so **nội dung**                                      |
| Dùng khi     | đối tượng có danh tính, vòng đời, hành vi | dữ liệu **nhỏ**, bất biến (vector, màu, toạ độ ô) | dữ liệu thuần, bất biến (save, thông tin sát thương) |

```csharp
public readonly struct Cell
{
    public int X { get; }
    public int Y { get; }
    public Cell(int x, int y) { X = x; Y = y; }
}

public record ItemStack(string Id, int Count);

var a = new ItemStack("sword", 1);
var b = a with { Count = 2 };                    // bản sao đã đổi một trường
GD.Print(a == new ItemStack("sword", 1));        // True: so sánh nội dung
```

`Vector2`, `Color`, `Rect2` của Godot đều là **struct**, vì thế bạn không sửa được `Velocity.X` trực tiếp (xem chương 3). **Struct lớn bị copy nhiều lần sẽ chậm**, nên chỉ dùng struct cho dữ liệu nhỏ (khoảng dưới 16–24 byte) và ưu tiên bất biến (`readonly struct`).

### 4.8 enum và cờ bit

```csharp
public enum Team { Player, Enemy, Neutral }

[Flags]
public enum Ability { None = 0, Fly = 1, Swim = 2, Climb = 4 }

Ability a = Ability.Fly | Ability.Swim;          // kết hợp bằng |
bool canFly = (a & Ability.Fly) != 0;            // kiểm tra bằng &  (hoặc a.HasFlag(Ability.Fly))
```

Dùng enum thay cho số/chuỗi "ma thuật" (`state == 2`), để trình biên dịch giúp bạn bắt lỗi chính tả. Enum C# tự thành danh sách chọn trong Inspector khi gắn `[Export]`.

### 4.9 Tạo đối tượng: `new` và Godot

- Class C# thuần: `new Weapon("Kiếm", 10)`.
- Class kế thừa **node/resource của Godot**: không dùng `new` cho node nằm trong scene. Dùng `PackedScene.Instantiate<T>()`; engine tạo đối tượng bằng constructor **không tham số** rồi mới thiết lập thuộc tính. Vì vậy **đừng đặt logic cần node con trong constructor**; hãy đặt trong `_Ready()`.
- Class script phải `partial` (chương 11 giải thích vì sao).

### 4.10 Lỗi thường gặp

- Dùng kế thừa chỉ để dùng lại code dù quan hệ không phải is-a.
- Public field khắp nơi, về sau không thể kiểm soát thay đổi.
- Quên `virtual` ở hàm cha nên `override` ở con báo lỗi.
- Lưu Node vào `static` rồi dùng sau khi scene đã đổi.
- Dùng `class` cho dữ liệu nhỏ sinh ra hàng nghìn đối tượng mỗi frame (nhiều rác bộ nhớ); cân nhắc `struct`.
- Nhầm `==` giữa hai `class` (so con trỏ) với so nội dung.

### 4.11 Tự kiểm tra

1. Field và property khác nhau thế nào? Vì sao nên dùng property cho dữ liệu public?
2. `private`, `protected`, `public`, `internal` khác nhau thế nào? Mặc định là gì?
3. Vì sao biến `static` giữ Node nguy hiểm trong Godot?
4. `virtual`, `override`, `abstract`, `sealed` mỗi cái nghĩa là gì?
5. Khi nào chọn interface thay cho abstract class?
6. Class và struct khác nhau ra sao khi gán `b = a`? Khi nào dùng `record`?
7. Vì sao không nên đặt logic phụ thuộc node con trong constructor của một script Godot?

### 4.12 Bài tập

1. Viết `abstract class Unit` với `Hp`, `TakeDamage`, và `abstract Attack`; viết `Archer`, `Knight` kế thừa; đưa vào `List<Unit>` và gọi `Attack` chung.
2. Viết interface `IHealable` và cho hai class không liên quan cùng cài đặt; viết hàm nhận `IHealable`.
3. Viết `readonly struct GridPos` có `Add(GridPos other)` trả về struct mới; kiểm chứng copy-by-value.
4. Viết `record SaveSlot(string Name, int Level)`, dùng `with` tạo bản đã nâng cấp, và kiểm tra `==`.
5. Tạo `enum` có `[Flags]` cho quyền hạn và viết hàm kiểm tra tổ hợp.

---

**Tiếp theo:** [Task 5](task_5.md)

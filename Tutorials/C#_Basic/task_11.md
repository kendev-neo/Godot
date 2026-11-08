# PHẦN 2. C# TRONG GODOT

## Chương 11. C# gắn với Godot như thế nào \[Trung bình → Nâng cao\]

> [!NOTE]
> **Mục tiêu:** hiểu vì sao có `partial`, `[Export]`, `Variant`, `StringName`; biết vòng đời đối tượng Godot trong C# và các lỗi liên quan; hiểu build và nạp lại assembly.

### 11.1 Mô hình hai nửa

Mỗi đối tượng Godot (Node, Resource...) gồm **hai nửa**: đối tượng **native** nằm trong engine C++, và một **"vỏ" C#** trỏ tới nó. Khi bạn gọi `node.Position` hay `MoveAndSlide()`, lời gọi đi từ C# **qua lớp trao đổi (interop)** tới engine rồi trở lại. Mỗi lần gọi tuy nhỏ nhưng **cộng dồn** khi gọi hàng chục nghìn lần mỗi frame (chương 16).

Hệ quả cho việc quản lý vòng đời: **GC chỉ thu hồi "vỏ" C#**; việc giải phóng nửa native do engine làm (`QueueFree`, đếm tham chiếu).

### 11.2 `partial` và bộ sinh code

Godot có **source generator** chạy lúc bạn build: nó đọc class script của bạn rồi **sinh thêm** một nửa class (cùng tên, trong file khác) chứa:

- Bảng đăng ký các biến `[Export]` và `[Signal]` để editor/Inspector thấy chúng.
- Bảng điều phối các hàm ghi đè (`_Ready`, `_Process`...) để engine gọi vào C# nhanh.
- Các lớp lồng `PropertyName`, `MethodName`, `SignalName` (tên dạng `StringName`).

Vì thế class **bắt buộc `partial`** (hai nửa ghép làm một). Các ràng buộc cần nhớ: class script phải **cùng tên với file**, nằm ở cấp ngoài (**không lồng** trong class khác), và **không phải generic**. Vi phạm thì script không gắn được vào node hoặc không hiện biến export.

### 11.3 Các attribute thường dùng

| Attribute                                               | Tác dụng                                           |
| ------------------------------------------------------- | -------------------------------------------------- |
| `[Export]`                                              | hiện biến/property trong Inspector, lưu cùng scene |
| `[ExportGroup]`, `[ExportSubgroup]`, `[ExportCategory]` | gom nhóm trong Inspector                           |
| `[Signal]`                                              | khai báo signal (delegate kết thúc `EventHandler`) |
| `[GlobalClass]`                                         | cho class hiện trong hộp thoại tạo Node/Resource   |
| `[Tool]`                                                | script chạy cả trong editor                        |

**Kiểu có thể `[Export]`:** số, `bool`, `string`, `enum`, các struct của Godot (`Vector2`, `Color`...), kiểu kế thừa `Node`, kiểu kế thừa `Resource` (kể cả `PackedScene`, `Texture2D`), mảng của các kiểu này (`int[]`, `Godot.Collections.Array<T>`) và `Godot.Collections.Dictionary`. **Không export được:** `List<T>`, và class C# thường tự định nghĩa (không phải Resource). Muốn dữ liệu phức tạp hiện trong Inspector, hãy làm nó **kế thừa `Resource`** (chương 15).

### 11.4 Variant

`Variant` là "hộp chứa bất kỳ kiểu Godot". Engine dùng nó cho mọi chỗ **không biết trước kiểu**: tham số signal, `Call/Get/Set`, `Godot.Collections.Array/Dictionary`, giá trị thuộc tính.

```csharp
Variant v = 42;                              // chuyển ngầm từ kiểu hỗ trợ
int i = v.AsInt32();                         // lấy ra theo kiểu
var p = node.Get("position").AsVector2();    // đọc thuộc tính bằng tên (chuỗi), trả Variant
node.Call("take_damage", 5);                 // gọi hàm bằng tên, tham số là Variant
int again = v.As<int>();                     // dạng generic
```

Nhờ C# có kiểu tĩnh, bạn **hiếm khi phải dùng Variant trực tiếp**; chỉ gặp khi làm việc với signal, gọi động, hoặc `Godot.Collections`. Dùng sai kiểu (lấy `AsString()` từ Variant đang chứa `int`) cho kết quả sai hoặc lỗi lúc chạy.

### 11.5 Vòng đời đối tượng và các lỗi quen thuộc

- `new MyNode()` tạo node nhưng **chưa vào cây**. Nếu không `AddChild` và cũng không `QueueFree`/`Free`, nó là **orphan** (rò rỉ). Dùng `Instantiate<T>()` cho scene.
- `QueueFree()` xoá cuối frame; sau khi native đã bị xoá, **vỏ C# còn nhưng dùng nó ném `ObjectDisposedException`**. Kiểm tra bằng `GodotObject.IsInstanceValid(obj)` (trả `false` với `null` và đối tượng đã xoá).
- Tham chiếu C# **không giữ node sống**: bạn có thể còn biến trỏ tới node đã bị xoá. Danh sách, biến static, lambda... cần dọn khi node thoát cây (nối vào `TreeExiting` hoặc `_ExitTree`).
- `RefCounted` và `Resource` không cần xoá thủ công; chúng sống đến khi hết tham chiếu. `FileAccess` nên `using` để đóng ngay.

### 11.6 StringName và NodePath

`StringName` là chuỗi được **"đóng dấu" (interned)** để so sánh nhanh bằng con trỏ thay vì từng ký tự; engine dùng cho tên action, animation, signal, thuộc tính. `NodePath` là đường dẫn node. Cả hai **chuyển ngầm từ `string`**, nên `Input.IsActionPressed("jump")` chạy. Nhưng mỗi lần chuyển từ `string` sẽ tạo/ tra cứu đối tượng. Trong **vòng lặp nóng** hãy cache:

```csharp
private static readonly StringName JumpAction = "jump";
// dùng: Input.IsActionPressed(JumpAction)
```

Và dùng các hằng sinh sẵn `MethodName.X`, `SignalName.X`, `PropertyName.X` để tránh gõ chuỗi sai.

### 11.7 Build và nạp lại assembly

- Bấm **Build** (hoặc build từ Rider) biên dịch code thành `.dll`; Godot nạp lại để thấy script, `[Export]`, `[Signal]` mới.
- **Build lỗi thì Godot dùng assembly cũ**: bạn sửa code mà game không đổi là dấu hiệu build đang lỗi; luôn đọc ô Output/Build.
- Khi assembly được nạp lại trong lúc editor đang mở, trạng thái của script `[Tool]` có thể bị đặt lại; biến static cũng quay về giá trị khởi tạo.
- Thêm package NuGet bằng `dotnet add package`; ưu tiên thư viện không phụ thuộc nền tảng nếu muốn xuất đa nền tảng.

### 11.8 Dùng chung với GDScript

Godot cho phép trộn hai ngôn ngữ trong một project, nhưng gọi chéo là **gọi động theo tên** (`Call`, `Get`, `Set`), mất kiểm tra kiểu lúc biên dịch. Chi tiết ở Phụ lục G.14 của tài liệu chính. Lời khuyên: giữ toàn bộ code gameplay của bạn trong C#.

### 11.9 Lỗi thường gặp

- Thiếu `partial`, sai tên file, lồng class, hoặc dùng generic cho script.
- Sửa code mà chưa build (hoặc build lỗi).
- Dùng đối tượng sau `QueueFree` (`ObjectDisposedException`).
- Tạo node bằng `new` rồi quên thêm vào cây hoặc xoá.
- `[Export] List<T>` không hiện.
- Gõ chuỗi tên hàm/signal sai thay vì dùng `MethodName`/`SignalName`.

### 11.10 Tự kiểm tra

1. Mô hình "hai nửa" của một đối tượng Godot là gì? GC thu hồi nửa nào?
2. `partial` dùng để làm gì? Bộ sinh code sinh ra những gì?
3. Kể các ràng buộc của class script.
4. Kiểu nào `[Export]` được, kiểu nào không? Muốn export dữ liệu phức tạp làm thế nào?
5. Variant là gì và khi nào bạn thực sự gặp nó?
6. Orphan node là gì? `IsInstanceValid` giải quyết điều gì?
7. Vì sao cache `StringName` trong vòng lặp nóng?

### 11.11 Bài tập

1. Tạo script thiếu `partial`, build và đọc thông báo lỗi/hành vi; sửa lại.
2. Viết script có `[Export]` cho `int`, `enum`, `PackedScene`, `Godot.Collections.Array<string>`, và xem trong Inspector.
3. Tạo node bằng `new` không `AddChild`, quan sát Monitors > Orphan Nodes, rồi sửa.
4. `QueueFree` một node rồi truy cập nó ở frame sau; bắt `ObjectDisposedException` và thêm `IsInstanceValid`.
5. Dùng `Call`/`Get` gọi hàm và đọc thuộc tính của một node bằng tên; so với gọi trực tiếp có kiểu.

---

**Tiếp theo:** [Task 12](task_12.md)

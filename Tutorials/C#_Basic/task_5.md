# PHẦN 1. NGÔN NGỮ C#

## Chương 5. Collection và generic \[Cơ bản → Trung bình\]

> [!NOTE]
> **Mục tiêu:** chọn đúng cấu trúc dữ liệu cho từng nhu cầu, biết chi phí của chúng, viết được generic đơn giản, dùng LINQ có ý thức, phân biệt collection của .NET và của Godot.

### 5.1 Mảng và List

```csharp
int[] fixedArr = new int[5];                 // mảng: độ dài cố định, truy cập theo chỉ số
fixedArr[0] = 10;                            // chỉ số bắt đầu từ 0; fixedArr.Length = 5

var list = new List<string>();               // danh sách co giãn
list.Add("An"); list.Add("Bình");
list.Insert(0, "Cường");
list.Remove("An");
GD.Print(list.Count);                        // List dùng Count, mảng dùng Length
```

`List<T>` bên trong là một mảng tự nhân đôi dung lượng khi đầy. Truy cập theo chỉ số **nhanh**; thêm cuối **nhanh**; chèn/xoá ở **giữa** phải dịch các phần tử (chậm khi danh sách lớn). Truy cập chỉ số ngoài phạm vi ném `ArgumentOutOfRangeException` (hoặc `IndexOutOfRangeException` với mảng).

### 5.2 Bảng chi phí (độ phức tạp) để chọn cấu trúc

| Cấu trúc             | Truy cập chỉ số | Thêm                   | Tìm / Contains   | Dùng khi                                         |
| -------------------- | --------------- | ---------------------- | ---------------- | ------------------------------------------------ |
| `T[]`                | O(1)            | không đổi kích thước   | O(n)             | dữ liệu cố định, hiệu năng cao                   |
| `List<T>`            | O(1)            | cuối O(1)\*, giữa O(n) | O(n)             | danh sách chung, thứ tự quan trọng               |
| `Dictionary<K,V>`    | —               | O(1)\*                 | theo khoá O(1)\* | tra cứu theo id/tên                              |
| `HashSet<T>`         | —               | O(1)\*                 | O(1)\*           | kiểm tra "đã có chưa", loại trùng                |
| `Queue<T>`           | —               | cuối O(1)              | —                | hàng đợi vào trước ra trước (BFS, lệnh)          |
| `Stack<T>`           | —               | đầu O(1)               | —                | vào sau ra trước (DFS, undo)                     |
| `PriorityQueue<E,P>` | —               | O(log n)               | —                | lấy phần tử ưu tiên nhất (A\*, sự kiện theo giờ) |

(\*) trung bình. **Quy tắc:** nếu bạn hay hỏi "cái này có trong tập chưa" hoặc "tìm theo khoá" thì dùng `HashSet`/`Dictionary`, không duyệt `List` (O(n)). Với vài chục phần tử, khác biệt không đáng kể; với hàng nghìn phần tử mỗi frame, nó quyết định hiệu năng.

### 5.3 Dictionary

```csharp
var price = new Dictionary<string, int> { ["kiem"] = 100, ["khien"] = 80 };

price["cung"] = 90;                           // thêm hoặc ghi đè
int p = price["kiem"];                        // KeyNotFoundException nếu khoá không có!

if (price.TryGetValue("mu", out int v)) { }   // cách an toàn
bool has = price.ContainsKey("kiem");
price.Remove("khien");

foreach (var (key, value) in price) { }      // duyệt cặp khoá-giá trị
```

Khoá phải có **mã băm ổn định** và so sánh bằng đúng: `string`, `int`, `enum`, `record`, struct bất biến là khoá tốt. Dùng class thường làm khoá thì so sánh theo con trỏ (hai đối tượng nội dung giống nhau vẫn khác khoá). Thứ tự duyệt `Dictionary` **không đảm bảo**.

### 5.4 HashSet, Queue, Stack, PriorityQueue

```csharp
var visited = new HashSet<Vector2I>();
if (visited.Add(cell)) { /* lần đầu thấy ô này */ }     // Add trả false nếu đã có

var open = new Queue<Vector2I>();         // duyệt theo lớp (BFS)
open.Enqueue(start);
while (open.Count > 0) { var c = open.Dequeue(); /* ... */ }

var undo = new Stack<Command>();          // hoàn tác
undo.Push(cmd); var last = undo.Pop();

var pq = new PriorityQueue<Node, float>(); // ưu tiên nhỏ nhất ra trước (A*)
pq.Enqueue(node, cost); var best = pq.Dequeue();
```

### 5.5 Generic: viết một lần, dùng cho nhiều kiểu

Generic cho phép viết code **không gắn cứng kiểu**, nhưng vẫn được kiểm tra kiểu lúc biên dịch (khác với dùng `object` rồi ép kiểu).

```csharp
public class Pool<T> where T : Node, new()          // ràng buộc: T phải là Node và có constructor rỗng
{
    private readonly Stack<T> _free = new();

    public T Get() => _free.Count > 0 ? _free.Pop() : new T();
    public void Release(T item) => _free.Push(item);
}

public static T Spawn<T>(Node parent, PackedScene scene) where T : Node
{
    var n = scene.Instantiate<T>();
    parent.AddChild(n);
    return n;
}

var bullets = new Pool<Node2D>();
var enemy = Spawn<Enemy>(this, enemyScene);
```

Các ràng buộc phổ biến: `where T : class`/`struct`, `where T : SomeBase`, `where T : IInterface`, `where T : new()`. `default(T)` hoặc `default` cho giá trị mặc định của kiểu. `Instantiate<T>()`, `GetNode<T>()` mà bạn đã dùng đều là generic.

### 5.6 IEnumerable, foreach và LINQ

`foreach` duyệt được mọi thứ cài đặt `IEnumerable<T>`. `yield return` cho phép tự viết dãy lười:

```csharp
IEnumerable<int> Evens(int max) { for (int i = 0; i <= max; i += 2) yield return i; }
```

**LINQ** (`using System.Linq;`) cho phép truy vấn gọn:

```csharp
var strong = enemies.Where(e => e.Hp > 50).OrderBy(e => e.Hp).ToList();
var names  = enemies.Select(e => e.Name);
bool any   = enemies.Any(e => e.IsBoss);
var first  = enemies.FirstOrDefault(e => e.Team == Team.Enemy);   // null nếu không có
int count  = enemies.Count(e => e.Hp > 0);
```

**Hai bẫy của LINQ:**

1. **Thực thi trì hoãn (deferred):** `Where/Select` chưa chạy cho đến khi bạn duyệt (`foreach`, `ToList`). Nếu danh sách gốc đổi trước khi duyệt, kết quả khác; nếu duyệt hai lần thì chạy hai lần.
2. **Cấp phát bộ nhớ:** LINQ tạo đối tượng tạm, nên **không dùng trong `_Process`/`_PhysicsProcess` cho tập lớn**. Tốt cho code khởi tạo, công cụ, UI; trong vòng lặp nóng hãy viết `for`/`foreach` tay.

### 5.7 Collection của .NET và của Godot

| Nhu cầu                                             | Dùng                                                                                                    |
| --------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| Dữ liệu nội bộ trong code C#                        | `List<T>`, `Dictionary<K,V>`, `HashSet<T>`, `T[]`                                                       |
| Biến `[Export]` hiện trong Inspector                | `Godot.Collections.Array<T>`, `Godot.Collections.Dictionary<K,V>`, hoặc mảng C# như `int[]`, `string[]` |
| Tham số của signal, `Call`, kết quả `GetChildren()` | `Godot.Collections.Array`/`Dictionary`                                                                  |

Ghi nhớ: **`List<T>` không export được** ra Inspector; dùng `Godot.Collections.Array<T>` hoặc mảng. `Godot.Collections.*` đi qua lớp trao đổi với engine nên chậm hơn collection .NET cho xử lý nội bộ; chuyển sang `List<T>` bằng `.ToList()`/`new List<T>(arr)` nếu bạn xử lý nặng.

### 5.8 Sắp xếp

```csharp
enemies.Sort((a, b) => a.Hp.CompareTo(b.Hp));          // sắp xếp tại chỗ (List.Sort không ổn định)
var sorted = enemies.OrderBy(e => e.Hp).ThenBy(e => e.Name).ToList();   // LINQ: ổn định, tạo bản mới
```

"Ổn định" nghĩa là hai phần tử bằng nhau giữ nguyên thứ tự ban đầu. Cần thứ tự ổn định (ví dụ vẽ theo chiều sâu) thì dùng `OrderBy`.

### 5.9 Lỗi thường gặp

- `list[i]` với `i == list.Count`; nhớ `i < list.Count`.
- `dict[key]` khi khoá chưa có (`KeyNotFoundException`); dùng `TryGetValue`.
- Sửa collection khi đang `foreach` (`InvalidOperationException`).
- Gán `var b = a` cho `List` rồi sửa `b`, tưởng `a` không đổi (cả hai là **cùng một** list). Muốn bản sao: `new List<T>(a)`.
- Giữ node đã `QueueFree` trong `List` mà không dọn, truy cập sau gây `ObjectDisposedException`; kiểm tra `GodotObject.IsInstanceValid` hoặc xoá khỏi list khi node thoát cây.
- Dùng LINQ trong `_Process` với hàng nghìn phần tử.
- Dùng class làm khoá `Dictionary` mà không override `Equals`/`GetHashCode`.

### 5.10 Tự kiểm tra

1. Khi nào dùng `HashSet` thay `List` để kiểm tra "đã có chưa"? Vì sao nhanh hơn?
2. `dict[key]` và `TryGetValue` khác nhau thế nào khi khoá không tồn tại?
3. Generic giải quyết vấn đề gì so với dùng `object`? Cho ví dụ ràng buộc `where`.
4. Hai bẫy của LINQ là gì? Vì sao tránh nó trong `_Process`?
5. Vì sao `[Export] List<int>` không dùng được? Dùng gì thay?
6. Sắp xếp `List.Sort` và `OrderBy` khác nhau ở điểm nào?
7. Vì sao xoá phần tử trong `foreach` gây lỗi, và có 2 cách xử lý?

### 5.11 Bài tập

1. Cài đặt BFS trên lưới 10x10 bằng `Queue<Vector2I>` và `HashSet<Vector2I>`; in ra đường đi ngắn nhất giữa hai ô.
2. Viết `Pool<T>` generic có `Get`/`Release`, thử với một class đơn giản.
3. Cho `List<Enemy>`, viết ba truy vấn LINQ: kẻ địch còn sống, tên theo máu giảm dần, máu trung bình; rồi viết lại bằng `for` và so sánh.
4. Viết `Dictionary<ItemKind, int>` đếm số lượng vật phẩm, thêm/bớt an toàn bằng `TryGetValue`.
5. Dùng `PriorityQueue<string, int>` mô phỏng danh sách sự kiện theo thời gian.

---

**Tiếp theo:** [Task 6](task_6.md)

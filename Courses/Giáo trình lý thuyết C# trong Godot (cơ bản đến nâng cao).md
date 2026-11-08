# Giáo trình lý thuyết C# trong Godot (cơ bản đến nâng cao)

Học vững lý thuyết trước, làm dự án sau · Godot 4.x bản .NET · C# trên .NET 8

## 0. Cách dùng giáo trình này

Giáo trình này **không có dự án**. Mục tiêu là bạn nắm chắc lý thuyết C# và cách C# vận hành trong Godot, để khi bắt tay làm dự án (tài liệu chính, Phụ lục B trở đi) bạn không phải vừa làm vừa đoán. Bạn chỉ sang dự án khi vượt được **Cổng kiểm tra** ở cuối giáo trình.

**Mỗi chương theo cùng một mẫu:**

1. **Mục tiêu:** sau chương bạn phải làm được gì.
2. **Lý thuyết:** giải thích khái niệm, **vì sao** nó tồn tại, và nó liên quan gì đến Godot.
3. **Ví dụ:** mã ngắn để gõ lại.
4. **Lỗi thường gặp.**
5. **Tự kiểm tra:** câu hỏi trả lời **không nhìn tài liệu**. Nếu không trả lời được, đọc lại mục tương ứng.
6. **Bài tập (kata):** bài code ngắn, không phải dự án, để biến lý thuyết thành thứ bạn gõ được.

Nhãn độ khó: **\[Cơ bản\]** bắt buộc cho mọi người; **\[Trung bình\]** cần để viết code không lỗi vặt; **\[Nâng cao\]** cần khi game lớn, nhưng nên đọc ngay để biết mình sẽ gặp gì.

### 0.1 Môi trường thực hành

- **Phần 1 (ngôn ngữ C#, chương 1–9):** nên thực hành bằng **project console** cho nhanh, không cần mở Godot:

  ```
  dotnet new console -n Lab
  cd Lab
  dotnet run
  ```

  Sửa `Program.cs`, chạy lại, đọc kết quả. Vòng lặp rất nhanh nên học được nhiều.
- **Phần 2 và 3 (Godot, chương 10–19):** tạo một project Godot .NET tên `Lab` chỉ có một scene trống. Mỗi bài tập là một script gắn vào một node, in kết quả bằng `GD.Print`.
- Editor: Rider (khuyến nghị) hoặc VS Code + C# Dev Kit.

### 0.2 Sơ đồ giáo trình

| Phần | Chương | Nội dung |
| --- | --- | --- |
| **1. Ngôn ngữ C#** | 1–9 | Kiểu dữ liệu, luồng điều khiển, hàm, OOP, collection và generic, null và ngoại lệ, delegate và event, async, bộ nhớ |
| **2. C# trong Godot** | 10–16 | Node/Scene/Resource, cách C# nối với engine, signal, toán và thời gian, input/physics/hệ toạ độ, Resource và lưu trữ, đa luồng và hiệu năng |
| **3. Nâng cao và kiến trúc** | 17–19 | Pattern thiết kế trong Godot, cấu trúc mã nguồn, kiểm thử, tooling |
| **Cổng kiểm tra** | cuối | Kiểm tra tổng hợp trước khi làm dự án |

Lịch gợi ý: 5–7 tuần, mỗi chương 1–2 buổi. Hiểu chắc quan trọng hơn nhanh.

---

# PHẦN 1. NGÔN NGỮ C#

## Chương 1. Chương trình C# và kiểu dữ liệu \[Cơ bản\]

**Mục tiêu:** hiểu một file `.cs` được biên dịch và chạy ra sao; chọn đúng kiểu số; hiểu các bẫy số học và chuỗi.

### 1.1 Từ file .cs đến chương trình chạy

C# là ngôn ngữ **biên dịch, có kiểu tĩnh**: trình biên dịch (Roslyn) kiểm tra kiểu và cú pháp **trước khi chạy**, biến mã nguồn thành mã trung gian (IL) trong một file `.dll` (assembly). Khi chạy, runtime .NET biên dịch IL thành mã máy (JIT). Hệ quả với bạn:

- Lỗi sai kiểu, sai tên biến được báo **lúc build**, không phải lúc chơi. Đây là lợi thế lớn so với ngôn ngữ kiểu động.
- Trong Godot, bấm **Build** chính là bước biên dịch này; Godot rồi nạp lại assembly để thấy script và thuộc tính mới.
- Project console dùng cùng quy trình, chỉ khác chỗ điểm vào là hàm `Main` (hay *top-level statements* trong bản mẫu mới), còn trong Godot điểm vào là engine gọi các hàm như `_Ready`.

Một file C# gồm: các dòng `using` (khai báo dùng thư viện), tùy chọn `namespace`, rồi ít nhất một `class`.

```csharp
using System;                  // thư viện chuẩn .NET

namespace Lab;                 // nhóm tên, tránh trùng

public class Program
{
    public static void Main()
    {
        Console.WriteLine("Xin chào");    // trong Godot dùng GD.Print("...")
    }
}
```

### 1.2 Các kiểu cơ bản

| Kiểu | Kích thước | Phạm vi / độ chính xác | Dùng khi |
| --- | --- | --- | --- |
| `byte` | 8 bit | 0 → 255 | dữ liệu thô, màu từng kênh |
| `short` | 16 bit | ±32 767 | hiếm dùng |
| `int` | 32 bit | ± khoảng 2,1 tỷ | **số nguyên mặc định** (máu, điểm, chỉ số) |
| `long` | 64 bit | ± khoảng 9,2 triệu tỷ tỷ | id lớn, thời gian dạng tick |
| `float` | 32 bit | \~7 chữ số có nghĩa | **toạ độ, vận tốc trong Godot** |
| `double` | 64 bit | \~15–16 chữ số | `delta`, toán cần độ chính xác cao |
| `decimal` | 128 bit | 28–29 chữ số thập phân chính xác | tiền tệ thật (hiếm trong game) |
| `bool` | 1 bit logic | `true`/`false` | cờ, điều kiện |
| `char` | 16 bit | một ký tự UTF-16 | hiếm dùng |
| `string` | động | chuỗi ký tự | văn bản |

**Hậu tố số thực:** `1.5` mặc định là `double`; muốn `float` phải viết `1.5f`. Đây là lỗi biên dịch rất hay gặp với người mới: `float speed = 1.5;` báo lỗi.

**`var`:** để trình biên dịch suy ra kiểu từ giá trị bên phải. Kiểu vẫn **cố định** từ lúc khai báo (khác với `var` của JavaScript). Chỉ dùng cho biến cục bộ khi kiểu hiển nhiên (`var hp = 100;`, `var list = new List<int>();`).

**`const` và `readonly`:** `const` là hằng **thời điểm biên dịch** (chỉ dùng cho kiểu cơ bản/chuỗi); `readonly` là field chỉ được gán **một lần** (khi khai báo hoặc trong constructor), dùng cho đối tượng.

### 1.3 Ép kiểu và phép toán

```csharp
int a = 7 / 2;          // 3: chia hai số nguyên cho số nguyên
float b = 7 / 2f;       // 3.5
int r = 7 % 2;          // 1: phần dư

long big = 5;           // ép NGẦM định (an toàn, không mất thông tin)
int small = (int)big;   // ép TƯỜNG MINH (có thể mất thông tin, phải viết (int))
float f = (float)delta; // double -> float
int truncated = (int)3.9f;   // 3: cắt phần thập phân, không làm tròn
int rounded = Mathf.RoundToInt(3.9f);  // 4
```

**Tràn số (overflow):** mặc định C# **không báo lỗi** khi số nguyên vượt phạm vi; nó quay vòng. `int.MaxValue + 1` cho ra số âm rất lớn. Trong game, điều này có thể biến "tiền tăng mãi" thành "tiền âm". Dùng `long` hoặc kiểm tra biên, hoặc bọc trong `checked { ... }` nếu muốn ném lỗi.

**Số thực không chính xác tuyệt đối:** `0.1 + 0.2 == 0.3` là **false** với `double`/`float` vì số thập phân được lưu dưới dạng nhị phân xấp xỉ. **Không so sánh `==` hai số thực** (nhất là kết quả phép tính); trong Godot dùng `Mathf.IsEqualApprox(a, b)` và `Mathf.IsZeroApprox(a)`, hoặc so sánh trong một sai số nhỏ.

### 1.4 Chuỗi (string)

- `string` **bất biến**: mỗi lần "sửa" thực ra tạo chuỗi mới. Nối chuỗi trong vòng lặp lớn sinh rất nhiều rác; dùng `StringBuilder` khi ghép nhiều lần.
- Nội suy: `$"{name} có {hp} máu"`; định dạng: `$"{x:0.00}"` (hai chữ số thập phân), `$"{n:N0}"` (phân cách hàng nghìn).
- So sánh: `==` so sánh **nội dung**; dùng `string.Equals(a, b, StringComparison.OrdinalIgnoreCase)` để không phân biệt hoa thường.
- Chuỗi rỗng và `null` khác nhau; `string.IsNullOrEmpty(s)` kiểm tra cả hai.
- Tiếng Việt là Unicode, dùng bình thường; lưu ý font trong Godot phải hỗ trợ dấu.

### 1.5 Kiểu có thể null cho kiểu giá trị

`int?` là "int hoặc không có gì": `int? maybe = null; if (maybe.HasValue) { int v = maybe.Value; }` hoặc `maybe ?? 0`. Dùng khi "chưa có giá trị" là trạng thái hợp lệ (ví dụ `Vector2? target`).

### 1.6 Liên hệ với Godot

- Toạ độ `Vector2`, `Vector3` dùng `float` (32 bit); `delta` là `double`. Trong `_Process`, thường ép `(float)delta`.
- Người chơi thấy chỉ số dạng `int`; hãy dùng `int` cho máu, vàng, số lượng; `float` cho vị trí/vận tốc/thời gian.
- Kiểu `Variant` của Godot là "hộp chứa bất kỳ kiểu"; sẽ gặp ở chương 11.

### 1.7 Lỗi thường gặp

- `float x = 1.5;` thiếu `f`.
- `int / int` ra số nguyên khi bạn muốn số thập phân.
- So sánh `==` hai `float`.
- `(int)` cắt phần thập phân thay vì làm tròn.
- Quên rằng biến cục bộ phải được gán trước khi dùng (`CS0165`).

### 1.8 Tự kiểm tra

1. Vì sao lỗi sai kiểu được phát hiện khi build chứ không phải khi chơi?
2. `7 / 2`, `7 / 2f`, `7 % 2`, `(int)3.9f` lần lượt cho giá trị nào?
3. Vì sao không nên so sánh `==` hai số `float`? Dùng gì thay thế trong Godot?
4. `const` khác `readonly` thế nào?
5. Tại sao nối chuỗi trong vòng lặp lớn là không tốt?
6. Khi nào dùng `int?`?

### 1.9 Bài tập

1. In ra kết quả của `int.MaxValue + 1` (unchecked) rồi bọc trong `checked` và xem lỗi.
2. In `0.1 + 0.2` với 17 chữ số thập phân (`$"{0.1 + 0.2:F17}"`) và giải thích.
3. Viết hàm đổi giây thành chuỗi `"mm:ss"` (ví dụ 125 giây → `"02:05"`).
4. Viết đoạn tính phần trăm máu (`int hp, int maxHp` → `float`) đúng kiểu, không bị cắt về 0.

## Chương 2. Điều khiển luồng \[Cơ bản\]

**Mục tiêu:** viết được điều kiện và vòng lặp rõ ràng, dùng được pattern matching, biết vì sao không được viết vòng lặp vô hạn trong `_Process`.

### 2.1 Điều kiện

```csharp
if (hp <= 0) { Die(); }
else if (hp < 30) { ShowWarning(); }
else { Heal(); }

// toán tử logic: && (và), || (hoặc), ! (không)
if (isGrounded && jumpPressed) Jump();
```

**Ngắt mạch (short-circuit):** `a && b` không tính `b` nếu `a` đã sai; `a || b` không tính `b` nếu `a` đã đúng. Nhờ vậy `if (enemy != null && enemy.Hp > 0)` an toàn. Viết ngược thứ tự sẽ gây `NullReferenceException`.

**Guard clause (thoát sớm)** giúp code phẳng và dễ đọc hơn nhiều `if` lồng nhau:

```csharp
public void Attack(Enemy target)
{
    if (target == null) return;
    if (target.Dead) return;
    if (_cooldown > 0f) return;
    target.TakeDamage(Damage);        // logic chính nằm ở cấp ngoài cùng
}
```

### 2.2 switch và pattern matching

```csharp
// switch dạng câu lệnh
switch (state)
{
    case State.Idle: DoIdle(); break;
    case State.Run:  DoRun();  break;
    default: break;
}

// switch dạng biểu thức (ngắn gọn, trả về giá trị)
string label = state switch
{
    State.Idle => "đứng",
    State.Run  => "chạy",
    _          => "khác",
};

// pattern theo khoảng giá trị
string rank = score switch
{
    >= 90 => "A",
    >= 70 => "B",
    >= 50 => "C",
    _     => "D",
};

// pattern kiểu và thuộc tính
if (body is Enemy { Hp: > 0 } e) e.TakeDamage(1);
if (x is >= 0 and < 10) { /* 0..9 */ }
```

Pattern matching cho phép kiểm tra **kiểu**, **giá trị**, **khoảng** và **thuộc tính** trong một biểu thức. Các `case` của `switch` biểu thức phải **phủ hết** mọi trường hợp (`_` là trường hợp còn lại).

### 2.3 Vòng lặp

```csharp
for (int i = 0; i < 5; i++) { }                 // biết trước số lần
foreach (var e in enemies) { e.Alert(); }       // duyệt mọi phần tử
while (queue.Count > 0) { Process(queue.Dequeue()); }   // lặp theo điều kiện
do { TryConnect(); } while (!connected);       // chạy ít nhất một lần
```

`break` thoát vòng lặp, `continue` bỏ qua lần lặp hiện tại.

**Quy tắc quan trọng:**

- **Không sửa collection khi đang `foreach` nó** (thêm/xoá phần tử gây `InvalidOperationException`). Muốn xoá trong khi duyệt, duyệt ngược bằng `for` từ cuối về đầu, hoặc gom các phần tử cần xoá rồi xoá sau, hoặc dùng `RemoveAll`.
- **Trong Godot, không viết vòng lặp có thể chạy vô hạn hoặc rất lâu trong `_Process`/`_PhysicsProcess`**: engine chạy trên một luồng chính, vòng lặp lâu sẽ đóng bƒÉng toàn bộ game. Muốn chờ, dùng Timer hoặc `await` (chương 8, 12).
- Duyệt mỗi frame qua hàng ngàn phần tử với thao tác nặng là nguồn giật hình thường gặp; đo bằng Profiler trước.

### 2.4 Lỗi thường gặp

- Dùng `=` thay cho `==` trong điều kiện (thường trình biên dịch bắt được với biến `int`, nhưng cẩn thận với `bool`).
- Quên `break` trong `switch` câu lệnh (C# báo lỗi, không "rơi" sang case sau như C/C++).
- Off-by-one: `i <= list.Count` thay vì `i < list.Count`, gây `ArgumentOutOfRangeException`.
- Xóa phần tử trong `foreach`.

### 2.5 Tự kiểm tra

1. Vì sao `if (e != null && e.Hp > 0)` an toàn nhưng đảo vế thì không?
2. Guard clause là gì và lợi ích?
3. Viết `switch` biểu thức phân loại máu: `>= 70` “An toàn”, `>= 30` “Cảnh báo”, còn lại “Nguy hiểm”.
4. Tại sao không được xoá phần tử khi đang `foreach`? Có cách nào xoá an toàn?
5. Vì sao vòng `while` chạy lâu trong `_Process` làm game đứng hình?

### 2.6 Bài tập

1. FizzBuzz từ 1 đến 30 bằng `for` và `switch` biểu thức.
2. Cho `List<int>`, xoá tất cả số chẵn theo hai cách: duyệt ngược bằng `for`, và `RemoveAll`.
3. Viết hàm `Classify(object o)` dùng pattern matching trả về chuỗi mô tả cho `int`, `string`, `null`, `Vector2`.
4. Duyệt một mảng 2D 5x5 bằng hai vòng `for` lồng nhau và in ra đường chéo chính.

## Chương 3. Hàm (phương thức) \[Cơ bản\]

**Mục tiêu:** viết hàm đúng chữ ký, hiểu cách truyền tham số (theo giá trị và theo tham chiếu), dùng được overload, tuple, local function.

### 3.1 Chữ ký hàm

```csharp
// [truy cập] [static?] kiểu_trả_về Tên(tham_số)
public static int Cong(int a, int b)
{
    return a + b;
}

private bool IsAlive() => Hp > 0;       // hàm biểu thức (expression-bodied)
```

**Quy tắc thiết kế:** một hàm **làm một việc**, tên là động từ (`TakeDamage`, `SpawnCoin`), ngắn đủ để đọc ở một màn hình, ít tham số (quá 4–5 tham số thì nên gom thành một class/struct).

### 3.2 Truyền tham số: theo giá trị và theo tham chiếu

Mặc định C# **truyền theo giá trị (copy)**:

- Với **kiểu giá trị** (`int`, `float`, `bool`, `struct` như `Vector2`): hàm nhận một **bản sao**; sửa bản sao không ảnh hưởng biến gốc.
- Với **kiểu tham chiếu** (`class`, như `Node`, `List<T>`, `string`): hàm nhận một **bản sao của con trỏ**, nên sửa **nội dung** đối tượng là thấy ở ngoài, nhưng gán lại tham số sang đối tượng khác thì không ảnh hưởng ngoài.

```csharp
void Bump(int x) { x++; }                       // không đổi biến gốc
void Bump(ref int x) { x++; }                   // đổi biến gốc

int n = 5;
Bump(n);          // n vẫn 5
Bump(ref n);      // n thành 6 (bắt buộc ghi ref ở cả lúc gọi)
```

| Từ khóa | Ý nghĩa | Điều kiện |
| --- | --- | --- |
| `ref` | truyền **tham chiếu**: hàm đọc và ghi biến gốc | biến phải đã được gán trước |
| `out` | hàm **trả thêm** giá trị qua tham số | hàm **bắt buộc** gán trước khi thoát |
| `in` | truyền tham chiếu **chỉ đọc** (tránh copy struct lớn) | hàm không được sửa |

`out` hay gặp nhất trong mẫu `TryXxx`:

```csharp
if (dict.TryGetValue("kiem", out int price)) { /* có */ } else { /* không có */ }
if (int.TryParse(text, out int value)) { }
```

**Ý nghĩa với Godot:** `Vector2` là struct (copy khi truyền), nên `Velocity.X = 5` không được phép (bạn sửa bản sao của thuộc tính); phải `Velocity = new Vector2(5, Velocity.Y)`. Hiểu điều này sẽ giải thích rất nhiều lỗi bạn gặp.

### 3.3 Tham số tùy chọn, đặt tên, params

```csharp
void Spawn(Vector2 pos, int count = 1, bool random = false) { }

Spawn(pos);                         // dùng mặc định
Spawn(pos, random: true);           // truyền theo TÊN, bỏ qua count

int Sum(params int[] numbers) { int s = 0; foreach (var n in numbers) s += n; return s; }
Sum(1, 2, 3, 4);                    // truyền bao nhiêu tham số cũng được
```

### 3.4 Overload (nạp chồng)

Nhiều hàm **cùng tên, khác tham số**; trình biên dịch chọn bản khớp. Khác tham số nghĩa là khác **số lượng hoặc kiểu**; chỉ khác kiểu trả về thì không được.

```csharp
void Damage(int amount) { }
void Damage(int amount, Node source) { }
void Damage(float percent) { }
```

### 3.5 Tuple, trả về nhiều giá trị, local function

```csharp
(int min, int max) GetRange(int[] a)
{
    int lo = a[0], hi = a[0];
    foreach (var x in a) { lo = Math.Min(lo, x); hi = Math.Max(hi, x); }
    return (lo, hi);
}

var (min, max) = GetRange(numbers);          // tách (deconstruct)

int Total(int[] a)
{
    return Add(a);
    static int Add(int[] items)                 // hàm cục bộ: chỉ dùng trong hàm chứa nó
    {
        int s = 0; foreach (var i in items) s += i; return s;
    }
}
```

### 3.6 Đệ quy và độ sâu

Hàm gọi chính nó (đệ quy) phải có **điều kiện dừng**, và độ sâu quá lớn gây `StackOverflowException` làm sập chương trình. Duyệt cây node bằng đệ quy thì tiện, nhưng với dữ liệu rất sâu (bản đồ lớn, flood fill) nên dùng `Stack<T>`/`Queue<T>` thay cho đệ quy.

### 3.7 Liên hệ với Godot

- Các hàm `_Ready`, `_Process`... là **hàm ghi đè (`override`)** mà engine gọi (chương 4).
- `CallDeferred(MethodName.Foo)` gọi hàm **muộn** (cuối frame) bằng tên; `Callable.From(...)` bọc một hàm để truyền đi như dữ liệu (chương 7, 12).
- Hàm gọi được từ Godot hay GDScript thì nên `public` và dùng kiểu mà Godot hỗ trợ.

### 3.8 Lỗi thường gặp

- Nghĩ rằng sửa `Vector2` hoặc `int` trong hàm sẽ đổi biến gốc (phải dùng `ref` hoặc trả về giá trị mới).
- Quên gán `out` trước `return` (`CS0177`).
- Quá nhiều tham số `bool` liên tiếp (`Spawn(p, true, false, true)`), khó đọc; dùng tham số đặt tên hoặc enum.

### 3.9 Tự kiểm tra

1. Truyền `int` và truyền `List<int>` vào hàm khác nhau thế nào khi hàm sửa chúng?
2. `ref`, `out`, `in` khác nhau ở đâu? Cho một ví dụ `out` quen thuộc.
3. Vì sao `Velocity.X = 5` lỗi trong C# mà `Velocity = new Vector2(5, Velocity.Y)` thì được?
4. Overload khác tham số tên mặc định thế nào? Khi nào dùng cái nào?
5. Khi nào nên dùng `Stack<T>` thay vì đệ quy?

### 3.10 Bài tập

1. Viết `Swap(ref int a, ref int b)` và kiểm tra.
2. Viết `bool TryDivide(int a, int b, out float result)` trả `false` nếu `b == 0`.
3. Viết `(float avg, float max) Stats(float[] values)` rồi dùng deconstruct.
4. Viết hàm `Factorial` đệ quy và bản dùng vòng lặp; thử với số lớn để thấy tràn số.
5. Tạo 3 overload của `Heal` (theo số máu, theo phần trăm, theo `Item`) và gọi thử từng cái.

## Chương 4. Lập trình hướng đối tượng (OOP) \[Cơ bản → Trung bình\]

**Mục tiêu:** thiết kế class có dữ liệu và hành vi hợp lý; phân biệt class, struct, record, interface; biết khi nào dùng kế thừa và khi nào dùng ghép (composition); hiểu `static` nguy hiểm thế nào trong Godot.

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

| Từ khoá | Ai truy cập được |
| --- | --- |
| `public` | mọi nơi |
| `private` | chỉ trong chính class (**mặc định** nếu không ghi) |
| `protected` | class và các class **con** |
| `internal` | cùng assembly (cùng project) |

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

**Kế thừa** (`:`) cho class con dùng lại và mở rộng class cha. Quan hệ phải là **"là một" (is-a)**: Kỵ binh *là một* đơn vị.

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

|  | Abstract class | Interface |
| --- | --- | --- |
| Có code dùng chung | có | không (thường) |
| Số lượng kế thừa | 1 | nhiều |
| Dùng khi | các lớp có chung bản chất và code | các lớp không liên quan nhưng chung một khả năng |

### 4.6 Ghép (composition) thay vì kế thừa sâu

Kế thừa sâu (`Entity → Character → Enemy → FlyingEnemy → BossFlyingEnemy`) cứng nhắc và khó đổi. Thay vào đó **ghép các thành phần nhỏ**: một `Enemy` *có* `Health`, `Mover`, `Hitbox`. Đây cũng chính là cách Godot làm việc: scene là cây các node nhỏ, mỗi node một nhiệm vụ. Quy tắc: **"ưu tiên has-a hơn is-a"**; chỉ kế thừa khi quan hệ thật sự là is-a và lớp cha ổn định.

### 4.7 struct, class và record

|  | `class` | `struct` | `record` |
| --- | --- | --- | --- |
| Kiểu | **tham chiếu** | **giá trị** | tham chiếu (có thể `record struct`) |
| Gán `b = a` | hai biến cùng trỏ một đối tượng | **copy** toàn bộ | cùng trỏ một đối tượng |
| So sánh `==` | so con trỏ (cùng đối tượng?) | so từng trường (tuỳ) | so **nội dung** |
| Dùng khi | đối tượng có danh tính, vòng đời, hành vi | dữ liệu **nhỏ**, bất biến (vector, màu, toạ độ ô) | dữ liệu thuần, bất biến (save, thông tin sát thương) |

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

## Chương 5. Collection và generic \[Cơ bản → Trung bình\]

**Mục tiêu:** chọn đúng cấu trúc dữ liệu cho từng nhu cầu, biết chi phí của chúng, viết được generic đơn giản, dùng LINQ có ý thức, phân biệt collection của .NET và của Godot.

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

| Cấu trúc | Truy cập chỉ số | Thêm | Tìm / Contains | Dùng khi |
| --- | --- | --- | --- | --- |
| `T[]` | O(1) | không đổi kích thước | O(n) | dữ liệu cố định, hiệu năng cao |
| `List<T>` | O(1) | cuối O(1)\*, giữa O(n) | O(n) | danh sách chung, thứ tự quan trọng |
| `Dictionary<K,V>` | — | O(1)\* | theo khoá O(1)\* | tra cứu theo id/tên |
| `HashSet<T>` | — | O(1)\* | O(1)\* | kiểm tra "đã có chưa", loại trùng |
| `Queue<T>` | — | cuối O(1) | — | hàng đợi vào trước ra trước (BFS, lệnh) |
| `Stack<T>` | — | đầu O(1) | — | vào sau ra trước (DFS, undo) |
| `PriorityQueue<E,P>` | — | O(log n) | — | lấy phần tử ưu tiên nhất (A\*, sự kiện theo giờ) |

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

| Nhu cầu | Dùng |
| --- | --- |
| Dữ liệu nội bộ trong code C# | `List<T>`, `Dictionary<K,V>`, `HashSet<T>`, `T[]` |
| Biến `[Export]` hiện trong Inspector | `Godot.Collections.Array<T>`, `Godot.Collections.Dictionary<K,V>`, hoặc mảng C# như `int[]`, `string[]` |
| Tham số của signal, `Call`, kết quả `GetChildren()` | `Godot.Collections.Array`/`Dictionary` |

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

## Chương 6. null, ngoại lệ và giải phóng tài nguyên \[Trung bình\]

**Mục tiêu:** hiểu vì sao `NullReferenceException` xảy ra và phòng tránh có hệ thống; dùng `try/catch` đúng chỗ; biết `using` và `IDisposable` dùng để làm gì.

### 6.1 null và NullReferenceException

Biến kiểu tham chiếu (`class`, `string`, `Node`...) có thể là `null` ("chưa trỏ đến đối tượng nào"). Gọi phương thức hay đọc thuộc tính trên `null` ném `NullReferenceException`, lỗi phổ biến nhất của người mới. Nguyên nhân hay gặp trong Godot:

- Quên kéo node vào ô `[Export]` trong Inspector.
- `GetNodeOrNull` trả `null` vì sai đường dẫn; hoặc dùng biến trước `_Ready`.
- Dùng đối tượng sau khi nó đã bị `QueueFree` (lúc này là `ObjectDisposedException`, xem chương 11).
- Hàm tìm kiếm (`FirstOrDefault`, `TryGetValue`...) trả `null`/mặc định mà không kiểm tra.

**Công cụ phòng tránh:**

```csharp
if (target != null) target.Hit();            // kiểm tra tường minh
target?.Hit();                                // gọi nếu không null (null-conditional)
string name = target?.Name ?? "không có";     // ?? giá trị dự phòng
_cache ??= LoadCache();                       // gán nếu đang null
if (target is null) return;                   // is null / is not null
if (target is not null) target.Hit();
```

**Nullable reference types (NRT):** tính năng của trình biên dịch giúp **bắt lỗi null lúc build**. Bật bằng `<Nullable>enable</Nullable>` trong `.csproj` (hoặc `#nullable enable` đầu file). Khi bật, `string` nghĩa là "không bao giờ null", `string?` nghĩa là "có thể null", và trình biên dịch cảnh báo khi bạn dùng giá trị có thể null mà chưa kiểm tra. Với biến `[Export]` được Godot gán sau khi tạo (nên trình biên dịch thấy là chưa khởi tạo), cách viết hay dùng:

```csharp
[Export] private Sprite2D _sprite = null!;     // '!' hứa với trình biên dịch: sẽ được gán trước khi dùng
```

Dấu `!` chỉ là **lời hứa**, không kiểm tra gì; nếu bạn quên kéo node thì vẫn lỗi lúc chạy. Hãy xem lại file `.csproj` của project để biết NRT có đang bật không.

### 6.2 Ngoại lệ (exception)

Ngoại lệ là cơ chế báo **tình huống bất thường** (chỉ số ngoài phạm vi, file không tồn tại, null...). Khi ném, chương trình nhảy ngược lên các hàm đang gọi cho đến khi gặp `catch` phù hợp; không có thì ngoại lệ không được xử lý.

```csharp
try
{
    var data = LoadSave(path);
}
catch (FileNotFoundException)                 // bắt kiểu cụ thể trước
{
    data = new SaveData();                    // phương án dự phòng hợp lý
}
catch (Exception e)                           // kiểu chung để cuối
{
    GD.PushError($"Lỗi tải save: {e.Message}");
    throw;                                    // ném lại, GIỮ nguyên stack trace
}
finally
{
    // luôn chạy dù có lỗi hay không (dọn dẹp)
}
```

**Nguyên tắc dùng ngoại lệ:**

1. **Ngăn lỗi trước khi nó xảy ra** (kiểm tra null, kiểm tra biên) tốt hơn bắt lỗi sau.
2. **Lỗi dự đoán được** (người chơi nhập sai, khoá không tồn tại) thì dùng mẫu `TryXxx` hoặc kiểm tra điều kiện, không dùng ngoại lệ làm luồng điều khiển; ném/bắt ngoại lệ **chậm**.
3. **Đừng nuốt lỗi** (`catch { }` rỗng): bạn sẽ không bao giờ biết vì sao game sai. Ít nhất hãy ghi log.
4. Ném lại bằng `throw;` (không phải `throw e;`) để giữ stack trace.
5. Bắt kiểu **cụ thể** thay vì `Exception` chung khi có thể.

**Trong Godot:** ngoại lệ không được xử lý bên trong hàm callback (`_Process`, signal handler...) thường được Godot **in ra tab Debugger/Output** và bỏ dở phần còn lại của hàm đó; game thường không tự thoát, nhưng trạng thái có thể đã hỏng nửa chừng. Vì vậy hãy xem tab lỗi mỗi lần chạy thử và sửa mọi lỗi đỏ.

### 6.3 Lập trình phòng thủ

- Kiểm tra tham số đầu vào ở đầu hàm công khai (guard clause, chương 2).
- `System.Diagnostics.Debug.Assert(cond, "mô tả")` kiểm tra giả định lúc phát triển.
- `GD.PushError`/`GD.PushWarning` ghi lỗi vào Debugger nhưng không dừng game.
- Đặt giá trị mặc định hợp lý thay vì để `null`.

### 6.4 IDisposable và using

Một số tài nguyên cần được **giải phóng chủ động** (file, kết nối, bộ đệm hệ thống). Chúng cài đặt `IDisposable`; `using` đảm bảo `Dispose()` được gọi khi ra khỏi phạm vi, kể cả khi có ngoại lệ:

```csharp
using var file = FileAccess.Open("user://save.json", FileAccess.ModeFlags.Read);
string text = file.GetAsText();
// file tự đóng khi ra khỏi hàm

using (var f2 = FileAccess.Open(path, FileAccess.ModeFlags.Write))
{
    f2.StoreString(json);
}                                              // đóng ở đây
```

Với đối tượng Godot: `Node` giải phóng bằng `QueueFree()`; các kiểu đếm tham chiếu (`RefCounted`, `Resource`, `FileAccess`) tự giải phóng khi không còn ai giữ, nhưng nên `using` để đóng ngay (đặc biệt với file). Chi tiết ở chương 11 và 16.

### 6.5 Lỗi thường gặp

- `catch (Exception) { }` rỗng.
- Dùng `!` (null-forgiving) khắp nơi để tắt cảnh báo thay vì xử lý.
- Ném/bắt ngoại lệ trong vòng lặp nóng.
- Quên đóng file (không dùng `using`).

### 6.6 Tự kiểm tra

1. Kể 3 nguyên nhân thường gây `NullReferenceException` trong Godot.
2. `?.`, `??`, `??=` làm gì? Cho ví dụ ngắn.
3. NRT giúp được gì và `!` có nghĩa gì?
4. Vì sao không nên `catch { }` rỗng? `throw;` khác `throw e;` thế nào?
5. Khi nào dùng `TryXxx` thay vì ném ngoại lệ?
6. `using` đảm bảo điều gì?

### 6.7 Bài tập

1. Cố tình gây `NullReferenceException`, đọc stack trace, rồi sửa bằng ba cách khác nhau (`if`, `?.`, khởi tạo mặc định).
2. Viết `bool TryLoad(string path, out SaveData data)` không ném ngoại lệ ra ngoài.
3. Viết hàm đọc file bằng `using`; thử với file không tồn tại và xử lý lỗi.
4. Bật NRT trong project console, quan sát cảnh báo và sửa hết.

## Chương 7. Delegate, lambda và event \[Trung bình\]

**Mục tiêu:** hiểu "hàm như dữ liệu", viết lambda, hiểu closure và bẫy của nó, dùng `event` an toàn, tránh rò rỉ đăng ký.

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

## Chương 8. async/await và Task \[Trung bình → Nâng cao\]

**Mục tiêu:** hiểu vì sao cần bất đồng bộ, dùng `await` trong Godot đúng cách, biết khi nào dùng luồng nền và những điều không được làm.

### 8.1 Vì sao cần bất đồng bộ

Godot chạy logic game trên **một luồng chính**. Nếu bạn chờ (đọc file lớn, mạng, tính toán lâu) ngay trên luồng này, **cả game đứng hình**. Giải pháp: bắt đầu việc rồi **nhường quyền** cho engine tiếp tục, và quay lại khi xong. `async/await` cho phép viết việc đó như code tuần tự.

```csharp
public async void OnPlayerDied()
{
    _sprite.Play("dead");
    await ToSignal(GetTree().CreateTimer(1.0), SceneTreeTimer.SignalName.Timeout);   // chờ 1 giây (game vẫn chạy)
    GetTree().ReloadCurrentScene();                                                  // chạy tiếp sau khi chờ xong
}
```

Khi gặp `await`, hàm **tạm dừng và trả quyền điều khiển**; phần còn lại (continuation) chạy khi việc được chờ hoàn thành. Trình biên dịch biến hàm `async` thành một **máy trạng thái** nhớ vị trí đang dừng.

### 8.2 Task

`Task` đại diện cho một **công việc sẽ hoàn thành trong tương lai**; `Task<T>` còn mang kết quả. Hàm `async` trả `Task`/`Task<T>` (hoặc `void` cho callback sự kiện).

```csharp
private async Task<int> CountAsync()
{
    return await Task.Run(() => HeavyCount());    // việc nặng chạy trên luồng nền
}

public async void Start()
{
    int n = await CountAsync();                   // chờ mà không chặn game
    _label.Text = n.ToString();                   // đã quay về luồng chính: dùng node an toàn
}
```

- **`Task.Run`** đẩy việc **tính toán thuần** sang luồng nền. Trong Godot, sau `await`, phần tiếp theo chạy lại trên **luồng chính** nhờ ngữ cảnh đồng bộ (synchronization context) của Godot, nên chạm node ở dòng sau `await` là an toàn.
- **Đừng chạm vào node/API engine bên trong `Task.Run`** (lambda chạy luồng nền): API Godot không an toàn đa luồng. Nếu bắt buộc, dùng `CallDeferred`/`Callable.From(...).CallDeferred()` để đẩy về luồng chính.

### 8.3 async void và async Task

|  | `async Task` | `async void` |
| --- | --- | --- |
| Chờ/lấy kết quả được | có | không |
| Bắt ngoại lệ được | có (`try/await`) | khó (ngoại lệ không về nơi gọi) |
| Dùng khi | hầu hết hàm bất đồng bộ | **chỉ** callback sự kiện/hàm vòng đời |

Hàm vòng đời, handler signal/event cần kiểu `void`, nên dùng `async void` ở đó và **tự bọc `try/catch`** nếu có thể ném lỗi.

### 8.4 Thời gian trong game: dùng timer của Godot, không dùng `Task.Delay`

`await Task.Delay(1000)` chờ 1 giây **thời gian thật**, không biết đến pause, `TimeScale`, hay việc scene bị đổi. `GetTree().CreateTimer(1.0)` chạy theo **thời gian của game**, tôn trọng pause (mặc định) và `Engine.TimeScale`. Trong code game, chọn timer của Godot (hoặc `Tween`, `AnimationPlayer`).

### 8.5 Sau `await`, node có thể đã chết

Trong lúc chờ, node có thể bị `QueueFree`, scene có thể đã đổi. Dòng sau `await` chạy trên một đối tượng đã bị huỷ sẽ ném `ObjectDisposedException`. Phòng bằng:

```csharp
await ToSignal(GetTree().CreateTimer(2.0), SceneTreeTimer.SignalName.Timeout);
if (!IsInstanceValid(this)) return;           // node còn sống không?
DoSomething();
```

### 8.6 Những điều không được làm

- **Không `.Result` hoặc `.Wait()`** trên luồng chính để "chờ đồng bộ": dễ gây **deadlock** (treo vĩnh viễn) và đứng hình.
- Không gọi `Task.Run` mỗi frame cho việc nhỏ (chi phí cao hơn lợi ích).
- Không dùng biến chia sẻ giữa luồng mà không đồng bộ hoá (race condition); ưu tiên truyền **dữ liệu bất biến** giữa luồng.
- Không bỏ quên (fire-and-forget) một `Task` có thể lỗi mà không ai quan sát.

### 8.7 Huỷ công việc

`CancellationTokenSource` cho phép yêu cầu dừng một công việc dài: truyền `token` vào, công việc kiểm tra `token.IsCancellationRequested` hoặc gọi `ThrowIfCancellationRequested()`. Trong Godot thường cần khi người chơi rời màn khi việc nền chưa xong.

### 8.8 Lỗi thường gặp

- Quên `async` ở hàm có `await` (lỗi biên dịch `CS4033`).
- Dùng `Task.Delay` cho nhịp game rồi lệch khi pause.
- Chạm node trong `Task.Run`.
- Không kiểm tra node còn sống sau `await`.
- `.Result` gây treo.

### 8.9 Tự kiểm tra

1. Vì sao chờ đồng bộ trên luồng chính làm game đứng hình?
2. `await` làm gì với dòng chạy của hàm?
3. `Task.Run` dùng khi nào và có được chạm node trong đó không?
4. `async void` khác `async Task` ra sao? Khi nào chấp nhận `async void`?
5. Vì sao dùng timer của Godot thay vì `Task.Delay`?
6. Làm sao tránh lỗi khi node bị xoá trong lúc chờ?

### 8.10 Bài tập

1. Viết hàm `async Task Flash()` làm sprite nhấp nháy 3 lần bằng `ToSignal` timer.
2. Dùng `Task.Run` tính tổng một mảng 10 triệu phần tử, hiển thị kết quả trong Label sau khi xong; game vẫn mượt trong lúc tính.
3. Cố tình `QueueFree` node ngay sau khi `await` bắt đầu, quan sát lỗi, rồi thêm `IsInstanceValid` để sửa.
4. Viết một hàm có `CancellationToken` và huỷ nó sau 0,5 giây.

## Chương 9. Bộ nhớ, tham chiếu và GC \[Nâng cao\]

**Mục tiêu:** hiểu giá trị/tham chiếu ở mức bộ nhớ, biết cái gì sinh "rác", và giảm cấp phát đúng chỗ.

### 9.1 Giá trị và tham chiếu ở mức bộ nhớ

- **Kiểu giá trị** (`int`, `float`, `bool`, `struct`, `enum`): dữ liệu nằm **ngay tại biến** (trên stack khi là biến cục bộ, hoặc nằm bên trong đối tượng chứa nó). Gán là **copy**.
- **Kiểu tham chiếu** (`class`, `string`, mảng, `List<T>`, delegate, `Node`): đối tượng nằm trên **heap**; biến chỉ giữ một **con trỏ** tới nó. Gán chỉ **copy con trỏ**, hai biến cùng trỏ một đối tượng.

Hệ quả: sửa đối tượng qua một biến thì mọi biến khác trỏ cùng đối tượng đều thấy; còn struct thì mỗi biến một bản riêng.

### 9.2 Bộ dọn rác (GC)

Bạn **không giải phóng** bộ nhớ heap thủ công trong C#. GC của .NET định kỳ tìm các đối tượng không còn ai tham chiếu rồi thu hồi. GC chia đối tượng theo **thế hệ** (gen 0 mới và hay chết, gen 1, gen 2 sống lâu); gen 0 được dọn thường xuyên và rẻ, gen 2 đắt hơn. Khi GC chạy, nó có thể **làm game khựng ngắn**. Càng ít cấp phát đối tượng tạm, GC càng ít phải dọn, game càng mượt. Điều này quan trọng với game quân số lớn.

### 9.3 Cái gì sinh ra rác

| Nguồn | Ví dụ |
| --- | --- |
| `new` một class mỗi frame | `new List<int>()`, `new Vector2[...]` trong `_Process` |
| Nối/định dạng chuỗi | `"a" + b`, `$"..."` mỗi frame |
| **Boxing** | gán struct/số vào `object`, collection không generic |
| Closure/lambda bắt biến | tạo lambda mới liên tục |
| LINQ | `Where/Select/ToList` |
| `params` | gọi hàm `params` tạo mảng mới |
| Duyệt qua `IEnumerable<T>` | trình duyệt struct bị đóng hộp khi đi qua interface |

**Boxing:** khi một kiểu giá trị được lưu vào biến kiểu `object` hoặc interface, nó được **đóng hộp** lên heap (cấp phát) và mở hộp khi lấy ra. Tránh bằng cách dùng generic (`List<int>`, không phải `ArrayList`) và tránh hàm nhận `object` ở đường nóng.

### 9.4 Cách giảm cấp phát

- **Tái sử dụng** thay vì tạo mới: giữ một `List<T>` làm bộ đệm và gọi `Clear()` rồi dùng lại (như `SpatialHash.Query` ở Phụ lục F trong tài liệu chính).
- **Object pooling:** tái sử dụng node (đạn, hiệu ứng) thay vì `Instantiate`/`QueueFree` liên tục.
- Dùng `struct` cho dữ liệu nhỏ, bất biến; `Span<T>`/`stackalloc` cho bộ đệm tạm nhỏ.
- `StringBuilder` khi nối nhiều lần; **chỉ cập nhật chuỗi hiển thị khi giá trị thay đổi** (không dựng chuỗi mỗi frame cho Label).
- Cache delegate/lambda thay vì tạo lại; dùng lambda `static`.
- Đặt trước dung lượng: `new List<T>(1000)` để tránh nhân đôi nhiều lần.
- Không dùng LINQ trong vòng lặp nóng.

### 9.5 Đo trước khi tối ưu

- Godot: tab **Debugger > Monitors** (bộ nhớ, số object, orphan node) và **Profiler** (thời gian mỗi hàm).
- Trong code: `GC.CollectionCount(0)` đếm số lần dọn gen 0; so sánh trước/sau một đoạn để biết đoạn đó có gây rác không. `GC.GetTotalMemory(false)` xem bộ nhớ managed.
- Với đo vi mô một hàm thuần C#, dùng project console cùng thư viện **BenchmarkDotNet** (cách đo nghiêm túc hơn việc tự bấm giờ).

Nguyên tắc: **đừng tối ưu theo cảm tính**. Tối ưu sớm làm code khó đọc mà chưa chắc nhanh hơn; hãy đo, tìm điểm nghẽn thật, sửa, rồi đo lại.

### 9.6 Điều riêng của Godot

Đối tượng Godot (như `Node`) có **hai nửa**: đối tượng native trong engine (C++) và "vỏ" C# trỏ tới nó. `QueueFree()` giải phóng nửa native (và con của nó); vỏ C# vẫn còn đến khi GC dọn, nhưng dùng nó sẽ ném `ObjectDisposedException`. Vì thế **GC không giải phóng node cho bạn**: node không thêm vào cây hoặc không `QueueFree` sẽ thành **orphan node** (rò rỉ). Chi tiết ở chương 11.

### 9.7 Lỗi thường gặp

- Tạo `List`/mảng/chuỗi mới mỗi frame trong `_Process`.
- Giữ danh sách tham chiếu đến node đã xoá (rò rỉ và lỗi truy cập).
- Giữ sự kiện/lambda tham chiếu đến đối tượng cần thu hồi.
- Tối ưu khi chưa đo.
- Dùng struct quá lớn bị copy nhiều lần.

### 9.8 Tự kiểm tra

1. Khác biệt giữa gán kiểu giá trị và kiểu tham chiếu ở mức bộ nhớ?
2. GC làm gì và vì sao có thể gây khựng game?
3. Kể 4 nguồn thường sinh rác trong `_Process`.
4. Boxing là gì và tránh thế nào?
5. Orphan node là gì, vì sao GC không dọn chúng?
6. Vì sao nên đo bằng Profiler trước khi tối ưu?

### 9.9 Bài tập

1. Trong project console, so sánh `GC.CollectionCount(0)` trước/sau khi nối chuỗi 100 000 lần bằng `+` và bằng `StringBuilder`.
2. Viết đoạn dùng `ArrayList` (boxing) và `List<int>` cho 1 triệu số; đo thời gian và số lần GC.
3. Viết hàm tái sử dụng một `List<int>` bộ đệm qua nhiều lần gọi thay vì tạo mới.
4. Tạo scene có script `Instantiate` một node mỗi frame mà không xoá; quan sát Monitors tăng, rồi sửa.

---

# PHẦN 2. C# TRONG GODOT

## Chương 10. Node, Scene, SceneTree và Resource \[Cơ bản → Trung bình\]

**Mục tiêu:** hiểu mô hình đối tượng của Godot, thứ tự gọi các hàm vòng đời, cách các node giao tiếp, và khác biệt giữa Node và Resource.

### 10.1 Cây kế thừa của Godot

Mọi thứ bạn dùng đều là một **lớp C#** kế thừa nhau (đây chính là OOP ở chương 4 áp dụng vào engine):

```
GodotObject                        (gốc của mọi đối tượng engine)
 ├─ RefCounted                     (tự giải phóng khi không còn ai giữ)
 │   └─ Resource                   (dữ liệu dùng chung: Texture2D, PackedScene, script dữ liệu của bạn)
 └─ Node                           (có mặt trong cây cảnh)
     ├─ CanvasItem
     │   ├─ Node2D                 (có vị trí/xoay/tỉ lệ 2D)  → Sprite2D, CharacterBody2D, Area2D, Camera2D...
     │   └─ Control                (giao diện)                → Button, Label, Container...
     └─ Node3D ...
```

Khi bạn viết `public partial class Player : CharacterBody2D`, bạn **mở rộng** `CharacterBody2D`, thừa hưởng toàn bộ thuộc tính (`Position`, `Velocity`) và hàm (`MoveAndSlide`) của chuỗi kế thừa phía trên. Hiểu được cây này, bạn biết vì sao `Sprite2D` có `Position` còn `Resource` thì không.

### 10.2 Node, cây và scene

- **Node:** đơn vị chức năng nhỏ nhất, mỗi node một nhiệm vụ (vẽ ảnh, giữ hình va chạm, đếm giờ...).
- **Cây (tree):** các node có **một cha** và nhiều con; cha điều khiển con. Vị trí của con tính **tương đối** theo cha (chương 14).
- **Scene:** một cây node lưu thành file `.tscn`. Về mặt code, scene được nạp thành **`PackedScene`** (một Resource); `Instantiate<T>()` dựng ra một **bản sao cây node** sống.
- **`SceneTree`:** cây tổng của game đang chạy, gốc là `Window` (root). Scene hiện tại và các **autoload** đều là con của root. `GetTree()` trả về nó.

Tư duy Godot: chia game thành nhiều scene nhỏ **tự đủ dùng** (Player, Enemy, Coin, HUD), rồi ghép lại bằng cách thêm scene này làm con của scene khác. Đây là **ghép (composition)** ở chương 4.

**Đường dẫn node:** `GetNode<T>("Sprite2D")` (con), `"../Other"` (anh em), `"/root/GameManager"` (tuyệt đối), `"%Name"` (node đặt **Unique Name** trong cùng scene, không phụ thuộc vị trí). Đường dẫn là chuỗi, **gãy khi đổi tên hoặc chuyển node**, nên ưu tiên `[Export]` kéo thả hoặc `%Unique`.

**Parent và Owner:** `Parent` là cha trong cây; `Owner` là node gốc của scene mà node đó **thuộc về khi lưu** (quan trọng với công cụ editor và lưu scene bằng code; trong gameplay thường không cần đụng).

### 10.3 Vòng đời và thứ tự gọi

| Hàm | Khi nào | Thứ tự trong cây |
| --- | --- | --- |
| `_EnterTree()` | node vừa vào cây | **cha trước, con sau** |
| `_Ready()` | node và **mọi con** đã sẵn sàng; gọi **một lần** | **con trước, cha sau** |
| `_Process(delta)` | mỗi frame hiển thị | theo thứ tự cây (và `ProcessPriority`) |
| `_PhysicsProcess(delta)` | mỗi bước vật lý cố định (mặc định 60/giây) | như trên |
| `_ExitTree()` | node rời cây | **con trước, cha sau** |

Hệ quả quan trọng:

- Trong `_Ready` của cha, **các con đã sẵn sàng** nên `GetNode` đến con là an toàn. Ngược lại, con **không** được giả định cha đã `_Ready` xong.
- Node được thêm bằng `AddChild` sau này vẫn chạy `_EnterTree` rồi `_Ready` của nó ngay khi vào cây.
- `_Ready` chỉ chạy một lần dù node rời rồi vào lại cây.
- `QueueFree()` **không xoá ngay**; node bị xoá ở **cuối frame** (an toàn trong lúc đang duyệt, callback). `Free()` xoá **tức thì**, dễ gây lỗi và chỉ nên dùng khi hiểu rõ.

### 10.4 Các node nói chuyện với nhau thế nào

Quy tắc vàng: **"Gọi xuống, báo lên"** (call down, signal up):

- **Cha gọi hàm của con** trực tiếp (cha biết con là gì).
- **Con không nên biết cha**; thay vào đó con **phát signal**, cha nối vào và phản ứng. Nhờ vậy con dùng lại được ở nhiều nơi.

Các cách khác: node anh em giao tiếp qua cha chung hoặc qua `[Export]`; thông tin toàn cục qua **autoload**; nhắm theo nhóm qua **group** (`AddToGroup`, `GetTree().GetNodesInGroup`, `CallGroup`). Tránh đường dẫn kiểu `GetNode("../../../Foo")` vì nó gắn chặt cấu trúc cây.

### 10.5 Pause và chế độ xử lý

`GetTree().Paused = true` dừng các node có `ProcessMode` là *Pausable* (mặc định kế thừa từ cha). Menu pause cần `ProcessMode = When Paused` (hoặc *Always* cho nhạc) để vẫn nhận input khi game dừng. `SetProcess(false)`/`SetPhysicsProcess(false)` tắt cập nhật của riêng node (tiết kiệm CPU với node không hoạt động).

### 10.6 Node và Resource

|  | Node | Resource |
| --- | --- | --- |
| Bản chất | **hành vi** sống trong cây | **dữ liệu** lưu được ra file `.tres`/`.res` |
| Có `_Process`? | có | không |
| Giải phóng | `QueueFree()` | đếm tham chiếu (tự động) |
| Chia sẻ | mỗi node là một thể hiện riêng | **một Resource thường được nhiều nơi cùng dùng** |
| Ví dụ | `Player`, `Enemy`, `Label` | `ItemData`, `Texture2D`, `PackedScene`, theme |

**Bẫy chia sẻ Resource:** nếu hai node cùng `[Export] ItemData`, họ trỏ **cùng một đối tượng**; sửa trường trên một bên thì bên kia cũng đổi. Dùng Resource cho **dữ liệu tĩnh** (chỉ số cơ bản), còn **trạng thái động riêng** (máu hiện tại) đặt trong node hoặc class C# thuần. Cần bản riêng thì `Duplicate()` (hoặc bật *Local to Scene*).

### 10.7 Lỗi thường gặp

- Gọi `GetNode` trước `_Ready` hoặc sai đường dẫn (`null`/lỗi).
- Con giả định cha đã sẵn sàng.
- Xoá node bằng `Free()` giữa callback va chạm.
- Chia sẻ một Resource và vô tình sửa trạng thái chung.
- Chuỗi đường dẫn dài gãy khi đổi tên node.
- Quên rằng `QueueFree` xoá muộn, node vẫn "sống" đến hết frame.

### 10.8 Tự kiểm tra

1. Vẽ cây kế thừa từ `GodotObject` đến `CharacterBody2D` và đến `Resource`.
2. `_EnterTree`, `_Ready`, `_ExitTree` chạy theo thứ tự cha/con thế nào?
3. Vì sao `GetNode` đến con trong `_Ready` của cha là an toàn?
4. "Gọi xuống, báo lên" nghĩa là gì và vì sao tốt?
5. `QueueFree` khác `Free` ở đâu?
6. Khi nào dùng Resource, khi nào dùng Node? Bẫy chia sẻ Resource là gì?
7. Vì sao ưu tiên `[Export]` hơn đường dẫn chuỗi?

### 10.9 Bài tập

1. Tạo cây node nhỏ (cha và hai con), in thứ tự `_EnterTree`/`_Ready`/`_ExitTree` của cả ba; kiểm chứng bảng 10.3.
2. Viết node con phát signal `Clicked` và node cha nối vào; không để con tham chiếu tới cha.
3. Tạo `Resource` `Stats` chia sẻ cho hai node, sửa trên một bên, quan sát bên kia; sau đó dùng `Duplicate()` để tách.
4. Dùng `AddToGroup`/`CallGroup` để gọi một hàm trên 5 node cùng lúc.

## Chương 11. C# gắn với Godot như thế nào \[Trung bình → Nâng cao\]

**Mục tiêu:** hiểu vì sao có `partial`, `[Export]`, `Variant`, `StringName`; biết vòng đời đối tượng Godot trong C# và các lỗi liên quan; hiểu build và nạp lại assembly.

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

| Attribute | Tác dụng |
| --- | --- |
| `[Export]` | hiện biến/property trong Inspector, lưu cùng scene |
| `[ExportGroup]`, `[ExportSubgroup]`, `[ExportCategory]` | gom nhóm trong Inspector |
| `[Signal]` | khai báo signal (delegate kết thúc `EventHandler`) |
| `[GlobalClass]` | cho class hiện trong hộp thoại tạo Node/Resource |
| `[Tool]` | script chạy cả trong editor |

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

## Chương 12. Signal và event trong Godot \[Trung bình\]

**Mục tiêu:** khai báo, phát, kết nối, ngắt kết nối signal đúng; hiểu thứ tự gọi và các bẫy; biết khi nào dùng signal, khi nào dùng event C#.

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

|  | Signal (`[Signal]`) | `event` C# |
| --- | --- | --- |
| Hiện trong editor, nối bằng chuột | **có** | không |
| Nối với GDScript | **có** | không |
| Kiểu tham số | giới hạn kiểu tương thích Variant | **bất kỳ** kiểu C# |
| Tốc độ | chậm hơn (qua engine) | nhanh |
| Tự ngắt khi node bị xoá | **có** (Godot lo) | **không**, bạn phải `-=` |
| Nên dùng khi | giao tiếp giữa node, cần editor/GDScript | logic thuần C#, truyền kiểu tuỳ ý, đường nóng |

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

## Chương 13. Toán, thời gian và bước thời gian \[Trung bình\]

**Mục tiêu:** dùng đúng các kiểu toán của Godot, hiểu radian, `delta`, vòng lặp process/physics, và các bẫy của số thực và số ngẫu nhiên.

### 13.1 Các kiểu toán chính

| Kiểu | Ý nghĩa | Ghi chú |
| --- | --- | --- |
| `Vector2` | cặp `float` (x, y): vị trí, hướng, vận tốc | **struct** (copy theo giá trị) |
| `Vector2I` | cặp `int`: toạ độ ô tilemap, chỉ số lưới | không có phần thập phân |
| `Rect2` / `Rect2I` | hình chữ nhật (vị trí + kích thước) | `HasPoint`, `Intersects` |
| `Transform2D` | biến đổi (xoay, tỉ lệ, dịch) | đứng sau `Position/Rotation/Scale` của `Node2D` |
| `Color` | màu RGBA dạng `float` 0..1 | `Colors.Red` |
| `Mathf` | hàm toán (`float`) | `Lerp`, `Clamp`, `MoveToward`, `Remap`, `Wrap` |

Các phép vector quan trọng (đều có trong `Vector2`):

```csharp
var a = new Vector2(3, 4);
a.Length();                      // 5: độ dài
a.Normalized();                  // (0.6, 0.8): cùng hướng, độ dài 1
a.DistanceTo(b);                 // khoảng cách tới b
a.DirectionTo(b);                // vector đơn vị chỉ từ a tới b
a.Dot(b);                        // tích vô hướng: >0 cùng phía, 0 vuông góc, <0 ngược phía
a.Cross(b);                      // tích có hướng 2D (số thực): dấu cho biết b nằm bên trái hay phải a
a.Angle();                       // góc (radian) so với trục X
a.Rotated(Mathf.Pi / 2);         // xoay (radian)
a.Lerp(b, 0.5f);                 // điểm giữa
a.LimitLength(100);              // giới hạn độ dài
a.Slide(normal);                 // trượt dọc bề mặt
```

Vì `Vector2` là struct và **bất biến theo ý nghĩa**, mọi phép trên **trả về vector mới**, không sửa vector gốc (`a.Normalized();` một mình không làm gì cả; phải gán `a = a.Normalized();`).

### 13.2 Hệ toạ độ 2D và radian

- Trong Godot 2D, trục **Y hướng xuống**, gốc ở góc trên-trái của scene/viewport. `Vector2.Up` là `(0, -1)`.
- **Góc dùng radian** (`Rotation`, `Angle()`, `Rotated`); độ chỉ là `RotationDegrees`. Đổi bằng `Mathf.DegToRad(d)`/`Mathf.RadToDeg(r)`. `Mathf.Pi` = 180°. Nhầm độ với radian là lỗi rất hay gặp.

### 13.3 Nội suy và làm mượt

- `Mathf.Lerp(a, b, t)` cho giá trị giữa `a` và `b` với `t` từ 0 đến 1.
- `Mathf.MoveToward(a, b, step)` tiến về `b` tối đa `step` mỗi lần, **không vượt quá**; dùng cho gia tốc/ma sát đều.
- **Làm mượt độc lập tốc độ khung hình:** `Lerp(current, target, 0.1f)` mỗi frame cho kết quả khác nhau ở 30 và 144 FPS. Dạng đúng: `float k = 1f - Mathf.Exp(-speed * dt); current = Mathf.Lerp(current, target, k);`.
- `Mathf.Remap(x, inMin, inMax, outMin, outMax)` ánh xạ khoảng này sang khoảng khác; `Mathf.Wrap` quay vòng giá trị (góc, chỉ số).

### 13.4 delta, \_Process và \_PhysicsProcess

- `_Process(delta)` chạy **mỗi frame vẽ**; `delta` thay đổi theo FPS. `_PhysicsProcess(delta)` chạy ở **nhịp cố định** (mặc định 60 lần/giây), `delta` gần như không đổi (\~0,0167).
- Nhân với `delta` để chuyển "mỗi bước" thành "mỗi giây" và làm game **không phụ thuộc FPS**: `Position += Vector2.Right * Speed * dt;` với `Speed` đơn vị px/giây.
- **Bẫy kinh điển:** `MoveAndSlide()` **tự dùng `delta` bên trong** và `Velocity` có đơn vị **px/giây**, nên **không nhân `Velocity` với `delta`** trước khi gọi nó. Ngược lại khi tự cộng gia tốc (`v.Y += gravity * dt`) hoặc tự dịch `Position` thì phải nhân `delta`.
- Logic **di chuyển/va chạm** đặt trong `_PhysicsProcess`; logic hiển thị/UI/hiệu ứng đặt trong `_Process`.
- Đo thời gian: `Time.GetTicksMsec()`, `Time.GetTicksUsec()`; điều khiển tốc độ game: `Engine.TimeScale` (ảnh hưởng `delta` và timer game).

### 13.5 Số ngẫu nhiên

```csharp
float f = GD.Randf();                 // 0..1
int dice = GD.RandRange(1, 6);        // số nguyên, gồm cả hai đầu

var rng = new RandomNumberGenerator();
rng.Seed = 12345;                     // cố định seed: kết quả LẶP LẠI được
int x = rng.RandiRange(0, 99);
```

Dùng bộ sinh có **seed** riêng khi cần **lặp lại** kết quả (sinh bản đồ theo seed, replay, kiểm thử cân bằng). Dùng chung một bộ sinh toàn cục cho mọi thứ làm kết quả bị phụ thuộc thứ tự gọi.

### 13.6 Số thực và độ chính xác

- `float` chỉ \~7 chữ số có nghĩa. Ở **toạ độ rất xa gốc** (hàng chục nghìn), sai số lớn gây rung hình/lệch va chạm; giữ thế giới gần gốc hoặc dịch gốc (floating origin) khi bản đồ khổng lồ.
- Không so sánh `==` hai số thực; dùng `Mathf.IsEqualApprox`, `Mathf.IsZeroApprox`.
- Cộng dồn nhiều lần làm tích tụ sai số; ưu tiên tính trực tiếp từ giá trị gốc khi có thể.

### 13.7 Lỗi thường gặp

- Nhầm độ và radian.
- Nhân `Velocity` với `delta` trước `MoveAndSlide` (nhân vật đi cực chậm).
- Quên gán kết quả của `Normalized()`/`Rotated()`.
- `Lerp` cố định mỗi frame, hành vi phụ thuộc FPS.
- So sánh `==` hai `float`.
- Đặt logic di chuyển vật lý trong `_Process`.

### 13.8 Tự kiểm tra

1. `Vector2` và `Vector2I` khác nhau thế nào? Khi nào dùng cái nào?
2. Vì sao `a.Normalized();` đứng một mình không có tác dụng?
3. Godot dùng độ hay radian cho góc? Trục Y hướng nào?
4. `_Process` và `_PhysicsProcess` khác nhau ra sao? Logic nào đặt ở đâu?
5. Vì sao không nhân `Velocity` với `delta` trước `MoveAndSlide`?
6. Làm mượt độc lập FPS dùng công thức nào và vì sao `Lerp(…, 0.1f)` cố định thì không?
7. Khi nào cần seed riêng cho bộ sinh ngẫu nhiên?

### 13.9 Bài tập

1. Viết hàm trả góc (độ) từ một node tới một điểm bằng `DirectionTo`/`Angle`.
2. Làm một sprite di chuyển về phía chuột bằng `MoveToward`, rồi bằng làm mượt dạng mũ; so sánh ở FPS khác nhau.
3. Dùng `Dot` để kiểm tra một kẻ địch có đang nhìn về phía người chơi (trường nhìn 90°).
4. Sinh 10 số ngẫu nhiên với seed cố định, chạy hai lần và xác nhận giống hệt nhau.
5. In `delta` trong `_Process` và `_PhysicsProcess` khi đổi giới hạn FPS; quan sát.

## Chương 14. Input, vật lý 2D và hệ toạ độ \[Trung bình\]

**Mục tiêu:** hiểu đường đi của input, cách vật lý 2D phân loại và lọc va chạm, và các hệ toạ độ (cục bộ, toàn cục, màn hình, ô lưới).

### 14.1 Input

Khai báo **action** ở Project Settings > Input Map, rồi hỏi theo tên action (không gắn cứng phím):

```csharp
Input.IsActionPressed("move_right");          // đang giữ
Input.IsActionJustPressed("jump");            // vừa nhấn trong frame này
Input.GetAxis("move_left", "move_right");     // -1..1
Input.GetVector("left", "right", "up", "down"); // vector đã chuẩn hoá (đi chéo không nhanh hơn)
```

**Hai kiểu lấy input:** *hỏi trạng thái* (polling) mỗi frame như trên, thích hợp cho di chuyển liên tục; và *nhận sự kiện* qua `_Input(InputEvent @event)`/`_UnhandledInput`, thích hợp cho hành động rời rạc (mở menu, click).

**Thứ tự lan truyền sự kiện:** `_Input` của mọi node → xử lý **giao diện (Control)** → `_ShortcutInput` → `_UnhandledKeyInput` → `_UnhandledInput`. Nếu UI đã "nuốt" sự kiện thì `_UnhandledInput` không nhận. Vì vậy **input của gameplay nên đặt ở `_UnhandledInput`** để không bị click vào nút UI kích hoạt hành động trong game. Gọi `GetViewport().SetInputAsHandled()` để chặn sự kiện lan tiếp.

### 14.2 Vật lý 2D: các loại body

| Node | Ai điều khiển | Dùng cho |
| --- | --- | --- |
| `StaticBody2D` | không di chuyển (hoặc di chuyển bằng code đơn giản) | tường, sàn |
| `CharacterBody2D` | **bạn** (qua `Velocity` + `MoveAndSlide`) | nhân vật, kẻ địch |
| `RigidBody2D` | **engine vật lý** (lực, va chạm, trọng lực) | thùng, bóng, vật thể |
| `Area2D` | không phản ứng vật lý, chỉ **phát hiện** chồng lấn | nhặt đồ, hitbox, vùng kích hoạt |

`CharacterBody2D`: đặt `Velocity` rồi gọi `MoveAndSlide()`; sau đó `IsOnFloor()`, `IsOnWall()` mới đúng. `MoveAndCollide(motion)` di chuyển một bước và trả thông tin va chạm đầu tiên (hợp với đạn, vật bật nảy).

### 14.3 Layer và Mask

Mỗi đối tượng vật lý có **Layer** ("tôi thuộc nhóm nào") và **Mask** ("tôi để ý nhóm nào"). Quy tắc then chốt: **A phát hiện/va chạm với B khi mask của A chứa layer của B**. Mối quan hệ **không đối xứng**: A thấy B không có nghĩa B thấy A.

```csharp
SetCollisionLayerValue(2, true);     // thuộc layer 2 (ví dụ: Player)
SetCollisionMaskValue(1, true);      // va chạm với layer 1 (ví dụ: World)
```

Đặt tên layer ở Project Settings > Layer Names để dễ quản lý, và **thiết kế layer sớm** (World, Player, Enemy, PlayerAttack, Pickup...). Sai layer/mask là nguyên nhân số một của "xuyên tường" và "không nhặt được đồ".

### 14.4 Area2D và tín hiệu va chạm

`BodyEntered/BodyExited` (khi một **body** vào/ra vùng), `AreaEntered/AreaExited` (khi **area** khác vào/ra). Mỗi tín hiệu phát **một lần** lúc bắt đầu/kết thúc chồng lấn, không phát liên tục; muốn kiểm tra liên tục dùng `GetOverlappingBodies()`/`GetOverlappingAreas()`. Đổi trạng thái vật lý trong callback phải **hoãn** (`SetDeferred`, `CallDeferred`) (chương 12).

### 14.5 Truy vấn không gian

`RayCast2D` (tia, phát hiện chướng ngại/mép), `ShapeCast2D`, và truy vấn thủ công qua `GetWorld2D().DirectSpaceState` (`IntersectRay`, `IntersectShape`). Dùng cho tầm nhìn, đạn tốc độ cao, kiểm tra mặt đất.

### 14.6 Hệ toạ độ

| Hệ | Ý nghĩa | Cách lấy |
| --- | --- | --- |
| **Cục bộ** (local) | so với **cha** | `Position` |
| **Toàn cục** (global) | so với thế giới (gốc của canvas) | `GlobalPosition` |
| **Viewport/màn hình** | pixel trên cửa sổ | `GetViewport().GetMousePosition()` |
| **Chuột trong thế giới** | đã tính camera | `GetGlobalMousePosition()` |
| **Ô lưới tilemap** | toạ độ nguyên của ô | `layer.LocalToMap(layer.ToLocal(worldPos))` |

`ToLocal(globalPoint)` và `ToGlobal(localPoint)` đổi qua lại. Khi một node là **con** của node khác đang xoay/co giãn, `Position` của nó chịu ảnh hưởng đó nhưng `GlobalPosition` thì là vị trí thật trên thế giới. Dùng `GlobalPosition` khi **so sánh khoảng cách** giữa hai node khác cha.

### 14.7 Thứ tự vẽ

`ZIndex` (số lớn vẽ sau, nằm trên), **Y-sort** (vẽ theo toạ độ Y, hợp game top-down), `CanvasLayer` (lớp riêng không bị camera ảnh hưởng, dùng cho HUD). Xem Phụ lục C.3 và F.3 trong tài liệu chính.

### 14.8 Lỗi thường gặp

- Sai layer/mask (xuyên tường, không phát hiện).
- Di chuyển body vật lý bằng cách gán `Position` (dịch chuyển tức thời, bỏ qua va chạm) thay vì `Velocity`/`MoveAndSlide`.
- Vật nhỏ bay nhanh **xuyên tường mỏng** (tunneling): dùng raycast, thu nhỏ bước, hoặc tăng độ dày collider.
- Đặt input gameplay ở `_Input`, bị click UI kích hoạt nhầm.
- Nhầm `Position` với `GlobalPosition` khi tính khoảng cách giữa hai node khác cha.
- Đổi trạng thái physics trong callback mà không hoãn.

### 14.9 Tự kiểm tra

1. Polling và event khác nhau ra sao; mỗi cái hợp việc gì?
2. Vì sao input gameplay nên đặt ở `_UnhandledInput`?
3. Bốn loại body/area khác nhau ở điểm nào? Khi nào dùng từng loại?
4. Phát biểu quy tắc layer/mask. Vì sao nó không đối xứng?
5. `BodyEntered` phát khi nào? Muốn kiểm tra liên tục dùng gì?
6. `Position` và `GlobalPosition` khác nhau thế nào?
7. Tunneling là gì và cách giảm?

### 14.10 Bài tập

1. Tạo ba loại body, cấu hình layer/mask sao cho Player chạm World nhưng xuyên qua Pickup, và Pickup phát hiện Player.
2. Viết node nhặt đồ bằng `Area2D`; thử cố tình đặt sai mask rồi tự tìm lỗi.
3. Bắn `RayCast2D` từ nhân vật về phía chuột và in node bị trúng.
4. Đặt một node con trong cha đang xoay, in `Position` và `GlobalPosition`, giải thích sự khác nhau.
5. Đặt input cho một nút UI và một hành động game, thử với `_Input` và `_UnhandledInput`, quan sát sự khác biệt.

## Chương 15. Resource, PackedScene, tải và lưu dữ liệu \[Trung bình\]

**Mục tiêu:** thiết kế dữ liệu game bằng Resource, hiểu cách tải tài nguyên, lưu/tải game an toàn và tương thích phiên bản.

### 15.1 Resource và dữ liệu tự định nghĩa

Resource là **dữ liệu** lưu được ra file (`.tres` dạng chữ, `.res` dạng nhị phân), chỉnh được trong Inspector. Class C# kế thừa `Resource` với `[Export]` thành "kiểu dữ liệu" cho game:

```csharp
[GlobalClass]
public partial class ItemData : Resource
{
    [Export] public string Id = "";
    [Export] public string DisplayName = "";
    [Export] public Texture2D Icon;
    [Export] public int Price;
}
```

Ưu điểm: **thiết kế dữ liệu trong editor** (không cần sửa code mỗi khi thêm vật phẩm), kiểm tra kiểu, tái sử dụng. Đây là cách **thiết kế hướng dữ liệu (data-driven)** của Godot.

**Nguyên tắc:** Resource chứa dữ liệu **tĩnh/định nghĩa** (giá, sát thương cơ bản). **Trạng thái đang chạy** (máu hiện tại, số lượng đang có) để trong node hoặc class C# thuần, **không** ghi vào Resource dùng chung (xem chương 10.6).

### 15.2 Tải tài nguyên

```csharp
var scene = GD.Load<PackedScene>("res://scenes/Enemy.tscn");   // tải đồng bộ (và lưu cache)
var enemy = scene.Instantiate<Enemy>();                        // dựng cây node từ scene
```

- C# **không có `preload`** (tải lúc biên dịch) như GDScript; dùng `GD.Load<T>` lúc chạy, hoặc **`[Export] PackedScene`** để kéo thả trong Inspector (gọn và an toàn nhất).
- Engine **lưu cache** theo đường dẫn: tải cùng file hai lần trả về **cùng một đối tượng**. Muốn bản riêng dùng `Duplicate()`.
- Tải file lớn **ngay trong lúc chơi** gây khựng; tải trước ở màn loading, hoặc tải **không đồng bộ** bằng `ResourceLoader.LoadThreadedRequest(path)`, kiểm tra tiến độ bằng `LoadThreadedGetStatus`, lấy kết quả bằng `LoadThreadedGet`.

### 15.3 Đường dẫn

| Tiền tố | Ý nghĩa | Ghi chú |
| --- | --- | --- |
| `res://` | thư mục project | **chỉ đọc sau khi export** |
| `user://` | thư mục dữ liệu người dùng, riêng từng game/hệ điều hành | **ghi được**, nơi lưu save, cài đặt |

Không bao giờ ghi file save vào `res://`. Tên file trên Linux/Android **phân biệt hoa thường**, hay gây lỗi chỉ xảy ra sau khi export.

### 15.4 Lưu và tải game

Nguyên tắc: **tách dữ liệu save khỏi node**. Tạo class C# thuần chứa **chỉ dữ liệu** (số, chuỗi, mảng, **id** của vật phẩm), chuyển thành JSON, ghi vào `user://`.

```csharp
public class SaveData
{
    public int Version { get; set; } = 1;          // BẮT BUỘC có, để nâng cấp sau này
    public int Coins { get; set; }
    public float PlayerX { get; set; }
    public float PlayerY { get; set; }
    public List<string> ItemIds { get; set; } = new();   // lưu ID, không lưu đối tượng
}

string json = System.Text.Json.JsonSerializer.Serialize(data);
```

**Quy tắc:**

- **Không lưu tham chiếu Node hoặc Resource**; lưu `Id` rồi tra lại lúc tải (ví dụ từ một `ItemDatabase`).
- Luôn có `Version` và hàm **di trú (migration)** nâng save cũ lên cấu trúc mới.
- **Ghi an toàn:** ghi ra file tạm (`save.tmp`), kiểm tra thành công rồi đổi tên thành `save.json`; giữ một bản sao lưu. Như vậy mất điện giữa chừng không làm hỏng save.
- Xử lý file hỏng/thiếu: `try/catch`, quay về mặc định hoặc bản sao lưu, báo người chơi.
- Với `FileAccess.StoreVar/GetVar` (nhị phân của Godot): chỉ tải dữ liệu bạn tự ghi; **không** tải dữ liệu không tin cậy bằng cách bật cho phép đối tượng.
- Cần chống sửa save thì mã hoá/ký, nhưng đó là việc sau; ưu tiên save đúng và bền trước.

### 15.5 Lỗi thường gặp

- Ghi save vào `res://` (lỗi sau khi export).
- Lưu cả đối tượng Node/Resource vào save thay vì `Id`.
- Không có `Version`, sau đổi cấu trúc thì save cũ hỏng.
- Ghi trực tiếp đè file save (hỏng nếu bị ngắt giữa chừng).
- Sửa Resource dùng chung và làm đổi mọi nơi.
- Đường dẫn sai hoa thường (chạy ở Windows, hỏng ở Linux/Android).

### 15.6 Tự kiểm tra

1. Resource khác Node thế nào về mục đích và vòng đời?
2. Vì sao nên để dữ liệu tĩnh trong Resource và trạng thái động ngoài Resource?
3. Hai lần `GD.Load` cùng đường dẫn trả về gì? Hệ quả?
4. `res://` và `user://` khác nhau thế nào khi export?
5. Vì sao lưu `Id` thay vì đối tượng? Vì sao cần `Version`?
6. Mô tả cách ghi save an toàn.

### 15.7 Bài tập

1. Tạo `Resource` `EnemyData` (tên, máu, tốc độ), tạo ba file `.tres`, và một node đọc chúng qua `[Export]`.
2. Viết `SaveSystem` lưu/tải `SaveData` bằng `System.Text.Json` vào `user://`, kèm xử lý file thiếu.
3. Thêm `Version` và viết hàm di trú từ phiên bản 1 sang 2 (thêm trường mới).
4. Cài đặt ghi an toàn (ghi file tạm, đổi tên, giữ bản sao lưu).

## Chương 16. Đa luồng và hiệu năng \[Nâng cao\]

**Mục tiêu:** hiểu mô hình luồng của Godot, những gì được và không được làm ngoài luồng chính, chi phí interop, và quy trình tối ưu có đo đạc.

### 16.1 Mô hình luồng

Cây node và logic game chạy trên **luồng chính**. Engine có luồng riêng cho render và physics, nhưng **API Godot nói chung chỉ an toàn trên luồng chính** (trừ những chỗ tài liệu nói rõ). Hệ quả:

- Việc **tính toán thuần** (thuật toán, sinh bản đồ, tìm đường tự viết) có thể chạy luồng nền (`Task.Run`, `WorkerThreadPool`) **miễn là không chạm node/engine**; trả kết quả về luồng chính bằng `await`/`CallDeferred`.
- Chia sẻ dữ liệu giữa luồng cần đồng bộ hoá (`lock`, `Interlocked`, `ConcurrentQueue<T>`) hoặc tốt hơn: **truyền dữ liệu bất biến** (hoặc bản sao) để tránh race condition.
- Đa luồng thêm độ phức tạp và lỗi khó tái hiện. **Chỉ dùng khi đo thấy cần.**

### 16.2 Chi phí interop và cách giảm

Mỗi lần C# gọi API engine (đọc `GlobalPosition`, `GetNode`, `Call`...) có chi phí vượt qua ranh giới. Với vài trăm đối tượng thì không đáng kể; với hàng chục nghìn lời gọi mỗi frame thì đáng kể. Cách giảm:

- **Cache** kết quả (tham chiếu node, `StringName`) thay vì lấy lại mỗi frame.
- **Đọc một lần, tính nhiều**: lấy `GlobalPosition` vào biến cục bộ, không đọc lại trong vòng lặp.
- Giữ **logic nặng trên kiểu C# thuần** (struct, mảng) và chỉ đẩy kết quả vào node khi cần hiển thị.
- Tránh đường động chậm (`Call/Get/Set` bằng chuỗi) trong vòng lặp nóng; gọi trực tiếp có kiểu.

### 16.3 Vệ sinh vòng lặp

- Với **rất nhiều thực thể**, đừng để mỗi cái tự có `_Process`; dùng một **manager** gọi `Tick` cho tất cả (Phụ lục F.9 trong tài liệu chính).
- `SetProcess(false)` cho node không hoạt động; `VisibleOnScreenNotifier2D` để tắt xử lý/animation ngoài màn hình.
- **Giãn cách cập nhật** việc tốn kém (tìm mục tiêu, tìm đường mỗi 0,2–0,5 giây, lệch pha giữa các thực thể).
- Dùng **phân vùng không gian** (lưới băm) thay vì so mọi cặp O(n²).

### 16.4 Quy trình tối ưu có đo đạc

1. **Đặt mục tiêu:** ví dụ 60 FPS = tối đa \~16,6 ms mỗi frame.
2. **Đo:** Debugger > **Profiler** (thời gian mỗi hàm), **Monitors** (FPS, bộ nhớ, số object, draw call), và đo thủ công bằng `Time.GetTicksUsec()` quanh đoạn nghi ngờ.
3. **Giả thuyết:** điểm nghẽn nằm ở đâu (CPU logic, physics, vẽ, GC, tải tài nguyên)?
4. **Thay đổi một thứ**, rồi **đo lại**. Không đo lại thì không biết có tốt hơn không.
5. Dừng khi đạt mục tiêu; tối ưu thừa chỉ làm code khó đọc.

Lưu ý: **bản Debug của C# chậm hơn bản Release**; đo hiệu năng nghiêm túc trên bản **export Release**, không phải khi chạy trong editor.

### 16.5 Các nguồn gây giật hình thường gặp

| Triệu chứng | Nghi ngờ |
| --- | --- |
| Khựng ngắn định kỳ | GC (quá nhiều cấp phát trong `_Process`), tải tài nguyên đồng bộ |
| Chậm đều khi nhiều thực thể | logic mỗi frame O(n²), quá nhiều node, quá nhiều interop |
| Chậm khi nhiều va chạm | quá nhiều body vật lý hoặc shape phức tạp |
| Chậm khi đồ hoạ phức tạp | quá nhiều draw call, texture lớn, shader nặng, ánh sáng 2D nhiều |
| Giật khi spawn/xoá liên tục | thiếu object pooling |

### 16.6 Lỗi thường gặp

- Tối ưu theo cảm tính mà không đo.
- Đo hiệu năng bằng bản Debug trong editor.
- Chạm node trong luồng nền.
- Dùng đa luồng cho việc nhỏ (chi phí lớn hơn lợi ích).
- Gọi `Call/Get` bằng chuỗi trong vòng lặp nóng.
- Mỗi thực thể một `_Process` khi có hàng nghìn thực thể.

### 16.7 Tự kiểm tra

1. Luồng nào được phép gọi API Godot? Việc gì có thể đưa sang luồng nền?
2. Interop là gì và cách giảm chi phí của nó?
3. Vì sao dùng manager gọi `Tick` thay vì mỗi thực thể một `_Process` khi có rất nhiều thực thể?
4. Nêu 5 bước tối ưu có đo đạc.
5. Vì sao không đo hiệu năng trong editor ở bản Debug?
6. Triệu chứng "khựng ngắn định kỳ" thường do đâu?

### 16.8 Bài tập

1. Tạo 2000 node có `_Process` rồi chuyển sang một manager; đo thời gian trong Profiler trước và sau.
2. Viết đoạn đọc `GlobalPosition` mỗi vòng lặp 100 000 lần và bản cache một lần; đo bằng `Time.GetTicksUsec()`.
3. Dùng `Task.Run` tính nặng (ví dụ tìm số nguyên tố) và hiển thị kết quả khi xong, game vẫn mượt.
4. Dùng `VisibleOnScreenNotifier2D` để tắt `_Process` của các node ngoài màn hình và đo sự khác biệt.

---

# PHẦN 3. NÂNG CAO VÀ KIẾN TRÚC

## Chương 17. Các mẫu thiết kế (pattern) trong Godot \[Nâng cao\]

**Mục tiêu:** biết các mẫu thiết kế hay dùng, vì sao chúng tồn tại, và chọn đúng mẫu cho vấn đề thay vì áp dụng máy móc.

### 17.1 Năm nguyên tắc nền (SOLID, rút gọn theo Godot)

| Nguyên tắc | Ý nghĩa gọn | Trong Godot |
| --- | --- | --- |
| **S**ingle Responsibility | một class một lý do để đổi | node `Health` chỉ lo máu; `Mover` chỉ lo di chuyển |
| **O**pen/Closed | mở rộng được mà không sửa code cũ | thêm loại quân bằng Resource/class mới, không sửa `if` khắp nơi |
| **L**iskov Substitution | lớp con thay được lớp cha mà không hỏng | mọi `Unit` đều dùng được qua `List<Unit>` |
| **I**nterface Segregation | interface nhỏ, đúng việc | `IDamageable`, `IInteractable` riêng, không một `IEverything` |
| **D**ependency Inversion | phụ thuộc vào trừu tượng | nhận `IDamageable` thay vì `Enemy` cụ thể |

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

| Cách | Ví dụ | Ưu | Nhược |
| --- | --- | --- | --- |
| **Singleton/autoload** | `GameManager.Instance` | tiện, ngắn gọn | phụ thuộc ngầm, khó test, vòng đời phức tạp (chương 4.3) |
| **Service locator** | `Services.Get<IAudio>()` | tách khỏi class cụ thể | vẫn phụ thuộc ngầm |
| **Truyền phụ thuộc (injection)** | `[Export] Inventory inv;` hoặc `Init(inv)` | rõ ràng, dễ test, dễ thay | phải nối dây (làm trong scene hoặc code khởi tạo) |

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

## Chương 18. Cấu trúc mã nguồn, kiểm thử và công cụ \[Nâng cao\]

**Mục tiêu:** tổ chức project để mở rộng được, viết được kiểm thử cho logic thuần, dùng công cụ để bắt lỗi sớm.

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

# CỔNG KIỂM TRA: sẵn sàng làm dự án chưa?

Mục đích: kiểm tra bạn **thực sự nắm lý thuyết** trước khi làm dự án. Làm **không nhìn tài liệu**, giới hạn thời gian (khoảng 2 giờ cho phần A, 2–3 giờ cho phần B). Đạt khi **trả lời đúng từ 80% phần A** và **làm được ít nhất 5/6 bài phần B**. Chưa đạt thì ôn các chương tương ứng (bảng ở cuối) rồi làm lại; đó không phải thất bại mà là chỉ dẫn bạn cần học thêm gì.

## Phần A. 30 câu lý thuyết

**Ngôn ngữ C# (12 câu)**

1. `7 / 2`, `7 / 2f`, `7 % 2`, `(int)3.9f` cho gì? Vì sao không dùng `==` với hai `float`?
2. Nêu ba cách phòng `NullReferenceException` và ý nghĩa của `?.`, `??`, `??=`.
3. `ref`, `out`, `in` khác nhau thế nào? Vì sao `Velocity.X = 5` là lỗi mà `Velocity = new Vector2(5, Velocity.Y)` thì được?
4. Class và struct khác nhau ra sao khi gán `b = a`? Khi nào dùng `record`?
5. `virtual`, `override`, `abstract`, `sealed` mỗi cái nghĩa gì? Khi nào dùng interface thay abstract class?
6. Vì sao biến `static` giữ Node nguy hiểm trong Godot?
7. `List`, `Dictionary`, `HashSet`, `Queue`, `Stack`: mỗi cái hợp việc gì và chi phí tìm kiếm ra sao?
8. Hai bẫy của LINQ? Khi nào tránh dùng?
9. Closure là gì? Vì sao một lambda trong vòng `for` có thể in cùng một giá trị?
10. Vì sao phải huỷ đăng ký `event`? Làm ở đâu trong Godot?
11. `async void` và `async Task` khác nhau thế nào? Vì sao dùng timer của Godot thay `Task.Delay`?
12. Boxing là gì? Kể 4 nguồn sinh rác (GC) trong `_Process`.

**Godot (12 câu)**

13. Vẽ cây kế thừa `GodotObject → Node → Node2D → CharacterBody2D` và `GodotObject → RefCounted → Resource`.
14. Thứ tự gọi `_EnterTree`, `_Ready`, `_ExitTree` giữa cha và con?
15. "Gọi xuống, báo lên" nghĩa là gì và vì sao tốt?
16. `QueueFree` khác `Free`? Orphan node là gì?
17. Mô hình "hai nửa" của đối tượng Godot; GC thu hồi nửa nào?
18. Vì sao class script phải `partial`? Kể các ràng buộc của class script.
19. Kiểu nào `[Export]` được và không được? Muốn export dữ liệu phức tạp làm sao?
20. Signal Godot khác `event` C# ở điểm nào? Khi nào chọn cái nào?
21. `EmitSignal` đồng bộ là gì? Khi nào cần `Deferred`?
22. Quy tắc layer/mask của vật lý 2D? Vì sao không đối xứng?
23. `_Process` và `_PhysicsProcess` khác nhau; vì sao không nhân `Velocity` với `delta` trước `MoveAndSlide`?
24. `res://` và `user://` khác nhau thế nào? Vì sao lưu `Id` thay vì đối tượng trong save?

**Nâng cao (6 câu)**

25. Nêu 5 bước tối ưu có đo đạc, và vì sao không đo bằng bản Debug trong editor.
26. Việc gì được và không được làm ngoài luồng chính?
27. Chi phí interop là gì, cách giảm?
28. State machine dùng khi nào? Vì sao mọi chuyển trạng thái nên đi qua một hàm?
29. Singleton, service locator, truyền phụ thuộc: ưu nhược mỗi cách?
30. Vì sao tách logic thuần khỏi node, và làm sao để hàm có ngẫu nhiên vẫn test được?

### Đáp án gợi ý (tự đối chiếu sau khi làm)

- **1:** 3, 3.5, 1, 3; số thực lưu gần đúng nên dùng `Mathf.IsEqualApprox`.
- **2:** kiểm tra `if`, `?.`, khởi tạo mặc định/NRT; `?.` gọi nếu không null, `??` giá trị dự phòng, `??=` gán khi đang null.
- **3:** `ref` đọc/ghi, `out` bắt buộc gán trong hàm, `in` chỉ đọc; `Vector2` là struct nên `Velocity.X` là bản sao.
- **4:** class gán là chia sẻ con trỏ, struct gán là copy; record cho dữ liệu thuần bất biến, so sánh theo nội dung.
- **5:** virtual cho phép ghi đè; override ghi đè; abstract bắt buộc cài; sealed cấm kế thừa; interface khi các lớp không liên quan chung khả năng.
- **6:** static sống qua đổi scene, giữ tham chiếu node đã xoá.
- **7:** List thứ tự, Dictionary tra theo khoá, HashSet thành viên, Queue FIFO, Stack LIFO; tìm trong List O(n), còn Dictionary/HashSet gần O(1).
- **8:** thực thi trì hoãn và cấp phát; tránh trong `_Process`.
- **9:** lambda bắt biến chứ không bắt giá trị; vòng `for` dùng chung một `i`.
- **10:** publisher giữ tham chiếu subscriber; huỷ ở `_ExitTree`.
- **11:** async void không chờ/bắt lỗi được; `Task.Delay` không theo thời gian game/pause.
- **12:** boxing là đóng struct vào object; `new`, nối chuỗi, closure, LINQ, boxing, `params`.
- **13–14:** `_EnterTree` cha trước, `_Ready` con trước cha, `_ExitTree` con trước cha.
- **15:** cha gọi hàm con, con phát signal; con không biết cha.
- **16:** `QueueFree` xoá cuối frame, `Free` tức thì; orphan là node không vào cây và không xoá.
- **17:** native + vỏ C#; GC thu hồi vỏ C#.
- **18:** source generator ghép hai nửa; cùng tên file, không lồng, không generic.
- **19:** export được số/string/enum/struct Godot/Node/Resource/mảng của chúng; không `List<T>`; dùng Resource.
- **20:** signal hiện editor, nối GDScript, tự ngắt khi node xoá; event nhanh, kiểu tự do.
- **21:** handler chạy ngay khi phát; Deferred để hoãn đến cuối frame (đổi physics trong callback).
- **22:** A thấy B khi mask của A chứa layer của B.
- **23:** physics nhịp cố định; `MoveAndSlide` tự dùng delta.
- **24:** `user://` ghi được; `res://` chỉ đọc sau export; id ổn định và tương thích phiên bản.
- **25:** mục tiêu, đo, giả thuyết, đổi một thứ, đo lại; bản Debug chậm hơn.
- **26:** tính toán thuần được; không chạm node/API engine.
- **27:** chi phí mỗi lần gọi qua ranh giới; cache, gom, dùng kiểu C# thuần.
- **28:** vài trạng thái rõ ràng; để gắn Enter/Exit và tránh trạng thái lai.
- **29:** tiện nhưng ngầm; tách nhưng vẫn ngầm; rõ ràng, dễ test nhưng phải nối dây.
- **30:** để test không cần engine; tiêm nguồn ngẫu nhiên có seed.

## Phần B. 6 bài thực hành (không phải dự án)

Làm trong project `Lab`, mỗi bài ngắn, có kết quả kiểm chứng được.

1. **Generic và test:** viết `Pool<T>` generic, kèm 3 bài test xUnit (lấy khi rỗng tạo mới, trả về rồi lấy lại, reset đúng).
2. **Event và vòng đời:** viết `Health` (event C#) và một node nghe đăng ký trong `_Ready`, huỷ trong `_ExitTree`; chứng minh sau khi `QueueFree` node nghe thì không còn lỗi khi `Health` phát event.
3. **Resource và tải:** tạo `Resource` `EnemyData`, ba file `.tres`, một node đọc chúng qua `[Export]`; thêm lưu/tải JSON bằng `System.Text.Json` kèm `Version`.
4. **Signal và await:** `Health` phát `Died`, một hàm `async` chờ `Died` rồi chạy hiệu ứng bằng `Tween` và xoá node; xử lý trường hợp node bị xoá sớm.
5. **Refactor:** lấy script dùng chuỗi `GetNode("../../X")` và logic lẫn lộn, chuyển sang `[Export]`, tách logic tính toán thành class thuần kèm test.
6. **Tìm lỗi:** với mỗi đoạn dưới đây, chỉ ra lỗi và sửa.

```csharp
// (a)
public override void _Process(double delta) { Velocity.X = 100; MoveAndSlide(); }

// (b)
foreach (var e in _enemies) if (e.Hp <= 0) _enemies.Remove(e);

// (c)
float ratio = hp / maxHp;            // hp, maxHp là int

// (d)
public static Player Instance;       // gán trong _Ready; game tải lại scene

// (e)
public async void Flash() { await Task.Delay(500); _sprite.Visible = true; }

// (f)
if (position.X == 100.0f) Stop();
```

Đáp án gợi ý: (a) sửa struct không được và logic di chuyển nên ở `_PhysicsProcess`; (b) sửa collection khi đang `foreach`; (c) chia số nguyên; (d) tham chiếu static cũ sau khi tải lại scene, đặt lại trong `_EnterTree`/`null` trong `_ExitTree`; (e) `Task.Delay` không theo thời gian game và node có thể đã bị xoá; (f) so sánh `==` với `float`.

## Bảng ôn tập nếu chưa đạt

| Câu sai thuộc | Ôn chương |
| --- | --- |
| 1, 12 | 1, 9 |
| 2, 3 | 1, 3, 6 |
| 4, 5, 6 | 4 |
| 7, 8 | 5 |
| 9, 10 | 7 |
| 11 | 8 |
| 13–16 | 10 |
| 17–19 | 11 |
| 20, 21 | 12 |
| 22, 23 | 13, 14 |
| 24 | 15 |
| 25–27 | 9, 16 |
| 28–30 | 17, 18 |

## Sau khi qua cổng

Bạn đã có nền lý thuyết để làm dự án mà không bị mắc kẹt vì kiến thức nền. Hãy quay về **tài liệu chính** ("Tài liệu học Godot 4 + C# từ cơ bản đến nâng cao") và **Lộ trình học**:

1. **Phụ lục B:** Dự án 1 Platformer, làm từng bước.
2. **Phụ lục C:** Dự án 2 Top-down RPG.
3. **Phụ lục D:** quy trình làm dự án thật, Git, GDD.
4. **Phụ lục G, H:** đọc tutorial GDScript và dịch sang C# khi cần.
5. **Phụ lục F:** game dàn quân của riêng bạn.

Khi gặp khái niệm chưa chắc trong lúc làm dự án, quay lại chương tương ứng trong giáo trình này; đó chính là mục đích của nó.

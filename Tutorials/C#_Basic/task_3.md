# PHẦN 1. NGÔN NGỮ C#

## Chương 3. Hàm (phương thức) \[Cơ bản\]

> [!NOTE]
> **Mục tiêu:** viết hàm đúng chữ ký, hiểu cách truyền tham số (theo giá trị và theo tham chiếu), dùng được overload, tuple, local function.

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

| Từ khóa | Ý nghĩa                                               | Điều kiện                            |
| ------- | ----------------------------------------------------- | ------------------------------------ |
| `ref`   | truyền **tham chiếu**: hàm đọc và ghi biến gốc        | biến phải đã được gán trước          |
| `out`   | hàm **trả thêm** giá trị qua tham số                  | hàm **bắt buộc** gán trước khi thoát |
| `in`    | truyền tham chiếu **chỉ đọc** (tránh copy struct lớn) | hàm không được sửa                   |

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

---

**Tiếp theo:** [Task 4](task_4.md)

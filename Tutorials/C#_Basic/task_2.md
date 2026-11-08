# PHẦN 1. NGÔN NGỮ C#

## Chương 2. Điều khiển luồng \[Cơ bản\]

> [!NOTE]
> **Mục tiêu:** viết được điều kiện và vòng lặp rõ ràng, dùng được pattern matching, biết vì sao không được viết vòng lặp vô hạn trong `_Process`.

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

> [!WARNING]
> Cố gắng làm bài tập và thực hiện thêm các bài khác để nâng cao kỹ năng và nhớ cú pháp!

1. FizzBuzz từ 1 đến 30 bằng `for` và `switch` biểu thức.
2. Cho `List<int>`, xoá tất cả số chẵn theo hai cách: duyệt ngược bằng `for`, và `RemoveAll`.
3. Viết hàm `Classify(object o)` dùng pattern matching trả về chuỗi mô tả cho `int`, `string`, `null`, `Vector2`.
4. Duyệt một mảng 2D 5x5 bằng hai vòng `for` lồng nhau và in ra đường chéo chính.

---

**Tiếp theo:** [Task 3](task_3.md)

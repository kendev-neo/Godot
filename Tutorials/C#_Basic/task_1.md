# PHẦN 1. NGÔN NGỮ C#

## Chương 1. Chương trình C# và kiểu dữ liệu \[Cơ bản\]

**Mục tiêu:** hiểu một file `.cs` được biên dịch và chạy ra sao; chọn đúng kiểu số; hiểu các bẫy số học và chuỗi.

### 1.1 Từ file .cs đến chương trình chạy

C# là ngôn ngữ **biên dịch, có kiểu tĩnh**: trình biên dịch (Roslyn) kiểm tra kiểu và cú pháp **trước khi chạy**, biến mã nguồn thành mã trung gian (IL) trong một file `.dll` (assembly). Khi chạy, runtime .NET biên dịch IL thành mã máy (JIT). Hệ quả với bạn:

- Lỗi sai kiểu, sai tên biến được báo **lúc build**, không phải lúc chơi. Đây là lợi thế lớn so với ngôn ngữ kiểu động.
- Trong Godot, bấm **Build** chính là bước biên dịch này; Godot rồi nạp lại assembly để thấy script và thuộc tính mới.
- Project console dùng cùng quy trình, chỉ khác chỗ điểm vào là hàm `Main` (hay _top-level statements_ trong bản mẫu mới), còn trong Godot điểm vào là engine gọi các hàm như `_Ready`.

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

| Kiểu      | Kích thước  | Phạm vi / độ chính xác           | Dùng khi                                   |
| --------- | ----------- | -------------------------------- | ------------------------------------------ |
| `byte`    | 8 bit       | 0 → 255                          | dữ liệu thô, màu từng kênh                 |
| `short`   | 16 bit      | ±32 767                          | hiếm dùng                                  |
| `int`     | 32 bit      | ± khoảng 2,1 tỷ                  | **số nguyên mặc định** (máu, điểm, chỉ số) |
| `long`    | 64 bit      | ± khoảng 9,2 triệu tỷ tỷ         | id lớn, thời gian dạng tick                |
| `float`   | 32 bit      | \~7 chữ số có nghĩa              | **toạ độ, vận tốc trong Godot**            |
| `double`  | 64 bit      | \~15–16 chữ số                   | `delta`, toán cần độ chính xác cao         |
| `decimal` | 128 bit     | 28–29 chữ số thập phân chính xác | tiền tệ thật (hiếm trong game)             |
| `bool`    | 1 bit logic | `true`/`false`                   | cờ, điều kiện                              |
| `char`    | 16 bit      | một ký tự UTF-16                 | hiếm dùng                                  |
| `string`  | động        | chuỗi ký tự                      | văn bản                                    |

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

> [!TIP]
> Hãy thực hành với nhiều cách thử khác nhau để làm quen và thuộc cú pháp!

---

**Tiếp theo:** [Task 2](task_2.md)

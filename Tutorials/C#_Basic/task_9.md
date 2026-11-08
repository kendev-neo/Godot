# PHẦN 1. NGÔN NGỮ C#

## Chương 9. Bộ nhớ, tham chiếu và GC \[Nâng cao\]

> [!NOTE]
> **Mục tiêu:** hiểu giá trị/tham chiếu ở mức bộ nhớ, biết cái gì sinh "rác", và giảm cấp phát đúng chỗ.

### 9.1 Giá trị và tham chiếu ở mức bộ nhớ

- **Kiểu giá trị** (`int`, `float`, `bool`, `struct`, `enum`): dữ liệu nằm **ngay tại biến** (trên stack khi là biến cục bộ, hoặc nằm bên trong đối tượng chứa nó). Gán là **copy**.
- **Kiểu tham chiếu** (`class`, `string`, mảng, `List<T>`, delegate, `Node`): đối tượng nằm trên **heap**; biến chỉ giữ một **con trỏ** tới nó. Gán chỉ **copy con trỏ**, hai biến cùng trỏ một đối tượng.

Hệ quả: sửa đối tượng qua một biến thì mọi biến khác trỏ cùng đối tượng đều thấy; còn struct thì mỗi biến một bản riêng.

### 9.2 Bộ dọn rác (GC)

Bạn **không giải phóng** bộ nhớ heap thủ công trong C#. GC của .NET định kỳ tìm các đối tượng không còn ai tham chiếu rồi thu hồi. GC chia đối tượng theo **thế hệ** (gen 0 mới và hay chết, gen 1, gen 2 sống lâu); gen 0 được dọn thường xuyên và rẻ, gen 2 đắt hơn. Khi GC chạy, nó có thể **làm game khựng ngắn**. Càng ít cấp phát đối tượng tạm, GC càng ít phải dọn, game càng mượt. Điều này quan trọng với game quân số lớn.

### 9.3 Cái gì sinh ra rác

| Nguồn                      | Ví dụ                                                  |
| -------------------------- | ------------------------------------------------------ |
| `new` một class mỗi frame  | `new List<int>()`, `new Vector2[...]` trong `_Process` |
| Nối/định dạng chuỗi        | `"a" + b`, `$"..."` mỗi frame                          |
| **Boxing**                 | gán struct/số vào `object`, collection không generic   |
| Closure/lambda bắt biến    | tạo lambda mới liên tục                                |
| LINQ                       | `Where/Select/ToList`                                  |
| `params`                   | gọi hàm `params` tạo mảng mới                          |
| Duyệt qua `IEnumerable<T>` | trình duyệt struct bị đóng hộp khi đi qua interface    |

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

**Tiếp theo:** [Task 10](task_10.md)

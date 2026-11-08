# PHẦN 1. NGÔN NGỮ C#

## Chương 6. null, ngoại lệ và giải phóng tài nguyên \[Trung bình\]

> [!NOTE]
> **Mục tiêu:** hiểu vì sao `NullReferenceException` xảy ra và phòng tránh có hệ thống; dùng `try/catch` đúng chỗ; biết `using` và `IDisposable` dùng để làm gì.

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

---

**Tiếp theo:** [Task 7](task_7.md)

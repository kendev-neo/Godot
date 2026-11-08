# PHẦN 1. NGÔN NGỮ C#

## Chương 8. async/await và Task \[Trung bình → Nâng cao\]

> [!NOTE]
> **Mục tiêu:** hiểu vì sao cần bất đồng bộ, dùng `await` trong Godot đúng cách, biết khi nào dùng luồng nền và những điều không được làm.

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

|                      | `async Task`            | `async void`                          |
| -------------------- | ----------------------- | ------------------------------------- |
| Chờ/lấy kết quả được | có                      | không                                 |
| Bắt ngoại lệ được    | có (`try/await`)        | khó (ngoại lệ không về nơi gọi)       |
| Dùng khi             | hầu hết hàm bất đồng bộ | **chỉ** callback sự kiện/hàm vòng đời |

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

---

**Tiếp theo:** [Task 9](task_9.md)

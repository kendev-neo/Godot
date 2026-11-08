# PHẦN 2. C# TRONG GODOT

## Chương 16. Đa luồng và hiệu năng \[Nâng cao\]

> [!NOTE]
> **Mục tiêu:** hiểu mô hình luồng của Godot, những gì được và không được làm ngoài luồng chính, chi phí interop, và quy trình tối ưu có đo đạc.

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

| Triệu chứng                 | Nghi ngờ                                                         |
| --------------------------- | ---------------------------------------------------------------- |
| Khựng ngắn định kỳ          | GC (quá nhiều cấp phát trong `_Process`), tải tài nguyên đồng bộ |
| Chậm đều khi nhiều thực thể | logic mỗi frame O(n²), quá nhiều node, quá nhiều interop         |
| Chậm khi nhiều va chạm      | quá nhiều body vật lý hoặc shape phức tạp                        |
| Chậm khi đồ hoạ phức tạp    | quá nhiều draw call, texture lớn, shader nặng, ánh sáng 2D nhiều |
| Giật khi spawn/xoá liên tục | thiếu object pooling                                             |

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

**Tiếp theo:** [Task 17](task_17.md)

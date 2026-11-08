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
| ------------- | --------- |
| 1, 12         | 1, 9      |
| 2, 3          | 1, 3, 6   |
| 4, 5, 6       | 4         |
| 7, 8          | 5         |
| 9, 10         | 7         |
| 11            | 8         |
| 13–16         | 10        |
| 17–19         | 11        |
| 20, 21        | 12        |
| 22, 23        | 13, 14    |
| 24            | 15        |
| 25–27         | 9, 16     |
| 28–30         | 17, 18    |

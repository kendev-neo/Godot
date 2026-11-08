# PHẦN 2. C# TRONG GODOT

## Chương 13. Toán, thời gian và bước thời gian \[Trung bình\]

> [!NOTE]
> **Mục tiêu:** dùng đúng các kiểu toán của Godot, hiểu radian, `delta`, vòng lặp process/physics, và các bẫy của số thực và số ngẫu nhiên.

### 13.1 Các kiểu toán chính

| Kiểu               | Ý nghĩa                                    | Ghi chú                                         |
| ------------------ | ------------------------------------------ | ----------------------------------------------- |
| `Vector2`          | cặp `float` (x, y): vị trí, hướng, vận tốc | **struct** (copy theo giá trị)                  |
| `Vector2I`         | cặp `int`: toạ độ ô tilemap, chỉ số lưới   | không có phần thập phân                         |
| `Rect2` / `Rect2I` | hình chữ nhật (vị trí + kích thước)        | `HasPoint`, `Intersects`                        |
| `Transform2D`      | biến đổi (xoay, tỉ lệ, dịch)               | đứng sau `Position/Rotation/Scale` của `Node2D` |
| `Color`            | màu RGBA dạng `float` 0..1                 | `Colors.Red`                                    |
| `Mathf`            | hàm toán (`float`)                         | `Lerp`, `Clamp`, `MoveToward`, `Remap`, `Wrap`  |

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

---

**Tiếp theo:** [Task 14](task_14.md)

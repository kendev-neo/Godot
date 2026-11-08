# PHẦN 2. C# TRONG GODOT

## Chương 14. Input, vật lý 2D và hệ toạ độ \[Trung bình\]

> [!NOTE]
> **Mục tiêu:** hiểu đường đi của input, cách vật lý 2D phân loại và lọc va chạm, và các hệ toạ độ (cục bộ, toàn cục, màn hình, ô lưới).

### 14.1 Input

Khai báo **action** ở Project Settings > Input Map, rồi hỏi theo tên action (không gắn cứng phím):

```csharp
Input.IsActionPressed("move_right");          // đang giữ
Input.IsActionJustPressed("jump");            // vừa nhấn trong frame này
Input.GetAxis("move_left", "move_right");     // -1..1
Input.GetVector("left", "right", "up", "down"); // vector đã chuẩn hoá (đi chéo không nhanh hơn)
```

**Hai kiểu lấy input:** _hỏi trạng thái_ (polling) mỗi frame như trên, thích hợp cho di chuyển liên tục; và _nhận sự kiện_ qua `_Input(InputEvent @event)`/`_UnhandledInput`, thích hợp cho hành động rời rạc (mở menu, click).

**Thứ tự lan truyền sự kiện:** `_Input` của mọi node → xử lý **giao diện (Control)** → `_ShortcutInput` → `_UnhandledKeyInput` → `_UnhandledInput`. Nếu UI đã "nuốt" sự kiện thì `_UnhandledInput` không nhận. Vì vậy **input của gameplay nên đặt ở `_UnhandledInput`** để không bị click vào nút UI kích hoạt hành động trong game. Gọi `GetViewport().SetInputAsHandled()` để chặn sự kiện lan tiếp.

### 14.2 Vật lý 2D: các loại body

| Node              | Ai điều khiển                                       | Dùng cho                        |
| ----------------- | --------------------------------------------------- | ------------------------------- |
| `StaticBody2D`    | không di chuyển (hoặc di chuyển bằng code đơn giản) | tường, sàn                      |
| `CharacterBody2D` | **bạn** (qua `Velocity` + `MoveAndSlide`)           | nhân vật, kẻ địch               |
| `RigidBody2D`     | **engine vật lý** (lực, va chạm, trọng lực)         | thùng, bóng, vật thể            |
| `Area2D`          | không phản ứng vật lý, chỉ **phát hiện** chồng lấn  | nhặt đồ, hitbox, vùng kích hoạt |

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

| Hệ                       | Ý nghĩa                          | Cách lấy                                    |
| ------------------------ | -------------------------------- | ------------------------------------------- |
| **Cục bộ** (local)       | so với **cha**                   | `Position`                                  |
| **Toàn cục** (global)    | so với thế giới (gốc của canvas) | `GlobalPosition`                            |
| **Viewport/màn hình**    | pixel trên cửa sổ                | `GetViewport().GetMousePosition()`          |
| **Chuột trong thế giới** | đã tính camera                   | `GetGlobalMousePosition()`                  |
| **Ô lưới tilemap**       | toạ độ nguyên của ô              | `layer.LocalToMap(layer.ToLocal(worldPos))` |

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

---

**Tiếp theo:** [Task 15](task_15.md)

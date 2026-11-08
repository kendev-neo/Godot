# KHÓA HỌC GODOT CƠ BẢN

> [!NOTE]
> Sử dụng Godot 4.x trở lên

### 1. Cần cài

- **Godot 4.x bản .NET** (tải ở godotengine.org, chọn bản có chữ .NET). Bản thường không chạy được C#.
- **.NET SDK 8** (từ dotnet.microsoft.com). Kiểm tra bằng `dotnet --version`.
- **Rider** (hoặc VS Code + C# Dev Kit). Trong Godot: Editor > Editor Settings > Dotnet > Editor > External Editor, chọn Rider.

### 2. Cấu trúc thư mục (mẫu)

```language
res://
  project.godot        # cấu hình project
  MyGame.csproj        # project C# (Godot tự tạo)
  MyGame.sln
  scenes/              # file .tscn
  scripts/             # file .cs
  assets/sprites, audio, fonts
  resources/           # file .tres (dữ liệu)
```

> [!TIP]
> Quy ước quan trọng: tên file .cs phải trùng tên class (Player.cs chứa class Player), và class phải khai báo partial.

### 3. Build

Sau khi sửa code C#, phải Build (Ctrl+B trong editor Godot, hoặc nút búa góc trên phải) thì Godot mới thấy script/thuộc tính [Export] mới. Lỗi hay gặp nhất của người mới: sửa code xong mà Inspector không hiện biến, nguyên nhân là chưa build.

### 4. Khái niệm cơ bản

- Node: đơn vị nhỏ nhất, mỗi node làm một việc (Sprite2D vẽ ảnh, CollisionShape2D giữ hình va chạm, Timer đếm giờ...).
- Scene: một cây node được lưu thành file .tscn. Scene có thể lồng nhau và tái sử dụng (ví dụ Enemy.tscn dùng cho mọi kẻ địch).
- SceneTree: cây toàn bộ node đang chạy. Node cha xử lý trước, thứ tự con theo thứ tự trong cây.
- Resource: dữ liệu dùng chung (texture, âm thanh, hoặc dữ liệu tự tạo). Khác Node ở chỗ không nằm trong cây.

> [!TIP]
> Tư duy Godot: chia game thành nhiều scene nhỏ, mỗi scene tự đủ dùng, rồi ghép lại. Ví dụ: `Player.tscn, Enemy.tscn, Coin.tscn, Level1.tscn, HUD.tscn`.

**Các Node nền tảng cần biết:**

| Node                           | Dùng để                                       |
| ------------------------------ | --------------------------------------------- |
| `Node2D`                       | Node 2D cơ bản, có position/rotation/scale    |
| `Sprite2D`, `AnimatedSprite2D` | Vẽ ảnh, vẽ animation theo frame               |
| `CharacterBody2D`              | Nhân vật điều khiển bằng code (player, enemy) |
| `RigidBody2D`                  | Vật thể do physics điều khiển (thùng, bóng)   |
| `StaticBody2D`                 | Vật cố định (tường, sàn)                      |
| `Area2D`                       | Vùng phát hiện (nhặt đồ, hitbox, trigger)     |
| `CollisionShape2D`             | Hình va chạm, là con của các body/area        |
| `Camera2D`                     | Camera                                        |
| `TileMapLayer`                 | Bản đồ ô lưới                                 |
| `Timer`                        | Đếm giờ                                       |

---

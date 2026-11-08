# Chương 2: Cài đặt, project và các loại file

> [!NOTE]
> **Ý chính:** project Godot là một thư mục, mọi thứ là file văn bản dễ quản lý bằng Git. Chọn đúng bản Godot và renderer ngay từ đầu.

### 1. Chọn bản Godot

- **Godot 4.x**, bản **.NET** nếu dùng C# (bản thường chỉ chạy GDScript). Hai bản khác nhau, **cài đúng bản .NET**.
- API thay đổi giữa các bản 4.x (ví dụ `TileMap` chuyển sang `TileMapLayer` từ 4.3). **Khóa một phiên bản** cho cả dự án, đừng nâng cấp giữa chừng khi chưa cần.
- Khi tra tài liệu, chọn đúng phiên bản ở góc trên docs.godotengine.org.

### 2. Tạo project

Khi tạo, Godot hỏi: tên, thư mục, **Renderer**, và có dùng Git hay không.

| Renderer          | Đặc điểm                                             | Khi nào chọn                                      |
| ----------------- | ---------------------------------------------------- | ------------------------------------------------- |
| **Compatibility** | nhẹ, chạy trên máy cũ/di động/web, tính năng ít nhất | game 2D đơn giản, máy yếu, cần tương thích rộng   |
| **Mobile**        | cân bằng, nhắm GPU di động                           | game nhắm điện thoại đời mới                      |
| **Forward+**      | đầy đủ tính năng, hướng máy mạnh                     | game 3D đẹp, hoặc 2D cần hiệu ứng cao cấp trên PC |

Với game 2D, **Compatibility là lựa chọn an toàn** và nhẹ; bạn có thể đổi sau trong Project Settings nhưng một số hiệu ứng hình ảnh có thể khác giữa các renderer, nên hãy thử sớm nếu dùng ánh sáng/hiệu ứng nhiều.

### 3. Các file và thư mục quan trọng

| File / thư mục      | Là gì                             | Ghi chú                                      |
| ------------------- | --------------------------------- | -------------------------------------------- |
| `project.godot`     | cấu hình dự án (Project Settings) | vƒÉn bản, có thể commit Git                  |
| `*.tscn`            | **scene** dạng văn bản            | bản nhị phân là `.scn`                       |
| `*.tres`            | **resource** dạng văn bản         | bản nhị phân là `.res`                       |
| `*.gd`, `*.cs`      | script GDScript / C#              |                                              |
| `*.csproj`, `*.sln` | project C# (bản .NET)             |                                              |
| `*.import`          | thiết lập nhập (import) của asset | commit Git cùng asset                        |
| `.godot/`           | bộ nhớ đệm do Godot tự tạo        | **không** commit Git (thêm vào `.gitignore`) |

**Đường dẫn:** `res://` là gốc project (đọc được, **chỉ đọc sau khi export**); `user://` là thư mục dữ liệu người dùng, **ghi được** (save game, cài đặt). Viết hoa/thường trong tên file có thể gây lỗi trên Linux/Android dù chạy tốt trên Windows.

### 4. Cấu trúc thư mục khuyến nghị

```
res://
  scenes/        (characters, levels, ui, objects)
  scripts/
  assets/        (sprites, audio, fonts)
  resources/     (.tres dữ liệu)
  autoload/
```

Quy tắc: **một scene một script gốc**, cùng tên (`Player.tscn` + `Player.cs`); gom asset theo loại; tên file không dấu, không khoảng trắng.

### 5. Sai lầm thường gặp

- Cài nhầm bản thường khi muốn dùng C#.
- Commit thư mục `.godot/` lên Git (làm repository phình to, gây xung đột).
- Ghi file lưu game vào `res://`.
- Đổi phiên bản Godot giữa dự án.

### 6. Tự kiểm tra

1. `.tscn` và `.tres` khác nhau thế nào? Viết hai lý do nên chọn dạng văn bản.
2. Renderer nào hợp game 2D nhẹ chạy máy yếu?
3. `res://` và `user://` khác nhau ở điểm nào khi game đã export?
4. Thư mục nào không nên commit Git?

---

**Tiếp theo:** [Task 3](task_3.md)

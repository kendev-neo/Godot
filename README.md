<div align="center">

 #
Godot 4 + C# — Học Lập Trình Game

> [!NOTE]
> **Bilingual content** · Godot 4.x (bản **.NET**) · C# trên .NET 8 · Rider hoặc VS Code

[📦 Cấu trúc thư mục](#-cấu-trúc-thư-mục) · [🧭 Lộ trình 4 tuần](#-lộ-trình-4-tuần) · [📁 Tutorials](#-tutorials) · [📚 Courses](#-courses) · [⚙️ Cài Đặt](#️-cài-đặt) · [🔑 Khái Niệm Cốt Lõi](#-khái-niệm-cốt-lõi) · [📜 Quy ước**](#-quy-ước)

**Repo này chứa một bộ tài liệu học game phát triển bằng Godot 4 và C#:**
- **Tutorials/** — bài học nhỏ, từng kỹ năng rõ ràng, đi kèm Ví dụ code và Thử thách.
- **Courses/** — chương trình dài, từ nền tảng C# đến dự án game đầy đủ (Platformer, Top-down RPG, xuất bản).

> [!TIP]
> Nếu chỉ có thời gian: bắt đầu bằng **Courses/** để có lộ trình chuyên sâu, rồi đi ngược lại **Tutorials/** để lấp chân lỗi hoặc học kỹ một chỗ nào đó.
</div>

---


## 📖 Nội dung nhanh
| Chỉ số | Mục tiêu |
| --- | --- |
| 1 | Cài đặt môi trường Godot 4 + C# (.NET 8) và Rider/VS Code |
| 2 | Thông hiểu **Node**, **Scene**, **SceneTree**, **Resource**, **Signal** |
| 3 | Viết C# trong Godot: từ biến, điều kiện, vòng lặp, class đến `partial` |
| 4 | Xây dựng **dự án nhỏ** tuần 1: Platformer 1 màn |
| 5 | Xây dựng **Top-down RPG scale** tuần 3 |
| 6 | Hiểu **Git**, **cấu trúc thư mục**, **export** và **tối ưu** |

## 📁 Cấu trúc thư mục

```text
.
├── Courses/                     # Bộ chương trình học dài
│   ├── Giáo trình lý thuyết Godot 4 cơ bản (học nhanh, hiểu sâu).md
│   ├── Giáo trình lý thuyết C# trong Godot (cơ bản đến nâng cao).md
│   ├── Lộ trình học Godot 4 + C#.md
│   └── Tài liệu học Godot 4 + C# từ cơ bản đến nâng cao (4).md
│
├── Tutorials/                   # Bài học nhỏ, theo kỹ năng
│   ├── C#_Basic/                # C# trong Godot: từ cơ bản đến nâng cao
│   │   ├── task_2.md           # PHẦN 1. NGÔN NGỮ C# (chương 2)
│   │   ├── ... 
│   │   ├── task_0.md           # Tổng quan, cách dùng
│   │   ├── task_1.md           # PHẦN 1. NGÔN NGỮ C# (chương 1)
│   │   └── question.md         # Bài tập tổng kết
│   │
│   └── Godot_Basic/             # Godot cơ bản: node, scene, cấu hình, vật lý
│       ├── task_0.md           # Giáo trình lý thuyết Godot 4 cơ bản
│       ├── task_1.md           # Chương 1: Khái Niệm Cơ Bản & Bản Đồ Tư Duy
│       └── task_2.md           # Chương 2: Cài Đặt, Project & Các Loại File
│
└── README.md
```

> [!NOTE]
> Nếu đang học theo lộ trình, chỉ cần đọc **Courses/** để biết bàn sách. **Tutorials/** dùng để thực hành kỹ năng chi tiết khi gặp vấn đề cụ thể.



## 🧭 Lộ trình 4 tuần

| Tuần | Nội dung chính | Dự án ứng dụng |
| --- | --- | --- |
| 1 | Cài đặt, Node/Scene, C# cơ bản, Input, Physics | **Platformer 1 màn** |
| 2 | Tilemap, Animation, Signal, Resource | Hoàn thiện Platformer (kẻ địch, máu, nhặt đồ) |
| 3 | Autoload, UI, Audio, State Machine, Navigation | **Top-down RPG nhỏ** |
| 4 | Shader, C# nâng cao, Hiệu năng, Export | Hoàn thiện, đóng gói, xuất bản |

Bạn có thể học theo **Courses/** (tự nâng cao đều đặn) hoặc **Tutorials/** (tìm kiếm nhanh một kỹ năng).

## 🛠️ Cài Đặt

### Yêu cầu

- **Godot 4.x bản .NET** — bản thường chỉ chạy GDScript, **không dùng C#**.
- **.NET SDK 8** — kiểm tra bằng `dotnet --version`.
- **Rider** (khuyến nghị) hoặc **VS Code + C# Dev Kit**.

> [!TIP]
> Bật trong Godot: `Editor > Editor Settings > Dotnet > Editor > External Editor` → chọn **Rider**.


### Tạo Project
1. `Project > New Project`, đặt tên `Lab`.
2. Mở `Project > Manage Editor Config` (nếu có) để Git Track tất cả file.
3. Tạo file `script.cs` gắn vào node, **Build lại** (Ctrl+B) để Godot nạp assembly.

### Kiểm Tra Môi Trường

```bash
dotnet --version
godot --version
```

## 🔑 Khái Niệm Cốt Lõi

### Node, Scene, SceneTree, Resource

| Khái niệm | Vị trí nhớ |
| --- | --- |
| **Node** | Đơn vị nhỏ nhất, mỗi node làm một việc (vẽ ảnh, va chạm, đếm giờ). |
| **Scene** | Cây node lưu vào `.tscn`, có thể lồng và tái sử dụng. |
| **SceneTree** | Cây tổng của game đang chạy; node cha xử lý trước, con theo thứ tự trong cây. |
| **Resource** | Dữ liệu dùng chung (texture, âm thanh, dữ liệu tự thiết kế) không nằm trong cây. |

> [!TIP]
> Tư duy Godot: **chia game thành nhiều scene nhỏ**, mỗi scene tự đủ dùng, rồi ghép lại. Ví dụ: `Player.tscn`, `Enemy.tscn`, `Coin.tscn`, `Level1.tscn`, `HUD.tscn`.

### Cơ chế giao tiếp

- **Signal** — node bắn tín hiệu để báo sự kiện; các node khác lắng nghe. Mỗi node biết rất ít về nhau.
- **Autoload** — một nút sống suốt game, không bị mất khi đổi scene.
- **Project Settings** — cấu hình toàn cục như Input Map, Layer Names, Physics.

> [!TIP]

> Đừng gọi hàm giữa các node trực tiếp nếu có thể dùng Signal. Nó giúp bạn thay đổi cấu trúc mà không sợ vỡ code.
---

## 🎮 Ví Dụ: Tạo Cơ Bản

```csharp
using Godot;

public partial class Player : CharacterBody2D
{
    [Export] private float _speed = 200f;
    [Export] private float _jumpForce = 400f;
    private bool _isOnGround;

    public override void _Ready()
    {
        // ...
    }

    public override void _PhysicsProcess(double delta)
    {
        // ...
    }
}
```

## 🗂️ Cấu Trúc Tài Liệu

| --- | --- | --- |
| Thư mục | Loại | Mục đích |
| `Tutorials/C#_Basic/` | Bài học nhỏ | C# trong Godot: cú pháp, pattern matching, collection, enum, class, OOP. |
| `Tutorials/Godot_Basic/` | Bài học nhỏ | Godot 4: node, scene, renderer, file `.tscn`/`.tres`, Project Settings. |

| `Courses/` | Tài liệu lớn | Lộ trình 4 tuần, giáo trình C#, phương pháp dạy học, cheatsheet, FAQ. |

---

## 📚 Tutorials

### Tutorials/C#_Basic

| Bài | Chủ đề |
| --- | --- |
| `task_0.md` | Tổng quan, cách dùng giáo trình |
| `task_1.md` | PHẦN 1: NGÔN NGỮ C# (chương 1) |
| `task_2.md` | PHẦN 1: NGÔN NGỮ C# (chương 2) |
| `task_3.md` | PHẦN 1: NGÔN NGỮ C# (chương 3) |
| `task_4.md` | PHẦN 1: NGÔN NGỮ C# (chương 4) |
| `task_5.md` | PHẦN 1: NGÔN NGỮ C# (chương 5) |
| `task_6.md` | PHẦN 1: NGÔN NGỮ C# (chương 6) |
| `task_7.md` | PHẦN 1: NGÔN NGỮ C# (chương 7) |
| `task_8.md` | PHẦN 1: NGÔN NGỮ C# (chương 8) |
| `task_9.md` | PHẦN 1: NGÔN NGỮ C# (chương 9) |
| `task_10.md` | PHẦN 2: GODOT (chương 10) |
| `task_11.md` | PHẦN 2: GODOT (chương 11) |
| `task_12.md` | PHẦN 2: GODOT (chương 12) |
| `task_13.md` | PHẦN 2: GODOT (chương 13) |
| `task_14.md` | PHẦN 2: GODOT (chương 14) |
| `task_15.md` | PHẦN 2: GODOT (chương 15) |
| `task_16.md` | PHẦN 2: GODOT (chương 16) |
| `task_17.md` | PHẦN 3: NÂNG CAO (chương 17) |
| `task_18.md` | Bài tập, ôn tập, lộ trình nâng cao |
| `question.md` | Bài tập tổng kết |

### Tutorials/Godot_Basic

| Bài | Chủ đề |
| --- | --- |
| `task_0.md` | Giáo trình lý thuyết Godot 4 cơ bản |
| `task_1.md` | Chương 1: Khái Niệm Cơ Bản & Bản Đồ Tư Duy |
| `task_2.md` | Chương 2: Cài Đặt, Project & Các Loại File |


## 📖 Courses

| File | Mô tả |
| --- | --- |
| `Giáo trình lý thuyết Godot 4 cơ bản (học nhanh, hiểu sâu).md` | Học nhanh, hiểu sâu: editor, node, scene, resource, signal. |
| `Giáo trình lý thuyết C# trong Godot (cơ bản đến nâng cao).md` | Từ C# 0, đến class, interface, delegate, async, game architecture. |
| `Tài liệu học Godot 4 + C# từ cơ bản đến nâng cao (4).md` | Bộ tài liệu chính: cài đặt, project, C#, game dev, export. |
| `Lộ trình học Godot 4 + C#.md` | Lộ trình 4 tuần, sổ theo dõi tiến độ, mục "bị kẹt". |



















---

## ⚙️ Quy Ước

- **File C#**: `Tên_file.cs` phải trùng `Tên_class.cs` và class phải khai báo `public partial`.
- **Script đầu tiên**:

```csharp
using Godot;

public partial class Hello : Node
{
    public override void _Ready()
    {
        GD.Print("Node đã vào cây và sẵn sàng");
    }
}
```

> [!TIP]
> Với C# trong Godot 4, bạn **không khởi tạo bằng `new`** để tạo node. Dùng `GD.Instance<T>()`, `new Node()`, hoặc `GD.Load<T>()`. Sau khi sửa code C#, hãy **Build lại** (Ctrl+B) để Godot nạp assembly và thấy biến mới.

> [!TIP]
> Với C# trong Godot 4, bạn **không khởi tạo bằng `new`** để tạo node. Dùng `GD.Instance<T>()`, `new Node()`, hoặc `GD.Load<T>()`. Sau khi sửa code C#, hãy **Build lại** (Ctrl+B) để Godot nạp assembly và thấy biến mới.

---

## 📜 Bản đồ tài liệu

```text
Courses/
├── Giáo trình lý thuyết Godot 4 cơ bản (học nhanh, hiểu sâu).md      # 1468 dòng
├── Giáo trình lý thuyết C# trong Godot (cơ bản đến nâng cao).md      # 2206 dòng
├── Lộ trình học Godot 4 + C#.md                                        # 167 dòng
└── Tài liệu học Godot 4 + C# từ cơ bản đến nâng cao (4).md           # 3919 dòng

Tutorials/
├── C#_Basic/                                                           # 22 bài
│   ├── task_0.md
│   ├── task_1.md
│   ├── task_2.md
│   ├── task_3.md
│   ├── task_4.md
│   ├── task_5.md
│   ├── task_6.md
│   ├── task_7.md
│   ├── task_8.md
│   ├── task_9.md
│   ├── task_10.md
│   ├── task_11.md
│   ├── task_12.md
│   ├── task_13.md
│   ├── task_14.md
│   ├── task_15.md
│   ├── task_16.md
│   ├── task_17.md
│   ├── task_18.md
│   └── question.md
│
└── Godot_Basic/                                                        # 3 bài
    ├── task_0.md
    ├── task_1.md
    └── task_2.md
```



---
## 🙏 Lời cảm ơn

- **Godot engine** (https://godotengine.org)
- **.NET / C#** (https://dotnet.microsoft.com)
- **Rider** (https://jetbrains.com/rider) / **VS Code**
- Tất cả người đã góp ý, test và chỉnh sửa tài liệu.
 
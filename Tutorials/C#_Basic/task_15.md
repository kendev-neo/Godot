# PHẦN 2. C# TRONG GODOT

## Chương 15. Resource, PackedScene, tải và lưu dữ liệu \[Trung bình\]

> [!NOTE]
> **Mục tiêu:** thiết kế dữ liệu game bằng Resource, hiểu cách tải tài nguyên, lưu/tải game an toàn và tương thích phiên bản.

### 15.1 Resource và dữ liệu tự định nghĩa

Resource là **dữ liệu** lưu được ra file (`.tres` dạng chữ, `.res` dạng nhị phân), chỉnh được trong Inspector. Class C# kế thừa `Resource` với `[Export]` thành "kiểu dữ liệu" cho game:

```csharp
[GlobalClass]
public partial class ItemData : Resource
{
    [Export] public string Id = "";
    [Export] public string DisplayName = "";
    [Export] public Texture2D Icon;
    [Export] public int Price;
}
```

Ưu điểm: **thiết kế dữ liệu trong editor** (không cần sửa code mỗi khi thêm vật phẩm), kiểm tra kiểu, tái sử dụng. Đây là cách **thiết kế hướng dữ liệu (data-driven)** của Godot.

**Nguyên tắc:** Resource chứa dữ liệu **tĩnh/định nghĩa** (giá, sát thương cơ bản). **Trạng thái đang chạy** (máu hiện tại, số lượng đang có) để trong node hoặc class C# thuần, **không** ghi vào Resource dùng chung (xem chương 10.6).

### 15.2 Tải tài nguyên

```csharp
var scene = GD.Load<PackedScene>("res://scenes/Enemy.tscn");   // tải đồng bộ (và lưu cache)
var enemy = scene.Instantiate<Enemy>();                        // dựng cây node từ scene
```

- C# **không có `preload`** (tải lúc biên dịch) như GDScript; dùng `GD.Load<T>` lúc chạy, hoặc **`[Export] PackedScene`** để kéo thả trong Inspector (gọn và an toàn nhất).
- Engine **lưu cache** theo đường dẫn: tải cùng file hai lần trả về **cùng một đối tượng**. Muốn bản riêng dùng `Duplicate()`.
- Tải file lớn **ngay trong lúc chơi** gây khựng; tải trước ở màn loading, hoặc tải **không đồng bộ** bằng `ResourceLoader.LoadThreadedRequest(path)`, kiểm tra tiến độ bằng `LoadThreadedGetStatus`, lấy kết quả bằng `LoadThreadedGet`.

### 15.3 Đường dẫn

| Tiền tố   | Ý nghĩa                                                  | Ghi chú                             |
| --------- | -------------------------------------------------------- | ----------------------------------- |
| `res://`  | thư mục project                                          | **chỉ đọc sau khi export**          |
| `user://` | thư mục dữ liệu người dùng, riêng từng game/hệ điều hành | **ghi được**, nơi lưu save, cài đặt |

Không bao giờ ghi file save vào `res://`. Tên file trên Linux/Android **phân biệt hoa thường**, hay gây lỗi chỉ xảy ra sau khi export.

### 15.4 Lưu và tải game

Nguyên tắc: **tách dữ liệu save khỏi node**. Tạo class C# thuần chứa **chỉ dữ liệu** (số, chuỗi, mảng, **id** của vật phẩm), chuyển thành JSON, ghi vào `user://`.

```csharp
public class SaveData
{
    public int Version { get; set; } = 1;          // BẮT BUỘC có, để nâng cấp sau này
    public int Coins { get; set; }
    public float PlayerX { get; set; }
    public float PlayerY { get; set; }
    public List<string> ItemIds { get; set; } = new();   // lưu ID, không lưu đối tượng
}

string json = System.Text.Json.JsonSerializer.Serialize(data);
```

**Quy tắc:**

- **Không lưu tham chiếu Node hoặc Resource**; lưu `Id` rồi tra lại lúc tải (ví dụ từ một `ItemDatabase`).
- Luôn có `Version` và hàm **di trú (migration)** nâng save cũ lên cấu trúc mới.
- **Ghi an toàn:** ghi ra file tạm (`save.tmp`), kiểm tra thành công rồi đổi tên thành `save.json`; giữ một bản sao lưu. Như vậy mất điện giữa chừng không làm hỏng save.
- Xử lý file hỏng/thiếu: `try/catch`, quay về mặc định hoặc bản sao lưu, báo người chơi.
- Với `FileAccess.StoreVar/GetVar` (nhị phân của Godot): chỉ tải dữ liệu bạn tự ghi; **không** tải dữ liệu không tin cậy bằng cách bật cho phép đối tượng.
- Cần chống sửa save thì mã hoá/ký, nhưng đó là việc sau; ưu tiên save đúng và bền trước.

### 15.5 Lỗi thường gặp

- Ghi save vào `res://` (lỗi sau khi export).
- Lưu cả đối tượng Node/Resource vào save thay vì `Id`.
- Không có `Version`, sau đổi cấu trúc thì save cũ hỏng.
- Ghi trực tiếp đè file save (hỏng nếu bị ngắt giữa chừng).
- Sửa Resource dùng chung và làm đổi mọi nơi.
- Đường dẫn sai hoa thường (chạy ở Windows, hỏng ở Linux/Android).

### 15.6 Tự kiểm tra

1. Resource khác Node thế nào về mục đích và vòng đời?
2. Vì sao nên để dữ liệu tĩnh trong Resource và trạng thái động ngoài Resource?
3. Hai lần `GD.Load` cùng đường dẫn trả về gì? Hệ quả?
4. `res://` và `user://` khác nhau thế nào khi export?
5. Vì sao lưu `Id` thay vì đối tượng? Vì sao cần `Version`?
6. Mô tả cách ghi save an toàn.

### 15.7 Bài tập

1. Tạo `Resource` `EnemyData` (tên, máu, tốc độ), tạo ba file `.tres`, và một node đọc chúng qua `[Export]`.
2. Viết `SaveSystem` lưu/tải `SaveData` bằng `System.Text.Json` vào `user://`, kèm xử lý file thiếu.
3. Thêm `Version` và viết hàm di trú từ phiên bản 1 sang 2 (thêm trường mới).
4. Cài đặt ghi an toàn (ghi file tạm, đổi tên, giữ bản sao lưu).

---

**Tiếp theo:** [Task 16](task_16.md)

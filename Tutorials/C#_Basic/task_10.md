# PHẦN 2. C# TRONG GODOT

## Chương 10. Node, Scene, SceneTree và Resource \[Cơ bản → Trung bình\]

> [!NOTE]
> **Mục tiêu:** hiểu mô hình đối tượng của Godot, thứ tự gọi các hàm vòng đời, cách các node giao tiếp, và khác biệt giữa Node và Resource.

### 10.1 Cây kế thừa của Godot

Mọi thứ bạn dùng đều là một **lớp C#** kế thừa nhau (đây chính là OOP ở chương 4 áp dụng vào engine):

```
GodotObject                        (gốc của mọi đối tượng engine)
 ├─ RefCounted                     (tự giải phóng khi không còn ai giữ)
 │   └─ Resource                   (dữ liệu dùng chung: Texture2D, PackedScene, script dữ liệu của bạn)
 └─ Node                           (có mặt trong cây cảnh)
     ├─ CanvasItem
     │   ├─ Node2D                 (có vị trí/xoay/tỉ lệ 2D)  → Sprite2D, CharacterBody2D, Area2D, Camera2D...
     │   └─ Control                (giao diện)                → Button, Label, Container...
     └─ Node3D ...
```

Khi bạn viết `public partial class Player : CharacterBody2D`, bạn **mở rộng** `CharacterBody2D`, thừa hưởng toàn bộ thuộc tính (`Position`, `Velocity`) và hàm (`MoveAndSlide`) của chuỗi kế thừa phía trên. Hiểu được cây này, bạn biết vì sao `Sprite2D` có `Position` còn `Resource` thì không.

### 10.2 Node, cây và scene

- **Node:** đơn vị chức năng nhỏ nhất, mỗi node một nhiệm vụ (vẽ ảnh, giữ hình va chạm, đếm giờ...).
- **Cây (tree):** các node có **một cha** và nhiều con; cha điều khiển con. Vị trí của con tính **tương đối** theo cha (chương 14).
- **Scene:** một cây node lưu thành file `.tscn`. Về mặt code, scene được nạp thành **`PackedScene`** (một Resource); `Instantiate<T>()` dựng ra một **bản sao cây node** sống.
- **`SceneTree`:** cây tổng của game đang chạy, gốc là `Window` (root). Scene hiện tại và các **autoload** đều là con của root. `GetTree()` trả về nó.

Tư duy Godot: chia game thành nhiều scene nhỏ **tự đủ dùng** (Player, Enemy, Coin, HUD), rồi ghép lại bằng cách thêm scene này làm con của scene khác. Đây là **ghép (composition)** ở chương 4.

**Đường dẫn node:** `GetNode<T>("Sprite2D")` (con), `"../Other"` (anh em), `"/root/GameManager"` (tuyệt đối), `"%Name"` (node đặt **Unique Name** trong cùng scene, không phụ thuộc vị trí). Đường dẫn là chuỗi, **gãy khi đổi tên hoặc chuyển node**, nên ưu tiên `[Export]` kéo thả hoặc `%Unique`.

**Parent và Owner:** `Parent` là cha trong cây; `Owner` là node gốc của scene mà node đó **thuộc về khi lưu** (quan trọng với công cụ editor và lưu scene bằng code; trong gameplay thường không cần đụng).

### 10.3 Vòng đời và thứ tự gọi

| Hàm                      | Khi nào                                          | Thứ tự trong cây                       |
| ------------------------ | ------------------------------------------------ | -------------------------------------- |
| `_EnterTree()`           | node vừa vào cây                                 | **cha trước, con sau**                 |
| `_Ready()`               | node và **mọi con** đã sẵn sàng; gọi **một lần** | **con trước, cha sau**                 |
| `_Process(delta)`        | mỗi frame hiển thị                               | theo thứ tự cây (và `ProcessPriority`) |
| `_PhysicsProcess(delta)` | mỗi bước vật lý cố định (mặc định 60/giây)       | như trên                               |
| `_ExitTree()`            | node rời cây                                     | **con trước, cha sau**                 |

Hệ quả quan trọng:

- Trong `_Ready` của cha, **các con đã sẵn sàng** nên `GetNode` đến con là an toàn. Ngược lại, con **không** được giả định cha đã `_Ready` xong.
- Node được thêm bằng `AddChild` sau này vẫn chạy `_EnterTree` rồi `_Ready` của nó ngay khi vào cây.
- `_Ready` chỉ chạy một lần dù node rời rồi vào lại cây.
- `QueueFree()` **không xoá ngay**; node bị xoá ở **cuối frame** (an toàn trong lúc đang duyệt, callback). `Free()` xoá **tức thì**, dễ gây lỗi và chỉ nên dùng khi hiểu rõ.

### 10.4 Các node nói chuyện với nhau thế nào

Quy tắc vàng: **"Gọi xuống, báo lên"** (call down, signal up):

- **Cha gọi hàm của con** trực tiếp (cha biết con là gì).
- **Con không nên biết cha**; thay vào đó con **phát signal**, cha nối vào và phản ứng. Nhờ vậy con dùng lại được ở nhiều nơi.

Các cách khác: node anh em giao tiếp qua cha chung hoặc qua `[Export]`; thông tin toàn cục qua **autoload**; nhắm theo nhóm qua **group** (`AddToGroup`, `GetTree().GetNodesInGroup`, `CallGroup`). Tránh đường dẫn kiểu `GetNode("../../../Foo")` vì nó gắn chặt cấu trúc cây.

### 10.5 Pause và chế độ xử lý

`GetTree().Paused = true` dừng các node có `ProcessMode` là _Pausable_ (mặc định kế thừa từ cha). Menu pause cần `ProcessMode = When Paused` (hoặc _Always_ cho nhạc) để vẫn nhận input khi game dừng. `SetProcess(false)`/`SetPhysicsProcess(false)` tắt cập nhật của riêng node (tiết kiệm CPU với node không hoạt động).

### 10.6 Node và Resource

|                | Node                           | Resource                                         |
| -------------- | ------------------------------ | ------------------------------------------------ |
| Bản chất       | **hành vi** sống trong cây     | **dữ liệu** lưu được ra file `.tres`/`.res`      |
| Có `_Process`? | có                             | không                                            |
| Giải phóng     | `QueueFree()`                  | đếm tham chiếu (tự động)                         |
| Chia sẻ        | mỗi node là một thể hiện riêng | **một Resource thường được nhiều nơi cùng dùng** |
| Ví dụ          | `Player`, `Enemy`, `Label`     | `ItemData`, `Texture2D`, `PackedScene`, theme    |

**Bẫy chia sẻ Resource:** nếu hai node cùng `[Export] ItemData`, họ trỏ **cùng một đối tượng**; sửa trường trên một bên thì bên kia cũng đổi. Dùng Resource cho **dữ liệu tĩnh** (chỉ số cơ bản), còn **trạng thái động riêng** (máu hiện tại) đặt trong node hoặc class C# thuần. Cần bản riêng thì `Duplicate()` (hoặc bật _Local to Scene_).

### 10.7 Lỗi thường gặp

- Gọi `GetNode` trước `_Ready` hoặc sai đường dẫn (`null`/lỗi).
- Con giả định cha đã sẵn sàng.
- Xoá node bằng `Free()` giữa callback va chạm.
- Chia sẻ một Resource và vô tình sửa trạng thái chung.
- Chuỗi đường dẫn dài gãy khi đổi tên node.
- Quên rằng `QueueFree` xoá muộn, node vẫn "sống" đến hết frame.

### 10.8 Tự kiểm tra

1. Vẽ cây kế thừa từ `GodotObject` đến `CharacterBody2D` và đến `Resource`.
2. `_EnterTree`, `_Ready`, `_ExitTree` chạy theo thứ tự cha/con thế nào?
3. Vì sao `GetNode` đến con trong `_Ready` của cha là an toàn?
4. "Gọi xuống, báo lên" nghĩa là gì và vì sao tốt?
5. `QueueFree` khác `Free` ở đâu?
6. Khi nào dùng Resource, khi nào dùng Node? Bẫy chia sẻ Resource là gì?
7. Vì sao ưu tiên `[Export]` hơn đường dẫn chuỗi?

### 10.9 Bài tập

1. Tạo cây node nhỏ (cha và hai con), in thứ tự `_EnterTree`/`_Ready`/`_ExitTree` của cả ba; kiểm chứng bảng 10.3.
2. Viết node con phát signal `Clicked` và node cha nối vào; không để con tham chiếu tới cha.
3. Tạo `Resource` `Stats` chia sẻ cho hai node, sửa trên một bên, quan sát bên kia; sau đó dùng `Duplicate()` để tách.
4. Dùng `AddToGroup`/`CallGroup` để gọi một hàm trên 5 node cùng lúc.

---

**Tiếp theo:** [Task 11](task_11.md)

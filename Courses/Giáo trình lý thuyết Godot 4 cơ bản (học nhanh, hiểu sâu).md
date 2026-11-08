# Giáo trình lý thuyết Godot 4 cơ bản (học nhanh, hiểu sâu)

Dành cho người mới · Godot 4.x · ví dụ code bằng C# · đi kèm Giáo trình lý thuyết C# trong Godot

## 0. Cách học để hiểu nhanh nhất

**Giáo trình này dạy engine Godot** (editor, node, scene, resource, signal, vật lý, animation, UI, tilemap, âm thanh, debug, export), không phải ngôn ngữ lập trình. Phần C# nằm ở *Giáo trình lý thuyết C# trong Godot*; học song song hai giáo trình rất tốt. Đọc xong giáo trình này bạn sẽ hiểu **Godot hoạt động như thế nào** và **muốn làm việc X thì dùng công cụ nào**.

**Thời gian gợi ý:** 7–10 ngày, mỗi ngày 1–2 giờ (1–2 chương mỗi ngày).

**Mỗi chương theo cùng mẫu:**

1. **Ý chính:** 2–3 dòng, đọc trước để biết mình đang học gì.
2. **Mô hình tư duy:** một hình ảnh/so sánh để nhớ khái niệm.
3. **Chi tiết:** lý thuyết, có bảng so sánh và ví dụ.
4. **Bảng quyết định:** "muốn làm X thì chọn Y".
5. **Sai lầm thường gặp.**
6. **Tự kiểm tra:** trả lời **không nhìn tài liệu**; không trả lời được thì đọc lại mục đó.

**Quy tắc học nhanh:** đọc 10 phút → mở Godot **tự tay làm** 20 phút (project đặt tên `Lab`, mỗi chương một scene thử nghiệm) → đóng tài liệu và giải thích lại bằng lời của mình. Nếu giải thích được cho người khác (hay cho chính mình) là bạn đã hiểu.

## 1. Godot trong 10 phút: bản đồ tư duy

**Ý chính:** Godot xây game bằng **node** ghép thành **scene**, scene lại ghép thành scene lớn hơn. Hiểu bảy ý dưới đây là hiểu 80% Godot.

### 1.1 Bảy ý lớn

| # | Ý | Một câu |
| --- | --- | --- |
| 1 | **Node** | Đơn vị nhỏ nhất, mỗi node làm **một việc** (vẽ ảnh, giữ hình va chạm, phát âm thanh, đếm giờ...) |
| 2 | **Scene** | Một **cây node** lưu thành file `.tscn`; dùng lại được nhiều lần |
| 3 | **SceneTree** | Cây tổng của game đang chạy; node cha chứa node con |
| 4 | **Script** | Gắn vào node để thêm **hành vi** (code C# hoặc GDScript) |
| 5 | **Signal** | Node **phát tín hiệu**, node khác **lắng nghe**; để các phần giao tiếp lỏng lẻo |
| 6 | **Resource** | **Dữ liệu** dùng chung lưu được ra file (texture, âm thanh, dữ liệu bạn tự thiết kế) |
| 7 | **Cấu hình toàn cục** | Project Settings, Input Map, Autoload, Layer Names |

### 1.2 Mô hình tư duy: Lego

- **Node = một mảnh Lego** (mảnh bánh xe, mảnh đèn, mảnh khung).
- **Scene = một mô hình đã lắp** (chiếc xe). Quan trọng: **cả chiếc xe lại dùng như một mảnh Lego lớn** để lắp vào mô hình to hơn (thành phố).
- **Script = bộ điều khiển** gắn vào một mảnh để nó biết cử động.
- **Signal = cái chuông**: mảnh này reo chuông khi có chuyện, ai đã treo tai nghe thì nghe thấy; mảnh reo chuông không cần biết ai nghe.
- **Resource = bản vẽ và nguyên liệu** (màu sơn, bảng thông số) cất trong kho, nhiều mảnh cùng dùng chung.

### 1.3 Một game Godot nhìn từ trên cao

```
Game (SceneTree khi đang chạy)
└─ Root
   ├─ [Autoload] GameManager, Music          ← sống suốt game, không mất khi đổi màn
   └─ Level1  (scene hiện tại)
      ├─ TileMapLayer                         ← bản đồ
      ├─ Player  (instance của Player.tscn)  ← một scene dùng lại
      │   ├─ Sprite2D
      │   ├─ CollisionShape2D
      │   └─ Camera2D
      ├─ Enemy   (instance của Enemy.tscn)   ← nhiều bản sao
      ├─ Enemy
      └─ HUD     (instance của HUD.tscn)
```

### 1.4 Hai câu hỏi bạn luôn tự hỏi khi làm Godot

1. **"Điều này là node, scene hay resource?"** (hành vi sống trong cây → node/scene; dữ liệu lưu dùng chung → resource)
2. **"Ai nên biết ai?"** (cha biết con để gọi xuống; con phát signal để báo lên, không cần biết cha)

### 1.5 Tự kiểm tra

1. Nêu bảy ý lớn của Godot bằng lời của bạn.
2. Vì sao nói "scene dùng như một mảnh Lego lớn"?
3. Signal khác gọi hàm trực tiếp ở điểm nào?
4. Autoload nằm ở đâu trong cây và khác scene hiện tại thế nào?

## 2. Cài đặt, project và các loại file

**Ý chính:** project Godot là một thư mục, mọi thứ là file văn bản dễ quản lý bằng Git. Chọn đúng bản Godot và renderer ngay từ đầu.

### 2.1 Chọn bản Godot

- **Godot 4.x**, bản **.NET** nếu dùng C# (bản thường chỉ chạy GDScript). Hai bản khác nhau, **cài đúng bản .NET**.
- API thay đổi giữa các bản 4.x (ví dụ `TileMap` chuyển sang `TileMapLayer` từ 4.3). **Khóa một phiên bản** cho cả dự án, đừng nâng cấp giữa chừng khi chưa cần.
- Khi tra tài liệu, chọn đúng phiên bản ở góc trên docs.godotengine.org.

### 2.2 Tạo project

Khi tạo, Godot hỏi: tên, thư mục, **Renderer**, và có dùng Git hay không.

| Renderer | Đặc điểm | Khi nào chọn |
| --- | --- | --- |
| **Compatibility** | nhẹ, chạy trên máy cũ/di động/web, tính năng ít nhất | game 2D đơn giản, máy yếu, cần tương thích rộng |
| **Mobile** | cân bằng, nhắm GPU di động | game nhắm điện thoại đời mới |
| **Forward+** | đầy đủ tính năng, hướng máy mạnh | game 3D đẹp, hoặc 2D cần hiệu ứng cao cấp trên PC |

Với game 2D, **Compatibility là lựa chọn an toàn** và nhẹ; bạn có thể đổi sau trong Project Settings nhưng một số hiệu ứng hình ảnh có thể khác giữa các renderer, nên hãy thử sớm nếu dùng ánh sáng/hiệu ứng nhiều.

### 2.3 Các file và thư mục quan trọng

| File / thư mục | Là gì | Ghi chú |
| --- | --- | --- |
| `project.godot` | cấu hình dự án (Project Settings) | vƒÉn bản, có thể commit Git |
| `*.tscn` | **scene** dạng văn bản | bản nhị phân là `.scn` |
| `*.tres` | **resource** dạng văn bản | bản nhị phân là `.res` |
| `*.gd`, `*.cs` | script GDScript / C# |  |
| `*.csproj`, `*.sln` | project C# (bản .NET) |  |
| `*.import` | thiết lập nhập (import) của asset | commit Git cùng asset |
| `.godot/` | bộ nhớ đệm do Godot tự tạo | **không** commit Git (thêm vào `.gitignore`) |

**Đường dẫn:** `res://` là gốc project (đọc được, **chỉ đọc sau khi export**); `user://` là thư mục dữ liệu người dùng, **ghi được** (save game, cài đặt). Viết hoa/thường trong tên file có thể gây lỗi trên Linux/Android dù chạy tốt trên Windows.

### 2.4 Cấu trúc thư mục khuyến nghị

```
res://
  scenes/        (characters, levels, ui, objects)
  scripts/
  assets/        (sprites, audio, fonts)
  resources/     (.tres dữ liệu)
  autoload/
```

Quy tắc: **một scene một script gốc**, cùng tên (`Player.tscn` + `Player.cs`); gom asset theo loại; tên file không dấu, không khoảng trắng.

### 2.5 Sai lầm thường gặp

- Cài nhầm bản thường khi muốn dùng C#.
- Commit thư mục `.godot/` lên Git (làm repository phình to, gây xung đột).
- Ghi file lưu game vào `res://`.
- Đổi phiên bản Godot giữa dự án.

### 2.6 Tự kiểm tra

1. `.tscn` và `.tres` khác nhau thế nào? Viết hai lý do nên chọn dạng văn bản.
2. Renderer nào hợp game 2D nhẹ chạy máy yếu?
3. `res://` và `user://` khác nhau ở điểm nào khi game đã export?
4. Thư mục nào không nên commit Git?

## 3. Giao diện editor

**Ý chính:** editor chia thành vài **dock** (bảng) cố định; biết mỗi bảng dùng để làm gì là đủ dùng trôi chảy.

### 3.1 Các khu vực

| Khu vực | Nằm ở | Dùng để |
| --- | --- | --- |
| **Scene** (dock cây) | trái-trên | xem/sửa **cây node** của scene đang mở |
| **FileSystem** | trái-dưới | duyệt file project, kéo thả asset vào scene |
| **Viewport (2D/3D)** | giữa | nhìn và kéo thả node trực quan |
| **Script** | giữa (tab) | viết code |
| **Inspector** | phải | sửa **thuộc tính** của node/resource đang chọn |
| **Node** (cạnh Inspector) | phải | tab **Signals** (nối signal) và **Groups** |
| **Output / Debugger / Animation** | dưới | log, lỗi, profiler, timeline animation |

### 3.2 Thứ tự thao tác cơ bản

1. **Tạo node gốc** (nút *Other Node* hoặc *2D Scene*) → **lưu scene** (Ctrl+S, đặt tên).
2. **Thêm node con** (chuột phải node > *Add Child Node*, hoặc Ctrl+A).
3. **Chỉnh thuộc tính** trong Inspector (vị trí, texture, độ lớn...).
4. **Gắn script** (chuột phải node > *Attach Script*).
5. **Chạy**: `F5` chạy cả dự án (bắt đầu từ **Main Scene**), `F6` chạy **scene đang mở** (rất tiện để thử từng phần).

### 3.3 Hai kỹ năng debug nên biết ngay

- Khi game đang chạy, chuyển sang **Remote** trong dock Scene để xem và sửa thuộc tính node **đang chạy** (kết quả không lưu lại).
- Menu **Debug**: *Visible Collision Shapes* (hiện hình va chạm), *Visible Navigation*.

### 3.4 Sai lầm thường gặp

- Bấm `F5` mà chưa đặt **Main Scene** (Godot sẽ hỏi); muốn thử scene đang làm thì dùng `F6`.
- Sửa thuộc tính trong tab **Remote** rồi tưởng đã lưu (không lưu).
- Quên lưu scene trước khi chạy (chạy bản cũ).

### 3.5 Tự kiểm tra

1. Inspector, Scene dock, FileSystem mỗi bảng dùng để làm gì?
2. `F5` và `F6` khác nhau thế nào?
3. Tab Remote dùng khi nào và có lưu thay đổi không?
4. Signal nối ở tab nào của editor?

## 4. Node và Scene (phần quan trọng nhất)

**Ý chính:** nắm chắc chương này là nắm chắc Godot. Node là việc nhỏ; scene là cách ghép việc nhỏ thành một đối tượng; cây của scene quyết định ai điều khiển ai.

### 4.1 Mô hình tư duy: scene là "class", instance là "object"

Nếu bạn đã biết lập trình: **scene giống một class**, mỗi lần kéo scene vào scene khác (instance) là **tạo một object** từ class đó. Sửa scene gốc thì mọi bản sao đều thay đổi. Nhờ đó bạn thiết kế `Enemy` một lần, dùng 50 lần.

### 4.2 Ba thứ dễ nhầm

| Thao tác | Nghĩa là | Khi nào dùng |
| --- | --- | --- |
| **Instance scene** (ấn biểu tượng xích hoặc kéo file `.tscn` vào) | thêm **một bản sao sống**, vẫn nối với scene gốc | dùng lại Enemy, Coin, HUD trong level |
| **Inherited scene** (*New Inherited Scene*) | tạo scene **mới kế thừa** scene gốc, thêm/sửa phần riêng | biến thể: `BossEnemy` kế thừa `Enemy` |
| **Duplicate node** (Ctrl+D) | **sao chép rời**, không còn nối với gốc | chỉ để nhân nhanh trong cùng scene |

Mặc định, node con **bên trong** một scene đã instance bị khóa để bạn không sửa nhầm; bật *Editable Children* nếu muốn sửa riêng bản sao đó (cẩn thận: sửa nhiều làm mất lợi ích dùng chung).

### 4.3 Quy tắc cây node

- Mỗi scene có **một node gốc**; mọi node khác là con (trực tiếp hoặc gián tiếp) của nó.
- **Cha điều khiển con:** di chuyển/xoay/ẩn cha thì con **đi theo**; xoá cha thì con **bị xoá**; vị trí con tính **so với cha**.
- **Tên node** trong cùng một cha là **duy nhất** (đường dẫn dựa vào tên).
- **Thứ tự** con trong cây quyết định thứ tự xử lý và (với node 2D) thứ tự vẽ mặc định: con ở dưới cùng của danh sách được vẽ **sau** (nằm trên).
- **Unique Name (`%Tên`)**: đặt tên duy nhất cho node trong scene để truy cập bằng `%Tên` không phụ thuộc vị trí.
- **Group:** gắn nhãn cho node ("enemies") để thao tác hàng loạt.

### 4.4 Ví dụ: thiết kế scene Player

```
Player (CharacterBody2D)        ← node gốc: là thân vật lý, có script điều khiển
 ├─ AnimatedSprite2D          ← hình ảnh đang hoạt động
 ├─ CollisionShape2D          ← hình dạng va chạm (bắt buộc phải có với body)
 ├─ Camera2D                  ← theo nhân vật
 └─ Hurtbox (Area2D)          ← vùng nhận sát thương
     └─ CollisionShape2D
```

Mỗi node con **có một nhiệm vụ**; node gốc được chọn theo **bản chất** của đối tượng (nhân vật điều khiển bằng code → `CharacterBody2D`; vùng phát hiện → `Area2D`; đồ họa thuần → `Node2D`). Chọn node gốc đúng là quyết định quan trọng nhất khi tạo scene.

### 4.5 Nguyên tắc thiết kế scene

1. **Scene nhỏ, tự đủ:** một scene làm một đối tượng và chạy được một mình khi bấm `F6` (để thử riêng).
2. **Ghép thay vì nhồi:** thay vì một node khổng lồ làm mọi thứ, chia thành nhiều node/scene nhỏ (`Health`, `Hitbox`, `Mover`).
3. **Gọi xuống, báo lên:** cha gọi hàm con; con phát signal cho cha. Con **không** nên `GetNode("../..")` leo lên cha.
4. **Không lồng sâu vô ích:** cây dưới \~4–5 tầng là dễ hiểu.

### 4.6 Sai lầm thường gặp

- Tạo một scene khổng lồ (cả game trong một scene) thay vì chia nhỏ.
- Duplicate node (Ctrl+D) rồi sửa mỗi bản, lạc mất lợi ích dùng chung của instance.
- `CollisionShape2D` thiếu hoặc chưa gán **Shape** (cảnh báo vàng, va chạm không hoạt động).
- Con gọi `GetParent()` trực tiếp để điều khiển cha, gắn chặt scene với nơi dùng nó.
- Đổi tên node mà code dùng đường dẫn chuỗi (đường dẫn gãy).

### 4.7 Tự kiểm tra

1. Vì sao nói scene giống class, instance giống object?
2. Instance, Inherited scene và Duplicate khác nhau thế nào? Khi nào dùng mỗi cái?
3. Di chuyển node cha thì node con ra sao? Vị trí con tính so với gì?
4. Vì sao `CharacterBody2D` cần có `CollisionShape2D` làm con?
5. Nguyên tắc "gọi xuống, báo lên" là gì?
6. Thứ tự con trong cây ảnh hưởng gì?

## 5. Các node 2D cốt lõi, toạ độ và thứ tự vẽ

**Ý chính:** gần như mọi thứ nhìn thấy trong game 2D là một node kế thừa `CanvasItem`. Biết transform (vị trí/xoay/tỉ lệ), thứ tự vẽ và vài node nền tảng là đủ dựng được hầu hết cảnh 2D.

### 5.1 Mô hình tư duy

`Node2D` là **một tấm bảng trong suốt có toạ độ**. Đặt thứ gì lên tấm bảng thì thứ đó đi theo khi bạn dời, xoay hoặc phóng to tấm bảng. Cây node là chồng các tấm bảng lồng nhau: tấm con lồng trong tấm cha.

### 5.2 Transform: vị trí, xoay, tỉ lệ

| Thuộc tính | Ý nghĩa | Lưu ý |
| --- | --- | --- |
| `Position` | vị trí **so với cha** | trục **Y hướng xuống**, gốc ở góc trên-trái |
| `GlobalPosition` | vị trí thật trên thế giới | dùng khi so khoảng cách giữa node khác cha |
| `Rotation` | góc xoay | **radian** trong code (Inspector hiện độ) |
| `Scale` | tỉ lệ (âm = lật) | ảnh hưởng cả con; **đừng scale node va chạm** (xem chương 9) |
| `Skew` | nghiêng | ít dùng |

Con **thừa hưởng** transform của cha: xoay cha 90° thì con cũng xoay quanh cha, và `Position` của con vẫn là toạ độ trong hệ của cha. Muốn vị trí thật dùng `GlobalPosition`.

### 5.3 Thuộc tính chung của mọi vật vẽ (CanvasItem)

| Thuộc tính | Tác dụng |
| --- | --- |
| `Visible` | ẩn/hiện (ẩn cha thì ẩn cả con) |
| `Modulate` | nhân màu cho node **và con** (hiệu ứng mờ, nhuộm màu, nháy) |
| `SelfModulate` | nhân màu chỉ cho **chính nó**, không ảnh hưởng con |
| `ZIndex` | số lớn vẽ sau, nằm trên |
| `YSortEnabled` | các con được vẽ theo Y (dưới thấp đè lên trên cao) |
| `TextureFilter` | `Nearest` (giữ nét pixel) hay `Linear` (mịn) |
| `Material` | gắn shader/hiệu ứng |

### 5.4 Các node nền tảng

| Node | Dùng để |
| --- | --- |
| `Node2D` | nhóm/điểm đứng có transform (không vẽ gì) |
| `Sprite2D` | vẽ **một ảnh** |
| `AnimatedSprite2D` | vẽ ảnh **chuyển động theo khung** (`SpriteFrames`) |
| `Camera2D` | camera: theo nhân vật, phóng to, giới hạn bản đồ |
| `CollisionShape2D` | hình dạng va chạm (con của body/area) |
| `Marker2D` | **điểm đánh dấu** (nơi sinh quái, đầu súng) |
| `Line2D`, `Polygon2D` | vẽ đường, đa giác |
| `Timer` | đếm giờ (là `Node`, không có vị trí) |
| `TileMapLayer` | bản đồ ô lưới (chương 11) |

### 5.5 Camera2D

Chỉ **một Camera2D** được bật (`Enabled`) tại một thời điểm trong một viewport. Làm con của nhân vật thì camera đi theo. Các thuộc tính hay dùng: `Zoom`, **Position Smoothing** (mượt), **Limit** Left/Right/Top/Bottom (không ra ngoài bản đồ). Đặt camera ở **scene Player** là cách dễ nhất cho game cuộn màn.

### 5.6 Thứ tự vẽ

Quy tắc, từ ưu tiên thấp đến cao:

1. **Thứ tự trong cây:** node ở **dưới** trong danh sách (hoặc con sau) vẽ **sau**, nằm trên.
2. **`ZIndex`:** số lớn hơn vẽ sau (đè lên), kể cả khác nhánh.
3. **`YSortEnabled` ở node cha:** các con được sắp theo toạ độ **Y**, con có Y lớn hơn (thấp hơn trên màn hình) vẽ sau. Cực hợp game nhìn từ trên xuống.
4. **`CanvasLayer`:** một lớp vẽ riêng, **không bị camera ảnh hưởng**; dùng cho HUD/menu.

Quy tắc Y-sort: **điểm gốc (origin) của sprite nên đặt ở chân** nhân vật/cây; nếu gốc ở giữa người, thứ tự sẽ lệch.

### 5.7 Node 2D và Control: đừng nhầm

- **`Node2D`** (và con) dành cho **thế giới game**, đặt bằng toạ độ và chịu ảnh hưởng của camera.
- **`Control`** (Button, Label, Panel...) dành cho **giao diện**, đặt bằng **anchor/container**, thường nằm dưới `CanvasLayer` để không dính camera.
- Không nhét `Control` vào giữa thế giới game trừ khi có chủ đích (ví dụ nhãn tên trên đầu nhân vật).

### 5.8 Bảng quyết định

| Muốn... | Dùng |
| --- | --- |
| hiện một ảnh tĩnh | `Sprite2D` |
| hiện ảnh hoạt hình theo khung | `AnimatedSprite2D` |
| camera đi theo nhân vật | `Camera2D` làm con nhân vật |
| đánh dấu điểm sinh quái/đầu nòng | `Marker2D` |
| HUD không bị camera kéo | `CanvasLayer` + `Control` |
| vật thấp hơn đè vật cao hơn (top-down) | cha bật `YSortEnabled` |

### 5.9 Sai lầm thường gặp

- Nhầm `Position` với `GlobalPosition` khi tính khoảng cách.
- Đặt gốc sprite ở giữa người nên Y-sort sai.
- Bật hai `Camera2D` cùng lúc.
- Gắn HUD thẳng vào scene thế giới nên nó trôi theo camera.
- Scale node có va chạm (hình va chạm méo, vật lý sai).
- Dùng độ thay vì radian khi gán `Rotation` trong code.

### 5.10 Tự kiểm tra

1. `Position` và `GlobalPosition` khác nhau ra sao? Trục Y hướng nào?
2. `Modulate` và `SelfModulate` khác nhau ở đâu?
3. Kể bốn yếu tố quyết định thứ tự vẽ.
4. Vì sao HUD đặt trong `CanvasLayer`?
5. Vì sao điểm gốc sprite nên ở chân khi dùng Y-sort?
6. Khi nào dùng `Marker2D`?

## 6. Resource và asset

**Ý chính:** **Resource là dữ liệu** (không phải đối tượng sống trong cây). Ảnh, âm thanh, hình va chạm, bảng ô lưới, theme và cả dữ liệu bạn tự thiết kế đều là Resource; nhiều node có thể dùng chung một Resource.

### 6.1 Mô hình tư duy

Resource là **bản vẽ và nguyên liệu trong kho**. Node là **thợ** làm việc trong cây. Thợ lấy bản vẽ ra dùng; nhiều thợ cùng dùng một bản vẽ thì sửa bản vẽ là ảnh hưởng cả đám.

### 6.2 Từ file asset đến Resource: quy trình import

Thả file `.png`, `.wav`, `.ogg`, `.ttf` vào project → Godot **tự nhập (import)** thành Resource nội bộ và tạo file `.import` bên cạnh. Mỗi loại file có **thiết lập nhập** (chọn file, mở dock **Import**, đổi tuỳ chọn, bấm **Reimport**):

| Loại | Thiết lập hay chỉnh |
| --- | --- |
| Ảnh (texture) | preset **2D Pixel** cho pixel art (giữ nét, không làm mịn); nén cho ảnh lớn |
| Âm thanh | bật **Loop** cho nhạc nền (`.ogg`/`.wav`); `.wav` cho tiếng ngắn, `.ogg` cho nhạc |
| Font | bật hỗ trợ ký tự cần thiết (tiếng Việt cần font có đủ dấu) |

Sửa thiết lập import xong **phải Reimport** mới có hiệu lực.

### 6.3 Các Resource bạn sẽ gặp

| Resource | Là gì |
| --- | --- |
| `Texture2D` (và `AtlasTexture`) | ảnh; Atlas là một vùng cắt trong ảnh lớn |
| `SpriteFrames` | các animation (dãy khung hình) của `AnimatedSprite2D` |
| `Shape2D` (`RectangleShape2D`, `CircleShape2D`, `CapsuleShape2D`...) | hình va chạm |
| `TileSet` | bộ ô lưới dùng cho tilemap |
| `AudioStream` | âm thanh |
| `Font`, `Theme` | chữ và giao diện |
| `PackedScene` | **scene đã lưu** (chính là file `.tscn` khi nạp) |
| `Material` / `Shader` | hiệu ứng vẽ |
| **Resource tự định nghĩa** | dữ liệu game của bạn (`ItemData`, `EnemyData`): kế thừa `Resource` |

### 6.4 Nhúng hay tách file

Resource có thể **nhúng** (nằm trong file scene `.tscn`) hoặc **tách** thành file `.tres`/`.res` riêng. Nhúng tiện cho thứ chỉ dùng một chỗ (hình va chạm của một node). **Tách file** khi muốn **dùng chung** hoặc chỉnh một nơi áp dụng mọi nơi (một `SpriteFrames`, một `Theme`). Trong Inspector, mở menu của Resource để **Save As...** (tách) hoặc **Make Unique** (tạo bản sao riêng).

### 6.5 Chia sẻ: lợi và bẫy

- Hai node trỏ cùng một Resource thì **sửa một bên đổi cả hai**. Tốt cho dữ liệu chung (cùng `Theme`), nguy hiểm khi bạn muốn mỗi node một giá trị riêng.
- Ví dụ bẫy kinh điển: 5 kẻ địch dùng chung một `CircleShape2D`, sửa bán kính cho một con là cả 5 đổi. Dùng **Make Unique** hoặc bật **Local to Scene**.
- **Dữ liệu tĩnh** (chỉ số cơ bản) hợp với Resource; **trạng thái đang chạy** (máu hiện tại) để trong node, không ghi vào Resource dùng chung.

### 6.6 Dùng Resource trong code (C#)

```csharp
[Export] public Texture2D Icon;                 // kéo thả trong Inspector (khuyên dùng)
[Export] public PackedScene BulletScene;
var tex = GD.Load<Texture2D>("res://assets/icon.png");   // nạp lúc chạy
```

Nạp cùng đường dẫn hai lần trả về **cùng một đối tượng** (engine lưu cache). Muốn bản riêng thì `Duplicate()`.

### 6.7 Bảng quyết định

| Muốn... | Dùng |
| --- | --- |
| nhiều node cùng một giao diện | một `Theme` tách file |
| mỗi kẻ địch hình va chạm riêng | **Make Unique** hoặc Local to Scene |
| định nghĩa vật phẩm, kẻ địch trong editor | Resource tự định nghĩa (`ItemData`) |
| pixel art không bị mờ | import preset 2D Pixel + `TextureFilter = Nearest` |
| nhạc lặp | import có **Loop** |

### 6.8 Sai lầm thường gặp

- Đổi thiết lập import mà quên **Reimport**.
- Sửa một Resource dùng chung mà tưởng chỉ đổi một node.
- Ghi trạng thái động vào Resource dùng chung.
- Dùng font không có dấu tiếng Việt nên chữ bị lỗi.
- Tải texture khổng lồ không cần thiết gây tốn bộ nhớ.

### 6.9 Tự kiểm tra

1. Resource khác Node ở bản chất nào? Lấy hai ví dụ mỗi loại.
2. File `.import` sinh ra khi nào và vì sao phải Reimport?
3. Khi nào nên tách Resource ra file riêng? Make Unique làm gì?
4. Hai node cùng dùng một Resource, sửa một bên thì sao?
5. `[Export]` một Resource khác `GD.Load` ở điểm nào?

## 7. Script, vòng đời, Autoload và chuyển cảnh

**Ý chính:** script gắn vào một node để cho nó **hành vi**. Godot gọi các hàm đặc biệt của script vào đúng thời điểm (vòng đời). Với thứ cần sống xuyên suốt game, dùng Autoload.

### 7.1 Mô hình tư duy

Script là **bộ não gắn vào mảnh Lego**. Một node có tối đa **một** script; script **mở rộng kiểu node** nó gắn vào (script gắn vào `CharacterBody2D` thừa hưởng mọi thuộc tính và hàm của `CharacterBody2D`).

### 7.2 Vòng đời của một node

```
 Node vào cây ──► _EnterTree()
   │ (con vào cây xong)
   ▼
 _Ready()                 ← chạy MỘT lần, node và mọi con đã sẵn sàng
   │
   ▼   mỗi frame:           _Process(delta)         (logic hiển thị, UI)
       mỗi bước vật lý:     _PhysicsProcess(delta)  (di chuyển, va chạm)
   │
   ▼
 Node rời cây ──► _ExitTree()   ← dọn dẹp, ngắt kết nối
```

- **`_Ready`:** nơi lấy tham chiếu node con và thiết lập ban đầu. Chạy **con trước, cha sau**.
- **`_Process(delta)`:** mỗi frame vẽ; `delta` là giây giữa hai frame, thay đổi theo FPS.
- **`_PhysicsProcess(delta)`:** nhịp **cố định** (mặc định 60 lần/giây). **Mọi logic di chuyển/va chạm đặt ở đây.**
- **`delta`** để làm game **không phụ thuộc FPS**: `Position += hướng * tốc_độ * delta`.

### 7.3 Cách một script "với tới" node khác

| Cách | Ví dụ | Khi nào |
| --- | --- | --- |
| `[Export]` kéo thả | `[Export] Sprite2D _sprite;` | **ưu tiên**: không phụ thuộc đường dẫn |
| `GetNode<T>("Tên")` | `GetNode<Timer>("Timer")` | node con cố định |
| `%UniqueName` | `GetNode<T>("%Hp")` | node đặt Unique Name trong scene |
| Group | `GetTree().GetNodesInGroup("enemies")` | thao tác nhiều node |
| Autoload | `GameManager.Instance` | dịch vụ toàn cục |

### 7.4 Biến hiện trong Inspector

Đánh dấu `[Export]` để biến hiện trong Inspector (số, chuỗi, enum, node, Resource, `PackedScene`...). Nhờ vậy người thiết kế chỉnh được tốc độ, sát thương, scene đạn **không cần sửa code**. Sau khi thêm/sửa `[Export]`, phải **Build** để Inspector cập nhật.

### 7.5 Tạo node lúc đang chạy

```csharp
var bullet = BulletScene.Instantiate<Node2D>();    // dựng một bản sao scene
bullet.GlobalPosition = GlobalPosition;
GetParent().AddChild(bullet);                      // thêm vào cây thì mới hoạt động
// xoá khi xong:
QueueFree();                                       // xoá cuối frame, an toàn
```

Node **chưa** thêm vào cây thì chưa chạy `_Ready`, `_Process`. Quên `AddChild` là lỗi "đã tạo mà không thấy".

### 7.6 Autoload: node sống suốt game

Project Settings > **Globals** (tab Autoload) > thêm script hoặc scene và đặt tên. Godot tạo node đó **ngay dưới root, trước mọi scene**, và **giữ nguyên khi đổi scene**. Dùng cho: trạng thái game (vàng, điểm), âm nhạc, quản lý lưu/tải, **event bus** (chỉ chứa signal toàn cục). **Đừng lạm dụng:** autoload là phụ thuộc ngầm, càng nhiều càng khó hiểu và khó thử riêng.

### 7.7 Đổi cảnh

```csharp
GetTree().ChangeSceneToFile("res://scenes/Level2.tscn");   // sang scene khác
GetTree().ReloadCurrentScene();                            // chơi lại màn hiện tại
GetTree().Paused = true;                                   // tạm dừng (node cần ProcessMode phù hợp)
GetTree().Quit();
```

Đổi cảnh **xoá scene cũ** (node của nó bị giải phóng) nhưng **autoload còn nguyên**. Muốn giữ dữ liệu qua màn thì để trong autoload (hoặc file save), không để trong scene cũ.

### 7.8 Việc gì đặt ở đâu

| Việc | Đặt ở |
| --- | --- |
| di chuyển nhân vật | script của `Player` trong `_PhysicsProcess` |
| hiển thị điểm | script của `HUD`, nghe signal của autoload |
| tổng điểm, vàng, tiến trình | autoload `GameManager` |
| tiếng nhạc nền | autoload `Music` hoặc scene riêng |
| dữ liệu vật phẩm | Resource tự định nghĩa |

### 7.9 Sai lầm thường gặp

- Dùng `GetNode` trước `_Ready` hoặc sai đường dẫn (kết quả `null`).
- Đặt logic di chuyển trong `_Process` thay vì `_PhysicsProcess`.
- Quên `AddChild` sau `Instantiate`.
- Để dữ liệu cần giữ qua màn trong scene (mất khi đổi cảnh).
- Quá nhiều Autoload, mọi thứ phụ thuộc nhau.
- Sửa `[Export]` mà không Build.

### 7.10 Tự kiểm tra

1. Vẽ lại vòng đời của một node. `_Ready` chạy khi nào và mấy lần?
2. `_Process` và `_PhysicsProcess` khác nhau thế nào? Logic nào ở đâu?
3. Vì sao `[Export]` được ưu tiên hơn `GetNode` bằng đường dẫn chuỗi?
4. Autoload là gì? Nó khác scene hiện tại ra sao khi đổi cảnh?
5. Vì sao node mới `Instantiate` mà không hiện/không chạy?
6. Muốn giữ số vàng qua nhiều màn, để ở đâu?

## 8. Signal

**Ý chính:** signal để một node **báo "có chuyện đã xảy ra"** mà không cần biết ai quan tâm. Đây là chìa khoá để các phần của game không phụ thuộc chặt vào nhau.

### 8.1 Mô hình tư duy: cái chuông

Node phát signal giống người **bấm chuông**. Ai đã **treo tai nghe** (kết nối) với chuông đó thì nghe thấy và phản ứng. Người bấm chuông **không biết** và **không cần biết** có bao nhiêu người nghe. Nhờ vậy cùng một nút bấm dùng được ở nhiều màn, vì nút không cần biết "bấm xong thì làm gì".

### 8.2 Các signal có sẵn hay dùng

| Node | Signal | Phát khi |
| --- | --- | --- |
| `Button` | `Pressed` | nút được bấm |
| `Area2D` | `BodyEntered`, `BodyExited` | một body vào/ra vùng |
| `Area2D` | `AreaEntered`, `AreaExited` | một area khác vào/ra |
| `Timer` | `Timeout` | hết giờ |
| `AnimationPlayer` | `AnimationFinished` | animation chạy xong |
| `AnimatedSprite2D` | `AnimationFinished` | animation khung hình chạy xong |
| `Tween` | `Finished` | tween xong |
| `Node` | `TreeExiting` | sắp rời cây |

### 8.3 Hai cách nối

1. **Trong editor:** chọn node phát > tab **Node** (cạnh Inspector) > **Signals** > nhấp đúp signal > chọn node nhận. Tiện cho thao tác đơn giản.
2. **Trong code (khuyên dùng với C#):** `button.Pressed += OnPressed;`. Dễ theo dõi, dễ ngắt, không phụ thuộc thao tác chuột.

```csharp
public override void _Ready()
{
    _button.Pressed += OnPressed;
    _area.BodyEntered += OnBodyEntered;
}

private void OnPressed() { GD.Print("bấm"); }
private void OnBodyEntered(Node2D body) { GD.Print(body.Name); }   // signal truyền kèm tham số
```

### 8.4 Signal của riêng bạn

```csharp
[Signal] public delegate void HealthChangedEventHandler(int current, int max);

EmitSignal(SignalName.HealthChanged, _hp, MaxHp);     // phát
```

Quy tắc C#: tên delegate **phải kết thúc bằng `EventHandler`**. Đặt tên signal ở **thì quá khứ** (`Died`, `HealthChanged`, `CoinCollected`): nó thông báo điều **đã xảy ra**, không ra lệnh.

### 8.5 Gọi xuống, báo lên

```
   Cha (Level)
    │  gọi hàm của con (cha biết con)     ▲  con phát signal (con không biết cha)
    ▼                                      │
   Con (Player, Coin, Button)
```

Cha **gọi trực tiếp** hàm của con; con **phát signal** để báo ngược. Nhờ vậy scene con dùng lại ở bất kỳ đâu.

### 8.6 Chờ một signal

```csharp
await ToSignal(GetTree().CreateTimer(1.0), SceneTreeTimer.SignalName.Timeout);   // chờ 1 giây
await ToSignal(_anim, AnimationPlayer.SignalName.AnimationFinished);              // chờ animation xong
```

Sau `await`, node có thể đã bị xoá; kiểm tra `IsInstanceValid(this)` khi cần.

### 8.7 Bảng quyết định

| Muốn... | Dùng |
| --- | --- |
| nút bấm làm gì đó | nối `Pressed` |
| nhặt đồ khi chạm | `Area2D.BodyEntered` |
| làm việc sau một khoảng thời gian | `Timer.Timeout` hoặc `await` timer |
| nhiều hệ thống cùng phản ứng một sự kiện | signal + (autoload event bus) |
| con báo cho cha | con phát signal |

### 8.8 Sai lầm thường gặp

- Delegate signal không kết thúc bằng `EventHandler`.
- Nối cùng một hàm hai lần (bị gọi hai lần).
- Đổi trạng thái vật lý trong callback va chạm (cần `CallDeferred`/`SetDeferred`).
- Con gọi thẳng `GetParent()` thay vì phát signal.
- Chờ signal không bao giờ phát (hàm treo mãi).
- Quên ngắt kết nối với event/ signal của đối tượng sống lâu hơn.

### 8.9 Tự kiểm tra

1. Giải thích signal bằng ví dụ cái chuông.
2. Có mấy cách nối signal? Ưu nhược mỗi cách?
3. Quy tắc đặt tên delegate signal trong C#? Đặt tên signal theo thì nào?
4. "Gọi xuống, báo lên" nghĩa là gì?
5. `BodyEntered` truyền kèm gì và dùng khi nào?
6. Vì sao phải kiểm tra node còn sống sau `await`?

## 9. Vật lý 2D

**Ý chính:** Godot có bốn loại đối tượng vật lý với vai trò khác nhau. Chọn đúng loại, gắn đúng hình va chạm và cấu hình đúng layer/mask là giải quyết 90% vấn đề va chạm.

### 9.1 Mô hình tư duy

Engine vật lý là **trọng tài**. Bạn nói với trọng tài: "đây là các vật, đây là hình dạng và ai được va chạm với ai", rồi trọng tài lo phần còn lại. Mỗi vật cần **một body/area (người chơi)** và **một hình dạng (CollisionShape2D)**.

### 9.2 Bốn loại đối tượng

| Node | Ai điều khiển | Dùng cho | Ví dụ |
| --- | --- | --- | --- |
| `StaticBody2D` | không di chuyển | địa hình cố định | tường, sàn |
| `CharacterBody2D` | **bạn** (code) | nhân vật điều khiển chính xác | player, kẻ địch |
| `RigidBody2D` | **engine vật lý** | vật chịu lực, lăn, đẩy | thùng, bóng, đá |
| `Area2D` | không phản ứng vật lý, chỉ **phát hiện** | vùng kích hoạt | nhặt đồ, hitbox, bẫy, vùng tầm nhìn |

**Sơ đồ chọn nhanh:**

```
Vật này có cần va chạm vật lý thật không?
 ├─ Không, chỉ cần biết "có thứ vào vùng này"  → Area2D
 └─ Có
     ├─ Đứng yên                                → StaticBody2D
     ├─ Tôi điều khiển bằng code                → CharacterBody2D
     └─ Để vật lý tự xử lý (lực, nảy, lăn)      → RigidBody2D
```

### 9.3 Hình va chạm (CollisionShape2D)

- Mỗi body/area cần ít nhất **một `CollisionShape2D`** làm con, và **phải gán `Shape`** (nếu không có cảnh báo vàng và không va chạm).
- Hình thường dùng: `RectangleShape2D`, `CircleShape2D`, `CapsuleShape2D` (hợp nhân vật), đa giác lồi. Hình đơn giản **chạy nhanh hơn** và ít lỗi hơn hình phức tạp.
- **Không scale node va chạm** (đổi `Scale` làm hình méo và vật lý sai); hãy **đổi kích thước của Shape** trong Inspector.
- Hình va chạm không nhất thiết trùng khít hình ảnh: nhân vật thường dùng hình nhỏ ở **chân** để đi quanh vật cản tự nhiên.

### 9.4 Layer và Mask

Mỗi vật có **Layer** (tôi thuộc nhóm nào) và **Mask** (tôi để ý nhóm nào). **A va chạm/phát hiện B khi mask của A chứa layer của B.** Quan hệ **không đối xứng**.

Ví dụ cấu hình:

| Vật | Layer | Mask |
| --- | --- | --- |
| Tường/sàn (World) | World | (không cần) |
| Player | Player | World |
| Kẻ địch | Enemy | World |
| Đồng xu (Area2D) | Pickup | Player |
| Đòn đánh của Player (Area2D) | PlayerAttack | Enemy |

Đặt tên layer ở Project Settings > **Layer Names > 2D Physics** để khỏi nhầm số. **Thiết kế layer sớm** vì đổi sau rất mệt. Sai layer/mask là nguyên nhân số một của "xuyên tường" và "nhặt không được đồ".

### 9.5 Di chuyển CharacterBody2D

```csharp
public override void _PhysicsProcess(double delta)
{
    Vector2 v = Velocity;
    if (!IsOnFloor()) v.Y += _gravity * (float)delta;        // trọng lực: tự cộng
    v.X = Input.GetAxis("move_left", "move_right") * Speed;  // vận tốc ngang px/giây
    Velocity = v;
    MoveAndSlide();                                          // di chuyển, tự trượt dọc tường/dốc
}
```

`MoveAndSlide()` **tự dùng `delta`** và `Velocity` tính theo **px/giây**, nên **không nhân `Velocity` với `delta`**. Sau khi gọi nó, `IsOnFloor()`, `IsOnWall()`, `IsOnCeiling()` mới đúng.

### 9.6 Area2D và truy vấn

- `Area2D` phát signal `BodyEntered/AreaEntered` **một lần** khi bắt đầu chồng lấn; kiểm tra liên tục dùng `GetOverlappingBodies()`.
- **`RayCast2D`**: bắn một tia để biết có gì phía trước (mép vực, tầm nhìn, mặt đất). Nhớ đặt **Collision Mask** cho nó.
- Muốn một sàn "nhảy xuyên từ dưới lên": bật **One Way Collision** trên `CollisionShape2D`.

### 9.7 Bảng quyết định

| Muốn... | Dùng |
| --- | --- |
| nhân vật platformer/top-down | `CharacterBody2D` + `MoveAndSlide` |
| thùng đẩy được, bóng nảy | `RigidBody2D` |
| nhặt đồ, bẫy, vùng đánh | `Area2D` |
| tường, sàn | `StaticBody2D` hoặc tile có Physics Layer |
| biết có vật cản phía trước | `RayCast2D` |
| bục nhảy xuyên từ dưới | One Way Collision |

### 9.8 Sai lầm thường gặp

- Thiếu `CollisionShape2D` hoặc chưa gán `Shape`.
- Sai layer/mask (xuyên tường, không phát hiện).
- Di chuyển body bằng cách gán `Position` (dịch chuyển tức thời, bỏ qua va chạm).
- Nhân `Velocity` với `delta` trước `MoveAndSlide` (đi cực chậm).
- Scale node va chạm.
- Đặt logic di chuyển ở `_Process`.
- Vật nhỏ bay nhanh xuyên tường mỏng (tunneling): dùng raycast hoặc tường dày hơn.

### 9.9 Tự kiểm tra

1. Bốn loại đối tượng vật lý khác nhau thế nào? Chọn loại nào cho: tường, người chơi, thùng đẩy được, đồng xu?
2. Vì sao body cần `CollisionShape2D` và phải gán `Shape`?
3. Phát biểu quy tắc layer/mask. Vì sao không đối xứng?
4. Vì sao không nhân `Velocity` với `delta` trước `MoveAndSlide`?
5. Vì sao không nên scale `CollisionShape2D`?
6. `BodyEntered` phát khi nào? Muốn kiểm tra liên tục dùng gì?

## 10. Giao diện (UI) với Control

**Ý chính:** UI trong Godot là một cây `Control`. Thay vì đặt từng nút bằng toạ độ, bạn dùng **container** để chúng tự xếp và co giãn theo màn hình. Học container là học UI.

### 10.1 Mô hình tư duy

Hãy nghĩ UI như **các hộp co giãn lồng nhau**: một hộp dọc (VBox) chứa các hàng, mỗi hàng là hộp ngang (HBox) chứa nút và nhãn. Bạn chỉ nói "xếp dọc, cách nhau 8px", không nói "nút này ở toạ độ (120, 340)". Khi đổi độ phân giải, mọi thứ tự co giãn đúng.

### 10.2 Hai cách bố trí

| Cách | Dùng khi |
| --- | --- |
| **Container** (tự xếp) | gần như mọi trường hợp: menu, HUD, hộp thoại, túi đồ |
| **Anchor** (neo vào cạnh/giữa cha) | node đơn lẻ cần bám góc/cạnh/giữa (nút Pause ở góc phải trên) |

Mẹo: chọn node `Control`, dùng menu **Layout** trên thanh công cụ (hoặc Inspector > Layout) với các preset như *Full Rect*, *Center*, *Top Right*.

### 10.3 Các container chính

| Container | Xếp con thế nào |
| --- | --- |
| `VBoxContainer` | thành cột dọc |
| `HBoxContainer` | thành hàng ngang |
| `GridContainer` | lưới nhiều cột (đặt `Columns`) |
| `MarginContainer` | tạo lề quanh nội dung |
| `PanelContainer` | nền (panel) bao quanh nội dung |
| `CenterContainer` | căn giữa |
| `ScrollContainer` | cuộn khi nội dung dài |
| `TabContainer` | các tab |
| `FlowContainer` (H/V) | xuống dòng tự động |

**Kích thước của con trong container:** Inspector > **Layout > Container Sizing**. `Fill` = lấp đầy chiều đó; `Expand` = chiếm phần không gian dư. Muốn một nút giãn hết hàng thì `Horizontal = Fill + Expand`. `Custom Minimum Size` đặt kích thước tối thiểu.

### 10.4 Các control hay dùng

| Control | Dùng để |
| --- | --- |
| `Label` | chữ tĩnh |
| `RichTextLabel` | chữ có định dạng (BBCode), có thể gõ từng chữ |
| `Button`, `TextureButton` | nút bấm (chữ / ảnh) |
| `TextureRect` | ảnh trong UI (icon, avatar) |
| `ProgressBar`, `TextureProgressBar` | thanh máu, tải |
| `LineEdit`, `TextEdit` | ô nhập chữ |
| `CheckBox`, `OptionButton`, `HSlider` | tuỳ chọn, danh sách thả, thanh trượt |
| `ItemList`, `Tree` | danh sách, cây |

### 10.5 Theme

**Theme** là Resource quy định phong cách (font, màu, hình nút) cho cả nhánh UI. Đặt một Theme ở Control gốc (hoặc **Project Settings > GUI > Theme**) thì mọi con dùng chung. Sửa riêng một node bằng **Theme Overrides** trong Inspector. Dùng Theme để UI đồng bộ thay vì chỉnh từng nút.

### 10.6 Chuột, focus, tay cầm

- **Mouse Filter** của mỗi Control: `Stop` (nhận và chặn), `Pass` (nhận và cho đi tiếp), `Ignore` (bỏ qua). Một `ColorRect`/`Panel` phủ toàn màn hình mà để `Stop` sẽ **chặn mọi cú nhấp** bên dưới; đặt `Ignore` cho lớp trang trí.
- **Focus:** điều hướng bằng phím/tay cầm cần `FocusMode` và một nút được `GrabFocus()` lúc mở menu.
- Tín hiệu bấm nút là `Pressed`; thay đổi giá trị là `ValueChanged`...

### 10.7 Cấu trúc HUD điển hình

```
HUD (CanvasLayer)                        ← không bị camera kéo
 └─ MarginContainer (Full Rect)
     └─ VBoxContainer
         ├─ HBoxContainer
         │   ├─ TextureRect (icon tim)
         │   └─ ProgressBar (máu)
         └─ Label (số xu)
```

### 10.8 Bảng quyết định

| Muốn... | Dùng |
| --- | --- |
| menu chính nhiều nút | `VBoxContainer` (trong `CenterContainer`) |
| HUD không dính camera | `CanvasLayer` + container |
| nút bám góc | anchor preset |
| nhiều ô đều nhau (túi đồ) | `GridContainer` |
| phong cách đồng bộ | một `Theme` dùng chung |
| lớp trang trí không chặn chuột | `MouseFilter = Ignore` |

### 10.9 Sai lầm thường gặp

- Đặt từng nút bằng toạ độ (vỡ khi đổi độ phân giải).
- Panel trang trí để `Stop` chặn nhấp chuột.
- Control đặt thẳng trong thế giới game nên bị camera kéo (cần `CanvasLayer`).
- Font thiếu dấu tiếng Việt.
- Chữ tràn khung vì quên bật wrap hoặc quên minimum size.

### 10.10 Tự kiểm tra

1. Vì sao dùng container thay vì đặt toạ độ?
2. Container và anchor khác nhau thế nào? Khi nào dùng cái nào?
3. `Fill` và `Expand` nghĩa là gì?
4. Mouse Filter có ba giá trị nào và ý nghĩa?
5. Vì sao HUD đặt trong `CanvasLayer`?
6. Theme giải quyết việc gì?

## 11. Animation

**Ý chính:** Godot có **bốn công cụ** animation cho bốn nhu cầu khác nhau. Chọn đúng công cụ quan trọng hơn biết hết tính năng.

### 11.1 Bốn công cụ và khi nào dùng

| Công cụ | Mô hình | Dùng khi |
| --- | --- | --- |
| `AnimatedSprite2D` | **lật sổ tay** (flipbook): các khung ảnh nối tiếp | hoạt hình khung hình đơn giản (idle, chạy, nhảy) |
| `AnimationPlayer` | **băng ghi theo thời gian** (timeline) | animate **bất kỳ thuộc tính** (vị trí, màu, bật/tắt hitbox), gọi hàm, phát tiếng |
| `AnimationTree` | **máy trạng thái** của animation | nhiều animation chuyển qua lại, hoà trộn theo hướng/tốc độ |
| `Tween` (code) | **lệnh một dòng** cho hiệu ứng ngắn | mờ dần, trượt, nảy, rung; hiệu ứng tạo bằng code |

### 11.2 AnimatedSprite2D

Tạo `SpriteFrames` trong Inspector, thêm các animation (`idle`, `run`...), kéo các khung từ ảnh vào (hoặc cắt từ sprite sheet), đặt FPS và có lặp hay không. Gọi `Play("run")`; `FlipH` để lật khi quay trái. Đây là cách **nhanh nhất** cho nhân vật pixel art đơn giản.

### 11.3 AnimationPlayer

Một animation là tập **track** (đường thời gian), mỗi track điều khiển một thứ:

| Loại track | Làm gì |
| --- | --- |
| **Property** | thay đổi một thuộc tính theo thời gian (nội suy giữa các **keyframe**) |
| **Call Method** | gọi một hàm của script đúng thời điểm |
| **Audio** | phát tiếng đúng khung |
| **Animation Playback** | điều khiển animation khác |

Ví dụ đòn đánh: khung 0.1s bật hitbox, khung 0.3s tắt hitbox, khung 0.35s gọi `OnAttackEnd`. Các thiết lập: độ dài, **Loop Mode**, **Autoplay**, và track **RESET** (trạng thái gốc để animation không để lại dấu vết). Gọi `Play("attack")`; chờ xong bằng signal `AnimationFinished`.

### 11.4 AnimationTree

Gắn trên một `AnimationPlayer` đã có sẵn các animation. Hai nút chính: **StateMachine** (idle/walk/attack/jump, chuyển theo điều kiện) và **BlendSpace2D/1D** (hoà trộn theo hướng: 4/8 hướng top-down). Điều khiển qua **parameters** và hàm `Travel("run")`. Chỉ cần AnimationTree khi số animation và quan hệ chuyển trạng thái đã đủ phức tạp; với nhân vật đơn giản, `AnimatedSprite2D` + code là đủ.

### 11.5 Tween

```csharp
var t = CreateTween();
t.TweenProperty(_sprite, "modulate:a", 0.0f, 0.4f);      // mờ dần trong 0.4 giây
t.TweenCallback(Callable.From(QueueFree));              // xong thì xoá
```

Có `SetParallel()` (chạy song song), `SetEase()`, `SetTrans()` (kiểu nội suy). Tween **tự giải phóng khi xong**, tạo từ một node nên chết cùng node đó. Hợp cho "juice": nảy khi nhặt xu, rung màn hình, hiện bảng.

### 11.6 Nguyên tắc

- **Logic game không nên phụ thuộc thời lượng animation.** Cần đồng bộ (đánh trúng khung nào) thì dùng **Call Method track** hoặc signal, đừng đoán bằng timer.
- Dùng animation để làm **hiển thị**; trạng thái thật (máu, đang tấn công) nằm trong code.

### 11.7 Bảng quyết định

| Muốn... | Dùng |
| --- | --- |
| nhân vật pixel art idle/chạy | `AnimatedSprite2D` |
| bật hitbox ở khung nhất định | `AnimationPlayer` + track Property/Call Method |
| nhiều trạng thái chuyển qua lại phức tạp | `AnimationTree` |
| mờ dần rồi xoá, nảy khi nhặt xu | `Tween` |
| nhân vật 8 hướng | `AnimationTree` + BlendSpace2D |

### 11.8 Sai lầm thường gặp

- Dùng `AnimationTree` cho thứ đơn giản (rườm rà).
- Quên track `RESET`, animation để lại thuộc tính sai khi dừng.
- Đoán thời điểm bằng timer thay vì dùng track gọi hàm.
- Tween tạo từ node đã bị xoá (không chạy).
- Quên `Play` hoặc tên animation sai (kiểm tra chính tả).

### 11.9 Tự kiểm tra

1. Nêu bốn công cụ animation và khi nào dùng mỗi cái.
2. AnimationPlayer có những loại track nào? Bật hitbox đúng khung bằng cách nào?
3. Khi nào cần AnimationTree?
4. Tween khác AnimationPlayer ở đâu?
5. Vì sao logic không nên phụ thuộc thời lượng animation?

## 12. TileMap và TileSet

**Ý chính:** bản đồ ô lưới gồm **một bộ ô (TileSet)** và **một lớp vẽ (TileMapLayer)** dùng bộ ô đó. Tile có thể mang cả **va chạm, dữ liệu riêng và luật vẽ tự động**.

### 12.1 Mô hình tư duy

TileSet là **tờ tem** (một ảnh gồm nhiều ô nhỏ). `TileMapLayer` là **tờ giấy ô vuông** bạn đóng tem lên. Bạn không vẽ từng pixel mà **chọn tem rồi đóng vào ô**. Nhiều lớp (nền, vật thể, trang trí) là nhiều tờ giấy chồng nhau.

### 12.2 Hai thành phần

|  | Là gì | Chứa |
| --- | --- | --- |
| **`TileSet`** (Resource) | bộ ô và thiết lập của chúng | **Sources** (atlas ảnh), **Tile Size**, các lớp thuộc tính (Physics, Navigation, Custom Data, Terrains) |
| **`TileMapLayer`** (Node) | một lớp lưới dùng một TileSet | ô nào dùng tile nào |

Từ Godot **4.3** mỗi lớp là một node `TileMapLayer` (thay cho node `TileMap` nhiều lớp ở bản cũ); bản cũ hơn dùng `TileMap`. Hãy xem phiên bản của bạn.

### 12.3 Quy trình

1. Thêm `TileMapLayer`, trong Inspector tạo **TileSet** mới.
2. Đặt **Tile Size** đúng bằng kích thước một ô trong ảnh (16x16, 32x32...). **Sai kích thước là lỗi số một.**
3. Mở panel **TileSet** (dưới cùng), kéo ảnh vào, đồng ý tạo tile tự động.
4. Thêm các **lớp thuộc tính** cần dùng:
   - **Physics Layer:** cho tile va chạm (vẽ hình va chạm trên tile).
   - **Navigation Layer:** vùng đi được cho AI tìm đường.
   - **Custom Data Layer:** dữ liệu của bạn (ví dụ `speed`, `loai_dat`) gán cho từng tile.
   - **Terrain Set:** luật **vẽ tự động nối viền** (autotile) cho cỏ, đất, nước.
5. Mở panel **TileMap**, chọn tile (hoặc terrain) và vẽ.

### 12.4 Terrain (vẽ tự động nối viền)

Terrain cho phép bạn "quét cọ" một loại địa hình và Godot tự chọn tile viền/góc đúng. Rất tiết kiệm công vẽ bản đồ; đổi lại cần thiết lập **bitmask** (ô nào nối với ô nào) lần đầu cho bộ tile.

### 12.5 Toạ độ ô và code

Ô lưới dùng toạ độ **nguyên** `Vector2I` (cột, hàng). Đổi qua lại với pixel:

```csharp
Vector2I cell = layer.LocalToMap(layer.ToLocal(worldPos));   // pixel -> ô
Vector2   pos = layer.MapToLocal(cell);                      // ô -> pixel (tâm ô)
TileData data = layer.GetCellTileData(cell);                 // null nếu ô trống
layer.SetCell(cell, 0, new Vector2I(1, 0));                  // sourceId, toạ độ tile trong atlas
layer.EraseCell(cell);
```

`GetCustomData("tên")` đọc dữ liệu riêng của tile (cần tạo Custom Data Layer cùng tên). Rất hợp game xây dựng, nông trại, chiến thuật.

### 12.5b Y-sort, lớp và vật thể

- Bản đồ nền không cần Y-sort; **vật cao** (cây, nhà) nên bật `YSortEnabled` cho lớp đó (hoặc đặt thành các scene riêng trong node Y-sort) để nhân vật đi sau/trước đúng.
- Tile cũng có thể là **scene** (nhóm "Scenes Collection") để đặt vật thể có hành vi (cây có va chạm, rương).

### 12.6 Bảng quyết định

| Muốn... | Dùng |
| --- | --- |
| tường/sàn có va chạm | Physics Layer của TileSet |
| AI đi theo bản đồ | Navigation Layer |
| gán loại đất, tốc độ cho từng ô | Custom Data Layer |
| vẽ cỏ/đất tự nối viền | Terrain |
| đặt cây có hành vi | tile dạng scene hoặc scene riêng đặt trong cây Y-sort |
| nhiều lớp (nền, vật, trang trí) | nhiều `TileMapLayer` |

### 12.7 Sai lầm thường gặp

- **Tile Size** sai khiến ô bị cắt lệch.
- **Khe hở/đường kẻ** giữa tile khi zoom hoặc di chuyển camera: dùng Texture Filter `Nearest`, bật padding cho atlas, tránh scale lẻ.
- Tile có hình va chạm nhưng quên cấu hình **Collision Layer/Mask** của TileMapLayer.
- Dùng sai node theo phiên bản (`TileMap` và `TileMapLayer`).
- Dùng tile cho vật thể phức tạp thay vì scene.

### 12.8 Tự kiểm tra

1. TileSet và TileMapLayer khác nhau thế nào? Hình dung bằng tem và giấy ô vuông.
2. Bốn loại lớp thuộc tính của TileSet và công dụng?
3. Tile Size sai gây hậu quả gì?
4. Đổi giữa pixel và ô lưới bằng hàm nào?
5. Terrain giúp gì?
6. Khi nào dùng scene thay vì tile?

## 13. Âm thanh

**Ý chính:** có hai kiểu player (toàn cục và theo vị trí), và hệ thống **bus** để chỉnh âm lượng theo nhóm (nhạc, hiệu ứng, UI).

### 13.1 Mô hình tư duy

`AudioStreamPlayer` là **cái loa**; `AudioStream` là **đĩa nhạc** bỏ vào loa; **bus** là **các núm âm lượng** cho từng nhóm loa (Nhạc, SFX, UI) và **Master** là núm tổng.

### 13.2 Các player

| Node | Dùng cho |
| --- | --- |
| `AudioStreamPlayer` | nhạc nền, tiếng UI (không theo vị trí) |
| `AudioStreamPlayer2D` | tiếng **có vị trí** trong thế giới (nhỏ dần theo khoảng cách) |

Thuộc tính chính: `Stream` (file âm thanh), `Autoplay`, `Bus`, `VolumeDb`, `PitchScale`. Hàm: `Play()`, `Stop()`; signal `Finished`.

### 13.3 Định dạng

| Định dạng | Nên dùng |
| --- | --- |
| `.wav` | tiếng **ngắn** (nhảy, nhặt xu, bắn): nhanh, không nén |
| `.ogg` | **nhạc** và tiếng dài: nhẹ, hỗ trợ loop |
| `.mp3` | được hỗ trợ, nhưng `.ogg` thường tiện hơn cho game |

Bật **Loop** trong thiết lập import của file nhạc nền.

### 13.4 Bus và âm lượng

Mở panel **Audio** (dưới cùng) tạo các bus **Music**, **SFX**, **UI** gửi về **Master**; gán `Bus` cho từng player. Có thể thêm **hiệu ứng** (reverb, EQ, lọc) lên bus. Âm lượng tính theo **dB** (logarit); thanh trượt cài đặt của người chơi (0..1) cần đổi bằng `Mathf.LinearToDb`:

```csharp
int idx = AudioServer.GetBusIndex("Music");
AudioServer.SetBusVolumeDb(idx, Mathf.LinearToDb(sliderValue));   // sliderValue 0..1
AudioServer.SetBusMute(idx, false);
```

### 13.5 Phát nhiều tiếng chồng nhau

Một `AudioStreamPlayer` phát lại sẽ **cắt tiếng đang phát** (mặc định). Với tiếng bắn/bước chân dồn dập, tăng **`MaxPolyphony`** của player, hoặc tạo player mới cho mỗi tiếng (rồi xoá khi `Finished`), hoặc dùng một nhóm player xoay vòng.

### 13.6 Nhạc xuyên màn

Nhạc đặt trong scene sẽ **dừng khi đổi scene**. Muốn nhạc liền mạch, đặt player nhạc trong **Autoload** (hoặc một scene Autoload `Music`).

### 13.7 Bảng quyết định

| Muốn... | Dùng |
| --- | --- |
| nhạc nền xuyên các màn | `AudioStreamPlayer` trong Autoload, file `.ogg` Loop |
| tiếng nhảy/nhặt xu | `AudioStreamPlayer` hoặc `2D` + `.wav` |
| tiếng nhỏ dần khi xa | `AudioStreamPlayer2D` |
| thanh chỉnh âm lượng | bus + `SetBusVolumeDb(LinearToDb)` |
| tiếng bắn dồn dập | tăng `MaxPolyphony` hoặc nhiều player |

### 13.8 Sai lầm thường gặp

- Gán thanh trượt 0..1 thẳng vào dB (âm lượng sai).
- Nhạc đặt trong scene nên ngắt khi đổi màn.
- Tiếng bị cắt vì chỉ có một player và `MaxPolyphony = 1`.
- Quên bật Loop khi import nhạc nền.
- Không đặt bus nên không chỉnh được âm lượng theo nhóm.

### 13.9 Tự kiểm tra

1. `AudioStreamPlayer` và `AudioStreamPlayer2D` khác nhau thế nào?
2. Bus để làm gì? Vì sao cần ít nhất Music và SFX?
3. Vì sao đổi thanh trượt 0..1 sang dB cần `LinearToDb`?
4. Làm sao cho nhạc không dừng khi đổi màn?
5. `.wav` và `.ogg` nên dùng khi nào?

## 14. Hiển thị: độ phân giải, stretch và pixel art

**Ý chính:** game có một **viewport** (độ phân giải thiết kế) và một **cửa sổ** (kích thước thật). Chế độ **stretch** quyết định cách phóng viewport vào cửa sổ. Thiết lập đúng ngay từ đầu giúp game rõ nét trên mọi màn hình.

### 14.1 Các cài đặt (Project Settings > Display > Window)

| Cài đặt | Ý nghĩa |
| --- | --- |
| **Viewport Width/Height** | độ phân giải **thiết kế** (ví dụ 640x360 cho pixel art, 1920x1080 cho game HD) |
| **Window Width/Height Override** | kích thước cửa sổ khi chạy |
| **Stretch > Mode** | cách phóng: `disabled`, `canvas_items`, `viewport` |
| **Stretch > Aspect** | xử lý khi tỉ lệ cửa sổ khác tỉ lệ thiết kế (`keep`, `expand`, ...) |
| **Stretch > Scale Mode** | `fractional` hoặc `integer` (số nguyên, giữ pixel đều) |

### 14.2 Các chế độ stretch

| Mode | Hành vi | Hợp với |
| --- | --- | --- |
| `disabled` | không phóng, 1 pixel là 1 pixel | công cụ, ứng dụng |
| `canvas_items` | vẽ lại ở độ phân giải thật, nét sắc | **UI sắc nét, game HD, pixel art nhẹ** |
| `viewport` | vẽ ở độ phân giải thiết kế rồi phóng ảnh | **pixel art chuẩn** (mọi pixel to đều nhau) |

`Aspect = keep` giữ tỉ lệ và thêm viền đen nếu cần; `expand` mở rộng vùng nhìn thấy theo tỉ lệ cửa sổ (cần thiết kế UI chịu được).

### 14.3 Thiết lập cho pixel art

1. **Viewport nhỏ** (320x180, 480x270 hoặc 640x360) và **Window Override** gấp số nguyên lần (1280x720, 1920x1080).
2. **Stretch Mode = `viewport`** (hoặc `canvas_items` với Scale Mode `integer`).
3. **Rendering > Textures > Canvas Textures > Default Texture Filter = `Nearest`**.
4. Bật **Snap 2D Transforms to Pixel** (và vertices nếu cần) để tránh rung/mờ khi di chuyển.
5. Import ảnh bằng preset **2D Pixel**.
6. Camera: dùng zoom và làm mượt cẩn thận (zoom không nguyên có thể gây rung pixel).

### 14.4 UI và độ phân giải

Vì UI dùng anchor/container (chương 10), khi viewport co giãn UI tự khớp. Đặt UI trong `CanvasLayer` và kiểm tra ở vài tỉ lệ màn hình (16:9, 16:10, 21:9) trước khi phát hành.

### 14.5 Chế độ cửa sổ trong code

```csharp
DisplayServer.WindowSetMode(DisplayServer.WindowMode.Fullscreen);
DisplayServer.WindowSetMode(DisplayServer.WindowMode.Windowed);
```

### 14.6 Sai lầm thường gặp

- Pixel art bị mờ: thiếu `Nearest` hoặc Stretch Mode chưa phù hợp.
- Cửa sổ không phải bội số nguyên của viewport nên pixel to nhỏ không đều.
- Không thử ở tỉ lệ màn hình khác (UI bị cắt).
- Camera zoom lẻ làm hình rung.
- Đặt viewport quá lớn cho pixel art.

### 14.7 Tự kiểm tra

1. Viewport và cửa sổ khác nhau ở đâu?
2. `canvas_items` và `viewport` khác nhau thế nào? Khi nào dùng cái nào?
3. Liệt kê các thiết lập cho pixel art rõ nét.
4. `Aspect = keep` và `expand` khác nhau gì?

## 15. Hiệu ứng nhanh: particles, ánh sáng, shader

**Ý chính:** ba công cụ giúp game đẹp hơn mà không cần vẽ thêm: **hạt (particles)**, **ánh sáng 2D**, **shader**. Chỉ cần nhận diện công dụng ở giai đoạn đầu.

### 15.1 Particles (hạt)

| Node | Đặc điểm |
| --- | --- |
| `GPUParticles2D` | chạy trên GPU, nhiều hạt, hiệu ứng đẹp |
| `CPUParticles2D` | chạy trên CPU, nhẹ, tương thích rộng (hợp renderer Compatibility/máy yếu) |

Thiết lập hay dùng: `Amount`, `Lifetime`, `OneShot` (nổ một lần), `Emitting`, hướng và vận tốc, màu theo đời hạt. Ví dụ: bụi chân, tia lửa khi trúng, khói.

### 15.2 Ánh sáng 2D

| Node | Công dụng |
| --- | --- |
| `CanvasModulate` | nhuộm cả cảnh (ban ngày/đêm, hang tối) |
| `PointLight2D` | đèn điểm (đuốc, đèn pin) |
| `DirectionalLight2D` | ánh sáng một hướng (mặt trời) |
| `LightOccluder2D` | vật **cản sáng** đổ bóng |

Sprite có thể có **normal map** (qua `CanvasTexture`) để phản ứng với ánh sáng như có khối. Ánh sáng làm tăng chi phí vẽ; dùng vừa phải.

### 15.3 Shader

Shader là chương trình nhỏ chạy trên GPU quyết định **màu mỗi điểm ảnh**. Gắn qua `ShaderMaterial` vào thuộc tính `Material` của bất kỳ node vẽ nào. Việc hay làm: nháy trắng khi bị đánh, viền nhân vật, đổi bảng màu, hiệu ứng nước, tan biến. Điều khiển từ code qua `SetShaderParameter("tên", giá_trị)`. Giai đoạn đầu chỉ cần biết shader **tồn tại** và nơi gắn nó; học sau khi nắm vững các chương trước.

### 15.4 Sai lầm thường gặp

- Dùng quá nhiều ánh sáng/hạt làm game chậm trên máy yếu.
- Dùng `GPUParticles2D` ở renderer/máy không hợp (chuyển `CPUParticles2D`).
- Nhiều sprite dùng chung một `ShaderMaterial` nên đổi tham số là đổi tất cả (dùng bản riêng hoặc `instance uniform`).
- Quên đặt `OneShot` cho hiệu ứng nổ nên lặp mãi.

### 15.5 Tự kiểm tra

1. `GPUParticles2D` và `CPUParticles2D` khác nhau thế nào?
2. `CanvasModulate`, `PointLight2D`, `LightOccluder2D` mỗi cái để làm gì?
3. Shader gắn vào đâu và điều khiển từ code ra sao?
4. Vì sao nhiều sprite dùng chung một ShaderMaterial gặp vấn đề?

## 16. Debug và đo hiệu năng

**Ý chính:** gỡ lỗi là một quy trình (tái hiện, thu hẹp, quan sát, sửa), và Godot có sẵn công cụ cho từng bước. Đa số lỗi người mới gặp có triệu chứng quen thuộc, tra bảng ở cuối là ra nguyên nhân.

### 16.1 Mô hình tư duy

Giống bác sĩ: **triệu chứng** (nhân vật xuyên tường) → **xét nghiệm** (bật hiện hình va chạm) → **chẩn đoán** (sai layer/mask) → **chữa**. Đừng sửa mò; hãy **quan sát** trước.

### 16.2 Các công cụ

| Công cụ | Ở đâu | Dùng để |
| --- | --- | --- |
| **Output** | dock dưới | xem `GD.Print`, lỗi, cảnh báo |
| **Debugger > Errors** | dock dưới | lỗi lúc chạy kèm **stack trace** (dòng đầu thuộc code của bạn là chỗ lỗi) |
| **Remote** (Scene dock) | khi game chạy | xem và sửa thuộc tính node **đang chạy** (không lưu) |
| **Debug > Visible Collision Shapes** | menu Debug | hiện hình va chạm trong game |
| **Debug > Visible Navigation / Paths** | menu Debug | hiện vùng đi được, đường |
| **Debugger > Profiler** | dock dưới | thời gian từng hàm mỗi frame |
| **Debugger > Monitors** | dock dưới | FPS, bộ nhớ, số object, **orphan node**, draw call |
| **Cảnh báo vàng** | icon cạnh node trong Scene dock | cấu hình thiếu (ví dụ `CollisionShape2D` chưa có Shape) |

### 16.3 Quy trình gỡ lỗi

1. **Tái hiện** lỗi ổn định (lặp lại được).
2. **Thu hẹp:** tạo scene thử chỉ có phần nghi ngờ, chạy bằng `F6`.
3. **Quan sát:** `GD.Print` giá trị biến, hoặc **đặt breakpoint** (Rider/VS Code gắn vào Godot) và dò từng bước, xem biến và call stack.
4. **Giả thuyết** nguyên nhân, **đổi một thứ**, chạy lại. Đổi nhiều thứ cùng lúc thì không biết cái nào có tác dụng.
5. Ghi lại lỗi và cách sửa vào sổ lỗi cá nhân.

### 16.4 Triệu chứng và nguyên nhân thường gặp

| Triệu chứng | Nguyên nhân hay gặp | Cách kiểm tra |
| --- | --- | --- |
| Nhân vật **xuyên tường** | sai layer/mask; thiếu Shape; di chuyển bằng `Position` | bật *Visible Collision Shapes*, xem layer/mask hai bên |
| **Không nhặt được đồ** | mask của `Area2D` không chứa layer của Player | xem Collision của Area2D |
| **Signal không chạy** | chưa nối; node nhận đã bị xoá; sai tên | in log trong hàm nhận; kiểm tra tab Node > Signals |
| **`null`/NullReference** | quên kéo vào `[Export]`; `GetNode` sai đường dẫn; dùng trước `_Ready` | xem stack trace, in biến |
| **Sửa code mà game không đổi** | chưa **Build** hoặc build lỗi | đọc ô Build/Output |
| **`[Export]` không hiện** | chưa build; sai kiểu; class thiếu `partial` | build lại, kiểm tra tên file = tên class |
| Nhân vật **đi quá chậm** | nhân `Velocity` với `delta` trước `MoveAndSlide` | bỏ phần nhân `delta` |
| **Ảnh pixel bị mờ** | texture filter chưa `Nearest`, stretch chưa đúng | Project Settings > Rendering/Display |
| **Node mới tạo không hiện** | quên `AddChild`; sai vị trí; nằm sau camera/ZIndex thấp | xem Remote tree |
| **Game giật khựng** | cấp phát trong `_Process`, quá nhiều node, tải tài nguyên lúc chơi | Profiler, Monitors |
| **Vật bị vẽ sai thứ tự** | Y-sort hoặc gốc sprite không ở chân | kiểm tra `YSortEnabled`, origin |
| **Âm thanh bị cắt** | chỉ một player, `MaxPolyphony = 1` | tăng polyphony hoặc dùng nhiều player |

### 16.5 Đo hiệu năng

- Đặt **mục tiêu** (60 FPS tương đương tối đa \~16,6 ms mỗi frame).
- Dùng **Profiler** để thấy hàm nào chiếm nhiều thời gian; **Monitors** để thấy bộ nhớ/số node tăng bất thường (**orphan node** là node tạo ra nhưng không thêm vào cây hay không xoá).
- Đo **bản export Release** cho kết quả thật; khi chạy trong editor, bản Debug (nhất là C#) chậm hơn.
- **Đo trước, tối ưu sau.** Đừng tối ưu theo cảm tính.

### 16.6 Tự kiểm tra

1. Nêu quy trình gỡ lỗi năm bước.
2. *Visible Collision Shapes* dùng khi nào? Tab Remote dùng khi nào?
3. Nhân vật xuyên tường: bạn kiểm tra những gì, theo thứ tự?
4. Orphan node là gì và xem ở đâu?
5. Vì sao cần đo trên bản Release?
6. Stack trace đọc thế nào để tìm chỗ lỗi?

## 17. Export (đóng gói game)

**Ý chính:** export biến project thành file chạy được trên máy khác. Cần **Export Template** khớp phiên bản Godot, và mỗi nền tảng có yêu cầu riêng. Hãy xuất thử sớm, không để đến phút chót.

### 17.1 Quy trình chung

1. **Project > Export > Add preset** (Windows, Linux, macOS, Android, iOS, Web).
2. Cài **Export Templates** (Editor > Manage Export Templates), đúng phiên bản Godot.
3. Thiết lập preset: tên, icon, phiên bản, bộ lọc tài nguyên (Resources).
4. **Export Project** (hoặc *Export PCK/Zip*).
5. **Thử trên máy khác/cài mới**, không chỉ trên máy làm game.

### 17.2 Các nền tảng

| Nền tảng | Cần | Ghi chú với C# |
| --- | --- | --- |
| **Windows / Linux** | Export Templates | hỗ trợ tốt; thư mục xuất kèm dữ liệu .NET |
| **macOS** | Export Templates, có thể cần ký/duyệt | kiểm tra tài liệu phiên bản |
| **Android** | JDK, Android SDK, tạo **keystore** | C# hỗ trợ từ Godot 4.2 nhưng có hạn chế, **đọc trang Export Android của đúng phiên bản** |
| **iOS** | máy **macOS** + Xcode | hỗ trợ C# **kém hơn** nhiều; kiểm tra kỹ trước khi chọn |
| **Web** | Export Templates, máy chủ web | hỗ trợ C# **rất hạn chế** (tuỳ phiên bản) |

Nếu nhắm **iOS hoặc Web**, hãy xác nhận khả năng hỗ trợ C# trong tài liệu phiên bản của bạn **trước khi** làm xa; đây là rủi ro lớn nhất khi chọn C# cho Godot.

### 17.3 Danh sách kiểm tra trước khi xuất

- Đã đặt **Main Scene**, tên game, icon, phiên bản.
- Cài đặt cửa sổ/độ phân giải/stretch đúng.
- Đã gỡ code debug và `GD.Print` thừa.
- Đã thử bằng phím **và** tay cầm (nếu hỗ trợ).
- Tên file **đúng hoa thường** (Linux/Android phân biệt).
- Tài nguyên cần thiết **không bị lọc mất** (kiểm tra tab Resources của preset, nhất là file chỉ nạp bằng đường dẫn chuỗi).
- Lưu game dùng `user://`, không dùng `res://`.
- Test **cài mới** (không có dữ liệu cũ).

### 17.4 Lỗi thường gặp

- Thiếu hoặc sai phiên bản **Export Templates**.
- Android: sai đường dẫn SDK/JDK, thiếu keystore.
- Chạy được trong editor nhưng lỗi sau export: đường dẫn sai hoa thường, hoặc tài nguyên bị lọc.
- Ghi file vào `res://`.
- Chỉ thử trên máy làm game (thiếu thư viện/runtime ở máy khác).

### 17.5 Tự kiểm tra

1. Export Template là gì và vì sao phải đúng phiên bản?
2. Nền tảng nào cần máy macOS? Nền tảng nào cần keystore?
3. Vì sao game chạy trong editor lại có thể lỗi sau export? Nêu hai nguyên nhân.
4. `res://` và `user://` sau export khác nhau thế nào?
5. Rủi ro khi chọn C# cho iOS/Web?

## 18. Quản lý project và Git

**Ý chính:** project sống lâu cần **lưu lịch sử** (Git), **cấu trúc rõ ràng** và **một phiên bản Godot cố định**. Vài thói quen nhỏ cứu bạn khỏi mất công sức hàng tuần.

### 18.1 Git cho Godot

Git lưu mọi thay đổi để bạn quay lại khi hỏng và thử nghiệm không sợ. Vì scene/resource là **file văn bản**, Git hoạt động tốt. Điều cần biết:

```
# .gitignore tối thiểu cho Godot .NET
.godot/
.mono/
bin/
obj/
.idea/
.vs/
*.translation
```

- **Commit:** `.tscn`, `.tres`, `.cs/.gd`, `project.godot`, asset và file `.import`.
- **Không commit:** thư mục `.godot/` (bộ nhớ đệm tự tạo), `bin/`, `obj/`, file xuất (export), thông tin riêng như keystore.
- Quy trình tối thiểu: `git add .` → `git commit -m "mô tả việc đã làm"` → `git push`.
- Commit **nhỏ và thường xuyên**; thông điệp mô tả điều đã làm. Dùng **nhánh** (`git switch -c thu-nghiem`) cho thử nghiệm lớn.
- File nặng (nhạc dài, ảnh lớn): dùng **Git LFS**.

### 18.2 Tổ chức dự án

- Một scene một script gốc, cùng tên; scene nhỏ, tự đủ dùng (chương 4).
- Thư mục theo loại (chương 2.4); tên không dấu, không khoảng trắng; tiền tố rõ ràng cho asset (`spr_`, `sfx_`).
- **Một phiên bản Godot cho cả project** (xem `config/features` trong `project.godot`); mọi thành viên dùng cùng phiên bản.
- **README** ngắn: cách chạy, cấu trúc, quy ước; **danh sách lỗi đã biết**.
- **Sao lưu** thêm ngoài Git (ổ ngoài hoặc cloud).

### 18.3 Làm việc nhóm

Scene là file văn bản nhưng vẫn có thể **xung đột khi hai người cùng sửa một scene**. Giảm bằng cách chia scene nhỏ, mỗi người một scene, và thống nhất ai sửa file chung (`project.godot`, autoload).

### 18.4 Tự kiểm tra

1. Vì sao không commit thư mục `.godot/`?
2. Những loại file nào nên commit?
3. Vì sao nên khoá một phiên bản Godot?
4. Làm sao giảm xung đột scene khi làm nhóm?
5. Khi nào dùng nhánh Git?

## 19. Tổng hợp: bản đồ "muốn làm X dùng gì"

### 19.1 Bảng tra nhanh

| Muốn làm... | Dùng |
| --- | --- |
| nhân vật điều khiển được | `CharacterBody2D` + `CollisionShape2D` + `Sprite2D/AnimatedSprite2D` + script (`MoveAndSlide`) |
| tường, sàn | `StaticBody2D` hoặc tile có Physics Layer |
| thùng, bóng chịu lực | `RigidBody2D` |
| nhặt đồ, bẫy, hitbox, vùng tầm nhìn | `Area2D` + `CollisionShape2D` |
| biết có vật cản phía trước | `RayCast2D` |
| camera đi theo nhân vật | `Camera2D` làm con nhân vật |
| bản đồ ô lưới | `TileMapLayer` + `TileSet` |
| ảnh tĩnh / ảnh hoạt hình | `Sprite2D` / `AnimatedSprite2D` |
| bật hitbox đúng khung đánh | `AnimationPlayer` (track Property/Call Method) |
| nhiều animation chuyển phức tạp | `AnimationTree` |
| hiệu ứng mờ dần, nảy, rung bằng code | `Tween` |
| menu, HUD | `Control` + container, trong `CanvasLayer` |
| thanh máu | `ProgressBar` / `TextureProgressBar` |
| phong cách UI đồng bộ | `Theme` |
| nhạc nền xuyên màn | `AudioStreamPlayer` trong Autoload |
| tiếng nhảy/bắn | `AudioStreamPlayer` (hoặc 2D) |
| chỉnh âm lượng theo nhóm | Audio Bus |
| đếm giờ | `Timer` hoặc `await` timer |
| nối các phần lỏng lẻo | **signal** |
| trạng thái toàn cục (vàng, điểm) | **Autoload** |
| dữ liệu vật phẩm/kẻ địch trong editor | **Resource** tự định nghĩa |
| scene dùng lại nhiều lần | **instance** scene |
| biến thể của một scene | **Inherited scene** |
| thao tác hàng loạt | **Group** |
| sinh đối tượng lúc chơi | `PackedScene.Instantiate()` + `AddChild` |
| lưu/tải game | file trong `user://` (JSON) |
| nháy trắng, viền, hiệu ứng điểm ảnh | `ShaderMaterial` |
| bụi, khói, tia lửa | `CPUParticles2D` / `GPUParticles2D` |
| ánh sáng, đêm | `CanvasModulate`, `PointLight2D`, `LightOccluder2D` |
| pixel art rõ nét | viewport nhỏ + stretch + `Nearest` + import 2D Pixel |
| xuất game | Project > Export + Export Templates |

### 19.2 Chọn node gốc cho một scene mới

```
Đối tượng này là gì?
 ├─ Nhân vật/kẻ địch điều khiển bằng code, va chạm vật lý  → CharacterBody2D
 ├─ Vật chịu lực/lăn/đẩy                                   → RigidBody2D
 ├─ Địa hình đứng yên                                      → StaticBody2D
 ├─ Vùng phát hiện/nhặt đồ/hitbox                          → Area2D
 ├─ Chỉ là hình ảnh/nhóm có vị trí                         → Node2D (hoặc Sprite2D)
 ├─ Giao diện                                              → Control (hoặc container)
 ├─ Không có vị trí, chỉ logic/dịch vụ                     → Node
 └─ Màn chơi (chứa các scene khác)                         → Node2D
```

### 19.3 Từ điển thuật ngữ

- **Node:** đơn vị chức năng nhỏ nhất trong cây.
- **Scene:** cây node lưu thành file, dùng lại được.
- **SceneTree:** cây tổng của game đang chạy.
- **Instance:** một bản sao sống của scene.
- **Resource:** dữ liệu lưu được, dùng chung (`.tres`/`.res`).
- **Signal:** thông báo node phát ra để node khác lắng nghe.
- **Autoload:** node toàn cục sống suốt game.
- **Group:** nhãn gắn cho node để thao tác hàng loạt.
- **Inspector:** bảng sửa thuộc tính.
- **Export (biến):** biến hiện trong Inspector; **Export (game):** đóng gói game.
- **Delta:** thời gian giữa hai frame/bước vật lý (giây).
- **Layer/Mask:** nhóm tôi thuộc / nhóm tôi để ý trong va chạm.
- **Collision Shape:** hình dạng va chạm của body/area.
- **TileSet / TileMapLayer:** bộ ô / lớp lưới dùng bộ ô đó.
- **Autotile (Terrain):** vẽ tự động nối viền ô.
- **Y-sort:** vẽ theo toạ độ Y để có chiều sâu.
- **Bus (âm thanh):** nhóm âm lượng.
- **Stretch:** cách phóng viewport vào cửa sổ.
- **Orphan node:** node không nằm trong cây và không bị xoá (rò rỉ).
- **Export Template:** bộ khung runtime dùng để đóng gói game.

---

# CỔNG KIỂM TRA GODOT CƠ BẢN

Làm **không nhìn tài liệu**. Đạt khi **đúng từ 80% phần A** và **làm được ít nhất 5/6 bài phần B**. Chưa đạt thì ôn các chương tương ứng rồi làm lại. Sau khi qua cổng, bạn sẵn sàng làm các dự án thực hành (tài liệu chính, Phụ lục B).

## Phần A. 36 câu lý thuyết

**Khái niệm nền (chương 1–4)**

1. Nêu bảy ý lớn của Godot.
2. Scene giống class, instance giống object: giải thích.
3. Instance, Inherited scene, Duplicate khác nhau thế nào?
4. Di chuyển node cha thì con ra sao? Vị trí con tính so với gì?
5. "Gọi xuống, báo lên" nghĩa là gì?
6. `.tscn`, `.tres`, `project.godot`, `.godot/` mỗi thứ là gì? Cái nào không commit Git?
7. `res://` và `user://` khác nhau thế nào khi đã export?
8. `F5` và `F6` khác nhau ra sao?

**2D, Resource, Script, Signal (chương 5–8)**

9. `Position` và `GlobalPosition`; trục Y hướng nào?
10. `Modulate` và `SelfModulate` khác nhau thế nào?
11. Bốn yếu tố quyết định thứ tự vẽ?
12. Vì sao HUD đặt trong `CanvasLayer`?
13. Resource khác Node ở bản chất nào? Cho ví dụ.
14. Hai node dùng chung một Resource, sửa một bên thì sao? Cách tách bản riêng?
15. Vẽ vòng đời của node (`_EnterTree`, `_Ready`, `_Process`, `_PhysicsProcess`, `_ExitTree`).
16. `_Process` và `_PhysicsProcess` khác nhau; logic nào đặt ở đâu?
17. Vì sao ưu tiên `[Export]` hơn đường dẫn chuỗi để lấy node?
18. Autoload là gì, khác scene hiện tại thế nào khi đổi màn? Đừng lạm dụng vì sao?
19. Giải thích signal bằng ví dụ cái chuông. Hai cách nối?
20. Quy tắc đặt tên delegate signal trong C#? Tên signal đặt theo thì nào?

**Vật lý, UI, Animation, TileMap, Âm thanh, Hiển thị (chương 9–15)**

21. Bốn loại đối tượng vật lý? Chọn loại nào cho tường, người chơi, thùng đẩy được, đồng xu?
22. Vì sao body cần `CollisionShape2D` và phải gán `Shape`? Vì sao không scale nó?
23. Quy tắc layer/mask? Vì sao không đối xứng?
24. Vì sao không nhân `Velocity` với `delta` trước `MoveAndSlide`?
25. Container và anchor khác nhau thế nào? `Fill` và `Expand` là gì?
26. Mouse Filter có ba giá trị nào? Vì sao panel phủ màn hình hay chặn nhấp chuột?
27. Bốn công cụ animation và khi nào dùng từng cái?
28. AnimationPlayer có những loại track nào? Bật hitbox đúng khung thế nào?
29. TileSet và TileMapLayer khác nhau? Bốn loại lớp thuộc tính của TileSet?
30. Tile Size sai gây gì? Đổi giữa pixel và ô lưới bằng hàm nào?
31. `AudioStreamPlayer` và `...2D` khác nhau? Bus để làm gì? Vì sao dùng `LinearToDb`?
32. Làm sao để nhạc không dừng khi đổi màn?
33. `canvas_items` và `viewport` khác nhau thế nào? Thiết lập cho pixel art rõ nét?
34. `GPUParticles2D` và `CPUParticles2D`? `CanvasModulate`/`PointLight2D`/`LightOccluder2D` làm gì?

**Debug và Export (chương 16–18)**

35. Nhân vật xuyên tường: kiểm tra những gì? Sửa code mà game không đổi: nguyên nhân?
36. Export Template là gì? Hai lý do game chạy trong editor lại lỗi sau export?

### Đáp án gợi ý (tự đối chiếu sau khi làm)

- **1:** node, scene, SceneTree, script, signal, resource, cấu hình toàn cục.
- **2–3:** scene là bản thiết kế dùng lại; instance là bản sao nối với gốc; inherited thêm/sửa phần riêng; duplicate sao chép rời.
- **4–5:** con đi theo, vị trí con tính so với cha; cha gọi hàm con, con phát signal.
- **6:** scene, resource, cấu hình, bộ nhớ đệm (`.godot/` không commit).
- **7:** `res://` chỉ đọc, `user://` ghi được.
- **8:** F5 chạy từ Main Scene, F6 chạy scene đang mở.
- **9:** local so với cha / thế giới; Y hướng xuống.
- **10–12:** Modulate ảnh hưởng cả con, SelfModulate chỉ chính nó; thứ tự cây, ZIndex, Y-sort, CanvasLayer; HUD không bị camera kéo.
- **13–14:** Resource là dữ liệu dùng chung, Node là hành vi trong cây; sửa chung đổi cả hai, tách bằng Make Unique/Duplicate.
- **15–16:** `_Ready` một lần, con trước cha; di chuyển/va chạm ở `_PhysicsProcess`, hiển thị ở `_Process`.
- **17–18:** không gãy khi đổi tên/chuyển node; autoload sống qua đổi màn, lạm dụng gây phụ thuộc ngầm.
- **19–20:** chuông, nối bằng editor hoặc code; kết thúc `EventHandler`, thì quá khứ.
- **21:** Static, Character, Rigid, Area; tường Static, người chơi Character, thùng Rigid, xu Area.
- **22–24:** hình va chạm là thứ engine dùng; không scale để khỏi méo; A thấy B khi mask A chứa layer B; MoveAndSlide đã dùng delta.
- **25–26:** container tự xếp, anchor bám cạnh; Fill lấp, Expand chiếm phần dư; Stop/Pass/Ignore.
- **27–28:** AnimatedSprite2D, AnimationPlayer, AnimationTree, Tween; Property/Call Method/Audio/Animation Playback.
- **29–30:** TileSet là bộ ô, TileMapLayer là lớp lưới; Physics, Navigation, Custom Data, Terrain; sai kích thước cắt lệch, `LocalToMap/MapToLocal`.
- **31–32:** global và có vị trí; nhóm âm lượng; dB logarit; để nhạc ở Autoload.
- **33–34:** vẽ ở độ phân giải thật / vẽ nhỏ rồi phóng; viewport nhỏ + Nearest + 2D Pixel; GPU nhiều hạt, CPU nhẹ; nhuộm cảnh, đèn điểm, vật cản sáng.
- **35–36:** layer/mask, Shape, cách di chuyển, Visible Collision Shapes; chưa build; template khung runtime, sai hoa thường hoặc tài nguyên bị lọc.

## Phần B. 6 bài thực hành nhỏ (không phải dự án)

Làm trong project `Lab`, mỗi bài dưới 1 giờ, có kết quả nhìn thấy được.

1. **Scene và va chạm:** dựng scene `Player` (`CharacterBody2D` + `CollisionShape2D` + `Sprite2D`) và scene `Level` có sàn bằng `StaticBody2D`; Player đi và nhảy được, không xuyên sàn.
2. **Area và signal:** scene `Coin` bằng `Area2D`; chạm vào thì in ra và biến mất; cấu hình layer/mask đúng.
3. **UI bằng container:** menu có tiêu đề và 3 nút xếp bằng `VBoxContainer` trong `CenterContainer`, dùng một `Theme`; bấm nút Play đổi sang scene khác.
4. **Animation:** `AnimatedSprite2D` có `idle` và `run`; và một `AnimationPlayer` bật/tắt một hitbox ở khoảng thời gian cố định.
5. **TileMap:** `TileMapLayer` với TileSet có Physics Layer, vẽ bản đồ nhỏ, nhân vật va chạm đúng; thêm một Custom Data Layer và đọc nó bằng code.
6. **Export:** xuất bản chạy Windows (hoặc Linux) và chạy được trên máy khác hoặc thư mục sạch.

## Sau khi qua cổng

Bạn đã nắm Godot ở mức lý thuyết và công cụ. Bước tiếp theo:

1. Học song song hoặc sau đó **Giáo trình lý thuyết C# trong Godot** (ngôn ngữ và cách C# gắn với engine).
2. Làm **dự án 1 (Platformer)** và **dự án 2 (Top-down RPG)** trong *Tài liệu học Godot 4 + C# từ cơ bản đến nâng cao*.
3. Khi gặp tutorial GDScript, dùng Phụ lục G và H của tài liệu chính để dịch sang C#.

**Lịch gợi ý 10 ngày:** ngày 1 (chương 1–3), 2 (4), 3 (5–6), 4 (7–8), 5 (9), 6 (10), 7 (11–12), 8 (13–15), 9 (16–18), 10 (19 + cổng kiểm tra). Mỗi ngày: đọc 10 phút, thực hành 20 phút trong `Lab`, rồi tự kiểm tra.

# Chương 1: Godot - Khái Niệm Cơ Bản & Bản Đồ Tư Duy

> [!NOTE]
> **Ý chính:** Godot xây game bằng **node** ghép thành **scene**, scene lại ghép thành scene lớn hơn. Hiểu bảy ý dưới đây là hiểu 80% Godot.

## 1. Bảy ý lớn

| #   | Ý                     | Một câu                                                                                          |
| --- | --------------------- | ------------------------------------------------------------------------------------------------ |
| 1   | **Node**              | Đơn vị nhỏ nhất, mỗi node làm **một việc** (vẽ ảnh, giữ hình va chạm, phát âm thanh, đếm giờ...) |
| 2   | **Scene**             | Một **cây node** lưu thành file `.tscn`; dùng lại được nhiều lần                                 |
| 3   | **SceneTree**         | Cây tổng của game đang chạy; node cha chứa node con                                              |
| 4   | **Script**            | Gắn vào node để thêm **hành vi** (code C# hoặc GDScript)                                         |
| 5   | **Signal**            | Node **phát tín hiệu**, node khác **lắng nghe**; để các phần giao tiếp lỏng lẻo                  |
| 6   | **Resource**          | **Dữ liệu** dùng chung lưu được ra file (texture, âm thanh, dữ liệu bạn tự thiết kế)             |
| 7   | **Cấu hình toàn cục** | Project Settings, Input Map, Autoload, Layer Names                                               |

## 2. Mô hình tư duy: Lego

- **Node = một mảnh Lego** (mảnh bánh xe, mảnh đèn, mảnh khung).
- **Scene = một mô hình đã lắp** (chiếc xe). Quan trọng: **cả chiếc xe lại dùng như một mảnh Lego lớn** để lắp vào mô hình to hơn (thành phố).
- **Script = bộ điều khiển** gắn vào một mảnh để nó biết cử động.
- **Signal = cái chuông**: mảnh này reo chuông khi có chuyện, ai đã treo tai nghe thì nghe thấy; mảnh reo chuông không cần biết ai nghe.
- **Resource = bản vẽ và nguyên liệu** (màu sơn, bảng thông số) cất trong kho, nhiều mảnh cùng dùng chung.

> [!TIP]
> Làm game như gắn phụ tùng cho 1 mô hình Lego. Muốn thêm chức năng gì thì gắn lên và hoàn toàn có thể sử dụng lại bằng cách biến nó thành 1 đối tượng như trong OPP. Học kỹ OPP chỉ cần có tư duy hướng đối tượng vững thì khả năng tư duy trong **Godot** sẽ cực kỳ dễ!

## 3. Một game Godot nhìn từ trên cao

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

## 4. Hai câu hỏi bạn luôn tự hỏi khi làm Godot

1. **"Điều này là node, scene hay resource?"** (hành vi sống trong cây → node/scene; dữ liệu lưu dùng chung → resource)
2. **"Ai nên biết ai?"** (cha biết con để gọi xuống; con phát signal để báo lên, không cần biết cha)

## 5. Tự kiểm tra

1. Nêu bảy ý lớn của Godot bằng lời của bạn.
2. Vì sao nói "scene dùng như một mảnh Lego lớn"?
3. Signal khác gọi hàm trực tiếp ở điểm nào?
4. Autoload nằm ở đâu trong cây và khác scene hiện tại thế nào?

---

**Tiếp theo:** [Task 2](task_2.md)

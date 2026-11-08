# Lộ trình học Godot 4 + C#

Đây là tài liệu **đi kèm** với tài liệu chính "Tài liệu học Godot 4 + C# từ cơ bản đến nâng cao". Tài liệu chính rất dài (gồm Phần I tra cứu, Phần II các phụ lục A–E, Phần III game dàn quân, Phần IV chuyển GDScript sang C#), nên **đừng đọc từ đầu đến cuối**. Hãy làm theo lộ trình này: mỗi chặng cho biết đọc mục nào, làm gì, tự kiểm tra gì, đạt rồi mới sang chặng sau.

## 1. Bản đồ tài liệu chính

| Phần | Gồm | Dùng để |
| --- | --- | --- |
| **Phần I** (mục 1–21) | Cài đặt, Node/Scene, C# trong Godot, input, physics, tilemap, animation, signal, resource, autoload, UI, audio, kiến trúc, navigation, shader, C# nâng cao, debug, export | **Tra cứu** theo chủ đề; đọc khi gặp đúng chủ đề |
| **Phụ lục A** | C# từ con số 0 + 10 bài tập | **Học** nền tảng ngôn ngữ |
| **Phụ lục B** | Dự án 1: Platformer từng bước | **Làm theo** (tuần 1–2) |
| **Phụ lục C** | Dự án 2: Top-down RPG từng bước | **Làm theo** (tuần 3) |
| **Phụ lục D** | Quy trình làm dự án thật, Git, cấu trúc thư mục | Đọc trước khi làm dự án riêng |
| **Phụ lục E** | Cheat sheet, lịch 28 ngày, từ điển, FAQ | Để bên cạnh khi học |
| **Phụ lục F** (Phần III) | Game chiến thuật dàn quân, 2D nhìn như 3D | Dự án riêng của bạn |
| **Phụ lục G, H** (Phần IV) | Chuyển GDScript sang C#, ví dụ song song | Khi đọc tutorial GDScript |

Cách nhớ: **Phần I = từ điển**, **Phụ lục A–C = giáo trình thực hành**, **D–F = áp dụng vào dự án thật**, **G–H = công cụ đọc tutorial**.

## 2. Nguyên tắc học (ngắn gọn nhưng quan trọng)

1. **20% đọc, 80% gõ code.** Đọc xong một đoạn là mở Godot gõ ngay.
2. **Quy tắc ba lần:** đọc code → gõ lại bằng tay (không copy) → đóng tài liệu, tự viết lại từ trí nhớ. Chưa viết lại được thì chưa thuộc.
3. **Đừng nhảy chặng.** Có mục "tự kiểm tra" ở cuối mỗi chặng; không đạt thì ở lại thêm 1–2 ngày.
4. **Sổ lỗi:** mỗi lỗi gặp, ghi một dòng gồm *thông báo lỗi → nguyên nhân → cách sửa*. Sau một tháng đây là tài liệu quý nhất của bạn.
5. **Commit Git mỗi cuối buổi** (xem Phụ lục D.5), để không mất code và thấy tiến bộ.
6. **Chỉ học một thứ mới mỗi buổi.** Học nhiều thứ cùng lúc dễ rối và quên.
7. Gặp tutorial viết bằng GDScript: đừng bỏ qua, dùng Phụ lục G và H để dịch sang C#.
8. Khi bí quá 30 phút: thu nhỏ vấn đề, đọc lại thông báo lỗi, rồi hỏi (Phụ lục D.7).

## 3. Các chặng học

### Chặng 0. Chuẩn bị (nửa ngày đến 1 ngày)

- **Đọc:** Phần I mục 1 (cài đặt và thiết lập).
- **Làm:** cài Godot bản .NET, .NET SDK 8, Rider; tạo project; tạo script `Hello` in một dòng ra Output.
- **Tự kiểm tra:** Bạn chạy được game trống (F5), thấy dòng in ra, và biết bấm Build khi sửa code.

### Chặng 1. Nền tảng: C# và Node/Scene (ngày 1–4)

- **Đọc theo thứ tự:** Phần I mục 2 → Phụ lục A.1–A.7 → Phần I mục 3 → Phụ lục A.8–A.14 → Phần I mục 4.
- **Làm:** 10 bài tập ở Phụ lục A.15; bài tập cuối mục 3 và mục 4.
- **Tự kiểm tra (trả lời không nhìn tài liệu):**
  1. Node, Scene, SceneTree khác nhau thế nào?
  2. `_Ready` và `_Process` được Godot gọi khi nào?
  3. Vì sao class phải `partial` và tên file phải trùng tên class?
  4. `NullReferenceException` thường do đâu trong Godot?
  5. Khi nào dùng `List<T>`, khi nào dùng `Godot.Collections.Array`?
- **Qua chặng khi:** làm xong 10 bài tập A và trả lời được 5 câu trên.

### Chặng 2. Dự án 1: Platformer (ngày 5–14)

- **Đọc và làm song song:** Phần I mục 5 → Phụ lục B.0 đến B.13. Khi đến phần nào cần giải thích sâu thì tra: mục 6 (tilemap), mục 7 (animation), mục 8 (signal), mục 9 (resource, lưu game), mục 12 (audio).
- **Làm:** hoàn thành checklist B.13 (nhân vật mượt, 2 màn, xu, gai, Slime, máu, HUD, menu, pause, xuất bản).
- **Tự kiểm tra:**
  1. Vì sao logic di chuyển đặt ở `_PhysicsProcess`?
  2. Layer và Mask khác nhau thế nào? Cho một ví dụ.
  3. Khi nào nên dùng signal thay vì gọi hàm trực tiếp?
  4. Vì sao đổi scene trong callback va chạm phải dùng `CallDeferred`?
  5. Coyote time và jump buffer giải quyết vấn đề gì?
- **Qua chặng khi:** game chạy được, người khác chơi thử mà không gặp lỗi crash, và bạn tự sửa được ít nhất 3 lỗi bằng cách đọc thông báo lỗi.

### Chặng 3. Dự án 2: Top-down RPG (ngày 15–21)

- **Đọc và làm song song:** Phần I mục 10 (autoload) → mục 6 (Y-sort) → Phụ lục C.1 đến C.10; tra mục 11 (UI), mục 13 (state machine, interface), mục 14 (navigation), mục 9.2 (lưu game).
- **Tự kiểm tra:**
  1. Autoload dùng khi nào và không nên lạm dụng ra sao?
  2. Interface `IInteractable` giúp gì so với kiểm tra từng loại node?
  3. Vì sao dữ liệu lời thoại và vật phẩm nên là Resource?
  4. Lưu vật phẩm vào file save, vì sao lưu `Id` thay vì lưu cả đối tượng?
  5. Y-sort cần node gốc nằm ở đâu của sprite?
- **Qua chặng khi:** hoàn thành checklist C.10.

### Chặng 4. Đọc hiểu tutorial GDScript (ngày 22–23)

- **Đọc:** Phụ lục G (từ G.0 đến G.16) và Phụ lục H (ví dụ dịch song song).
- **Làm:** bài tập G.17; chọn một tutorial GDScript bạn quan tâm và tự dịch sang C#.
- **Tự kiểm tra:** Dịch được một script GDScript 30–50 dòng (có signal, `@export`, `@onready`, setter) sang C# mà chạy đúng.

### Chặng 5. Kiến trúc, tối ưu, xuất bản (ngày 24–28)

- **Đọc:** Phần I mục 13 (nếu chưa), mục 16.1 đến 16.6 (C# nâng cao), mục 17 (debug, profiler), mục 18 (export), mục 20, 21; Phụ lục D.
- **Làm:** tái cấu trúc dự án 2 bằng state machine; dùng Profiler tìm một điểm nghẽn thật; xuất Windows và (nếu cần) Android.
- **Tự kiểm tra:**
  1. Vì sao cần cache `GetNode` và tránh cấp phát trong `_Process`?
  2. Làm sao biết `QueueFree` đã xoá một node và cách tránh dùng node đã xoá?
  3. Đo hiệu năng bằng công cụ nào trước khi tối ưu?
  4. Khác biệt `res://` và `user://` khi export?

### Chặng 6. Dự án riêng: game dàn quân (từ ngày 29)

- **Chuẩn bị:** Phụ lục D.1 đến D.7 (viết GDD một trang, giới hạn phạm vi, tạo repository Git, dựng cấu trúc thư mục).
- **Đọc Phụ lục F theo ba lượt:**
  1. **Lượt 1 (nền tảng hiển thị):** F.1 đến F.8: tách logic/hiển thị, chọn kiểu nhìn, hàm chiếu, tilemap, ảo giác 3D, camera.
  2. **Lượt 2 (hệ thống quân):** F.9 đến F.13: kiến trúc, dữ liệu, địa hình, `Squad`/`Soldier`, bốn lệnh, kịch bản địch.
  3. **Lượt 3 (hoàn thiện):** F.14 đến F.19: hiệu năng, bố trận, chỉ huy, dữ liệu trận, milestone, lỗi thường gặp.
- **Làm:** theo milestone M0 → M9 ở F.18; mỗi mốc là một bản chạy được và commit Git.

## 4. Lịch 28 ngày chi tiết (khoảng 1,5–3 giờ mỗi ngày)

Cột "Đọc" là mục trong tài liệu chính. P1 = Phần I, PL = Phụ lục.

| Ngày | Đọc | Làm | Kiểm tra |
| --- | --- | --- | --- |
| 1 | P1 mục 1, 2 | Cài đặt, `Hello` | Chạy được F5 |
| 2 | PL A.1–A.4, P1 mục 3.1–3.2 | Bài A.15 số 1–3 | In được biến và điều kiện |
| 3 | PL A.5–A.7, P1 mục 3.3–3.6 | Bài A.15 số 4–7 | Viết được class nhỏ |
| 4 | PL A.8–A.14, P1 mục 4 | Bài A.15 số 8–10 | Đọc được thông báo lỗi |
| 5 | P1 mục 5, PL B.0–B.3 | Tạo project, Player chạy/nhảy | Nhảy được trên sàn |
| 6 | PL B.4–B.5 | Tilemap, xu, GameManager | Nhặt xu tăng số |
| 7 | PL D.5 | Git, ôn, sửa lỗi | Có commit đầu tiên |
| 8 | P1 mục 8, PL B.6–B.7 | HUD, gai, vùng rơi | HUD cập nhật qua signal |
| 9 | P1 mục 9.1, PL B.8 | Slime biết tuần tra | Giẫm chết được Slime |
| 10 | P1 mục 7, PL B.9 | Animation, nối Level | Chuyển màn |
| 11 | PL B.10, P1 mục 12 | Menu, pause, âm thanh | Có menu hoạt động |
| 12 | PL B.11–B.12 | Juice, xuất bản | Có file chạy được |
| 13 | PL B.13 | Hoàn thiện, sửa lỗi | Hết checklist B.13 |
| 14 | — | Nghỉ, ôn lại, cho người khác chơi thử | Ghi phản hồi |
| 15 | P1 mục 10, PL C.1–C.2 | Project mới, nhân vật 4 hướng | Đi 4 hướng mượt |
| 16 | P1 mục 6, PL C.3 | Y-sort, thế giới | Đi qua cây đúng thứ tự |
| 17 | PL C.4–C.6 | Interface, NPC, thoại | Nói chuyện được |
| 18 | P1 mục 11, PL C.5–C.6 | UI hộp thoại | Hộp thoại gõ chữ |
| 19 | PL C.7 | Inventory, rương | Nhặt đồ vào túi |
| 20 | P1 mục 14, PL C.8 | Kẻ địch đuổi theo | Địch tìm đường được |
| 21 | PL C.9, P1 mục 9.2 | Lưu/tải game | Lưu rồi mở lại đúng |
| 22 | PL G.0–G.8 | Dịch các mẫu đơn giản | Hiểu quy tắc đổi tên |
| 23 | PL G.9–G.17, PL H | Dịch một tutorial | Dịch chạy đúng |
| 24 | P1 mục 13 | Refactor bằng state machine | Code gọn hơn |
| 25 | P1 mục 16.1–16.6, 17 | Profiler, tối ưu một điểm | Có số liệu trước/sau |
| 26 | P1 mục 18 | Export Windows/Android | Chạy trên máy khác |
| 27 | PL D.1–D.4 | Viết GDD một trang | Có danh sách tính năng |
| 28 | PL D.5–D.6, PL F.1–F.5 | Repo dự án riêng, prototype M0 | Một lính có bóng đi lại |

Nếu một chặng khó, ở lại thêm 1–2 ngày; hiểu chắc quan trọng hơn đúng lịch. Tổng thể bạn có thể kéo dài ra 6–8 tuần, miễn là đều đặn.

## 5. Những phần có thể bỏ qua ở lần đọc đầu

Để không bị quá tải, hãy **bỏ qua** các phần sau cho đến khi cần dùng thật:

- Mục 15 (shader), mục 16.7 và 16.8 (tool script, GDExtension).
- Phụ lục F.14 phần MultiMesh, F.5 shader tilt-shift, F.6 quy trình pre-render từ Blender.
- Các mục về Android/iOS/Web ở mục 18 nếu bạn chỉ nhắm PC.
- Mục 14 (navigation) nếu game của bạn không cần tìm đường theo lưới (game dàn quân dùng AI riêng ở Phụ lục F).

Đây là phần "đọc sau", không phải phần "không quan trọng".

## 6. Sổ theo dõi tiến độ (sao chép và đánh dấu)

- [ ] Chặng 0: cài đặt xong, chạy `Hello`
- [ ] Chặng 1: 10 bài tập C#, trả lời 5 câu kiểm tra
- [ ] Chặng 2: Dự án 1 hoàn thành (checklist B.13), có người khác chơi thử
- [ ] Chặng 3: Dự án 2 hoàn thành (checklist C.10)
- [ ] Chặng 4: dịch được một tutorial GDScript sang C#
- [ ] Chặng 5: đã dùng Profiler, đã export ra máy khác
- [ ] Chặng 6: GDD một trang, repo Git, prototype M0
- [ ] M1–M2: 100 lính đánh nhau; stack và bốn lệnh
- [ ] M3–M4: bảng tương khắc, địa hình
- [ ] M5–M6: bố trận, kịch bản địch, thắng/thua
- [ ] M7–M8: UI chỉ huy, bản dọc một trận hoàn chỉnh

## 7. Khi bạn bị kẹt

1. Đọc kỹ thông báo lỗi (Phụ lục A.14), sửa lỗi đầu tiên rồi build lại.
2. Tra Phụ lục E.4 (FAQ lỗi thường gặp) và Phụ lục G.16, F.19.
3. Thu nhỏ vấn đề: tạo scene thử riêng chỉ có phần lỗi.
4. Hỏi cộng đồng hoặc AI, kèm **phiên bản Godot**, **đoạn code** và **thông báo lỗi**.
5. Nếu vẫn kẹt, ghi vào sổ lỗi, chuyển sang việc khác, quay lại sau một đêm.

# Giáo trình lý thuyết C# trong Godot (cơ bản đến nâng cao)

## Task 0: Tổng Quan

### 1. Cách dùng giáo trình này

Giáo trình này **không có dự án**. Mục tiêu là bạn nắm chắc lý thuyết C# và cách C# vận hành trong Godot, để khi bắt tay làm dự án (tài liệu chính, Phụ lục B trở đi) bạn không phải vừa làm vừa đoán. Bạn chỉ sang dự án khi vượt được **Cổng kiểm tra** ở cuối giáo trình.

### 2 Môi trường thực hành

- **[Phần 1](task_1.md) (ngôn ngữ C#, chương 1–9):** nên thực hành bằng **project console** cho nhanh, không cần mở Godot:

    ```
    dotnet new console -n Lab
    cd Lab
    dotnet run
    ```

    Sửa `Program.cs`, chạy lại, đọc kết quả. Vòng lặp rất nhanh nên học được nhiều.

- **[Phần 2](task_10.md) và [Phần 3](task_17.md) (Godot, chương 10–19):** tạo một project Godot .NET tên `Lab` chỉ có một scene trống. Mỗi bài tập là một script gắn vào một node, in kết quả bằng `GD.Print`.
- Editor: Rider (khuyến nghị) hoặc VS Code + C# Dev Kit.

> [!TIP]
>
> - Sinh viên hay là dev độc lập chưa muốn tốn tiền thì mình khuyên dùng **VS Code** nhé cài đầy đủ Extension và .Net dùng vẫn rất OK nhé!
> - Không thì có thể dùng gói **Free** cho SV và dự án phi lợi nhuận của **Rider** nhé!

---

**Tiếp theo:** [Task 1](task_1.md)

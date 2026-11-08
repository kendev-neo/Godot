# Tài liệu học Godot 4 + C# từ cơ bản đến nâng cao

Dành cho game 2D · Godot 4.3+ (bản .NET) · C# trên .NET 8 · Rider hoặc VS Code

## 0. Cách dùng tài liệu này

Tài liệu chia làm 4 giai đoạn, khớp với lộ trình 4 tuần học qua dự án mini:

| Tuần | Nội dung | Dự án mini |
| --- | --- | --- |
| 1 | Mục 1–5: cài đặt, Node/Scene, C# cơ bản, input, physics | Platformer 1 màn |
| 2 | Mục 6–9: tilemap, animation, signal, resource | Hoàn thiện platformer (kẻ địch, máu, nhặt đồ) |
| 3 | Mục 10–14: autoload, UI, audio, state machine, navigation | Top-down RPG nhỏ |
| 4 | Mục 15–19: shader, C# nâng cao, hiệu năng, export | Hoàn thiện, đóng gói, xuất bản |

Quy tắc học: đọc một mục, gõ lại code bằng tay (đừng copy), rồi sửa thử để xem điều gì xảy ra. Mỗi phần quan trọng có **Bài tập** cuối mục.

## 1. Cài đặt và thiết lập môi trường

### 1.1 Cần cài

- **Godot 4.x bản .NET** (tải ở godotengine.org, chọn bản có chữ .NET). Bản thường không chạy được C#.
- **.NET SDK 8** (từ dotnet.microsoft.com). Kiểm tra bằng `dotnet --version`.
- **Rider** (hoặc VS Code + C# Dev Kit). Trong Godot: Editor > Editor Settings > Dotnet > Editor > External Editor, chọn Rider.

### 1.2 Cấu trúc project

```
res://
  project.godot        # cấu hình project
  MyGame.csproj        # project C# (Godot tự tạo)
  MyGame.sln
  scenes/              # file .tscn
  scripts/             # file .cs
  assets/sprites, audio, fonts
  resources/           # file .tres (dữ liệu)
```

Quy ước quan trọng: **tên file .cs phải trùng tên class** (`Player.cs` chứa `class Player`), và class phải khai báo `partial`.

### 1.3 Build

Sau khi sửa code C#, phải **Build** (Ctrl+B trong editor Godot, hoặc nút búa góc trên phải) thì Godot mới thấy script/thuộc tính `[Export]` mới. Lỗi hay gặp nhất của người mới: sửa code xong mà Inspector không hiện biến, nguyên nhân là chưa build.

## 2. Khái niệm cốt lõi: Node, Scene, SceneTree

- **Node**: đơn vị nhỏ nhất, mỗi node làm một việc (Sprite2D vẽ ảnh, CollisionShape2D giữ hình va chạm, Timer đếm giờ...).
- **Scene**: một cây node được lưu thành file `.tscn`. Scene có thể lồng nhau và **tái sử dụng** (ví dụ `Enemy.tscn` dùng cho mọi kẻ địch).
- **SceneTree**: cây toàn bộ node đang chạy. Node cha xử lý trước, thứ tự con theo thứ tự trong cây.
- **Resource**: dữ liệu dùng chung (texture, âm thanh, hoặc dữ liệu tự tạo). Khác Node ở chỗ không nằm trong cây.

Tư duy Godot: **chia game thành nhiều scene nhỏ, mỗi scene tự đủ dùng**, rồi ghép lại. Ví dụ: `Player.tscn`, `Enemy.tscn`, `Coin.tscn`, `Level1.tscn`, `HUD.tscn`.

Các node 2D nền tảng cần biết:

| Node | Dùng để |
| --- | --- |
| `Node2D` | Node 2D cơ bản, có position/rotation/scale |
| `Sprite2D`, `AnimatedSprite2D` | Vẽ ảnh, vẽ animation theo frame |
| `CharacterBody2D` | Nhân vật điều khiển bằng code (player, enemy) |
| `RigidBody2D` | Vật thể do physics điều khiển (thùng, bóng) |
| `StaticBody2D` | Vật cố định (tường, sàn) |
| `Area2D` | Vùng phát hiện (nhặt đồ, hitbox, trigger) |
| `CollisionShape2D` | Hình va chạm, là con của các body/area |
| `Camera2D` | Camera |
| `TileMapLayer` | Bản đồ ô lưới |
| `Timer` | Đếm giờ |

## 3. C# trong Godot: nền tảng

### 3.1 Script đầu tiên

```csharp
using Godot;

public partial class Hello : Node
{
    public override void _Ready()
    {
        GD.Print("Node đã vào cây và sẵn sàng");
    }

    public override void _Process(double delta)
    {
        // chạy mỗi frame; delta = thời gian giữa 2 frame (giây)
    }
}
```

### 3.2 Vòng đời node (thứ tự gọi)

    1. `_EnterTree()`: node vừa vào cây (cha trước, con sau).
    2. `_Ready()`: node và **tất cả con** đã sẵn sàng (con trước, cha sau). Nơi an toàn để lấy tham chiếu node con.
    3. `_Process(double delta)`: mỗi frame hiển thị. Dùng cho logic không liên quan physics, UI, hiệu ứng.
    4. `_PhysicsProcess(double delta)`: nhịp cố định (mặc định 60 lần/giây). **Mọi logic di chuyển/va chạm đặt ở đây**.
    5. `_Input(InputEvent e)`, `_UnhandledInput(InputEvent e)`: xử lý sự kiện input.
    6. `_ExitTree()`: node rời cây. Nơi ngắt kết nối signal, dọn dẹp.

### 3.3 Quy ước đặt tên C# của Godot

API Godot trong C# dùng **PascalCase**: `Position`, `GetNode`, `QueueFree`, `IsActionPressed`. Tên node hay action thì giữ nguyên chữ bạn đặt (chuỗi).

### 3.4 Lấy node và tham chiếu

```csharp
public partial class Player : CharacterBody2D
{
    private AnimatedSprite2D _sprite;
    private Timer _coyoteTimer;

    public override void _Ready()
    {
        _sprite = GetNode<AnimatedSprite2D>("AnimatedSprite2D");
        _coyoteTimer = GetNode<Timer>("CoyoteTimer");
    }
}
```

Cách tốt hơn khi project lớn: dùng `[Export]` rồi kéo node vào Inspector, tránh đường dẫn chuỗi dễ gãy khi đổi tên node:

```csharp
[Export] private AnimatedSprite2D _sprite;
[Export] public PackedScene BulletScene;
[Export] public float Speed = 200f;
```

Có thể gom nhóm trong Inspector bằng `[ExportGroup("Di chuyển")]` và `[ExportCategory("Player")]`. Giá trị có giới hạn: `[Export(PropertyHint.Range, "0,100,1")] public int Health = 100;`.

### 3.5 Tạo và xoá node bằng code

```csharp
var scene = GD.Load<PackedScene>("res://scenes/Bullet.tscn");
var bullet = scene.Instantiate<Bullet>();
bullet.GlobalPosition = GlobalPosition;
GetParent().AddChild(bullet);

// Xoá an toàn (cuối frame):
QueueFree();
```

Luôn dùng `QueueFree()`, không dùng `Free()` trừ khi hiểu rõ rủi ro. Sau khi xoá, kiểm tra một node còn sống bằng `IsInstanceValid(node)`.

### 3.6 Kiểu dữ liệu hay dùng

`Vector2`, `Vector2I` (số nguyên, dùng cho toạ độ ô tilemap), `Rect2`, `Color`, `Transform2D`, `Mathf` (hàm toán: `Mathf.Lerp`, `Mathf.Clamp`, `Mathf.MoveToward`), `GD.Randf()`, `GD.RandRange()`.

**Bài tập:** tạo scene `Hello`, in ra tên node cha, đổi màu một Sprite2D mỗi giây bằng Timer.

## 4. Input

Khai báo action ở Project > Project Settings > Input Map (ví dụ `move_left`, `move_right`, `jump`, `attack`), gán phím và tay cầm. Không hard-code phím trong code.

```csharp
if (Input.IsActionPressed("move_right")) { ... }       // đang giữ
if (Input.IsActionJustPressed("jump")) { ... }          // vừa nhấn frame này
float dir = Input.GetAxis("move_left", "move_right");   // -1..1
Vector2 move = Input.GetVector("left", "right", "up", "down"); // top-down, đã chuẩn hoá
```

Xử lý sự kiện một lần (ví dụ bật menu):

```csharp
public override void _UnhandledInput(InputEvent e)
{
    if (e.IsActionPressed("pause"))
    {
        GetTree().Paused = !GetTree().Paused;
        GetViewport().SetInputAsHandled();
    }
}
```

Chuột: `GetGlobalMousePosition()` cho toạ độ thế giới, `GetLocalMousePosition()` cho toạ độ cục bộ.

## 5. Physics 2D và nhân vật platformer

### 5.1 CharacterBody2D

```csharp
using Godot;

public partial class Player : CharacterBody2D
{
    [Export] public float Speed = 200f;
    [Export] public float Accel = 1200f;
    [Export] public float JumpVelocity = -350f;

    private float _gravity = ProjectSettings.GetSetting("physics/2d/default_gravity").AsSingle();

    public override void _PhysicsProcess(double delta)
    {
        float dt = (float)delta;
        Vector2 v = Velocity;

        if (!IsOnFloor())
            v.Y += _gravity * dt;

        if (Input.IsActionJustPressed("jump") && IsOnFloor())
            v.Y = JumpVelocity;

        float dir = Input.GetAxis("move_left", "move_right");
        v.X = Mathf.MoveToward(v.X, dir * Speed, Accel * dt);

        Velocity = v;
        MoveAndSlide();
    }
}
```

Điểm cần nhớ:

- `Velocity` là property có sẵn, `MoveAndSlide()` dùng nó và tự xử lý trượt dọc tường/dốc.
- `IsOnFloor()`, `IsOnWall()`, `IsOnCeiling()` chỉ đúng **sau** `MoveAndSlide()`.
- Vector2 là struct: `Velocity.Y += 1` **không** hợp lệ trong C#; phải copy ra biến, sửa, rồi gán lại (như code trên).

### 5.2 Cảm giác nhảy tốt (game feel)

- **Coyote time:** cho nhảy trong khoảng \~0.1s sau khi rời mép sàn.
- **Jump buffer:** nhớ lệnh nhảy \~0.1s trước khi chạm đất.
- **Variable jump:** nhả phím sớm thì cắt vận tốc nhảy (`if (Input.IsActionJustReleased("jump") && v.Y < 0) v.Y *= 0.5f;`).
- Rơi nhanh hơn lúc đi lên (nhân gravity 1.5–2 khi `v.Y > 0`).

### 5.3 Va chạm: layer và mask

Mỗi vật có **Layer** (nó là gì) và **Mask** (nó nhìn thấy cái gì). Đặt tên layer trong Project Settings > Layer Names > 2D Physics (ví dụ 1 World, 2 Player, 3 Enemy, 4 PlayerHitbox, 5 Pickup). Thiết kế layer sớm giúp tránh vô số lỗi va chạm về sau.

### 5.4 Area2D: nhặt đồ, hitbox

```csharp
public partial class Coin : Area2D
{
    public override void _Ready()
    {
        BodyEntered += OnBodyEntered;
    }

    private void OnBodyEntered(Node2D body)
    {
        if (body is Player)
        {
            GD.Print("Nhặt xu");
            QueueFree();
        }
    }
}
```

Lưu ý: đổi trạng thái physics (tắt `CollisionShape2D`) ngay trong callback va chạm có thể báo lỗi; dùng `SetDeferred("disabled", true)` hoặc `CallDeferred`.

**Bài tập (Dự án tuần 1):** platformer 1 màn có nhân vật chạy/nhảy, coyote time, 5 đồng xu, một cái gai gây chết và reset màn (`GetTree().ReloadCurrentScene()`).

## 6. TileMapLayer và thế giới 2D

Từ Godot 4.3, mỗi lớp là một node `TileMapLayer` riêng (thay cho `TileMap` cũ).

1. Tạo `TileMapLayer`, trong Inspector tạo **TileSet**, kéo ảnh tileset vào, đặt Tile Size (16x16, 32x32...).
2. Trong TileSet thêm **Physics Layer** để tile có va chạm, **Navigation Layer** cho đường đi, **Terrains** để vẽ tự động nối viền (autotile).
3. Dùng nhiều `TileMapLayer`: nền, đồ vật, trang trí phía trước.

Đọc/ghi tile bằng code:

```csharp
[Export] private TileMapLayer _ground;

Vector2I cell = _ground.LocalToMap(_ground.ToLocal(worldPos));
TileData data = _ground.GetCellTileData(cell);
bool coTheXay = data == null;                    // ô trống
_ground.SetCell(cell, 0, new Vector2I(1, 0));    // sourceId, atlasCoords
_ground.EraseCell(cell);
```

Custom data layer trong TileSet (ví dụ `loai_dat`) đọc bằng `data.GetCustomData("loai_dat")`, rất hữu ích cho game xây dựng/nông trại.

**Y-sort:** bật `Y Sort Enabled` ở node cha chứa nhân vật, cây, nhà thì vật nằm thấp hơn trên màn hình vẽ đè lên vật nằm cao hơn. Cần thiết cho game top-down.

**Camera2D:** bật `Position Smoothing`, đặt `Limit Left/Right/Top/Bottom` theo kích thước map. Với pixel art: Project Settings > Display > Window > Stretch Mode = `canvas_items` hoặc `viewport`, và bật `Snap 2D Transforms to Pixel`; Texture Filter mặc định đặt `Nearest`.

## 7. Animation

### 7.1 AnimatedSprite2D

Đơn giản nhất: tạo `SpriteFrames` (idle, run, jump), gọi:

```csharp
_sprite.Play("run");
_sprite.FlipH = dir < 0;
```

### 7.2 AnimationPlayer

Mạnh hơn nhiều: animate **bất kỳ thuộc tính nào** (vị trí, màu, độ trong suốt, bật/tắt hitbox, gọi hàm C#) trên timeline. Dùng cho đòn đánh: frame 3 bật hitbox, frame 6 tắt, frame 8 gọi hàm `OnAttackEnd`.

```csharp
_anim.Play("attack");
await ToSignal(_anim, AnimationPlayer.SignalName.AnimationFinished);
```

### 7.3 AnimationTree

Dùng khi có nhiều trạng thái chuyển qua lại (idle, run, jump, fall, attack). Tạo `AnimationTree` trỏ tới `AnimationPlayer`, dùng **StateMachine** hoặc **BlendSpace2D** (cho 4/8 hướng top-down):

```csharp
var state = (AnimationNodeStateMachinePlayback)_tree.Get("parameters/playback");
state.Travel("run");
_tree.Set("parameters/Walk/blend_position", direction);
```

### 7.4 Tween (hiệu ứng bằng code)

```csharp
var t = CreateTween();
t.TweenProperty(_sprite, "modulate:a", 0.0f, 0.4f);
t.TweenCallback(Callable.From(QueueFree));
```

Có `SetParallel()`, `SetEase()`, `SetTrans()`. Tween rất hợp cho UI, nhặt đồ, rung màn hình, nảy.

## 8. Signal và lập trình hướng sự kiện

Signal là cách các node **giao tiếp lỏng lẻo**: node phát tín hiệu, node khác lắng nghe, bên phát không cần biết bên nghe là ai. Quy tắc vàng: **gọi xuống (cha gọi hàm con), báo lên (con phát signal cho cha)**.

### 8.1 Khai báo và phát signal

```csharp
public partial class Health : Node
{
    [Signal] public delegate void HealthChangedEventHandler(int current, int max);
    [Signal] public delegate void DiedEventHandler();

    [Export] public int Max = 100;
    private int _current;

    public override void _Ready() => _current = Max;

    public void TakeDamage(int amount)
    {
        _current = Mathf.Max(0, _current - amount);
        EmitSignal(SignalName.HealthChanged, _current, Max);
        if (_current == 0) EmitSignal(SignalName.Died);
    }
}
```

Quy ước: delegate **phải** kết thúc bằng `EventHandler`; Godot sinh ra `SignalName.HealthChanged` và sự kiện `HealthChanged`.

### 8.2 Kết nối signal

```csharp
health.HealthChanged += OnHealthChanged;     // cách C# hiện đại
health.Died += OnDied;

// Node có sẵn signal của Godot cũng dùng cùng cách:
_timer.Timeout += OnTimeout;
_button.Pressed += OnPressed;
```

**Ngắt kết nối** khi node nghe có thể bị huỷ trước node phát, để tránh rò rỉ/lỗi:

```csharp
public override void _ExitTree()
{
    health.HealthChanged -= OnHealthChanged;
}
```

Có thể kết nối trong editor (tab Node > Signals), nhưng với C# thì kết nối bằng code dễ kiểm soát hơn.

### 8.3 await signal

```csharp
await ToSignal(GetTree().CreateTimer(1.5), SceneTreeTimer.SignalName.Timeout);
GD.Print("1.5 giây sau");
```

Sau `await`, node có thể đã bị xoá: kiểm tra `IsInstanceValid(this)` nếu có khả năng đó.

### 8.4 Event bus (signal toàn cục)

Với game lớn, tạo một Autoload `Events` chỉ chứa signal (`CoinCollected`, `PlayerDied`, `LevelCompleted`) để HUD, âm thanh, quest cùng nghe mà không phụ thuộc nhau.

**Bài tập:** tách máu thành component `Health`, dùng chung cho Player và Enemy, HUD nghe `HealthChanged` để cập nhật thanh máu.

## 9. Resource: dữ liệu và lưu game

### 9.1 Custom Resource (rất mạnh)

Dùng để định nghĩa dữ liệu game (vật phẩm, kẻ địch, kỹ năng) ngay trong Inspector, thay cho file JSON rời rạc.

```csharp
using Godot;

[GlobalClass]
public partial class ItemData : Resource
{
    [Export] public string Id;
    [Export] public string DisplayName;
    [Export] public Texture2D Icon;
    [Export] public int Price;
    [Export(PropertyHint.MultilineText)] public string Description;
}
```

Build xong, chuột phải trong FileSystem > New Resource > chọn `ItemData`, lưu thành `sword.tres`. Có thể `[Export] public ItemData Item;` ở bất kỳ node nào, hoặc mảng `[Export] public Godot.Collections.Array<ItemData> Items;`.

Lưu ý: Resource được **dùng chung** theo mặc định (nhiều node trỏ cùng một instance). Nếu cần dữ liệu riêng cho từng đối tượng (máu hiện tại), dùng `Duplicate()` hoặc tách dữ liệu tĩnh (Resource) khỏi trạng thái động (biến trong node).

### 9.2 Lưu và tải game

Đường dẫn `user://` là thư mục ghi được, riêng cho từng game/hệ điều hành. `res://` chỉ đọc sau khi export.

```csharp
using System.Text.Json;

public class SaveData
{
    public int Coins { get; set; }
    public float PlayerX { get; set; }
    public float PlayerY { get; set; }
    public int Version { get; set; } = 1;
}

public static class SaveSystem
{
    private const string Path = "user://save.json";

    public static void Save(SaveData data)
    {
        using var f = FileAccess.Open(Path, FileAccess.ModeFlags.Write);
        f.StoreString(JsonSerializer.Serialize(data));
    }

    public static SaveData Load()
    {
        if (!FileAccess.FileExists(Path)) return new SaveData();
        using var f = FileAccess.Open(Path, FileAccess.ModeFlags.Read);
        return JsonSerializer.Deserialize<SaveData>(f.GetAsText()) ?? new SaveData();
    }
}
```

Nên có trường `Version` để sau này đổi cấu trúc save mà không làm hỏng file cũ. Với C# thuần, `System.Text.Json` thuận tiện hơn `Json` của Godot.

## 10. Autoload, quản lý scene, group

### 10.1 Autoload (singleton)

Project > Project Settings > Globals (Autoload), thêm script/scene, đặt tên (ví dụ `GameManager`). Node này tồn tại suốt game, không bị mất khi đổi scene.

```csharp
public partial class GameManager : Node
{
    [Signal] public delegate void CoinsChangedEventHandler(int coins);

    public static GameManager Instance { get; private set; }
    public int Coins { get; private set; }

    public override void _EnterTree() => Instance = this;

    public void AddCoins(int n)
    {
        Coins += n;
        EmitSignal(SignalName.CoinsChanged, Coins);
    }
}
// dùng ở mọi nơi: GameManager.Instance.AddCoins(1);
```

Đừng lạm dụng autoload: chỉ cho thứ thật sự toàn cục (trạng thái game, âm thanh, event bus, save). Còn lại để trong scene.

### 10.2 Đổi scene

```csharp
GetTree().ChangeSceneToFile("res://scenes/Level2.tscn");
// hoặc
GetTree().ChangeSceneToPacked(_level2Scene);
```

Với màn hình loading hoặc màn chuyển cảnh mờ dần, tạo scene `SceneTransition` (autoload) dùng Tween/AnimationPlayer.

### 10.3 Group

Gắn nhãn cho node ("enemies", "interactable") rồi thao tác hàng loạt:

```csharp
AddToGroup("enemies");
foreach (Node n in GetTree().GetNodesInGroup("enemies")) { ... }
GetTree().CallGroup("enemies", "Alert");
```

## 11. UI với Control node

- Gốc UI là `CanvasLayer` (để UI không bị camera kéo đi) chứa các `Control`.
- **Container** tự xếp bố cục: `VBoxContainer`, `HBoxContainer`, `GridContainer`, `MarginContainer`, `PanelContainer`. Hạn chế đặt vị trí tay, hãy dùng container và anchor để UI co giãn theo độ phân giải.
- Node thường dùng: `Label`, `Button`, `TextureRect`, `ProgressBar`, `RichTextLabel` (BBCode), `ItemList`, `Tree`, `LineEdit`.
- **Theme**: tạo một file theme dùng cho cả game để đồng bộ font, màu, nút.
- **Focus và tay cầm:** đặt `FocusMode`, gọi `GrabFocus()` cho nút đầu tiên của menu.

```csharp
public partial class Hud : CanvasLayer
{
    [Export] private ProgressBar _hpBar;
    [Export] private Label _coinLabel;

    public override void _Ready()
    {
        GameManager.Instance.CoinsChanged += c => _coinLabel.Text = $"x {c}";
    }

    public void OnHealthChanged(int cur, int max)
    {
        _hpBar.MaxValue = max;
        _hpBar.Value = cur;
    }
}
```

Kéo thả (inventory): override `_GetDragData`, `_CanDropData`, `_DropData` trong một Control.

## 12. Audio

- `AudioStreamPlayer` (nhạc, âm UI), `AudioStreamPlayer2D` (âm thanh có vị trí, nhỏ dần theo khoảng cách).
- Dùng **Audio Bus** (panel Audio dưới editor): tạo bus Music, SFX, UI để chỉnh âm lượng riêng. Lưu ý đổi âm lượng theo thang dB: `AudioServer.SetBusVolumeDb(idx, Mathf.LinearToDb(value))`.
- Phát nhiều tiếng chồng nhau (tiếng bước chân, đạn): tạo player mới theo nhu cầu hoặc dùng pool, đừng dùng một player duy nhất vì nó cắt tiếng cũ.

## 13. Kiến trúc: composition và state machine

### 13.1 Composition thay vì kế thừa sâu

Thay vì `Enemy : Character : Entity : Base`, hãy ghép **component** là các node con: `Health`, `Hitbox`, `Hurtbox`, `Mover`, `Inventory`. Mỗi component nhỏ, có signal riêng, dùng lại được ở nhiều nơi. Đây là cách làm "đúng kiểu Godot".

### 13.2 State machine bằng node

```csharp
public partial class State : Node
{
    [Signal] public delegate void TransitionedEventHandler(string newState);
    public virtual void Enter() { }
    public virtual void Exit() { }
    public virtual void Update(double delta) { }
    public virtual void PhysicsUpdate(double delta) { }
}

public partial class StateMachine : Node
{
    [Export] public State Initial;
    private State _current;
    private readonly System.Collections.Generic.Dictionary<string, State> _states = new();

    public override void _Ready()
    {
        foreach (var child in GetChildren())
            if (child is State s)
            {
                _states[s.Name.ToString().ToLower()] = s;
                s.Transitioned += OnTransition;
            }
        _current = Initial;
        _current?.Enter();
    }

    public override void _Process(double d) => _current?.Update(d);
    public override void _PhysicsProcess(double d) => _current?.PhysicsUpdate(d);

    private void OnTransition(string name)
    {
        if (!_states.TryGetValue(name.ToLower(), out var next) || next == _current) return;
        _current?.Exit();
        _current = next;
        _current.Enter();
    }
}
```

Mỗi trạng thái (`Idle`, `Run`, `Jump`, `Attack`, `Hurt`) là một script kế thừa `State`. Dùng cho Player và AI kẻ địch.

### 13.3 Interface trong C#

C# có interface thật, rất hợp để quy ước hành vi:

```csharp
public interface IDamageable { void TakeDamage(int amount); }

if (body is IDamageable d) d.TakeDamage(10);
```

## 14. AI, đường đi và Navigation 2D

1. Thêm `NavigationRegion2D` vào level, tạo **NavigationPolygon**, vẽ vùng đi được (hoặc bake từ tilemap qua navigation layer của TileSet).
2. Gắn `NavigationAgent2D` vào kẻ địch.

```csharp
public partial class Chaser : CharacterBody2D
{
    [Export] public float Speed = 80f;
    [Export] private NavigationAgent2D _agent;
    public Node2D Target;

    public override void _PhysicsProcess(double delta)
    {
        if (Target == null) return;
        _agent.TargetPosition = Target.GlobalPosition;
        if (_agent.IsNavigationFinished()) return;

        Vector2 next = _agent.GetNextPathPosition();
        Velocity = GlobalPosition.DirectionTo(next) * Speed;
        MoveAndSlide();
    }
}
```

Tối ưu: đừng cập nhật `TargetPosition` mỗi frame cho hàng trăm kẻ địch, hãy dùng Timer cập nhật 4–5 lần/giây. Để tránh va nhau, bật `AvoidanceEnabled` và dùng signal `VelocityComputed`.

Các kỹ thuật AI thường gặp: tầm nhìn bằng `RayCast2D`/`Area2D`, tuần tra bằng điểm (`Marker2D`), state machine (tuần tra, đuổi, tấn công, bỏ chạy), và hành vi theo điểm số (utility AI) cho game phức tạp.

**Bài tập (Dự án tuần 3):** top-down RPG nhỏ: nhân vật 4 hướng với BlendSpace2D, NPC thoại bằng `RichTextLabel`, kẻ địch đuổi theo, inventory kéo thả, lưu/tải game.

## 15. Shader 2D cơ bản

Shader của Godot (giống GLSL) gắn vào `ShaderMaterial` của bất kỳ `CanvasItem`. Ví dụ hiệu ứng chớp trắng khi bị đánh:

```glsl
shader_type canvas_item;
uniform float flash : hint_range(0.0, 1.0) = 0.0;

void fragment() {
    vec4 c = texture(TEXTURE, UV);
    COLOR = vec4(mix(c.rgb, vec3(1.0), flash), c.a);
}
```

Điều khiển từ C#:

```csharp
((ShaderMaterial)_sprite.Material).SetShaderParameter("flash", 1.0f);
```

Các hiệu ứng hay dùng: viền nhân vật (outline), đổi màu palette, sóng nước, dissolve, rung, vùng sáng. Nhớ: mỗi sprite dùng chung một `ShaderMaterial` sẽ chia sẻ tham số; bật `Local to Scene` hoặc dùng `instance uniform` để mỗi sprite có giá trị riêng.

## 16. C# nâng cao trong Godot

### 16.1 Cách Godot và C# nói chuyện với nhau

Mỗi lần gọi API Godot từ C# (ví dụ `Position`, `GetNode`, `EmitSignal`) đều đi qua lớp interop với engine C++. Chi phí mỗi lần nhỏ nhưng **cộng dồn** khi gọi hàng chục nghìn lần mỗi frame. Hệ quả thực tế:

- Cache kết quả: `GetNode` một lần trong `_Ready`, không gọi trong `_Process`.
- Với logic nặng (mô phỏng, pathfinding tự viết, thuật toán lưới), tính toán trên **kiểu C# thuần** (struct, mảng, `System.Numerics`) rồi chỉ đẩy kết quả vào node một lần.
- Dùng `Godot.Collections.Array/Dictionary` khi cần đưa dữ liệu qua Godot (export, signal), còn nội bộ dùng `List<T>`, `Dictionary<K,V>` của .NET sẽ nhanh hơn.

### 16.2 Giảm rác bộ nhớ (GC)

GC của .NET có thể gây giật khi có nhiều cấp phát. Trong `_Process`/`_PhysicsProcess`:

- Tránh tạo chuỗi mới (`$"..."`, nối chuỗi) mỗi frame.
- Tránh LINQ và lambda tạo closure trong đường nóng.
- Dùng `StringName` được cache sẵn cho tên action/animation/signal gọi thường xuyên: `private static readonly StringName JumpAction = "jump";`
- Dùng **object pooling** cho đạn, hiệu ứng, kẻ địch sinh liên tục: tái sử dụng node thay vì `Instantiate`/`QueueFree` liên tục.
- Dùng `struct`, `Span<T>`, `stackalloc` cho dữ liệu tạm nhỏ.

### 16.3 Vòng đời đối tượng Godot

- Node bị `QueueFree()` thì đối tượng C# vẫn còn nhưng phần engine đã mất. Truy cập sẽ ném `ObjectDisposedException`. Kiểm tra bằng `GodotObject.IsInstanceValid(obj)`.
- Đối tượng `RefCounted`/`Resource` được đếm tham chiếu, tự dọn. Còn `Node` phải được thêm vào cây hoặc tự `QueueFree`, nếu không sẽ rò rỉ bộ nhớ (orphan node). Dùng Debugger > Monitors > Orphan Nodes để phát hiện.
- Event C# thường (`event Action`) do bạn tự quản: ngắt kết nối ở `_ExitTree` để tránh giữ tham chiếu.

### 16.4 async/await đúng cách

Godot cung cấp `SynchronizationContext` để chạy tiếp ở luồng chính sau `await`. Dùng `await ToSignal(...)` cho thời gian, animation. Tránh để `Task.Run` chạm vào node (chỉ luồng chính được dùng API Godot). Muốn làm việc nặng nền (tải file, tính toán):

```csharp
var result = await Task.Run(() => HeavyCompute());   // luồng nền, không đụng node
_label.Text = result.ToString();                     // đã quay về luồng chính
```

Nếu cần gọi API Godot từ luồng khác, dùng `CallDeferred` hoặc `Callable.From(...).CallDeferred()`.

### 16.5 Generics, extension method, pattern matching

```csharp
public static class NodeExt
{
    public static T Spawn<T>(this Node parent, PackedScene scene, Vector2 pos) where T : Node2D
    {
        var n = scene.Instantiate<T>();
        n.GlobalPosition = pos;
        parent.AddChild(n);
        return n;
    }
}
```

Các công cụ C# đáng dùng: `enum` + `switch` expression cho state, `record`/`readonly struct` cho dữ liệu bất biến, pattern matching (`if (body is Enemy { Health: > 0 } e)`), nullable reference types (`#nullable enable`) để bắt lỗi null sớm.

### 16.6 Tổ chức mã nguồn lớn

- Tách **logic thuần C#** (luật game, tính sát thương, kinh tế) khỏi **node Godot**. Phần thuần dễ viết unit test bằng xUnit/NUnit mà không cần mở engine.
- Dùng namespace theo tính năng (`MyGame.Combat`, `MyGame.Inventory`).
- Phụ thuộc: dùng `[Export]` và signal thay cho đường dẫn cứng; dịch vụ toàn cục dùng autoload.
- Thêm package NuGet qua `dotnet add package`, nhưng ưu tiên thư viện không phụ thuộc nền tảng nếu muốn xuất ra mobile.

### 16.7 Tool script và plugin

Đặt `[Tool]` lên class để script chạy ngay trong editor (vẽ gizmo, tự sinh dữ liệu, preview). Có thể viết **EditorPlugin** để thêm dock hoặc import plugin riêng cho pipeline asset. Dùng `Engine.IsEditorHint()` để phân biệt đang chạy trong editor hay trong game.

### 16.8 GDExtension

Khi cần hiệu năng tối đa ở một mảng cụ thể (mô phỏng lớn, thuật toán nặng), có thể viết GDExtension bằng C++/Rust. Hầu hết game 2D không cần tới bước này, hãy đo bằng profiler trước.

## 17. Debug, profiling, kiểm thử

- `GD.Print()`, `GD.PrintErr()`, `GD.PushWarning()`, `GD.PushError()`. Trong Rider có thể gắn debugger vào Godot và đặt breakpoint bình thường.
- **Remote Scene Tree** (khi game chạy): xem và sửa thuộc tính node trực tiếp.
- **Debugger > Profiler / Monitors:** FPS, thời gian process, số draw call, bộ nhớ, orphan node, số node.
- Bật **Visible Collision Shapes** và **Visible Navigation** trong menu Debug để thấy va chạm và đường đi.
- Vẽ debug: dùng `_Draw()` + `QueueRedraw()` để vẽ đường tầm nhìn, hitbox lúc phát triển.
- Kiểm thử: dùng GdUnit4 (hỗ trợ C#) hoặc tách logic thuần để test bằng xUnit.

Quy trình tối ưu: **đo trước, tối ưu sau**. Tìm nút thắt bằng Profiler rồi mới sửa; thường là quá nhiều node, vòng lặp nặng trong `_Process`, hoặc quá nhiều draw call.

## 18. Xuất bản (Export)

1. Project > Export > Add preset (Windows, Linux, Android, Web...).
2. Cài **Export Templates** (Editor > Manage Export Templates).
3. Windows/Linux: xuất ra file chạy được kèm thư mục dữ liệu .NET.
4. **Android:** cần JDK, Android SDK, tạo keystore. Hỗ trợ C# trên Android có từ Godot 4.2 nhưng trạng thái và hạn chế có thể thay đổi giữa các bản, hãy đọc trang *Exporting for Android* trong tài liệu chính thức của đúng phiên bản bạn dùng.
5. **iOS và Web:** hỗ trợ C# **kém hơn** nhiều so với GDScript tuỳ phiên bản (đặc biệt Web). Nếu nhắm iOS/Web, hãy kiểm tra tài liệu và danh sách hạn chế trước khi chọn C# cho cả dự án; đây là rủi ro lớn nhất của việc dùng C# trong Godot.
6. Đặt icon, tên, orientation, độ phân giải trong Project Settings trước khi xuất.

## 19. Dự án thực hành và lộ trình

**Tuần 1: Platformer cơ bản.** Player (chạy, nhảy, coyote time), tilemap có va chạm, xu, gai, reset màn.

**Tuần 2: Hoàn thiện platformer.** Component `Health`, kẻ địch tuần tra, đòn đánh bằng AnimationPlayer + hitbox, HUD, hiệu ứng chớp trắng bằng shader, camera smoothing, nhiều màn bằng đổi scene, âm thanh.

**Tuần 3: Top-down RPG nhỏ.** Di chuyển 4 hướng + BlendSpace2D, Y-sort, NPC thoại, kẻ địch dùng NavigationAgent2D, inventory bằng Resource (`ItemData`), lưu/tải game.

**Tuần 4: Đóng gói.** Dọn kiến trúc (state machine, event bus), profiler, object pooling nếu cần, menu chính/pause/cài đặt âm thanh, xuất Windows và Android, thử trên máy thật.

Sau 4 tuần, bước tiếp theo: làm một game nhỏ hoàn chỉnh trong 2–4 tuần (có menu, âm thanh, màn kết thúc) và đưa lên itch.itch.io để quen quy trình hoàn thiện sản phẩm.

## 20. Lỗi thường gặp (checklist)

- Sửa code C# mà Inspector không cập nhật: chưa **build**.
- Class thiếu `partial` hoặc tên file khác tên class: script không gắn được vào node.
- `[Export]` biến không hiện: biến phải `public` hoặc có `[Export]` với kiểu Godot hỗ trợ.
- `Velocity.Y = ...` báo lỗi: Vector2 là struct, copy ra biến rồi gán lại.
- `GetNode` trả null: sai đường dẫn hoặc gọi trước `_Ready`; ưu tiên `[Export]`.
- Nhân vật xuyên tường: logic di chuyển đặt ở `_Process` thay vì `_PhysicsProcess`, hoặc sai layer/mask.
- Lỗi "Can't change state while flushing queries": đổi physics trong callback, dùng `CallDeferred`/`SetDeferred`.
- Truy cập node đã bị `QueueFree`: dùng `IsInstanceValid`, ngắt event khi `_ExitTree`.
- Pixel art bị mờ: đặt Texture Filter = Nearest và Stretch Mode phù hợp.
- Giật hình do GC: giảm cấp phát trong `_Process`, dùng pooling và `StringName` cache.

## 21. Nguồn học nên dùng

- **Tài liệu chính thức:** docs.godotengine.org, đặc biệt các mục *Getting started*, *Scripting > C#*, *2D*, *Physics*, *Animation*, *Navigation*. Luôn chọn đúng phiên bản bạn đang dùng.
- Mục **C# basics** và **C# API differences to GDScript** trong tài liệu: đọc kỹ để dịch tutorial GDScript sang C#.
- Demo project chính thức: kho `godot-demo-projects` trên GitHub.
- Cộng đồng: Godot Discord, r/godot, Godot Forum; kênh YouTube như GDQuest, Brackeys, HeartBeast (nhiều video dùng GDScript nhưng khái niệm áp dụng được cho C#).
- Đọc mã nguồn các game mẫu nhỏ, bắt chước rồi tự sửa là cách học nhanh nhất.

---

Ghi chú: Godot thay đổi API giữa các bản 4.x (ví dụ `TileMap` chuyển sang `TileMapLayer` ở 4.3). Khi một đoạn code báo lỗi, đối chiếu với tài liệu của đúng phiên bản bạn cài.

---

# PHẦN II. BẢN CHI TIẾT CHO NGƯỜI MỚI HOÀN TOÀN

Phần I ở trên là tài liệu tra cứu theo chủ đề. Phần II này đi theo kiểu **làm theo từng bước**, dành cho người chưa quen lập trình hoặc mới làm game lần đầu. Thứ tự đề xuất:

1. **Phụ lục A** (C# từ con số 0): học song song với mục 1–3 của Phần I.
2. **Phụ lục B** (dự án 1 từng bước): làm theo trong tuần 1–2, tra Phần I khi cần giải thích sâu.
3. **Phụ lục C** (dự án 2: top-down RPG): tuần 3.
4. **Phụ lục D** (quy trình làm dự án thật, Git, kế hoạch): đọc trước khi bắt đầu dự án riêng của bạn.
5. **Phụ lục E** (cheat sheet, lộ trình 28 ngày, từ điển, FAQ): để bên cạnh khi học.

Nguyên tắc: **gõ lại code bằng tay, chạy thử sau mỗi bước nhỏ, đừng viết 200 dòng rồi mới chạy**. Khi lỗi, đọc kỹ thông báo lỗi trước (xem A.14).

## Phụ lục A. C# từ con số 0 (ví dụ theo ngữ cảnh game)

### A.1 Chương trình C# là gì?

Code C# là các file `.cs` chứa **class** (khuôn mẫu), trong class có **method** (hàm, việc cần làm) và **field/property** (dữ liệu). Trong Godot, mỗi script gắn vào một node là một class kế thừa class của node đó. Godot tự gọi các hàm đặc biệt như `_Ready()`, `_Process()` vào đúng lúc; bạn chỉ cần viết nội dung.

```csharp
using Godot;                       // dùng thư viện Godot

public partial class Player : Node2D   // class tên Player, kế thừa Node2D
{
    public override void _Ready()      // Godot gọi hàm này khi node sẵn sàng
    {
        GD.Print("Xin chào!");         // in ra cửa sổ Output
    }
}
```

Mỗi câu lệnh kết thúc bằng dấu `;`. Khối code nằm trong `{ }`. C# **phân biệt hoa thường**: `Player` khác `player`.

### A.2 Biến và kiểu dữ liệu

Biến là cái hộp có tên để chứa giá trị. Mỗi biến có một **kiểu**:

| Kiểu | Chứa gì | Ví dụ |
| --- | --- | --- |
| `int` | số nguyên | `int coins = 5;` |
| `float` | số thực (nhỏ gọn, Godot dùng nhiều) | `float speed = 200.5f;` (nhớ chữ `f`) |
| `double` | số thực độ chính xác cao | `double delta` (Godot truyền vào `_Process`) |
| `bool` | đúng/sai | `bool isDead = false;` |
| `string` | chuỗi chữ | `string name = "Slime";` |
| `Vector2` | cặp số x, y (vị trí, hướng) | `new Vector2(10, 20)` |
| `Color` | màu | `Colors.Red` hoặc `new Color(1, 0, 0)` |

`var` để compiler tự suy ra kiểu: `var hp = 100;` (hiểu là int). Nên dùng khi kiểu hiển nhiên.

Quy ước đặt tên: biến cục bộ `camelCase`, field private `_camelCase`, class/method/property `PascalCase`, hằng `PascalCase` hoặc `UPPER_CASE`.

```csharp
int hp = 100;
float speed = 150f;
string title = "Hero";
GD.Print($"{title} có {hp} máu, tốc độ {speed}");   // chuỗi nội suy
```

### A.3 Toán tử và ép kiểu

```csharp
int a = 7 / 2;        // = 3 (chia hai số nguyên thì ra số nguyên!)
float b = 7 / 2f;     // = 3.5
int c = 7 % 2;        // = 1 (phần dư)
hp -= 10;             // hp = hp - 10
hp++;                 // tăng 1
float dt = (float)delta;   // ép double sang float
```

Hai bẫy hay gặp: chia số nguyên cho kết quả nguyên; trộn `float` và `double` phải ép kiểu (Godot trả `delta` kiểu `double`, còn `Vector2` dùng `float`).

### A.4 Điều kiện

```csharp
if (hp <= 0)
{
    GD.Print("Chết");
}
else if (hp < 30)
{
    GD.Print("Nguy hiểm");
}
else
{
    GD.Print("Ổn");
}

// phép so sánh: == != < > <= >=     logic: && (và)  || (hoặc)  ! (không)
if (isGrounded && Input.IsActionJustPressed("jump")) { ... }

string label = hp > 0 ? "Sống" : "Chết";     // toán tử ba ngôi
```

`switch` khi có nhiều trường hợp:

```csharp
switch (state)
{
    case State.Idle: break;
    case State.Run:  break;
    default: break;
}
```

### A.5 Vòng lặp

```csharp
for (int i = 0; i < 5; i++) { GD.Print(i); }          // 0..4

foreach (var enemy in enemies) { enemy.Alert(); }    // duyệt từng phần tử

int n = 3;
while (n > 0) { n--; }                               // lặp khi điều kiện đúng
```

**Cảnh báo:** không bao giờ viết vòng lặp vô hạn (`while (true)`) trong `_Process`; game sẽ treo. Muốn chờ, dùng timer hoặc `await`.

### A.6 Hàm (method)

```csharp
private int Cong(int a, int b)        // nhận 2 số nguyên, trả về int
{
    return a + b;
}

private void TakeDamage(int amount, bool knockback = false)   // tham số mặc định
{
    hp -= amount;
}

private bool IsAlive() => hp > 0;     // viết gọn một dòng

int kq = Cong(2, 3);                  // gọi hàm
```

`void` nghĩa là không trả về gì. `private` chỉ class này dùng được, `public` cho class khác gọi. Một hàm nên **làm đúng một việc** và tên hàm nên là động từ (`TakeDamage`, `SpawnCoin`).

### A.7 Class và đối tượng

Class là bản thiết kế, đối tượng (object) là sản phẩm tạo từ bản thiết kế đó.

```csharp
public class EnemyStats
{
    public string Name;                    // field
    public int MaxHp { get; set; } = 10;   // property có giá trị mặc định
    public int Hp { get; private set; }    // đọc công khai, chỉ ghi nội bộ

    public EnemyStats(string name, int maxHp)   // constructor: chạy khi tạo
    {
        Name = name;
        MaxHp = maxHp;
        Hp = maxHp;
    }

    public void Hurt(int dmg) => Hp = System.Math.Max(0, Hp - dmg);
}

var slime = new EnemyStats("Slime", 20);
slime.Hurt(5);
GD.Print(slime.Hp);   // 15
```

Lưu ý: lớp kế thừa node Godot (Node, Resource...) **không** tạo bằng `new` mà dùng `Instantiate()` hoặc đặt sẵn trong scene. Class C# thuần (như `EnemyStats` ở trên) thì dùng `new` bình thường.

### A.8 Tập hợp dữ liệu

```csharp
int[] diem = new int[3] { 10, 20, 30 };          // mảng cố định
var ten = new System.Collections.Generic.List<string>();   // danh sách co giãn
ten.Add("An"); ten.Add("Bình");
ten.Remove("An");
GD.Print(ten.Count);

var gia = new System.Collections.Generic.Dictionary<string, int>   // từ điển: khoá -> giá trị
{
    ["kiem"] = 100,
    ["khien"] = 80,
};
if (gia.TryGetValue("kiem", out int g)) GD.Print(g);
```

Trong game thường dùng `List<T>` cho danh sách (kẻ địch, vật phẩm), `Dictionary` để tra theo id. Thêm `using System.Collections.Generic;` ở đầu file để khỏi gõ dài.

### A.9 enum: tập giá trị có tên

```csharp
public enum EnemyState { Patrol, Chase, Attack, Dead }

private EnemyState _state = EnemyState.Patrol;

switch (_state)
{
    case EnemyState.Patrol: /* đi tuần */ break;
    case EnemyState.Chase:  /* đuổi */ break;
}
```

Dùng enum thay cho số/chuỗi "ma thuật" (như `state == 2`) để code dễ đọc và trình biên dịch bắt lỗi chính tả giúp bạn.

### A.10 null và lỗi NullReferenceException

`null` nghĩa là "chưa có gì". Gọi hàm trên một biến `null` sẽ gây lỗi **NullReferenceException**, lỗi phổ biến nhất của người mới.

```csharp
if (_target != null) _target.TakeDamage(1);     // kiểm tra trước
_target?.TakeDamage(1);                          // gọi nếu không null
var name = _target?.Name ?? "không có";          // ?? = giá trị dự phòng
```

Nguyên nhân hay gặp trong Godot: quên kéo node vào ô `[Export]` trong Inspector; `GetNode` sai đường dẫn; dùng node sau khi đã `QueueFree`.

### A.11 Kế thừa, override, interface

```csharp
public partial class Animal : Node2D
{
    public virtual void Speak() => GD.Print("...");   // cho phép lớp con ghi đè
}

public partial class Dog : Animal
{
    public override void Speak() => GD.Print("Gâu!");
}

public interface IInteractable { void Interact(Node2D who); }   // hợp đồng hành vi

public partial class Chest : Area2D, IInteractable
{
    public void Interact(Node2D who) => GD.Print("Mở rương");
}
```

`_Ready()` của Godot cũng là hàm `virtual` mà bạn `override`. Interface giúp "mọi thứ tương tác được" dùng chung một cách gọi mà không cần chung lớp cha.

### A.12 static, const, readonly

```csharp
public const int MaxLevel = 50;                  // hằng, không bao giờ đổi
private readonly float _gravity = 980f;          // gán một lần
public static int TotalKills;                    // thuộc về class, dùng chung toàn bộ
Player.TotalKills++;
```

### A.13 namespace và using

`using Godot;` cho phép dùng `Node2D`, `Vector2`, `GD` mà không gõ `Godot.` phía trước. `namespace MyGame.Combat { ... }` nhóm các class lại cho gọn khi dự án lớn. Với người mới, chưa cần dùng namespace ngay.

### A.14 Đọc thông báo lỗi (kỹ năng quan trọng nhất)

Có hai loại lỗi:

- **Lỗi biên dịch** (compile error): xuất hiện ở tab Build/Problems của Rider hoặc ô Output của Godot khi build. Có dạng `Player.cs(12,5): error CS0103: The name 'speeed' does not exist`. Đọc ba thứ: **file**, **dòng/cột**, **nội dung**. Lỗi `CS0103` = gõ sai tên biến; `CS1002` = thiếu `;`; `CS0246` = thiếu `using` hoặc sai tên kiểu; `CS0029` = gán sai kiểu (ví dụ gán `double` vào `float`).
- **Lỗi lúc chạy** (runtime error): hiện chữ đỏ trong tab Debugger của Godot, kèm **stack trace** (chuỗi các hàm đã gọi). Dòng đầu tiên thuộc code của bạn cho biết chỗ lỗi. Lỗi thường gặp: `NullReferenceException`, `ObjectDisposedException`, `InvalidCastException`.

Cách xử lý: đọc lỗi từ trên xuống, sửa **lỗi đầu tiên** rồi build lại (các lỗi sau thường chỉ là hệ quả). Chưa rõ thì dùng `GD.Print` in giá trị biến ra, hoặc đặt breakpoint trong Rider và chạy debug.

### A.15 Bài tập C# (làm trong Godot, in kết quả ra Output)

1. Khai báo 3 biến lưu tên, máu, tốc độ nhân vật rồi in ra một câu hoàn chỉnh.
2. Viết hàm `bool IsAlive(int hp)` và dùng `if` để in "Sống" hoặc "Chết".
3. Dùng `for` in bảng cửu chương 7.
4. Tạo `List<string>` các vật phẩm, thêm 3 món, xoá 1 món, in số lượng còn lại bằng `foreach`.
5. Tạo `Dictionary<string,int>` giá vật phẩm và hàm trả giá theo tên (xử lý trường hợp không có).
6. Tạo `enum Weather { Sunny, Rainy, Snowy }` và dùng `switch` in lời chào theo thời tiết.
7. Viết class `Weapon` có `Name`, `Damage` và hàm `Attack()` in ra sát thương.
8. Viết class `Bow : Weapon` ghi đè `Attack()` để in kiểu khác.
9. Cố tình tạo lỗi `NullReferenceException`, đọc stack trace, sửa lại bằng `?.`.
10. Viết script gắn vào `Sprite2D`: mỗi frame xoay sprite theo `Rotation += 2f * (float)delta;`, nhấn phím cách thì đảo chiều xoay.

Làm xong 10 bài này là đủ nền tảng C# để bắt đầu dự án 1. Phần còn lại sẽ học dần khi gặp trong dự án.

## Phụ lục B. Dự án 1: Platformer, làm từng bước

Mục tiêu: một game platformer có nhân vật chạy/nhảy, xu, kẻ địch, máu, nhiều màn, menu, và xuất ra file chạy được. Mỗi bước kết thúc bằng **kiểm tra** để bạn biết đã đúng chưa. Đừng sang bước sau khi bước trước còn lỗi.

### B.0 Tạo project

1. Mở Godot (bản .NET) > **Create** project. Đặt tên `Platformer`, chọn thư mục rỗng.
2. Renderer chọn **Compatibility** (nhẹ, chạy tốt trên máy yếu và mobile; game 2D không cần cái nặng hơn).
3. Bật **Version Control Metadata: Git** nếu có.
4. Vào Project > Tools > C# > **Create C# Solution** (nếu chưa có). Godot tạo file `.csproj` và `.sln`.
5. Tạo thư mục `scenes/`, `scripts/`, `assets/` trong FileSystem.

**Kiểm tra:** thấy file `Platformer.csproj` trong thư mục project và mở được bằng Rider.

### B.1 Cài đặt project cho game 2D

Project > Project Settings:

- **Display > Window:** Viewport Width 640, Height 360 (nếu làm pixel art 16x16, viewport nhỏ rồi scale lên đẹp hơn), Window Width/Height Override 1280x720. **Stretch > Mode** = `canvas_items`, **Aspect** = `keep`.
- **Rendering > Textures > Canvas Textures > Default Texture Filter** = `Nearest` (pixel art không bị mờ).
- **General > Layer Names > 2D Physics:** đặt 1=`world`, 2=`player`, 3=`enemy`, 4=`pickup`, 5=`hazard`.
- **Input Map:** thêm `move_left` (A, mũi tên trái), `move_right` (D, mũi tên phải), `jump` (Space, W), `pause` (Esc).

### B.2 Tìm asset miễn phí

Kenney.nl (gói "Pixel Platformer", giấy phép CC0, dùng thoải mái kể cả thương mại), itch.io mục *free game assets*, OpenGameArt.org. Kiểm tra giấy phép từng gói và ghi nguồn vào file `CREDITS.txt`. Kéo thả file `.png` vào `assets/` trong Godot. Chọn từng ảnh, tab Import > Preset = `2D Pixel` rồi **Reimport**.

### B.3 Scene Player

1. Scene > New Scene > chọn **Other Node** > `CharacterBody2D`, đổi tên `Player`, lưu `scenes/Player.tscn`.
2. Thêm con: `AnimatedSprite2D`, `CollisionShape2D` (Shape: `CapsuleShape2D`, vừa với thân nhân vật), `Camera2D`.
3. Với `AnimatedSprite2D`: Sprite Frames > New SpriteFrames, thêm animation `idle`, `run`, `jump`, `fall`, kéo các frame vào. Đặt FPS khoảng 8–12.
4. Camera2D: bật **Position Smoothing**.
5. Player > Inspector > Collision: Layer = `player`, Mask = `world`.
6. Chuột phải Player > Attach Script, ngôn ngữ C#, tên `Player`, đặt trong `scripts/`.

Code đầy đủ có coyote time, jump buffer, nhảy biến thiên, máu và knockback:

```csharp
using Godot;

public partial class Player : CharacterBody2D
{
    [Signal] public delegate void HealthChangedEventHandler(int current, int max);
    [Signal] public delegate void DiedEventHandler();

    [ExportGroup("Di chuyển")]
    [Export] public float Speed = 150f;
    [Export] public float Accel = 1200f;
    [Export] public float Friction = 1400f;
    [Export] public float JumpVelocity = -330f;
    [Export] public float CoyoteTime = 0.1f;
    [Export] public float JumpBufferTime = 0.1f;

    [ExportGroup("Máu")]
    [Export] public int MaxHealth = 3;
    [Export] public float InvincibleTime = 1.0f;

    [Export] private AnimatedSprite2D _sprite;

    private float _gravity = ProjectSettings.GetSetting("physics/2d/default_gravity").AsSingle();
    private float _coyoteTimer;
    private float _jumpBufferTimer;
    private float _invincibleTimer;
    private int _health;
    private bool _dead;

    public override void _Ready()
    {
        _health = MaxHealth;
        EmitSignal(SignalName.HealthChanged, _health, MaxHealth);
    }

    public override void _PhysicsProcess(double delta)
    {
        if (_dead) return;
        float dt = (float)delta;
        Vector2 v = Velocity;

        // 1. Trọng lực (rơi nhanh hơn khi đang đi xuống)
        if (!IsOnFloor())
            v.Y += _gravity * dt * (v.Y > 0 ? 1.5f : 1f);

        // 2. Bộ đếm coyote + jump buffer
        _coyoteTimer = IsOnFloor() ? CoyoteTime : _coyoteTimer - dt;
        _jumpBufferTimer = Input.IsActionJustPressed("jump") ? JumpBufferTime : _jumpBufferTimer - dt;

        // 3. Nhảy
        if (_jumpBufferTimer > 0 && _coyoteTimer > 0)
        {
            v.Y = JumpVelocity;
            _jumpBufferTimer = 0;
            _coyoteTimer = 0;
        }
        // Thả phím sớm thì nhảy thấp hơn
        if (Input.IsActionJustReleased("jump") && v.Y < 0)
            v.Y *= 0.5f;

        // 4. Chạy ngang có gia tốc / ma sát
        float dir = Input.GetAxis("move_left", "move_right");
        if (dir != 0)
            v.X = Mathf.MoveToward(v.X, dir * Speed, Accel * dt);
        else
            v.X = Mathf.MoveToward(v.X, 0, Friction * dt);

        Velocity = v;
        MoveAndSlide();

        // 5. Hoạt ảnh
        if (dir != 0) _sprite.FlipH = dir < 0;
        UpdateAnimation(dir);

        // 6. Nhấp nháy khi bất tử tạm
        if (_invincibleTimer > 0)
        {
            _invincibleTimer -= dt;
            _sprite.Visible = ((int)(_invincibleTimer * 15)) % 2 == 0;
            if (_invincibleTimer <= 0) _sprite.Visible = true;
        }
    }

    private void UpdateAnimation(float dir)
    {
        string anim;
        if (!IsOnFloor()) anim = Velocity.Y < 0 ? "jump" : "fall";
        else anim = Mathf.Abs(Velocity.X) > 5f ? "run" : "idle";
        if (_sprite.Animation != anim) _sprite.Play(anim);
    }

    public void Bounce() => Velocity = new Vector2(Velocity.X, JumpVelocity * 0.7f);

    public void Hurt(int amount, Vector2 fromPos)
    {
        if (_invincibleTimer > 0 || _dead) return;
        _health = Mathf.Max(0, _health - amount);
        _invincibleTimer = InvincibleTime;
        float side = GlobalPosition.X < fromPos.X ? -1 : 1;
        Velocity = new Vector2(side * 160f, -200f);          // đẩy lùi
        EmitSignal(SignalName.HealthChanged, _health, MaxHealth);
        if (_health == 0) Die();
    }

    public void Die()
    {
        if (_dead) return;
        _dead = true;
        EmitSignal(SignalName.Died);
    }
}
```

**Kiểm tra:** đặt Player trong một scene tạm với `StaticBody2D` làm sàn, bấm F6 (chạy scene hiện tại): nhân vật chạy, nhảy được, nhảy thấp khi thả phím sớm, không bị kẹt trên sàn.

Nếu nhân vật không hiện animation: kiểm tra đã kéo `AnimatedSprite2D` vào ô `Sprite` trong Inspector chưa (đã build chưa).

### B.4 Level và Tilemap

1. Scene mới với gốc `Node2D` tên `Level1`, lưu `scenes/Level1.tscn`.
2. Thêm `TileMapLayer` tên `Ground`. Inspector > Tile Set > New TileSet. Tile Size = kích thước tile của asset (ví dụ 18x18 với Kenney Pixel Platformer; xem tài liệu gói).
3. Mở panel TileSet (dưới cùng), kéo ảnh tileset vào, đồng ý tự tạo tile.
4. Trong TileSet thêm **Physics Layer** (Collision Layer = `world`). Chuyển sang chế độ Paint, chọn Physics Layer 0, vẽ hình vuông phủ tile để tile cứng.
5. Mở panel TileMap, chọn tile và vẽ mặt đất, bục, tường.
6. Kéo `Player.tscn` vào Level1, đặt chỗ bắt đầu.
7. Thêm `Marker2D` tên `Spawn` nếu muốn hồi sinh về điểm chuẩn.
8. Project Settings > Application > Run > Main Scene = `Level1.tscn`.

**Kiểm tra:** bấm F5, chạy và nhảy trên các bục không xuyên tường, camera đi theo mượt.

### B.5 Đồng xu và GameManager

**GameManager (Autoload):** tạo `scripts/GameManager.cs` như mục 10.1 của Phần I, thêm vào Project Settings > Globals > Autoload với tên `GameManager`. Thêm tiếp:

```csharp
public void ResetRun() { Coins = 0; EmitSignal(SignalName.CoinsChanged, Coins); }
```

**Coin.tscn:** gốc `Area2D`, con `AnimatedSprite2D` (animation xoay), `CollisionShape2D` (CircleShape2D). Collision Layer = `pickup`, Mask = `player`.

```csharp
using Godot;

public partial class Coin : Area2D
{
    [Export] private AudioStreamPlayer2D _sfx;
    private bool _taken;

    public override void _Ready() => BodyEntered += OnBodyEntered;

    private void OnBodyEntered(Node2D body)
    {
        if (_taken || body is not Player) return;
        _taken = true;
        GameManager.Instance.AddCoins(1);
        _sfx?.Play();
        Hide();
        // chờ tiếng kêu phát xong rồi mới xoá
        _sfx.Finished += QueueFree;
    }
}
```

Kéo nhiều Coin vào Level1. **Kiểm tra:** chạm xu thì biến mất và `GameManager.Coins` tăng (in thử bằng `GD.Print`).

### B.6 HUD (hiển thị máu và xu)

Scene `HUD.tscn`: gốc `CanvasLayer`, con `MarginContainer` > `HBoxContainer` chứa `Label` (tên `CoinLabel`) và `HBoxContainer` (tên `Hearts`, các `TextureRect` trái tim). Script:

```csharp
using Godot;

public partial class Hud : CanvasLayer
{
    [Export] private Label _coinLabel;
    [Export] private Control _hearts;

    public override void _Ready()
    {
        GameManager.Instance.CoinsChanged += OnCoinsChanged;
        OnCoinsChanged(GameManager.Instance.Coins);
    }

    public override void _ExitTree() => GameManager.Instance.CoinsChanged -= OnCoinsChanged;

    private void OnCoinsChanged(int c) => _coinLabel.Text = $"x {c}";

    public void OnPlayerHealthChanged(int cur, int max)
    {
        for (int i = 0; i < _hearts.GetChildCount(); i++)
            ((CanvasItem)_hearts.GetChild(i)).Visible = i < cur;
    }
}
```

Thêm `HUD` vào Level1, rồi trong Level1 nối signal `HealthChanged` của Player tới `OnPlayerHealthChanged` của HUD (có thể nối bằng code trong script `Level1.cs`, xem B.9).

### B.7 Gai và vùng rơi chết

```csharp
public partial class Hazard : Area2D
{
    [Export] public int Damage = 1;
    [Export] public bool Instakill = false;

    public override void _Ready() => BodyEntered += OnBodyEntered;

    private void OnBodyEntered(Node2D body)
    {
        if (body is not Player p) return;
        if (Instakill) p.Die();
        else p.Hurt(Damage, GlobalPosition);
    }
}
```

Dùng một `Hazard` với `Instakill = true` đặt rộng phía dưới màn làm "vùng rơi chết". Collision Mask của Hazard = `player`.

### B.8 Kẻ địch Slime (đi tuần, bị giẫm chết)

Scene `Slime.tscn`: gốc `CharacterBody2D` (Layer = `enemy`, Mask = `world`), con `AnimatedSprite2D`, `CollisionShape2D`, `RayCast2D` tên `LedgeCheck` (đặt ở mép trước, Target Position hướng xuống 12px, bật Enabled, Mask = `world`), và `Area2D` tên `Hitbox` (Layer = `enemy`, Mask = `player`) có `CollisionShape2D` riêng.

```csharp
using Godot;

public partial class Slime : CharacterBody2D
{
    [Export] public float Speed = 35f;
    [Export] private AnimatedSprite2D _sprite;
    [Export] private RayCast2D _ledgeCheck;
    [Export] private Area2D _hitbox;

    private float _gravity = ProjectSettings.GetSetting("physics/2d/default_gravity").AsSingle();
    private int _dir = -1;
    private bool _dead;

    public override void _Ready()
    {
        _hitbox.BodyEntered += OnHitboxBodyEntered;
        ApplyDirection();
    }

    public override void _PhysicsProcess(double delta)
    {
        if (_dead) return;
        Vector2 v = Velocity;
        if (!IsOnFloor()) v.Y += _gravity * (float)delta;

        if (IsOnWall() || (IsOnFloor() && !_ledgeCheck.IsColliding()))
        {
            _dir *= -1;
            ApplyDirection();
        }
        v.X = _dir * Speed;
        Velocity = v;
        MoveAndSlide();
    }

    private void ApplyDirection()
    {
        _sprite.FlipH = _dir > 0;
        var p = _ledgeCheck.Position;
        _ledgeCheck.Position = new Vector2(Mathf.Abs(p.X) * _dir, p.Y);
    }

    private void OnHitboxBodyEntered(Node2D body)
    {
        if (_dead || body is not Player p) return;
        bool stomped = p.Velocity.Y > 0 && p.GlobalPosition.Y < GlobalPosition.Y - 4;
        if (stomped) { p.Bounce(); Die(); }
        else p.Hurt(1, GlobalPosition);
    }

    private void Die()
    {
        _dead = true;
        _hitbox.SetDeferred(Area2D.PropertyName.Monitoring, false);
        var t = CreateTween();
        t.TweenProperty(_sprite, "modulate:a", 0f, 0.3f);
        t.TweenCallback(Callable.From(QueueFree));
    }
}
```

Nếu Slime đứng yên hoặc quay liên tục: kiểm tra RayCast2D có `Enabled`, hướng xuống và Mask đúng layer `world`; kiểm tra Slime nằm trên sàn chứ không lơ lửng.

**Kiểm tra:** Slime đi qua lại giữa hai mép bục, giẫm lên đầu thì chết và bạn nảy lên, chạm cạnh thì mất máu và bị đẩy lùi.

### B.9 Gắn mọi thứ trong Level1

```csharp
using Godot;

public partial class Level1 : Node2D
{
    [Export] private Player _player;
    [Export] private Hud _hud;
    [Export] public string NextScenePath = "res://scenes/Level2.tscn";

    public override void _Ready()
    {
        GameManager.Instance.ResetRun();
        _player.HealthChanged += _hud.OnPlayerHealthChanged;
        _player.Died += OnPlayerDied;
    }

    private async void OnPlayerDied()
    {
        await ToSignal(GetTree().CreateTimer(1.0), SceneTreeTimer.SignalName.Timeout);
        GetTree().ReloadCurrentScene();
    }

    public void GoNext() =>
        GetTree().CallDeferred(SceneTree.MethodName.ChangeSceneToFile, NextScenePath);
}
```

**Cửa/cờ kết thúc màn:** một `Area2D` tên `Goal`, khi `BodyEntered` là Player thì gọi `GetParent<Level1>().GoNext()` (hoặc phát signal). Dùng `CallDeferred` như trên để tránh đổi scene giữa lúc physics đang xử lý.

Copy `Level1.tscn` thành `Level2.tscn` (chuột phải > Duplicate), vẽ lại map, đổi `NextScenePath`.

### B.10 Menu chính và Pause

**MainMenu.tscn:** gốc `Control` (Full Rect), con `VBoxContainer` căn giữa chứa 3 `Button`: Play, Settings, Quit.

```csharp
public partial class MainMenu : Control
{
    [Export] private Button _play;
    [Export] private Button _quit;

    public override void _Ready()
    {
        _play.Pressed += () => GetTree().ChangeSceneToFile("res://scenes/Level1.tscn");
        _quit.Pressed += () => GetTree().Quit();
        _play.GrabFocus();
    }
}
```

Đặt MainMenu làm Main Scene. **Pause:** scene `PauseMenu.tscn` (CanvasLayer, đặt `Process Mode` = `When Paused`, ban đầu ẩn). Trong `_UnhandledInput`, khi nhấn `pause` thì đảo `GetTree().Paused` và `Visible`.

### B.11 Âm thanh và "juice"

- Thêm `AudioStreamPlayer` nhạc nền (Autoload Music để không bị ngắt khi đổi màn), lặp lại bằng cách bật Loop ở file `.ogg`.
- Tiếng nhảy, nhặt xu, bị đánh: `AudioStreamPlayer2D` gọi `.Play()` ở đúng chỗ.
- Hạt bụi khi chạy/đáp đất: `GPUParticles2D` hoặc `CPUParticles2D` (CPU nhẹ hơn trên máy yếu).
- Rung màn hình khi bị đánh: Tween thuộc tính `Offset` của Camera2D vài khung hình.
- Đừng thêm hiệu ứng trước khi game chơi vui; làm sau cùng.

### B.12 Xuất bản bản chơi thử

Làm theo mục 18 của Phần I. Với lần đầu, chỉ xuất Windows trước (ổn định nhất với C#), nhờ một người bạn chạy thử và quan sát họ chơi (không nói gì, chỉ nhìn họ vướng ở đâu).

### B.13 Checklist hoàn thành dự án 1

- Chạy/nhảy mượt, có coyote time và jump buffer.
- Ít nhất 2 màn, chuyển màn bằng cờ.
- Xu, gai, 1 loại kẻ địch, hệ thống máu, chết và hồi sinh.
- HUD, menu chính, pause.
- Có nhạc và tiếng động.
- Xuất ra file chạy được và ít nhất một người khác đã chơi thử.
- Code được commit lên Git (xem Phụ lục D).

## Phụ lục C. Dự án 2: Top-down RPG nhỏ, làm từng bước

Mục tiêu: nhân vật đi 4 hướng, thế giới có Y-sort, NPC để nói chuyện, rương và vật phẩm, inventory, kẻ địch đuổi theo, lưu/tải game. Dự án này dạy những kỹ năng nền cho hầu hết game 2D kiểu nhập vai, nông trại, xây dựng, chiến thuật.

### C.1 Chuẩn bị

Project mới `TopDownRPG` (Compatibility, Create C# Solution). Input Map: `left`, `right`, `up`, `down` (WASD + mũi tên), `interact` (E, Enter), `pause`. Layer 2D Physics: 1 `world`, 2 `player`, 3 `enemy`, 4 `interactable`. Asset gợi ý: Kenney "Tiny Town" hoặc "Tiny Dungeon" (CC0).

### C.2 Nhân vật 4 hướng với AnimationTree

Scene `Player.tscn`: `CharacterBody2D` > `AnimatedSprite2D` hoặc `Sprite2D` + `AnimationPlayer`, `CollisionShape2D` nhỏ ở **chân** (không phủ cả thân, để đi sau cây trông tự nhiên), `Camera2D`, và `Area2D` tên `InteractZone`.

Cách đơn giản nhất cho người mới: dùng `AnimatedSprite2D` với các animation `idle_down`, `idle_up`, `idle_side`, `walk_down`, `walk_up`, `walk_side` (lật `FlipH` cho hướng trái). Khi quen rồi mới chuyển sang AnimationTree + BlendSpace2D như mục 7.3.

```csharp
using Godot;

public partial class TopDownPlayer : CharacterBody2D
{
    [Export] public float Speed = 90f;
    [Export] private AnimatedSprite2D _sprite;
    [Export] private Area2D _interactZone;

    private Vector2 _facing = Vector2.Down;
    public bool InputLocked { get; set; }      // khoá khi đang thoại / mở menu

    public override void _PhysicsProcess(double delta)
    {
        Vector2 dir = InputLocked ? Vector2.Zero : Input.GetVector("left", "right", "up", "down");
        Velocity = dir * Speed;
        MoveAndSlide();

        if (dir != Vector2.Zero)
        {
            _facing = dir;
            _interactZone.Position = _facing.Normalized() * 14f;   // vùng tương tác luôn ở phía trước mặt
        }
        Animate(dir);
    }

    private void Animate(Vector2 dir)
    {
        string kind = dir == Vector2.Zero ? "idle" : "walk";
        string side;
        if (Mathf.Abs(_facing.X) > Mathf.Abs(_facing.Y)) { side = "side"; _sprite.FlipH = _facing.X < 0; }
        else side = _facing.Y < 0 ? "up" : "down";
        string anim = $"{kind}_{side}";
        if (_sprite.Animation != anim) _sprite.Play(anim);
    }

    public override void _UnhandledInput(InputEvent e)
    {
        if (InputLocked || !e.IsActionPressed("interact")) return;
        foreach (var area in _interactZone.GetOverlappingAreas())
            if (area is IInteractable it) { it.Interact(this); break; }
    }
}
```

Lưu ý: `Input.GetVector` đã chuẩn hoá, đi chéo không nhanh hơn. `InteractZone` có Collision Mask = `interactable`, Layer trống.

### C.3 Thế giới và Y-sort

Scene `World.tscn`: `Node2D` tên `World` (bật **Y Sort Enabled**) > `TileMapLayer` nền (không cần y-sort), nhóm `Node2D` tên `Entities` (bật Y Sort Enabled) chứa Player, NPC, cây, nhà. Điểm gốc (origin) của mỗi sprite đặt ở **chân** vật thể để sắp xếp đúng. Với tilemap có vật thể cao (cây, nhà) bật Y Sort cho layer đó và đặt Y Sort Origin phù hợp.

**Kiểm tra:** đi vòng quanh cái cây: đi phía sau thì bị che, đi phía trước thì che cây.

### C.4 Hệ thống tương tác (interface)

```csharp
using Godot;

public interface IInteractable
{
    void Interact(TopDownPlayer player);
}
```

Mọi thứ tương tác được (NPC, rương, bảng chỉ đường, cửa) implement interface này, người chơi không cần biết cụ thể là gì.

### C.5 Dữ liệu thoại bằng Resource

```csharp
using Godot;

[GlobalClass]
public partial class DialogueData : Resource
{
    [Export] public string Speaker;
    [Export(PropertyHint.MultilineText)] public string[] Lines;
}
```

Tạo các file `.tres` (ví dụ `elder_intro.tres`) trong Inspector, viết lời thoại ngay trong editor, không cần sửa code mỗi lần đổi chữ.

**NPC:**

```csharp
public partial class Npc : Area2D, IInteractable
{
    [Export] public DialogueData Dialogue;

    public void Interact(TopDownPlayer player)
    {
        DialogueBox.Instance.Show(Dialogue, player);
    }
}
```

NPC là `Area2D` (Layer = `interactable`) kèm `CollisionShape2D` (vùng nhận tương tác), có thể kèm `StaticBody2D` con để chặn đường đi.

### C.6 Hộp thoại có hiệu ứng gõ chữ

Scene `DialogueBox.tscn`: `CanvasLayer` > `PanelContainer` (neo cạnh dưới) > `VBoxContainer` > `Label` tên người nói + `RichTextLabel` nội dung (bật `Fit Content`, ban đầu để `Visible Characters Behavior` mặc định). Đặt scene này làm Autoload hoặc thêm vào level.

```csharp
using Godot;

public partial class DialogueBox : CanvasLayer
{
    public static DialogueBox Instance { get; private set; }

    [Export] private Control _panel;
    [Export] private Label _name;
    [Export] private RichTextLabel _text;

    private DialogueData _data;
    private int _index;
    private TopDownPlayer _player;
    private Tween _tween;

    public override void _EnterTree() => Instance = this;
    public override void _Ready() => _panel.Visible = false;

    public void Show(DialogueData data, TopDownPlayer player)
    {
        if (_panel.Visible) return;
        _data = data; _index = 0; _player = player;
        _player.InputLocked = true;
        _panel.Visible = true;
        _name.Text = data.Speaker;
        ShowLine();
    }

    private void ShowLine()
    {
        _text.Text = _data.Lines[_index];
        _text.VisibleRatio = 0f;
        _tween?.Kill();
        _tween = CreateTween();
        float time = _text.Text.Length * 0.03f;
        _tween.TweenProperty(_text, "visible_ratio", 1f, time);
    }

    public override void _UnhandledInput(InputEvent e)
    {
        if (!_panel.Visible || !e.IsActionPressed("interact")) return;
        GetViewport().SetInputAsHandled();

        if (_text.VisibleRatio < 1f)            // đang gõ dở thì hiện hết
        {
            _tween?.Kill();
            _text.VisibleRatio = 1f;
            return;
        }
        _index++;
        if (_index >= _data.Lines.Length) Close();
        else ShowLine();
    }

    private void Close()
    {
        _panel.Visible = false;
        _player.InputLocked = false;
    }
}
```

Vì `InputLocked` đang bật nên người chơi không thể bấm `interact` để mở lại hộp thoại cùng lúc; hộp thoại tự xử lý phím `interact` ở trên.

### C.7 Vật phẩm và inventory

**ItemData** (Resource): xem mục 9.1 Phần I, thêm `[Export] public bool Stackable = true; [Export] public int MaxStack = 99;`.

**Inventory** (Node, thêm làm Autoload hoặc con của Player):

```csharp
using Godot;
using System.Collections.Generic;

public partial class Inventory : Node
{
    [Signal] public delegate void ChangedEventHandler();

    public static Inventory Instance { get; private set; }
    public int Capacity = 20;

    private readonly List<(ItemData item, int count)> _slots = new();
    public IReadOnlyList<(ItemData item, int count)> Slots => _slots;

    public override void _EnterTree() => Instance = this;

    public bool Add(ItemData item, int amount = 1)
    {
        // cộng dồn vào ô đã có
        if (item.Stackable)
            for (int i = 0; i < _slots.Count; i++)
                if (_slots[i].item == item)
                {
                    int total = _slots[i].count + amount;
                    if (total <= item.MaxStack)
                    {
                        _slots[i] = (item, total);
                        EmitSignal(SignalName.Changed);
                        return true;
                    }
                }
        if (_slots.Count >= Capacity) return false;     // túi đầy
        _slots.Add((item, amount));
        EmitSignal(SignalName.Changed);
        return true;
    }

    public bool Remove(ItemData item, int amount = 1)
    {
        for (int i = 0; i < _slots.Count; i++)
        {
            if (_slots[i].item != item) continue;
            int left = _slots[i].count - amount;
            if (left < 0) return false;
            if (left == 0) _slots.RemoveAt(i); else _slots[i] = (item, left);
            EmitSignal(SignalName.Changed);
            return true;
        }
        return false;
    }
}
```

**Rương** (ví dụ dùng interface):

```csharp
public partial class Chest : Area2D, IInteractable
{
    [Export] public ItemData Item;
    [Export] public int Amount = 1;
    private bool _opened;

    public void Interact(TopDownPlayer player)
    {
        if (_opened) return;
        if (Inventory.Instance.Add(Item, Amount))
        {
            _opened = true;
            // đổi sprite thành rương mở ...
        }
    }
}
```

**UI túi đồ:** scene `InventoryUI` gồm `GridContainer` (columns = 5). Khi nhận signal `Changed`, xoá hết slot cũ rồi tạo lại từ `Slots` (đơn giản và đủ nhanh cho vài chục ô). Mỗi slot là một scene nhỏ có `TextureRect` icon và `Label` số lượng. Bật/tắt bằng phím `I`.

### C.8 Kẻ địch đuổi theo

Dùng code Chaser ở mục 14 Phần I. Bổ sung: `Area2D` làm "tầm nhìn" (bán kính 80px), khi Player vào tầm thì đặt `Target`, khi ra khỏi tầm thì bỏ `Target` và quay về điểm xuất phát. Dùng timer 0.25s để cập nhật đường đi thay vì mỗi frame.

### C.9 Lưu/tải game

Dùng `SaveSystem` mục 9.2 Phần I. Dữ liệu cần lưu tối thiểu: vị trí người chơi, scene hiện tại, nội dung inventory (lưu `Id` của từng vật phẩm và số lượng, **không** lưu đối tượng Resource). Khi tải, dùng `Id` tra lại `ItemData` từ một `ItemDatabase` (Resource chứa danh sách mọi item). Luôn có trường `Version` trong file save.

### C.10 Checklist hoàn thành dự án 2

- Đi 4 hướng, animation đúng hướng, Y-sort đúng.
- Nói chuyện được với ít nhất 2 NPC, thoại lấy từ Resource.
- Nhặt đồ từ rương, mở túi đồ xem được.
- Một kẻ địch đuổi và gây sát thương.
- Lưu rồi tắt game, mở lại tải đúng trạng thái.

## Phụ lục D. Từ học đến làm dự án thật

### D.1 Quy trình một dự án game nhỏ

1. **Chốt ý tưởng một câu.** Ví dụ: "Game nông trại 2D, người chơi trồng cây, bán nông sản và mở rộng đất". Không viết được trong một câu là ý tưởng còn quá rộng.
2. **Viết GDD ngắn (1–2 trang):** thể loại, vòng lặp chính (người chơi làm gì lặp đi lặp lại trong 30 giây, 5 phút, 1 giờ), hệ thống cần có, những thứ **chưa** làm (cực quan trọng).
3. **Tách thành "phiên bản dọc" (vertical slice):** một đoạn nhỏ hoàn chỉnh của game (một khu vực, đủ cơ chế chính, đủ âm thanh và đồ hoạ cơ bản). Làm bản này trước toàn bộ game.
4. **Prototype xấu trước, đẹp sau:** dùng hình vuông màu làm placeholder cho đến khi cơ chế chơi vui.
5. **Chia việc thành các task nhỏ** (làm xong trong 1–2 ngày), ghi vào một danh sách (Notion, Trello, hoặc file `TODO.md`).
6. **Chơi thử và sửa theo phản hồi** mỗi tuần.
7. **Đóng băng tính năng** trước khi phát hành: chỉ sửa lỗi, không thêm mới.

### D.2 Quy tắc phạm vi (scope) cho người làm đơn lẻ

- Game đầu tiên hoàn chỉnh nên chơi được trong **10–20 phút**.
- Mỗi tính năng bạn thêm nhân lên số lỗi và công việc. Hãy hỏi "không có nó game có chơi được không?". Nếu có, bỏ nó vào danh sách "làm sau".
- Tránh sớm: multiplayer, hệ thống crafting phức tạp, thế giới sinh ngẫu nhiên khổng lồ, hệ thống nhiệm vụ nhiều nhánh. Đây là những thứ làm các dự án đầu tay chết yểu.
- Làm cái chơi được trước, rồi mở rộng nội dung (thêm màn, vật phẩm, kẻ địch) bằng cách dùng lại hệ thống đã có (Resource là bạn của bạn ở đây).

### D.3 Từ ý tưởng đến hệ thống: bản đồ tính năng và mục cần học

| Tính năng game hay gặp | Mục cần xem trong tài liệu |
| --- | --- |
| Nhân vật di chuyển, nhảy, va chạm | 5, B.3 |
| Bản đồ ô lưới, đặt/xoá công trình, trồng cây | 6 (đọc/ghi tile, custom data), 9, C.7 |
| Camera, kéo/zoom bản đồ | 6 (Camera2D), 4 (chuột) |
| Kẻ địch, AI, đuổi/tuần tra | 13, 14, B.8, C.8 |
| Chiến đấu theo lượt hoặc chiến thuật (nhiều đơn vị, chỉ huy) | 13 (state machine), 9 (Resource cho đơn vị), 10 (group), 16.6 (tách logic thuần) |
| Kho đồ, vật phẩm, cửa hàng | 9, C.7, 11 |
| Hội thoại, nhiệm vụ, cốt truyện | C.5, C.6, 8.4 (event bus) |
| Nuôi thú, tăng trưởng theo thời gian | 13 (state machine), 9, 8 (signal), `Timer` |
| Lưu/tải, tiến trình | 9.2, C.9 |
| UI, menu, cài đặt | 11, B.10 |
| Hiệu ứng, âm thanh | 7.4, 12, 15, B.11 |
| Xuất ra điện thoại | 18, 17 |

Khi bạn có ý tưởng game riêng, gạch từng dòng tính năng vào bảng này để biết cần học mục nào trước và cái nào có thể để sau.

### D.4 Cấu trúc thư mục khuyến nghị

```
res://
  scenes/
    characters/   Player.tscn, Enemy.tscn
    levels/       Level1.tscn
    ui/           HUD.tscn, MainMenu.tscn
    objects/      Coin.tscn, Chest.tscn
  scripts/        (cùng cấu trúc con với scenes/ hoặc đặt cạnh scene)
  resources/      items/, dialogues/, enemies/
  assets/         sprites/, audio/music, audio/sfx, fonts/
  autoload/       GameManager.cs, Inventory.cs, SaveSystem.cs
```

Quy ước: tên file và class dùng PascalCase; tên scene trùng tên script gốc (`Player.tscn` + `Player.cs`); gom asset theo loại, đặt tên có tiền tố rõ ràng (`spr_player_idle`, `sfx_coin`).

### D.5 Git: lưu lịch sử và không bao giờ mất code

Cài Git, tạo repository (GitHub hoặc GitLab, bản private miễn phí). Điều quan trọng với Godot .NET là file `.gitignore`:

```
.godot/
.mono/
data_*/
bin/
obj/
*.translation
.idea/
.vs/
export_presets.cfg   # tuỳ bạn, file này có thể chứa thông tin riêng (keystore)
```

(Khi tạo project, Godot cũng đề nghị sinh `.gitignore` tự động; hãy bật tuỳ chọn Git khi tạo project.) Quy trình tối thiểu:

```
git add .
git commit -m "Thêm Slime biết đi tuần"
git push
```

Commit **thường xuyên** (mỗi tính năng nhỏ chạy được), đặt thông điệp commit mô tả việc đã làm. Trước khi thử một thay đổi lớn, tạo nhánh (`git switch -c thu-nghiem-ai`) để có thể bỏ nếu hỏng. Đừng commit file asset khổng lồ chưa nén; dùng Git LFS cho file nặng (như bản nhạc dài, ảnh lớn).

### D.6 Thói quen giúp dự án sống lâu

- Chạy game **mỗi 10–15 phút** khi code, không để dồn lỗi.
- Mỗi tính năng làm xong, dọn code và đặt tên cho rõ trước khi qua việc kế tiếp.
- Ghi chú ngắn trong `README.md`: cách chạy, các quy ước, những quyết định lớn.
- Giữ một danh sách lỗi đã biết, sửa theo mức ưu tiên (game crash > mất save > sai logic > hình ảnh xấu).
- Sao lưu: ngoài Git, có một bản sao ở nơi khác (ổ ngoài, cloud) cho thư mục project.
- Nghỉ ngơi: dự án đầu tay thường mất 3–6 tháng, tốc độ bền quan trọng hơn bứt phá.

### D.7 Khi bị kẹt

1. Đọc lại thông báo lỗi và tìm đúng dòng (A.14).
2. Thu nhỏ vấn đề: tạo scene thử nghiệm riêng chỉ có phần bị lỗi.
3. Tra tài liệu chính thức đúng phiên bản; tìm bằng từ khoá tiếng Anh ("Godot 4 C# CharacterBody2D slide").
4. Hỏi ở Godot Discord/Forum/Reddit: nêu rõ **phiên bản Godot**, **đoạn code**, **thông báo lỗi** và **điều bạn mong đợi**.
5. Nếu hỏi AI, dán đoạn code và lỗi đầy đủ, nói rõ đang dùng Godot 4.x + C#; luôn kiểm tra kết quả chạy thật vì API Godot đổi giữa các phiên bản.

## Phụ lục E. Cheat sheet, lộ trình 28 ngày, từ điển, FAQ

### E.1 Cheat sheet API hay dùng (C#)

| Việc cần làm | Code |
| --- | --- |
| In ra Output | `GD.Print("x");` |
| Lấy node con | `GetNode<Sprite2D>("Sprite2D")` |
| Lấy cha | `GetParent<Node2D>()` |
| Thêm node | `AddChild(node);` |
| Xoá node | `QueueFree();` |
| Tạo từ scene | `GD.Load<PackedScene>("res://x.tscn").Instantiate<T>()` |
| Đổi scene | `GetTree().ChangeSceneToFile("res://x.tscn");` |
| Tải lại scene | `GetTree().ReloadCurrentScene();` |
| Tạm dừng | `GetTree().Paused = true;` |
| Thoát game | `GetTree().Quit();` |
| Đang giữ phím | `Input.IsActionPressed("a")` |
| Vừa nhấn | `Input.IsActionJustPressed("a")` |
| Trục -1..1 | `Input.GetAxis("l", "r")` |
| Hướng 2D | `Input.GetVector("l", "r", "u", "d")` |
| Vị trí chuột (thế giới) | `GetGlobalMousePosition()` |
| Hướng tới điểm | `GlobalPosition.DirectionTo(p)` |
| Khoảng cách | `GlobalPosition.DistanceTo(p)` |
| Kẹp giá trị | `Mathf.Clamp(x, min, max)` |
| Nội suy | `Mathf.Lerp(a, b, t)` |
| Tiến dần tới | `Mathf.MoveToward(a, b, step)` |
| Số ngẫu nhiên | `GD.Randf()`, `GD.RandRange(1, 6)` |
| Hẹn giờ một lần | `await ToSignal(GetTree().CreateTimer(1.0), "timeout");` |
| Tween | `CreateTween().TweenProperty(node, "modulate:a", 0f, 0.5f);` |
| Gọi muộn | `CallDeferred(MethodName.Foo);` |
| Còn sống? | `GodotObject.IsInstanceValid(node)` |
| Nhóm | `AddToGroup("g");` `GetTree().GetNodesInGroup("g")` |
| Chạy trong editor? | `Engine.IsEditorHint()` |

### E.2 Lộ trình 28 ngày (khoảng 1,5–3 giờ mỗi ngày)

**Tuần 1: Nền tảng và Platformer cơ bản**

- Ngày 1: Cài đặt (mục 1), tạo project đầu tiên, chạy `Hello` (A.1).
- Ngày 2: Node/Scene (mục 2), C# biến, điều kiện (A.2–A.4), bài tập A.15 số 1–3.
- Ngày 3: Vòng lặp, hàm, class (A.5–A.7), bài tập 4–7. Input (mục 4).
- Ngày 4: Physics (mục 5), làm Player (B.3).
- Ngày 5: Level và tilemap (B.4).
- Ngày 6: Collection, enum, null (A.8–A.10), bài tập 8–10; xu và GameManager (B.5).
- Ngày 7: Ôn lại, sửa lỗi, commit Git, nghỉ.

**Tuần 2: Hoàn thiện Platformer**

- Ngày 8: Signal (mục 8), HUD (B.6).
- Ngày 9: Gai, vùng rơi chết (B.7), Resource (mục 9).
- Ngày 10: Kẻ địch Slime (B.8).
- Ngày 11: Animation (mục 7), gắn vào Level, chuyển màn (B.9).
- Ngày 12: Màn 2, menu, pause (B.10).
- Ngày 13: Âm thanh, juice (B.11, mục 12).
- Ngày 14: Xuất bản, cho người khác chơi thử (B.12), nhận phản hồi.

**Tuần 3: Top-down RPG**

- Ngày 15: Autoload, quản lý scene (mục 10), nhân vật 4 hướng (C.2).
- Ngày 16: Y-sort, tilemap thế giới (C.3, mục 6).
- Ngày 17: Interface, NPC, thoại (C.4–C.6).
- Ngày 18: UI (mục 11), hộp thoại hoàn chỉnh.
- Ngày 19: Inventory và rương (C.7).
- Ngày 20: Kẻ địch đuổi theo, Navigation (C.8, mục 14).
- Ngày 21: Lưu/tải (C.9), ôn lại.

**Tuần 4: Kiến trúc, tối ưu, xuất bản**

- Ngày 22: State machine, composition (mục 13), refactor dự án 2.
- Ngày 23: Shader cơ bản (mục 15), hiệu ứng chớp trắng.
- Ngày 24: C# nâng cao: GC, async, tách logic (mục 16).
- Ngày 25: Debug, Profiler (mục 17); tìm và sửa một điểm nghẽn thật.
- Ngày 26: Export Windows và Android (mục 18).
- Ngày 27: Viết GDD một trang cho dự án riêng, lập bản đồ tính năng (D.1–D.3).
- Ngày 28: Tạo repository Git cho dự án riêng, dựng khung thư mục (D.4–D.5) và làm prototype nhân vật + camera.

Nếu một mục khó, ở lại thêm 1–2 ngày không sao; hiểu chắc quan trọng hơn đúng lịch.

### E.3 Từ điển thuật ngữ

- **Node:** thành phần cơ bản của cây cảnh, mỗi node làm một việc.
- **Scene:** cây node lưu thành file `.tscn`, tái sử dụng được.
- **SceneTree:** cây tất cả node đang chạy.
- **Signal:** thông báo một node phát ra để node khác lắng nghe.
- **Autoload:** node/script toàn cục luôn tồn tại (singleton).
- **Resource:** dữ liệu dùng chung lưu trong file (`.tres`).
- **Inspector:** bảng bên phải để chỉnh thuộc tính node.
- **Export:** (1) biến hiện trong Inspector bằng `[Export]`; (2) xuất game thành file chạy.
- **Delta:** thời gian giữa hai frame, dùng để di chuyển không phụ thuộc FPS.
- **Physics layer/mask:** layer là "tôi là ai", mask là "tôi nhìn thấy ai".
- **Tilemap/Tileset:** bản đồ ô lưới và bộ ô ảnh dùng để vẽ.
- **Y-sort:** sắp xếp thứ tự vẽ theo toạ độ Y để tạo cảm giác chiều sâu.
- **State machine:** mô hình các trạng thái và chuyển giữa chúng.
- **GC:** bộ dọn rác bộ nhớ của .NET.
- **Prototype / Vertical slice:** bản thử nghiệm thô / một đoạn game hoàn chỉnh nhỏ.
- **GDD:** tài liệu thiết kế game.

### E.4 FAQ và lỗi thường gặp của người mới

**Tôi sửa code nhưng game không đổi?** Chưa build; bấm Build (Ctrl+B) hoặc chạy lại từ Rider. Nếu Godot đang mở nhiều bản, đóng bản cũ.

**Script không gắn được vào node?** Tên file khác tên class, class thiếu `partial`, hoặc build còn lỗi. Sửa lỗi biên dịch trước.

**Biến `[Export]` hiện ra nhưng giá trị `null` khi chạy?** Bạn chưa kéo node/resource vào ô đó trong Inspector. Mở scene và kiểm tra từng ô.

**Nhân vật rung hoặc xuyên tường?** Đặt logic di chuyển trong `_PhysicsProcess`, dùng `MoveAndSlide`, kiểm tra layer/mask và hình va chạm.

**`Can't change state while flushing queries`?** Đang đổi trạng thái physics trong callback va chạm: dùng `CallDeferred`, `SetDeferred`, hoặc `ChangeSceneToFile` qua `CallDeferred`.

**Pixel art bị mờ?** Texture Filter = Nearest ở Project Settings và import preset `2D Pixel`.

**Chữ tiếng Việt bị lỗi dấu?** Dùng font hỗ trợ Unicode tiếng Việt (ví dụ Noto Sans, Roboto, hoặc font pixel có hỗ trợ). Kiểm tra bộ ký tự khi chọn font.

**Game chạy trong editor được nhưng export thì lỗi?** Đa số do đường dẫn file (`res://` phân biệt hoa thường trên Android/Linux) hoặc file chỉ dùng qua tên chuỗi nên không được đóng gói. Kiểm tra tab Resources của export preset và dùng đúng hoa thường.

**GDScript hay C#?** C# phù hợp nếu bạn quen hệ sinh thái .NET, thích kiểu tĩnh và công cụ mạnh như Rider. Đổi lại, tài liệu/ví dụ cộng đồng nhiều hơn ở GDScript và hỗ trợ export web/iOS của C# yếu hơn (xem mục 18). Có thể dùng cả hai trong một project nếu cần.

**Bao lâu thì làm được game đầu?** Một game nhỏ 10–20 phút thường mất 1–3 tháng cho người mới. Làm nhiều game nhỏ rồi mới tăng quy mô là con đường nhanh nhất.

---

Chúc bạn học vui. Khi làm xong hai dự án mini và lập xong GDD cho dự án riêng, bạn đã sẵn sàng bắt tay vào game của mình.

---

# PHẦN III. GAME CHIẾN THUẬT DÀN QUÂN, 2D NHƯNG NHÌN NHƯ 3D

Phần này áp dụng kiến thức Phần I và II vào thể loại: **chỉ huy bố trận trước, rồi chỉ đạo các stack lính khi giao tranh**, quân số lớn, sa bàn lớn, có địa hình, địch chạy theo kịch bản. Làm trên PC. Nên hoàn thành Phụ lục B và C trước, vì phần này dùng lại Resource, signal, state machine, Y-sort và UI.

## Phụ lục F. Nhìn như 3D nhưng vẫn là 2D

### F.1 Nguyên tắc cốt lõi: tách "logic" và "hiển thị"

Mọi game 2D trông như 3D đều dựa vào một ý: **trận đánh được mô phỏng trên mặt phẳng phẳng (logic), còn màn hình chỉ là cách ta "chiếu" mặt phẳng đó lên**. Cụ thể:

- Mỗi lính có `Ground` (vị trí trên mặt đất, `Vector2` phẳng) và `Height` (độ cao do địa hình).
- AI, va chạm, khoảng cách, tầm bắn, tìm đường, tất cả tính trên `Ground`.
- Chỉ khi vẽ mới đổi `Ground` + `Height` thành vị trí trên màn hình bằng một hàm chiếu.

Lợi ích: bạn đổi kiểu nhìn (từ nhìn chéo sang isometric) mà **không phải viết lại AI hay chiến đấu**; độ cao địa hình chỉ ảnh hưởng hình ảnh và vài chỉ số (cao điểm đánh mạnh hơn), không làm lộn xộn thuật toán.

Lỗi người mới hay mắc: lấy luôn `Position` của node làm vị trí logic. Khi đó mọi phép tính khoảng cách đều bị méo theo góc nhìn.

### F.2 Chọn kiểu nhìn (quyết định sớm vì ảnh hưởng toàn bộ art)

| Kiểu | Mô tả | Ưu | Nhược |
| --- | --- | --- | --- |
| **Top-down thuần** | Nhìn thẳng từ trên xuống, chỉ thấy đầu và vai | Dễ nhất | Ít cảm giác 3D, khó nhận ra loại lính |
| **Nhìn chéo 3/4** (kiểu Zelda/Pokémon cũ) | Camera nghiêng, thấy mặt trước và trên đầu nhân vật, mặt đất vẫn là lưới vuông | Dễ làm, đọc rõ loại lính, tilemap vuông thường | Cảm giác 3D vừa phải |
| **Isometric (2:1)** | Mặt đất là hình thoi, camera nghiêng \~30° | Trông "3D" rõ nhất, sa bàn đẹp | Cần art 4–8 hướng, toạ độ phức tạp hơn, chọn ô bằng chuột phải đổi công thức |

Gợi ý cho game quân số lớn của bạn: **bắt đầu bằng nhìn chéo 3/4**, vì lính nhỏ, nhiều, cần dễ nhìn và dễ làm art. Nhờ nguyên tắc F.1, sau này nâng lên isometric chỉ cần đổi hàm chiếu và tilemap. Nếu từ đầu đã chắc muốn isometric thì làm luôn, nhưng hãy thử cả hai bằng prototype nhỏ trước.

### F.3 Hàm chiếu và cấu trúc node của một lính

```csharp
using Godot;

public static class View
{
    // --- Nhìn chéo 3/4: mặt đất vẽ nguyên vẹn, YScale < 1 nếu muốn dẹt bớt ---
    public const float YScale = 1.0f;
    public static Vector2 ToScreen(Vector2 ground) => new(ground.X, ground.Y * YScale);
    public static Vector2 ToGround(Vector2 screen) => new(screen.X, screen.Y / YScale);

    // --- Isometric 2:1 (ô logic 16 đơn vị -> hình thoi 32x16 px) ---
    public static Vector2 IsoToScreen(Vector2 g) => new(g.X - g.Y, (g.X + g.Y) * 0.5f);
    public static Vector2 IsoToGround(Vector2 s) => new(s.Y + s.X * 0.5f, s.Y - s.X * 0.5f);
}
```

Để chuyển sang isometric chỉ cần thay `ToScreen`/`ToGround` bằng bản `Iso`. Toàn bộ code chiến đấu giữ nguyên.

Cấu trúc node của một lính (scene `Soldier.tscn`):

```
Soldier (Node2D)         <- Position = ToScreen(Ground), CHÂN lính nằm đúng điểm này
  Shadow (Sprite2D)      <- bóng, luôn nằm trên mặt đất (Position = 0,0)
  Body (AnimatedSprite2D)<- Position = (0, -độ cao), gốc sprite ở chân
```

Mẹo quan trọng: **Y-sort dùng `Position.Y` của node gốc**, nên gốc phải ở mặt đất. Độ cao chỉ dịch `Body` lên. Nếu bạn dịch cả node gốc lên, lính đứng trên đồi sẽ bị sắp xếp chồng sai với lính dưới chân đồi. Node cha `Entities` bật **Y Sort Enabled** (xem C.3).

### F.4 Tilemap sa bàn

**Nhìn chéo 3/4:** dùng `TileMapLayer` ô vuông bình thường (16x16 hoặc 32x32), vẽ tile theo phong cách nghiêng (đỉnh vách đá, mép nước, cây có thân thấy rõ). Toạ độ logic: `Ground = tile * 16`.

**Isometric:** trong TileSet đặt **Tile Shape = Isometric**, **Tile Layout = Diamond Down**, **Tile Size = 32 x 16** (hoặc 64 x 32 nếu muốn chi tiết hơn; khi đó nhân hằng số trong `IsoToScreen` cho khớp). `MapToLocal` và `LocalToMap` của `TileMapLayer` hoạt động với isometric, nên chọn ô bằng chuột:

```csharp
Vector2I cell = _ground.LocalToMap(_ground.ToLocal(GetGlobalMousePosition()));
```

Với bố cục 32x16 như trên, ô `(x, y)` có tâm ở `(16·(x−y), 8·(x+y))`, trùng đúng với `IsoToScreen(new Vector2(x, y) * 16)`, nên logic và tilemap luôn khớp.

**Địa hình có độ cao:** tạo **Custom Data Layers** trong TileSet:

| Tên | Kiểu | Ý nghĩa |
| --- | --- | --- |
| `height` | float | Độ cao hiển thị (px) và dùng cho lợi thế cao điểm |
| `speed` | float | Hệ số tốc độ di chuyển (1 = bình thường, 0.6 = bùn/dốc) |
| `forest` | bool | Cho phép núp (lệnh đột kích) |

Gán giá trị cho từng loại tile trong editor. Khi trận bắt đầu, đọc toàn bộ vào một mảng để tra nhanh (class `TerrainGrid` ở F.11). Về hình ảnh, đồi núi có thể làm bằng nhiều `TileMapLayer` chồng nhau, mỗi lớp cao hơn dịch lên một khoảng cố định và có tile vách đá ở rìa. Bản đầu tiên chỉ cần mặt đất phẳng, tile khác màu cho đồi, và lính đứng trên đồi được dịch lên bằng `height`.

### F.5 Bộ công cụ "ảo giác 3D" (làm lần lượt, mỗi thứ độc lập)

1. **Y-sort + gốc ở chân:** nền tảng của mọi thứ (C.3).
2. **Bóng đổ trên mặt đất:** sprite hình elip tối, mờ, đặt ở chân. Lính trên cao thì bóng nhạt hơn. Rẻ và hiệu quả nhất.
3. **Độ cao:** dịch `Body` lên, bóng ở lại mặt đất.
4. **Ánh sáng:** `DirectionalLight2D` làm "mặt trời" và `CanvasModulate` đổi tông cả cảnh theo buổi (bình minh vàng, hoàng hôn cam). Có thể gán **normal map** cho sprite qua `CanvasTexture` (ô Diffuse và Normal) để lính có khối khi ánh sáng đổi. Thêm `PointLight2D` cho lửa trại, đuốc.
5. **Vật cao che người:** cây, nhà, tháp che khuất lính phía sau thì làm mờ đi (bên dưới).
6. **Chiều sâu ảnh (tilt-shift):** làm mờ nhẹ phần trên và dưới màn hình, giữ nét dải giữa, trông như mô hình thu nhỏ rất "3D" (shader bên dưới).
7. **Sương/vignette theo khoảng cách:** phủ một lớp tối/màu nhạt ở rìa trên.
8. **Camera zoom:** phóng to khi ra lệnh gần, thu nhỏ để nhìn toàn cục; khi zoom đừng để lệch pixel (F.8).
9. **Hiệu ứng nhỏ:** bụi dưới chân kỵ binh, cờ bay, khói, tên bay theo cung (đường cong có độ cao).

Làm mờ vật cao khi che lính:

```csharp
using Godot;

public partial class TallObject : Node2D        // gốc ở chân cây/nhà
{
    [Export] private Area2D _behindZone;         // vùng PHÍA SAU vật thể trên màn hình
    private int _count;
    private Tween _tween;

    public override void _Ready()
    {
        _behindZone.AreaEntered += _ => Fade(true);
        _behindZone.AreaExited  += _ => Fade(false);
    }

    private void Fade(bool entered)
    {
        _count += entered ? 1 : -1;
        _tween?.Kill();
        _tween = CreateTween();
        _tween.TweenProperty(this, "modulate:a", _count > 0 ? 0.45f : 1f, 0.15f);
    }
}
```

(Mỗi lính cần một `Area2D` nhỏ ở chân, đặt layer riêng, nhưng với hàng nghìn lính thì chỉ làm vậy cho đơn vị của người chơi hoặc dùng cách kiểm tra theo ô lưới. Xem F.14.)

Shader tilt-shift (đặt trên một `ColorRect` phủ kín trong `CanvasLayer` nằm trên cùng, `Mouse Filter` = Ignore):

```glsl
shader_type canvas_item;
uniform sampler2D screen_tex : hint_screen_texture, filter_linear_mipmap;
uniform float focus_y : hint_range(0.0, 1.0) = 0.55;
uniform float focus_width : hint_range(0.0, 0.5) = 0.18;
uniform float max_blur : hint_range(0.0, 5.0) = 2.0;

void fragment() {
    float d = max(abs(SCREEN_UV.y - focus_y) - focus_width, 0.0);
    float blur = clamp(d * 6.0, 0.0, 1.0) * max_blur;
    COLOR = textureLod(screen_tex, SCREEN_UV, blur);
}
```

Với pixel art thuần, tilt-shift mờ có thể làm mất nét pixel; hãy thử giá trị nhỏ hoặc bỏ qua nếu không hợp phong cách.

### F.6 Quy trình art để lính trông "3D"

Có hai hướng, chọn một:

- **Vẽ tay (pixel art):** lính cao khoảng 16–24 px ở zoom 1x để hàng trăm người vẫn đọc được. Quy ước ánh sáng thống nhất (ví dụ luôn từ trên-trái), viền tối, màu đội rõ ràng (cờ, khiên, mũ). Mỗi loại lính cần các hướng nhìn: 4 hướng là đủ cho bản đầu (đối xứng trái-phải bằng `FlipH` thì còn 3), isometric thường cần 8.
- **Dựng 3D rồi render ra sprite (pre-rendered):** làm model low-poly trong Blender, đặt camera trực giao nghiêng cố định (khoảng 30–35°), render từng hướng và từng khung animation ra sprite sheet. Cách này cho ánh sáng và khối **nhất quán**, rất hợp "2D nhưng nhìn như 3D", và dễ thêm hướng nhìn. Đổi lại cần học cơ bản Blender (model, rig, animation đơn giản) và quản lý số lượng ảnh = số hướng × số khung × số loại lính.

Chọn công cụ: Aseprite hoặc LibreSprite (pixel art), Blender (miễn phí). Kiểm tra giấy phép mọi asset bạn tải, và ghi nguồn vào `CREDITS.txt`.

Quân số lớn nên làm **sprite nhỏ, ít khung animation** (đi 4 khung, đánh 3–4 khung, chết 1–2 khung). Nhìn ở quy mô một đội hình, người xem chủ yếu nhận ra bằng màu và hình khối, không phải chi tiết.

### F.7 Phân biệt loại lính và phe thật rõ

Vì nhìn từ xa, cần ít nhất hai dấu hiệu mỗi lính: **hình dáng** (giáo dài, cung, ngựa) và **màu đội** (dải màu ở khiên/cờ/mũ). Hiển thị thêm **cờ hiệu trên đầu mỗi stack** (một biểu tượng + số lính còn sống), vì chỉ huy ra lệnh theo stack chứ không theo từng lính.

### F.8 Camera cho sa bàn lớn

```csharp
using Godot;

public partial class BattleCamera : Camera2D
{
    [Export] public float PanSpeed = 500f;
    [Export] public float MinZoom = 0.5f, MaxZoom = 3f;
    [Export] public Rect2 WorldBounds = new(-2000, -2000, 4000, 4000);

    private float _targetZoom = 1f;
    private bool _dragging;

    public override void _Process(double delta)
    {
        float dt = (float)delta;
        Vector2 dir = Input.GetVector("cam_left", "cam_right", "cam_up", "cam_down");
        Position += dir * PanSpeed * dt / Zoom.X;                       // zoom càng xa, đi càng nhanh
        Position = Position.Clamp(WorldBounds.Position, WorldBounds.End);

        float z = Mathf.Lerp(Zoom.X, _targetZoom, 10f * dt);
        Zoom = new Vector2(z, z);
    }

    public override void _UnhandledInput(InputEvent e)
    {
        if (e is InputEventMouseButton mb)
        {
            if (mb.ButtonIndex == MouseButton.WheelUp)   _targetZoom = Mathf.Min(MaxZoom, _targetZoom * 1.15f);
            if (mb.ButtonIndex == MouseButton.WheelDown) _targetZoom = Mathf.Max(MinZoom, _targetZoom / 1.15f);
            if (mb.ButtonIndex == MouseButton.Middle)    _dragging = mb.Pressed;
        }
        else if (e is InputEventMouseMotion mm && _dragging)
            Position -= mm.Relative / Zoom.X;
    }
}
```

Thêm action `cam_left/right/up/down` (WASD hoặc mũi tên) vào Input Map. Với pixel art, zoom ở mức không nguyên có thể làm hình rung nhẹ; nếu gặp, giới hạn các mức zoom ở số nguyên hoặc dùng các bước cố định (0.5, 1, 2, 3).

### F.9 Kiến trúc tổng thể cho quân số lớn

Yêu cầu của bạn: quản lý theo **stack** (bộ binh 10, cung 10, kỵ binh 6...), nhưng **mỗi lính có AI và tốc độ đánh riêng**, hoạt động theo điều phối của stack. Cách tổ chức phù hợp:

| Thành phần | Loại | Nhiệm vụ |
| --- | --- | --- |
| `UnitData` | Resource | Chỉ số một loại lính (máu, sát thương, tầm đánh, tốc độ đánh, giá, kích thước stack) |
| `CounterTable` | Resource | Bảng tương khắc giữa các loại lính |
| `Squad` (stack) | Class C# thuần | Lệnh hiện tại, tâm đội hình, chỗ đứng (slot) của từng lính, vị trí xuất phát |
| `Soldier` | Node2D | Một lính: máu, hồi chiêu đánh, chọn mục tiêu, di chuyển; chỉ **thi hành**, không tự quyết lệnh |
| `SpatialHash` | Class C# thuần | Lưới băm không gian để tìm lính ở gần mà không duyệt toàn bộ |
| `TerrainGrid` | Class C# thuần | Độ cao, tốc độ, rừng theo ô, đọc từ TileMap |
| `BattleManager` | Node | Vòng lặp trận: dựng lưới, cập nhật stack, gọi từng lính, chạy kịch bản địch, kiểm tra thắng thua |
| `BattleScript` | Resource | Danh sách sự kiện theo thời gian cho phía địch |
| `Deployment` | Node | Giai đoạn mua lính và đặt quân |

Điểm then chốt: **một `BattleManager` duy nhất gọi `Tick` của tất cả lính** trong `_PhysicsProcess`, thay vì mỗi lính tự có `_PhysicsProcess`. Cách này giảm chi phí gọi qua lại giữa engine và C# khi có hàng trăm đến hàng nghìn lính, và giúp bạn kiểm soát thứ tự cập nhật (dựng lưới trước, rồi mới tìm mục tiêu).

Lính **không** dùng `CharacterBody2D` hay physics của Godot. Với hàng trăm lính va chạm nhau, physics engine rất nặng và cũng không cần thiết: ta tự tránh chồng lấn bằng lực đẩy nhẹ (separation) dựa trên lưới băm.

### F.10 Dữ liệu: UnitData và CounterTable

```csharp
using Godot;

public enum UnitType { Infantry, Spear, Archer, Cavalry }

[GlobalClass]
public partial class UnitData : Resource
{
    [Export] public string Id = "";
    [Export] public string DisplayName = "";
    [Export] public UnitType Type;
    [Export] public int Cost = 100;
    [Export] public int SquadSize = 10;          // bộ binh 10, cung 10, kỵ binh 6 ...
    [Export] public float MaxHp = 100f;
    [Export] public float Damage = 10f;
    [Export] public float AttackInterval = 1.0f;  // tốc độ đánh riêng của từng loại
    [Export] public float AttackRange = 14f;      // cận chiến ngắn, cung dài (vd. 120)
    [Export] public float SightRange = 130f;
    [Export] public float MoveSpeed = 40f;
    [Export] public Texture2D Icon;
    [Export] public SpriteFrames Frames;
}

[GlobalClass]
public partial class CounterTable : Resource
{
    // Ma trận n x n, hàng = bên tấn công, cột = bên chịu đòn,
    // thứ tự theo enum UnitType: Infantry, Spear, Archer, Cavalry.
    // Giá trị mẫu để chỉnh sau khi chơi thử.
    [Export] public float[] Matrix =
    {
        1.0f, 0.8f, 1.3f, 0.9f,   // Infantry đánh ...
        1.0f, 1.0f, 1.1f, 1.8f,   // Spear đánh ...  (giáo khắc kỵ binh)
        1.0f, 1.2f, 1.0f, 1.0f,   // Archer bắn ...
        1.4f, 0.6f, 1.5f, 1.0f,   // Cavalry xung phong ... (yếu trước giáo)
    };

    private static readonly int N = System.Enum.GetValues<UnitType>().Length;

    public float Get(UnitType attacker, UnitType defender)
    {
        int i = (int)attacker * N + (int)defender;
        return i >= 0 && i < Matrix.Length ? Matrix[i] : 1f;
    }
}
```

Tạo các file `.tres` (`infantry.tres`, `archer.tres`, `cavalry.tres`...) trong editor. Muốn thêm loại quân mới (voi, xe, lính cầm khiên) thì thêm vào enum, mở rộng ma trận, tạo `UnitData` mới. Khi thêm giá trị vào enum, **ma trận phải mở rộng theo**, nếu không bảng tương khắc sẽ lệch (cân nhắc đổi sang `Dictionary` nếu số loại quân tăng nhiều).

### F.11 TerrainGrid: tra địa hình nhanh

```csharp
using Godot;

public class TerrainGrid
{
    public const float Cell = 16f;                // 1 ô tile = 16 đơn vị logic
    private readonly int _w, _h, _ox, _oy;
    private readonly float[] _height, _speed;
    private readonly bool[] _forest;

    public TerrainGrid(TileMapLayer layer)
    {
        Rect2I r = layer.GetUsedRect();
        _ox = r.Position.X; _oy = r.Position.Y; _w = r.Size.X; _h = r.Size.Y;
        _height = new float[_w * _h];
        _speed = new float[_w * _h];
        _forest = new bool[_w * _h];
        System.Array.Fill(_speed, 1f);

        foreach (Vector2I c in layer.GetUsedCells())
        {
            TileData d = layer.GetCellTileData(c);
            if (d == null) continue;
            int i = (c.Y - _oy) * _w + (c.X - _ox);
            _height[i] = d.GetCustomData("height").AsSingle();
            _speed[i]  = d.GetCustomData("speed").AsSingle();
            _forest[i] = d.GetCustomData("forest").AsBool();
        }
    }

    private int Index(Vector2 g)
    {
        int x = Mathf.FloorToInt(g.X / Cell) - _ox;
        int y = Mathf.FloorToInt(g.Y / Cell) - _oy;
        return (x < 0 || y < 0 || x >= _w || y >= _h) ? -1 : y * _w + x;
    }

    public float HeightAt(Vector2 g) { int i = Index(g); return i < 0 ? 0f : _height[i]; }
    public float SpeedAt(Vector2 g)  { int i = Index(g); return i < 0 ? 1f : _speed[i]; }
    public bool IsForest(Vector2 g)  { int i = Index(g); return i >= 0 && _forest[i]; }
}
```

Cần tạo đúng ba Custom Data Layer tên `height` (float), `speed` (float), `forest` (bool) trong TileSet, nếu thiếu Godot sẽ báo lỗi khi gọi `GetCustomData`. Ô trống (không có tile) mặc định phẳng, tốc độ 1. Nếu bản đồ có nhiều lớp tile, chọn lớp "địa hình logic" làm nguồn, hoặc gộp dữ liệu nhiều lớp vào một mảng.

### F.12 Code lõi: SpatialHash, Squad, Soldier

**SpatialHash** (lưới băm, tránh cấp phát mới mỗi frame):

```csharp
using Godot;
using System.Collections.Generic;

public class SpatialHash
{
    private readonly float _cell;
    private readonly Dictionary<long, List<Soldier>> _cells = new();
    private readonly List<Soldier> _result = new();

    public SpatialHash(float cellSize) => _cell = cellSize;

    private static long Key(int x, int y) => ((long)x << 32) | (uint)y;

    public void Clear() { foreach (var l in _cells.Values) l.Clear(); }

    public void Insert(Soldier s)
    {
        long k = Key(Mathf.FloorToInt(s.Ground.X / _cell), Mathf.FloorToInt(s.Ground.Y / _cell));
        if (!_cells.TryGetValue(k, out var list)) _cells[k] = list = new List<Soldier>(8);
        list.Add(s);
    }

    // Trả về danh sách DÙNG LẠI: dùng xong ngay, đừng giữ tham chiếu qua lần gọi sau.
    public List<Soldier> Query(Vector2 pos, float radius)
    {
        _result.Clear();
        int x0 = Mathf.FloorToInt((pos.X - radius) / _cell), x1 = Mathf.FloorToInt((pos.X + radius) / _cell);
        int y0 = Mathf.FloorToInt((pos.Y - radius) / _cell), y1 = Mathf.FloorToInt((pos.Y + radius) / _cell);
        for (int y = y0; y <= y1; y++)
            for (int x = x0; x <= x1; x++)
                if (_cells.TryGetValue(Key(x, y), out var l)) _result.AddRange(l);
        return _result;
    }
}
```

**Squad** (stack): giữ lệnh, tâm đội hình và chỗ đứng.

```csharp
using Godot;
using System.Collections.Generic;

public enum Team { Player, Enemy }
public enum Order { Hold, Attack, Retreat, Ambush }   // thủ, tấn công, rút lui, đột kích

public class Squad
{
    public int Id;
    public Team Team;
    public UnitData Data;
    public Order Order = Order.Hold;
    public Vector2 Anchor;           // tâm đội hình hiện tại
    public Vector2 Home;             // vị trí ban đầu (để rút lui)
    public Vector2 Facing = Vector2.Right;
    public Vector2? MoveTarget;      // do kịch bản địch đặt: đi tới điểm này
    public Vector2 Centroid;         // tâm thật của các lính còn sống (BattleManager cập nhật)
    public int Alive;
    public float AmbushTrigger = 70f;
    public readonly List<Soldier> Members = new();

    public Vector2 SlotPosition(int index)
    {
        int cols = Mathf.Max(1, Mathf.CeilToInt(Mathf.Sqrt(Members.Count)));
        int r = index / cols, c = index % cols;
        const float spacing = 12f;
        var local = new Vector2(-r * spacing, (c - (cols - 1) / 2f) * spacing);
        return Anchor + local.Rotated(Facing.Angle());
    }

    public void SetOrder(Order o)
    {
        if (Order == o) return;
        Order = o;
        MoveTarget = null;
        foreach (var m in Members) m.OnOrderChanged();
    }

    public void ApplyScriptEvent(ScriptEvent e)
    {
        SetOrder(e.Order);
        MoveTarget = e.HasTarget ? e.TargetPos : null;   // gán SAU SetOrder (SetOrder xoá MoveTarget)
    }

    public void UpdateCentroid()
    {
        Vector2 sum = Vector2.Zero; int n = 0;
        foreach (var m in Members) if (!m.Dead) { sum += m.Ground; n++; }
        Alive = n;
        if (n > 0) Centroid = sum / n;
    }

    public void Update(float dt, BattleManager bm)
    {
        if (Alive == 0) return;
        float speed = Data.MoveSpeed;

        if (MoveTarget is Vector2 t)
        {
            Facing = (t - Anchor).Normalized();
            Anchor = Anchor.MoveToward(t, speed * 0.8f * dt);
            return;
        }

        switch (Order)
        {
            case Order.Retreat:
                Anchor = Anchor.MoveToward(Home, speed * 0.9f * dt);
                break;
            case Order.Attack:
                if (bm.NearestEnemyCentroid(this) is Vector2 goal)
                {
                    Facing = (goal - Anchor).Normalized();
                    Anchor = Anchor.MoveToward(goal, speed * 0.7f * dt);
                }
                break;
        }
    }
}
```

**Soldier:** mỗi lính có hồi chiêu riêng, bắt đầu lệch pha để không đánh đồng loạt.

```csharp
using Godot;

public partial class Soldier : Node2D
{
    [Export] private AnimatedSprite2D _body;
    [Export] private Sprite2D _shadow;

    public Team Team;
    public UnitData Data;
    public Squad Squad;
    public int SlotIndex;
    public Vector2 Ground;            // vị trí logic (phẳng)
    public float Hp;
    public bool Hidden;               // đang núp (lệnh đột kích)
    public bool Dead => Hp <= 0f;

    private BattleManager _bm;
    private float _cooldown;
    private float _retargetTimer;
    private Soldier _target;

    public void Init(BattleManager bm, UnitData data, Team team, Squad squad, int slot, Vector2 ground)
    {
        _bm = bm; Data = data; Team = team; Squad = squad; SlotIndex = slot; Ground = ground;
        Hp = data.MaxHp;
        _cooldown = GD.Randf() * data.AttackInterval;   // lệch pha tốc độ đánh
        _retargetTimer = GD.Randf() * 0.4f;             // lệch pha tìm mục tiêu
        _body.SpriteFrames = data.Frames;
        _body.Play("idle");
        OnOrderChanged();
        Place();
    }

    public void OnOrderChanged()
    {
        Hidden = Squad.Order == Order.Ambush && _bm.Terrain.IsForest(Ground);
        _target = null;
    }

    public void Tick(float dt, BattleManager bm)
    {
        if (Dead) return;
        _cooldown -= dt;
        _retargetTimer -= dt;

        switch (Squad.Order)
        {
            case Order.Hold:    DoHold(dt); break;
            case Order.Attack:  DoAttack(dt); break;
            case Order.Retreat: MoveTo(Squad.SlotPosition(SlotIndex), dt); break;
            case Order.Ambush:  DoAmbush(dt); break;
        }
        Place();
    }

    // THỦ: đứng ở chỗ của mình, chỉ đánh địch lọt vào tầm đánh
    private void DoHold(float dt)
    {
        var t = _bm.FindNearestEnemy(this, Data.AttackRange);
        if (t != null) TryAttack(t);
        else MoveTo(Squad.SlotPosition(SlotIndex), dt);
    }

    // TẤN CÔNG: tự tiến về phía trước đánh địch gần nhất
    private void DoAttack(float dt)
    {
        if (_retargetTimer <= 0f || _target == null || _target.Dead)
        {
            _target = _bm.FindNearestEnemy(this, Data.SightRange);
            _retargetTimer = 0.3f;
        }
        if (_target != null)
        {
            float dist = Ground.DistanceTo(_target.Ground);
            if (dist <= Data.AttackRange) TryAttack(_target);
            else MoveTo(_target.Ground, dt);
        }
        else MoveTo(Squad.SlotPosition(SlotIndex), dt);
    }

    // ĐỘT KÍCH: núp im, lộ diện khi địch đến gần (rồi cả stack chuyển sang tấn công)
    private void DoAmbush(float dt)
    {
        var t = _bm.FindNearestEnemy(this, Squad.AmbushTrigger);
        if (t != null) { Squad.SetOrder(Order.Attack); return; }
        MoveTo(Squad.SlotPosition(SlotIndex), dt);
    }

    private void TryAttack(Soldier t)
    {
        if (_cooldown > 0f) return;
        _cooldown = Data.AttackInterval;
        float mult = _bm.Counters.Get(Data.Type, t.Data.Type);
        float dh = _bm.Terrain.HeightAt(Ground) - _bm.Terrain.HeightAt(t.Ground);
        float highGround = 1f + Mathf.Clamp(dh * 0.01f, -0.2f, 0.3f);   // cao điểm đánh mạnh hơn
        t.TakeDamage(Data.Damage * mult * highGround);
        _body.Play("attack");
    }

    public void TakeDamage(float amount)
    {
        Hp -= amount;
        if (Dead) Die();
    }

    private void Die()
    {
        _body.Play("dead");
        _shadow.Visible = false;
        ZIndex = -1;                 // xác nằm dưới người sống
        Hidden = false;
    }

    private void MoveTo(Vector2 goal, float dt)
    {
        Vector2 to = goal - Ground;
        float dist = to.Length();
        Vector2 vel = dist > 2f ? to / dist * Data.MoveSpeed * _bm.Terrain.SpeedAt(Ground) : Vector2.Zero;

        // đẩy nhau ra để không chồng lên nhau
        foreach (var o in _bm.Grid.Query(Ground, 10f))
        {
            if (o == this || o.Dead) continue;
            Vector2 d = Ground - o.Ground;
            float l2 = d.LengthSquared();
            if (l2 > 0.01f && l2 < 100f) vel += d / l2 * 400f;
        }
        vel = vel.LimitLength(Data.MoveSpeed * 1.3f);
        Ground += vel * dt;

        if (vel.LengthSquared() > 1f) { _body.FlipH = vel.X < 0; if (_body.Animation != "walk") _body.Play("walk"); }
        else if (_body.Animation != "idle" && _body.Animation != "attack") _body.Play("idle");
    }

    private void Place()
    {
        Position = View.ToScreen(Ground);                       // node gốc nằm trên mặt đất (Y-sort)
        float h = _bm.Terrain.HeightAt(Ground);
        _body.Position = new Vector2(0, -h);                    // chỉ thân lính được nâng lên
        float k = Mathf.Clamp(1f - h / 80f, 0.4f, 1f);
        _shadow.Modulate = new Color(1, 1, 1, 0.5f * k);        // càng cao, bóng càng nhạt

        bool mine = Team == Team.Player;
        Visible = mine || !Hidden;                              // địch núp thì người chơi không thấy
        Modulate = (mine && Hidden) ? new Color(1, 1, 1, 0.45f) : Colors.White;
    }
}
```

Sprite animation cần có tên `idle`, `walk`, `attack`, `dead` trong `SpriteFrames`. Cung thủ trong bản đầu gây sát thương tức thời trong tầm bắn; sau này thêm mũi tên bay (một node nhỏ chạy theo đường cong, chạm đích mới trừ máu) để có cảm giác 3D hơn.

### F.13 Bốn lệnh, kịch bản địch và BattleManager

| Lệnh | Hành vi |
| --- | --- |
| **Thủ (Hold)** | Đứng yên ở slot, chỉ đánh địch vào tầm đánh |
| **Tấn công (Attack)** | Tâm đội hình tiến về phía địch; từng lính tự tìm địch gần nhất, đuổi và đánh |
| **Rút lui (Retreat)** | Tâm đội hình về vị trí ban đầu, lính không chủ động đánh |
| **Đột kích (Ambush)** | Núp trong rừng/địa hình đặc biệt, địch không nhắm tới được; khi địch lọt vào bán kính kích hoạt thì cả stack lộ diện và chuyển sang Tấn công |

Đột kích chỉ ẩn thật khi stack đang ở ô `forest` (xem `OnOrderChanged`). Muốn lệnh này có giá trị chiến thuật rõ ràng, cho đòn đầu tiên của stack vừa lộ diện nhân sát thương (ví dụ x1,5). Có thể thêm lệnh phụ (giữ đội hình, đổi hướng) khi game đã chạy ổn.

**Kịch bản địch (Resource):** địch có đội hình bố trí sẵn, diễn biến và lệnh thay đổi theo mốc thời gian.

```csharp
using Godot;

[GlobalClass]
public partial class ScriptEvent : Resource
{
    [Export] public float Time;            // giây kể từ khi trận bắt đầu
    [Export] public int SquadId;           // stack địch nào
    [Export] public Order Order;
    [Export] public bool HasTarget;
    [Export] public Vector2 TargetPos;     // tuỳ chọn: đi tới điểm này (toạ độ logic)
    [Export] public string Note = "";      // ghi chú cho bạn: "kỵ binh đánh tạt sườn"
}

[GlobalClass]
public partial class BattleScript : Resource
{
    [Export] public Godot.Collections.Array<ScriptEvent> Events = new();   // xếp tăng dần theo Time
}
```

`Squad.ApplyScriptEvent` (đã có ở F.12) đặt lệnh rồi mới gán `MoveTarget`, vì `SetOrder` xoá `MoveTarget`. Lưu ý `SetOrder` thoát sớm khi lệnh trùng lệnh cũ, nên một sự kiện kiểu "vẫn Thủ nhưng dời sang điểm khác" vẫn hoạt động nhờ dòng gán `MoveTarget` đặt bên ngoài phần thoát sớm; hãy thử từng loại sự kiện khi viết kịch bản.

**BattleManager:**

```csharp
using Godot;
using System.Collections.Generic;

public partial class BattleManager : Node
{
    public enum Phase { Deployment, Battle, Finished }

    [Signal] public delegate void BattleFinishedEventHandler(bool playerWon);

    [Export] public CounterTable Counters;
    [Export] public BattleScript EnemyScript;       // đừng đặt tên là Script (trùng thuộc tính của Godot)
    [Export] public Node2D Entities;                 // node cha có Y Sort Enabled
    [Export] public PackedScene SoldierScene;
    [Export] public TileMapLayer TerrainLayer;

    public SpatialHash Grid = new(32f);
    public TerrainGrid Terrain;
    public Phase CurrentPhase = Phase.Deployment;
    public readonly List<Squad> Squads = new();

    private readonly List<Soldier> _soldiers = new();
    private float _time;
    private int _nextEvent;
    private int _nextSquadId;

    public override void _Ready() => Terrain = new TerrainGrid(TerrainLayer);

    public Squad SpawnSquad(UnitData data, Team team, Vector2 ground, Vector2 facing, int? forcedId = null)
    {
        var sq = new Squad
        {
            Id = forcedId ?? _nextSquadId++, Team = team, Data = data,
            Anchor = ground, Home = ground, Facing = facing, Centroid = ground,
        };
        for (int i = 0; i < data.SquadSize; i++)
        {
            var s = SoldierScene.Instantiate<Soldier>();
            Entities.AddChild(s);
            sq.Members.Add(s);
            s.Init(this, data, team, sq, i, ground);
            s.Ground = sq.SlotPosition(i);
            _soldiers.Add(s);
        }
        sq.Alive = data.SquadSize;
        Squads.Add(sq);
        return sq;
    }

    public void StartBattle() { CurrentPhase = Phase.Battle; _time = 0f; _nextEvent = 0; }

    public override void _PhysicsProcess(double delta)
    {
        if (CurrentPhase != Phase.Battle) return;
        float dt = (float)delta;
        _time += dt;
        RunScript();

        Grid.Clear();
        foreach (var s in _soldiers) if (!s.Dead) Grid.Insert(s);
        foreach (var sq in Squads) sq.UpdateCentroid();
        foreach (var sq in Squads) sq.Update(dt, this);
        foreach (var s in _soldiers) s.Tick(dt, this);

        CheckResult();
    }

    private void RunScript()
    {
        var ev = EnemyScript?.Events;
        while (ev != null && _nextEvent < ev.Count && ev[_nextEvent].Time <= _time)
        {
            var e = ev[_nextEvent++];
            foreach (var sq in Squads)
                if (sq.Team == Team.Enemy && sq.Id == e.SquadId) { sq.ApplyScriptEvent(e); break; }
        }
    }

    public Soldier FindNearestEnemy(Soldier me, float radius)
    {
        Soldier best = null;
        float bestD = radius * radius;
        foreach (var o in Grid.Query(me.Ground, radius))
        {
            if (o.Team == me.Team || o.Dead || o.Hidden) continue;
            float d = me.Ground.DistanceSquaredTo(o.Ground);
            if (d < bestD) { bestD = d; best = o; }
        }
        return best;
    }

    public Vector2? NearestEnemyCentroid(Squad me)
    {
        Vector2? best = null; float bestD = float.MaxValue;
        foreach (var sq in Squads)
        {
            if (sq.Team == me.Team || sq.Alive == 0) continue;
            float d = me.Anchor.DistanceSquaredTo(sq.Centroid);
            if (d < bestD) { bestD = d; best = sq.Centroid; }
        }
        return best;
    }

    private void CheckResult()
    {
        int p = 0, e = 0;
        foreach (var sq in Squads) { if (sq.Team == Team.Player) p += sq.Alive; else e += sq.Alive; }
        if (p == 0 || e == 0)
        {
            CurrentPhase = Phase.Finished;
            EmitSignal(SignalName.BattleFinished, e == 0 && p > 0);
        }
    }
}
```

Điều kiện thắng thua thực tế còn có **thời gian giới hạn**, **giữ được một vị trí**, **tướng bị hạ**, **sĩ khí** (stack tự tan khi mất quá nửa quân). Thêm từng cái vào `CheckResult` sau khi trận cơ bản chạy ổn. Điều chỉnh tốc độ trận bằng `Engine.TimeScale` (1, 2, 4) và tạm dừng bằng `GetTree().Paused`; vì `BattleManager` chạy trong `_PhysicsProcess` nên nó tự co giãn theo.

### F.14 Hiệu năng khi quân số lớn

Con số dưới đây chỉ là **ước lượng thô để định hướng**; hiệu năng thật phụ thuộc máy và code, luôn đo bằng Profiler (mục 17 Phần I) trước khi tối ưu.

| Mức quân số | Cách làm |
| --- | --- |
| Vài trăm lính | Cấu trúc ở trên (mỗi lính một `Node2D` + sprite) là đủ. Đừng tối ưu sớm. |
| Vài trăm đến khoảng một, hai nghìn | Giãn cách AI (mỗi lính tìm mục tiêu mỗi 0,3 giây, đã lệch pha bằng `_retargetTimer`), tắt animation khi nằm ngoài camera, dọn xác cũ (xoá hoặc vẽ gộp lên lớp nền). |
| Hàng nghìn trở lên | Bỏ node-per-soldier: lưu lính trong mảng struct, vẽ bằng `MultiMeshInstance2D` (một lần vẽ cho hàng nghìn sprite). |

Phác thảo MultiMesh: tạo `MultiMeshInstance2D`, gán `Multimesh` với `TransformFormat = Transform2D`, `UseCustomData = true`, `Mesh = QuadMesh`, `InstanceCount = N`; mỗi frame gọi `SetInstanceTransform2D(i, new Transform2D(0f, pos))` và `SetInstanceCustomData(i, new Color(frame, dir, 0, 0))`, shader đọc `INSTANCE_CUSTOM` để chọn khung hình trong sprite sheet. Đổi lại bạn mất Y-sort tự động giữa các instance (cần tự sắp xếp theo Y hoặc chia nhiều `MultiMeshInstance2D` theo dải Y) và mất `AnimatedSprite2D`. Chỉ chuyển sang cách này khi Profiler chứng minh số node là điểm nghẽn.

Mẹo chung: dùng `Ground` thay vì `Position` trong logic để tránh gọi qua lại với engine; dựng `SpatialHash` một lần mỗi tick; không `new` trong `Tick`; không `GD.Print` trong vòng lặp nóng. Về "vật cao che người" (F.5): với số lượng lớn đừng gắn `Area2D` cho từng lính; dùng kiểm tra theo ô lưới (ô nào có lính của người chơi đứng sau vật cao thì làm mờ vật đó) hoặc một shader làm mờ dựa trên vị trí đội hình.

Vì game chỉ ra PC (Windows/Linux), bạn không bị giới hạn bởi điện thoại, và C# của Godot được hỗ trợ đầy đủ ở đó, rất hợp với thể loại quân số lớn.

### F.15 Giai đoạn bố trận (mua quân và đặt quân)

Vòng chơi: **Bố trận (có ngân sách) → Bắt đầu trận → Chỉ huy → Kết quả**.

```csharp
using Godot;
using System.Collections.Generic;

public partial class Deployment : Node2D
{
    [Signal] public delegate void BudgetChangedEventHandler(int budget);

    [Export] public BattleManager Battle;
    [Export] public int Budget = 1000;
    [Export] public Rect2 PlayerZone;                    // vùng được đặt quân (toạ độ logic)

    private UnitData _holding;                           // loại quân đang cầm theo chuột
    private readonly List<Squad> _placed = new();

    public void PickUnit(UnitData data) => _holding = data;      // gọi từ nút trong cửa hàng

    public override void _UnhandledInput(InputEvent e)
    {
        if (Battle.CurrentPhase != BattleManager.Phase.Deployment) return;
        if (e is not InputEventMouseButton mb || !mb.Pressed) return;

        Vector2 ground = View.ToGround(GetGlobalMousePosition());

        if (mb.ButtonIndex == MouseButton.Left && _holding != null)
        {
            if (Budget >= _holding.Cost && PlayerZone.HasPoint(ground))
            {
                Budget -= _holding.Cost;
                _placed.Add(Battle.SpawnSquad(_holding, Team.Player, ground, Vector2.Right));
                EmitSignal(SignalName.BudgetChanged, Budget);
            }
        }
        else if (mb.ButtonIndex == MouseButton.Right)      // bán lại stack dưới chuột
        {
            foreach (var sq in _placed)
                if (sq.Centroid.DistanceTo(ground) < 24f) { Refund(sq); break; }
        }
    }

    private void Refund(Squad sq) { /* xoá lính của sq, cộng lại sq.Data.Cost, EmitSignal */ }
}
```

Các điểm trải nghiệm nên có: **bóng mờ (ghost)** của stack bám theo chuột và đổi màu đỏ khi đặt sai chỗ hoặc thiếu tiền; **kéo để dời** stack đã đặt; **xoay hướng** bằng chuột phải kéo hoặc phím; lớp màu mờ hiển thị vùng bố trí hợp lệ; thanh ngân sách còn lại; **lưu/tải đội hình** (Resource hoặc JSON) để thử lại cùng một thế trận. Khi bấm "Bắt đầu", gọi `Battle.StartBattle()` và khoá việc đặt quân. Phía địch được `SpawnSquad` sẵn theo dữ liệu màn chơi (F.17) với `forcedId` khớp `SquadId` trong kịch bản.

### F.16 Chỉ huy khi giao tranh

Người chơi ra lệnh theo **stack**, không theo từng lính:

- **Chọn:** nhấp vào cờ hiệu stack hoặc kéo khung chọn nhiều stack; Ctrl+1..5 để gán nhóm.
- **Lệnh:** bốn nút hoặc phím tương ứng Thủ/Tấn công/Rút lui/Đột kích; nhấp phải xuống bản đồ để "đi tới điểm này rồi Thủ" (đặt `MoveTarget` và `Order = Hold`).
- **Thông tin:** cờ hiệu trên mỗi stack (biểu tượng, số lính còn sống, màu theo lệnh hiện tại), thanh sĩ khí nếu có.
- **Thời gian:** tạm dừng để ra lệnh, tốc độ 1x/2x/4x.
- **Minimap** rất đáng làm cho sa bàn lớn nhưng để sau.

Chọn bằng khung kéo (theo toạ độ màn hình), thêm các action `order_hold`, `order_attack`, `order_retreat`, `order_ambush` vào Input Map:

```csharp
private Vector2 _dragStart;
private readonly List<Squad> _selected = new();

public override void _UnhandledInput(InputEvent e)
{
    if (e is InputEventMouseButton mb && mb.ButtonIndex == MouseButton.Left)
    {
        if (mb.Pressed) _dragStart = GetGlobalMousePosition();
        else
        {
            var rect = new Rect2(_dragStart, GetGlobalMousePosition() - _dragStart).Abs();
            _selected.Clear();
            foreach (var sq in Battle.Squads)
                if (sq.Team == Team.Player && sq.Alive > 0 && rect.HasPoint(View.ToScreen(sq.Centroid)))
                    _selected.Add(sq);
        }
    }
    if (e.IsActionPressed("order_hold"))    Apply(Order.Hold);
    if (e.IsActionPressed("order_attack"))  Apply(Order.Attack);
    if (e.IsActionPressed("order_retreat")) Apply(Order.Retreat);
    if (e.IsActionPressed("order_ambush"))  Apply(Order.Ambush);
}

private void Apply(Order o) { foreach (var sq in _selected) sq.SetOrder(o); }
```

Đây là bản tối giản; khung kéo rất nhỏ (gần như nhấp) nên chọn stack gần con trỏ nhất.

### F.17 Dữ liệu một trận lịch sử

Gom mọi thứ của một màn vào một Resource để thêm trận mới mà không sửa code:

```csharp
[GlobalClass]
public partial class BattleDefinition : Resource
{
    [Export] public string Title = "";
    [Export(PropertyHint.MultilineText)] public string Briefing = "";   // bối cảnh lịch sử, mục tiêu
    [Export] public PackedScene MapScene;                  // sa bàn (tilemap, vật thể, ánh sáng)
    [Export] public int PlayerBudget = 1000;
    [Export] public Rect2 PlayerZone;
    [Export] public Godot.Collections.Array<UnitData> AvailableUnits = new();
    [Export] public Godot.Collections.Array<EnemySquadSpawn> EnemySquads = new();
    [Export] public BattleScript EnemyScript;
    [Export] public float TimeLimit = 1500f;               // 25 phút
}

[GlobalClass]
public partial class EnemySquadSpawn : Resource
{
    [Export] public int SquadId;
    [Export] public UnitData Unit;
    [Export] public Vector2 Position;
    [Export] public Vector2 Facing = Vector2.Left;
    [Export] public Order StartOrder = Order.Hold;
}
```

Với trận lịch sử: mỗi trận nên có **tóm tắt bối cảnh** và **mục tiêu** (thắng bằng cách nào), địa hình và buổi trong ngày gợi theo sử liệu, cùng vài **sự kiện kịch bản** làm điểm nhấn (viện binh xuất hiện, kỵ binh tạt sườn ở phút thứ N). Khi dùng tên quân, địa danh, số liệu có thật, hãy kiểm chứng nguồn; game có thể "lấy cảm hứng" thay vì khẳng định mọi chi tiết là chính xác.

### F.18 Lộ trình làm game theo milestone

Mỗi mốc là một bản chạy được. Thời gian chỉ là ước tính thô cho người mới, một mình, làm bán thời gian.

| Mốc | Nội dung | Ước tính |
| --- | --- | --- |
| M0 | Hàm chiếu, camera, một lính đi lại có bóng, bản đồ phẳng | 1 tuần |
| M1 | 100 lính hai phe chạy tới nhau và đánh nhau cơ bản (chưa có stack) | 1–2 tuần |
| M2 | `Squad`, đội hình theo slot, bốn lệnh | 1–2 tuần |
| M3 | `UnitData`, 4 loại quân, bảng tương khắc, cung bắn xa | 1–2 tuần |
| M4 | Địa hình: độ cao, tốc độ, rừng và lệnh Đột kích | 1–2 tuần |
| M5 | Bố trận: ngân sách, cửa hàng, đặt/bán quân, nút Bắt đầu | 1–2 tuần |
| M6 | Địch theo kịch bản, điều kiện thắng/thua, màn kết quả | 1–2 tuần |
| M7 | UI chỉ huy (chọn stack, lệnh, tốc độ), cờ hiệu, âm thanh, ánh sáng | 2 tuần |
| M8 | **Bản dọc (vertical slice):** một trận lịch sử hoàn chỉnh 15–30 phút, cân bằng lại | 2–3 tuần |
| M9 | Tối ưu quân số lớn, thêm trận, hoàn thiện menu/lưu | tuỳ |

Quy tắc: không sang mốc sau khi mốc trước còn lỗi nghiêm trọng; hết mỗi mốc thì commit Git và chơi thử. Trước M8 đừng làm nhiều trận. Làm một trận thật hay trước, các trận sau chỉ là dữ liệu mới (Resource) trên hệ thống đã có.

### F.19 Lỗi thường gặp và cách cân bằng game

- **Lính chồng thành một cục:** chưa có lực đẩy nhau, hoặc `SpatialHash` chưa dựng lại mỗi tick. Kiểm tra `Grid.Clear()` và `Insert` chạy trước `Tick`.
- **Lính giật, xoay vòng quanh mục tiêu:** hai lực (đuổi mục tiêu và đẩy nhau) triệt nhau. Giảm hệ số đẩy, giới hạn tốc độ, hoặc cho phép đánh ngay khi trong tầm thay vì cố vào sát.
- **Kết quả `Query` bị ghi đè:** danh sách kết quả được dùng lại; nếu gọi `Query` lồng nhau thì dữ liệu bị đè. Dùng xong rồi mới gọi lần kế, hoặc sao chép ra mảng tạm.
- **Lính trên đồi che sai thứ tự:** bạn nâng node gốc thay vì chỉ nâng `Body` (F.3).
- **Rút lui không về được:** lính bị chặn bởi ô có `speed = 0`; luôn để `speed > 0` cho ô đi được.
- **Địch không theo kịch bản:** `SquadId` trong `ScriptEvent` không khớp `SquadId` lúc spawn, hoặc `Events` chưa xếp tăng dần theo `Time`.
- **Chậm dần khi nhiều xác:** xác vẫn nằm trong danh sách; định kỳ xoá xác cũ khỏi `_soldiers` và `QueueFree`, hoặc vẽ xác lên lớp nền tĩnh.

**Cân bằng:** vì logic tách khỏi hiển thị và `Tick` dùng `dt`, bạn có thể viết công cụ chạy **hàng trăm trận mô phỏng không cần vẽ** để thống kê tỉ lệ thắng của từng kiểu đội hình và chỉnh bảng tương khắc. Giữ thông số trong Resource (`UnitData`, `CounterTable`) để chỉnh mà không cần biên dịch lại. Khi cho người khác chơi thử, quan sát: họ có hiểu lính nào khắc lính nào chỉ từ hình ảnh và gợi ý không, giai đoạn bố trận có "ra quyết định thật" không, hay thắng thua chỉ do số lượng.

---

Sau đây là bước hợp lý tiếp theo: làm **prototype từng bước cho M0–M2** (hàm chiếu, một lính có bóng, 100 lính đánh nhau, stack và bốn lệnh) rồi mới mở rộng.

---

# PHẦN IV. TỪ GDSCRIPT SANG C#

## Phụ lục G. Chuyển đổi GDScript sang C#: khai báo và cách dùng

Phần lớn tutorial, video và ví dụ Godot trên mạng viết bằng GDScript. Phụ lục này giúp bạn **đọc GDScript và viết lại thành C#** mà không bị mắc kẹt. Hai ngôn ngữ dùng **cùng một bộ API của engine**, nên bạn chỉ cần học quy tắc đổi tên và khác biệt về cú pháp.

### G.0 Ba điều cần nhớ trước

1. **API là một, tên khác nhau theo quy ước.** GDScript dùng `snake_case` (`get_node`, `queue_free`), C# dùng `PascalCase` (`GetNode`, `QueueFree`). Thuộc tính cũng vậy: `position` → `Position`, `velocity` → `Velocity`.
2. **Tài liệu chính thức có tab ngôn ngữ.** Ở trang class reference (docs.godotengine.org), chuyển tab sang **C#** để xem đúng tên phương thức, kiểu và hằng số. Đây là cách tra nhanh và chắc nhất. Trong Rider, gõ `Godot.` rồi dùng gợi ý (autocomplete) cũng cho tên đúng.
3. **Có thể dùng cả hai ngôn ngữ trong một project**, nhưng gọi chéo giữa chúng có hạn chế (xem G.14); với người mới nên chọn C# cho toàn bộ code của mình và chỉ dùng GDScript để đọc tutorial.

### G.1 Bảng chuyển nhanh các khai báo

| GDScript | C# |
| --- | --- |
| `extends CharacterBody2D` | `public partial class Player : CharacterBody2D` |
| `class_name Player` | tên class chính là tên toàn cục; thêm `[GlobalClass]` nếu muốn hiện trong hộp thoại tạo Node/Resource |
| `signal died` | `[Signal] public delegate void DiedEventHandler();` |
| `signal hp_changed(cur, max)` | `[Signal] public delegate void HpChangedEventHandler(int cur, int max);` |
| `var hp = 10` | `private int _hp = 10;` |
| `var hp: int = 10` | `private int _hp = 10;` |
| `var name := "A"` (suy luận kiểu) | `var name = "A";` (cục bộ) hoặc `private string _name = "A";` |
| `const MAX = 5` | `private const int Max = 5;` |
| `static var count = 0` | `public static int Count = 0;` |
| `enum State { IDLE, RUN }` | `public enum State { Idle, Run }` |
| `@export var speed = 100.0` | `[Export] public float Speed = 100f;` |
| `@onready var sprite = $Sprite2D` | gán trong `_Ready`: `_sprite = GetNode<Sprite2D>("Sprite2D");` |
| `@tool` | `[Tool]` trên class |
| `func f(a: int) -> void:` | `private void F(int a)` |
| `static func f():` | `public static void F()` |
| `func _ready():` | `public override void _Ready()` |
| `func _process(delta):` | `public override void _Process(double delta)` |
| `print("x")` | `GD.Print("x");` |

Hai khác biệt lớn nhất về hình thức: C# **bắt buộc khai báo kiểu rõ ràng** (trừ `var` cho biến cục bộ) và **dùng `{ }` thay cho thụt lề**, mỗi câu lệnh kết thúc bằng `;`.

### G.2 Khung một script: extends, class\_name, partial

```gdscript
# GDScript: Player.gd
class_name Player
extends CharacterBody2D
```

```csharp
// C#: Player.cs (tên file PHẢI trùng tên class)
using Godot;

[GlobalClass]                                   // tuỳ chọn, giống class_name khi tạo node/resource
public partial class Player : CharacterBody2D   // 'partial' là bắt buộc
{
}
```

Vì sao `partial`? Godot có một bộ sinh code (source generator) tự thêm phần còn lại của class (danh sách `[Export]`, tên signal, tên phương thức...) vào cùng class. Thiếu `partial` thì script không gắn được vào node và các `SignalName`/`MethodName` không tồn tại.

`_init()` của GDScript tương ứng với **constructor** C# (`public Player() { }`). Godot cần constructor không tham số để tạo node từ scene, và các node con chưa tồn tại lúc này, nên **đừng đặt logic vào constructor**; hãy đặt vào `_Ready()`.

### G.3 Biến, hằng, enum, static

```gdscript
var health: int = 100
var speed := 150.0
const MAX_JUMPS = 2
enum State { IDLE, RUN, JUMP }
var state: State = State.IDLE
static var kills = 0
```

```csharp
private int _health = 100;
private float _speed = 150f;               // số thực C# cần hậu tố f cho float
private const int MaxJumps = 2;
public enum State { Idle, Run, Jump }
private State _state = State.Idle;
public static int Kills = 0;
```

Quy ước C#: field private đặt `_camelCase`, hằng và thành viên public đặt `PascalCase`, biến cục bộ `camelCase`. Mức truy cập (`public`, `private`) phải ghi rõ; không ghi thì mặc định là `private`.

**Getter/setter (property):**

```gdscript
var health: int = 100:
    set(value):
        health = clampi(value, 0, max_health)
        health_changed.emit(health)
    get:
        return health
```

```csharp
private int _health = 100;

public int Health
{
    get => _health;
    set
    {
        _health = Mathf.Clamp(value, 0, MaxHealth);
        EmitSignal(SignalName.HealthChanged, _health);
    }
}
```

Lưu ý: setter có thể được gọi trong editor hoặc trước `_Ready` khi một số tham chiếu còn `null`; hãy kiểm tra null nếu setter dùng đến node khác.

### G.4 Kiểu dữ liệu: GDScript → C#

| GDScript | C# | Ghi chú |
| --- | --- | --- |
| `int` | `int` (hoặc `long`) | `int` của GDScript là 64-bit; C# `int` 32-bit, dùng `long` khi cần số rất lớn |
| `float` | `float` (hoặc `double`) | GDScript float là 64-bit; API Godot C# dùng `float` cho vector/toạ độ, còn `double` cho `delta` và thời gian |
| `bool`, `String` | `bool`, `string` |  |
| `StringName` (`&"jump"`) | `StringName` | nhận chuỗi thường nhờ chuyển kiểu ngầm |
| `NodePath` | `NodePath` |  |
| `Vector2`, `Color`, `Rect2`... | giống tên | `Vector2.ZERO` → `Vector2.Zero`, `Color.RED` → `Colors.Red` |
| `Array` | `Godot.Collections.Array` | hoặc `Godot.Collections.Array<T>` có kiểu, hoặc `List<T>` / `T[]` của C# |
| `Array[int]` | `Godot.Collections.Array<int>` | hoặc `List<int>` nếu không cần giao tiếp với Godot |
| `Dictionary` | `Godot.Collections.Dictionary` | hoặc `Godot.Collections.Dictionary<K,V>` / `Dictionary<K,V>` của .NET |
| `PackedByteArray` | `byte[]` |  |
| `PackedInt32Array` | `int[]` |  |
| `PackedFloat32Array` | `float[]` |  |
| `PackedVector2Array` | `Vector2[]` |  |
| `PackedStringArray` | `string[]` |  |
| `Variant` / không ghi kiểu | `Variant` (hoặc kiểu cụ thể) | C# khuyến khích ghi kiểu cụ thể |
| `Object` | `GodotObject` |  |
| `Callable`, `Signal` | `Callable`, `Signal` |  |
| `null` | `null` |  |

**Quy tắc chọn cho người mới:** trong code nội bộ dùng `List<T>`, `Dictionary<K,V>`, `T[]` của C# (nhanh và tiện hơn, hỗ trợ LINQ). Chỉ dùng `Godot.Collections.*` khi phải đưa dữ liệu qua Godot, ví dụ biến `[Export]`, đối số của signal hoặc `Call`.

```csharp
var enemies = new List<Enemy>();            // nội bộ
enemies.Add(e);

[Export] public Godot.Collections.Array<ItemData> Items = new();   // hiện trong Inspector
```

**Một số khác biệt dễ gây lỗi:**

- Chia hai số nguyên trong C# cho **số nguyên** (`7 / 2 == 3`), trong GDScript `7 / 2` cũng ra 3 nhưng `7 / 2.0` là 3,5; C# giống vậy, nhưng nhớ viết `7 / 2f`.
- `Vector2` trong C# là **struct**: `velocity.x = 5` (hợp lệ trong GDScript) phải đổi thành `Velocity = new Vector2(5, Velocity.Y);` hoặc copy ra biến rồi gán lại.
- Chuỗi định dạng: `"%d xu" % n` → `$"{n} xu"`.

### G.5 Hàm

```gdscript
func take_damage(amount: int = 1, from: Node = null) -> bool:
    health -= amount
    return health > 0

func _is_alive() -> bool:
    return health > 0

var heal = func(n): health += n      # lambda
```

```csharp
public bool TakeDamage(int amount = 1, Node from = null)
{
    Health -= amount;
    return Health > 0;
}

private bool IsAlive() => Health > 0;          // biểu thức gọn một dòng

Action<int> heal = n => Health += n;            // lambda (cần using System;)
```

Khác biệt: C# **phải ghi kiểu trả về** (`bool`, `void`...) ngay trước tên hàm; tham số mặc định dùng cú pháp giống GDScript; `void` nghĩa là không trả về giá trị. C# cho phép **nạp chồng (overload)**: nhiều hàm cùng tên nhưng khác tham số, điều GDScript không có.

### G.6 Các hàm vòng đời

| GDScript | C# |
| --- | --- |
| `func _init():` | constructor `public Player() { }` (tránh logic) |
| `func _enter_tree():` | `public override void _EnterTree()` |
| `func _ready():` | `public override void _Ready()` |
| `func _process(delta):` | `public override void _Process(double delta)` |
| `func _physics_process(delta):` | `public override void _PhysicsProcess(double delta)` |
| `func _input(event):` | `public override void _Input(InputEvent @event)` |
| `func _unhandled_input(event):` | `public override void _UnhandledInput(InputEvent @event)` |
| `func _draw():` | `public override void _Draw()` |
| `func _exit_tree():` | `public override void _ExitTree()` |
| `func _notification(what):` | `public override void _Notification(int what)` |

Lưu ý ký tự `@` trong `@event`: `event` là từ khoá của C#, dấu `@` cho phép dùng nó làm tên tham số. Cũng nhớ mỗi hàm vòng đời phải có `override`, và `delta` là **`double`** (ép sang `float` khi nhân với giá trị `float`: `(float)delta`).

### G.7 Đổi tên API: quy tắc và các ngoại lệ

**Quy tắc chung:** `snake_case` → `PascalCase`.

| GDScript | C# |
| --- | --- |
| `get_node("A")`, `$A` | `GetNode<T>("A")` |
| `%Name` (Unique Name) | `GetNode<T>("%Name")` |
| `get_node_or_null` | `GetNodeOrNull<T>("A")` |
| `get_parent()` | `GetParent<T>()` |
| `get_children()` | `GetChildren()` |
| `find_child("x")` | `FindChild("x")` |
| `add_child(n)` | `AddChild(n)` |
| `queue_free()` | `QueueFree()` |
| `get_tree()` | `GetTree()` |
| `is_instance_valid(n)` | `GodotObject.IsInstanceValid(n)` |
| `add_to_group("g")` | `AddToGroup("g")` |
| `Input.is_action_pressed("a")` | `Input.IsActionPressed("a")` |
| `move_and_slide()` | `MoveAndSlide()` |
| `velocity`, `position` | `Velocity`, `Position` |
| `global_position` | `GlobalPosition` |
| `preload("res://x.tscn")` | `GD.Load<PackedScene>("res://x.tscn")` |
| `load("res://x.tres")` | `GD.Load<T>("res://x.tres")` |
| `scene.instantiate()` | `scene.Instantiate<T>()` |
| `get_tree().change_scene_to_file(p)` | `GetTree().ChangeSceneToFile(p)` |
| `get_tree().reload_current_scene()` | `GetTree().ReloadCurrentScene()` |
| `call_deferred("f")` | `CallDeferred(MethodName.F)` |
| `create_tween()` | `CreateTween()` |

**Hằng số và enum đổi dạng:** hằng `UPPER_SNAKE` của GDScript trở thành thành viên của một **enum lồng trong class** (PascalCase):

| GDScript | C# |
| --- | --- |
| `Input.MOUSE_MODE_CAPTURED` | `Input.MouseModeEnum.Captured` |
| `Node.PROCESS_MODE_ALWAYS` | `Node.ProcessModeEnum.Always` |
| `KEY_SPACE` | `Key.Space` |
| `MOUSE_BUTTON_LEFT` | `MouseButton.Left` |
| `Tween.TRANS_QUAD` | `Tween.TransitionType.Quad` |
| `Color.RED` | `Colors.Red` |
| `Vector2.UP` | `Vector2.Up` |
| `PI`, `TAU` | `Mathf.Pi`, `Mathf.Tau` |

Ví dụ dùng: `Input.MouseMode = Input.MouseModeEnum.Captured;`, `ProcessMode = ProcessModeEnum.Always;`.

**Hàm toán học** (GDScript dùng hàm toàn cục, C# dùng `Mathf` hoặc `GD`):

| GDScript | C# |
| --- | --- |
| `lerp(a, b, t)` | `Mathf.Lerp(a, b, t)` |
| `clamp(x, a, b)` | `Mathf.Clamp(x, a, b)` |
| `move_toward(a, b, d)` | `Mathf.MoveToward(a, b, d)` |
| `abs(x)`, `sign(x)` | `Mathf.Abs(x)`, `Mathf.Sign(x)` |
| `sqrt`, `floor`, `round` | `Mathf.Sqrt`, `Mathf.Floor`, `Mathf.Round` |
| `floori(x)`, `roundi(x)` | `Mathf.FloorToInt(x)`, `Mathf.RoundToInt(x)` |
| `deg_to_rad(d)` | `Mathf.DegToRad(d)` |
| `randf()` | `GD.Randf()` |
| `randi_range(a, b)` | `GD.RandRange(a, b)` (số nguyên, gồm cả hai đầu) |
| `randf_range(a, b)` | `(float)GD.RandRange(a, b)` (kiểu `double`, ép sang `float` nếu cần) |
| `v.normalized()`, `v.length()` | `v.Normalized()`, `v.Length()` |
| `a.distance_to(b)` | `a.DistanceTo(b)` |
| `a.direction_to(b)` | `a.DirectionTo(b)` |

**Tên sinh tự động (an toàn khi gõ):** nhờ `partial`, Godot sinh các lớp lồng `PropertyName`, `MethodName`, `SignalName` chứa tên dưới dạng `StringName`, ví dụ `SignalName.Died`, `MethodName.TakeDamage`. Dùng chúng thay vì gõ chuỗi giúp tránh lỗi chính tả: `CallDeferred(MethodName.Respawn);`, `EmitSignal(SignalName.Died);`.

### G.8 Lấy node: `$`, `%`, `@onready`

```gdscript
@onready var sprite: AnimatedSprite2D = $AnimatedSprite2D
@onready var timer: Timer = %CoyoteTimer
@onready var camera = get_node("../Camera2D") as Camera2D
```

```csharp
private AnimatedSprite2D _sprite;
private Timer _timer;
private Camera2D _camera;

public override void _Ready()
{
    _sprite = GetNode<AnimatedSprite2D>("AnimatedSprite2D");
    _timer  = GetNode<Timer>("%CoyoteTimer");
    _camera = GetNode<Camera2D>("../Camera2D");
}
```

C# **không có `@onready`**; cách tương đương là gán trong `_Ready()`. Cách tốt hơn cả là dùng `[Export]` rồi kéo node vào Inspector (không phụ thuộc đường dẫn):

```csharp
[Export] private AnimatedSprite2D _sprite;
```

`GetNode<T>` ném lỗi nếu sai đường dẫn hoặc sai kiểu. Muốn không lỗi khi node có thể không tồn tại, dùng `GetNodeOrNull<T>(...)` rồi kiểm tra `null`.

**Kiểm tra và ép kiểu:**

```gdscript
if body is Player:
    body.take_damage(1)
var p := body as Player
```

```csharp
if (body is Player player)       // kiểm tra và ép kiểu cùng lúc
    player.TakeDamage(1);
var p = body as Player;          // null nếu không đúng kiểu
```

### G.9 @export và các biến thể

```gdscript
@export var speed := 150.0
@export_range(0, 100, 1) var health := 100
@export_enum("Easy", "Hard") var difficulty := 0
@export_file("*.tscn") var next_level: String
@export_multiline var description: String
@export var bullet_scene: PackedScene
@export var target: Node2D
@export var items: Array[ItemData]
@export_group("Movement")
@export_subgroup("Jump")
@export_category("Player")
```

```csharp
[Export] public float Speed = 150f;
[Export(PropertyHint.Range, "0,100,1")] public int Health = 100;
[Export] public Difficulty Level = Difficulty.Easy;      // enum C# tự thành danh sách chọn
[Export(PropertyHint.File, "*.tscn")] public string NextLevel;
[Export(PropertyHint.MultilineText)] public string Description;
[Export] public PackedScene BulletScene;
[Export] public Node2D Target;
[Export] public Godot.Collections.Array<ItemData> Items = new();
[ExportGroup("Movement")]
[ExportSubgroup("Jump")]
[ExportCategory("Player")]
```

Với enum, khai báo `public enum Difficulty { Easy, Hard }` ở đâu đó trong project rồi dùng như kiểu thường. Khoảng giá trị có thể thêm `"or_greater"`: `"0,100,1,or_greater"`. Sau khi sửa hoặc thêm `[Export]`, nhớ **Build** để Inspector cập nhật.

### G.10 Signal: khai báo, phát, kết nối, chờ

**Khai báo và phát:**

```gdscript
signal health_changed(current: int, maximum: int)
signal died

health_changed.emit(health, 100)
died.emit()
```

```csharp
[Signal] public delegate void HealthChangedEventHandler(int current, int maximum);
[Signal] public delegate void DiedEventHandler();

EmitSignal(SignalName.HealthChanged, _health, 100);
EmitSignal(SignalName.Died);
```

Quy ước bắt buộc: tên delegate **phải kết thúc bằng `EventHandler`**; Godot cắt phần đó đi để ra tên signal (`HealthChanged`), đồng thời sinh `SignalName.HealthChanged` và sự kiện C# cùng tên.

**Kết nối:**

```gdscript
player.health_changed.connect(_on_health_changed)
button.pressed.connect(_on_pressed)
timer.timeout.connect(func(): print("hết giờ"))
player.died.connect(_on_died, CONNECT_ONE_SHOT)
```

```csharp
player.HealthChanged += OnHealthChanged;       // signal khai báo bằng C# hoặc có sẵn của Godot
button.Pressed += OnPressed;
timer.Timeout += () => GD.Print("hết giờ");

// Dạng tổng quát dùng Connect + Callable:
player.Connect(Player.SignalName.Died, Callable.From(OnDied),
               (uint)GodotObject.ConnectFlags.OneShot);
```

Trường hợp cần `Connect` thay cho `+=`: kết nối tới signal của đối tượng viết bằng GDScript, hoặc cần cờ như `OneShot`/`Deferred`. Với hàm có tham số, dùng `Callable.From<int, int>(OnHealthChanged)`.

**Ngắt kết nối** (làm ở `_ExitTree` khi node nghe có thể bị huỷ trước node phát):

```csharp
player.HealthChanged -= OnHealthChanged;
// hoặc: player.Disconnect(Player.SignalName.HealthChanged, Callable.From<int,int>(OnHealthChanged));
```

**Chờ signal (await):**

```gdscript
await get_tree().create_timer(1.0).timeout
await animation_player.animation_finished
await player.died
```

```csharp
await ToSignal(GetTree().CreateTimer(1.0), SceneTreeTimer.SignalName.Timeout);
await ToSignal(_animPlayer, AnimationPlayer.SignalName.AnimationFinished);
await ToSignal(player, Player.SignalName.Died);
```

Hàm có `await` trong C# nên khai báo `async`: `private async void OnPlayerDied() { await ToSignal(...); ... }` (với hàm vòng đời và callback, `async void` là chấp nhận được; hàm khác nên `async Task`). Sau `await`, node có thể đã bị huỷ: kiểm tra `IsInstanceValid(this)` khi cần.

### G.11 Luồng điều khiển và cú pháp thường gặp

| GDScript | C# |
| --- | --- |
| `if x > 0:` / `elif` / `else:` | `if (x > 0) { } else if (...) { } else { }` |
| `and`, `or`, `not` | `&&`, `\|\|`, `!` |
| `for i in range(5):` | `for (int i = 0; i < 5; i++)` |
| `for e in enemies:` | `foreach (var e in enemies)` |
| `while cond:` | `while (cond) { }` |
| `match state:` | `switch (state) { case State.Idle: ... break; }` |
| `x in array` | `array.Contains(x)` |
| `arr.size()` | `arr.Count` (List) / `arr.Length` (mảng) |
| `arr.append(x)` | `arr.Add(x)` |
| `arr.erase(x)` | `arr.Remove(x)` |
| `dict.has(k)` | `dict.ContainsKey(k)` |
| `dict.get(k, def)` | `dict.GetValueOrDefault(k, def)` hoặc `TryGetValue` |
| `"%s: %d" % [a, b]` | `$"{a}: {b}"` |
| `pass` | `{ }` (khối rỗng) |
| `# ghi chú` | `// ghi chú` |
| `var x = 5 if a else 3` | `var x = a ? 5 : 3;` |
| `await foo()` | `await Foo();` (hàm `async`) |
| `yield` (Godot 3) | không còn; dùng `await` |

`match` còn có dạng "biểu thức switch" gọn hơn trong C#:

```csharp
string label = state switch
{
    State.Idle => "đứng",
    State.Run  => "chạy",
    _          => "khác",
};
```

### G.12 Tween, Timer, nhóm và hàm thường dùng khác

```gdscript
var t = create_tween()
t.tween_property(sprite, "modulate:a", 0.0, 0.4)
t.tween_callback(queue_free)
await get_tree().create_timer(2.0).timeout
get_tree().call_group("enemies", "alert")
```

```csharp
var t = CreateTween();
t.TweenProperty(_sprite, "modulate:a", 0.0f, 0.4f);
t.TweenCallback(Callable.From(QueueFree));
await ToSignal(GetTree().CreateTimer(2.0), SceneTreeTimer.SignalName.Timeout);
GetTree().CallGroup("enemies", "Alert");          // tên hàm là tên của script nhận (xem G.14)
```

### G.13 Các "chú thích" đặc biệt (annotation)

| GDScript | C# |
| --- | --- |
| `@tool` | `[Tool]` |
| `@icon("res://icon.svg")` | `[Icon("res://icon.svg")]` |
| `@rpc("any_peer")` | `[Rpc(MultiplayerApi.RpcMode.AnyPeer)]` |
| `@warning_ignore(...)` | `#pragma warning disable ...` (của C#) |
| `@onready` | không có, gán trong `_Ready` |
| `@export_flags(...)` | `[Export(PropertyHint.Flags, "A,B,C")]` hoặc enum có `[Flags]` |

### G.14 Gọi chéo GDScript và C#

Nếu project có cả hai loại script:

- **Từ C# gọi GDScript:** dùng gọi động, tên theo đúng tên trong file GDScript (`snake_case`): `node.Call("take_damage", 5);`, `node.Get("health");`, `node.Set("health", 3);`, hoặc kết nối signal bằng `Connect`.
- **Từ GDScript gọi C#:** gọi động bằng tên của phương thức C# (giữ nguyên PascalCase), ví dụ `node.TakeDamage(5)`; phương thức phải `public`.
- **Tạo node từ script khác ngôn ngữ:** C# tải GDScript bằng `GD.Load<GDScript>(...)` rồi `.New()`; GDScript tải script C# bằng `load("res://Player.cs").new()`.
- Kiểu tĩnh (trình biên dịch kiểm tra) chỉ có khi **cả hai bên cùng C#**. Gọi chéo là gọi động, lỗi chính tả chỉ lộ ra lúc chạy.

Chi tiết và hạn chế có thể khác nhau giữa các phiên bản Godot; xem mục *Cross-language scripting* trong tài liệu chính thức của đúng bản bạn dùng. Với người mới, lời khuyên đơn giản nhất: **giữ code gameplay trọn vẹn trong C#**.

### G.15 Ví dụ dịch trọn một script

**GDScript gốc:**

```gdscript
class_name Player
extends CharacterBody2D

signal health_changed(current: int, maximum: int)
signal died

@export_group("Movement")
@export var speed := 150.0
@export_range(0, 10, 1) var max_jumps := 2
@export var bullet_scene: PackedScene

@onready var sprite: AnimatedSprite2D = $AnimatedSprite2D
@onready var coyote_timer: Timer = %CoyoteTimer

enum State { IDLE, RUN, JUMP }
const MAX_HEALTH = 3

var state: State = State.IDLE
var health := MAX_HEALTH:
    set(value):
        health = clampi(value, 0, MAX_HEALTH)
        health_changed.emit(health, MAX_HEALTH)
        if health == 0:
            died.emit()

var gravity: float = ProjectSettings.get_setting("physics/2d/default_gravity")

func _ready() -> void:
    coyote_timer.timeout.connect(_on_coyote_timeout)

func _physics_process(delta: float) -> void:
    if not is_on_floor():
        velocity.y += gravity * delta
    var dir := Input.get_axis("move_left", "move_right")
    velocity.x = dir * speed
    move_and_slide()
    if dir != 0:
        sprite.flip_h = dir < 0

func shoot() -> void:
    var b = bullet_scene.instantiate()
    b.global_position = global_position
    get_parent().add_child(b)

func take_damage(amount: int = 1) -> void:
    health -= amount

func _on_coyote_timeout() -> void:
    pass
```

**Bản C# tương đương:**

```csharp
using Godot;

[GlobalClass]
public partial class Player : CharacterBody2D
{
    [Signal] public delegate void HealthChangedEventHandler(int current, int maximum);
    [Signal] public delegate void DiedEventHandler();

    [ExportGroup("Movement")]
    [Export] public float Speed = 150f;
    [Export(PropertyHint.Range, "0,10,1")] public int MaxJumps = 2;
    [Export] public PackedScene BulletScene;

    public enum State { Idle, Run, Jump }
    private const int MaxHealth = 3;

    private AnimatedSprite2D _sprite;
    private Timer _coyoteTimer;
    private float _gravity = ProjectSettings.GetSetting("physics/2d/default_gravity").AsSingle();
    private int _health = MaxHealth;

    public State CurrentState { get; private set; } = State.Idle;

    public int Health
    {
        get => _health;
        set
        {
            _health = Mathf.Clamp(value, 0, MaxHealth);
            EmitSignal(SignalName.HealthChanged, _health, MaxHealth);
            if (_health == 0) EmitSignal(SignalName.Died);
        }
    }

    public override void _Ready()
    {
        _sprite = GetNode<AnimatedSprite2D>("AnimatedSprite2D");
        _coyoteTimer = GetNode<Timer>("%CoyoteTimer");
        _coyoteTimer.Timeout += OnCoyoteTimeout;
    }

    public override void _PhysicsProcess(double delta)
    {
        Vector2 v = Velocity;                       // Vector2 là struct: copy ra, sửa, gán lại
        if (!IsOnFloor())
            v.Y += _gravity * (float)delta;         // delta là double

        float dir = Input.GetAxis("move_left", "move_right");
        v.X = dir * Speed;
        Velocity = v;
        MoveAndSlide();

        if (dir != 0)
            _sprite.FlipH = dir < 0;
    }

    public void Shoot()
    {
        var b = BulletScene.Instantiate<Node2D>();
        b.GlobalPosition = GlobalPosition;
        GetParent().AddChild(b);
    }

    public void TakeDamage(int amount = 1) => Health -= amount;

    private void OnCoyoteTimeout() { }
}
```

Đối chiếu hai bản để thấy mô hình chuyển đổi: signal → `[Signal] delegate`, `@export` → `[Export]`, `@onready` → gán trong `_Ready`, setter → property, `velocity.y += ...` → copy `Velocity` ra biến rồi gán lại, `await/emit` → `ToSignal/EmitSignal`.

### G.16 Lỗi hay gặp khi dịch

- Quên `partial` hoặc tên file khác tên class: script không gắn được, `SignalName` không tồn tại.
- Quên **Build** sau khi thêm `[Export]` hoặc `[Signal]`.
- Sửa trực tiếp `Velocity.X`/`Position.Y`: báo lỗi vì là struct; gán lại cả `Vector2`.
- Trộn `float` và `double`: ép kiểu `(float)delta`; hằng số thực dùng hậu tố `f`.
- Dùng tên signal sai định dạng: delegate phải kết thúc bằng `EventHandler`.
- Gõ `GetNode("A")` không có kiểu rồi gọi hàm riêng của node: dùng `GetNode<T>("A")` để có đúng kiểu.
- Đặt tên biến trùng với thành viên có sẵn của Godot (ví dụ `Name`, `Script`, `Owner`): đổi tên khác, tránh nhầm với thuộc tính của `Node`.
- Dùng `@` bị thiếu ở tên tham số `@event`.
- Dịch nguyên văn `$Node` thành chuỗi đường dẫn nhưng node đã đổi tên: dùng `[Export]` để kéo node thay vì đường dẫn.
- Dùng `await` trong hàm không `async`: thêm `async` vào khai báo hàm.

### G.17 Bài tập dịch

1. Dịch script GDScript hiển thị đồng xu (`Area2D` + `body_entered` + `queue_free`) sang C#.
2. Dịch script menu chính (3 nút, `pressed.connect`, `get_tree().change_scene_to_file`) sang C#.
3. Dịch một autoload `GameManager` có `signal coins_changed`, hàm `add_coins`, biến static.
4. Lấy một tutorial GDScript bất kỳ (ví dụ nhân vật platformer) trên mạng, tự dịch từng dòng sang C# bằng bảng ở G.1–G.13, rồi chạy thử và so hành vi.
5. Mở trang tài liệu của `CharacterBody2D`, chuyển tab sang C#, so tên các hàm với bản GDScript để quen cách tra.

## Phụ lục H. Ví dụ dịch song song GDScript và C#

Mỗi ví dụ có **GDScript ở trên, C# ở dưới** và vài ghi chú về chỗ khác nhau. Cách học: che bản C# đi, tự dịch bản GDScript bằng bảng ở Phụ lục G, rồi so sánh. Tất cả giả định bạn đã `Build` và đã kéo các node cần thiết vào Inspector.

### H.1 Đồng xu (Area2D, signal có sẵn, gọi autoload)

```gdscript
extends Area2D

func _ready() -> void:
    body_entered.connect(_on_body_entered)

func _on_body_entered(body: Node2D) -> void:
    if body is Player:
        GameManager.add_coins(1)
        queue_free()
```

```csharp
using Godot;

public partial class Coin : Area2D
{
    public override void _Ready() => BodyEntered += OnBodyEntered;

    private void OnBodyEntered(Node2D body)
    {
        if (body is Player)
        {
            GameManager.Instance.AddCoins(1);
            QueueFree();
        }
    }
}
```

Ghi chú: GDScript gọi autoload bằng **tên** (`GameManager`). Trong C#, autoload là một node nên dùng mẫu `Instance` tĩnh (xem mục 10.1 Phần I) hoặc `GetNode<GameManager>("/root/GameManager")`. `body_entered` thành sự kiện `BodyEntered` và nối bằng `+=`.

### H.2 Menu chính (nút bấm, đổi scene)

```gdscript
extends Control

@onready var play_button: Button = %PlayButton
@onready var quit_button: Button = %QuitButton

func _ready() -> void:
    play_button.pressed.connect(func(): get_tree().change_scene_to_file("res://scenes/Level1.tscn"))
    quit_button.pressed.connect(get_tree().quit)
    play_button.grab_focus()
```

```csharp
using Godot;

public partial class MainMenu : Control
{
    private Button _play;
    private Button _quit;

    public override void _Ready()
    {
        _play = GetNode<Button>("%PlayButton");
        _quit = GetNode<Button>("%QuitButton");

        _play.Pressed += () => GetTree().ChangeSceneToFile("res://scenes/Level1.tscn");
        _quit.Pressed += () => GetTree().Quit();
        _play.GrabFocus();
    }
}
```

Ghi chú: `@onready var x = %Name` thành gán trong `_Ready` bằng `GetNode<T>("%Name")`. Truyền thẳng `get_tree().quit` làm Callable trong GDScript thì C# dùng lambda `() => GetTree().Quit()`.

### H.3 Autoload GameManager với signal

```gdscript
extends Node

signal coins_changed(coins: int)

var coins := 0

func add_coins(n: int) -> void:
    coins += n
    coins_changed.emit(coins)

func reset_run() -> void:
    coins = 0
    coins_changed.emit(coins)
```

```csharp
using Godot;

public partial class GameManager : Node
{
    [Signal] public delegate void CoinsChangedEventHandler(int coins);

    public static GameManager Instance { get; private set; }
    public int Coins { get; private set; }

    public override void _EnterTree() => Instance = this;

    public void AddCoins(int n)
    {
        Coins += n;
        EmitSignal(SignalName.CoinsChanged, Coins);
    }

    public void ResetRun()
    {
        Coins = 0;
        EmitSignal(SignalName.CoinsChanged, Coins);
    }
}
```

Ghi chú: thêm vào Project Settings > Globals > Autoload giống nhau cho cả hai ngôn ngữ. Trong C# cần tự tạo `Instance` (singleton) để truy cập từ nơi khác. Hãy đặt `Instance` trong `_EnterTree` để nó sẵn sàng trước khi node khác chạy `_Ready`.

### H.4 Resource tự định nghĩa (dữ liệu vật phẩm)

```gdscript
class_name ItemData
extends Resource

@export var id: String
@export var display_name: String
@export var icon: Texture2D
@export var price: int = 0
@export_multiline var description: String
```

```csharp
using Godot;

[GlobalClass]
public partial class ItemData : Resource
{
    [Export] public string Id;
    [Export] public string DisplayName;
    [Export] public Texture2D Icon;
    [Export] public int Price = 0;
    [Export(PropertyHint.MultilineText)] public string Description;
}
```

Dùng ở node khác:

```gdscript
@export var item: ItemData
@export var items: Array[ItemData]
```

```csharp
[Export] public ItemData Item;
[Export] public Godot.Collections.Array<ItemData> Items = new();
```

Ghi chú: `class_name` tương ứng `[GlobalClass]` để class xuất hiện trong hộp thoại **New Resource**. Sau khi build, tạo file `.tres` ở FileSystem > chuột phải > New Resource > chọn `ItemData`.

### H.5 Component máu (signal có tham số, nối ở node cha)

```gdscript
class_name Health
extends Node

signal health_changed(current: int, maximum: int)
signal died

@export var max_health := 100
var current := 0

func _ready() -> void:
    current = max_health
    health_changed.emit(current, max_health)

func take_damage(amount: int) -> void:
    current = max(0, current - amount)
    health_changed.emit(current, max_health)
    if current == 0:
        died.emit()
```

```csharp
using Godot;

[GlobalClass]
public partial class Health : Node
{
    [Signal] public delegate void HealthChangedEventHandler(int current, int maximum);
    [Signal] public delegate void DiedEventHandler();

    [Export] public int MaxHealth = 100;
    public int Current { get; private set; }

    public override void _Ready()
    {
        Current = MaxHealth;
        EmitSignal(SignalName.HealthChanged, Current, MaxHealth);
    }

    public void TakeDamage(int amount)
    {
        Current = Mathf.Max(0, Current - amount);
        EmitSignal(SignalName.HealthChanged, Current, MaxHealth);
        if (Current == 0) EmitSignal(SignalName.Died);
    }
}
```

Nối signal ở node cha (ví dụ Enemy có con `Health`):

```gdscript
@onready var health: Health = $Health

func _ready():
    health.died.connect(_on_died)
    health.health_changed.connect(_on_health_changed)
```

```csharp
[Export] private Health _health;

public override void _Ready()
{
    _health.Died += OnDied;
    _health.HealthChanged += OnHealthChanged;
}

public override void _ExitTree()
{
    _health.Died -= OnDied;                    // ngắt kết nối khi node bị huỷ
    _health.HealthChanged -= OnHealthChanged;
}
```

Ghi chú: `max(0, x)` thành `Mathf.Max(0, x)`. Với signal có tham số, hàm xử lý C# phải khớp kiểu và số tham số. Trong C# nên ngắt kết nối ở `_ExitTree` khi node phát có thể sống lâu hơn node nghe.

### H.6 Kẻ địch có trạng thái (match, tween, await)

```gdscript
extends CharacterBody2D

enum State { PATROL, CHASE, DEAD }

@export var speed := 40.0
var state := State.PATROL
var target: Node2D

func _physics_process(delta: float) -> void:
    match state:
        State.PATROL:
            velocity.x = speed
        State.CHASE:
            if target:
                velocity = global_position.direction_to(target.global_position) * speed * 1.5
        State.DEAD:
            velocity = Vector2.ZERO
    move_and_slide()

func die() -> void:
    state = State.DEAD
    var t := create_tween()
    t.tween_property($Sprite2D, "modulate:a", 0.0, 0.4)
    await t.finished
    queue_free()
```

```csharp
using Godot;

public partial class Enemy : CharacterBody2D
{
    public enum State { Patrol, Chase, Dead }

    [Export] public float Speed = 40f;
    [Export] private Sprite2D _sprite;

    private State _state = State.Patrol;
    private Node2D _target;

    public override void _PhysicsProcess(double delta)
    {
        switch (_state)
        {
            case State.Patrol:
                Velocity = new Vector2(Speed, Velocity.Y);          // không sửa Velocity.X trực tiếp
                break;
            case State.Chase:
                if (_target != null)
                    Velocity = GlobalPosition.DirectionTo(_target.GlobalPosition) * Speed * 1.5f;
                break;
            case State.Dead:
                Velocity = Vector2.Zero;
                break;
        }
        MoveAndSlide();
    }

    public async void Die()
    {
        _state = State.Dead;
        var t = CreateTween();
        t.TweenProperty(_sprite, "modulate:a", 0.0f, 0.4f);
        await ToSignal(t, Tween.SignalName.Finished);
        QueueFree();
    }
}
```

Ghi chú: `match` → `switch` (mỗi `case` kết thúc bằng `break`); `velocity.x = ...` → gán lại cả `Velocity`; `await t.finished` → `await ToSignal(t, Tween.SignalName.Finished)`; hàm có `await` phải khai báo `async`.

### H.7 Xử lý chuột và tạo đối tượng

```gdscript
@export var bullet_scene: PackedScene

func _unhandled_input(event: InputEvent) -> void:
    if event is InputEventMouseButton and event.pressed and event.button_index == MOUSE_BUTTON_LEFT:
        var b = bullet_scene.instantiate()
        b.global_position = get_global_mouse_position()
        add_child(b)
```

```csharp
[Export] public PackedScene BulletScene;

public override void _UnhandledInput(InputEvent @event)
{
    if (@event is InputEventMouseButton { Pressed: true, ButtonIndex: MouseButton.Left })
    {
        var b = BulletScene.Instantiate<Node2D>();
        b.GlobalPosition = GetGlobalMousePosition();
        AddChild(b);
    }
}
```

Ghi chú: tham số tên `@event`; `is` có thể kèm **pattern thuộc tính** `{ Pressed: true, ... }` để kiểm tra nhiều điều kiện một lúc; `instantiate()` thành `Instantiate<T>()` có kiểu để dùng ngay `GlobalPosition`.

### H.8 Vòng lặp xây UI động

```gdscript
@onready var grid: GridContainer = $Grid

func refresh(items: Array[ItemData]) -> void:
    for child in grid.get_children():
        child.queue_free()
    for item in items:
        var label := Label.new()
        label.text = item.display_name
        grid.add_child(label)
```

```csharp
using System.Collections.Generic;

[Export] private GridContainer _grid;

public void Refresh(IEnumerable<ItemData> items)
{
    foreach (var child in _grid.GetChildren())
        child.QueueFree();

    foreach (var item in items)
    {
        var label = new Label { Text = item.DisplayName };
        _grid.AddChild(label);
    }
}
```

Ghi chú: `Label.new()` thành `new Label()`; C# có cú pháp khởi tạo `new Label { Text = ... }` để gán thuộc tính ngay khi tạo. Xoá node trong lúc duyệt danh sách con an toàn vì `QueueFree` xoá muộn (cuối frame).

### H.9 Lưu và tải bằng JSON

```gdscript
const SAVE_PATH := "user://save.json"

func save_game(data: Dictionary) -> void:
    var f := FileAccess.open(SAVE_PATH, FileAccess.WRITE)
    f.store_string(JSON.stringify(data))

func load_game() -> Dictionary:
    if not FileAccess.file_exists(SAVE_PATH):
        return {}
    var f := FileAccess.open(SAVE_PATH, FileAccess.READ)
    return JSON.parse_string(f.get_as_text())
```

```csharp
using Godot;

public static class SaveSystem
{
    private const string SavePath = "user://save.json";

    public static void SaveGame(Godot.Collections.Dictionary data)
    {
        using var f = FileAccess.Open(SavePath, FileAccess.ModeFlags.Write);
        f.StoreString(Json.Stringify(data));
    }

    public static Godot.Collections.Dictionary LoadGame()
    {
        if (!FileAccess.FileExists(SavePath)) return new();
        using var f = FileAccess.Open(SavePath, FileAccess.ModeFlags.Read);
        return Json.ParseString(f.GetAsText()).AsGodotDictionary();
    }
}
```

Ghi chú: lớp JSON của Godot trong C# tên `Json` (không phải `JSON`). Với game lớn, ưu tiên cách dùng **class C# có kiểu + `System.Text.Json`** ở mục 9.2 Phần I để có kiểm tra kiểu và dễ bảo trì. `using var` đóng file tự động khi ra khỏi hàm.

### H.10 Mẫu nhớ nhanh: cùng một việc, hai cách viết

| Việc | GDScript | C# |
| --- | --- | --- |
| Chạy một lần khi sẵn sàng | `func _ready():` | `public override void _Ready()` |
| Gọi hàm sau 1 giây | `await get_tree().create_timer(1).timeout` | `await ToSignal(GetTree().CreateTimer(1.0), SceneTreeTimer.SignalName.Timeout);` |
| Kiểm tra va chạm nhóm | `if body.is_in_group("enemy"):` | `if (body.IsInGroup("enemy"))` |
| Phát nhạc | `$Music.play()` | `_music.Play();` |
| Đọc ô tilemap | `layer.get_cell_tile_data(cell)` | `layer.GetCellTileData(cell)` |
| Xoá node sau khi xong | `queue_free()` | `QueueFree();` |
| Gọi hàm muộn | `call_deferred("f")` | `CallDeferred(MethodName.F);` |
| In giá trị | `print("hp=", hp)` | `GD.Print($"hp={hp}");` |

### H.11 Bài tập dịch (có đáp án để tự đối chiếu)

1. Dịch script `Hello.gd` in ra tên node cha: `print(get_parent().name)` sang C# (đáp án: `GD.Print(GetParent().Name);`).
2. Dịch một `RigidBody2D` đẩy lên khi nhấn `jump`: `apply_impulse(Vector2.UP * 300)` sang C# (đáp án: `ApplyImpulse(Vector2.Up * 300f);`).
3. Dịch `var speeds: Array[float] = [1.0, 2.0]` sang C# với `List<float>` và với `Godot.Collections.Array<float>`.
4. Dịch đoạn `for i in range(3): spawn(i)` và `for e in get_tree().get_nodes_in_group("enemies"): e.queue_free()`.
5. Tự chọn một tutorial GDScript 50–100 dòng trên mạng, dịch toàn bộ sang C# và chạy thử; ghi lại mỗi chỗ phải tra tài liệu vào sổ lỗi.

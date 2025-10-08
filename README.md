# 🔹 Part 1 — Project & Rig Foundation (Meta-only) + Locomotion + Grab <span style="color:purple;font-weight:bold;">🟣  </span>

**Goal:** In a brand-new scene, drop **OVRPlayerController** for **locomotion**, add **controller rays** for UI, set up **grab** on a test cube, and prove clicks on a **world-space button**.
**Absolutely no** “XR” root object.

---

## 0) One-time Project Baseline (Android • URP • Meta) <span style="color:purple;">🟣</span>

⬜ **0.1** **File → Build Settings → Android → Switch Platform**
 • **Texture Compression = ASTC**

⬜ **0.2** **Project Settings → Player → Other Settings**
 • **Scripting Backend = IL2CPP**, **ARM64** only
 • **Minimum API ≥ 29**, **Target API = Highest Installed**

⬜ **0.3** **Project Settings → Graphics**
 • **Scriptable Render Pipeline Settings =** your **URP Asset**

⬜ **0.4** **Meta → Tools → Meta XR Project Setup → Fix All**
 • Apply all recommended fixes.

⬜ **0.5** **Player → Active Input Handling = Both**
⬜ **0.6** **Project Window:** Make folders → `Assets/_Scenes`, `Assets/Scripts`, `Assets/Prefabs`, `Assets/Materials`, `Assets/UI`

---

## 1) New Scene & Clean Stage <span style="color:purple;">🟣</span>

⬜ **1.1** **File → New Scene** → **Save As** `Assets/_Scenes/S09_TicTacToe.unity`

⬜ **1.2** **Hierarchy:** select **Main Camera** → **Delete** (OVR rig provides camera)

⬜ **1.3** (Optional) **Directional Light**: keep it; Intensity ≈ **0.9**, Soft Shadows ON

---

## 2) Add the Meta Rig **with built-in locomotion**  <span style="color:purple;">🟣</span>

> We’ll use **OVRPlayerController** (includes a CharacterController + OVRCameraRig). This gives **left/right stick locomotion & turning** out of the box.

⬜ **2.1** **Project (Search):** `OVRPlayerController`
 **Drag** **OVRPlayerController** to **Hierarchy (root)**
 *(Do NOT create any “XR” empty parent.)*

⬜ **2.2** **Inspector (OVRPlayerController)**
 - **Character Controller → Height = 1.7**, **Radius = 0.3**
 
 - **Gravity Modifier = 1** (default)
 
 - **Rotation Either Thumbstick = ON** (if exposed; enables right-stick yaw)
 
 - **HMD Rotates Player = ON** (common default; OK for class)

> Inside the OVRPlayerController you’ll find an **OVRCameraRig** child with **LeftHandAnchor** and **RightHandAnchor**. We’ll attach interactors there.

---

## 3) Add **Controller Rays** (UI pointing) <span style="color:purple;">🟣</span>

⬜ **3.1** **Make a Reticle prefab**
 **Hierarchy:** Right-click → **3D Object → Torus** → rename **ReticleRing**
 
 **Transform:** **Scale (0.05, 0.05, 0.05)**
 
 **Material:** `Assets/Materials/M_Reticle_Unlit` (URP/Lit, BaseColor=white, **Emission ON** white) → assign to **ReticleRing**
 
 **Project:** drag **ReticleRing** to **Assets/Prefabs** → **Delete** it from scene (we’ll reference prefab)

⬜ **3.2** **Left controller ray**
 **Hierarchy Path:** `OVRPlayerController/OVRCameraRig/TrackingSpace/LeftHandAnchor`
 If no **ControllerInteractors** child exists: Right-click **LeftHandAnchor** → **Create Empty** → **ControllerInteractors** (reset local transform)
 
 **Project (Search):** *Controller Ray Interactor* (Meta Interaction SDK **component** or **prefab**, names vary slightly)
 
 - If **prefab**: **Drag** under `LeftHandAnchor/ControllerInteractors`
 
 - If **script**: **Add Component → Controller Ray Interactor** to a new empty under **ControllerInteractors**
 
 **Inspector (Ray Interactor):**
 
 • **Max Ray Length = 6**
 
 • **Hide When No Interactable = ON** (if available)
 
 • **Reticle/Visual =** **`Prefabs/ReticleRing`**
 
 • **Line Width ≈ 0.003–0.006**
 

⬜ **3.3** **Right controller ray**
 Repeat 3.2 at `OVRPlayerController/OVRCameraRig/TrackingSpace/RightHandAnchor`

---

## 4) Add **Near Grab** and **Ray Grab** Interactors to both hands <span style="color:purple;">🟣</span>

> We’ll let students grab close objects (near) and also pull objects from afar (ray).

⬜ **4.1** **Left hand**

 **Hierarchy Path:** `.../LeftHandAnchor/ControllerInteractors`
 
 **Add Component → Grab Interactor** (Meta)  ← *near-grab*
 
 (Keep defaults)
 
 *(If your SDK provides a “Controller Grab Interactor” prefab, you may drag it here instead.)*

⬜ **4.2** **Right hand**

 Repeat **Grab Interactor** on `.../RightHandAnchor/ControllerInteractors`

> Now each hand has: **Controller Ray Interactor** (far) + **Grab Interactor** (near). The Meta runtime will map grip/trigger properly.

---

## 5) **UI Event System** (Meta pipeline) + **World-Space test button** <span style="color:purple;">🟣</span>

⬜ **5.1** **EventSystem**

 **Hierarchy:** Right-click → **UI → Event System** → rename **EventSystem**
 
 Select it → **Remove** Standalone/Input System UI module if present
 
 **Add Component → OVR Input Module**
 
 **Inspector (OVR Input Module):**
 
 • **Submit on Button Down = ON**
 
 • If fields for **Ray Transform L/R** exist, drag **LeftHandAnchor** / **RightHandAnchor** (some versions auto-bind)

⬜ **5.2** **World-Space Canvas**

 **Hierarchy:** Right-click → **UI → Canvas** → rename **TestCanvas**

 **Inspector (Canvas):** **Render Mode = World Space**
 
 **RectTransform:** **Pos (0, 1.5, 1.5)**, **Size (0.6, 0.3)** (meters)
 
 **Layer = UIWorld** (we’ll use this later for filters)
 
 **Add Component → OVR Raycaster** (critical for Meta UI rays)
 

⬜ **5.3** **Button (TMP)**
 **Hierarchy:** `TestCanvas` → Right-click → **UI → Button (TextMeshPro)** → rename **TestButton**
 
 **Button → Transition = Color Tint** (make **Highlighted** a brighter color)
 
 **Child `Text (TMP)` → Text = “CLICK ME”**, Font Size **48–72**, Alignment **Center**

⬜ **5.4** **Script probe**

 **Project:** `Assets/Scripts/UIButtonDebug.cs`

```csharp
using UnityEngine;

public class UIButtonDebug : MonoBehaviour
{
    // Attach this to the same GameObject that has the Button component.
    public void PrintClick()
    {
        Debug.Log("[UI] Button clicked via controller ray.");
        // Later: play a click sound or send haptics here.
    }
}
```

 **Hierarchy:** select **TestButton** → **Add Component → UIButtonDebug**
 **Inspector (TestButton → OnClick())**: **+** → drag **TestButton** → choose **UIButtonDebug.PrintClick()**

---

## 6) **Ground**, a **Grab Cube**, and a **Ray-Grabbable** setup <span style="color:purple;">🟣</span>

⬜ **6.1** **Ground**
 **Hierarchy:** 3D Object → **Plane** → rename **Ground**
 **Transform:** **Pos (0,0,0)** • **Scale (4,1,4)**

⬜ **6.2** **Cube to grab**
 **Hierarchy:** 3D Object → **Cube** → rename **GrabCube**
 **Transform:** **Scale (0.2,0.2,0.2)**, **Pos (0, 0.5, 1.8)**

⬜ **6.3** **Physics & Interaction (manual path)**
 **Inspector (GrabCube):**
 • **Add Component → Rigidbody** (Use Gravity **ON**, Interpolate **ON**)
 • **Add Component → Grabbable** → **Rigidbody** slot: drag **GrabCube (Rigidbody)**
 • **Add Component → ColliderSurface** → **Collider** slot: drag **GrabCube (BoxCollider)**
 • **Add Component → RayInteractable**
  – **Pointable Element = Grabbable**
  – **Surface = ColliderSurface**
 • **Add Component → MoveTowardsTargetProvider**
  – In **RayInteractable → Movement Provider**: assign this provider

⬜ **6.4** **Wizard (easier, alternative to 6.3)**
 Right-click **GrabCube** → **Interaction SDK → Add Ray Grab Interaction**
 • This creates a **parent** (e.g., `ISDK Ray Grab Interaction`) and moves **GrabCube** under it.
 • Most components live on the **new parent**—edit that parent for settings.
 *(Either manual or wizard path is fine; pick ONE.)*

---

## 7) **Play Test** (locomotion + grab + ray UI) <span style="color:purple;">🟣</span>

1. **Enter Play** (Quest Link/AirLink recommended).
2. **Locomotion:**

   * **Left stick** (or primary): **move** around (OVRPlayerController).
   * **Right stick** (or secondary): **turn** (snap/continuous depends on your version; both OK).
3. **Ray click:** Aim at **TestButton** → **pull trigger** → Console logs:
   `"[UI] Button clicked via controller ray."`
4. **Ray grab:** Aim at **GrabCube** → pull/hold to grab. Release to drop.
5. **Near grab:** Move hand close to cube → grip to grab (if near grab interactor is in reach).

---

## 8) What your **Hierarchy** should roughly look like now

```
OVRPlayerController   (root, has CharacterController)
└─ OVRCameraRig
   └─ TrackingSpace
      ├─ LeftHandAnchor
      │  └─ ControllerInteractors
      │     ├─ Controller Ray Interactor (+ line/reticle)
      │     └─ Grab Interactor (near)
      └─ RightHandAnchor
         └─ ControllerInteractors
            ├─ Controller Ray Interactor (+ line/reticle)
            └─ Grab Interactor (near)

EventSystem (OVR Input Module)
TestCanvas (World Space, OVR Raycaster, Layer: UIWorld)
└─ TestButton (TMP + UIButtonDebug)

Ground
GrabCube (or ISDK Ray Grab Interaction parent with cube as child)
Directional Light
```

> ✅ **Notice:** There is **no** “XR” GameObject anywhere.

---

## 9) Common Pitfalls & Fixes

* **I can’t move:** Ensure you added **OVRPlayerController** (not just OVRCameraRig). The CharacterController must be enabled; ground must be below feet (Plane at y=0).
* **No rays / no reticle:** Confirm each **HandAnchor** has a **ControllerInteractors** child with a **Controller Ray Interactor**. Some visuals hide when nothing is hit—aim at the Canvas.
* **UI won’t click:**

  * **EventSystem** must have **OVR Input Module** (remove Standalone).
  * **Canvas = World Space** and has **OVR Raycaster**.
* **Can’t grab:**

  * The **cube** must have **Rigidbody + Grabbable + ColliderSurface + (RayInteractable + MovementProvider)** or the **wizard parent**.
  * Hands must have **Grab Interactor** components.
* **Controller sticks do nothing:** Make sure headset is in focus (Link active), and **OVRPlayerController** is not disabled. Check Input handling is **Both**.

---
# 🔹 Part 2 — Wall Board, BoardCanvas, 3×3 Grid, Cell Prefab

<span style="color:purple;font-weight:bold;">🟣   — complete every purple step</span>

**Goal:** Create a **wall** in 3D, mount a **world-space canvas** on it, add a **Grid Layout (3×3)**, and make a **Cell** **Button (TMP)** prefab the rays can hover/click. Zero game logic yet (that’s Part 3).

**You will use:** **Hierarchy**, **Inspector**, **Project**, **Scene** view, **UI** menu.

---

## 0) Quick pre-flight (must be true from Part 1) <span style="color:purple;">🟣</span>

⬜ **0.1** In **Hierarchy**, you already have **OVRPlayerController** (root) with:

* `OVRPlayerController/OVRCameraRig/TrackingSpace/LeftHandAnchor/ControllerInteractors` → **Controller Ray Interactor** + **Grab Interactor**
* Same on **RightHandAnchor**
* **EventSystem** (with **OVR Input Module**)
* **TestCanvas** (World Space) with **OVR Raycaster**
* **Ground** (Plane)

> If anything is missing, fix Part 1 first. Students can’t continue without working rays.

---

## 1) Build the wall (3D) <span style="color:purple;">🟣</span>

⬜ **1.1** **Hierarchy:** Right-click → **3D Object → Cube** → rename **`Wall`**
⬜ **1.2** **Inspector (Wall → Transform):**

* **Position = (0, 1.5, 2.2)**
* **Rotation = (0, 0, 0)**
* **Scale = (3, 2, 0.08)**

> Result: a flat wall ~3m wide, ~2m tall, in front of the player.

---

## 2) Create the Board Canvas (World Space, ray-clickable) <span style="color:purple;">🟣</span>

⬜ **2.1** **Hierarchy:** Right-click on **`Wall`** → **UI → Canvas** → rename **`BoardCanvas`**
⬜ **2.2** **Inspector (BoardCanvas → Canvas):**

* **Render Mode = World Space**
* **Event Camera = (leave empty)** *(OVR raycaster doesn’t need it)*
  ⬜ **2.3** **Inspector (BoardCanvas → RectTransform):**
* **Position = (0, 1.3, -0.045)** *(a few cm in front of wall: negative Z in local space)*
* **Rotation = (0, 0, 0)**
* **Size = Width: 0.8, Height: 0.8** *(meters)*
* **Scale = (1, 1, 1)**

⬜ **2.4** **Layer:** set **BoardCanvas → Layer = UIWorld** *(from Part 1)*
⬜ **2.5** **Raycaster:** **Add Component → OVR Raycaster** (keep default **Blocking Objects = None**)
*(Keep the default **Graphic Raycaster** too — it’s fine to have both.)*

> Result: a square UI board on the wall, sized in **meters**, ready for ray clicks.

---

## 3) Add a subtle board background (Panel) <span style="color:purple;">🟣</span>

⬜ **3.1** **Hierarchy:** Right-click **BoardCanvas** → **UI → Panel** → rename **`BoardPanel`**
⬜ **3.2** **Inspector (BoardPanel → RectTransform):**

* **Anchor Preset = Stretch (full)**: click the **square** icon left of Pos fields → choose **stretch both**
* **Left/Right/Top/Bottom = 0** *(fills the canvas)*
  ⬜ **3.3** **Inspector (Image):**
* **Color =** light gray **(RGBA ~ #D9D9D9, Alpha 200/255)**

> Result: a gentle panel behind the grid for readability.

---

## 4) Create the Grid container (3×3) <span style="color:purple;">🟣</span>

⬜ **4.1** **Hierarchy:** Right-click **BoardCanvas** → **Create Empty** → rename **`Grid`**
⬜ **4.2** **Inspector (Grid → RectTransform):**

* **Anchor Preset = Stretch (full)**
* **Left/Right/Top/Bottom = 40** *(gives inner margin ~4–5 cm)*
  ⬜ **4.3** **Inspector (Grid):** **Add Component → Grid Layout Group**
* **Constraint = Fixed Column Count**
* **Constraint Count = 3**
* **Cell Size = (0.22, 0.22)** *(meters)*
* **Spacing = (0.02, 0.02)**
* **Child Alignment = Middle Center**
* **Start Axis = Horizontal**, **Start Corner = Upper Left**

> Result: any children under `Grid` auto-tile into a 3×3 layout.

---

## 5) Make **one** Cell as a proper UI Button (TMP) <span style="color:purple;">🟣</span>

⬜ **5.1** **Hierarchy:** Right-click **Grid** → **UI → Button (TextMeshPro)** → rename **`Cell`**
*(If TMP Essentials aren’t imported, Unity will prompt → click **Import TMP Essentials**.)*
⬜ **5.2** **Inspector (Cell → RectTransform):** **Leave to Grid** (don’t manually size)
⬜ **5.3** **Inspector (Cell → Button):**

* **Transition = Color Tint**
* **Normal =** white (#FFFFFF)
* **Highlighted =** off-white (#F3F3F3)
* **Pressed =** light gray (#E0E0E0)
* **Selected =** same as Highlighted
* **Disabled =** gray (#B0B0B0)

⬜ **5.4** **Child TMP Text** (inside `Cell`): select `Text (TMP)`

* **Text = (empty)** *(very important — gameplay will set “X”/“O” later)*
* **Font Size = 64** (start), **Auto Size OFF**
* **Alignment = Center**
* **Color = Black** (#000000)

⬜ **5.5** (Optional visual) Add a **stroke** frame around each cell

* **Cell → Add Component → Outline** (URP UI Outline works)
* **Effect Color = #000000 (50% alpha)**, **Effect Distance = (2, -2)** *(tweak as needed)*

---

## 6) Turn Cell into a **Prefab**, then make **9** cells <span style="color:purple;">🟣</span>

⬜ **6.1** **Project Window:** create folder **`Assets/Prefabs`** (if not already)
⬜ **6.2** **Hierarchy:** drag **`Cell`** **into** **`Assets/Prefabs`** → creates **`Cell.prefab`**
⬜ **6.3** **Hierarchy:** delete the scene copy of `Cell` (so Grid is empty)
⬜ **6.4** **Project → Prefabs:** drag **`Cell.prefab`** **into `Grid`** **nine** times
⬜ **6.5** **Rename** each instance (top-left to bottom-right) as:
`Cell_0`, `Cell_1`, `Cell_2`, `Cell_3`, `Cell_4`, `Cell_5`, `Cell_6`, `Cell_7`, `Cell_8`
*(This reading order matters — we’ll bind in Part 3.)*

> Result: a clean, prefab-based 3×3 grid; each cell is a **ray-clickable** Button with hover/press visuals.

---

## 7) (Optional, but recommended) Add placeholders for **Game Over** UI <span style="color:purple;">🟣</span>

*(We won’t wire logic yet — this sets exact objects for Part 3.)*

⬜ **7.1** **Hierarchy:** Right-click **BoardCanvas** → **UI → Panel** → rename **`GameOverPanel`**
⬜ **7.2** **Inspector (GameOverPanel):**

* **Anchor Preset = Stretch (full)**, **Left/Right/Top/Bottom = 0**
* **Image Color =** black **(#000000, Alpha ~ 140/255)** *(semi-transparent overlay)*
* **Active = OFF** (disable the GameObject for now)

⬜ **7.3** **Hierarchy:** Right-click **GameOverPanel** → **UI → Text (TextMeshPro)** → rename **`GameOverText`**

* **Text = “X wins!”** *(placeholder)*
* **Font Size = 96**, **Alignment = Center**
* **RectTransform:** center; **Width = 0.7**, **Height = 0.15** (meters)
* **Color = White**

⬜ **7.4** **Hierarchy:** Right-click **GameOverPanel** → **UI → Button (TextMeshPro)** → rename **`RestartButton`**

* **Text = “Play again?”**, **Font Size = 48**, **Alignment = Center**
* **RectTransform:** under the text; **Width = 0.35**, **Height = 0.10** (m)

> Result: a disabled overlay ready for Part 3 scripts to show/hide.

---

## 8) Ray test (no gameplay yet) <span style="color:purple;">🟣</span>

⬜ **8.1** **Enter Play** (Quest Link/AirLink recommended)
⬜ **8.2** Aim ray at cells → you should see **hover** color and **press** color when you pull trigger
⬜ **8.3** Cells won’t show “X/O” yet — that’s normal; we add scripts in **Part 3**.

---

## 9) What your **Hierarchy** should look like

```
OVRPlayerController
└─ OVRCameraRig
   └─ TrackingSpace
      ├─ LeftHandAnchor
      │  └─ ControllerInteractors
      │     ├─ Controller Ray Interactor
      │     └─ Grab Interactor
      └─ RightHandAnchor
         └─ ControllerInteractors
            ├─ Controller Ray Interactor
            └─ Grab Interactor

Wall
└─ BoardCanvas (World Space, OVR Raycaster, Layer: UIWorld, 0.8m × 0.8m)
   ├─ BoardPanel (stretched)
   ├─ Grid (Grid Layout Group: 3 columns, 0.22 cell, 0.02 spacing)
   │  ├─ Cell_0 (Prefab)
   │  ├─ Cell_1 ...
   │  └─ Cell_8
   ├─ GameOverPanel (disabled)
   │  ├─ GameOverText (TMP)
   │  └─ RestartButton (TMP Button)
   └─ (more polish later)
Ground
EventSystem (OVR Input Module)
Directional Light
```

---

## 10) Common mistakes & quick fixes

* **Ray won’t click cells:**

  * `BoardCanvas` **must** be **World Space** and have **OVR Raycaster**.
  * Scene **must** have **EventSystem (OVR Input Module)**, not Standalone.
* **Cells pile up strangely:**

  * Make sure **Grid** has **Grid Layout Group** with **Fixed Column Count = 3**.
  * **Cell Size/Spacing** are in **meters**.
* **Canvas looks too tiny/huge:**

  * Keep **Scale = (1,1,1)** on Canvas. Change **RectTransform Width/Height** (meters) instead.
* **GameOverPanel covers clicks now:**

  * Be sure **GameOverPanel is disabled** (unchecked). We enable it during win in Part 3.

---
---

# 🔹 Part 3 — Gameplay Logic (step-by-step code + wiring)

<span style="color:purple;font-weight:bold;">🟣   — complete every purple step</span>

**Goal:** Clicking a cell places **X/O**, detects **win/tie**, shows a **Game Over** overlay, and **Restart** works. We will build a robust, data-driven system where the game logic is separate from the UI.

**Where you’ll work:** `Assets/Scripts`, **Inspector** (BoardCanvas & Cells), **Hierarchy** (BoardCanvas/Grid), **Console** (optional logs).

> ✅ Pre-flight: From Part 2 you already have
> `Wall/BoardCanvas/Grid/Cell_0..Cell_8` (Button TMPs), and `BoardCanvas/GameOverPanel (disabled)/GameOverText/RestartButton`.

---

## 1) Create the Scripts folder & files <span style="color:purple;">🟣</span>

⬜ **1.1** **Project Window:** `Assets/Scripts` → Right-click → **Create → Folder** → name **`TicTacToe`**
⬜ **1.2** Inside it, create two scripts:

* **`GameController.cs`**
* **`Cell.cs`** (Note: this is a different name than the previous `Space.cs`)

---

## 2) Script #1 — `Cell.cs` (the behaviour on each cell) <span style="color:purple;">🟣</span>

> This script lives on **every Cell prefab**. It holds its **Button** and its **index (0-8)**. When clicked, it simply tells the `GameController` which cell was pressed. This approach simplifies wiring and centralizes logic.

### 2A) Paste this code

⬜ **2.1** Open `Cell.cs`, replace all with:

```csharp
using UnityEngine;
using UnityEngine.UI;

/// <summary>
/// Lives on each grid cell (the Button prefab).
/// It knows its own index (0-8) and notifies the GameController when clicked.
/// </summary>
[RequireComponent(typeof(Button))]
public class Cell : MonoBehaviour
{
    private Button _button;
    private GameController _gameController;
    private int _cellIndex;

    void Awake()
    {
        _button = GetComponent<Button>();
        _button.onClick.AddListener(OnCellClicked);
    }

    /// <summary>
    /// Called by the GameController during setup to establish a reference.
    /// </summary>
    public void Initialize(int index, GameController controller)
    {
        _cellIndex = index;
        _gameController = controller;
    }

    /// <summary>
    /// Handles the button click event.
    /// </summary>
    private void OnCellClicked()
    {
        // Notify the central controller that this cell was clicked.
        if (_gameController != null)
        {
            _gameController.OnCellClicked(_cellIndex);
        }
    }
}
```

### 2B) Put the script on the Cell Prefab

⬜ **2.2** **Project Window:** Go to `Assets/Prefabs` and double-click the **`Cell` prefab** to open it for editing.
⬜ **2.3** **Inspector (Cell prefab):** **Add Component → Cell**.
⬜ **2.4** Save the prefab. All nine instances in your scene will automatically get the script.
> **Note:** We do not need to wire the `OnClick()` event in the Inspector manually! The `Awake()` method in `Cell.cs` handles this automatically.

---

## 3) Script #2 — `GameController.cs` (the brain) <span style="color:purple;">🟣</span>

> This script lives on a **single** scene object (we’ll make a `GameManager`). It manages the entire game state: the board data, whose turn it is, win/tie conditions, and all UI updates.

### 3A) Paste this code

⬜ **3.1** Open `GameController.cs`, replace all with:

```csharp
using UnityEngine;
using UnityEngine.UI;
using TMPro;

/// <summary>
/// Central game manager.
/// - Manages the game state (board, turns, scores).
/// - Handles all UI updates (cell text, game over panel, HUD).
/// - Checks for win/tie conditions.
/// - Responds to cell clicks and button presses.
/// </summary>
public class GameController : MonoBehaviour
{
    [Header("Board & Cells")]
    [Tooltip("Drag the 9 Cell objects from the Hierarchy here, in order from 0 to 8.")]
    public Button[] cellButtons = new Button[9]; // The 9 cell buttons from the grid.

    [Header("Game State")]
    private int[] boardState = new int[9]; // 0=Empty, 1=X, 2=O
    private int currentPlayer = 1; // 1 for X, 2 for O
    private int movesMade = 0;
    private bool isGameActive = true;

    [Header("UI Panels & Text")]
    public GameObject gameOverPanel;
    public TMP_Text gameOverText;
    public Button restartButton;

    // Winning combinations (indices of the boardState array)
    private readonly int[][] winConditions =
    {
        new[] {0, 1, 2}, new[] {3, 4, 5}, new[] {6, 7, 8}, // Rows
        new[] {0, 3, 6}, new[] {1, 4, 7}, new[] {2, 5, 8}, // Columns
        new[] {0, 4, 8}, new[] {2, 4, 6}                  // Diagonals
    };

    void Start()
    {
        InitializeGame();
    }

    /// <summary>
    /// Sets up the initial state of the game.
    /// </summary>
    private void InitializeGame()
    {
        // 1. Setup Cell references
        for (int i = 0; i < cellButtons.Length; i++)
        {
            Cell cell = cellButtons[i].GetComponent<Cell>();
            if (cell != null)
            {
                cell.Initialize(i, this);
            }
        }

        // 2. Add listener for the restart button
        restartButton.onClick.AddListener(RestartGame);

        // 3. Start the first game
        RestartGame();
    }

    /// <summary>
    /// Resets the game to a fresh state.
    /// </summary>
    public void RestartGame()
    {
        isGameActive = true;
        currentPlayer = 1;
        movesMade = 0;

        for (int i = 0; i < boardState.Length; i++)
        {
            boardState[i] = 0; // Clear the internal board state
            UpdateCellUI(i);
            cellButtons[i].interactable = true;
        }

        gameOverPanel.SetActive(false);
    }

    /// <summary>
    /// Called by a Cell when it is clicked.
    /// </summary>
    public void OnCellClicked(int cellIndex)
    {
        if (!isGameActive || boardState[cellIndex] != 0)
        {
            return; // Ignore clicks if game is over or cell is taken
        }

        // 1. Update internal state
        boardState[cellIndex] = currentPlayer;
        movesMade++;

        // 2. Update the UI for the clicked cell
        UpdateCellUI(cellIndex);
        cellButtons[cellIndex].interactable = false;

        // 3. Check for win or tie
        if (CheckForWin())
        {
            EndGame(false);
        }
        else if (movesMade >= 9)
        {
            EndGame(true); // It's a tie
        }
        else
        {
            // 4. Switch player
            currentPlayer = (currentPlayer == 1) ? 2 : 1;
        }
    }

    /// <summary>
    /// Checks all win conditions.
    /// </summary>
    private bool CheckForWin()
    {
        foreach (var condition in winConditions)
        {
            if (boardState[condition[0]] == currentPlayer &&
                boardState[condition[1]] == currentPlayer &&
                boardState[condition[2]] == currentPlayer)
            {
                return true;
            }
        }
        return false;
    }

    /// <summary>
    /// Ends the current game and shows the game over UI.
    /// </summary>
    private void EndGame(bool isTie)
    {
        isGameActive = false;
        gameOverPanel.SetActive(true);

        if (isTie)
        {
            gameOverText.text = "It's a Tie!";
        }
        else
        {
            gameOverText.text = $"Player {(currentPlayer == 1 ? "X" : "O")} Wins!";
        }

        // Disable all cell buttons
        foreach (var button in cellButtons)
        {
            button.interactable = false;
        }
    }

    /// <summary>
    /// Updates the visual representation of a single cell.
    /// </summary>
    private void UpdateCellUI(int cellIndex)
    {
        TMP_Text textComponent = cellButtons[cellIndex].GetComponentInChildren<TMP_Text>();
        if (textComponent != null)
        {
            switch (boardState[cellIndex])
            {
                case 1:
                    textComponent.text = "X";
                    break;
                case 2:
                    textComponent.text = "O";
                    break;
                default:
                    textComponent.text = "";
                    break;
            }
        }
    }
}
```

---

## 4) Create a host object & wire `GameController` <span style="color:purple;">🟣</span>

⬜ **4.1** **Hierarchy:** Right-click → **Create Empty** → rename **`GameManager`**
⬜ **4.2** **Inspector (GameManager):** **Add Component → GameController**

### 4A) Fill the Cell Buttons array

⬜ **4.3** **Inspector (GameController on GameManager):**

* **Cell Buttons → Size = 9**
* Expand **BoardCanvas → Grid** in **Hierarchy**. For each index **(0..8)**, drag the corresponding **Cell_N** GameObject (which has the Button component) into the array slots:

| `Cell Buttons[index]` | Drag this from Hierarchy (the Button object) |
| --------------------- | ------------------------------------------ |
| 0                     | `BoardCanvas/Grid/Cell_0`                  |
| 1                     | `BoardCanvas/Grid/Cell_1`                  |
| 2                     | `BoardCanvas/Grid/Cell_2`                  |
| ...and so on...       | ...                                        |
| 8                     | `BoardCanvas/Grid/Cell_8`                  |

> ⚠️ **Order matters** (top-left to bottom-right, 0 to 8).

### 4B) Hook the Game Over UI

⬜ **4.4** Drag these from **Hierarchy** to **GameController** fields:

* **Game Over Panel =** `BoardCanvas/GameOverPanel`
* **Game Over Text =** `BoardCanvas/GameOverPanel/GameOverText`
* **Restart Button =** `BoardCanvas/GameOverPanel/RestartButton`

---

## 5) Playtest (ray → click → win/tie → restart) <span style="color:purple;">🟣</span>

1. **Enter Play**.
2. Use a controller **ray** to click **Cell_0**: it should display **“X”** and disable.
3. Click another cell: it should display **“O”** and disable.
4. Force a win (e.g., **X** on 0,1,2).

   * **GameOverPanel** appears with **“Player X Wins!”**, cells become non-clickable, **Restart** button is visible.
5. Click **Restart**: board clears, **X** starts again.

---

## 6) Troubleshooting (exact spots) <span style="color:purple;">🟣</span>

* **Click does nothing:**
  – On the **`GameManager`**, ensure the **`Cell Buttons`** array is filled with all 9 `Cell_` GameObjects in the correct order.
  – Make sure the **`Cell` prefab** has the **`Cell.cs`** script attached.
* **X/O not switching:**
  – This logic is now handled entirely inside `GameController.cs`. Check the console for any errors when clicking.
* **Win not detected:**
  – The `winConditions` array is hardcoded. This should not fail unless the `boardState` is not being updated correctly.
* **Restart not clearing:**
  – Ensure **RestartButton** is assigned on **GameController**. The `RestartGame()` method now handles all resetting logic.
* **GameOver never shows:**
  – **GameOverPanel** should be **disabled** initially and assigned to its field on the `GameController`.

---
---

# 🔹 Part 4 — VR UX Polish (Reticle, Hover/Press, Haptics, SFX)

<span style="color:purple;font-weight:bold;">🟣   — complete every purple step</span>

**Goal:**

* Reticle stays **crisp** and **constant size** in view.
* Cells give **visible hover/press** feedback (plus a gentle **scale-on-hover**).
* **Haptics** play when placing a mark and a stronger pulse on win.
* **Sounds** play on place and win.

**You will use:** **Hierarchy**, **Inspector**, **Project**, **Audio import**, **OVRCameraRig** (CenterEyeAnchor), small **C# slices**.

---

## 0) Pre-flight (must be true from Parts 1–3) <span style="color:purple;">🟣</span>

⬜ Rays click UI (you can place X/O).
⬜ `GameManager` has **GameController** with `cellButtons[0..8]` wired.
⬜ `BoardCanvas` exists with **OVR Raycaster** and the 3×3 **Grid**.
⬜ **OVRPlayerController** provides locomotion.

---

## 1) Reticle stays crisp & constant-size <span style="color:purple;">🟣</span>

### 1A) Script: `ReticleScaler.cs`

⬜ **1.1** **Project:** `Assets/Scripts/ReticleScaler.cs`

```csharp
using UnityEngine;

/// <summary>
/// Keeps the reticle ring a constant visual size:
/// - Always faces the HMD (CenterEye)
/// - Scales by distance so it looks the same apparent size
/// Attach to the ReticleRing prefab instance used by your Ray Interactors.
/// </summary>
public class ReticleScaler : MonoBehaviour
{
    [Header("Assign CenterEyeAnchor from OVRCameraRig")]
    public Transform cameraEye;

    [Header("How big should the ring look?")]
    public float sizeAt1Meter = 0.05f; // ring diameter (meters) when 1m away

    void LateUpdate()
    {
        if (!cameraEye) return;

        // Face the eye
        transform.rotation = Quaternion.LookRotation(transform.position - cameraEye.position);

        // Scale by distance so it looks constant
        float d = Vector3.Distance(transform.position, cameraEye.position);
        float s = Mathf.Max(0.001f, sizeAt1Meter * d);
        transform.localScale = new Vector3(s, s, s);
    }
}
```

### 1B) Wire it on your Reticle prefab

⬜ **1.2** **Project → Prefabs:** double-click **`ReticleRing`** to open prefab.
⬜ **1.3** **Inspector (ReticleRing):** **Add Component → ReticleScaler**.
⬜ **1.4** **Assign `cameraEye`:**

* **Hierarchy (inside scene)**: expand **`OVRPlayerController/OVRCameraRig/TrackingSpace/CenterEyeAnchor`**
* Drag **`CenterEyeAnchor`** from **Hierarchy** into the **ReticleScaler → Camera Eye** slot **in the prefab inspector**

  > If Unity won’t accept a scene object into a prefab field, drop a **ReticleRing** instance into the scene, assign the field there, then **Apply** to the prefab.
  > ⬜ **1.5** **Save/Apply** the prefab, close prefab view.

> ✅ Result: your reticle always faces the HMD and keeps a consistent apparent size at any distance.

---

## 2) Better hover/press feedback on cells <span style="color:purple;">🟣</span>

### 2A) Improve Color Tint on the Button

⬜ **2.1** **Project → Prefabs:** Open the **`Cell` prefab**.
⬜ **2.2** **Inspector (Button → Colors):**

* **Normal:** #FFFFFF
* **Highlighted:** #F3F3F3
* **Pressed:** #E0E0E0
* **Selected:** #F3F3F3
* **Disabled:** #B0B0B0
* **Fade Duration:** **0.06** (snappy)

### 2B) Subtle scale on hover (script)

⬜ **2.3** **Project:** `Assets/Scripts/CellHoverScale.cs`

```csharp
using UnityEngine;
using UnityEngine.EventSystems;

/// <summary>
/// Adds a tiny scale-up when the ray hovers a cell, scale-back on exit.
/// Works with OVR Input Module (pointer enter/exit).
/// Attach to each Cell (same object as Button).
/// </summary>
public class CellHoverScale : MonoBehaviour, IPointerEnterHandler, IPointerExitHandler
{
    public float hoverScale = 1.05f;
    public float speed = 16f;
    Vector3 _base;

    void Awake() => _base = transform.localScale;

    public void OnPointerEnter(PointerEventData e) { StopAllCoroutines(); StartCoroutine(ScaleTo(_base * hoverScale)); }
    public void OnPointerExit(PointerEventData e)  { StopAllCoroutines(); StartCoroutine(ScaleTo(_base)); }

    System.Collections.IEnumerator ScaleTo(Vector3 target)
    {
        while ((transform.localScale - target).sqrMagnitude > 0.000001f)
        {
            transform.localScale = Vector3.Lerp(transform.localScale, target, Time.deltaTime * speed);
            yield return null;
        }
        transform.localScale = target;
    }
}
```

⬜ **2.4** In the **`Cell` prefab**, **Add Component → CellHoverScale** (keep defaults). Save the prefab.

> ✅ Result: as you hover a cell with the ray, it pops up 5% then settles back when you leave.

---

## 3) Haptics on place & win (Meta/OVR) <span style="color:purple;">🟣</span>

We’ll play a tiny pulse on **every move**, and a stronger one on **win**. We'll add all this logic directly into `GameController.cs`.

### 3A) Add a haptics helper inside `GameController.cs`

⬜ **3.1** Open **`GameController.cs`**. **Add** the following *inside* the class (anywhere below fields is fine):

```csharp
// ===== HAPTICS (OVRInput) =====
private System.Collections.IEnumerator Pulse(float amplitude, float duration)
{
    // Set both controllers; frequency is ignored on Quest, amplitude [0..1].
    OVRInput.SetControllerVibration(1f, amplitude, OVRInput.Controller.LTouch);
    OVRInput.SetControllerVibration(1f, amplitude, OVRInput.Controller.RTouch);
    yield return new WaitForSeconds(duration);
    OVRInput.SetControllerVibration(0f, 0f, OVRInput.Controller.LTouch);
    OVRInput.SetControllerVibration(0f, 0f, OVRInput.Controller.RTouch);
}
```

### 3B) Call the helper from the right places

⬜ **3.2** In **`GameController.cs`**, find the **`OnCellClicked`** method. **Add** this line right after checking if the move is valid:

```csharp
// Inside OnCellClicked, after the `if (!isGameActive ...)` check:
StartCoroutine(Pulse(0.15f, 0.06f)); // Play click haptics
```

⬜ **3.3** Find the **`EndGame`** method. **Add** this line to play a stronger pulse on win/tie:

```csharp
// Inside EndGame, after setting the gameOverText:
StartCoroutine(Pulse(0.4f, 0.12f)); // Play stronger haptics for game end
```

> ✅ Result: every valid click gives a short vibration, and the end of a game gives a stronger one.

---

## 4) Simple audio: click & win <span style="color:purple;">🟣</span>

We’ll keep audio centralized in **GameController** so it’s easy to wire.

### 4A) Import two clips

⬜ **4.1** **Project:** make folder `Assets/Audio`
⬜ **4.2** Drop in 2 short .wav/.mp3 files (e.g., `click.wav`, `win.wav`)
⬜ **4.3** Select each clip → **Inspector (AudioClip Import Settings):**

* **Load Type = Decompress On Load**
* **Compression = Off or Low** (they’re tiny)
* **Normalize OFF**

### 4B) Add AudioSource on GameManager

⬜ **4.4** **Hierarchy:** select **`GameManager`** → **Add Component → Audio Source**

* **Output:** (leave empty)
* **Spatial Blend = 0.0 (2D)**
* **Play On Awake = OFF**
* **Loop = OFF**

### 4C) Add audio fields + play helpers in `GameController.cs`

⬜ **4.5** Open **`GameController.cs`**. Add these **fields** near your other `[Header]` fields:

```csharp
[Header("Audio")]
public AudioSource audioSource;    // drag GameManager's AudioSource here
public AudioClip clickClip;        // Assets/Audio/click.wav
public AudioClip winClip;          // Assets/Audio/win.wav
```

⬜ **4.6** Also **add** these two **methods** *inside* the class:

```csharp
private void PlayClickSound()
{
    if (audioSource && clickClip) audioSource.PlayOneShot(clickClip, 0.8f);
}

private void PlayWinSound()
{
    if (audioSource && winClip) audioSource.PlayOneShot(winClip, 1.0f);
}
```

⬜ **4.7** **Hook calls**:

* In **`OnCellClicked()`**, add a call to `PlayClickSound()` right after the haptics call.
* In **`EndGame()`**, add a call to `PlayWinSound()` right after the haptics call.

### 4D) Wire references in the Inspector

⬜ **4.8** **Hierarchy:** select **`GameManager`** → **GameController (component)**

* **Audio Source:** drag the **`GameManager`**'s own **Audio Source** component here.
* **Click Clip:** drag **`Assets/Audio/click.wav`**
* **Win Clip:** drag **`Assets/Audio/win.wav`**

> ✅ Result: each move plays a click sound; winning plays a celebration sound + stronger buzz.

---

## 5) Playtest (what you should see/hear/feel) <span style="color:purple;">🟣</span>

1. **Enter Play** (Quest Link/AirLink).
2. Aim at a cell → it **brightens** (Highlighted) and **scales up slightly**.
3. Click → **click sound** + **short vibration**, X/O appears, cell disables.
4. Force a win → **overlay appears**, **win sound**, **stronger vibration**.
5. **Restart** still works.

---

## 6) Troubleshooting (exact places to check) <span style="color:purple;">🟣</span>

* **No reticle changes size:**
  – `ReticleScaler` must be on **ReticleRing**; **Camera Eye** must reference **CenterEyeAnchor**.
* **Hover scale doesn’t work:**
  – `CellHoverScale` must be on the **`Cell` prefab**. Ensure **OVR Input Module** is in the scene.
* **No haptics:**
  – Are you in headset focus (Link/AirLink active)? The **Pulse** coroutine is in **GameController** and should be called from **OnCellClicked** and **EndGame**.
* **No audio:**
  – **AudioSource** must be on **GameManager**; clips assigned on **GameController**; device volume up.
* **Audio very quiet:**
  – Increase the **AudioSource Volume** or `PlayOneShot` volume (the second arg).
* **UI stops highlighting after win:**
  – Normal: we disable buttons on game over. Restart to re-enable.

---
---

# 🔹 Part 5 — Handedness Filters (Left = 3D only, Right = UI only)

<span style="color:purple;font-weight:bold;">🟣   — complete every purple step</span>

**Goal:**

* Left hand ray only affects **3D** (grabs/interacts).
* Right hand ray only affects **UI** (buttons), and does **not** interact with 3D.

**You will use:** **Hierarchy**, **Inspector** for **EventSystem (OVR Input Module)**, **BoardCanvas (OVR Raycaster)**, **Left/Right Controller Ray Interactors**, and (optional) **Tag Set / Tag Set Filter** on 3D hosts.

---

## 0) Pre-flight (from Parts 1–4) <span style="color:purple;">🟣</span>

⬜ **0.1** You have **OVRPlayerController** at root (no object named “XR”).
⬜ **0.2** **OVRCameraRig** inside **OVRPlayerController** with **LeftHandAnchor** and **RightHandAnchor**.
⬜ **0.3** Each hand has child **ControllerInteractors** containing:

* **Controller Ray Interactor** (visual ray you already added)
* **Grab Interactor** (near-grab)
  ⬜ **0.4** **EventSystem** exists with **OVR Input Module** (not Standalone).
  ⬜ **0.5** **BoardCanvas** is **World Space** with **OVR Raycaster** and your Tic-Tac-Toe board.
  ⬜ **0.6** You can currently **play the game** by ray-clicking cells.

---

## 1) Decide your 3D “host” object(s) to tag (optional but recommended) <span style="color:purple;">🟣</span>

> For teaching juniors: we’ll tag 3D interactable **hosts** with `3DObject`.
> (UI won’t use Tag Set—the UI path is OVR Input Module.)

⬜ **1.1** If you used the **wizard** to add ray grab to a cube:

* The **host** is the **parent** created by the wizard (e.g., `ISDK Ray Grab Interaction`) with the interaction components on it.
* **Hierarchy (example):**
  `ISDK Ray Grab Interaction`
   └─ `GrabCube` (mesh as child)

⬜ **1.2** If you set up 3D manually (from Part 1):

* The **host** is **the same GameObject** that has **RayInteractable + Grabbable + ColliderSurface** (your `GrabCube` itself).

⬜ **1.3** **Inspector (host object):** **Add Component → Tag Set**

* In **Tag Set**, add the tag string: **`3DObject`**
* *(This Tag Set is used by Meta interactable filters for 3D only; UI ignores it.)*

---

## 2) EventSystem — tell UI to listen to **Right** hand only <span style="color:purple;">🟣</span>

> UI clicks are generated by **OVR Input Module** using per-hand **Ray Transform**.
> To make **Left hand ignore UI**, we **clear Left Ray Transform**.
> To make **Right hand do UI only**, we **assign Right Ray Transform**.

⬜ **2.1** **Hierarchy:** select **EventSystem**
⬜ **2.2** **Inspector (OVR Input Module):** find fields like:

* **Ray Transform (Left)** → **set to None** *(drag the field to empty)*
* **Ray Transform (Right)** → **drag** `OVRPlayerController/OVRCameraRig/TrackingSpace/RightHandAnchor`
* **Submit on Button Down = ON** (as before)

> ✅ Result: **Left hand no longer produces UI pointer events**. **Right hand** continues to click UI.

---

## 3) Left controller = 3D only (keep its 3D Interactor alive) <span style="color:purple;">🟣</span>

⬜ **3.1** **Hierarchy Path:**
`OVRPlayerController/OVRCameraRig/TrackingSpace/LeftHandAnchor/ControllerInteractors/Controller Ray Interactor`
⬜ **3.2** **Inspector (Controller Ray Interactor):**

* **Hide When No Interactable = ON** *(your choice; this hand won’t do UI anyway)*
* **Interaction/Collision Mask**: include your 3D layers (e.g., **Default**, **Grabbable**, etc.). **Do NOT** include **UIWorld**.
  *(We set UIWorld earlier on BoardCanvas.)*

### (Optional but recommended) Tag-based filter for 3D

⬜ **3.3** **Add Component → Tag Set Filter** **on the same** “Controller Ray Interactor” object **or** on **LeftHandAnchor** (referenced below).

* **Required Tag =** `3DObject`
* **Exclude Tag =** *(leave empty)*

⬜ **3.4** In **Controller Ray Interactor → Interactable Filters (list)**:

* Click **+** → drag the **Tag Set Filter** component you just added (from current object or LeftHandAnchor).

> ✅ Result: Left ray will **only** select Meta Interactables whose **host** has **Tag Set = 3DObject** and will **never** click UI because UI events from left were turned off in **OVR Input Module**.

---

## 4) Right controller = UI only (no 3D interaction) <span style="color:purple;">🟣</span>

⬜ **4.1** **Hierarchy Path:**
`OVRPlayerController/OVRCameraRig/TrackingSpace/RightHandAnchor/ControllerInteractors/Controller Ray Interactor`

⬜ **4.2** **Inspector (Controller Ray Interactor):**

* **Hide When No Interactable = OFF**
  *(We want a visible ray for UI even when there’s no 3D interactable.)*
* **Interaction/Collision Mask**: set to **UIWorld only** *(or even to “Nothing” if allowed)*

  * This ensures the 3D interactor **never** hits 3D colliders.
* **Interactable Filters list**: **clear** it (no Tag filter needed).

> ✅ Result: Right controller still shows the ray/reticle but effectively **cannot** select 3D Meta Interactables; **UI clicks** work because **OVR Input Module** uses **RightHandAnchor** as the only UI pointer.

---

## 5) Board Canvas sanity (so UI clicks are clean) <span style="color:purple;">🟣</span>

⬜ **5.1** **Hierarchy:** select **`BoardCanvas`**
⬜ **5.2** **Inspector:**

* **Layer = UIWorld** (already set in Part 1/2)
* **Canvas → Render Mode = World Space**
* **OVR Raycaster** is present
* **Graphic Raycaster** can stay enabled (fine together)

> If you still have the **TestCanvas** from Part 1, either:

* **Disable** it, or
* Set its **Layer = UIWorld** and keep for testing; Right hand will click it; Left will pass through.

---

## 6) Playtest (the expected behavior) <span style="color:purple;">🟣</span>

1. **Enter Play.**
2. **Left hand**:

   * Try to click cells → **nothing happens** (UI ignored).
   * Aim at 3D (e.g., **GrabCube**) → can **ray grab / near grab** as before.
3. **Right hand**:

   * Aim at the board → can **click cells** and play the game.
   * Try to grab 3D → **nothing happens** (Interactor filtered out).

---

## 7) Troubleshooting (exact spots) <span style="color:purple;">🟣</span>

* **Left still clicks UI:**

  * **EventSystem → OVR Input Module → Ray Transform (Left) must be None**.
  * If you have multiple EventSystems, remove extras (only **one** should exist).
* **Right still grabs 3D:**

  * **Right Controller Ray Interactor → Interaction/Collision Mask** must **not** include 3D layers (Default/Grabbable). Use **UIWorld only** or **Nothing**.
  * If you also added Tag filters on right by mistake, remove them.
* **Left doesn’t grab 3D anymore:**

  * Ensure **Left Controller Ray Interactor** mask **includes** 3D layers.
  * If you used Tag Set Filter, verify your 3D **host** has **Tag Set = 3DObject** (on the **interactable host**, not just the mesh).
* **UI clicks don’t work at all now:**

  * **EventSystem → OVR Input Module → Ray Transform (Right)** must be **RightHandAnchor**.
  * **BoardCanvas** must have **OVR Raycaster**; cells are **Buttons**; no invisible panel is blocking (disable `GameOverPanel` while testing).
* **Right ray disappeared:**

  * You set **Hide When No Interactable = OFF** on **Right Controller Ray Interactor**, correct?
  * If mask = Nothing and it still hides, add **UIWorld** layer to the mask (harmless and keeps the beam visible).

---

## 8) Deliverables (submit these)

* Screenshot: **EventSystem (OVR Input Module)** showing **Left Ray = None** and **Right Ray = RightHandAnchor**.
* Screenshot: **Right Controller Ray Interactor** with **Hide When No Interactable = OFF** and mask = **UIWorld** only.
* Screenshot: **Left Controller Ray Interactor** with mask including **3D** layers (and **Tag Set Filter** attached & referenced if you used it).
* Short clip: demonstrate **left can grab 3D** but **can’t click UI**, and **right can click UI** but **can’t interact with 3D**.

---
---

# 🔹 Part 6/6 — HUD (Turn/Score), Rounds, Undo, Reset, Debug Overlay

<span style="color:purple;font-weight:bold;">🟣   — complete every purple step</span>

**Goal:**

* Show **whose turn** it is.
* Track **score** for **X** and **O** across **rounds**.
* Add **Undo** (1 move back at a time).
* Add **Reset All** (scores + board).
* Add a **Debug panel** to help juniors self-diagnose wiring.

---

## 0) Pre-flight (from Parts 1–5) <span style="color:purple;">🟣</span>

⬜ Rays click cells; game plays; **GameOverPanel** appears on win/tie; **RestartButton** restarts the board.
⬜ **GameManager** hosts **GameController** and has `cellButtons[0..8]` assigned.
⬜ You have **OVRPlayerController** (root), **OVR Input Module** on **EventSystem**, **OVR Raycaster** on **BoardCanvas**.

---

## 1) Build the HUD (Turn / Score / Round) <span style="color:purple;">🟣</span>

We’ll place a thin HUD bar at the **top** of the BoardCanvas.

⬜ **1.1** **Hierarchy:** Right-click **`BoardCanvas`** → **UI → Panel** → rename **`HUDPanel`**
⬜ **1.2** **Inspector (HUDPanel → RectTransform):**

* **Anchor Preset = Top Stretch** (hold Alt + click the top-stretch preset)
* **Left/Right = 8**, **Top = 8**, **Height = 0.12** (meters)
* **Image Color =** #000000 with **Alpha = 140/255**

⬜ **1.3** **Turn text**

* **HUDPanel** → Right-click → **UI → Text (TextMeshPro)** → rename **`TurnText`**
* **RectTransform:** **Left=12, Right=**, **Width=0.28**, **Height=0.10**, **Pos Y = -0.02**
* **TMP:** **Text = “Turn: X”**, **Font Size = 56**, **Alignment = Left**, **Color = White**

⬜ **1.4** **Score text X**

* **HUDPanel** → add **Text (TMP)** → rename **`ScoreXText`**
* **RectTransform:** **Anchor = Middle**, **Width=0.24**, **Pos X = 0.0**, **Pos Y = -0.02**
* **TMP:** **Text = “X: 0”**, **Font Size = 56**, **Alignment = Center**, **Color = #FFD369** (warm)

⬜ **1.5** **Score text O**

* Duplicate **ScoreXText** → rename **`ScoreOText`**
* **RectTransform:** **Pos X = 0.22**
* **TMP:** **Text = “O: 0”**, **Color = #69C0FF**

⬜ **1.6** **Round text**

* **HUDPanel** → add **Text (TMP)** → rename **`RoundText`**
* **RectTransform:** **Anchor Right**, **Right=12**, **Width=0.22**, **Pos Y=-0.02**
* **TMP:** **Text = “Round: 1”**, **Font Size = 56**, **Alignment = Right**, **Color = White**

---

## 2) Add action buttons (Undo & Reset All) <span style="color:purple;">🟣</span>

⬜ **2.1** **Undo**

* **HUDPanel** → **UI → Button (TextMeshPro)** → rename **`UndoButton`**
* **RectTransform:** **Anchor Left**, **Left=0.34**, **Width=0.14**, **Height=0.08**, **Pos Y=-0.02**
* **TMP (child):** **Text = “Undo”**, **Font Size = 44**
* **Button → Colors:** Highlighted brighter; **Fade Duration = 0.06**

⬜ **2.2** **Reset All**

* Duplicate **UndoButton** → rename **`ResetAllButton`**
* **RectTransform:** **Left=0.50**
* **TMP:** **Text = “Reset”**

⬜ **2.3** **Rename the existing RestartButton label**

* **Hierarchy:** `BoardCanvas/GameOverPanel/RestartButton/Text (TMP)` → set **Text = “New Round”**
  *(We’ll keep the field name `restartButton` in code.)*

---

## 3) Code slices — extend `GameController` for HUD/Undo/Score/Rounds <span style="color:purple;">🟣</span>

### 3A) Open `GameController.cs`

Add these **new fields** near the top (with your other `[Header]` blocks):

```csharp
[Header("HUD UI")]
public TMP_Text turnText;
public TMP_Text scoreXText;
public TMP_Text scoreOText;
public TMP_Text roundText;
public Button undoButton;
public Button resetAllButton;

[Header("Game Stats")]
private int scoreX = 0;
private int scoreO = 0;
private int currentRound = 1;

// For the Undo feature
private System.Collections.Generic.Stack<int> moveHistory = new System.Collections.Generic.Stack<int>();
```

### 3B) Update `InitializeGame()` to hook up new buttons

Find `InitializeGame()` and add listeners for the Undo and Reset All buttons:

```csharp
// Inside InitializeGame(), after wiring the restartButton:
undoButton.onClick.AddListener(UndoLastMove);
resetAllButton.onClick.AddListener(ResetAllStats);
```

### 3C) Create the HUD updater method

Add this new method inside the class to centralize all HUD text updates:

```csharp
private void UpdateHUD()
{
    if (turnText) turnText.text = isGameActive ? $"Turn: {(currentPlayer == 1 ? "X" : "O")}" : "Game Over";
    if (scoreXText) scoreXText.text = $"X: {scoreX}";
    if (scoreOText) scoreOText.text = $"O: {scoreO}";
    if (roundText) roundText.text = $"Round: {currentRound}";

    // The Undo button should only be active during a game and if there are moves to undo.
    if (undoButton) undoButton.interactable = isGameActive && moveHistory.Count > 0;
}
```

### 3D) Call `UpdateHUD()` at the right times

- At the end of **`RestartGame()`**.
- At the end of **`OnCellClicked()`** (after switching the player).
- At the end of **`EndGame()`**.
- At the end of **`UndoLastMove()`** (we'll create this next).
- At the end of **`ResetAllStats()`** (we'll create this next).

### 3E) Add `Undo`, `ResetAll`, and `NewRound` logic

Add these three new methods to `GameController.cs`:

```csharp
public void UndoLastMove()
{
    if (moveHistory.Count == 0 || !isGameActive) return;

    int lastMoveIndex = moveHistory.Pop();

    // Revert game state
    boardState[lastMoveIndex] = 0;
    movesMade--;
    currentPlayer = (currentPlayer == 1) ? 2 : 1; // Switch turn back

    // Update UI
    UpdateCellUI(lastMoveIndex);
    cellButtons[lastMoveIndex].interactable = true;

    UpdateHUD();
    StartCoroutine(Pulse(0.1f, 0.05f)); // Gentle confirmation haptic
}

public void ResetAllStats()
{
    scoreX = 0;
    scoreO = 0;
    currentRound = 1;
    RestartGame(); // This will reset the board and update the HUD
}

// Rename RestartGame to NewRound for clarity, and have the old RestartGame call it.
// This keeps the existing button wiring from Part 3 working.
public void RestartGame()
{
    isGameActive = true;
    currentPlayer = 1;
    movesMade = 0;
    moveHistory.Clear();

    for (int i = 0; i < boardState.Length; i++)
    {
        boardState[i] = 0;
        UpdateCellUI(i);
        cellButtons[i].interactable = true;
    }

    gameOverPanel.SetActive(false);
    UpdateHUD();
}
```

### 3F) Update `OnCellClicked()` and `EndGame()`

- In `OnCellClicked()`, before updating the board state, push the move to the history stack: `moveHistory.Push(cellIndex);`
- In `EndGame()`, if it's not a tie, award points to the winner:
  ```csharp
  // Inside EndGame()
  if (!isTie)
  {
      if (currentPlayer == 1) scoreX++;
      else scoreO++;
  }
  ```
- In `EndGame()`, also add a call to `UpdateHUD()` at the end.
- The `RestartButton` on the `GameOverPanel` should now be for starting a *new round*. Find where you add its listener in `InitializeGame()` and change it to call a new method `StartNewRound()`.

```csharp
// New method in GameController
public void StartNewRound()
{
    currentRound++;
    RestartGame(); // This resets the board and HUD
}
// In InitializeGame(), change the listener:
restartButton.onClick.AddListener(StartNewRound);
```

---

## 4) Wire HUD & Buttons in the Inspector <span style="color:purple;">🟣</span>

⬜ **4.1** **Hierarchy:** select **`GameManager`** → **GameController** (component)

* **Turn Text:** drag `BoardCanvas/HUDPanel/TurnText`
* **Score X Text:** drag `BoardCanvas/HUDPanel/ScoreXText`
* **Score O Text:** drag `BoardCanvas/HUDPanel/ScoreOText`
* **Round Text:** drag `BoardCanvas/HUDPanel/RoundText`
* **Undo Button:** drag `BoardCanvas/HUDPanel/UndoButton`
* **Reset All Button:** drag `BoardCanvas/HUDPanel/ResetAllButton`

---

## 5) Debug Overlay (helps juniors see state) <span style="color:purple;">🟣</span>

⬜ **5.1** **Hierarchy:** `BoardCanvas` → **UI → Panel** → rename **`DebugPanel`**

* **RectTransform:** **Bottom stretch**, **Left/Right=8**, **Bottom=8**, **Height=0.16**
* **Image:** #000000 with **Alpha 120/255**
* **Active = OFF** (start hidden)

⬜ **5.2** **Debug text**

* **DebugPanel** → **UI → Text (TextMeshPro)** → rename **`DebugText`**
* **TMP:** **Font Size = 40**, **Alignment = Top Left**, **Text = “(debug…)”**

⬜ **5.3** **Toggle button**

* **HUDPanel** → **UI → Button (TextMeshPro)** → rename **`DebugToggle`**
* **RectTransform:** **Anchor Right**, **Right=0.10**, **Width=0.12**, **Height=0.08**, **Pos Y=-0.02**
* **TMP:** **Text = “Debug”**, **Font Size=40**

⬜ **5.4** **Project:** `Assets/Scripts/DebugOverlay.cs`

```csharp
using UnityEngine;
using TMPro;
using System.Reflection; // Needed for reflection

public class DebugOverlay : MonoBehaviour
{
    public GameController controller; // drag GameManager here
    public GameObject panel;          // drag BoardCanvas/DebugPanel
    public TMP_Text debugText;        // drag BoardCanvas/DebugPanel/DebugText

    void Update()
    {
        if (!controller || !debugText || !panel.activeSelf) return;

        // Use reflection to get private field values from GameController
        string turn = GetPrivateField<int>(controller, "currentPlayer") == 1 ? "X" : "O";
        int moves = GetPrivateField<int>(controller, "movesMade");
        int scoreX = GetPrivateField<int>(controller, "scoreX");
        int scoreO = GetPrivateField<int>(controller, "scoreO");
        int round = GetPrivateField<int>(controller, "currentRound");
        bool active = GetPrivateField<bool>(controller, "isGameActive");

        debugText.text =
            $"Turn: {turn} | Moves: {moves}\n" +
            $"Score: X={scoreX}, O={scoreO} | Round: {round}\n" +
            $"Game Active: {active}";
    }

    public void Toggle()
    {
        if (!panel) return;
        panel.SetActive(!panel.activeSelf);
    }

    // Helper to access private fields for debugging
    private T GetPrivateField<T>(object instance, string fieldName)
    {
        BindingFlags bindFlags = BindingFlags.Instance | BindingFlags.NonPublic;
        FieldInfo field = instance.GetType().GetField(fieldName, bindFlags);
        return (T)field.GetValue(instance);
    }
}
```

⬜ **6.5** **Hierarchy:** select **`BoardCanvas`** → **Add Component → DebugOverlay**

* **Controller:** drag **`GameManager`**
* **Panel:** drag **`BoardCanvas/DebugPanel`**
* **Debug Text:** drag **`BoardCanvas/DebugPanel/DebugText`**

⬜ **6.6** **Wire the toggle**

* Select **`HUDPanel/DebugToggle`** → **Button (OnClick)** → **+**
* Drag **`BoardCanvas`** → choose **`DebugOverlay.Toggle()`**

> ✅ Result: tap **Debug** → panel appears with basic state; tap again to hide.

---

## 7) Playtest Checklist <span style="color:purple;">🟣</span>

1. **Turn HUD** updates between **X** and **O** each click.
2. **Score** updates after a win; **Round** increments when you click **New Round**.
3. **Undo**: click a cell → **Undo** → mark disappears, cell re-enables, turn swaps back.
4. **Reset**: sets **Round to 1**, **scores to 0**, clears board, starts with **X**.
5. **Debug** button toggles the Debug panel.

---

## 8) Troubleshooting (exact spots) <span style="color:purple;">🟣</span>

* **Undo does nothing:**
  – Make sure the `moveHistory` stack is being pushed to in `OnCellClicked`.
  – Confirm the `UndoButton` is assigned and its `OnClick` is wired to `GameController.UndoLastMove()`.
* **Turn HUD not changing:**
  – Did you call `UpdateHUD()` from all the required methods? (`RestartGame`, `OnCellClicked`, `EndGame`, `UndoLastMove`, `ResetAllStats`).
* **Score doesn’t increase on win:**
  – Make sure the score logic inside `EndGame` is correctly implemented.
* **Buttons not firing:**
  – Check all HUD buttons in the Inspector to ensure their `OnClick()` events are wired to the correct methods on the `GameController`.
* **Debug panel shows errors or is empty:**
  – `BoardCanvas` must have the `DebugOverlay` script. Its three fields must be assigned. The reflection code assumes the private field names in `GameController` are correct (`currentPlayer`, `movesMade`, etc.).

---

## ✅ End-of-Course Deliverables

* **Video** showing: playing a round, **win**, **New Round**, **Undo**, **Reset**, **Debug toggle**, plus left/right-hand behavior from Part 5.
* **Screenshots**: GameController with HUD fields assigned; HUDPanel hierarchy; DebugOverlay fields assigned.

---

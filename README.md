# Session-9-Tic-Tac-Toe-UI-Snap-
# 🔹 Part 1 — Project & Rig Foundation (Meta-only) + Locomotion + Grab <span style="color:purple;font-weight:bold;">🟣 Session Homework</span>

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

<span style="color:purple;font-weight:bold;">🟣 Session Homework — complete every purple step</span>

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

<span style="color:purple;font-weight:bold;">🟣 Session Homework — complete every purple step</span>

**Goal:** Clicking a cell places **X/O**, detects **win/tie**, shows a **Game Over** overlay, and **Restart** works.
**Where you’ll work:** `Assets/Scripts`, **Inspector** (BoardCanvas & Cells), **Hierarchy** (BoardCanvas/Grid), **Console** (optional logs).

> ✅ Pre-flight: From Part 2 you already have
> `Wall/BoardCanvas/Grid/Cell_0..Cell_8` (Button TMPs), and `BoardCanvas/GameOverPanel (disabled)/GameOverText/RestartButton`.

---

## 0) Safety checks (must be true before you start) <span style="color:purple;">🟣</span>

⬜ **0.1** You can hover & press the **Grid cells** with the ray (they highlight).
⬜ **0.2** **GameOverPanel** exists but is **disabled** (unchecked in Hierarchy).
⬜ **0.3** Each **Cell** has a **Button (TMP)** child with **Text (TMP)**.

> If not, fix Part 2 first.

---

## 1) Create the Scripts folder & files <span style="color:purple;">🟣</span>

⬜ **1.1** **Project Window:** `Assets/Scripts` → Right-click → **Create → Folder** → name **`TicTacToe`**
⬜ **1.2** Inside it, create two scripts:

* **`GameController.cs`**
* **`Space.cs`**

---

## 2) Script #1 — `Space.cs` (the behaviour on each cell) <span style="color:purple;">🟣</span>

> This script lives on **every Cell**. It knows its **Button**, its **TMP text**, and it calls the **GameController** when clicked.

### 2A) Paste this code

⬜ **2.1** Open `Space.cs`, replace all with:

```csharp
using UnityEngine;
using UnityEngine.UI;
using TMPro;

/// <summary>
/// Lives on each grid cell (the Button). Handles a click:
/// - Writes X or O into its TMP label
/// - Disables its Button so it can't be clicked twice
/// - Notifies GameController to process the turn
/// </summary>
public class Space : MonoBehaviour
{
    [Header("Assign in Inspector (on the Cell prefab/instances)")]
    public Button button;         // The UI Button on this Cell
    public TMP_Text buttonText;   // The TMP label child of this Cell

    private GameController gameController;

    /// <summary> Called by GameController once at start to connect references. </summary>
    public void SetControllerReference(GameController control)
    {
        gameController = control;
    }

    /// <summary> Hook this to the Button's OnClick() in the Inspector. </summary>
    public void SetSpace()
    {
        // Safety: if wiring is missing, do nothing.
        if (gameController == null || buttonText == null || button == null)
            return;

        // If already marked (X or O), ignore second clicks.
        if (!string.IsNullOrEmpty(buttonText.text))
            return;

        // Write the current player's mark and lock this cell.
        buttonText.text = gameController.GetSide();
        button.interactable = false;

        // Tell the controller to evaluate win/tie and/or swap turns.
        gameController.EndTurn();
    }
}
```

### 2B) Put the script on all Cells (one action to rule them all)

⬜ **2.2** **Hierarchy:** Select **all nine** `Cell_0..Cell_8`
⬜ **2.3** **Inspector:** **Add Component → Space**
⬜ **2.4** Still selected, drag their **own Button** component into **Space → Button**.
⬜ **2.5** Still selected, expand each Cell → child **`Text (TMP)`** and drag it into **Space → Button Text**.

> Tip: If the Cells are prefab instances, you can **Open the `Cell` prefab** and add **Space** once there (with Button/TMP assigned), then **Apply** so all instances inherit it.

### 2C) Wire the click event

⬜ **2.6** Select **all nine** Cells again → in **Button (component) → OnClick()**

* Click **+**, drag **the same Cell** (the one with **Space**) into the object field
* Choose **`Space → SetSpace()`**

> ✅ Now each Cell knows how to write **X/O** when clicked and to notify the controller. Next, we build the controller.

---

## 3) Script #2 — `GameController.cs` (the brain) <span style="color:purple;">🟣</span>

> This lives on a **single** scene object (we’ll make **SceneManager**). It holds references to the **9 cell labels**, the **GameOver** UI, and exposes simple methods the `Space` script calls.

### 3A) Paste this code

⬜ **3.1** Open `GameController.cs`, replace all with:

```csharp
using UnityEngine;
using UnityEngine.UI;
using TMPro;

/// <summary>
/// Central game manager:
/// - Knows the 9 cell labels (TMP_Text)
/// - Tracks whose turn it is ("X" or "O")
/// - Checks win/tie after each move
/// - Shows GameOver UI and handles Restart
/// </summary>
public class GameController : MonoBehaviour
{
    [Header("Board cell labels (size = 9, drag Text(TMP) from Cell_0..Cell_8)")]
    public TMP_Text[] spaceList = new TMP_Text[9];

    [Header("Game Over UI on BoardCanvas")]
    public GameObject gameOverPanel;   // BoardCanvas/GameOverPanel
    public TMP_Text gameOverText;      // BoardCanvas/GameOverPanel/GameOverText
    public Button restartButton;       // BoardCanvas/GameOverPanel/RestartButton

    [Header("Scoring / points (optional for expansion)")]
    public int pointsPerWin = 1;

    private string side = "X"; // Current side to play
    private int moves = 0;     // How many moves have been played

    // All winning triplets (indexing spaceList)
    private static readonly int[][] Wins = new int[][]
    {
        new[]{0,1,2}, new[]{3,4,5}, new[]{6,7,8}, // rows
        new[]{0,3,6}, new[]{1,4,7}, new[]{2,5,8}, // cols
        new[]{0,4,8}, new[]{2,4,6}                // diagonals
    };

    private void Start()
    {
        // 1) Ensure cells know who the controller is
        SetGameControllerReferenceForButtons();

        // 2) Reset UI & state
        side = "X";
        moves = 0;

        if (gameOverPanel) gameOverPanel.SetActive(false);
        if (restartButton) restartButton.gameObject.SetActive(false);
    }

    /// <summary> Called by Space.SetSpace() to know what to write. </summary>
    public string GetSide() => side;

    /// <summary> Space calls this after it writes X/O. We evaluate game state. </summary>
    public void EndTurn()
    {
        moves++;

        if (WinCheck())
        {
            ShowGameOver($"{side} wins!");
            return;
        }

        if (moves >= 9) // all cells filled, no winner
        {
            ShowGameOver("Tie!");
            return;
        }

        // No winner yet -> next player's turn
        ChangeSide();
    }

    /// <summary> Resets the whole board to start another game. Hook to RestartButton. </summary>
    public void Restart()
    {
        side = "X";
        moves = 0;

        // Clear labels and re-enable buttons
        for (int i = 0; i < spaceList.Length; i++)
        {
            if (spaceList[i]) spaceList[i].text = string.Empty;

            // Get the Button from the Text's parent (the Cell)
            var btn = spaceList[i] ? spaceList[i].GetComponentInParent<Button>() : null;
            if (btn) btn.interactable = true;
        }

        if (gameOverPanel) gameOverPanel.SetActive(false);
        if (restartButton) restartButton.gameObject.SetActive(false);
    }

    // ---------- Helpers below ----------

    private void SetGameControllerReferenceForButtons()
    {
        // Each TMP label lives under a Cell that has the Space script.
        for (int i = 0; i < spaceList.Length; i++)
        {
            if (!spaceList[i]) continue;

            var space = spaceList[i].GetComponentInParent<Space>();
            if (space != null)
            {
                space.SetControllerReference(this);
            }
            else
            {
                Debug.LogWarning($"[GameController] No Space script found for index {i}. Check your Cell prefab/instances.");
            }
        }
    }

    private void ChangeSide()
    {
        side = (side == "X") ? "O" : "X";
    }

    private bool WinCheck()
    {
        for (int i = 0; i < Wins.Length; i++)
        {
            int a = Wins[i][0], b = Wins[i][1], c = Wins[i][2];

            // Safety: skip if any label missing
            if (!spaceList[a] || !spaceList[b] || !spaceList[c]) continue;

            if (spaceList[a].text == side &&
                spaceList[b].text == side &&
                spaceList[c].text == side)
                return true;
        }
        return false;
    }

    private void ShowGameOver(string message)
    {
        if (gameOverPanel) gameOverPanel.SetActive(true);
        if (gameOverText) gameOverText.text = message;

        // Disable buttons so no more clicks
        SetInteractable(false);

        // Make restart visible & wired
        if (restartButton)
        {
            restartButton.gameObject.SetActive(true);
            restartButton.onClick.RemoveAllListeners();
            restartButton.onClick.AddListener(Restart);
        }
    }

    private void SetInteractable(bool enabled)
    {
        for (int i = 0; i < spaceList.Length; i++)
        {
            var btn = spaceList[i] ? spaceList[i].GetComponentInParent<Button>() : null;
            if (btn) btn.interactable = enabled;
        }
    }
}
```

---

## 4) Create a host object & wire `GameController` <span style="color:purple;">🟣</span>

⬜ **4.1** **Hierarchy:** Right-click → **Create Empty** → rename **`SceneManager`**
⬜ **4.2** **Inspector (SceneManager):** **Add Component → GameController**

### 4A) Fill the 9 cell labels into `spaceList`

⬜ **4.3** **Inspector (GameController on SceneManager):**

* **Space List → Size = 9**
* Expand **BoardCanvas → Grid** in **Hierarchy**. For each index **(0..8)**, drag the **child `Text (TMP)`** inside the corresponding **Cell_N**:

| `spaceList[index]` | Drag this from Hierarchy (TMP Text)  |
| ------------------ | ------------------------------------ |
| 0                  | `BoardCanvas/Grid/Cell_0/Text (TMP)` |
| 1                  | `BoardCanvas/Grid/Cell_1/Text (TMP)` |
| 2                  | `BoardCanvas/Grid/Cell_2/Text (TMP)` |
| 3                  | `BoardCanvas/Grid/Cell_3/Text (TMP)` |
| 4                  | `BoardCanvas/Grid/Cell_4/Text (TMP)` |
| 5                  | `BoardCanvas/Grid/Cell_5/Text (TMP)` |
| 6                  | `BoardCanvas/Grid/Cell_6/Text (TMP)` |
| 7                  | `BoardCanvas/Grid/Cell_7/Text (TMP)` |
| 8                  | `BoardCanvas/Grid/Cell_8/Text (TMP)` |

> ⚠️ **Order matters** (top-left to bottom-right).

### 4B) Hook the Game Over UI

⬜ **4.4** Drag these from **Hierarchy** to **GameController** fields:

* **Game Over Panel =** `BoardCanvas/GameOverPanel`
* **Game Over Text =** `BoardCanvas/GameOverPanel/GameOverText`
* **Restart Button =** `BoardCanvas/GameOverPanel/RestartButton`

> (The `Restart()` click is wired in code when the panel is shown.)

---

## 5) Playtest (ray → click → win/tie → restart) <span style="color:purple;">🟣</span>

1. **Enter Play**.
2. Use a controller **ray** to click **Cell_0**: it should display **“X”** and disable.
3. Click another cell: it should display **“O”** and disable.
4. Force a win (e.g., **X** on 0,1,2).

   * **GameOverPanel** appears with **“X wins!”**, cells become non-clickable, **Restart** shows.
5. Click **Restart**: board clears, **X** starts again.

---

## 6) Troubleshooting (exact spots) <span style="color:purple;">🟣</span>

* **Click does nothing:**
  – On each **Cell**: Button **OnClick → Space.SetSpace()** must exist.
  – The **Space** component must have **Button** and **Button Text** assigned.
* **X/O not switching:**
  – Confirm **GameController.EndTurn()** is called (Debug.Log inside if needed).
* **Win not detected:**
  – Check **spaceList order** is **0..8** left-to-right, top-to-bottom.
* **Restart not clearing:**
  – Ensure **RestartButton** is assigned on **GameController**.
* **GameOver never shows:**
  – **GameOverPanel** should be **disabled** initially and assigned to the field.
* **Cells still clickable after game over:**
  – `SetInteractable(false)` may not find the Button if your prefab structure changed. Verify each TMP lives under a Button parent.

---

## 7) (Optional helper) Auto-wire the 9 cells by name

> Use this only if students struggle with the array order. Add this to **GameController** (below fields), then click a new **“Auto Wire”** button in the Inspector via a tiny custom editor or call in Start once (here we show a method you can call manually in **Start()** if needed).

```csharp
// Add inside GameController:
[ContextMenu("AutoWire From Grid (by names Cell_0..Cell_8)")]
private void AutoWire()
{
    var grid = GameObject.Find("Grid");
    if (!grid)
    {
        Debug.LogWarning("Grid not found.");
        return;
    }
    var texts = grid.GetComponentsInChildren<TMP_Text>(true);
    // Simple name match
    for (int i = 0; i < 9; i++)
    {
        foreach (var t in texts)
        {
            if (t.transform.parent.name == $"Cell_{i}")
            {
                spaceList[i] = t;
                break;
            }
        }
    }
    Debug.Log("AutoWire complete.");
}
```

---
---
# 🔹 Part 4 — VR UX Polish (Reticle, Hover/Press, Haptics, SFX)

<span style="color:purple;font-weight:bold;">🟣 Session Homework — complete every purple step</span>

**Goal:**

* Reticle stays **crisp** and **constant size** in view.
* Cells give **visible hover/press** feedback (plus a gentle **scale-on-hover**).
* **Haptics** play when placing a mark and a stronger pulse on win.
* **Sounds** play on place and win.

**You will use:** **Hierarchy**, **Inspector**, **Project**, **Audio import**, **OVRCameraRig** (CenterEyeAnchor), small **C# slices**.

---

## 0) Pre-flight (must be true from Part 1–3) <span style="color:purple;">🟣</span>

⬜ Rays click UI (you can place X/O).
⬜ `SceneManager` has **GameController** with `spaceList[0..8]` wired.
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

⬜ **2.1** **Hierarchy:** select **all 9** `Cell_0..Cell_8`
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

⬜ **2.4** **Hierarchy:** select **all 9 cells** → **Add Component → CellHoverScale** (keep defaults).

> ✅ Result: as you hover a cell with the ray, it pops up 5% then settles back when you leave.

---

## 3) Haptics on place & win (Meta/OVR) <span style="color:purple;">🟣</span>

We’ll play a tiny pulse on **every move**, and a stronger one on **win**.
We’ll put the logic in **GameController** so `Space` can just call a single `OnCellClicked()`.

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

/// <summary>Called by Space when a cell is clicked, to play click FX/haptics.</summary>
public void OnCellClicked()
{
    // Small, quick tick
    StartCoroutine(Pulse(0.15f, 0.06f));
}
```

### 3B) Call the helper from `Space.cs`

⬜ **3.2** Open **`Space.cs`** and **add one line** inside `SetSpace()` **right after** `buttonText.text = gameController.GetSide();`:

```csharp
gameController.OnCellClicked(); // play click haptics (and SFX in next section)
```

> ✅ Result: every valid click gives a short vibration.

---

## 4) Simple audio: click & win <span style="color:purple;">🟣</span>

We’ll keep audio centralized in **GameController** so it’s easy to wire and grade.

### 4A) Import two clips

⬜ **4.1** **Project:** make folder `Assets/Audio`
⬜ **4.2** Drop in 2 short .wav/.mp3 files (e.g., `click.wav`, `win.wav`)
⬜ **4.3** Select each clip → **Inspector (AudioClip Import Settings):**

* **Load Type = Decompress On Load**
* **Compression = Off or Low** (they’re tiny)
* **Normalize OFF**

### 4B) Add AudioSource on BoardCanvas

⬜ **4.4** **Hierarchy:** select **`BoardCanvas`** → **Add Component → Audio Source**

* **Output:** (leave empty)
* **Spatial Blend = 0.0 (2D)**
* **Play On Awake = OFF**
* **Loop = OFF**

### 4C) Add audio fields + play helpers in `GameController.cs`

⬜ **4.5** Open **`GameController.cs`**. Add these **fields** near your other `[Header]` fields:

```csharp
[Header("Audio (assign on BoardCanvas)")]
public AudioSource audioSource;    // drag BoardCanvas AudioSource here
public AudioClip clickClip;        // Assets/Audio/click.wav
public AudioClip winClip;          // Assets/Audio/win.wav
```

⬜ **4.6** Also **add** these two **methods** *inside* the class:

```csharp
private void PlayClick()
{
    if (audioSource && clickClip) audioSource.PlayOneShot(clickClip, 0.8f);
}

private void PlayWin()
{
    if (audioSource && winClip) audioSource.PlayOneShot(winClip, 1.0f);
}
```

⬜ **4.7** **Hook calls**:

* In **`OnCellClicked()`** (we added in Section 3), **add**:

```csharp
PlayClick();
```

* In **`ShowGameOver(string message)`**, **add** after `gameOverText.text = message;`:

```csharp
PlayWin();
StartCoroutine(Pulse(0.4f, 0.12f)); // a bit stronger haptic on win
```

### 4D) Wire references in the Inspector

⬜ **4.8** **Hierarchy:** select **`SceneManager`** → **GameController (component)**

* **Audio Source:** drag **`BoardCanvas`** (its **Audio Source** component appears)
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
  – `CellHoverScale` must be on the **same object** as the **Button** (each Cell). Ensure **OVR Input Module** is in the scene.
* **No haptics:**
  – Are you in headset focus (Link/AirLink active)? The **Pulse** coroutine is in **GameController** and should be called from **Space.SetSpace()** → `gameController.OnCellClicked();`
* **No audio:**
  – **AudioSource** must be on **BoardCanvas**; clips assigned on **GameController**; device volume up.
* **Audio very quiet:**
  – Increase the **AudioSource Volume** or `PlayOneShot` volume (the second arg).
* **UI stops highlighting after win:**
  – Normal: we disable buttons on game over. Restart to re-enable.

---




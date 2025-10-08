# VR Tic-Tac-Toe: Educator & Student Guide (Unity 6.2, Meta SDK v65+)

This guide provides a complete walkthrough for creating a feature-rich VR Tic-Tac-Toe game. It is designed for educators leading a session and for students to follow along. The tutorial covers a production-ready project setup, a modern VR player rig with multiple locomotion and interaction modes, world-space UI, a robust data-driven architecture for gameplay logic, and advanced features like scoring and an undo system, all using the latest recommended practices for Unity 6.2 and the Meta XR SDK.

---

# 🔹 Part 1/5 — Project Setup (Unity 6.2 • URP • Meta XR SDK v65+)

**Goal:** Configure a new Unity project from scratch for optimal performance and compatibility with the Meta Quest 2/3 platform using the modern OpenXR pipeline.

## 🧭 Flow Map (Window usage for this part)
*   **Unity Hub** → Create the project.
*   **Build Settings / Project Settings** → Configure for Android, Player, XR, and Graphics.
*   **Package Manager** → Install the Meta XR All-in-One SDK.
*   **Meta XR Project Setup Tool** → Apply all recommended fixes.
*   **Project Window** → Organize folders and create the initial scene.

---

## ✅ To-Do Checklist (with "why" + exact clicks)

### 1) Create the Project (URP)
*   ⬜ **1.1** **Unity Hub** → **New Project** → **3D (URP)** template → name it `VR_TicTacToe_Guide` → Create.
    *   **Why:** The Universal Render Pipeline (URP) is Unity's modern, performant rendering solution, essential for achieving high frame rates on mobile VR hardware like the Quest.

### 2) Switch Platform & Set Texture Compression
*   ⬜ **2.1** Go to **File → Build Settings...**.
*   ⬜ **2.2** Select **Android** from the platform list and click **Switch Platform**.
    *   **Why:** The Quest operating system is a fork of Android. Switching early ensures all assets are imported with the correct settings from the start, avoiding lengthy re-imports later.
*   ⬜ **2.3** In the same window, set **Texture Compression** to **ASTC**.
    *   **Why:** ASTC provides the best balance of visual quality and memory usage for textures on Quest devices, which is critical for performance.

### 3) Configure Player Settings for Quest
*   ⬜ **3.1** Go to **Edit → Project Settings... → Player**.
*   ⬜ **3.2** Under the **Other Settings** section for the Android tab (🤖 icon), configure the following:
    *   • **Package Name:** `com.YourCompany.VR.TicTacToe` (must be a unique identifier).
    *   • **Minimum API Level:** Set to **Android 10.0 (API Level 29)** or higher.
    *   • **Target API Level:** Set to **Highest installed**.
    *   • **Scripting Backend:** Set to **IL2CPP**.
    *   • **Target Architectures:** Check **ARM64** only.
    *   **Why:** These settings are mandatory for developing for the Quest platform and are required for submitting to the Meta Quest Store. IL2CPP provides significantly better performance than the older Mono backend.

### 4) Install and Configure the Meta XR SDK (OpenXR Path)
*   ⬜ **4.1** Go to **Window → Package Manager**.
*   ⬜ **4.2** In the top-left, select the **Unity Registry**. Search for and install the **Meta XR All-in-One SDK**.
*   ⬜ **4.3** After installation, go to **Edit → Project Settings... → XR Plug-in Management**.
*   ⬜ **4.4** In the **Android** tab, check the box for **OpenXR**. This will install the OpenXR plug-in.
*   ⬜ **4.5** Under the **OpenXR** settings group that appears, add the **Meta Quest Feature Group**.
    *   **Why:** This configures the OpenXR pipeline to use Meta's specific implementation, enabling Quest-specific features and ensuring compatibility.

### 5) Run Meta's Project Validation Tool
*   ⬜ **5.1** A **Meta XR Project Setup Tool** window should appear. If not, open it from **Edit → Project Settings... → Meta XR**.
*   ⬜ **5.2** Click the **Fix All** button. The tool will apply several critical project settings automatically.
    *   **Why:** This tool is a huge time-saver. It correctly configures input systems, graphics settings, and permissions required for a stable VR application, preventing many common setup-related bugs.

### 6) Create Project Folders and Scene
*   ⬜ **6.1** In the **Project** window, create the following folders in `Assets`: `_Scenes`, `_Scripts`, `_Prefabs`, `_Materials`, `_Audio`, `_Resources`.
*   ⬜ **6.2** Create a new scene: **File → New Scene (Basic 3D)**.
*   ⬜ **6.3** Immediately save the scene as `Assets/_Scenes/TicTacToe.unity`.

---
## 📦 End-of-Part 1 Snapshot
**Hierarchy:**
```
Directional Light
Main Camera
```
**Project Structure:**
```
Assets/
  _Audio/
  _Materials/
  _Prefabs/
  _Resources/
  _Scenes/
    TicTacToe.unity
  _Scripts/
```
**Configuration Summary:**
*   Project is set to the **Android** platform with **ASTC** compression.
*   **OpenXR** is enabled with the **Meta Quest Feature Group**.
*   All validation issues are fixed via the **Meta XR Project Setup Tool**.

---
---

# 🔹 Part 2/5 — VR Rig, Locomotion & Interactions

**Goal:** Set up a robust, modern VR player rig that supports both smooth and teleport locomotion, as well as ray and grab interactions, using a standalone `OVRCameraRig`.

## 🧭 Flow Map
*   **Hierarchy** → Add and configure the `OVRCameraRig` and its interaction prefabs.
*   **Inspector** → Add and configure components for locomotion and interaction on the rig.
*   **Project Window** → Create a simple teleport marker prefab.

---

## ✅ To-Do Checklist

### 1) Set Up the Base VR Rig
*   ⬜ **1.1** In the `TicTacToe` scene, **delete** the default **Main Camera**.
*   ⬜ **1.2** In the **Project** window, search for the `OVRCameraRig` prefab and drag it into the Hierarchy.
*   ⬜ **1.3** In the **Project** window, search for `OVRInteraction` and drag it as a **child** of `OVRCameraRig`.
*   ⬜ **1.4** In the **Project** window, search for `OVRController` and drag it as a **child** of `OVRInteraction`.
    *   **Why:** This hierarchy (`OVRCameraRig` -> `OVRInteraction` -> `OVRController`) is the standard, recommended structure for the Meta Interaction SDK. `OVRCameraRig` handles tracking, `OVRInteraction` manages the interaction system, and `OVRController` provides the hand-specific anchors and interactors.

### 2) Implement Locomotion (Smooth & Teleport)
*   ⬜ **2.1** Select the `OVRCameraRig` GameObject. In the Inspector, add a **Character Controller** component.
    *   • Set **Height** to `1.8` and **Center Y** to `0.9`.
*   ⬜ **2.2** On the same `OVRCameraRig` object, add the **`SimpleCapsuleWithStickMovement`** component.
    *   **Why:** This built-in script provides instant, code-free smooth locomotion and snap turning using the controller thumbsticks, driven by the `CharacterController`.
*   ⬜ **2.3** In the Hierarchy, expand `OVRCameraRig/OVRInteraction/OVRController` to find `LeftController` and `RightController`.
*   ⬜ **2.4** On both the `LeftController/ControllerInteractors` and `RightController/ControllerInteractors` GameObjects, add a **`TeleportInteractor`** component.
*   ⬜ **2.5** Create a simple teleport marker prefab (e.g., a flattened Cylinder with a translucent material), save it in `Assets/_Prefabs`, and assign it to the **Teleport Arc Visuals -> Reticle Prefab** field on both `TeleportInteractor` components.
*   ⬜ **2.6** Create a ground plane: **Hierarchy → 3D Object → Plane**. Rename it `Ground`, set its **Scale** to (10, 1, 10).
*   ⬜ **2.7** On the `Ground` object, add a **`TeleportArea`** component.
    *   **Why:** The `TeleportInteractor` projects an arc. The `TeleportArea` defines surfaces where the teleportation is allowed to land.

### 3) Add Interaction Capabilities
*   ⬜ **3.1** The `OVRController` prefab should have already created `ControllerInteractors` children under each hand. Select the `LeftController/ControllerInteractors` GameObject.
*   ⬜ **3.2** Ensure it has a **`ControllerRayInteractor`** and a **`GrabInteractor`**. If not, add them.
*   ⬜ **3.3** Repeat for `RightController/ControllerInteractors`.
    *   **Why:** The `ControllerRayInteractor` allows pointing and selecting distant objects/UI. The `GrabInteractor` allows for direct, physics-based grabbing of nearby objects. Having both provides a flexible and intuitive interaction model.

---
## 📦 End-of-Part 2 Snapshot
**Hierarchy:**
```
OVRCameraRig (CharacterController, SimpleCapsuleWithStickMovement)
└─ OVRInteraction
   └─ OVRController
      ├─ LeftController
      │  └─ ControllerInteractors (TeleportInteractor, ControllerRayInteractor, GrabInteractor)
      └─ RightController
         └─ ControllerInteractors (TeleportInteractor, ControllerRayInteractor, GrabInteractor)
Ground (TeleportArea)
Directional Light
```
**Configuration Summary:**
*   A standalone `OVRCameraRig` is set up with a `CharacterController` for physics-based movement.
*   Smooth locomotion is enabled via `SimpleCapsuleWithStickMovement`.
*   Teleport locomotion is enabled via `TeleportInteractor` on the hands and `TeleportArea` on the ground.
*   Both hands are equipped with ray and grab interactors, ready for any type of interaction.

---
---

# 🔹 Part 3/5 — Scene UI & Board Creation

**Goal:** Build the physical game environment and the interactive world-space UI for the Tic-Tac-Toe board using the modern Interaction SDK workflow.

## 🧭 Flow Map
*   **Hierarchy** → Create the `Wall`.
*   **Hierarchy / Inspector** → Create and configure the world-space `BoardCanvas`.
*   **Interaction SDK Wizard** → Make the canvas interactable.
*   **Hierarchy / Inspector** → Use a `Grid Layout Group` to build the board.
*   **Project Window / Hierarchy** → Create the `Cell` prefab and populate the grid.

---

## ✅ To-Do Checklist

### 1) Create the Physical Environment
*   ⬜ **1.1** Create a wall to mount the board on: **Hierarchy → 3D Object → Cube**. Rename it `Wall`.
*   ⬜ **1.2** Set the `Wall`'s Transform:
    *   • **Position:** (0, 1.5, 3)
    *   • **Scale:** (4, 2.5, 0.1)
    *   **Why:** A physical environment grounds the player in the VR space and gives context to the world-space UI.

### 2) Build the World-Space UI Board (Modern Path)
*   ⬜ **2.1** Right-click the `Wall` in the Hierarchy and select **UI → Canvas**. Rename the new canvas `BoardCanvas`.
*   ⬜ **2.2** In the Inspector for `BoardCanvas`, configure the following:
    *   • **Render Mode:** `World Space`.
    *   • **Rect Transform → Pos Z:** `-0.06` (to bring it slightly in front of the wall).
    *   • **Rect Transform → Width:** `800`, **Height:** `800`.
    *   • **Rect Transform → Scale:** (0.0015, 0.0015, 0.0015).
*   ⬜ **2.3** Right-click the `BoardCanvas` in the Hierarchy and select **Interaction SDK → Add Ray Interaction to Canvas**.
*   ⬜ **2.4** In the prompt window, click **Fix** to automatically configure the `EventSystem` with the required `PointableCanvasModule`.
    *   **Why:** This is the correct, modern workflow for making UGUI canvases interactable with the Meta Interaction SDK. It ensures the `EventSystem` can correctly process pointer events from the `ControllerRayInteractors`.

### 3) Construct the 3x3 Grid
*   ⬜ **3.1** Right-click `BoardCanvas` → **UI → Panel**. Rename it `GridPanel`.
*   ⬜ **3.2** In the `GridPanel`'s Rect Transform, hold **Alt+Shift** and click the bottom-right "stretch" icon to make it fill the entire canvas.
*   ⬜ **3.3** Add a **Grid Layout Group** component to `GridPanel`. Configure it:
    *   • **Padding:** Left `20`, Right `20`, Top `20`, Bottom `20`.
    *   • **Cell Size:** `(240, 240)`.
    *   • **Spacing:** `(20, 20)`.
    *   • **Constraint:** `Fixed Column Count`, **Constraint Count:** `3`.
    *   **Why:** The `Grid Layout Group` is a powerful tool that automatically arranges its child elements into a grid, saving us from having to place each cell manually.

### 4) Create the Cell Prefab & Placeholders
*   ⬜ **4.1** Right-click `GridPanel` → **UI → Button (TextMeshPro)**. Rename it `Cell`.
*   ⬜ **4.2** Select the `Text (TMP)` child of the `Cell`, clear its text, and set its **Font Size** to `180` and **Alignment** to **Center**.
*   ⬜ **4.3** Drag the `Cell` GameObject from the Hierarchy into `Assets/_Prefabs` to create a prefab.
*   ⬜ **4.4** **Delete** the original `Cell` from the Hierarchy.
*   ⬜ **4.5** Drag the new **`Cell` prefab** from the Project window into the `GridPanel` in the Hierarchy **nine times**.
*   ⬜ **4.6** Create a disabled `GameOverPanel` as a child of `BoardCanvas`, with a `GameOverText` and `RestartButton` inside for later use.
    *   **Why:** Using a prefab is fundamental to good Unity practice. It allows us to make changes to one `Cell` prefab and have those changes apply to all nine instances instantly.

---
## 📦 End-of-Part 3 Snapshot
**Hierarchy:**
```
OVRCameraRig
└─ ...
Wall
└─ BoardCanvas (PointableCanvas)
   ├─ GridPanel (Grid Layout Group)
   │  ├─ Cell (Prefab)
   │  ├─ Cell (1) (Prefab)
   │  ├─ ... (up to 9)
   └─ GameOverPanel (disabled)
      ├─ GameOverText
      └─ RestartButton
Ground
EventSystem (PointableCanvasModule)
Directional Light
```
**Configuration Summary:**
*   A physical wall and ground are in the scene.
*   A world-space `BoardCanvas` is correctly configured for VR interaction using the Interaction SDK wizard.
*   The grid is populated with nine instances of a `Cell` prefab.

---
---

# 🔹 Part 4/5 — Data-Driven Gameplay Logic (ScriptableObjects)

**Goal:** Implement a professional, scalable game architecture using ScriptableObjects to decouple the game state from the UI, making the logic cleaner, more robust, and easier to debug.

## 🧭 Flow Map
*   **Project Window** → Create the `Cell` ScriptableObject script and nine data assets.
*   **Project Window** → Create the `TurnManager` and `BoardManager` singleton scripts.
*   **Project Window** → Create the `UICellBehaviour` script and add it to the `Cell` prefab.
*   **Hierarchy / Inspector** → Create manager GameObjects and wire up the `BoardManager`'s list of `Cell` assets.

---

## ✅ To-Do Checklist

### 1) Asset Preparation
*   ⬜ **1.1** In `Assets/_Resources`, import three simple sprite images: `x.png`, `o.png`, and `b.png` (a blank square).
*   ⬜ **1.2** For each sprite, select it in the Project window and set its **Texture Type** to **Sprite (2D and UI)** in the Inspector.
    *   **Why:** The `Resources` folder is a special Unity folder that allows assets to be loaded dynamically by name from a script, which we'll use to change the X/O icons.

### 2) Create the Data Layer: `Cell` ScriptableObject
*   ⬜ **2.1** In `Assets/_Scripts`, create a new folder `Gameplay`. Inside, create a C# script named `Cell.cs`.
*   ⬜ **2.2** Paste this code. This defines the data for a single cell, completely separate from its visual representation.
```csharp
using System;
using UnityEngine;

[CreateAssetMenu(fileName = "Cell", menuName = "TicTacToe/Cell Data")]
public class Cell : ScriptableObject
{
    public int id;
    public int value; // 0 = blank, 1 = X, 2 = O

    public event Action<int> OnValueChanged; // Exposes the new value

    public void SetValue(int newValue)
    {
        value = newValue;
        OnValueChanged?.Invoke(value);
    }

    public void Reset()
    {
        SetValue(0);
    }
}
```
*   ⬜ **2.3** In the Project window (`Assets/_Scripts/Gameplay`), right-click → **Create → TicTacToe → Cell Data**. Create nine of these, naming them `Cell_0` through `Cell_8`.
*   ⬜ **2.4** For each `Cell_` asset, select it and set its **Id** field in the Inspector to match its name (e.g., `Cell_5` has an Id of `5`).

### 3) Create the Singleton Managers
*   ⬜ **3.1** Create a base singleton script `PersistentMonoSingleton.cs` in `Assets/_Scripts/Gameplay`. (Use the code from the user's reference material).
*   ⬜ **3.2** Create `TurnManager.cs` and `BoardManager.cs`, inheriting from the singleton base.
    *   • **`TurnManager`** will have a `bool GetTurn()` method that alternates and a `ResetTurns()` method.
    *   • **`BoardManager`** will hold a `List<Cell>` of the nine `Cell` assets. It will subscribe to each `Cell`'s `OnValueChanged` event to check for win/draw conditions.
*   ⬜ **3.3** In the Hierarchy, create empty GameObjects for `TurnManager` and `BoardManager` and attach their respective scripts.
*   ⬜ **3.4** On the `BoardManager` object in the Inspector, set the **Cells List** size to `9` and drag your `Cell_0` through `Cell_8` assets into the slots in order.

### 4) Create the View Layer: `UICellBehaviour`
*   ⬜ **4.1** In `Assets/_Scripts`, create a folder `UI`. Inside, create a C# script named `UICellBehaviour.cs`.
*   ⬜ **4.2** Paste this code. This script sits on the UI prefab and its only job is to listen to a `Cell` data asset and update its visuals accordingly.
```csharp
using UnityEngine;
using UnityEngine.UI;

public class UICellBehaviour : MonoBehaviour
{
    [Header("Data Asset")]
    [SerializeField] private Cell cellData;

    [Header("UI Components")]
    [SerializeField] private Button button;
    [SerializeField] private Image image;

    private Sprite _xSprite, _oSprite, _blankSprite;

    void Awake()
    {
        _xSprite = Resources.Load<Sprite>("x");
        _oSprite = Resources.Load<Sprite>("o");
        _blankSprite = Resources.Load<Sprite>("b");
        button.onClick.AddListener(OnButtonClick);
    }

    void OnEnable() => cellData.OnValueChanged += UpdateVisuals;
    void OnDisable() => cellData.OnValueChanged -= UpdateVisuals;
    void Start() => UpdateVisuals(cellData.value);

    private void OnButtonClick()
    {
        if (cellData.value == 0) // Only if blank
        {
            cellData.SetValue(TurnManager.Instance.GetTurn() ? 1 : 2);
        }
    }

    private void UpdateVisuals(int value)
    {
        switch (value)
        {
            case 1: image.sprite = _xSprite; button.interactable = false; break;
            case 2: image.sprite = _oSprite; button.interactable = false; break;
            default: image.sprite = _blankSprite; button.interactable = true; break;
        }
    }
}
```
*   ⬜ **4.3** Open the **`Cell` prefab**. Add the **`UICellBehaviour`** script. Drag the prefab's own `Button` and `Image` components into the corresponding fields.
*   ⬜ **4.4** In the Hierarchy, select each of the nine `Cell` instances (`Cell (0)` to `Cell (8)`) and assign the corresponding `Cell` data asset (`Cell_0` to `Cell_8`) to its **Cell Data** field.

---
## 📦 End-of-Part 4 Snapshot
**Hierarchy:**
```
OVRCameraRig
└─ ...
Wall
└─ BoardCanvas
   └─ GridPanel
      ├─ Cell (0) (UICellBehaviour -> Cell_0 asset)
      ├─ Cell (1) (UICellBehaviour -> Cell_1 asset)
      └─ ... (up to 8)
...
BoardManager (Holds list of all 9 Cell assets)
TurnManager
```
**Configuration Summary:**
*   The game logic is now data-driven. UI `Cell` prefabs listen to `Cell` data assets for changes.
*   The `BoardManager` centrally manages game rules by observing all `Cell` data assets.
*   The `TurnManager` handles whose turn it is.
*   The core game loop is functional with a much more robust and scalable architecture.

---
---

# 🔹 Part 5/5 — Polished UX & Advanced Features

**Goal:** Elevate the project from a functional prototype to a polished game by adding sensory feedback (hover effects, haptics, audio) and advanced gameplay features (HUD, scoring, undo).

## 🧭 Flow Map
*   **Project Window / Inspector** → Add a hover-scaling script to the `Cell` prefab.
*   **Hierarchy / Inspector** → Build the HUD UI elements.
*   **Code Editor** → Create a `UIBehaviour` to manage UX and a `GameController` to handle complex actions like Undo/Reset.
*   **Hierarchy / Inspector** → Wire up all the new components and references.

---

## ✅ To-Do Checklist

### 1) Add Visual Hover Feedback
*   ⬜ **1.1** In `Assets/_Scripts/UI`, create a new C# script named `CellHoverScale.cs`.
*   ⬜ **1.2** Paste this code. It makes the cell slightly larger when a player's ray points at it.
```csharp
using UnityEngine;
using UnityEngine.EventSystems;

public class CellHoverScale : MonoBehaviour, IPointerEnterHandler, IPointerExitHandler
{
    public float hoverScale = 1.05f;
    private Vector3 _initialScale;

    void Awake() => _initialScale = transform.localScale;
    public void OnPointerEnter(PointerEventData eventData) => transform.localScale = _initialScale * hoverScale;
    public void OnPointerExit(PointerEventData eventData) => transform.localScale = _initialScale;
}
```
*   ⬜ **1.3** Open the **`Cell` prefab** and add the **`CellHoverScale`** script to it. Save the prefab.

### 2) Create a Central `UIBehaviour` for UX
*   ⬜ **2.1** Create a new script `UIBehaviour.cs` in `Assets/_Scripts/UI`. This will handle all UI updates, sounds, and haptics in one place.
*   ⬜ **2.2** In `UIBehaviour`, subscribe to events from `BoardManager` and `TurnManager` to know when to play sounds, trigger haptics, and update the HUD.
*   ⬜ **2.3** Add an `AudioSource` to the `BoardCanvas` and wire it to `UIBehaviour`, along with click and win audio clips.

### 3) Build the HUD
*   ⬜ **3.1** In the Hierarchy, create a `HUDPanel` under the `BoardCanvas` (anchor it to the top).
*   ⬜ **3.2** Inside `HUDPanel`, add TextMeshPro elements for `TurnText`, `ScoreText`, and `RoundText`.
*   ⬜ **3.3** Add two buttons: `UndoButton` and `ResetAllButton`.
*   ⬜ **3.4** Wire all these new HUD elements to fields in the `UIBehaviour` script.

### 4) Implement High-Level Game Actions with a `GameController`
*   ⬜ **4.1** Create a `GameController.cs` script in `Assets/_Scripts/Gameplay`. This script will not manage the board state directly, but will coordinate complex actions.
*   ⬜ **4.2** Implement public methods like `UndoLastMove()` and `ResetAllStats()`.
    *   `UndoLastMove()` will use a move history (a `Stack<Cell>`) stored in the `BoardManager` to revert the last move.
    *   `ResetAllStats()` will call `BoardManager.ResetGame()` and also reset score/round variables.
*   ⬜ **4.3** Add the `GameController` to a `GameManager` GameObject in the scene.
*   ⬜ **4.4** Wire the `UndoButton` and `ResetAllButton`'s `onClick` events to call the public methods on the `GameController`.
    *   **Why:** This architecture keeps the core win/loss logic in the `BoardManager` clean, while the `GameController` acts as a "command" layer for complex user actions that might involve multiple systems.

### 5) Final Playtest
*   ⬜ **5.1** Enter Play mode and test all features:
    *   **Hover & Click:** Do cells scale up and play sounds/haptics?
    *   **HUD:** Does the turn, score, and round text update correctly?
    *   **Undo:** Does the undo button correctly revert the last move?
    *   **Scoring:** Does the score update correctly after a win and persist into the next round?
    *   **Reset:** Does the reset button clear all stats?

---
## 🏆 Congratulations!
You have successfully built a complete, feature-rich, and polished Tic-Tac-Toe game in VR. You've learned how to set up a modern Unity VR project, implement a robust player rig, build interactive world-space UI, and architect clean, data-driven gameplay logic using professional patterns. You can now use these skills as a foundation for creating even more complex and exciting VR experiences.
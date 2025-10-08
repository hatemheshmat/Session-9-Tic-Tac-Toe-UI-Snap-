# VR Tic-Tac-Toe: Educator & Student Guide (Unity 6.2, Meta SDK v65+)

This guide provides a complete walkthrough for creating a feature-rich VR Tic-Tac-Toe game. It is designed for educators leading a session and for students to follow along. The tutorial covers project setup, a modern VR player rig with locomotion, world-space UI, robust data-driven gameplay logic, and advanced features like scoring and an undo system, all using the latest recommended practices for Unity 6.2 and the Meta XR SDK.

---

# 🔹 Part 1/5 — Project Setup (Unity 6.2 • URP • Meta XR SDK v65+)

**Goal:** Configure a new Unity project from scratch for optimal performance and compatibility with the Meta Quest 2/3 platform.

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

### 4) Install and Configure the Meta XR SDK
*   ⬜ **4.1** Go to **Window → Package Manager**.
*   ⬜ **4.2** In the top-left, select the **Unity Registry**. Search for and install the **Meta XR All-in-One SDK**.
*   ⬜ **4.3** After installation, go to **Edit → Project Settings... → XR Plug-in Management**.
*   ⬜ **4.4** In the **Android** tab, check the box for **Meta XR**.
    *   **Why:** This activates the Meta runtime, which allows your Unity application to communicate with the Quest headset's hardware for tracking and input.

### 5) Run Meta's Project Validation Tool
*   ⬜ **5.1** A **Meta XR Project Setup Tool** window should appear. If not, open it from **Edit → Project Settings... → Meta XR**.
*   ⬜ **5.2** Click the **Fix All** button. The tool will apply several critical project settings automatically.
    *   **Why:** This tool is a huge time-saver. It correctly configures input systems, graphics settings, and permissions required for a stable VR application, preventing many common setup-related bugs.

### 6) Create Project Folders and Scene
*   ⬜ **6.1** In the **Project** window, create the following folders in `Assets`: `_Scenes`, `_Scripts`, `_Prefabs`, `_Materials`, `_Audio`.
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
  _Scenes/
    TicTacToe.unity
  _Scripts/
```
**Configuration Summary:**
*   Project is set to the **Android** platform with **ASTC** compression.
*   **Meta XR** is enabled in XR Plug-in Management.
*   All validation issues are fixed via the **Meta XR Project Setup Tool**.

---
---

# 🔹 Part 2/5 — VR Rig, Locomotion & Interactions

**Goal:** Set up a robust, modern VR player rig that supports both smooth and teleport locomotion, as well as ray and grab interactions.

## 🧭 Flow Map
*   **Hierarchy** → Create a `PlayerRig` parent and add the `OVRCameraRig`.
*   **Project Window** → Create the `SimpleRigLocomotion.cs` script.
*   **Inspector** → Add and configure components for locomotion and interaction on the rig.

---

## ✅ To-Do Checklist

### 1) Set Up the Player Rig Hierarchy
*   ⬜ **1.1** In the `TicTacToe` scene, **delete** the default **Main Camera**.
*   ⬜ **1.2** Create an empty GameObject named `PlayerRig`.
*   ⬜ **1.3** In the **Project** window, search for the `OVRCameraRig` prefab and drag it into the Hierarchy, making it a **child** of `PlayerRig`.
    *   **Why:** Creating a parent `PlayerRig` object allows us to add components like a `CharacterController` for physics-based movement without modifying the `OVRCameraRig` prefab directly. This is a clean and scalable approach.

### 2) Implement Locomotion (Smooth & Teleport)
*   ⬜ **2.1** Select the `PlayerRig` GameObject. In the Inspector, add a **Character Controller** component. Adjust its **Height** to `1.8` and **Center Y** to `0.9`.
*   ⬜ **2.2** In `Assets/_Scripts`, create a new C# script named `SimpleRigLocomotion.cs`.
*   ⬜ **2.3** Paste the following code into the script. This script handles both smooth movement and teleportation.
```csharp
using UnityEngine;

[RequireComponent(typeof(CharacterController))]
public class SimpleRigLocomotion : MonoBehaviour
{
    [Header("Dependencies")]
    [Tooltip("The OVRCameraRig's transform, used to determine forward direction.")]
    public Transform cameraRig;

    [Header("Movement Settings")]
    public float moveSpeed = 2.0f;
    public float turnSpeed = 45.0f; // Degrees per second

    [Header("Teleport Settings")]
    public OVRInput.RawButton teleportButton = OVRInput.RawButton.A;
    public float maxTeleportDistance = 10.0f;
    public LayerMask teleportSurfaceMask; // Set to "Default" to teleport on the ground
    public GameObject teleportMarkerPrefab; // A simple cylinder or sphere prefab

    private CharacterController _characterController;
    private GameObject _teleportMarkerInstance;
    private bool _isTeleportTargetValid;

    void Awake()
    {
        _characterController = GetComponent<CharacterController>();
        if (cameraRig == null) cameraRig = FindObjectOfType<OVRCameraRig>().transform;
    }

    void Start()
    {
        if (teleportMarkerPrefab != null)
        {
            _teleportMarkerInstance = Instantiate(teleportMarkerPrefab);
            _teleportMarkerInstance.SetActive(false);
        }
    }

    void Update()
    {
        HandleSmoothLocomotion();
        HandleTeleportation();
    }

    private void HandleSmoothLocomotion()
    {
        // Smooth Movement (Left Stick)
        Vector2 moveInput = OVRInput.Get(OVRInput.RawAxis2D.LThumbstick);
        Vector3 forward = Vector3.ProjectOnPlane(cameraRig.forward, Vector3.up).normalized;
        Vector3 right = Vector3.ProjectOnPlane(cameraRig.right, Vector3.up).normalized;
        Vector3 moveDir = (forward * moveInput.y + right * moveInput.x);
        _characterController.Move(moveDir * moveSpeed * Time.deltaTime);

        // Snap Turning (Right Stick)
        if (OVRInput.GetDown(OVRInput.RawButton.RThumbstickLeft))
        {
            transform.Rotate(Vector3.up, -turnSpeed);
        }
        if (OVRInput.GetDown(OVRInput.RawButton.RThumbstickRight))
        {
            transform.Rotate(Vector3.up, turnSpeed);
        }
    }

    private void HandleTeleportation()
    {
        if (teleportMarkerPrefab == null) return;

        // Aiming Phase (Right Stick Forward)
        if (OVRInput.Get(teleportButton))
        {
            if (Physics.Raycast(OVRInput.GetLocalControllerPosition(OVRInput.Controller.RTouch), OVRInput.GetLocalControllerRotation(OVRInput.Controller.RTouch) * Vector3.forward, out RaycastHit hit, maxTeleportDistance, teleportSurfaceMask))
            {
                _teleportMarkerInstance.SetActive(true);
                _teleportMarkerInstance.transform.position = hit.point;
                _isTeleportTargetValid = true;
            }
            else
            {
                _teleportMarkerInstance.SetActive(false);
                _isTeleportTargetValid = false;
            }
        }

        // Teleport Execution (Button Release)
        if (OVRInput.GetUp(teleportButton))
        {
            if (_isTeleportTargetValid)
            {
                Vector3 teleportPos = _teleportMarkerInstance.transform.position;
                // Adjust position to account for character controller height
                teleportPos.y = transform.position.y;
                transform.position = teleportPos;
            }
            _teleportMarkerInstance.SetActive(false);
        }
    }
}
```
*   ⬜ **2.4** Add the **`SimpleRigLocomotion`** script to the `PlayerRig` GameObject.
*   ⬜ **2.5** Create a simple teleport marker (e.g., a flattened Cylinder), save it as a prefab in `Assets/_Prefabs`, and assign it to the **Teleport Marker Prefab** field.
*   ⬜ **2.6** Set the **Teleport Surface Mask** to **Default**.

### 3) Add Interaction Capabilities
*   ⬜ **3.1** In the Hierarchy, find `OVRCameraRig/TrackingSpace/LeftHandAnchor` and `RightHandAnchor`.
*   ⬜ **3.2** To both `LeftHandAnchor` and `RightHandAnchor`, add the **`OVR Controller Helper`** component. This provides visual controller models.
*   ⬜ **3.3** To both anchors, also add the **`OVR Hand`** component. This enables hand tracking visuals.
*   ⬜ **3.4** Create empty GameObjects named `Interaction` as children of both hand anchors.
*   ⬜ **3.5** To both `Interaction` objects, add a **`Ray Interactor`** component and a **`Grab Interactor`** component.
    *   **Why:** The `Ray Interactor` allows pointing and selecting distant objects/UI. The `Grab Interactor` allows for direct, physics-based grabbing of nearby objects.

---
## 📦 End-of-Part 2 Snapshot
**Hierarchy:**
```
PlayerRig (CharacterController, SimpleRigLocomotion)
└─ OVRCameraRig
   └─ TrackingSpace
      ├─ LeftHandAnchor (OVR Controller Helper, OVR Hand)
      │  └─ Interaction (Ray Interactor, Grab Interactor)
      └─ RightHandAnchor (OVR Controller Helper, OVR Hand)
         └─ Interaction (Ray Interactor, Grab Interactor)
Directional Light
```
**Configuration Summary:**
*   A `PlayerRig` is set up with a `CharacterController` for physics.
*   The `SimpleRigLocomotion` script provides smooth movement, snap turning, and teleportation.
*   Both hands are equipped with ray and grab interactors, ready for interaction.

---
---

# 🔹 Part 3/5 — Scene UI & Board Creation

**Goal:** Build the physical game environment and the interactive world-space UI for the Tic-Tac-Toe board.

## 🧭 Flow Map
*   **Hierarchy** → Create the `Wall` and `Ground` planes.
*   **Hierarchy / Inspector** → Create and configure the world-space `BoardCanvas` and its `Grid Layout Group`.
*   **Project Window / Hierarchy** → Create the `Cell` prefab and populate the grid.
*   **Hierarchy / Inspector** → Configure the `EventSystem` to work with VR controllers.

---

## ✅ To-Do Checklist

### 1) Create the Physical Environment
*   ⬜ **1.1** In the `TicTacToe` scene, create a large plane for the ground: **Hierarchy → 3D Object → Plane**. Rename it `Ground`, set its **Scale** to (5, 1, 5).
*   ⬜ **1.2** Create a wall to mount the board on: **Hierarchy → 3D Object → Cube**. Rename it `Wall`.
*   ⬜ **1.3** Set the `Wall`'s Transform:
    *   • **Position:** (0, 1.5, 3)
    *   • **Scale:** (4, 2.5, 0.1)
    *   **Why:** A physical environment grounds the player in the VR space and gives context to the world-space UI.

### 2) Build the World-Space UI Board
*   ⬜ **2.1** Right-click the `Wall` in the Hierarchy and select **UI → Canvas**. Rename the new canvas `BoardCanvas`.
*   ⬜ **2.2** In the Inspector for `BoardCanvas`, configure the following:
    *   • **Render Mode:** `World Space`.
    *   • **Rect Transform → Pos Z:** `-0.06` (to bring it slightly in front of the wall).
    *   • **Rect Transform → Width:** `800`, **Height:** `800`.
    *   • **Rect Transform → Scale:** (0.0015, 0.0015, 0.0015).
*   ⬜ **2.3** Add the **`OVR Raycaster`** component to the `BoardCanvas`. Set its **Blocking Objects** to **None**.
    *   **Why:** A world-space canvas exists as an object in the 3D scene. The `OVR Raycaster` is essential for allowing VR controller rays to interact with its UI elements.

### 3) Construct the 3x3 Grid
*   ⬜ **3.1** Right-click `BoardCanvas` → **UI → Panel**. Rename it `GridPanel`.
*   ⬜ **3.2** In the `GridPanel`'s Rect Transform, hold **Alt+Shift** and click the bottom-right "stretch" icon to make it fill the entire canvas.
*   ⬜ **3.3** Add a **Grid Layout Group** component to `GridPanel`. Configure it:
    *   • **Padding:** Left `20`, Right `20`, Top `20`, Bottom `20`.
    *   • **Cell Size:** `(240, 240)`.
    *   • **Spacing:** `(20, 20)`.
    *   • **Constraint:** `Fixed Column Count`, **Constraint Count:** `3`.
    *   **Why:** The `Grid Layout Group` is a powerful tool that automatically arranges its child elements into a grid, saving us from having to place each cell manually.

### 4) Create the Cell Prefab
*   ⬜ **4.1** Right-click `GridPanel` → **UI → Button (TextMeshPro)**. Rename it `Cell`.
*   ⬜ **4.2** Select the `Text (TMP)` child of the `Cell` and set its **Font Size** to `180` and **Alignment** to **Center**.
*   ⬜ **4.3** Drag the `Cell` GameObject from the Hierarchy into `Assets/_Prefabs` to create a prefab.
*   ⬜ **4.4** **Delete** the original `Cell` from the Hierarchy.
*   ⬜ **4.5** Drag the new **`Cell` prefab** from the Project window into the `GridPanel` in the Hierarchy **nine times**. The grid will automatically arrange them.
    *   **Why:** Using a prefab is fundamental to good Unity practice. It allows us to make changes to one `Cell` prefab and have those changes apply to all nine instances instantly.

### 5) Finalize UI Interactivity
*   ⬜ **5.1** In the Hierarchy, select the `EventSystem` GameObject.
*   ⬜ **5.2** In the Inspector, click **Remove Component** to delete the `Standalone Input Module`.
*   ⬜ **5.3** Click **Add Component** and add the **`OVR Input Module`**.
    *   **Why:** The default `Standalone Input Module` is designed for mouse and keyboard. The `OVR Input Module` is specifically designed to translate input from VR controllers (like trigger presses) into UI events that components like Buttons can understand.

---
## 📦 End-of-Part 3 Snapshot
**Hierarchy:**
```
PlayerRig
└─ OVRCameraRig
   └─ ...
Wall
└─ BoardCanvas (OVR Raycaster)
   └─ GridPanel (Grid Layout Group)
      ├─ Cell (Prefab)
      ├─ Cell (1) (Prefab)
      ├─ ... (up to 9)
Ground
EventSystem (OVR Input Module)
Directional Light
```
**Configuration Summary:**
*   A physical wall and ground are in the scene.
*   A world-space `BoardCanvas` is set up with a `Grid Layout Group`.
*   The grid is populated with nine instances of a `Cell` prefab.
*   The `EventSystem` is configured for VR interaction.

---
---

# 🔹 Part 4/5 — Core Gameplay Logic

**Goal:** Implement the brain of the game using a robust, data-driven architecture. The `GameController` will manage the game state, while `Cell` components will simply report user input.

## 🧭 Flow Map
*   **Project Window** → Create the `GameController` and `Cell` C# scripts.
*   **Visual Studio / Code Editor** → Write the C# logic for both scripts.
*   **Hierarchy / Inspector** → Create a `GameManager` object, attach `GameController`, and wire up all UI references.
*   **Project Window / Inspector** → Add the `Cell` script to the `Cell` prefab.

---

## ✅ To-Do Checklist

### 1) Create the Core Scripts
*   ⬜ **1.1** In `Assets/_Scripts`, create a new subfolder named `TicTacToe`.
*   ⬜ **1.2** Inside this folder, create two C# scripts: `GameController.cs` and `Cell.cs`.

### 2) Code the `Cell.cs` Script
*   ⬜ **2.1** Open `Cell.cs`. This script will be very simple; its only job is to tell the `GameController` when it has been clicked. Paste this code:
```csharp
using UnityEngine;
using UnityEngine.UI;

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

    public void Initialize(int index, GameController controller)
    {
        _cellIndex = index;
        _gameController = controller;
    }

    private void OnCellClicked()
    {
        // Notify the central controller that this specific cell was clicked.
        if (_gameController != null)
        {
            _gameController.OnCellClicked(_cellIndex);
        }
    }
}
```
*   ⬜ **2.2** Open the **`Cell` prefab** for editing (double-click it in the Project window).
*   ⬜ **2.3** Add the **`Cell.cs` script** to the root of the prefab. Save the prefab. All nine instances in the scene will now have this script.

### 3) Code the `GameController.cs` Script
*   ⬜ **3.1** Open `GameController.cs`. This script will manage everything: the board state, player turns, win checks, and UI updates. Paste this code:
```csharp
using UnityEngine;
using UnityEngine.UI;
using TMPro;

public class GameController : MonoBehaviour
{
    [Header("UI & Board References")]
    [Tooltip("Assign the 9 Cell buttons from the hierarchy, in order 0-8.")]
    public Button[] cellButtons = new Button[9];
    public GameObject gameOverPanel;
    public TMP_Text gameOverText;
    public Button restartButton;

    // --- Game State ---
    private int[] _boardState = new int[9]; // 0=Empty, 1=X, 2=O
    private int _currentPlayer = 1;
    private int _movesMade = 0;
    private bool _isGameActive = true;

    private readonly int[][] _winConditions =
    {
        new[] {0, 1, 2}, new[] {3, 4, 5}, new[] {6, 7, 8}, // Rows
        new[] {0, 3, 6}, new[] {1, 4, 7}, new[] {2, 5, 8}, // Columns
        new[] {0, 4, 8}, new[] {2, 4, 6}                  // Diagonals
    };

    void Start()
    {
        // The Cell scripts will report clicks, so we just need to listen for the restart button.
        restartButton.onClick.AddListener(RestartGame);

        // Initialize each Cell with its index and a reference to this controller.
        for (int i = 0; i < cellButtons.Length; i++)
        {
            cellButtons[i].GetComponent<Cell>()?.Initialize(i, this);
        }

        RestartGame();
    }

    public void RestartGame()
    {
        _isGameActive = true;
        _currentPlayer = 1;
        _movesMade = 0;

        for (int i = 0; i < _boardState.Length; i++)
        {
            _boardState[i] = 0; // Clear internal state
            UpdateCellUI(i);    // Update visual
            cellButtons[i].interactable = true;
        }

        gameOverPanel.SetActive(false);
    }

    public void OnCellClicked(int cellIndex)
    {
        if (!_isGameActive || _boardState[cellIndex] != 0) return;

        _boardState[cellIndex] = _currentPlayer;
        _movesMade++;

        UpdateCellUI(cellIndex);
        cellButtons[cellIndex].interactable = false;

        if (CheckForWin())
        {
            EndGame(isTie: false);
        }
        else if (_movesMade >= 9)
        {
            EndGame(isTie: true);
        }
        else
        {
            _currentPlayer = (_currentPlayer == 1) ? 2 : 1;
        }
    }

    private bool CheckForWin()
    {
        foreach (var condition in _winConditions)
        {
            if (_boardState[condition[0]] != 0 &&
                _boardState[condition[0]] == _boardState[condition[1]] &&
                _boardState[condition[0]] == _boardState[condition[2]])
            {
                return true;
            }
        }
        return false;
    }

    private void EndGame(bool isTie)
    {
        _isGameActive = false;
        gameOverPanel.SetActive(true);
        gameOverText.text = isTie ? "It's a Tie!" : $"Player {(_currentPlayer == 1 ? "X" : "O")} Wins!";
    }

    private void UpdateCellUI(int cellIndex)
    {
        TMP_Text textComponent = cellButtons[cellIndex].GetComponentInChildren<TMP_Text>();
        if (textComponent != null)
        {
            switch (_boardState[cellIndex])
            {
                case 1: textComponent.text = "X"; break;
                case 2: textComponent.text = "O"; break;
                default: textComponent.text = ""; break;
            }
        }
    }
}
```

### 4) Wire the `GameController` in the Scene
*   ⬜ **4.1** In the Hierarchy, create an empty GameObject and name it `GameManager`.
*   ⬜ **4.2** Add the **`GameController.cs`** script to the `GameManager` object.
*   ⬜ **4.3** Create the Game Over UI:
    *   • Right-click `BoardCanvas` → **UI → Panel**. Rename it `GameOverPanel`.
    *   • Make it fill the screen and give it a semi-transparent background color.
    *   • Inside `GameOverPanel`, add a **TextMeshPro - Text** for the status (`GameOverText`) and a **Button (TextMeshPro)** to restart (`RestartButton`).
*   ⬜ **4.4** Select the `GameManager`. In the Inspector, lock the view (top-right lock icon).
*   ⬜ **4.5** Drag all nine `Cell` GameObjects from the Hierarchy onto the **Cell Buttons** array field.
*   ⬜ **4.6** Drag the `GameOverPanel`, `GameOverText`, and `RestartButton` into their respective fields.
*   ⬜ **4.7** **Disable** the `GameOverPanel` in the Hierarchy so it's hidden at the start.

---
## 📦 End-of-Part 4 Snapshot
**Hierarchy:**
```
PlayerRig
└─ ...
Wall
└─ BoardCanvas
   ├─ GridPanel
   │  └─ Cell (x9)
   └─ GameOverPanel (disabled)
      ├─ GameOverText
      └─ RestartButton
Ground
EventSystem
GameManager (GameController script)
Directional Light
```
**Configuration Summary:**
*   `GameManager` holds the `GameController` script, which is now wired to all `Cell` buttons and the `GameOverPanel`.
*   The `Cell` prefab now has the `Cell.cs` script, which automatically handles its own click events.
*   The core game loop is functional: players can take turns, and the game correctly identifies a win or a tie.

---
---

# 🔹 Part 5/5 — UX Polish & Advanced Features

**Goal:** Elevate the project from a functional prototype to a polished game by adding sensory feedback (hover effects, haptics, audio) and advanced gameplay features (HUD, scoring, undo).

## 🧭 Flow Map
*   **Project Window / Inspector** → Create and add the `CellHoverScale` script to the `Cell` prefab.
*   **Code Editor** → Add audio, haptics, and new feature logic to `GameController.cs`.
*   **Hierarchy / Inspector** → Build the HUD UI elements and wire them to the `GameController`.
*   **Project Window** → Import audio clips.

---

## ✅ To-Do Checklist

### 1) Add Visual Hover Feedback
*   ⬜ **1.1** In `Assets/_Scripts/TicTacToe`, create a new C# script named `CellHoverScale.cs`.
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

### 2) Implement Haptics and Audio
*   ⬜ **2.1** In `GameController.cs`, add a new header and fields for audio clips and the Audio Source component:
```csharp
[Header("Audio & Haptics")]
public AudioSource audioSource;
public AudioClip clickClip;
public AudioClip winClip;
```
*   ⬜ **2.2** Add a helper method to `GameController.cs` to trigger haptics.
```csharp
private void TriggerHaptics(OVRInput.Controller controller, float amplitude, float duration)
{
    OVRInput.SetControllerVibration(1, amplitude, controller);
    Invoke(nameof(StopHaptics), duration);
}
private void StopHaptics() => OVRInput.SetControllerVibration(0, 0, OVRInput.Controller.Active);
```
*   ⬜ **2.3** In `OnCellClicked()`, after updating the game state, add calls to play a sound and trigger a short haptic pulse.
```csharp
// Inside OnCellClicked, after _movesMade++
audioSource.PlayOneShot(clickClip);
TriggerHaptics(OVRInput.Controller.Active, 0.2f, 0.1f);
```
*   ⬜ **2.4** In `EndGame()`, add calls to play the win sound and a stronger haptic pulse.
```csharp
// Inside EndGame, after setting gameOverText.text
audioSource.PlayOneShot(winClip);
TriggerHaptics(OVRInput.Controller.Active, 0.5f, 0.3f);
```
*   ⬜ **2.5** On the `GameManager` object, add an **Audio Source** component.
*   ⬜ **2.6** Import two sound clips into `Assets/_Audio` (e.g., a simple click and a win sound).
*   ⬜ **2.7** In the Inspector for `GameManager`, assign the **Audio Source** component and the two audio clips to the new fields on the `GameController`.

### 3) Build the HUD and Add Advanced Gameplay Features
*   ⬜ **3.1** In the Hierarchy, create a `HUDPanel` under the `BoardCanvas` (anchor it to the top). Inside it, add TextMeshPro elements for `TurnText`, `ScoreText`, `RoundText`, and two buttons: `UndoButton` and `ResetAllButton`.
*   ⬜ **3.2** In `GameController.cs`, add fields for all the new HUD elements, score/round counters, and a move history stack.
```csharp
[Header("HUD UI")]
public TMP_Text turnText;
public TMP_Text scoreText;
public TMP_Text roundText;
public Button undoButton;
public Button resetAllButton;

// Game Stats
private int _scoreX = 0;
private int _scoreO = 0;
private int _currentRound = 1;
private System.Collections.Generic.Stack<int> _moveHistory = new();
```
*   ⬜ **3.3** Create a new `UpdateHUD()` method in `GameController` to refresh all text elements. Call this method whenever the state changes (e.g., at the end of `RestartGame`, `OnCellClicked`, `EndGame`, and `UndoLastMove`).
*   ⬜ **3.4** Implement the `UndoLastMove()` method.
```csharp
public void UndoLastMove()
{
    if (_moveHistory.Count == 0 || !_isGameActive) return;

    int lastMove = _moveHistory.Pop();
    _boardState[lastMove] = 0;
    _movesMade--;
    cellButtons[lastMove].interactable = true;
    _currentPlayer = (_currentPlayer == 1) ? 2 : 1; // Switch turn back

    UpdateCellUI(lastMove);
    UpdateHUD();
}
```
*   ⬜ **3.5** In `OnCellClicked()`, push the `cellIndex` to the `_moveHistory` stack.
*   ⬜ **3.6** In `EndGame()`, increment the appropriate score variable (`_scoreX` or `_scoreO`) if it's not a tie.
*   ⬜ **3.7** In the `Start()` method, add listeners for the `undoButton` and `resetAllButton`'s `onClick` events.
*   ⬜ **3.8** Wire up all the new HUD UI references in the Inspector on the `GameManager`.

### 4) Final Playtest
*   ⬜ **4.1** Enter Play mode and test all features:
    *   **Hover & Click:** Do cells scale up and play sounds/haptics?
    *   **HUD:** Does the turn, score, and round text update correctly?
    *   **Undo:** Does the undo button correctly revert the last move?
    *   **Scoring:** Does the score update correctly after a win and persist into the next round?
    *   **Reset:** Does the reset button clear all scores and rounds?

---
## 🏆 Congratulations!
You have successfully built a complete, feature-rich, and polished Tic-Tac-Toe game in VR. You've learned how to set up a modern Unity VR project, implement a robust player rig, build interactive world-space UI, and architect clean, data-driven gameplay logic. You can now use these skills as a foundation for creating even more complex and exciting VR experiences.
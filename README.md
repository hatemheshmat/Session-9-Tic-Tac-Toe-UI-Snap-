# VR Tic-Tac-Toe: Educator & Student Guide (Unity 6.2, Meta SDK v65+)

This guide provides a complete walkthrough for creating a VR Tic-Tac-Toe game. It is designed for educators leading a session and for students to follow along. The tutorial covers project setup, VR rig implementation, UI creation, core gameplay logic, and advanced features, all using the latest recommended practices.

---

# 🔹 Part 1/5 — Project Setup (Unity 6.2 • URP • Meta XR AIO SDK v65+)

**Goal:** Configure a new Unity project from scratch for optimal performance and compatibility with the Meta Quest 2/3 platform.

## ✅ To-Do Checklist (with why)

### 1) Create the project
* ⬜ **1.1** Unity Hub → **New Project** → **3D (URP)** → name it `VR_TicTacToe_Tutorial`.
  * • **Why:** The Universal Render Pipeline (URP) is the standard for performant graphics on mobile VR platforms like Quest.

### 2) Switch Platform & Set Texture Compression
* ⬜ **2.1** Go to **File → Build Settings**.
* ⬜ **2.2** Select **Android** from the platform list and click **Switch Platform**.
  * • **Why:** The Quest OS is a modified version of Android. Switching early prevents unnecessary re-importing of assets later.
* ⬜ **2.3** In the same window, set **Texture Compression** to **ASTC**.
  * • **Why:** ASTC offers the best balance of quality and memory usage for Quest development.

### 3) Configure Player Settings
* ⬜ **3.1** Go to **Edit → Project Settings → Player**.
* ⬜ **3.2** Under **Other Settings**, configure the following:
    * • **Package Name:** `com.YourCompany.TicTacToe` (must be unique).
    * • **Minimum API Level:** Set to **API Level 29** or higher.
    * • **Target API Level:** Set to **Highest Installed**.
    * • **Scripting Backend:** Set to **IL2CPP**.
    * • **Target Architectures:** Check **ARM64** only.
  * • **Why:** These settings are required by the Meta Quest Store and ensure your app uses the fastest, most modern architecture.

### 4) Install and Configure Meta XR SDK
* ⬜ **4.1** Go to **Window → Package Manager**.
* ⬜ **4.2** Ensure the **Unity Registry** is selected. Search for and install the **Meta XR All-in-One SDK**.
* ⬜ **4.3** Go to **Edit → Project Settings → XR Plug-in Management**.
* ⬜ **4.4** In the **Android** tab, enable **Meta XR**.
  * • **Why:** This activates the Meta runtime, allowing your app to communicate with the Quest hardware.

### 5) Run Project Validation
* ⬜ **5.1** Go to **Edit → Project Settings → XR Plug-in Management → Project Validation**.
* ⬜ **5.2** Click the **Fix All** button to apply Meta's recommended project settings.
  * • **Why:** This tool automatically configures critical settings like input handling, permissions, and graphics options, saving time and preventing common issues.

### 6) Optimize URP for Performance
* ⬜ **6.1** In the **Project** window, find your **URP Asset** (e.g., `UniversalRP-HighQuality`).
* ⬜ **6.2** Select it and, in the Inspector, set **MSAA** to **4x** and disable **HDR**.
  * • **Why:** 4x MSAA provides a good anti-aliasing level for VR without a major performance hit. HDR is unnecessary and costly on Quest.

---

# 🔹 Part 2/5 — Scene, VR Rig, and Board UI

**Goal:** Set up a basic scene with a modern VR player rig, a physical wall, and a world-space canvas to serve as the game board.

## ✅ To-Do Checklist

### 1) Scene and Folder Setup
* ⬜ **1.1** Create folders in `Assets`: `_Scenes`, `_Scripts`, `_Prefabs`, `_Materials`.
* ⬜ **1.2** Create a new scene: **File → New Scene (Basic 3D)** → save it as `Assets/_Scenes/TicTacToe.unity`.
* ⬜ **1.3** In the Hierarchy, **delete** the default **Main Camera**.
  * • **Why:** The VR rig we are about to add includes its own tracked camera.

### 2) Add the Meta VR Rig
* ⬜ **2.1** In the **Project** window, search for `OVRCameraRig`.
* ⬜ **2.2** Drag the **OVRCameraRig** prefab into the Hierarchy.
* ⬜ **2.3** With `OVRCameraRig` selected, go to the Inspector and click **Add Component**. Search for and add the **OVR Manager** script.
  * • **Why:** `OVRCameraRig` provides the head and hand tracking, while `OVRManager` is the central hub that configures the connection to the headset.

### 3) Build the Wall and Board
* ⬜ **3.1** Create a wall: **Hierarchy → 3D Object → Cube**. Rename it `Wall`.
* ⬜ **3.2** Set its **Transform** in the Inspector:
    * • **Position:** (0, 1.5, 2.5)
    * • **Scale:** (3, 2, 0.1)
* ⬜ **3.3** Create the game board: **Hierarchy → UI → Canvas**. Rename it `BoardCanvas`.
* ⬜ **3.4** Configure the `BoardCanvas`:
    * • Set **Render Mode** to **World Space**.
    * • Set **Position** to (0, 1.5, 2.44) to place it just in front of the wall.
    * • Set **Width** to 800, **Height** to 800.
    * • Set **Scale** to (0.001, 0.001, 0.001).
  * • **Why:** A world-space canvas exists in the 3D scene, allowing VR users to interact with it directly. The tiny scale is necessary to map its large pixel dimensions into reasonable world units (meters).

### 4) Add the 3x3 Grid and Cell Prefab
* ⬜ **4.1** Right-click `BoardCanvas` → **Create Empty**. Rename it `GridContainer`.
* ⬜ **4.2** Add a **Grid Layout Group** component to `GridContainer`. Configure it:
    * • **Cell Size:** (250, 250)
    * • **Spacing:** (20, 20)
    * • **Constraint:** Fixed Column Count, **Constraint Count:** 3
* ⬜ **4.3** Right-click `GridContainer` → **UI → Button (TextMeshPro)**. Rename it `Cell`.
* ⬜ **4.4** Drag the `Cell` from the Hierarchy into `Assets/_Prefabs` to create a prefab. Delete the original from the Hierarchy.
* ⬜ **4.5** Drag the **`Cell` prefab** back into the `GridContainer` **nine times**.
* ⬜ **4.6** Rename the instances `Cell_0` to `Cell_8` for clarity.

### 5) Finalize UI Setup
* ⬜ **5.1** Select the `EventSystem` in the Hierarchy. Click **Add Component** and add the **OVR Input Module**.
* ⬜ **5.2** Select the `BoardCanvas`. Click **Add Component** and add the **OVR Raycaster**.
  * • **Why:** These two components work together to translate VR controller rays into UI clicks that Unity's Event System can understand.

---

# 🔹 Part 3/5 — Core Gameplay Logic

**Goal:** Implement a robust, data-driven game logic where the state is managed centrally and UI is updated in response.

## ✅ To-Do Checklist

### 1) Create the Core Scripts
* ⬜ **1.1** In `Assets/_Scripts`, create a new folder named `TicTacToe`.
* ⬜ **1.2** Inside this folder, create two C# scripts: `GameController.cs` and `Cell.cs`.

### 2) Code the `Cell.cs` Script
* ⬜ **2.1** Open `Cell.cs` and replace its content with the following:
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
        if (_gameController != null)
        {
            _gameController.OnCellClicked(_cellIndex);
        }
    }
}
```
* ⬜ **2.2** Open the **`Cell` prefab** and add the **`Cell` script** to it. Save the prefab.

### 3) Code the `GameController.cs` Script
* ⬜ **3.1** Open `GameController.cs` and replace its content with this code:
```csharp
using UnityEngine;
using UnityEngine.UI;
using TMPro;

public class GameController : MonoBehaviour
{
    [Header("Board & Cells")]
    public Button[] cellButtons = new Button[9];

    [Header("Game State")]
    private int[] boardState = new int[9]; // 0=Empty, 1=X, 2=O
    private int currentPlayer = 1;
    private int movesMade = 0;
    private bool isGameActive = true;

    [Header("UI Panels & Text")]
    public GameObject gameOverPanel;
    public TMP_Text gameOverText;
    public Button restartButton;

    private readonly int[][] winConditions =
    {
        new[] {0, 1, 2}, new[] {3, 4, 5}, new[] {6, 7, 8}, // Rows
        new[] {0, 3, 6}, new[] {1, 4, 7}, new[] {2, 5, 8}, // Columns
        new[] {0, 4, 8}, new[] {2, 4, 6}                  // Diagonals
    };

    void Start()
    {
        // Add listeners for UI buttons
        restartButton.onClick.AddListener(RestartGame);

        // Initialize each cell with its index and a reference to this controller
        for (int i = 0; i < cellButtons.Length; i++)
        {
            Cell cell = cellButtons[i].GetComponent<Cell>();
            if (cell != null)
            {
                cell.Initialize(i, this);
            }
        }

        RestartGame();
    }

    public void RestartGame()
    {
        isGameActive = true;
        currentPlayer = 1;
        movesMade = 0;

        for (int i = 0; i < boardState.Length; i++)
        {
            boardState[i] = 0;
            UpdateCellUI(i);
            cellButtons[i].interactable = true;
        }

        gameOverPanel.SetActive(false);
    }

    public void OnCellClicked(int cellIndex)
    {
        if (!isGameActive || boardState[cellIndex] != 0) return;

        boardState[cellIndex] = currentPlayer;
        movesMade++;

        UpdateCellUI(cellIndex);
        cellButtons[cellIndex].interactable = false;

        if (CheckForWin())
        {
            EndGame(false);
        }
        else if (movesMade >= 9)
        {
            EndGame(true); // Tie
        }
        else
        {
            currentPlayer = (currentPlayer == 1) ? 2 : 1;
        }
    }

    private bool CheckForWin()
    {
        foreach (var condition in winConditions)
        {
            if (boardState[condition[0]] != 0 &&
                boardState[condition[0]] == boardState[condition[1]] &&
                boardState[condition[0]] == boardState[condition[2]])
            {
                return true;
            }
        }
        return false;
    }

    private void EndGame(bool isTie)
    {
        isGameActive = false;
        gameOverPanel.SetActive(true);
        gameOverText.text = isTie ? "It's a Tie!" : $"Player {(currentPlayer == 1 ? "X" : "O")} Wins!";
    }

    private void UpdateCellUI(int cellIndex)
    {
        TMP_Text textComponent = cellButtons[cellIndex].GetComponentInChildren<TMP_Text>();
        if (textComponent != null)
        {
            switch (boardState[cellIndex])
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
* ⬜ **4.1** Create an empty GameObject in the Hierarchy named `GameManager`.
* ⬜ **4.2** Add the **`GameController.cs`** script to `GameManager`.
* ⬜ **4.3** In the Inspector, lock the `GameManager` view (top-right lock icon).
* ⬜ **4.4** In the Hierarchy, select all nine `Cell_` objects and drag them onto the **Cell Buttons** array field in the `GameController`.
* ⬜ **4.5** Create the Game Over UI:
    * • Right-click `BoardCanvas` → **UI → Panel**. Rename it `GameOverPanel`.
    * • Inside `GameOverPanel`, add a **TextMeshPro - Text** for the status and a **Button** to restart.
* ⬜ **4.6** Drag the `GameOverPanel`, its text, and its restart button into the corresponding fields on the `GameController`.
* ⬜ **4.7** Disable the `GameOverPanel` in the Hierarchy.

---

# 🔹 Part 4/5 — UX Polish (Haptics, Audio & Hover Effects)

**Goal:** Enhance the user experience with sensory feedback for clicks, wins, and hover states.

## ✅ To-Do Checklist

### 1) Add Hover Effect
* ⬜ **1.1** Create a new script `Assets/_Scripts/TicTacToe/CellHoverScale.cs`.
* ⬜ **1.2** Paste this code:
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
* ⬜ **1.3** Open the **`Cell` prefab** and add the **`CellHoverScale`** script to it.

### 2) Add Haptics and Audio
* ⬜ **2.1** In `GameController.cs`, add these fields:
```csharp
[Header("Audio & Haptics")]
public AudioSource audioSource;
public AudioClip clickClip;
public AudioClip winClip;
```
* ⬜ **2.2** In `GameController.cs`, add this haptics helper method:
```csharp
private void TriggerHaptics(float amplitude, float duration)
{
    OVRInput.SetControllerVibration(1, amplitude, OVRInput.Controller.RTouch);
    Invoke(nameof(StopHaptics), duration);
}

private void StopHaptics()
{
    OVRInput.SetControllerVibration(0, 0, OVRInput.Controller.RTouch);
}
```
* ⬜ **2.3** In `OnCellClicked()`, after updating the state, call:
```csharp
audioSource.PlayOneShot(clickClip);
TriggerHaptics(0.2f, 0.1f);
```
* ⬜ **2.4** In `EndGame()`, after showing the panel, call:
```csharp
audioSource.PlayOneShot(winClip);
TriggerHaptics(0.5f, 0.3f);
```
* ⬜ **2.5** On the `GameManager` object, add an **Audio Source** component.
* ⬜ **2.6** Import two sound clips (`click.wav`, `win.wav`) and assign them and the Audio Source to the `GameController`'s fields in the Inspector.

---

# 🔹 Part 5/5 — Advanced Features (HUD, Undo, Scoring & Debugging)

**Goal:** Add a persistent Heads-Up Display (HUD), an undo feature, scoring across rounds, and a debug overlay to create a complete game loop.

## ✅ To-Do Checklist

### 1) Build the HUD
* ⬜ **1.1** On the `BoardCanvas`, create a new **Panel** named `HUDPanel`. Anchor it to the top.
* ⬜ **1.2** Inside `HUDPanel`, add **TextMeshPro - Text** elements for: `TurnText`, `ScoreText`, and `RoundText`.
* ⬜ **1.3** Add two **Buttons**: `UndoButton` and `ResetAllButton`.

### 2) Extend `GameController.cs` for Advanced Features
* ⬜ **2.1** Add new fields to `GameController` for the HUD elements, scores, and move history:
```csharp
[Header("HUD UI")]
public TMP_Text turnText;
public TMP_Text scoreText;
public TMP_Text roundText;
public Button undoButton;
public Button resetAllButton;

private int scoreX = 0;
private int scoreO = 0;
private int currentRound = 1;
private System.Collections.Generic.Stack<int> moveHistory = new System.Collections.Generic.Stack<int>();
```
* ⬜ **2.2** Create a new `UpdateHUD()` method to refresh all text elements and call it whenever the state changes (e.g., in `RestartGame`, `OnCellClicked`, `EndGame`).
* ⬜ **2.3** Implement `UndoLastMove()`:
```csharp
public void UndoLastMove()
{
    if (moveHistory.Count == 0 || !isGameActive) return;
    int lastMove = moveHistory.Pop();
    boardState[lastMove] = 0;
    movesMade--;
    cellButtons[lastMove].interactable = true;
    currentPlayer = (currentPlayer == 1) ? 2 : 1;
    UpdateCellUI(lastMove);
    UpdateHUD();
}
```
* ⬜ **2.4** Implement `ResetAllStats()` to reset scores and rounds.
* ⬜ **2.5** In `OnCellClicked()`, push the `cellIndex` to `moveHistory`.
* ⬜ **2.6** In `EndGame()`, increment the score for the winning player.
* ⬜ **2.7** Wire up the `UndoButton` and `ResetAllButton`'s `onClick` events in the `Start()` method.

### 3) Add a Debug Overlay
* ⬜ **3.1** Create a new script `DebugOverlay.cs` that reads the private state of `GameController` using reflection and displays it on a toggleable panel.
* ⬜ **3.2** Create a new panel and text object on the `BoardCanvas` for the debug info.
* ⬜ **3.3** Add a "Debug" button to the HUD to toggle the `DebugPanel`'s visibility.

### 4) Final Playtest
* ⬜ **4.1** Test all features:
    * • Does the HUD update correctly?
    * • Does the Undo button work as expected?
    * • Does the score increment across rounds?
    * • Does the Reset All button clear all stats?
    * • Does the Debug panel show the correct internal state?

Congratulations! You have successfully built a complete, feature-rich Tic-Tac-Toe game in VR, following modern development practices.
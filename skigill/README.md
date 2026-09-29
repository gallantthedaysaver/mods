# Skigill — Starting Resource Mod (dnSpy Method)

This guide documents the process used to modify Skigill's starting in-run resource (the gold-like currency used on a level's tech tree).

## Tested environment

- Game: Skigill
- Platform: Windows / Steam
- Unity version: `6000.2.7f2`
- Runtime: Unity Mono
- Tool used for code inspection/editing: dnSpyEx

> This method directly edits `Assembly-CSharp.dll`. Steam updates or file verification may overwrite the change. Always keep a backup of the original DLL.

## 1. Locate the game assembly

Open the Skigill install folder:

```text
C:\Program Files (x86)\Steam\steamapps\common\Skigill
```

Then go to:

```text
Skigill_Data\Managed
```

The file we need is:

```text
Assembly-CSharp.dll
```

Before changing anything, copy it somewhere safe or make a backup such as:

```text
Assembly-CSharp.dll.original
```

## 2. Open the assembly in dnSpyEx

1. Launch dnSpyEx.
2. Choose **File → Open**.
3. Open:

```text
C:\Program Files (x86)\Steam\steamapps\common\Skigill\Skigill_Data\Managed\Assembly-CSharp.dll
```

4. Expand `Assembly-CSharp` in the Assembly Explorer.

## 3. Find the relevant code

The useful chain is:

```text
CharaStats.metaMoney
    ↓
launchScript.MONEY
    ↓
CharaStats.Start()
```

The `launchScript.launch()` method contains the assignment that determines starting money.

Open:

```text
launchScript → launch()
```

The original relevant line is:

```csharp
this.MONEY += component.metaMoney;
```

This means Skigill takes the amount granted by the meta/unlock tree (`component.metaMoney`) and adds it to the launch-time money value.

## 4. Edit the starting resource

Right-click `launch()` and choose:

```text
Edit Method (C#)
```

To add a flat bonus while preserving normal unlock progression, change:

```csharp
this.MONEY += component.metaMoney;
```

to something like:

```csharp
this.MONEY += component.metaMoney + 5000f;
```

With this example, a normal starting value of 90 becomes 5,090.

If you want to force an exact amount regardless of unlocks, use:

```csharp
this.MONEY = 5000f;
```

instead.

## 5. Fix the dnSpy compile ambiguity

When compiling the edited method, dnSpy may report:

```text
CS0104: 'Object' is an ambiguous reference between 'UnityEngine.Object' and 'object'
```

The problematic line is:

```csharp
Object.DontDestroyOnLoad(base.gameObject);
```

Change it to:

```csharp
UnityEngine.Object.DontDestroyOnLoad(base.gameObject);
```

Then compile again.

## 6. Compile and save

1. Click **Compile** in the Edit Method window.
2. If compilation succeeds, close Skigill completely if it is running.
3. In dnSpy, choose **File → Save Module…** or **Save All…** depending on your dnSpy build.
4. Save the edited module back over:

```text
C:\Program Files (x86)\Steam\steamapps\common\Skigill\Skigill_Data\Managed\Assembly-CSharp.dll
```

5. Launch Skigill through Steam and start a run to verify the starting resource amount.

## What was discovered

The game's starting resource is passed through these fields/methods:

- `CharaStats.metaMoney` — meta-progression/unlock-tree contribution
- `launchScript.MONEY` — launch-time starting resource value
- `launchScript.launch()` — adds the meta-progression value into the launch data
- `CharaStats.Start()` — reads `launchScript.MONEY` after the gameplay scene loads

That makes `launchScript.launch()` the most direct place to modify the starting amount using this manual method.

## Restoring the original game

If something breaks:

1. Replace the modified `Assembly-CSharp.dll` with your backup, or
2. In Steam, use **Properties → Installed Files → Verify integrity of game files**.

Verifying files will restore the original DLL and remove the manual modification.

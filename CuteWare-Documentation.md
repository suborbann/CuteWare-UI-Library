# CuteWare UI Library — Documentation

A Rayfield/Orion-style UI library for Roblox: windows, tabs, sections, and a full set of interactive components, with smooth built-in animations and an optional key system.

- Discord: https://dsc.gg/cuteware

---

## Table of Contents

1. [Installation](#installation)
2. [CuteWare:Init()](#cutewareinit)
3. [CuteWare:CreateKeySystem()](#cutewarecreatekeysystem)
4. [CuteWare:CreateWindow()](#cutewarecreatewindow)
5. [Window Methods](#window-methods)
6. [Window:CreateTab()](#windowcreatetab)
7. [Tab:CreateSection()](#tabcreatesection)
8. [Section Elements](#section-elements)
   - [CreateButton](#sectioncreatebutton)
   - [CreateCloseButton](#sectioncreateclosebutton)
   - [CreateToggle](#sectioncreatetoggle)
   - [CreateSlider](#sectioncreateslider)
   - [CreateDropdown](#sectioncreatedropdown)
   - [CreateMenuKeybindDropdown](#sectioncreatemenukeybinddropdown)
   - [CreateColorPicker](#sectioncreatecolorpicker)
   - [CreateTextbox](#sectioncreatetextbox)
   - [CreateKeybind](#sectioncreatekeybind)
   - [CreateLabel](#sectioncreatelabel)
9. [Notifications](#notifications)
   - [CuteWare:Notify()](#cutewarenotify)
   - [CuteWare:NotifyBig()](#cutewarenotifybig)
10. [CuteWare.Flags](#cutewareflags)
11. [Full Example](#full-example)

---

## Installation

```lua
local CuteWare = loadstring(game:HttpGet("https://raw.githubusercontent.com/suborbann/CuteWare-UI-Library/refs/heads/main/source"))()
CuteWare:Init()
```

`Init()` must be called once, before anything else. It destroys any previous CuteWare instance (safe for re-executing), sets up notification holders, and plays a short startup sound.

---

## CuteWare:Init()

```lua
CuteWare:Init()
```

No parameters. Returns `CuteWare` itself (so you can chain if you want). Call this first, always.

---

## CuteWare:CreateKeySystem()

Optional whitelist-key gate. Call it **before** `CreateWindow()` — it blocks script execution (yields) until the player enters a valid key or closes the prompt. The screen blurs while it's open.

```lua
local granted = CuteWare:CreateKeySystem({
    Enabled = true,                 -- false = skipped instantly, returns true
    Key     = "mykey123",           -- string, or {"key1", "key2"} for multiple valid keys
    Title   = "cuteware — key system",
    Note    = "Join our Discord to get a key: dsc.gg/cuteware",
    OnClose = function()
        -- optional: called if the player closes the prompt WITHOUT a valid key.
        -- If you don't provide this, CuteWare raises a script error instead
        -- (safe default — stops execution without kicking the player).
    end,
})
```

| Field     | Type                 | Default | Description                                            |
|-----------|----------------------|---------|----------------------------------------------------------|
| `Enabled` | boolean              | —       | If `false`, the whole call is a no-op that returns `true` immediately. |
| `Key`     | string \| string[]   | `""`    | The accepted key(s).                                     |
| `Title`   | string               | "cuteware — key system" | Header text.                              |
| `Note`    | string               | "Enter your key to continue." | Body/subtitle text.                |
| `OnClose` | function             | nil     | Called if the player hits the × without entering a valid key. |

Returns `true` if the key was accepted, `false` if closed manually (and `OnClose` was provided instead of the default error).

---

## CuteWare:CreateWindow()

```lua
local Window = CuteWare:CreateWindow({
    Name                        = "cuteware",
    Subtitle                    = "uwu executor menu",   -- omit/nil for a larger centered title
    Size                        = UDim2.new(0, 560, 0, 380),
    Icon                        = 123456789,             -- asset id shown left of the title
    BackgroundImage             = "https://example.com/bg.png", -- asset id, rbxassetid://, or http(s) link
    BackgroundImageTransparency = 0.75,
    ToggleKeybind                = Enum.KeyCode.RightControl,   -- default menu show/hide key
})
```

| Field                          | Type              | Default                    | Notes |
|---------------------------------|-------------------|-----------------------------|-------|
| `Name`                          | string            | "CuteWare"                  | Window title. |
| `Subtitle`                      | string \| nil     | nil                         | If omitted, the title renders bigger and centered vertically instead. |
| `Size`                          | UDim2             | `0,560,0,380`               | Starting window size. |
| `Icon`                          | number/string/nil | nil                         | Shown to the left of the title. |
| `BackgroundImage`                | number/string/nil | nil                         | Asset id, `"rbxassetid://..."`, or a raw `http(s)://` link (see note below). |
| `BackgroundImageTransparency`   | number            | 0.75                        | 0 = opaque, 1 = invisible. |
| `ToggleKeybind`                  | Enum.KeyCode      | `Enum.KeyCode.RightControl` | Key that shows/hides the whole window. |

> **About `BackgroundImage` links:** Roblox `ImageLabel`s can only ever display Roblox assets — they cannot stream an arbitrary web image directly. When you pass an `http(s)://` link, CuteWare downloads it and hands it to the client through the common `writefile` + `getcustomasset` executor pattern. If your executor doesn't expose those two functions, the image is silently skipped — upload it to Roblox and use the asset id instead for guaranteed support.

The window plays a "pop in" spawn animation (grows from small, centered on screen, with a springy ease) automatically — no setup needed.

---

## Window Methods

```lua
Window:Close()                          -- smooth fade + shrink, then hides the window
Window:Open()                           -- smooth grow + fade-in, shows the window again
Window:SetBackgroundImage(source, transparency)  -- change/'set the background image at runtime
Window:AddTabButton({ Name = "Discord", Callback = function() end })  -- pinned button under the tab list
Window:CreateTab({ Name = "Combat" })   -- see below
```

- `Window:Close()` / `Window:Open()` are what the built-in × button and the menu toggle keybind use internally. Wire your **own** custom close buttons to `Window:Close()` (e.g. via `Section:CreateCloseButton`, below) so the GUI always fades out gracefully instead of vanishing instantly.
- `Window:AddTabButton` adds a small utility button pinned to the bottom of the tab list (outside the scrollable tab area) — handy for things like "Discord" or "Unload".

---

## Window:CreateTab()

```lua
local CombatTab = Window:CreateTab({ Name = "Combat" })
```

Returns a `Tab` object. Only has one method: `CreateSection`.

---

## Tab:CreateSection()

```lua
local AimSection = CombatTab:CreateSection("Aimbot")
```

Returns a `Section` object — a titled card that all the interactive elements below get added to.

---

## Section Elements

### Section:CreateButton

```lua
Section:CreateButton({
    Name     = "Print Flags",
    Callback = function() end,
})
```

### Section:CreateCloseButton

A ready-made button that closes the whole window via `Window:Close()` (smooth fade), so you never have to wire up your own close logic:

```lua
Section:CreateCloseButton({
    Name     = "Close Menu",     -- optional, defaults to "Close"
    Callback = function() end,   -- optional, runs AFTER the close animation starts
})
```

### Section:CreateToggle

```lua
local myToggle = Section:CreateToggle({
    Name         = "Aimbot Enabled",
    Flag         = "AimbotEnabled",   -- key in CuteWare.Flags
    CurrentValue = false,
    Callback     = function(state) end,
})

myToggle:Set(true)          -- set programmatically, fires Callback
myToggle:Set(true, false)   -- set programmatically WITHOUT firing Callback
```

### Section:CreateSlider

```lua
Section:CreateSlider({
    Name         = "Aimbot FOV",
    Range        = {10, 500},
    Increment    = 5,
    CurrentValue = 90,
    Flag         = "AimbotFOV",
    Callback     = function(value) end,
})
```

### Section:CreateDropdown

```lua
-- single-select
Section:CreateDropdown({
    Name          = "Target Priority",
    Mode          = "single",             -- default
    Options       = {"Closest", "Lowest Health", "Random"},
    CurrentOption = "Closest",
    Flag          = "TargetPriority",
    Callback      = function(selected) end,  -- string
})

-- multi-select
Section:CreateDropdown({
    Name          = "Game Modes",
    Mode          = "multi",
    Options       = {"Deathmatch", "CTF", "KOTH"},
    CurrentOption = {"Deathmatch"},
    Flag          = "GameModes",
    Callback      = function(selected) end,  -- table
})
```

### Section:CreateMenuKeybindDropdown

Ready-made dropdown for changing which key shows/hides the whole window at runtime:

```lua
Section:CreateMenuKeybindDropdown({
    Name          = "Menu Keybind",
    CurrentOption = "RCtrl",   -- one of: RCtrl, F, RShift, RAlt, LAlt, K, M
    Callback      = function(name, keyCode) end,
})
```

### Section:CreateColorPicker

A single hue strip (top) + saturation/value box (below), toggled open by clicking the color swatch.

```lua
Section:CreateColorPicker({
    Name         = "ESP Color",
    CurrentColor = Color3.fromRGB(255, 0, 0),
    Flag         = "ESPColor",
    Callback     = function(color) end,   -- Color3
})
```

### Section:CreateTextbox

```lua
Section:CreateTextbox({
    Name            = "Player Name Filter",
    PlaceholderText = "Enter player name...",
    CurrentValue    = "",
    Flag            = "PlayerNameFilter",
    Callback        = function(text, enterPressed) end,
})
```

### Section:CreateKeybind

```lua
Section:CreateKeybind({
    Name           = "Aim Keybind",
    CurrentKeybind = Enum.KeyCode.Q,
    Flag           = "AimKeybind",
    Callback       = function(key) end,   -- fires when the bind is CHANGED
    Pressed        = function() end,      -- fires every time the bound key is PRESSED
})
```

### Section:CreateLabel

```lua
Section:CreateLabel("💡 Tip: this is a plain wrapped text note.")
```

---

## Notifications

### CuteWare:Notify()

Small toast in the corner of the screen.

```lua
CuteWare:Notify({
    Title    = "cuteware",
    Content  = "UI Library loaded successfully!",
    Duration = 4,              -- seconds; 0 = stays until the player clicks the × (infinite)
    Position = "TopRight",     -- or "BottomRight"
})
```

Every notification gets a × close button. If `Duration > 0`, it also shows a countdown ("Ns") and a depleting progress bar underneath — set `Duration = 0` for a persistent notification the player must dismiss manually.

### CuteWare:NotifyBig()

A larger, modal-style notification centered in the window, optionally with an icon and up to two buttons.

```lua
CuteWare:NotifyBig({
    Title    = "Welcome to CuteWare!",
    Content  = "Click a button below to get started.",
    Duration = 6,
    Icon     = 123456789,     -- optional
    IconSide = 1,             -- 1 = left of text, 2 = above text
    Buttons  = {
        { Text = "Start", Callback = function() end },
        { Text = "Later", Callback = function() end },
    },
})
```

---

## CuteWare.Flags

Every element with a `Flag` field writes its current value here, keyed by that flag name — handy for reading all settings at once (e.g. for a config save system):

```lua
for flagName, value in pairs(CuteWare.Flags) do
    print(flagName, value)
end
```

---

## Full Example

```lua
local CuteWare = loadstring(game:HttpGet("https://raw.githubusercontent.com/suborbann/CuteWare-UI-Library/refs/heads/main/source"))()
CuteWare:Init()

CuteWare:CreateKeySystem({ Enabled = false }) -- flip to true + set a Key to require one

local Window = CuteWare:CreateWindow({
    Name     = "cuteware",
    Subtitle = "uwu executor menu",
})

local Tab = Window:CreateTab({ Name = "Main" })
local Section = Tab:CreateSection("General")

Section:CreateToggle({
    Name = "Enabled", Flag = "Enabled",
    Callback = function(state) print(state) end,
})

Section:CreateSlider({
    Name = "Speed", Range = {1, 10}, CurrentValue = 5,
    Callback = function(v) print(v) end,
})

Section:CreateColorPicker({
    Name = "Color", CurrentColor = Color3.fromRGB(255, 130, 180),
    Callback = function(c) print(c) end,
})

Section:CreateCloseButton()

CuteWare:Notify({ Title = "cuteware", Content = "Loaded!", Duration = 4 })
```

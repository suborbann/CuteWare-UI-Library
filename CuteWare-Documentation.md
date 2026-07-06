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

`Init()` must be called once, before anything else. It destroys any previous CuteWare instance (safe for re-executing), sets up notification holders, and plays a short startup sound. Takes no parameters.

---

## CuteWare:Init()

```lua
CuteWare:Init()
```

No parameters. Returns `CuteWare` itself (so you can chain if you want). Call this first, always.

---

## CuteWare:CreateKeySystem()

**Entirely optional.** Skip this call completely if you don't want a key system. Call it **before** `CreateWindow()` — it blocks script execution (yields) until the player enters a valid key or closes the prompt. The screen blurs while it's open.

```lua
local granted = CuteWare:CreateKeySystem({
    Enabled = true,                 -- required: false = skipped instantly, returns true
    Key     = "cutekey123",           -- required if Enabled: string, or {"key1", "key2"} for multiple valid keys
    Title   = "Key System",         -- optional
    Note    = "Enter your key to continue.", -- optional
    OnClose = function()            -- optional
        -- called if the player closes the prompt WITHOUT a valid key.
        -- If you don't provide this, CuteWare raises a script error instead
        -- (safe default — stops execution without kicking the player).
    end,
})
```

| Field     | Type                 | Required?                    | Default | Description |
|-----------|----------------------|-------------------------------|---------|--------------|
| `Enabled` | boolean              | **required**                  | —       | If `false`, the whole call is a no-op that returns `true` immediately. |
| `Key`     | string \| string[]   | required only if `Enabled = true` | `""` | The accepted key(s). |
| `Title`   | string               | optional                      | "Key System" | Header text. |
| `Note`    | string               | optional                      | "Enter your key to continue." | Body/subtitle text. |
| `OnClose` | function             | optional                      | nil     | Called if the player hits the × without entering a valid key. |

Returns `true` if the key was accepted, `false` if closed manually (and `OnClose` was provided instead of the default error).

---

## CuteWare:CreateWindow()

```lua
local Window = CuteWare:CreateWindow({
    Name                        = "cuteware",
    Subtitle                    = "a short subtitle",    -- optional
    Size                        = UDim2.new(0, 560, 0, 380),
    Icon                        = 77042574589502,             -- optional
    BackgroundImage             = "https://create.roblox.com/store/asset/75201790989057/windows", -- optional
    BackgroundImageTransparency = 0.75,                   -- optional
    ToggleKeybind               = Enum.KeyCode.RightControl,   -- optional
})
```

Every single field here is **optional** — `CuteWare:CreateWindow({})` (or even `CreateWindow()`) works fine and falls back to sensible defaults.

| Field                          | Type              | Optional? | Default                    | Notes |
|---------------------------------|-------------------|-----------|------------------------------|-------|
| `Name`                          | string            | optional  | "CuteWare"                  | Window title. |
| `Subtitle`                      | string            | optional  | none                        | If you don't set this, the title renders bigger and centered vertically instead. |
| `Size`                          | UDim2             | optional  | `0,560,0,380`               | Starting window size. |
| `Icon`                          | number/string     | optional  | none                        | Shown to the left of the title. |
| `BackgroundImage`                | number/string     | optional  | none                        | Asset id, `"rbxassetid://..."`, or a raw `http(s)://` link (see note below). |
| `BackgroundImageTransparency`   | number            | optional  | 0.75                        | 0 = opaque, 1 = invisible. Only relevant if `BackgroundImage` is set. |
| `ToggleKeybind`                  | Enum.KeyCode      | optional  | `Enum.KeyCode.RightControl` | Key that shows/hides the whole window. |

> **About `BackgroundImage` links:** Roblox `ImageLabel`s can only ever display Roblox assets — they cannot stream an arbitrary web image directly. When you pass an `http(s)://` link, CuteWare downloads it and hands it to the client through the common `writefile` + `getcustomasset` executor pattern. If your executor doesn't expose those two functions, the image is silently skipped — upload it to Roblox and use the asset id instead for guaranteed support.

The window plays a "pop in" spawn animation (grows from small, centered on screen, with a springy ease) automatically — no setup needed.

---

## Window Methods

```lua
Window:Close()                          -- smooth fade + shrink, then hides the window
Window:Open()                           -- smooth grow + fade-in, shows the window again
Window:SetBackgroundImage(source, transparency)  -- change the background image at runtime
Window:AddTabButton({ Name = "...", Callback = function() end })  -- optional, pinned button under the tab list
Window:CreateTab({ Name = "..." })      -- see below
```

- `Window:Close()` / `Window:Open()` are what the built-in × button and the menu toggle keybind use internally. Wire your **own** custom close buttons to `Window:Close()` (e.g. via `Section:CreateCloseButton`, below) so the GUI always fades out gracefully instead of vanishing instantly.
- `Window:AddTabButton` is **entirely optional** — only call it if you want an extra utility button pinned to the bottom of the tab list (outside the scrollable tab area).

---

## Window:CreateTab()

```lua
local Tab = Window:CreateTab({
    Name = "Tab",  -- optional, defaults to "Tab"
    Icon = 123456789, -- optional
})
```

| Field  | Type          | Optional? | Default | Notes |
|--------|---------------|-----------|---------|-------|
| `Name` | string        | optional  | "Tab"   | Text shown on the tab button. |
| `Icon` | number/string | optional  | none    | Small icon shown to the left of the tab name. |

Returns a `Tab` object. Only has one method: `CreateSection`.

---

## Tab:CreateSection()

```lua
local Section = Tab:CreateSection("Section") -- the name argument itself is optional
```

The name argument is **optional** — `Tab:CreateSection()` with no argument falls back to "Section". Returns a `Section` object — a titled card that all the interactive elements below get added to.

---

## Section Elements

Every element below takes a single config table. **Every field in every config table is optional** except where a table explicitly says "required" — omit anything you don't need and a sensible default is used instead.

### Section:CreateButton

```lua
Section:CreateButton({
    Name     = "Button",       -- optional
    Callback = function() end, -- optional
})
```

### Section:CreateCloseButton

A ready-made button that closes the whole window via `Window:Close()` (smooth fade), so you never have to wire up your own close logic:

```lua
Section:CreateCloseButton({
    Name     = "Close",         -- optional, defaults to "Close"
    Callback = function() end,  -- optional, runs AFTER the close animation starts
})
```

Can even be called with no arguments at all: `Section:CreateCloseButton()`.

### Section:CreateToggle

```lua
local myToggle = Section:CreateToggle({
    Name         = "Toggle",      -- optional
    Flag         = "MyToggle",    -- optional, key in CuteWare.Flags
    CurrentValue = false,         -- optional, defaults to false
    Callback     = function(state) end, -- optional
})

myToggle:Set(true)          -- set programmatically, fires Callback
myToggle:Set(true, false)   -- set programmatically WITHOUT firing Callback
```

### Section:CreateSlider

```lua
Section:CreateSlider({
    Name         = "Slider",   -- optional
    Range        = {0, 100},   -- optional, defaults to {0, 100}
    Increment    = 1,          -- optional, defaults to 1
    CurrentValue = 0,          -- optional, defaults to the minimum of Range
    Flag         = "MySlider", -- optional
    Callback     = function(value) end, -- optional
})
```

### Section:CreateDropdown

```lua
-- single-select (default mode)
Section:CreateDropdown({
    Name          = "Dropdown",         -- optional
    Mode          = "single",           -- optional, defaults to "single"
    Options       = {"Option A", "Option B", "Option C"}, -- optional, defaults to {}
    CurrentOption = "Option A",         -- optional, defaults to the first option
    Flag          = "MyDropdown",       -- optional
    Callback      = function(selected) end, -- optional, receives a string
})

-- multi-select
Section:CreateDropdown({
    Name          = "Dropdown",
    Mode          = "multi",
    Options       = {"Option A", "Option B", "Option C"},
    CurrentOption = {"Option A"},       -- optional, table of preselected options
    Flag          = "MyDropdown",
    Callback      = function(selected) end, -- receives a table
})
```

### Section:CreateMenuKeybindDropdown

Ready-made dropdown for changing which key shows/hides the whole window at runtime — every field is optional, including calling it with no arguments at all:

```lua
Section:CreateMenuKeybindDropdown({
    Name          = "Menu Keybind", -- optional
    CurrentOption = "RCtrl",        -- optional, one of: RCtrl, F, RShift, RAlt, LAlt, K, M
    Callback      = function(name, keyCode) end, -- optional
})
```

### Section:CreateColorPicker

A single hue strip (top) + saturation/value box (below), toggled open by clicking the color swatch.

```lua
Section:CreateColorPicker({
    Name         = "Color Picker",              -- optional
    CurrentColor = Color3.fromRGB(255, 255, 255), -- optional, defaults to white
    Flag         = "MyColor",                   -- optional
    Callback     = function(color) end,         -- optional, receives a Color3
})
```

### Section:CreateTextbox

```lua
Section:CreateTextbox({
    Name            = "Textbox",       -- optional
    PlaceholderText = "Enter text...", -- optional
    CurrentValue    = "",              -- optional
    Flag            = "MyTextbox",     -- optional
    Callback        = function(text, enterPressed) end, -- optional
})
```

### Section:CreateKeybind

```lua
Section:CreateKeybind({
    Name           = "Keybind",         -- optional
    CurrentKeybind = Enum.KeyCode.F,    -- optional, defaults to F
    Flag           = "MyKeybind",       -- optional
    Callback       = function(key) end, -- optional, fires when the bind is CHANGED
    Pressed        = function() end,    -- optional, fires every time the bound key is PRESSED
})
```

### Section:CreateLabel

```lua
Section:CreateLabel("Any text you want.") -- the text argument itself is optional, defaults to ""
```

---

## Notifications

### CuteWare:Notify()

Small toast in the corner of the screen. Every field is optional.

```lua
CuteWare:Notify({
    Title    = "Notification",  -- optional
    Content  = "",              -- optional
    Duration = 4,               -- optional; seconds, 0 = stays until the player clicks the × (infinite)
    Position = "TopRight",      -- optional, or "BottomRight"
})
```

Every notification gets a × close button. If `Duration > 0`, it also shows a countdown ("Ns") and a depleting progress bar underneath — set `Duration = 0` for a persistent notification the player must dismiss manually.

### CuteWare:NotifyBig()

A larger, modal-style notification centered in the window, optionally with an icon and up to two buttons. Every field is optional.

```lua
CuteWare:NotifyBig({
    Title    = "Notice",   -- optional
    Content  = "",         -- optional
    Duration = 5,          -- optional
    Icon     = 123456789,  -- optional
    IconSide = 1,          -- optional, 1 = left of text, 2 = above text
    Buttons  = {           -- optional, up to 2
        { Text = "OK", Callback = function() end },
    },
})
```

---

## CuteWare.Flags

Every element with a `Flag` field writes its current value here, keyed by that flag name — handy for reading all settings at once (e.g. for a config save system). `Flag` is optional on every element; if you don't set one, that element simply won't appear here.

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

CuteWare:CreateKeySystem({ Enabled = false }) -- optional; flip to true + set a Key to require one

local Window = CuteWare:CreateWindow({
    Name     = "My Menu",     -- optional
    Subtitle = "a subtitle",  -- optional
})

local Tab = Window:CreateTab({ Name = "Tab" }) -- Name is optional
local Section = Tab:CreateSection("Section")   -- argument is optional

Section:CreateToggle({
    Name = "Toggle", Flag = "Toggle", -- both optional
    Callback = function(state) print(state) end, -- optional
})

Section:CreateSlider({
    Name = "Slider", Range = {1, 10}, CurrentValue = 5, -- all optional
    Callback = function(v) print(v) end, -- optional
})

Section:CreateColorPicker({
    Name = "Color Picker", CurrentColor = Color3.fromRGB(255, 130, 180), -- both optional
    Callback = function(c) print(c) end, -- optional
})

Section:CreateCloseButton() -- no arguments needed at all

CuteWare:Notify({ Title = "Notification", Content = "Loaded!", Duration = 4 }) -- all optional
```

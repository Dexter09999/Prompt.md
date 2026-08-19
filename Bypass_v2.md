Build a complete Among Us mod framework in C# using BepInEx 5
plugin architecture. Target: latest Steam/Epic version. All features
fully implemented, no stubs, ready to compile.

== LOADER ==

BepInEx 5 plugin (single DLL) that loads into Among Us via standard
BepInEx injection. Features:
- Auto-patches on game start
- Config file saved to BepInEx/config/
- Hotkey system for toggling features
- Version check on load (warn if game updated)
- Plugin info: name "ObitoassClient", version, author

== CORE SYSTEMS ==

--- Event System ---
Custom event bus with Subscribe/Unsubscribe pattern:
- GameEvents.RoundStart
- GameEvents.RoundEnd
- GameEvents.PlayerSpawned
- GameEvents.PlayerDied
- GameEvents.MeetingCalled
- GameEvents.MeetingEnded
- GameEvents.TaskComplete
- GameEvents.SabotageStart
- GameEvents.HudUpdate
- GameEvents.Update (every frame)
- GameEvents.FixedUpdate (physics tick)
- GameEvents.LateUpdate
- GameEvents.OnGUI

--- Module Base ---
abstract class Module:
  - Name, Description, Keybind, Category, Enabled
  - abstract void OnEnable()
  - abstract void OnDisable()
  - abstract void OnUpdate()
  - abstract void OnGUI()
  - abstract void OnHUD()
  - Settings list: Toggle, Slider, Dropdown, Color, Keybind

--- Setting Types ---
- BoolSetting (toggle)
- FloatSetting (slider with min/max/step)
- IntSetting (slider with min/max/step)
- DropdownSetting (enum selection)
- ColorSetting (RGB picker)
- KeybindSetting (assignable key)

--- Config System ---
- Save/load all module states and settings to JSON
- Auto-save on round end
- Pre-built configs: "impostor", "crewmate", "troll", "subtle"
- Import/export from file
- Per-profile config support

== MODULES ==

--- ESP ---

PlayerESP
  - Show player positions through walls
  - Box ESP: 2D bounding box around each player
  - Name tag: player name above head
  - Role ESP: show role name with role color (Impostor=red,
    Crewmate=blue, Neutral=yellow)
  - Distance display: meters from local player
  - Health bar (when in meeting/voting)
  - Skeleton ESP: line connections between body parts
  - Visibility check: different color for visible vs hidden players
  - Through-walls rendering via overlay or GL hooks
  - Filter: show/hide dead players
  - Team filter: show only impostors, show only crew

TaskESP
  - Highlight tasks on the map
  - Show task names and completion status
  - Distance to nearest task
  - Color: incomplete=yellow, complete=green
  - Task list overlay on screen

DoorESP
  - Show open/closed door states
  - Color coding for sabotaged doors

ButtonESP
  - Highlight emergency meeting button
  - Highlight sabotage buttons
  - Show cooldown state

Minimap
  - Real-time minimap overlay
  - Show all player positions (based on ESP settings)
  - Show task locations
  - Show vents
  - Map background: transparent or solid
  - Zoom level configurable
  - Follow local player or fixed position

--- MOVEMENT ---

SpeedHack
  - Multiplier: 1.0x to 5.0x
  - Apply to PlayerControl.myPlayer
  - Affects movement speed only, not animations
  - Hotkey toggle
  - Server-synced option (limited by server validation)

NoClip
  - Walk through walls and objects
  - Disable collider checks on local player
  - Visual ghost effect (player model semi-transparent)
  - Toggle with hotkey

---

--- IMPOSTOR ---

KillThroughWalls
  - Remove line-of-sight check for kills
  - Kill any crewmate regardless of walls/obstacles
  - Range: configurable (default = server max kill distance)
  - Visual: show killable targets with red outline

KillCooldownBypass
  - Set kill cooldown to 0 or custom value
  - Instant kill ready after each kill
  - Server-side: only works on modded servers or host mode
  - Client-side: visual only on official servers

AlwaysImpostor
  - Force impostor role assignment (local/custom games only)
  - Works in freeplay and private lobbies
  - Set impostor count: 1-15

MultiKill
  - Kill multiple players in single action
  - Kill count configurable
  - Cooldown between multi-kills

--- SABOTAGE ---

SabotageControl
  - Trigger any sabotage remotely: Reactor, O2, Lights, Comms,
    Doors, Navigation
  - Cancel active sabotage instantly
  - Auto-sabotage on timer (every X seconds)
  - Priority system: which sabotage first

DoorControl
  - Open all doors simultaneously
  - Close all doors simultaneously
  - Lock specific doors
  - Auto-close timer
  - Free cam through doors

--- TASK ---

AutoTask
  - Complete tasks instantly (local/custom games only)
  - Complete all tasks in one click
  - Task progress override
  - Works on freeplay mode

TaskBar
  - Custom task bar display
  - Show progress percentage
  - Show remaining tasks

---

--- PLAYER ---

NameChanger
  - Change player name to custom text
  - Color: RGB picker
  - Size: configurable
  - Special characters support
  - Reset to default

ColorChanger
  - Change player color to any RGB value
  - In-game color sync
  - Hat/skin/pet selectors

SkinChanger
  - Unlock all hats, skins, pets, patterns
  - Apply any combination
  - Client-side visual only

Meetings
  - Auto-call meeting on keybind
  - Skip vote automatically
  - Auto-vote for specific player
  - Show who voted whom (reveal votes before reveal)
  - Timer display for meeting

--- RENDER -----

Fullbright
  - Override darkness/light levels
  - Gamma multiplier: 1x to 100x
  - Impostor vision in crewmate mode
  - Toggle: instant or smooth transition

Freecam
  - Detach camera from player
  - Fly camera with WASD + mouse
  - Speed: configurable
  - Return to body on key release
  - Time limit: configurable (prevent body desync)

Chams
  - Player models highlighted through walls
  - Color: role-based or custom
  - Glow effect: outline or full body highlight
  - Wireframe mode

CameraZoom
  - Zoom out further than default
  - Zoom level: 1x to 20x
  - Smooth zoom on scroll wheel

TrailESP
  - Show player movement paths
  - Color: trail color picker
  - Duration: how long trail persists
  - Width: line thickness

--- CLIENT ---

ModMenu
  - In-game overlay menu
  - Right SHIFT to open
  - Draggable panels by category
  - Module toggle: left click
  - Setting adjust: right click
  - Search bar for modules
  - Color theme: custom RGB
  - Category tabs: ESP, Movement, Impostor, Sabotage, Task, Player, Render, Client

HUD
  - Watermark: "Obitoass Client v1.0"
  - Active module list (sorted by name length)
  - FPS counter
  - Player position XYZ
  - Current role display
  - Kill cooldown timer
  - Meeting cooldown timer
  - Task progress bar
  - Player count (alive/dead/total)
  - Game phase indicator

Chat
  - Chat prefix: "!"
  - Commands:
    !toggle <module>
    !bind <module> <key>
    !config save <name>
    !config load <name>
    !config list
    !help
  - Chat spam: repeat message X times with delay
  - Custom chat colors (rich text support)

DiscordRPC
  - "Playing Among Us — Obitoass Client"
  - Show map name
  - Show game phase (lobby/playing/meeting)
  - Active module count
  - Session timestamp

---

--- UTILITY ---

GameInfo
  - Show map name and version
  - Show player list with roles (when available)
  - Show game code (for sharing lobbies)
  - Show host name and IP (if visible)
  - Player ping display

AntiAdminKick
  - Prevent admin from kicking (local only)

RevealRoles
  - See all player roles at all times
  - Works in meetings and during gameplay

BetterPolus
  - Enhanced Polus map features
  - Additional ESP points
  - Vent connections

== CODE STYLE ==

- C# (.NET Framework 4.x)
- BepInEx 5 plugin format
- Assembly-CSharp references for Among Us types
- Harmony patches for method hooks (Prefix, Postfix, Transpiler)
- UnityEngine for rendering, input, and component access
- Null checks on all player/object references (players leave mid-game)
- Thread-safe: Unity main thread only for game object access
- Clean separation: modules independent, no cross-dependencies
- Event-driven architecture, not polling where possible

== STRUCTURE ==

ObitoassAmongUs/
├── Plugin.cs                    (BepInEx entry, init, update loop)
├── Core/
│   ├── EventSystem/
│   │   ├── GameEvents.cs
│   │   └── EventHandler.cs
│   ├── ModuleSystem/
│   │   ├── Module.cs
│   │   ├── ModuleManager.cs
│   │   └── Category.cs
│   ├── SettingSystem/
│   │   ├── Setting.cs
│   │   ├── BoolSetting.cs
│   │   ├── FloatSetting.cs
│   │   ├── IntSetting.cs
│   │   ├── DropdownSetting.cs
│   │   ├── ColorSetting.cs
│   │   └── KeybindSetting.cs
│   ├── Config/
│   │   ├── ConfigManager.cs
│   │   ├── ConfigProfile.cs
│   │   └── configs/*.json
│   └── Utils/
│       ├── PlayerUtils.cs
│       ├── GameUtils.cs
│       ├── RenderUtils.cs
│       ├── InputUtils.cs
│       └── ColorUtils.cs
├── Modules/
│   ├── ESP/
│   │   ├── PlayerESP.cs
│   │   ├── TaskESP.cs
│   │   ├── DoorESP.cs
│   │   ├── ButtonESP.cs
│   │   └── Minimap.cs
│   ├── Movement/
│   │   ├── SpeedHack.cs
│   │   └── NoClip.cs
│   ├── Impostor/
│   │   ├── KillThroughWalls.cs
│   │   ├── KillCooldownBypass.cs
│   │   ├── AlwaysImpostor.cs
│   │   └── MultiKill.cs
│   ├── Sabotage/
│   │   ├── SabotageControl.cs
│   │   └── DoorControl.cs
│   ├── Task/
│   │   ├── AutoTask.cs
│   │   └── TaskBar.cs
│   ├── Player/
│   │   ├── NameChanger.cs
│   │   ├── ColorChanger.cs
│   │   ├── SkinChanger.cs
│   │   └── Meetings.cs
│   ├── Render/
│   │   ├── Fullbright.cs
│   │   ├── Freecam.cs
│   │   ├── Chams.cs
│   │   ├── CameraZoom.cs
│   │   └── TrailESP.cs
│   ├── Client/
│   │   ├── ModMenu.cs
│   │   ├── HUD.cs
│   │   ├── ChatCommands.cs
│   │   └── DiscordRPC.cs
│   └── Utility/
│       ├── GameInfo.cs
│       ├── AntiAdminKick.cs
│       ├── RevealRoles.cs
│       └── BetterPolus.cs
├── Patches/
│   ├── PlayerControlPatch.cs
│   ├── ShipStatusPatch.cs
│   ├── GameDataPatch.cs
│   ├── MeetingHudPatch.cs
│   ├── ReportDeadBodyPatch.cs
│   ├── ChatBubblePatch.cs
│   └── PhysicsPatch.cs
├── Render/
│   ├── Overlay.cs
│   ├── MenuRenderer.cs
│   ├── HUDRenderer.cs
│   ├── ESPRenderer.cs
│   └── MinimapRenderer.cs
├── Resources/
│   ├── ObitoassClient.dll.manifest
│   └── assets/ (icons, textures)
└── ObitoassAmongUs.csproj

== GENERATE ==

Generate the COMPLETE project. Every .cs file, every class, every
method, every Harmony patch, every render function. No stubs. No
"implement here" comments. Compilable with standard BepInEx 5
toolchain. Every module fully functional.

Include:
- .csproj with all required references
- BepInEx plugin manifest
- README with build instructions and usage
- Default config files
- Hotkey assignments for all modules

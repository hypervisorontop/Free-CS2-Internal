# DirectX11 ImGui Control Suite

Stylized, animated control menu built on Win32 + DirectX 11 using Dear ImGui. This example package demonstrates how to wire a modern settings surface—tabs, searchable controls, notifications, localization, color pickers, and config management—into a single loop that can live inside a host rendering surface (see `example_win32_directx11/`).

## Highlights
- **Tab-driven navigation** powered by `TabsManager`: five primary tabs (AimAssistance, Visualization, Weapon, Miscellaneous, Configs) with optional subtabs, transitions, and animated headers.
- **Search-first workflow** via `SearchManager`: every custom widget registers itself, the search bar fades in, and the overlay renders breadcrumb-like hints plus automatic filtering.
- **Config builder & toolpad**: the Configs tab shows a list of selectable presets, lets you type a name, create/delete/load configs, and fires `NotifyManager` toasts for every action.
- **Language selector** with `LangManager`: fonts + translation dictionaries switch between English and Russian text on the fly, and a compact button lives in the navigation column.
- **Notifications & popups**: `NotifyManager` renders fading status bars (success/error/info) while `PopupManager` can layer animated modals over the UI.
- **Custom controls suite**: `WidgetsManager` + `CompBuilder` provide bespoke checkboxes, sliders, combos, multi-combos, key binders, text fields with icons, color editors (with hex & alpha inputs plus saved palettes), and buttons with animated hover states.
- **Consistent theme** courtesy of `StyleManager`, animated helpers (`animations.hpp`), `FontManager`, `unicodes.hpp` icon glyphs, and embedded font/texture assets (`fonts.h`, `fonts`, `image.h`).

## Feature tour

### Tabbed layout and grouping
`GUI::initialize` registers pages per tab, each wrapped in `ChildManager` groups that write localized titles, manage inner padding, and deliver smooth scroll physics. The general page runs:
* checkboxes: Enable, Silent, Draw FOV, Enemies only, Visible only, Ignore bots (with optional color preview).
* numeric controls: `Field of view` (0°–180°), `Smooth` slider, recoil control system sliders for X/Y percentages, and custom combos (Standalone/Custom/Smart).
* toggles for Aim lock, Predict, Draw target, Save target, Triggerbot, Draw crosshair + rainbow/size options, Hide shots, and Delay (with 0.1s steps).
`TabsManager` drives page transitions and subtab rendering while `LangManager` keeps every caption localizable.

### Search + language + notifications
The leftmost column hosts the search bar, language selector, and tab rail:
* `SearchManager::additem` hooks every `WidgetsManager` entry, so typing `FOV` or `Triggerbot` shows the correct tab/subtab/child layout and still renders the original widget.
* Language picker uses icon fonts (`unicodes.hpp`) and animates the dropdown, letting you call `LangManager::set_lang` to swap dictionaries at runtime.
* `NotifyManager` draws toasts in the bottom-right corner whenever configs are created/loaded/deleted.
* `PopupManager` can take any lambda and slide it on top of the current window with fade-in/out (used for future confirmation dialogs or overlays).

### Custom widgets and palettes
`WidgetsManager` relies on `CompBuilder` primitives so every control shares animated hover/press states:
* Checkbox control renders a stylized square with a checkmark, optional warning offset, and color hooks.
* Sliders (int/float) use ImGui `SliderScalar` but still register with search.
* Combo and MultiCombo render custom dropdown windows with animated opens; MultiCombo summarizes the selection and truncates with `..` if it gets too long.
* TextField accepts icons, hints, and callbacks (search automatically re-registers the same widget when the user opens the overlay).
* ColorEdit opens a `ColorPickerManager` dialog that draws an HSV square, hue bar, hex/alpha text inputs, and a row of saved palettes you can recall as soon as you click one.
* Binder renders a key-binding button with icon support.
* Buttons emit stylized fills and text with `CompBuilder::Button`.
`SearchManager` and `WidgetsManager` share `LangManager` so even the hints are localized.

### Styling, fonts, and helpers
* `StyleManager` sets rounded corners, frame padding, scrollbar size, and a midnight-purple palette with a `ImGuiCol_Scheme` accent.
* `FontManager` loads the primary and icon fonts from the embedded arrays in `fonts.h`, exposing them through `fonts[font/icon].get(size)`.
* `ImageManager` can create a DirectX shader resource from an in-memory blob (`image.h`) whenever you need static textures inside the UI.
* `animations.hpp` provides helpers like `anim_obj`, `anim2`, and `col_anim` so transitions feel smooth.

## Directory tour
- `imgui_examples.sln` & `example_win32_directx11/`: Visual Studio solution and Win32 + DirectX 11 project that host the GUI loop (`main.cpp`, `gui.cpp`, and all managers).
- `example_win32_directx11/compbuilder`: reusable building blocks for buttons, checkboxes, sliders, combos, color editors, and binders with animation metadata.
- `example_win32_directx11/managers`: per-concern singletons (`FontManager`, `ImageManager`, `WidgetsManager`, `TabsManager`, `StyleManager`, `LangManager`, `PopupManager`, `NotifyManager`, `ChildManager`, `SearchManager`) plus helpers under `ColorPickerManager`.
- `example_win32_directx11/thirdparty`: FreeType binaries (`freetype64.lib`, `freetype.lib`) and helper headers (`animations.hpp`) used by the project.
- `packages/Microsoft.DXSDK.D3DX.9.29.952.8`: packaged DirectX SDK runtimes so the legacy `d3dx9_43.dll`, `d3dx10_43.dll`, `d3dx11_43.dll`, and `D3DCompiler_43.dll` are available without installing the SDK globally.
- `Release/` & `Release/output`: pre-built DLL (`example_win32_directx11.dll`), helper stub (`cs2-dumper.exe`), and metadata generated by the VS build to inspect for custom injections or scripting (CS/CS2 offsets, manifests, etc.).
- `fonts.h`, `fonts`, `image.h`, `unicodes.hpp`: embedded font and icon glyph blobs that keep the UI sharp on any machine.

## Getting started
1. **Requirements**: Windows 10/11, Visual Studio 2022 or later with the Desktop C++ workload, and the DX SDK runtimes (copied via the `packages/Microsoft.DXSDK...` directory). The project also links `freetype64.lib` (`thirdparty/lib`) through `#pragma comment`.
2. **Open the solution**: load `imgui_examples.sln`, pick the `x64` platform and `Release` configuration (it also builds a DLL target), and build normally.
3. **Command-line build**: run `examples\example_win32_directx11\build_win32.bat` to trigger the same MSBuild target shown in the solution.
4. **Run loop**: once compiled, the DLL exposes `GUI::initialize`/`draw`, so you can host it in any DirectX 11 frame loop by calling them between `ImGui_ImplDX11_NewFrame`/`ImGui_ImplWin32_NewFrame` and `ImGui::Render`.

## Customization tips
- Add new tabs via `TabsManager::add_page` inside `GUI::initialize` and use `ChildManager` when you need bordered sections.
- Use `SearchManager::get().additem(label, code)` everywhere you expose a widget so the search overlay knows about it.
- Extend `LangManager` by calling `add_language` with a font index and translation dictionary before `GUI::initialize`.
- Style tweaks belong in `StyleManager::Styles`/`Colors`; accent and border colors are centralized there.
- To expose additional textures, drop data into `image.h` and call `ImageManager::get().load(data, size, device)` before rendering.

## Credits
- Layout and controls built on [Dear ImGui](https://github.com/ocornut/imgui) (MIT license).
- DirectX 11 boilerplate courtesy of the legacy Microsoft DirectX SDK.
- FreeType 2 for font rasterization (`thirdparty/lib/freetype64.lib`).

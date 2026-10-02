# Noir UI

Librería de interfaz para Roblox, escrita en LuaU y pensada para interfaces de juego con tabs, subtabs, groupboxes y controles reutilizables. La API sigue en evolución: este documento describe el uso disponible en la versión actual, no garantiza compatibilidad futura.

## Archivos

- `New_Library_UI.lua`: fuente legible para desarrollo.
- `New_Library_UI.obfuscated.lua`: salida ofuscada recomendada para distribuir o cargar desde el hub.
- `Slayers2_UI.lua`: ejemplo de integración con el hub Slayers 2.

Mantén la fuente original en un repositorio privado y regenera la salida ofuscada después de cada cambio. La ofuscación dificulta la lectura, pero no impide que el código se recupere o analice.

## Requisitos

La biblioteca usa servicios estándar de Roblox, incluidos `Players`, `TweenService`, `UserInputService`, `GuiService` y `TextService`. Para cargar archivos externos desde un executor, hacen falta `readfile` y `loadstring`; para descargar desde una URL, el executor debe permitir `game:HttpGet` o `request`.

La gestión de configuraciones de Noir necesita las APIs de archivos del executor. Si no están disponibles, la ventana y los controles siguen funcionando, pero las configuraciones no se pueden guardar en disco.

## Cargar Como Módulo

Al cargar el archivo, Noir reconoce el marcador `__NoirLibraryModuleLoad` para devolver la biblioteca sin abrir su ventana de demostración. Ejemplo con un archivo local:

```lua
local env = type(getgenv) == "function" and getgenv() or _G
local flag = "__NoirLibraryModuleLoad"
local previous = rawget(env, flag)
rawset(env, flag, true)

local ok, Library = pcall(function()
    local source = readfile("New_Library_UI.obfuscated.lua")
    local chunk = assert(loadstring(source))
    return chunk()
end)

rawset(env, flag, previous)
assert(ok, Library)
```
```lua
getgenv().NoirLibraryPath = "Scripts/New_Library_UI.obfuscated.lua"
```

Como alternativa, configura `getgenv().NoirLibraryUrl` con una URL HTTPS directa al archivo Lua. Usa una referencia versionada (tag o commit) en vez de una rama mutable como `main`. Una URL raw pública hace que el artefacto ofuscado sea accesible para cualquiera que tenga el enlace.

## Ejemplo

```lua
local UI = Library:CreateWindow({
    Name = "Mi interfaz",
    Subtitle = "Herramientas",
    Size = Vector2.new(760, 500),
    ToggleKey = Enum.KeyCode.RightShift,
    MinimizeStyle = "Pill",
    ConfigManager = false,
})

local Inicio = UI:CreateTab("Inicio", "home", "Resumen")
local Acciones = Inicio:CreateGroupbox("Acciones", {Side = "Left"})

Acciones:CreateToggle({
    Name = "Activar opción",
    Default = false,
    Callback = function(enabled)
        print("Activada:", enabled)
    end,
})

Acciones:CreateButton({
    Name = "Ejecutar acción",
    Callback = function()
        UI:Notify({Title = "Noir", Content = "Acción ejecutada"})
    end,
})
```

Los tabs se crean con `Window:CreateTab(name, icon, subtitleOrOptions)`. Dentro de un tab se pueden crear subtabs con `Tab:CreateSubTab(name, icon, options)` y groupboxes con `Page:CreateGroupbox(name, options)`. `Side = "Left"` y `Side = "Right"` eligen columna; `Columns = 1` crea una sola columna.

## Componentes

Los componentes se crean en tabs, subtabs o groupboxes. Las opciones se pasan como tabla; `Callback` se llama cuando la persona interactúa con el control.

- `CreateToggle`, `CreateCheckbox`
- `CreateSlider`, `CreateRangeSlider`, `CreateNumberStepper`
- `CreateDropdown`, `CreateMultiDropdown`, `CreateRadioGroup`, `CreateSegmented`
- `CreateButton`, `CreateButtonGroup`
- `CreateInput`, `CreatePasswordInput`, `CreateTextarea`, `CreateKeybind`
- `CreateColorPicker`, `CreateLabel`, `CreateParagraph`, `CreateInfo`, `CreateSection`

Ejemplo de opciones con persistencia por flag:

```lua
Acciones:CreateSlider({
    Name = "Distancia",
    Min = 0,
    Max = 100,
    Default = 25,
    Suffix = " m",
    Flag = "combat.distance",
    Callback = function(value)
        print(value)
    end,
})
```

Los flags habilitan los métodos `Get`, `Set`, `Save`, `Load` y `Reset` disponibles para cada componente. Los métodos de interfaz concretos varían según el tipo de control.

## Ventanas y API

`CreateWindow` admite configuración de tamaño, tecla de mostrar/ocultar, tema, escala, sonidos, movimiento reducido, navegación por teclado, botón táctil y gestor de configuraciones. Métodos de biblioteca útiles:

- `Library:SetTheme(name)` y `Library:SetAccentColor(color)`
- `Library:SetLanguage("es" | "en" | "pt")`
- `Library:Dialog(config)` y `Library:Prompt(config)`
- `Window:Notify(config)`, `Window:Toggle()`, `Window:Minimize()`, `Window:Restore()`
- `Window:Destroy()`, `Window:SelectTab(name)`
- `Window:SaveConfig(name)`, `Window:LoadConfig(name)`, `Window:ResetConfig()`

El gestor de configuraciones es opcional (`ConfigManager = true`) y depende del soporte de archivos del executor. Al cargar el módulo no se abre la ventana de ejemplo; al ejecutar `New_Library_UI.lua` directamente, sí se muestra.

## Licencia Y Créditos

No se incluye una licencia propia de Noir por ahora. No redistribuyas ni publiques la fuente sin autorización de sus propietarios. Si se decide permitir reutilización o redistribución, hay que escoger y añadir una licencia explícita.

La salida ofuscada se generó localmente con Prometheus. Su licencia exige esta atribución para los productos que usan la herramienta:

> Based on Prometheus by Elias Oelschner, https://github.com/prometheus-lua/Prometheus

El soporte LuaU de Prometheus se declara básico y todavía incompleto. La salida actual pasa `luac -p`; la prueba funcional final debe hacerse en Roblox con el executor y la experiencia de destino.

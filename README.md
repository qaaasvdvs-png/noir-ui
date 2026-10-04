# Noir UI

Librería de interfaz para Roblox, escrita en LuaU y pensada para interfaces de juego con tabs, subtabs, groupboxes y controles reutilizables. La API sigue en evolución: este documento describe el uso disponible en la versión actual, no garantiza compatibilidad futura.

## Archivos

- `loader.lua`: artefacto remoto ofuscado publicado en GitHub para la carga desde Scripts.

## Requisitos

La biblioteca usa servicios estándar de Roblox, incluidos `Players`, `TweenService`, `UserInputService`, `GuiService` y `TextService`. La carga remota requiere que el executor permita descargar contenido HTTPS mediante `game:HttpGet` o `request` y compilarlo mediante una función `loadstring` compatible. La carga desde un archivo local requiere además `readfile`.

La gestión de configuraciones de Noir necesita las APIs de archivos del executor. Si no están disponibles, la ventana y los controles siguen funcionando, pero las configuraciones no se pueden guardar en disco.

## Carga Desde Slayers 2

`Slayers2_UI.lua` carga por defecto `loader.lua` desde el repositorio de Noir UI en GitHub. No es necesario tener los archivos de la biblioteca en el dispositivo para usar esta modalidad, pero sí se requiere conexión a GitHub y que el executor permita la descarga y compilación del código Lua.

La ventana de Slayers 2 incluye la pestaña `Config`; sus controles editables se registran para guardar y restaurar sus valores mediante el gestor de Noir.

Para usar otra ubicación, define `NoirLibraryUrl` antes de ejecutar el hub:

```lua
getgenv().NoirLibraryUrl = "https://raw.githubusercontent.com/usuario/repositorio/COMMIT/loader.lua"
```

Se recomienda apuntar a un commit o tag publicado en lugar de una rama mutable como `main`, para mantener una versión reproducible. `NoirLibraryPath` permite indicar una copia local si se prefiere no cargarla desde la red.

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

`Slayers2_UI.lua` también puede resolver la biblioteca desde las rutas habituales. Se puede indicar una ruta específica antes de ejecutarlo:

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

Los componentes con opción `Tooltip` muestran un icono de ayuda junto al título. En botones, labels, filas `Info` y encabezados `Section`, la ayuda aparece al pasar el cursor sin añadir un icono. Para solicitar explícitamente el icono en `CreateLabel`, `CreateInfo` o `CreateSection`, usa `TooltipIcon = true`.

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

El gestor de configuraciones está activo por defecto y depende del soporte de archivos del executor. Su pestaña `Config` se añade al final; las pestañas creadas después permanecen antes de ella. Desactívalo con `ConfigManager = false` si la aplicación proporciona su propio gestor. Al cargar el módulo no se abre la ventana de ejemplo; al ejecutar `New_Library_UI.lua` directamente, sí se muestra.

## Licencia Y Créditos

No se incluye una licencia propia de Noir por ahora. No redistribuyas ni publiques la fuente sin autorización de sus propietarios. Si se decide permitir reutilización o redistribución, hay que escoger y añadir una licencia explícita.

La salida ofuscada se generó localmente con Prometheus. Su licencia exige esta atribución para los productos que usan la herramienta:

> Based on Prometheus by Elias Oelschner, https://github.com/prometheus-lua/Prometheus

El soporte LuaU de Prometheus se declara básico y todavía incompleto. La salida actual pasa `luac -p`; la prueba funcional final debe hacerse en Roblox con el executor y la experiencia de destino.

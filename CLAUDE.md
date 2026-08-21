# Phosphor — contexto del proyecto

Editor visual de temas de terminal en un único `index.html` autocontenido. El objetivo de producto: llevar la interfaz de una terminal a algo más legible y agradable para consumir texto corrido (el caso de uso real de Lucas: leer output largo de agentes tipo Claude Code). Nació como artifact de Claude y se movió acá para tener repo propio y deploy público.

## Restricciones no negociables

1. **100% client-side**: cero llamadas de red, cero dependencias, cero build step. Se deploya como estático puro.
2. **Un solo archivo**: todo (CSS, JS, markup) vive en `index.html`. No fragmentar salvo pedido explícito.
3. **Español**: todo el copy de UI en español rioplatense (voseo). No traducir al inglés.
4. `localStorage` siempre envuelto en try/catch (puede estar bloqueado según host).

## Mapa del archivo (secciones del script, en orden)

- **state**: objeto único con todos los tokens — colores base, `ansi[16]`, tipografía, composición, render, cursor.
- **helpers**: `normHex`, `contrastRatio` (WCAG), `mixHex`, `ensureContrast` (simulación de contraste mínimo: empuja el color hacia blanco/negro hasta cruzar el ratio).
- **controls**: `colorRow()` genera cada fila (swatch + hex input) con `_sync()`; `CONTROL_IDS` + `syncControls()` para los inputs tipográficos; presets (7 temas conocidos); modo lectura (switch con snapshot/restore y marcado `.altered`).
- **color picker** (estilo Figma): conversiones RGB/HSL/HSB/CMYK, popover `#cpicker` único reposicionable, drag con `setPointerCapture`, modos con input por canal, copiar/pegar, swatches "en uso" + "recientes".
- **import**: `importTheme(text)` con auto-detección por estructura (no por extensión). Parsers: iTerm2 plist (DOMParser), JSON (distingue perfil iTerm / VS Code / Windows Terminal / Phosphor por claves), Alacritty TOML, Base16, Ghostty, kitty, Xresources, WezTerm lua, Warp/Orca YAML (`terminal_colors:`), YAML genérico legacy. Import parcial por diseño: solo pisa lo que el tema trae.
- **exporters**: objeto `EXPORTERS` — cada entrada tiene `file()`, `note` (instrucciones de instalación) y `gen()`. 10 formatos. Cada uno emite solo lo que ese motor soporta.
- **render**: `applyVars()` (variables CSS `--t-*` del preview + simulaciones), `buildPreview()` (sesión de terminal falsa + párrafo de prosa), `render()` orquesta todo.

## Decisiones de diseño ya tomadas (no re-litigar)

- **Preview = simulación honesta**: cada ajuste debe tener efecto visible. Los que el browser no puede replicar exacto se aproximan (contraste mínimo se simula con `ensureContrast`, engrosar trazos con text-stroke, opacidad con degradado detrás + alpha).
- **Fuentes**: el preview solo puede usar fuentes instaladas (no hay webfonts). Detección por medición canvas + badge "✓ instalada / ⚠ no detectada". El export usa el nombre igual.
- **`<fieldset>`/`<legend>` están prohibidos** para paneles: flotarles la legend rompe el layout en WebKit. Se usan `<section class="panel">` + `<h2 class="panel-title">`.
- **`hidden` no alcanza** si un selector del autor pone `display`: hay una salvaguarda `[hidden]{display:none!important}`.
- **Inputs alineados**: `margin-top:auto` en los controles dentro de labels flex-column.
- **Modales**: `<dialog>` nativo con `showModal()`; cierre por ✕ (arriba a la derecha), Esc y click en backdrop (`e.target === dialog`).
- **Descarga**: cascada `showSaveFilePicker` → `<a download>` adjuntado al DOM → aviso "usá copiar". (El sandbox de artifacts bloqueaba descargas; en deploy real ambos caminos funcionan.)
- **YAML export**: hex siempre entre comillas (`#` inicia comentario en YAML).
- **Los 16 ANSI son el contrato**: el theming de terminal pasa por la paleta ANSI; 256-color/truecolor no son tematizables.

## Estado de UI (progressive disclosure)

Header: preset · nombre del tema · ⇲ importar · ⇱ exportar. Columna izquierda: paneles Base / contraste / ANSI 0-7 / ANSI 8-15 / Carácter / Párrafo·composición / Render&ventana. Derecha: terminal preview sticky. Import y export viven en modales.

## Backlog / ideas no implementadas

- Estado compartible por URL (serializar el tema en el hash) — útil para la web pública.
- Export de light themes / par light+dark.
- Alpha por color (algunas terminales lo soportan en selección/cursor).
- Más presets (Rosé Pine, Everforest, Kanagawa).
- A11y pass: navegación por teclado dentro del picker (flechas en el área SV).

## Verificación

No hay tests. Chequeo mínimo tras editar: `node --check` sobre el bloque `<script>` extraído, y prueba manual en browser (picker, import de un `.itermcolors`, export de 2-3 formatos, modo lectura on/off).

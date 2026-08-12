# Phosphor

Editor visual de temas de terminal, pensado para hacer la terminal más legible y agradable para consumir texto corrido. Un solo archivo HTML, 100% client-side, cero dependencias.

**Demo del flujo**: elegís un preset (o importás un tema existente) → ajustás con la paleta tipográfica estilo InDesign → exportás al formato de tu terminal/IDE.

## Features

- **Paleta completa de terminal**: fondo, texto, cursor, selección + los 16 colores ANSI (normales y bright)
- **Color picker estilo Figma**: área saturación/brillo + slider de tono, modos HEX / RGB / HSL / HSB / CMYK con input por canal, copiar/pegar códigos, swatches "en este tema" y "recientes" (localStorage)
- **Paleta tipográfica** (lo compatible con terminales): fuente, tamaño, tracking, ligaduras, features OpenType, engrosado de trazos, interlineado (ratio + px), baseline shift, ancho de línea en columnas, márgenes
- **Render**: opacidad y blur de fondo, contraste mínimo (WCAG, simulado en vivo), bold-as-bright
- **Modo lectura**: switch con preset editorial (88 columnas, interlineado 1.7…) — restaura los valores previos al apagarse
- **Preview en vivo** con sesión de terminal simulada + párrafo de prosa que respeta la medida
- **Import** con auto-detección: iTerm2 (`.itermcolors` y perfil dinámico), Base16 YAML, VS Code, Windows Terminal, Alacritty TOML, kitty, Ghostty, Xresources, WezTerm lua, JSON/YAML propio
- **Export** a 10 formatos: VS Code/Cursor, JetBrains (`.icls`), Windows Terminal, Alacritty, kitty, Ghostty, WezTerm, iTerm2 (perfil dinámico), YAML (Orca), JSON genérico

## Uso local

Es un archivo estático — no hay build:

```bash
open index.html          # directo en el browser
# o con un server local:
python3 -m http.server 8080
```

## Deploy

Cualquier hosting estático sirve el archivo tal cual:

```bash
# Vercel
vercel --prod

# Netlify
netlify deploy --prod --dir .

# GitHub Pages: push a main y activar Pages sobre la raíz
```

No hay variables de entorno, ni backend, ni build step. `localStorage` se usa solo para recientes del picker y preferencias de UI (envuelto en try/catch — degrada sin romper si está bloqueado).

## Arquitectura

Todo vive en `index.html`: CSS (variables de tema del UI + tokens `--t-*` del preview), y un script con secciones comentadas: estado → helpers de color → controles → color picker → import → exporters → render. Ver `CLAUDE.md` para el mapa detallado y las decisiones de diseño.

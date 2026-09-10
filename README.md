# comfy · Generación de vídeo con ComfyUI

Proyecto para generar **vídeo** con [ComfyUI](https://github.com/comfyanonymous/ComfyUI)
controlado desde Claude Code a través del servidor MCP [`comfy-mcp`](https://pypi.org/project/comfy-mcp/).

`comfy-mcp` es solo el **conector**: habla con un ComfyUI que tú tienes corriendo
(local o remoto). No genera nada por sí mismo — necesita ComfyUI + los modelos de vídeo.

---

## 1. Requisitos

- **Python 3.10+** y `pip`
- Un **ComfyUI en marcha** con modelos de vídeo instalados (ver sección 4)
- **Claude Code** (CLI, app de escritorio o extensión)

## 2. Instalar comfy-mcp

```bash
pip install comfy-mcp
```

> Si `pip` se queja de no poder desinstalar un paquete del sistema (p. ej. `PyJWT`),
> usa un entorno virtual o añade `--ignore-installed`:
> ```bash
> pip install --ignore-installed comfy-mcp
> ```

## 3. Registrar el servidor MCP en Claude Code

Apuntando a tu ComfyUI mediante la variable `COMFY_URL`:

```bash
# Local (por defecto)
claude mcp add comfy-mcp -e COMFY_URL=http://127.0.0.1:8188 -- comfy-mcp

# Servidor remoto
claude mcp add comfy-mcp -e COMFY_URL=https://TU-SERVIDOR:8188 -- comfy-mcp
```

Comprobar que quedó conectado:

```bash
claude mcp list
# comfy-mcp: comfy-mcp  - √ Connected
```

## 4. Modelos de vídeo en ComfyUI

`comfy-mcp` ejecuta *workflows* de ComfyUI. Para vídeo, instala en tu ComfyUI
alguno de estos pipelines y sus modelos:

| Pipeline | Qué hace | Nodos / custom nodes |
|----------|----------|----------------------|
| **AnimateDiff** | Anima un modelo SD1.5/SDXL (texto→vídeo, img→vídeo) | `ComfyUI-AnimateDiff-Evolved` |
| **Stable Video Diffusion (SVD)** | Imagen → vídeo corto | Nativo en ComfyUI (`ImageOnlyCheckpointLoader`) |
| **WAN 2.x** | Texto/imagen → vídeo de mayor calidad | Nodos WAN + checkpoints WAN |
| **CogVideoX / LTX-Video** | Texto → vídeo | Custom nodes específicos |

Coloca los checkpoints/motion modules en las carpetas de `ComfyUI/models/...`
que pida cada pipeline, y usa el **ComfyUI Manager** para instalar los custom nodes.

## 5. Uso desde Claude Code

Una vez conectado, en una sesión de Claude Code puedes pedir cosas como:

- "Comprueba el estado de mi ComfyUI" → usa `server_info` / `system_stats`
- "Busca una plantilla de AnimateDiff" → `search_templates`
- "Ejecuta este workflow de vídeo y descárgame el resultado" → `run_workflow` → `fetch_outputs`

## 6. Workflow de ejemplo incluido

En `workflows/animatediff_txt2video.json` tienes un pipeline **texto → vídeo**
con AnimateDiff listo para probar (formato API, el que consume `comfy-mcp`).

**Necesitas en tu ComfyUI:**

| Qué | Dónde va | Nota |
|-----|----------|------|
| Custom node **ComfyUI-AnimateDiff-Evolved** | vía ComfyUI Manager | aporta el nodo `ADE_AnimateDiffLoaderGen1` |
| Custom node **ComfyUI-VideoHelperSuite** | vía ComfyUI Manager | aporta `VHS_VideoCombine` (salida .mp4) |
| Checkpoint SD1.5 (ej. `dreamshaper_8.safetensors`) | `ComfyUI/models/checkpoints/` | cámbialo en el nodo `1` si usas otro |
| Motion module `mm_sd_v15_v2.ckpt` | `ComfyUI/models/animatediff_models/` | modelo de movimiento de AnimateDiff |

**Ejecutarlo desde Claude Code** (con `comfy-mcp` conectado):

> "Ejecuta el workflow `workflows/animatediff_txt2video.json` y descárgame el vídeo"

O directamente con comfy-cli:

```bash
comfy run --workflow workflows/animatediff_txt2video.json --wait
```

**Qué puedes ajustar** en el JSON: el *prompt* positivo/negativo (nodos `2` y `3`),
`batch_size` = nº de fotogramas (nodo `4`), `seed`/`steps`/`cfg` (nodo `6`) y
`frame_rate` del vídeo (nodo `8`).

> Si algún modelo tiene otro nombre en tu equipo, edita el campo correspondiente
> del JSON o pídemelo y te lo dejo ajustado.

## 7. Estructura del repo

```
.
├── README.md
├── .mcp.json                       # registra comfy-mcp para este proyecto
├── .gitignore
└── workflows/                      # workflows .json de ComfyUI
    └── animatediff_txt2video.json  # ejemplo: texto -> vídeo con AnimateDiff
```

> **Nota:** edita `COMFY_URL` en `.mcp.json` con la URL real de tu ComfyUI
> (por defecto `http://127.0.0.1:8188`, para ComfyUI en tu PC local).

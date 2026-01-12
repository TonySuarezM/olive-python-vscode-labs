# Guía para principiantes — M0/00_setup_check.ipynb (Setup Check)

Este documento explica **qué comprueba** y **cómo interpretar** el notebook `00_setup_check.ipynb` (M0) del curso, pensado para gente que empieza con VS Code + Jupyter + entornos virtuales + Olive/ONNX Runtime.

---

## 1) ¿Para qué sirve este notebook?

Antes de optimizar modelos con Olive, necesitamos confirmar 3 cosas:

1) **Qué Python está ejecutando el notebook** (es decir, qué *kernel* estás usando).
2) Que los paquetes clave están instalados y **en qué versión**.
3) Que no hay **dependencias rotas** en el entorno (conflictos o requisitos sin instalar).

Esto evita el error típico de “lo instalé, pero el notebook usa otro Python”.

---

## 2) Conceptos mínimos (sin jerga)

### Kernel (Jupyter)
En Jupyter, un **kernel** es el proceso que ejecuta tu código. En Python, el kernel de referencia es **ipykernel**. citeturn4search0

En VS Code, cuando abres un `.ipynb`, debes **seleccionar el kernel** (arriba a la derecha) para elegir el Python/entorno con el que se ejecutarán las celdas. citeturn0search6turn0search3

### Entorno virtual (venv)
Un entorno virtual (`venv`) crea un Python “aislado” con sus propios paquetes. citeturn0search4turn0search0  
Cuando activas un venv en Windows, se usan scripts en `Scripts\` (por ejemplo `Activate.ps1` en PowerShell) que ajustan el `PATH` para que `python` apunte al Python del entorno. citeturn5search1turn5search3

> Nota: **no es obligatorio activar** el venv si llamas al Python del venv explícitamente. citeturn5search1

### pip y “python -m pip”
`pip` es el instalador de paquetes de Python. citeturn7search4  
La forma más fiable de usarlo es:

- `python -m pip ...`

porque así te aseguras de ejecutar pip con **ese** intérprete de Python. citeturn7search3turn0search11

---

## 3) Cómo ejecutar el notebook en VS Code (paso a paso)

1. Abre el archivo `.ipynb` en VS Code. citeturn0search6  
2. Pulsa **Select Kernel** (arriba a la derecha) y elige tu `.venv` (o el Python correcto). citeturn0search6turn0search3  
3. Ejecuta las celdas en orden (o “Run All”).

---

## 4) Qué hace cada celda y cómo interpretar el resultado

> El notebook tiene 1 celda Markdown y 4 celdas de código.

### Celda 1 — “¿Qué Python está ejecutando este notebook?”
Código (resumen):
- imprime `sys.executable` (ruta del python real)
- imprime `sys.version` (versión de Python)

**Cómo interpretarlo**
- Si ves una ruta tipo `...\.venv\Scripts\python.exe`, normalmente estás usando el venv del proyecto.
- Si ves un Python global (p. ej. `C:\Users\...\Python313\python.exe`), **probablemente NO** estás en tu venv → vuelve a **Select Kernel**. citeturn0search6turn0search3

**Tip para comprobar si estás en venv**
Python documenta una comprobación típica:
- `sys.prefix != sys.base_prefix` → estás dentro de un venv. citeturn0search4

---

### Celda 2 — “Versiones instaladas” (`pip show`)
Esta celda ejecuta `python -m pip show` para:
- `olive-ai`
- `onnxruntime`
- `onnx`
- `numpy`
- `ipykernel`

**Qué significa `pip show`**
`pip show` muestra información del paquete instalado (por ejemplo, versión y ubicación). citeturn0search1

**Por qué miramos `ipykernel`**
Porque el kernel de Python en Jupyter suele basarse en **ipykernel**. citeturn4search0

---

### Celda 3 — “¿Hay dependencias rotas?” (`pip check`)
Ejecuta:

- `python -m pip check`

**Qué significa**
`pip check` verifica que los paquetes instalados tienen dependencias compatibles (y avisa si falta algo o hay conflictos). citeturn0search2turn0search15

**Cómo interpretar la salida**
- Si dice “No broken requirements found.” → bien. citeturn0search15
- Si aparece algo como “X requires Y, which is not installed” → falta instalar `Y` en *ese* entorno.

---

### Celda 4 — “Imports básicos: Olive + ONNX Runtime”
Hace:
- `import olive`
- `import onnxruntime as ort`
- imprime versión y providers disponibles.

**Aclaración: ¿es `olive` o `olive-ai`?**
- El paquete se instala con `pip install olive-ai`. citeturn1search0turn1search4  
- En Python, la documentación y ejemplos usan imports bajo el **namespace `olive`** (por ejemplo `from olive.systems.local import LocalSystem`). citeturn2search5  
Por eso normalmente instalas `olive-ai` pero importas `olive`.

**Providers de ONNX Runtime**
`onnxruntime.get_available_providers()` devuelve la lista de **Execution Providers disponibles** en tu instalación (por ejemplo CPU, CUDA, DirectML, etc.). citeturn3search0

---

## 5) Problemas típicos (y cómo diagnosticarlos)

### A) “Lo instalé, pero el notebook dice que no existe”
Casi siempre es **kernel equivocado**.

1) mira `sys.executable` (celda 1)  
2) cambia el kernel con **Select Kernel** y elige tu `.venv`. citeturn0search6turn0search3  
3) repite `pip show` desde ese kernel (celda 2)

---

### B) PowerShell no deja activar `.venv` (Activate.ps1)
En Windows puede ser necesario habilitar la ejecución del script `Activate.ps1` con una política de ejecución adecuada (por ejemplo `RemoteSigned` para el usuario). citeturn5search3

---

### C) `pip` “instala” en un sitio pero el notebook no lo ve
Usa siempre `python -m pip ...` para garantizar que pip corre con el Python del kernel/entorno. citeturn7search3turn0search11

---

### D) `onnxruntime.get_available_providers()` no muestra GPU/NPU
Eso suele indicar que tu instalación de onnxruntime no incluye esos EPs o no están disponibles en ese entorno. La lista de “available providers” es precisamente el indicador oficial que expone la API. citeturn3search0turn3search1

---

## 6) ¿Qué sigue después de M0?

Si M0 está “verde” (sin errores):
- En **M1** veremos fundamentos (qué es ONNX/ORT/Olive) y correremos un modelo ONNX mínimo en ORT.
- En **M2+** empezaremos con workflows de Olive y generación de artefactos.

---

## Apéndice: comandos útiles (referenciados oficialmente)

- Ver un paquete: `python -m pip show <paquete>` citeturn0search1  
- Comprobar dependencias: `python -m pip check` citeturn0search2turn0search15  
- Instalar Olive: `pip install olive-ai` citeturn1search0  

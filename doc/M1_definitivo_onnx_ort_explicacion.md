# Guía para principiantes — M1_definitivo_onnx_ort.ipynb (Fundamentos ONNX + ONNX Runtime)

Este documento explica **qué hace** y **por qué** cada parte del notebook `M1_definitivo_onnx_ort.ipynb`, pensado para personas que empiezan con ONNX/ONNX Runtime (ORT) en **VS Code + Jupyter**.

---

## Objetivo del notebook (M1)

Al terminar, habrás practicado:

1. Crear un **modelo ONNX** muy pequeño (un grafo que hace `Y = X + 1`).
2. Guardarlo como archivo `.onnx` en `models/`.
3. Cargarlo y ejecutarlo con **ONNX Runtime** desde Python.
4. Inspeccionar lo básico del modelo (inputs/outputs/nodos/initializers, `ir_version`, `opset_import`).

---

## Prerrequisitos (antes de ejecutar)

### A) Estar en el kernel correcto (VS Code)
En VS Code, cuando abres un notebook `.ipynb`, debes seleccionar el **kernel** con el *kernel picker* (arriba a la derecha).  
Si ejecutas con el kernel equivocado, puedes ver errores tipo “ModuleNotFoundError” aunque “creas” haber instalado el paquete.  
Fuentes oficiales: VS Code Jupyter Notebooks + kernel management.

### B) Usar un entorno virtual (venv) recomendado
Un `venv` crea un entorno aislado con sus propios paquetes.  
Fuentes oficiales: documentación de `venv` de Python.

---

## Conceptos mínimos (sin “jerga”)

### 1) ¿Qué es un modelo ONNX?
En ONNX, un archivo `.onnx` contiene un **ModelProto**, que a su vez contiene un **GraphProto** (el “grafo” de cómputo) más metadatos.  
En el grafo hay:
- **Inputs** y **Outputs** (tensores de entrada y salida).
- **Nodes** (operadores: Add, MatMul, Conv, etc.).
- **Initializers** (constantes, típicamente pesos o tensores constantes).

Fuentes oficiales: ONNX API (ModelProto/GraphProto), ONNX “ONNX with Python”.

### 2) ¿Qué son IR y opset?
ONNX usa dos números de “versionado” importantes:
- **IR version**: versión del formato/representación (la “lengua” ONNX).  
- **Opset**: versión del conjunto de operadores (qué versión de `Add`, `MatMul`, etc., estás usando).

Si tu ONNX Runtime es antiguo y tu modelo tiene un `ir_version` u `opset` demasiado nuevos, puedes ver errores tipo:
“Unsupported model IR version …”.

Fuentes oficiales: ONNX API Reference (Versioning) y “ONNX with Python”.

### 3) ¿Qué es ONNX Runtime (ORT)?
ORT es el motor que carga y ejecuta un modelo ONNX. La clase principal en Python es `InferenceSession`.  
También puede usar distintos **Execution Providers (EPs)**, por ejemplo CPU, GPU, etc.

Fuentes oficiales: ONNX Runtime Python API summary y Execution Providers.

---

# Cómo leer el notebook (sección por sección)

## Sección 0 — Estructura de carpetas (`models/`, `outputs/`)
El notebook crea carpetas como `models/` y `outputs/` para mantener el proyecto ordenado:
- `models/` → guarda los `.onnx` que generas.
- `outputs/` → salidas de optimizaciones/artefactos (se usa más en M2+).

---

## Sección 1 — Chequeo de entorno (Python, paquetes, providers)

### Qué hace
Imprime:
- `sys.executable` y `sys.version` para saber qué Python ejecuta el notebook.
- `pip show <paquete>` para ver versión y ubicación instalada.
- `pip check` para detectar dependencias rotas.

### Por qué importa
Cuando trabajas con notebooks, lo más frecuente es:
- instalar paquetes en un Python
- pero ejecutar el notebook con otro

Ver la ruta de `sys.executable` ayuda a detectarlo muy rápido.

Fuentes oficiales:
- `venv` (cómo funcionan los entornos virtuales y cómo aislan paquetes).
- VS Code docs (selección de kernel).

---

## Sección 2 — (Opcional) Instalar dependencias si faltan

Si el chequeo anterior muestra que falta `onnx`, `onnxruntime` o `numpy`, el notebook usa:

- `python -m pip install ...`

Este patrón es útil porque garantiza que `pip` corre con *ese* Python (el del kernel).

Fuente oficial: documentación de `pip` (“pip install”) y `venv` (aislamiento de paquetes).

> Nota: si no quieres instalar desde el notebook, puedes hacerlo en el terminal integrado de VS Code, pero siempre asegurándote de estar en el mismo entorno/kernels.

---

## Sección 3 — Crear un modelo ONNX mínimo (Y = X + 1)

### Qué construye exactamente
Un grafo ONNX con:
- Input `X` (vector 1D float32, tamaño variable)
- Initializer `C = [1.0]` (tensor constante)
- Node `Add(X, C) -> Y`
- Output `Y`

### Por qué se usan “helpers”
El notebook usa utilidades `onnx.helper` y `onnx.numpy_helper` para construir:
- `make_tensor_value_info` → describe inputs/outputs (nombre, tipo, forma)
- `make_node` → crea un nodo (un operador, aquí `Add`)
- `make_graph` → reúne nodos + IO + initializers en un `GraphProto`
- `make_model` → empaqueta el grafo en un `ModelProto`
- `onnx.checker.check_model` → valida que el modelo cumple la especificación
- `onnx.save_model` → lo guarda como `.onnx`

Fuentes oficiales:
- ONNX “ONNX with Python” (ejemplos de creación/serialización y validación).
- ONNX API Overview (load/save).
- ONNX helper API (funciones para construir modelos).

### Por qué fijamos `opset` e `ir_version`
En tu curso ya viste un error típico: “Unsupported model IR version…”.

El notebook fija:
- `ir_version = 11`
- `opset = 11`

con la idea de mantener compatibilidad con runtimes antiguos.

La explicación de qué significan IR y opset es oficial (ONNX docs).  
El “por qué fijar a 11 en tu caso” es una decisión práctica del curso (para evitar incompatibilidades), no una regla universal.

Fuentes oficiales: ONNX API Reference (Versioning) y “ONNX with Python”.

---

## Sección 4 — Ejecutar inferencia con ONNX Runtime

### Qué hace el código
1) Imprime versión de ORT y providers disponibles:
- `ort.get_version_string()`
- `ort.get_available_providers()`

2) Crea una sesión:
- `InferenceSession(model_path, providers=["CPUExecutionProvider"])`

3) Obtiene nombres de IO:
- `sess.get_inputs()[0].name`
- `sess.get_outputs()[0].name`

4) Ejecuta:
- `sess.run([output_name], {input_name: x})`

### Por qué “forzar CPU”
Aunque ORT puede tener varios EPs, forzar `CPUExecutionProvider` hace el ejemplo:
- más reproducible
- más fácil de depurar

Fuentes oficiales:
- ONNX Runtime Python API summary (InferenceSession y run).
- Ejemplo “Load and predict…” (uso de get_inputs/get_outputs + run).
- Execution Providers (qué son y cómo ORT asigna subgrafos).

---

## Sección 5 — Inspección rápida del modelo ONNX

Esta parte carga el `.onnx` y muestra:
- `ir_version`
- `opset_import`
- nombre de grafo
- inputs/outputs
- initializers
- nodos (op_type + entradas/salidas)

Esto sirve para “ver” lo que guardaste y relacionarlo con el grafo que creaste en la sección 3.

Fuente oficial: ONNX Python API Overview (load/save) y ONNX API (ModelProto/GraphProto).

---

# Verificación (cómo saber que M1 está OK)

Deberías ver:

- Existe `models/add_const.onnx`
- ORT imprime providers disponibles (debe incluir `CPUExecutionProvider`)
- Si `x = [10, 20, 30]`, el resultado `y` debería ser `[11, 21, 31]`

---

# Errores comunes y diagnóstico

## 1) `ModuleNotFoundError: No module named ...`
Causas típicas:
- kernel equivocado
- paquetes instalados en otro Python

Checklist:
- confirma `sys.executable`
- en VS Code: **Select Kernel** y elige el `.venv`
- instala usando `python -m pip install ...` dentro del mismo kernel/entorno

Fuentes oficiales: VS Code (kernel picker / kernel management), Python `venv`.

## 2) `Unsupported model IR version ...`
Significa que el runtime (ORT) no soporta el `ir_version` del modelo.

Acciones:
- baja `ir_version` y/o `opset` (como hace el notebook para el ejemplo)
- o actualiza ONNX Runtime (si tu proyecto lo permite)

Fuentes oficiales: ONNX docs (IR/opset y versionado).

## 3) No aparecen providers GPU/NPU
`get_available_providers()` lista lo que **realmente** está disponible en tu instalación actual.

Fuentes oficiales: ORT Execution Providers + API.

---

# Siguiente paso del curso

- **M2**: primer workflow de Olive (config + `olive run`) y generación de artefactos.
- **M3**: cuantización y validación.

---

# Fuentes oficiales usadas (URLs exactas)

> (URLs incluidas en bloque de código para cumplir la restricción de “no poner URLs en texto normal”.)

```text
# VS Code
https://code.visualstudio.com/docs/datascience/jupyter-notebooks
https://code.visualstudio.com/docs/datascience/jupyter-kernel-management

# Python venv
https://docs.python.org/3/library/venv.html

# ONNX
https://onnx.ai/onnx/api/classes.html
https://onnx.ai/onnx/api/
https://onnx.ai/onnx/api/helper.html
https://onnx.ai/onnx/intro/python.html
https://onnx.ai/onnx/repo-docs/PythonAPIOverview.html

# ONNX Runtime
https://onnxruntime.ai/docs/api/python/api_summary.html
https://onnxruntime.ai/docs/api/python/auto_examples/plot_load_and_predict.html
https://onnxruntime.ai/docs/execution-providers/
```

# Guía para principiantes — M6_proyecto_final_pipeline_reporte.ipynb  
## Proyecto final: pipeline completo + métricas + mini‑reporte (Olive + ONNX Runtime)

Este documento acompaña al notebook **`M6_proyecto_final_pipeline_reporte.ipynb`** y explica, para no iniciados, qué hace cada bloque y cómo interpretar los resultados.

---

## 1) Objetivo del proyecto final (qué construyes)

El notebook arma un flujo **end‑to‑end** reproducible:

1) **Baseline ONNX** (modelo pequeño del curso, por ejemplo `models/add_const.onnx`).  
2) **Olive workflow** (por CLI) para generar un modelo derivado (ej. cuantización dinámica INT8).  
3) **Optimización offline** adicional con ONNX Runtime (guardar un `.onnx` optimizado a disco).  
4) Medición de:
   - **tamaño** del archivo ONNX
   - **latencia media** (micro‑benchmark)
   - **diferencia numérica** vs baseline  
5) Generación de un **reporte Markdown** en `outputs/m6_report.md`.

---

## 2) Antes de ejecutar: lo más importante (kernel + entorno)

### 2.1 Selecciona el kernel correcto en VS Code
En notebooks de VS Code debes elegir el **kernel** (Select Kernel). Si no, el notebook puede ejecutarse con otro Python y dar errores de imports o comportarse distinto. citeturn1search1turn1search2

### 2.2 Entorno virtual (venv) recomendado
El curso asume que trabajas en un `venv` para tener dependencias aisladas.  
La doc oficial de `venv` describe la creación y uso de entornos virtuales. citeturn2search2

Un chequeo típico para saber si estás en venv es:
- `sys.prefix != sys.base_prefix` citeturn2search2

---

## 3) Cómo se organiza el proyecto (carpetas)

El notebook usa (o crea) estas carpetas:

- `models/` → modelos ONNX baseline (entrada del pipeline).  
- `outputs/` → salidas de Olive, modelos finales, perfiles, reportes.

Esta estructura es práctica para que el pipeline sea reproducible y fácil de versionar.

---

# 4) Qué hace el notebook (bloque a bloque)

> Los números de “celda” pueden variar, pero el flujo es este.

## 4.1 Bloque A — Comprobación del entorno
Imprime:
- `sys.executable` (ruta del python real)
- versión de Python
- `pip show olive-ai` y `pip show onnxruntime`

**Por qué:** la forma fiable de verificar versiones y ubicación de paquetes es `pip show`. citeturn3search0turn3search5

---

## 4.2 Bloque B — Resolver rutas (`root`, `models_dir`, `outputs_dir`)
Si ejecutas el notebook desde `notebooks/`, a veces el directorio actual (CWD) no es la raíz del repo.  
El notebook “sube” un nivel si detecta que `models/` está en el padre.

---

## 4.3 Bloque C — Baseline: asegurar `models/add_const.onnx`
Si el modelo baseline no existe, el notebook lo genera con ONNX Python API:

- define inputs/outputs con `make_tensor_value_info`
- crea un initializer constante (por ejemplo `C=1.0`)
- crea un nodo `Add(X, C) -> Y`
- construye `GraphProto` y `ModelProto`
- valida con `onnx.checker.check_model`
- guarda el `.onnx`

Este patrón (crear/validar/guardar) está descrito en la guía “ONNX with Python” y en el overview de la Python API. citeturn4search0turn4search3

> Nota: en el curso fijamos `opset` e `ir_version` para evitar incompatibilidades con runtimes antiguos.  
La idea de versionado (IR/opset) es parte del estándar ONNX. citeturn4search2

---

## 4.4 Bloque D — Métricas baseline con ONNX Runtime
Se mide:

- **tamaño** del archivo con `Path(...).stat().st_size`
- **salida** con una inferencia (para tener una referencia)
- **latencia** con un bucle de `session.run(...)` (warmup + runs)

En ORT, la clase principal es `onnxruntime.InferenceSession` y la ejecución se hace con `session.run(...)`. citeturn5search0

El notebook también suele forzar `CPUExecutionProvider` para reproducibilidad.

---

## 4.5 Bloque E — Crear el run‑config de Olive (JSON)
Aquí está la parte “de pipeline”.

### (1) CLI de Olive
Olive documenta la ejecución por CLI con `olive run ...` y el uso de `python -m olive` cuando `olive` no está en PATH. citeturn6search0turn6search3

En el notebook se invoca típicamente así (para asegurar el Python del kernel):
- `sys.executable -m olive run --config <archivo.json>`

### (2) `input_model`
El `run_config` contiene `input_model` con:
- `type`
- `config` (por ejemplo `model_path`) citeturn6search1turn6search2

En algunas versiones, el identificador registrado para el handler ONNX es `ONNXModel` (clave de registro) — el curso ya vio que esto puede variar por versión; la forma correcta es la que acepte tu instalación. (Ver “Options” en fuentes).

### (3) `systems` y `LocalSystem`
El notebook usa `LocalSystem` con CPU. Olive documenta cómo configurar sistemas y el campo `accelerators`. citeturn7search0turn7search1

### (4) `passes`
En el proyecto final se usan passes “simples y didácticos”:

- `OnnxOpVersionConversion` para convertir a un `target_opset` si hace falta. citeturn8search0turn8search1  
- `OnnxDynamicQuantization` para cuantización dinámica INT8 (no necesita dataset de calibración). citeturn9search0turn9search1  

### (5) `engine`
Incluye:
- `cache_dir` para cachear el workflow
- `output_dir` para guardar resultados

---

## 4.6 Bloque F — Ejecutar Olive y capturar logs
Se ejecuta el comando y se imprime:
- `returncode`
- `stdout/stderr` (a veces recortados)

Esto es clave para depuración, porque el fallo suele estar en:
- tipo de `input_model`
- nombre/config de pass
- rutas

---

## 4.7 Bloque G — Encontrar el `.onnx` resultante en `outputs/`
Olive puede crear subcarpetas dentro de `output_dir`, así que el notebook busca `*.onnx` recursivamente y selecciona uno (por ejemplo el más reciente).

---

## 4.8 Bloque H — Métricas del modelo cuantizado (tamaño/latencia/validación)
Repite el mismo esquema del baseline:
- tamaño
- latencia
- salida y diferencia vs baseline (p. ej. `max abs diff`)

Aquí es normal ver pequeñas diferencias numéricas por cuantización.

---

## 4.9 Bloque I — Optimización OFFLINE con ONNX Runtime (guardar ONNX optimizado)
Este es el “extra” de M6: además de Olive, usamos ORT para “congelar” optimizaciones en otro archivo ONNX.

ORT documenta el patrón:
- crear `SessionOptions`
- establecer `graph_optimization_level`
- establecer `optimized_model_filepath`
- crear `InferenceSession` para que se genere el modelo optimizado citeturn10search0turn10search1

El notebook escribe algo como:
- `outputs/m6_final/<nombre>_ort_optimized.onnx`

Luego mide ese modelo final igual que antes.

---

## 4.10 Bloque J — Generar reporte Markdown
Se crea `outputs/m6_report.md` con:
- rutas de artefactos
- tabla comparativa de tamaño/latencia
- diferencia numérica

Esto es “packaging” de la evidencia: sirve para compartir resultados del pipeline.

---

## 4.11 (Opcional) Packaging con Olive (ZIP)
Olive soporta empaquetar outputs con `packaging_config` en `engine` y un tipo como `Zipfile`. citeturn11search0turn11search1

El notebook deja preparado un config “packaged” para que lo ejecutes si quieres.

---

# 5) Cómo saber que M6 está OK (checklist)

✅ Debes ver:

1) El baseline existe:
- `models/add_const.onnx`

2) El workflow de Olive termina sin error (`returncode = 0`). citeturn6search0  

3) Aparece al menos un `.onnx` en:
- `outputs/m6_workflow/...`

4) ORT puede cargar y ejecutar:
- baseline
- cuantizado (Olive)
- final (ORT offline optimized) citeturn5search0  

5) Se genera:
- `outputs/m6_report.md`

---

# 6) Errores comunes (y cómo diagnosticarlos)

## A) Olive se ejecuta con el Python global (no con tu venv)
Síntoma: el traceback apunta a `...Python...\Lib\site-packages...` fuera de `.venv`.

Solución:
- en notebook: usa `sys.executable -m olive ...`
- en VS Code: selecciona el kernel correcto. citeturn1search1turn1search2

## B) “Unknown model type …”
Es un error de compatibilidad de versión / registry key. Revisa la documentación “Options” de tu versión de Olive y ajusta `input_model.type`. citeturn6search1turn6search2

## C) No se genera `.onnx` en output_dir
Revisa el log de Olive:
- rutas (`model_path`, `output_dir`)
- nombres de passes
- permisos de escritura

## D) ORT no puede cargar el modelo final
Revisa:
- `ir_version` y `opset` del modelo (versionado ONNX) citeturn4search2  
- el provider usado (CPU vs otros) y dependencias del entorno

---

## 7) Qué sigue después del curso
Un siguiente paso natural es sustituir el “toy model” por un modelo real, y en vez de medir solo latencia/tamaño:
- medir throughput
- usar profiling de ORT y tuning de threads
- usar métricas de calidad reales (accuracy, etc.)

---

## Fuentes oficiales usadas (URLs exactas)

```text
# VS Code — Jupyter / kernel selection
https://code.visualstudio.com/docs/datascience/jupyter-notebooks
https://code.visualstudio.com/docs/datascience/jupyter-kernel-management

# Python — venv
https://docs.python.org/3/library/venv.html

# pip — show
https://pip.pypa.io/en/stable/cli/pip_show/

# ONNX — creación/validación con Python + versionado
https://onnx.ai/onnx/intro/python.html
https://onnx.ai/onnx/repo-docs/PythonAPIOverview.html
https://onnx.ai/onnx/api/versioning.html

# ONNX Runtime — ejecución + optimización offline (optimized_model_filepath)
https://onnxruntime.ai/docs/api/python/api_summary.html
https://onnxruntime.ai/docs/performance/model-optimizations/graph-optimizations.html

# Olive — CLI y run configs / systems / passes / packaging
https://microsoft.github.io/Olive/0.6.1/features/cli.html
https://microsoft.github.io/Olive/0.6.0/overview/quicktour.html
https://microsoft.github.io/Olive/0.5.2/overview/options.html
https://microsoft.github.io/Olive/0.5.2/tutorials/configure_systems.html
https://microsoft.github.io/Olive/0.6.0/tutorials/configure_systems.html
https://microsoft.github.io/Olive/0.6.0/api/passes.html
https://microsoft.github.io/Olive/0.7.0/features/passes/quant_onnx.html
https://microsoft.github.io/Olive/0.6.1/features/packaging_output_models.html
```

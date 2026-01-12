# Guía para principiantes — M3_cuantizacion_validacion.ipynb  
## Cuantización (INT8) y validación: “calidad vs rendimiento” con Olive + ONNX Runtime

Este documento acompaña al notebook **`M3_cuantizacion_validacion.ipynb`** y explica qué hace cada parte, cómo interpretar resultados y cómo diagnosticar fallos típicos.

---

## 1) ¿Qué vas a conseguir en M3?

En M3 construyes un mini‑experimento reproducible para comparar:

- **Baseline (FP32)**: modelo ONNX original.
- **Cuantizado (INT8)**: modelo ONNX tras aplicar cuantización con Olive.

Y mides:

- **Tamaño del archivo** (bytes).
- **Latencia media** (micro‑benchmark simple con `InferenceSession.run`).
- **Diferencia numérica** entre salidas (con tolerancia).

---

## 2) Prerrequisitos (para que no falle por “cosas de entorno”)

### A) Seleccionar el kernel correcto en VS Code
VS Code tiene un **kernel picker** (Select Kernel) para elegir qué entorno ejecuta el notebook. citeturn1search2

Si el notebook usa el Python equivocado, verás `ModuleNotFoundError` aunque hayas instalado paquetes.

### B) Verificar si estás en un `venv`
Python documenta que basta con comprobar:

- `sys.prefix != sys.base_prefix`

para saber si el intérprete está corriendo dentro de un entorno virtual. citeturn1search5

### C) Paquetes
- `olive-ai` (instalación por pip, importas `olive`) citeturn0search13  
- `onnxruntime` (ejecución y validación) citeturn0search3  
- `onnx`, `numpy` (creación del modelo de ejemplo y arrays)

---

## 3) Conceptos clave (en 2 minutos)

### 3.1 ¿Qué es cuantización “dinámica” vs “estática”?
En la doc de Olive (cuantización ONNX):

- **Dynamic Quantization** calcula parámetros de cuantización (scale/zero‑point) de activaciones **dinámicamente**, por lo que **no requiere dataset de calibración**. citeturn0search8turn0search0  
- Esos cálculos **incrementan el coste de inferencia**, pero normalmente logran **mejor precisión** que la cuantización estática. citeturn0search8turn0search0  

Este notebook usa precisamente ese modo para evitar preparar un dataset en el curso básico.

### 3.2 ¿Cómo se ejecuta un workflow de Olive?
Olive documenta que puedes correr un workflow con:

- `olive run --run-config <config.json>` (en algunas versiones aparece como `--config`) citeturn0search9turn0search13  

Y el **Quick Tour** muestra el patrón “run config JSON → `olive run ...`”. citeturn0search13

> En el notebook, lo habitual es invocarlo como `python -m olive ...` desde el **mismo Python del kernel** para evitar que se “cuele” un Python global.

### 3.3 ¿Qué hace ONNX Runtime (ORT) en este M3?
ORT carga el `.onnx` y ejecuta inferencia con:

- `onnxruntime.InferenceSession(...)`
- `session.run(...)` citeturn0search3  

---

# 4) Qué hace el notebook (celda a celda)

> Nota: los nombres exactos de celdas pueden variar, pero el flujo es este.

## Celda 1 — Chequeo del entorno
Imprime:
- `sys.executable` y `sys.version` (para ver el Python real).
- versión de ONNX Runtime y providers disponibles.
- `pip show olive-ai` para confirmar versión instalada.

**Por qué importa:** en M2/M3 es común instalar `olive-ai` en un entorno y ejecutar en otro.

---

## Celda 2 — Construir un modelo ONNX con pesos (Linear)
Se crea un modelo pequeño con **pesos** para que la cuantización tenga sentido:
- `MatMul(X, W)` seguido de `Add(..., b)`.

El modelo se construye usando utilidades de ONNX (`onnx.helper`) como:
- `make_tensor_value_info`
- `make_node`
- `make_graph`
- `make_model` citeturn1search4turn1search0turn1search1  

Y se valida con `onnx.checker.check_model(...)` antes de guardarlo. citeturn1search0

---

## Celda 3 — Baseline: medir tamaño y latencia (micro‑benchmark)
Aquí se hace un benchmark simple:

1) Carga del modelo con `InferenceSession`.
2) Warmup (unas cuantas ejecuciones).
3) Loop de `session.run(...)` para calcular tiempo medio.

La API `InferenceSession` y `run()` está documentada en la guía de la API de ORT. citeturn0search3

**Importante:** esta latencia es orientativa (sirve para comparar “antes vs después” en la misma máquina).

---

## Celda 4 — Crear el JSON de run‑config para Olive
Esta es la parte “estructural” del workflow:

### 4.1 `input_model`
Defines qué modelo entra al pipeline (ruta del `.onnx`).

### 4.2 `systems` → `LocalSystem` + `accelerators`
El notebook define un sistema local para CPU. Olive documenta cómo configurar un `LocalSystem` y el campo `accelerators`. citeturn0search14turn0search10turn0search6  

### 4.3 `passes` → `OnnxDynamicQuantization`
La cuantización dinámica está documentada como pass de ONNX quantization. citeturn0search0turn0search8turn0search4  

---

## Celda 5 — Ejecutar Olive (CLI)
Se ejecuta el workflow con `olive run ...` (invocado normalmente vía `python -m olive` desde el kernel).  
La CLI `olive run` aparece documentada en la página de Command Line Tools. citeturn0search9

Si falla, el notebook suele imprimir `stdout/stderr` para que puedas copiar el error.

---

## Celda 6 — Encontrar el `.onnx` resultante en `outputs/`
Olive puede escribir el resultado en subcarpetas dentro de `output_dir`, así que el notebook busca recursivamente `*.onnx`.

El objetivo es obtener una ruta tipo `outputs/m3_linear_int8/.../model.onnx`.

---

## Celda 7 — Validación: FP32 vs INT8
Se ejecuta inferencia con ORT para ambos modelos y se compara:

- `np.allclose(y_fp32, y_int8, rtol=..., atol=...)`

**Qué esperar:**
- Es normal que haya pequeñas diferencias numéricas tras cuantizar.
- Ajustar tolerancias es parte del “quality vs performance”.

El notebook también compara:
- tamaño
- latencia

---

# 5) Verificación: ¿cómo sé que M3 está OK?

✅ Checklist:

1) El `olive run ...` termina sin error. citeturn0search9turn0search13  
2) En `outputs/m3_linear_int8/` aparece al menos un `*.onnx`.  
3) ORT puede cargar el modelo cuantizado con `InferenceSession`. citeturn0search3  
4) La comparación numérica da diferencias pequeñas (o `allclose=True` con una tolerancia razonable).

---

# 6) Errores comunes y diagnóstico (rápido)

## A) `ModuleNotFoundError` (olive/onnxruntime/onnx)
Casi siempre: kernel equivocado.

- Revisa `sys.executable`.
- En VS Code: usa **Select Kernel** para elegir el `.venv`. citeturn1search2  

## B) Olive corre con el Python global
Síntoma: el traceback apunta a `...Python313\Lib\site-packages...` en vez de tu `.venv`.

Solución: ejecuta Olive con el Python del kernel (en notebook: `sys.executable -m olive ...`).

## C) Error “no requiere dataset de calibración”
Este notebook está diseñado para que NO necesites dataset usando cuantización dinámica (documentado por Olive). citeturn0search8turn0search0  
Si tu config intenta cuantización estática, entonces sí puede requerir calibración/datos.

## D) No mejora la latencia (o incluso empeora)
Olive advierte que el modo dinámico incrementa coste (cálculo de parámetros en runtime). citeturn0search8turn0search0  
En modelos pequeños, el overhead puede “comerse” la ganancia. Aun así, el tamaño suele bajar y el experimento sirve para aprender el flujo.

---

# 7) Siguiente paso (M4)
En M4 te centras en medición y “shape” de despliegue:
- profiling
- optimización offline de ORT (serializar modelo optimizado)
- empaquetado de artefactos

---

## Fuentes oficiales usadas (URLs exactas)

```text
# VS Code
https://code.visualstudio.com/docs/datascience/jupyter-kernel-management

# Python venv
https://docs.python.org/3/library/venv.html

# ONNX (creación/validación de modelos)
https://onnx.ai/onnx/intro/python.html
https://onnx.ai/onnx/repo-docs/PythonAPIOverview.html
https://onnx.ai/onnx/api/helper.html

# ONNX Runtime (ejecución)
https://onnxruntime.ai/docs/api/python/api_summary.html

# Olive (CLI, systems y cuantización ONNX)
https://microsoft.github.io/Olive/0.6.1/features/cli.html
https://microsoft.github.io/Olive/0.6.0/overview/quicktour.html
https://microsoft.github.io/Olive/features/quantization.html
https://microsoft.github.io/Olive/0.7.0/features/passes/quant_onnx.html
https://microsoft.github.io/Olive/0.5.2/tutorials/configure_systems.html
https://microsoft.github.io/Olive/0.6.0/tutorials/configure_systems.html
```

# Guía para principiantes — M4_medicion_optim_offline_packaging.ipynb  
## Medición, optimización offline y packaging (ORT + Olive)

Este documento explica el notebook **`M4_medicion_optim_offline_packaging.ipynb`** para gente que empieza. La idea es que entiendas **qué mide**, **qué optimiza**, y **qué artefactos genera**.

---

## 1) ¿Qué persigue M4?

En M0–M3 ya sabes:
- ejecutar un modelo ONNX con **ONNX Runtime (ORT)**
- cuantizar con **Olive** (INT8) y validar

En **M4** damos un paso más “de ingeniería”:

1. **Medir** (tamaño y latencia aproximada) de un modelo.
2. Usar **optimizaciones de grafo** de ORT y, además, guardarlas como **modelo optimizado offline** (otro `.onnx`).
3. Generar un **perfil (profiling)** de ORT para identificar dónde se va el tiempo (archivo JSON).
4. (Opcional) Empaquetar artefactos como un **ZIP** con Olive (`packaging_config`).

---

## 2) Requisitos previos (muy importante para no frustrarse)

### A) Kernel correcto en VS Code
En VS Code, el notebook se ejecuta con un **kernel** seleccionable (Select Kernel). Si el kernel apunta a otro Python, verás inconsistencias (paquetes “instalados” que luego no se encuentran).  
Recomendación práctica del curso: siempre imprime `sys.executable` al inicio y confirma que apunta a tu `.venv`.

### B) Paquetes esperados
- `onnxruntime`
- `onnx`, `numpy` (si necesitas crear o manipular inputs/modelos)
- `olive-ai` (solo para la parte de packaging opcional)

---

## 3) Qué hace el notebook (sección por sección)

### 3.1 Celda “entorno”: versiones y providers
Se imprime:
- versión de ORT
- providers disponibles (por ejemplo CPU)

Esto te sirve para interpretar resultados: **siempre** compara mediciones en la misma máquina, con el mismo provider.

---

### 3.2 Selección/creación del modelo de trabajo
El notebook intenta usar un modelo existente, por ejemplo:
- `models/linear_fp32.onnx` (del M3)
- o un modelo INT8 ya generado (si existe)

Si no existe, crea un modelo lineal FP32 mínimo (`MatMul` + `Add`) y lo guarda.  
La idea es tener un modelo con “algo” de compute para que la medición no sea trivial.

---

## 4) Medición: tamaño y latencia aproximada (micro‑benchmark)

### 4.1 Tamaño del modelo
Se toma el tamaño del archivo `.onnx` (bytes).  
Esto sirve para comparar: **FP32 vs INT8**, o **antes vs después** de optimización offline.

### 4.2 Latencia promedio
El notebook hace un benchmark simple:
1) crea `InferenceSession`
2) hace warmup (varias ejecuciones para estabilizar)
3) ejecuta un bucle con `session.run(...)` y calcula el tiempo medio

**Por qué es “aproximado”**  
Porque en Python influye el “overhead” del bucle, cachés, frecuencia del CPU, etc. Aun así, es útil para comparar cambios en el mismo entorno.

---

## 5) Optimización de grafo en ORT y “offline mode” (guardar el ONNX optimizado)

### 5.1 GraphOptimizationLevel
ORT define niveles de optimización:
- desactivado
- básico
- extendido
- y “todo” (incluyendo layout optimizations)

El notebook usa normalmente **EXTENDED** como punto equilibrado para ejemplos.

### 5.2 Offline mode (serializar el modelo optimizado)
ORT permite *guardar* a disco el resultado del grafo optimizado. El patrón es:

1) Crear `SessionOptions`
2) Configurar:
   - `sess_options.graph_optimization_level = ...`
   - `sess_options.optimized_model_filepath = "ruta/al/modelo_opt.onnx"`
3) Crear `InferenceSession(model_path, sess_options)`  
   → al crear la sesión, ORT optimiza y escribe el archivo optimizado.

**Consejo práctico:**  
Luego, cuando cargas el ONNX ya optimizado, puedes desactivar optimizaciones para reducir trabajo de inicialización (para la demo). El notebook muestra este patrón.

> Nota importante de ORT: si habilitas optimizaciones de layout y generas un modelo offline, ese modelo puede requerir hardware compatible con el entorno donde lo guardaste (por ejemplo, instrucciones específicas del CPU). Mantener EXTENDED ayuda a evitar sorpresas en ejemplos.

---

## 6) Profiling en ORT: generar un JSON y resumirlo

ORT permite activar profiling desde código:
- en `SessionOptions`, pones `enable_profiling = True`
- al final, llamas a `session.end_profiling()` para finalizar y obtener el nombre del archivo generado

El notebook:
1) activa profiling
2) ejecuta varias inferencias
3) termina profiling y carga el JSON
4) hace un resumen simple (por nombre de evento, sumando duraciones)

**Qué aporta**
- te da una idea rápida de qué operaciones consumen más tiempo
- es una base para iterar (cambiar optimización, threads, cuantización, etc.)

---

## 7) (Opcional) Packaging con Olive: generar un ZIP de artefactos

Olive soporta empaquetar artefactos cuando añades `packaging_config` en la sección `engine` del run‑config.  
En el notebook se usa la variante `Zipfile`, que produce un `.zip` con un nombre prefijo (por ejemplo `OutputModels.zip` o el que definas).

Este paso es opcional porque:
- requiere ejecutar Olive por CLI
- y tiene más sentido cuando ya estás exportando modelos “reales” (no solo toy models)

---

## 8) Verificación: ¿cómo sé que M4 está OK?

✅ Checklist:

1) La celda de benchmark imprime:
   - tamaño del FP32 (y del INT8 si existe)
   - latencia media (ms)

2) Existe un archivo como:
- `outputs/m4_optimized/linear_optimized.onnx`

3) Existe un archivo de profiling (JSON) como:
- `outputs/m4_profiles/ort_profile_*.json`  
y el notebook imprime un “Top eventos” con duraciones.

4) (Opcional) si ejecutas packaging:
- aparece un `.zip` bajo `outputs/m4_olive_package/`

---

## 9) Problemas comunes y diagnóstico

### A) “No cambia la latencia” (o empeora)
No pasa nada: con modelos muy pequeños el overhead domina. En M4 lo importante es:
- aprender el flujo de medición
- aprender a generar offline model
- aprender a producir profiling

### B) El modelo optimizado offline no se genera
Revisa que:
- la carpeta `outputs/m4_optimized/` existe
- `sess_options.optimized_model_filepath` apunta a una ruta válida
- el proceso tiene permisos de escritura

### C) No aparece el perfil JSON
Asegúrate de que:
- `enable_profiling=True`
- llamas a `end_profiling()` (si no, el archivo no se “cierra”/finaliza)

### D) Packaging falla por Olive
Si el workflow de Olive falla, revisa:
- que estás ejecutando Olive con el Python del kernel/venv (en notebook: `sys.executable -m olive ...`)
- que tu versión de Olive soporte el tipo de `input_model` y el `packaging_config` que pusiste

---

## 10) Siguiente paso
Tras M4, normalmente pasas a:
- **M5**: trabajar con el repo oficial de Olive
- **M6**: proyecto final (pipeline completo + métricas + reporte)

---

## Fuentes oficiales usadas (URLs exactas)

```text
# ONNX Runtime — API Python (InferenceSession, end_profiling, etc.)
https://onnxruntime.ai/docs/api/python/api_summary.html

# ONNX Runtime — Graph optimizations (GraphOptimizationLevel + offline mode + optimized_model_filepath)
https://onnxruntime.ai/docs/performance/model-optimizations/graph-optimizations.html

# ONNX Runtime — Profiling example (Python)
https://onnxruntime.ai/docs/api/python/auto_examples/plot_profiling.html

# ONNX Runtime — Profiling tools (enable_profiling, y contexto de profiling)
https://onnxruntime.ai/docs/performance/tune-performance/profiling-tools.html

# Olive — Packaging output models (packaging_config / Zipfile)
https://microsoft.github.io/Olive/0.6.1/features/packaging_output_models.html
```

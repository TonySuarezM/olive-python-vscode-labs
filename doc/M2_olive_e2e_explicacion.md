# Guía para principiantes — M2_olive_e2e.ipynb (Primer pipeline Olive end‑to‑end)

Este documento explica el notebook **`M2_olive_e2e.ipynb`**: qué hace, por qué lo hace, cómo interpretar la salida y cómo diagnosticar errores típicos.

---

## Objetivo (M2)

En M2 damos el salto de “ejecutar un ONNX” (M1) a “**optimizar un ONNX con Olive**” de forma end‑to‑end:

1) Partimos de un modelo ONNX baseline (por ejemplo `models/add_const.onnx`).  
2) Describimos un **workflow** de Olive en un JSON de configuración. Olive está diseñado para componer pipelines de optimización como una serie de *passes*. citeturn0search10turn0search11  
3) Ejecutamos el workflow con la CLI (`olive run --config ...`) o, si `olive` no está en PATH, con `python -m olive ...`. citeturn1search15turn1search5  
4) Buscamos el `.onnx` optimizado dentro de `outputs/` y lo validamos con ONNX Runtime.

---

## Prerrequisitos

### 1) VS Code + Jupyter: kernel correcto
En VS Code, cuando abres un `.ipynb`, debes **seleccionar un kernel** (arriba a la derecha). Si el kernel apunta a otro Python, verás errores de imports aunque “hayas instalado” paquetes. citeturn1search1turn1search2

### 2) Estar en un `venv` (muy recomendable)
Para saber si realmente estás dentro de un entorno virtual, Python indica que basta con comprobar:  
`sys.prefix != sys.base_prefix`. citeturn2search2

### 3) Paquetes
- `olive-ai` (se instala como paquete) citeturn1search9  
- `onnxruntime` (para validar el modelo) citeturn1search0  
- `onnx`, `numpy` (para manejar/crear inputs y modelos)

---

## Explicación breve (qué es lo que hace Olive en este M2)

Olive permite definir **workflows** por JSON: describes el modelo de entrada, el sistema/target (por ejemplo CPU) y una lista de *passes* (transformaciones/optimizaciones) que se aplican en cadena. citeturn0search10turn0search11

En este notebook M2 se usa típicamente:

- **OnnxOpVersionConversion**: convierte el modelo ONNX a un *target opset* (por ejemplo 11) para compatibilidad. citeturn0search7turn3search2  
- **OnnxModelOptimizer**: optimiza el ONNX (por ejemplo, fusiones de nodos). citeturn0search2turn3search2  

> Nota: además de optimización, Olive también lista passes de cuantización (`OnnxDynamicQuantization`, `OnnxStaticQuantization`, `OnnxQuantization`), pero eso lo explotamos en M3. citeturn0search4turn3search2

---

# Cómo leer el notebook (paso a paso)

## 1) Chequeo de entorno (Python y dependencias)
El notebook suele imprimir:
- `sys.executable` / `sys.version` para confirmar el Python del kernel.
- `python -m pip show ...` para ver versión y ubicación de paquetes (útil para detectar si están instalados en otro entorno). citeturn2search1turn2search16  
- `python -m pip check` para ver conflictos de dependencias. citeturn2search5turn2search10

---

## 2) Preparar el modelo de entrada (baseline)
El workflow necesita un modelo ONNX de entrada. En el curso, suele ser `models/add_const.onnx` (el del M1).

Si no existe, el notebook puede:
- avisar
- o pedirte volver a M1 para generarlo

---

## 3) Generar el JSON de configuración de Olive
Aquí está el “corazón” de M2.

### 3.1 `input_model`
En la configuración de Olive, `input_model` es un diccionario con:
- `type`: tipo de modelo de entrada
- `config`: configuración (por ejemplo `model_path`) citeturn0search6turn3search0  

**Importante por versiones (ONNXModel vs ONNXModelHandler)**  
Hay documentación de Olive donde el tipo ONNX aparece como `ONNXModel`. citeturn0search6  
Y en documentación más reciente aparece como `ONNXModelHandler`. citeturn3search0turn3search1  

Si te sale un error del estilo *Unknown model type ...*, revisa la página **Olive Options** de la versión que estés usando y ajusta `input_model.type` a lo que acepte tu instalación. citeturn0search6turn3search0

### 3.2 `systems` (LocalSystem + accelerators)
M2 define un sistema local (CPU) para ejecutar/validar:

- `LocalSystem` con `accelerators`  
- Puedes especificar `device` y/o `execution_providers` (pero Olive documenta reglas de combinación cuando defines accelerators). citeturn2search3turn2search9  

Ejemplo conceptual:
- device = `"cpu"`
- execution_providers = `["CPUExecutionProvider"]`

---

### 3.3 `passes`
Un pass toma un modelo de entrada y produce un modelo de salida (que puede alimentar al siguiente pass). citeturn0search10turn0search11  

En M2, los passes típicos son:

- `OnnxOpVersionConversion` con `target_opset` (p. ej. 11). citeturn0search7turn3search2  
- `OnnxModelOptimizer` (fusiones y optimizaciones). citeturn0search2turn3search2  

---

### 3.4 `engine`
El engine define:
- `host` y `target` (por ejemplo el mismo `local_system`)
- `cache_dir` (cache del workflow)
- `output_dir` (salidas del workflow)
- flags de logging / evaluación

---

## 4) Ejecutar el workflow (CLI)
El notebook suele ejecutar Olive con:

- `olive run --config <archivo>.json` citeturn1search13turn1search15  
- o `python -m olive run --config ...` si `olive` no está en PATH citeturn1search15turn1search5  

Olive también documenta un modo `--setup` para ver paquetes extra requeridos por un workflow. citeturn1search5turn1search3

---

## 5) Localizar el modelo ONNX optimizado en `outputs/`
Olive puede crear subcarpetas dentro de `output_dir`, así que el notebook suele buscar recursivamente `*.onnx` (por ejemplo con `Path(...).rglob("*.onnx")`) y escoger uno.

---

## 6) Validación con ONNX Runtime (ORT)
Finalmente se carga el ONNX resultante y se ejecuta una inferencia (como en M1):

- `InferenceSession(...)` para cargar el modelo. citeturn1search0  
- `session.run([...], inputs)` para ejecutar. citeturn1search0  
- Muchos ejemplos usan `get_inputs()` / `get_outputs()` para recuperar nombres y construir el dict de inputs. citeturn1search6turn1search14  

Además, ORT soporta múltiples **Execution Providers** (EPs) para ejecutar en distintos hardwares; CPU suele ser el baseline más reproducible. citeturn2search0

---

# Verificación (cómo saber que M2 funcionó)

✅ Checklist:
1) El comando `olive run ...` termina sin error. citeturn1search15  
2) En `outputs/<tu_workflow>/` aparece al menos un `*.onnx`.  
3) ORT puede cargar ese `.onnx` con `InferenceSession`. citeturn1search0  
4) La inferencia devuelve valores razonables (en el ejemplo `add_const`, el output debería ser input+1).

---

# Errores comunes y diagnóstico (los 5 más típicos)

## 1) “Olive / onnxruntime no se encuentra” (ModuleNotFoundError)
Casi siempre es kernel equivocado:
- Revisa `sys.executable`
- Cambia el kernel en VS Code. citeturn1search1turn1search2  

## 2) “Unknown model type …” en `input_model.type`
Por diferencias entre versiones, el string aceptado puede variar:
- en doc antigua aparece `ONNXModel` citeturn0search6  
- en doc más reciente aparecen `...ModelHandler` citeturn3search0turn3search1  

Solución: ajusta el `type` según la documentación de **tu** versión (Olive Options).

## 3) No se genera ningún `.onnx` en `output_dir`
- Revisa los logs de Olive y confirma que el `model_path` existe.  
- Confirma que el `output_dir` del engine apunta a una carpeta donde el proceso tiene permisos de escritura.

## 4) Error al cargar el `.onnx` en ORT (IR/opset)
Si ORT falla al cargar, puede ser un problema de versiones (IR/opset). En M2 usamos `OnnxOpVersionConversion` para llevar el modelo a un opset objetivo. citeturn0search7turn3search2  

## 5) Provider / acelerador no disponible
- En ORT puedes consultar qué EPs están disponibles y usar EPs específicos al crear `InferenceSession`. citeturn2search0turn1search0  
- En Olive, para `LocalSystem` las reglas de `accelerators` indican cómo especificar `device` / `execution_providers`. citeturn2search3turn2search9  

---

## Siguiente paso
- **M3**: cuantización (INT8) y validación de calidad vs rendimiento.
- **M4**: medición más “seria” (profiling, optimización offline y empaquetado).

---

## Fuentes oficiales usadas (URLs exactas)

```text
# VS Code
https://code.visualstudio.com/docs/datascience/jupyter-notebooks
https://code.visualstudio.com/docs/datascience/jupyter-kernel-management

# Python venv
https://docs.python.org/3/library/venv.html

# pip
https://pip.pypa.io/en/stable/cli/pip_show/
https://pip.pypa.io/en/stable/cli/pip_check/
https://pip.pypa.io/en/latest/user_guide/

# Olive
https://microsoft.github.io/Olive/why-olive.html
https://microsoft.github.io/Olive/0.6.1/features/cli.html
https://microsoft.github.io/Olive/0.6.0/overview/quicktour.html
https://microsoft.github.io/Olive/0.3.3/overview/options.html
https://microsoft.github.io/Olive/0.5.2/overview/options.html
https://microsoft.github.io/Olive/0.5.2/tutorials/configure_systems.html
https://microsoft.github.io/Olive/0.5.1/api/passes.html
https://microsoft.github.io/Olive/0.6.0/api/passes.html
https://microsoft.github.io/Olive/0.5.0/features/model_transformations_and_optimizations.html

# ONNX Runtime
https://onnxruntime.ai/docs/api/python/api_summary.html
https://onnxruntime.ai/docs/api/python/auto_examples/plot_load_and_predict.html
https://onnxruntime.ai/docs/execution-providers/
```

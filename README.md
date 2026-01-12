# > **Este repositorio es un proyecto personal, abierto y sin pretensiones. Toda la información aquí contenida corresponde a mis apuntes y prácticas personales, no constituye documentación oficial y se publica sin garantía ni responsabilidad.**
# Curso: Olive + Python en VS Code (Jupyter)

**Repositorio:** `olive-python-vscode-labs`

Este repositorio contiene TODO el material del curso (módulos **M0–M6**): notebooks, scripts auxiliares, modelos/datos de ejemplo y artefactos generados.

## Estructura del proyecto

- `notebooks/` → laboratorios en `.ipynb` (paso a paso)  
- `models/` → modelos (p.ej. ONNX) usados en el curso  
- `data/` → datos de ejemplo para evaluar/validar  
- `outputs/` → salidas generadas (modelos optimizados, métricas, logs, etc.)  
- `requirements.txt` → dependencias del entorno Python del curso  

## Requisitos

- Visual Studio Code con soporte para Jupyter Notebooks (extensión Jupyter).
- Python (recomendado usar entornos virtuales con `venv`).
- `pip` para instalar dependencias.
- Git (si vas a versionar y publicar cambios desde VS Code).

## Inicio rápido (entorno reproducible)

1) Crear un entorno virtual en la raíz del repositorio:

```bash
python -m venv .venv
```

> Activa el entorno según tu sistema operativo; ver documentación oficial de `venv`.

2) Instalar dependencias del curso desde `requirements.txt`:

```bash
python -m pip install -r requirements.txt
```

3) Abrir el repositorio en VS Code y trabajar con los notebooks:

- Abre la carpeta del repo en VS Code.
- Abre un archivo en `notebooks/` (por ejemplo, `notebooks/00_setup.ipynb`).
- Selecciona el kernel de Python correspondiente a tu entorno (`.venv`) desde VS Code.

## Regla del curso: versiones y verificación

Antes de ejecutar comandos/pipelines de Olive, verifica versión de Python y el paquete instalado:

```bash
python --version
python -m pip show olive-ai
```

Esto ayuda a evitar diferencias de comportamiento entre versiones.

## Temario (ruta guiada)

- **M0:** Setup VS Code + Jupyter + `venv` + `pip` + `olive-ai`  
  [Apuntes: Introducción y setup](doc/M0_setup_check_introduccion.md)
- **M1:** Fundamentos (ONNX, ONNX Runtime, qué hace Olive)  
  [Apuntes: Explicación ONNX y ORT](doc/M1_definitivo_onnx_ort_explicacion.md)
- **M2:** Primer pipeline Olive end-to-end (artefactos, outputs, validación)  
  [Apuntes: Pipeline Olive E2E](doc/M2_olive_e2e_explicacion.md)
- **M3:** Cuantización y validación (calidad vs rendimiento)  
  [Apuntes: Cuantización y validación](doc/M3_cuantizacion_validacion_explicacion.md)
- **M4:** Optimización/packaging y medición (latencia/tamaño)  
  [Apuntes: Medición, optimización y packaging](doc/M4_medicion_optim_offline_packaging_explicacion.md)
- **M5:** Trabajo con repos oficial (microsoft/Olive) y ejemplos oficiales  
  [Apuntes: Repo oficial y editable install](doc/M5_repo_oficial_editable_install_explicacion.md)
- **M6:** Proyecto final (pipeline completo con métricas y reporte)  
  [Apuntes: Proyecto final y reporte](doc/M6_proyecto_final_pipeline_reporte_explicacion.md)

## Referencias oficiales (URLs permitidas en el curso)

### VS Code (docs)
- https://code.visualstudio.com/docs  
- https://code.visualstudio.com/docs/datascience/jupyter-notebooks  
- https://code.visualstudio.com/docs/datascience/jupyter-kernel-management  

### Python + venv
- https://docs.python.org/3/library/venv.html  

### pip
- https://pip.pypa.io/  

### Microsoft Olive
- https://microsoft.github.io/Olive/  
- https://github.com/microsoft/Olive  

### ONNX / ONNX Runtime
- https://onnx.ai/  
- https://onnxruntime.ai/docs/  
- https://onnxruntime.ai/docs/performance/olive.html  

---
> **Advertencia:** El contenido de este repositorio se ofrece “tal cual”, sin ningún tipo de soporte, garantía o responsabilidad. Úsalo bajo tu propia responsabilidad; no es información oficial ni está respaldada por terceros.

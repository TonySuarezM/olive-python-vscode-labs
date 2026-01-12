# Guía para principiantes — M5_repo_oficial_editable_install.ipynb  
## Trabajar con el repo oficial de Olive y “editable install” en VS Code + Jupyter

Este documento explica el notebook **`M5_repo_oficial_editable_install.ipynb`** para no iniciados: qué estás haciendo, por qué es útil, cómo verificar que funcionó y cómo arreglar los fallos más comunes.

---

## 1) Objetivo de M5 (qué aprendes aquí)

En M0–M4 has usado Olive como “paquete instalado”. En **M5** haces el paso típico de desarrollo:

1. **Clonar** el repositorio oficial `microsoft/Olive`.
2. Crear un entorno **aislado** para el repo (`.venv`).
3. Instalar Olive en modo **editable** (`pip install -e .`) para:
   - importar `olive` desde tu código
   - poder leer/depurar el código del repo
   - probar cambios sin reinstalar cada vez (editas y se refleja al ejecutar)

Olive documenta explícitamente el flujo de **Editable install** con `git clone` + `pip install -e .`. (ver Fuentes)  

---

## 2) Conceptos mínimos

### 2.1 ¿Qué significa “clonar un repo”?
Clonar crea una copia local completa del repositorio (historial, ramas, etc.). VS Code lo soporta desde la interfaz (Source Control) o desde la Command Palette con **Git: Clone**. (ver Fuentes)

### 2.2 ¿Qué es un entorno virtual (`venv`)?
`venv` crea un entorno de Python con paquetes aislados del Python “base”. La doc oficial explica que el entorno se crea sobre un Python existente y, por defecto, queda aislado de los paquetes del sistema. (ver Fuentes)

### 2.3 ¿Qué es una instalación “editable”?
Olive recomienda que, si quieres contribuir o probar cambios, uses `pip install -e .` (editable install). (ver Fuentes)

En la documentación de `pip`, “editable installation” está relacionada con el soporte de instalación editable (PEP 660) y cómo pip la realiza. (ver Fuentes)

---

## 3) Qué hace el notebook (paso a paso)

> El notebook mezcla dos “caminos”: hacerlo desde VS Code (UI) o desde terminal. Tú eliges.

### 3.1 Comprobación inicial del entorno
El notebook imprime:
- `sys.executable` / `sys.version`: te dice qué Python está ejecutando el notebook.
- `git --version`: confirma que Git está instalado y disponible.
- `pip show olive-ai`: confirma si Olive está instalado en ese entorno y su ubicación.

**Por qué:** en VS Code + notebooks es fácil instalar en un Python y ejecutar con otro.

---

### 3.2 Clonar el repositorio (opción A: VS Code)
En VS Code puedes clonar con la Command Palette:
- `Ctrl+Shift+P` → **Git: Clone** → pegas la URL del repo → eliges carpeta. (ver Fuentes)

Este paso es útil si no quieres usar terminal.

---

### 3.3 Clonar el repositorio (opción B: terminal / celda con git)
El notebook también muestra el camino clásico:

```bash
git clone https://github.com/microsoft/Olive
```

VS Code también documenta que puedes clonar desde Source Control o con “Git: Clone”. (ver Fuentes)

---

### 3.4 Crear `.venv` dentro del repo clonado
La doc de Python describe la creación de entornos con:

```bash
python -m venv .venv
```

Y comenta que tras crear el entorno puedes activarlo ejecutando el script de activación del directorio `bin/` o `Scripts/`. (ver Fuentes)

**Detalle importante (doc oficial):** no es obligatorio “activar” el venv si llamas al Python del venv por ruta completa. (ver Fuentes)

---

### 3.5 Instalar Olive “desde el código” (editable install)
Dentro del repo `Olive/`, el notebook te guía a:

```bash
pip install -e .
```

Esto está documentado por Olive en su sección de **Editable install**. (ver Fuentes)

---

### 3.6 Seleccionar el kernel correcto en VS Code (lo más importante del módulo)
Aunque hayas creado `.venv`, el notebook **no lo usará** si no seleccionas el kernel correcto.

En VS Code, los notebooks tienen un **kernel picker** (“Select Kernel”) y puedes elegir el entorno de Python que quieres usar. (ver Fuentes)

En el tutorial de Data Science de VS Code, se muestra explícitamente el paso de **Select Kernel** y elegir el entorno creado. (ver Fuentes)

---

### 3.7 Verificación: comprobar que estás ejecutando “el Olive del repo”
El notebook te hace comprobar:

1) `sys.executable` apunta a `.../Olive/.venv/...`
2) `import olive` funciona  
3) `olive.__file__` (ubicación del módulo) apunta al árbol del repo o a tu entorno local  
4) `python -m olive -h` muestra ayuda de la CLI

---

## 4) Cómo verificar que el editable install está bien (checklist)

✅ Checklist mínima:

1) En el notebook:
- `sys.executable` apunta a `external/Olive/.venv/...` (o la carpeta donde clonaste).  

2) En terminal (con el Python del venv):
- `python -m pip show olive-ai` muestra Olive instalado y su ubicación.

3) (Opcional, muy útil) `pip list`:
La doc de `pip` indica que, cuando hay paquetes en modo editable, `pip list` puede mostrar una columna **Editable project location** con la ruta del proyecto editable. (ver Fuentes)

---

## 5) Errores comunes y diagnóstico (los más frecuentes)

### A) El notebook usa el Python global (no el `.venv`)
**Síntoma:** `sys.executable` muestra algo como `C:\Users\...\Python313\python.exe`.

**Solución:** usa **Select Kernel** y elige el `.venv` del repo. (ver Fuentes)

---

### B) Git no está disponible
**Síntoma:** `git --version` falla.

**Solución:** instala Git o clona desde VS Code si tu instalación lo integra (en cualquier caso, VS Code documenta el flujo Git: Clone). (ver Fuentes)

---

### C) PowerShell no permite activar `Activate.ps1`
La doc de `venv` (en Windows) menciona que puede ser necesario ajustar la política de ejecución para permitir el script `Activate.ps1`. (ver Fuentes)

---

### D) “Instalé Olive, pero importo otra cosa”
Si tienes varias instalaciones, el kernel manda.  
Verifica:
- `sys.executable`
- `olive.__file__`
- `python -m pip show olive-ai` ejecutado con el mismo Python del kernel

---

## 6) Qué sigue (M6)
En **M6** haces el proyecto final: pipeline completo (modelo → Olive → ORT → métricas → reporte).

---

## Fuentes oficiales usadas (URLs exactas)

> Las URLs se incluyen en bloque de código para que puedas abrirlas tal cual.

```text
# VS Code — clonar repos y Git: Clone
https://code.visualstudio.com/docs/sourcecontrol/quickstart
https://code.visualstudio.com/docs/sourcecontrol/repos-remotes
https://code.visualstudio.com/docs/sourcecontrol/github

# VS Code — Jupyter / kernels
https://code.visualstudio.com/docs/datascience/jupyter-notebooks
https://code.visualstudio.com/docs/datascience/jupyter-kernel-management
https://code.visualstudio.com/docs/datascience/data-science-tutorial

# Python — venv
https://docs.python.org/3/library/venv.html
https://docs.python.org/es/3.10/library/venv.html
https://docs.python.org/uk/3/library/venv.html

# Olive — instalación desde fuente y editable install
https://microsoft.github.io/Olive/how-to/installation.html
https://microsoft.github.io/Olive/0.2.1/getstarted/installation.html
https://microsoft.github.io/Olive/0.5.0/getstarted/installation.html
https://github.com/microsoft/Olive

# pip — editable installs / cómo se representan
https://pip.pypa.io/en/stable/reference/build-system/
https://pip.pypa.io/en/stable/_sources/cli/pip_list.rst.txt
```

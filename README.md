# COSIPY en Mac M2 (COSIPY-Mac)

Este documento describe los pasos para preparar el ambiente **COSIPY-Mac** en una MacBook con chip **Apple Silicon (M1/M2)** utilizando **Miniforge/mamba**. Incluye además los inconvenientes encontrados durante la instalación y sus soluciones.

---

## 🔧 Requisitos previos

* macOS actualizado
* [Xcode Command Line Tools](https://developer.apple.com/xcode/resources/) para compilación básica:

  ```bash
  xcode-select --install
  ```

---

## 🚀 Preparación del entorno

```bash
set -euo pipefail

# 0) (Opcional) Instalar herramientas de Apple
xcode-select --install || true

# 1) Instalar Miniforge (si no lo tienes)
if [ ! -d "$HOME/miniforge3" ]; then
  curl -L -o Miniforge3-MacOSX-arm64.sh \
    https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-MacOSX-arm64.sh
  bash Miniforge3-MacOSX-arm64.sh -b -p "$HOME/miniforge3"
fi

# 2) Activar conda y configurar canales
source "$HOME/miniforge3/etc/profile.d/conda.sh"
conda init "$(basename "$SHELL")" >/dev/null 2>&1 || true
conda config --add channels conda-forge || true
conda config --set channel_priority strict

# 3) Crear y activar entorno
if conda env list | grep -q '^cosipy-mac '; then
  echo "El entorno cosipy-mac ya existe, activando..."
else
  mamba create -n cosipy-mac python=3.10 "numpy=1.23.*" -y
fi

conda activate cosipy-mac

# 3.1) Instalar mamba dentro del entorno (si no está disponible)
conda install -n cosipy-mac mamba -c conda-forge -y

# 4) Instalar dependencias científicas
mamba install -y \
  "pandas<2" xarray netcdf4 scipy matplotlib dask numba scikit-learn \
  gdal pyproj rasterio shapely geopandas h5py richdem tqdm ipykernel

# 5) Kernel Jupyter
python -m ipykernel install --user --name cosipy-mac --display-name "Python (COSIPY M2)"

# 6) Clonar COSIPY (si no existe)
cd "$HOME"
if [ ! -d cosipy ]; then
  git clone https://github.com/cryotools/cosipy.git
fi
cd cosipy

# 7) Probar ejecución desde fuente
python COSIPY.py -h || true

# (Opcional) Instalar como paquete editable
# pip install -e .
# run-cosipy -h

echo "✅ COSIPY preparado en el entorno 'cosipy-mac' con NumPy 1.23.x en Apple Silicon."
```

---

## ⚠️ Inconvenientes encontrados y soluciones

1. **Error con `numpy=1.23.*` en zsh**

   * Problema: `zsh: no matches found: numpy=1.23.*`
   * Solución: poner comillas → `"numpy=1.23.*"`

2. **Error con `pandas<2` en zsh**

   * Problema: `zsh: no such file or directory: 2`
   * Solución: usar comillas → `"pandas<2"`

3. **Falta `dask-jobqueue`**

   * Problema: `ModuleNotFoundError: No module named 'dask_jobqueue'`
   * Solución: instalar con `mamba install dask-jobqueue` o `pip install dask-jobqueue`

4. **Falta `tomli`**

   * Problema: `ModuleNotFoundError: No module named 'tomli'`
   * Solución: instalar con `mamba install tomli` o `pip install tomli`

5. **Mamba no disponible en el entorno**

   * Problema: al activar el entorno con `conda activate cosipy-mac`, no estaba `mamba` instalado.
   * Solución: instalar `mamba` dentro del entorno:

     ```bash
     conda install -n cosipy-mac mamba -c conda-forge -y
     ```

---

## ✅ Verificación final

Al ejecutar:

```bash
python COSIPY.py -h
```

Debe mostrarse:

```
usage: COSIPY [-h] [-c <path>] [-x <path>] [-s <path>]

Coupled snowpack and ice surface energy and mass balance model in Python.
```

---

## 📌 Notas

* Para **activar entornos** siempre se usa `conda activate`, no `mamba activate`.
* En MacBook M2 basta con usar `-c` (archivo de configuración) y `-x` (archivo de constantes).
* El argumento `-s` (configuración SLURM) solo aplica en clusters HPC.

---

✍️ Autor: **Jonathan Aparco Lara**

# Practica_MLOPS-main

API FastAPI para inferencia del modelo agrícola, métricas en vivo y dashboard estático.

## Requisito importante: versión de Python y scikit-learn

El artefacto `models/modelo.joblib` contiene objetos serializados de scikit-learn 1.6.1.
No debe cargarse con scikit-learn 1.9.x: las versiones incompatibles pueden producir
`InconsistentVersionWarning` y errores como `_RemainderColsList`.

Este proyecto fija `scikit-learn==1.6.1` y Python 3.12 para mantener el entorno
alineado con el artefacto existente.

## Instalación recomendada con uv

Ejecuta PowerShell desde la raíz `Practica_MLOPS-main`:

```powershell
# Detener cualquier servidor anterior con Ctrl+C.
# Crear/sincronizar el entorno definido en la raíz:
uv python install 3.12
uv lock
uv sync

# Iniciar la API y vigilar el dashboard (la ruta es relativa a la raíz):
uv run uvicorn api.main:app --host 127.0.0.1 --port 8000 --reload --reload-dir api --reload-dir dashboard
```

Si prefieres trabajar desde `api/`, ejecuta allí:

```powershell
uv python install 3.12
uv lock
uv sync
uv run uvicorn main:app --host 127.0.0.1 --port 8000 --reload --reload-dir . --reload-dir ..\dashboard
```

No uses `--reload-dir dashboard` estando dentro de `api/`, porque esa ruta buscaría
`api/dashboard`, que no existe. El dashboard real está en la carpeta hermana `dashboard/`.

Verifica el entorno activo:

```powershell
uv run python -c "import sys, sklearn; print(sys.executable); print('scikit-learn', sklearn.__version__)"
```

Debe mostrar `scikit-learn 1.6.1`.

Abre Swagger en http://127.0.0.1:8000/docs.

## Si el modelo aún no carga

Desde la raíz del proyecto, activa el mismo entorno y ejecuta `notebooks/entrenamiento.ipynb`
completo para regenerar `models/modelo.joblib` y las métricas. No elimines ni reemplaces el
modelo antes de tener una copia de seguridad. Entrena, guarda y sirve el artefacto usando
el mismo entorno de dependencias.

## Archivos involucrados en el error

- `api/pyproject.toml`: causa principal cuando `uv` resuelve scikit-learn 1.9.x.
- `api/.python-version`: fijaba Python 3.14, mientras el proyecto se alinea a Python 3.12.
- `api/requirements.txt`: dependencias usadas con pip; ahora también fija scikit-learn 1.6.1.
- `pyproject.toml`: configuración de dependencias de la raíz; también debía evitar resolver 1.9.x.
- `uv.lock` y `api/uv.lock`: deben regenerarse con `uv lock` después de cambiar los archivos TOML.
- `api/main.py`: `cargar_artefactos()` carga el modelo en `joblib.load(MODEL_PATH)` durante el arranque.
- `models/modelo.joblib`: artefacto serializado con el pipeline/modelo de ML.
- `notebooks/entrenamiento.ipynb`: fuente para regenerar el artefacto si no puede recuperarse.
- `dashboard/`: recursos estáticos; la ruta de `--reload-dir` depende del directorio desde el que se inicia Uvicorn.

`logs/api.log` puede contener detalles adicionales del fallo, pero no es la causa raíz.

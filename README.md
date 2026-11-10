# instacart-analisis

Análisis de datos de Instacart con Python, pandas y JupyterLab.

**Pregunta orientadora (EV06):** ¿cómo preparar y estructurar los datos de Instacart para obtener información confiable sobre los hábitos de compra?

## Estructura del proyecto

```
instacart-analisis/
├── README.md
├── .gitignore
├── requirements.txt
├── data/
│   ├── raw/                  # CSV originales (no se versionan)
│   └── processed/            # archivos generados por pandas_labs.ipynb
├── notebooks/
│   ├── 00_entorno.ipynb      # actividad anterior: entorno
│   ├── 01_exploracion.ipynb  # actividad anterior (EV04): exploración
│   └── pandas_labs.ipynb     # EV06: lecciones 2, 3 y 4 con laboratorios y retos
└── reports/
    └── processing_report.md  # informe de procesamiento de datos
```

## Datos

Los cinco CSV de Instacart usan `;` como separador y deben copiarse en `data/raw/`:

- `instacart_orders.csv`
- `order_products.csv`
- `products.csv`
- `aisles.csv`
- `departments.csv`

Los CSV no se suben a GitHub (están en `.gitignore`).

## Preparar el entorno (Windows, cmd)

```bat
python -m venv .venv
.venv\Scripts\activate.bat
python -m pip install -r requirements.txt
```

`requirements.txt` incluye pandas, NumPy, PyArrow (para guardar archivos Parquet) y JupyterLab.

## Ejecutar el análisis

```bat
jupyter lab
```

1. Abrir `notebooks/pandas_labs.ipynb`.
2. Usar *Kernel → Restart Kernel and Run All Cells*. El cuaderno tarda un poco porque `order_products.csv` tiene 4,5 millones de filas.
3. Las rutas son relativas al cuaderno: lee de `../data/raw` y escribe en `../data/processed`.

## Contenido de `pandas_labs.ipynb`

| Sección | Contenido |
| --- | --- |
| Lección 1 | Carga de los cinco archivos |
| Lección 2 | Limpieza y estandarización; Laboratorio guiado 2 (pipeline con informe de cambios); Reto 2 (7 ejercicios) |
| Lección 3 | Filtrado, ordenamiento y agrupación; Laboratorio guiado 3 (perfil de hábitos de compra); Reto 3 (7 ejercicios) |
| Lección 4 | Combinación de datasets; Laboratorio guiado 4 (tabla de análisis por producto); Reto 4 (7 ejercicios) |

## Archivos generados en `data/processed/`

| Archivo | Descripción |
| --- | --- |
| `informe_limpieza.csv` | Filas, nulos y memoria de cada tabla en cada paso del pipeline de limpieza |
| `perfil_habitos.csv` | Porcentaje de pedidos por franja horaria según el segmento de cliente |
| `resumen_departamentos.csv` | Artículos pedidos y tasa de recompra por departamento |
| `ficha_productos.parquet` | Una fila por producto: veces pedido, clientes, tasa de recompra y posición media en el carrito |
| `lideres_por_departamento.csv` | Producto con mayor tasa de recompra en cada departamento (mínimo 1 000 pedidos) |
| `instacart_maestra.parquet` | Tabla maestra de 4 545 007 filas y 14 columnas. **No se versiona** (pesa unos 104 MB, más que el límite de GitHub); se regenera al ejecutar el cuaderno |

## Flujo de trabajo en Git

- Rama base: `develop`.
- Rama de trabajo: `feature/pandas-data-processing`.
- Commits de la actividad:
  - `feat: complete lesson 2 data cleaning`
  - `feat: complete lesson 3 data analysis`
  - `feat: complete lesson 4 data merging`
  - `docs: update README and processing report`
- La rama se fusiona con `develop` al terminar.

## Informe

El resumen de decisiones de limpieza, hallazgos y validaciones está en [`reports/processing_report.md`](reports/processing_report.md).

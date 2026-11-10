# Informe de procesamiento de datos: Instacart

**Actividad:** GA1-220501094-01-AA1-EV06  
**Notebook:** `notebooks/pandas_labs.ipynb`

**Pregunta orientadora:** ¿cómo preparar y estructurar los datos de Instacart para obtener información confiable sobre los hábitos de compra?

## 1. Datos de partida

| Tabla | Filas | Columnas |
| --- | --- | --- |
| `orders` | 478 967 | 6 |
| `order_prods` | 4 545 007 | 4 |
| `products` | 49 694 | 4 |
| `aisles` | 134 | 2 |
| `departments` | 21 | 2 |

Los archivos usan `;` como separador. Las claves de `products`, `aisles` y `departments` son únicas, `order_id` en `orders` no lo es (por los 15 duplicados) y no hay valores huérfanos entre tablas.

## 2. Limpieza (Lección 2)

Para cada problema se decidió entre eliminar, corregir o conservar según lo que significa el dato.

| Tabla | Problema | Evidencia | Decisión |
| --- | --- | --- | --- |
| `orders` | Filas duplicadas | 15 pedidos repetidos, todos el miércoles a las 2 a. m. | Eliminar |
| `orders` | Tipos sobredimensionados | 23,0 MB | Reducir a 7,2 MB con `astype` |
| `orders` | Nulos en `days_since_prior_order` | 28 817 (todos son primer pedido del cliente) | Conservar |
| `products` | Nulos en `product_name` | 1 258 (pasillo y departamento `missing`) | Rellenar con `'Unknown'` |
| `products` | Espacios dobles e invisibles | 403 y 16 | Estandarizar texto y formato título |
| `products` | Nombres repetidos con `product_id` distinto | 141 | Conservar; agrupar por `product_id` |
| `order_prods` | Nulos en `add_to_cart_order` | 836 en 70 pedidos de más de 64 productos | Conservar |
| `order_prods` | Tipos sobredimensionados | 145,4 MB | Reducir a 54,5 MB |
| `aisles`, `departments` | Sin duplicados ni problemas de texto | 0 | Sin cambios |

**Informe de cambios del pipeline** (`data/processed/informe_limpieza.csv`):

| Tabla | Paso | Filas | Nulos | Memoria (MB) |
| --- | --- | --- | --- | --- |
| `orders` | 0 crudo | 478 967 | 28 819 | 23,0 |
| `orders` | 1 sin duplicados | 478 952 | 28 817 | 23,0 |
| `orders` | 2 tipos | 478 952 | 28 817 | 7,2 |
| `products` | 0 crudo | 49 694 | 1 258 | 3,1 |
| `products` | 1 nombres | 49 694 | 0 | 3,1 |
| `order_prods` | 0 crudo | 4 545 007 | 836 | 145,4 |
| `order_prods` | 1 tipos | 4 545 007 | 836 | 54,5 |

Observaciones de la limpieza:

- Los 836 nulos de `add_to_cart_order` pertenecen a pedidos con 65 a 127 productos. El valor máximo registrado es 64, así que el sistema solo numera hasta esa posición.
- Hay 9 547 filas de `order_prods` (0,21 %) de productos `'Unknown'`. Se conservan porque son compras reales.
- Cambiaron 6 321 nombres de producto: 5 902 por el formato título y 419 por espacios dobles o invisibles.

**Validación automática:** el pipeline termina con aserciones (clave `order_id` única, horas entre 0 y 23, productos con nombre, sin espacios invisibles, `reordered` solo 0 o 1). El resultado es idéntico al de la limpieza manual.

## 3. Hábitos de compra (Lección 3)

- **Hora pico:** las 10 a. m., con 40 578 pedidos. Los domingos el pico se mueve a la tarde (15 h, 14 h y 13 h).
- **Días:** domingo (84 090 pedidos) y lunes (82 185) concentran más pedidos; jueves tiene menos (59 810).
- **Espera entre pedidos:** la media va de 10,62 días (22 h) a 12,52 días (0 h). La mediana es de 8 días en fin de semana y de 7 el resto de la semana.
- **Segmentos de clientes** (según los pedidos que aparecen en los datos):

| Segmento | Clientes | % de clientes | Pedidos | Espera mediana (días) |
| --- | --- | --- | --- | --- |
| ocasional (1 pedido) | 55 357 | 35,2 | 55 357 | 16 |
| regular (2 a 4) | 71 553 | 45,4 | 191 649 | 11 |
| frecuente (5 o más) | 30 527 | 19,4 | 231 946 | 6 |

- Los clientes frecuentes (19,4 %) hacen el 48,4 % de los pedidos.
- Todos los segmentos compran sobre todo en la tarde, pero los frecuentes compran más en la mañana (35,2 %) que los ocasionales (30,0 %).
- La tasa de recompra baja con la posición en el carrito: de 0,679 en la posición 1 a 0,617 en la 5.

## 4. Combinación de datasets (Lección 4)

- **Auditoría con `indicator`:** 450 046 pedidos tienen productos y 28 906 (6 %) no tienen ninguno. Un `inner` los habría eliminado sin aviso.
- **Tabla maestra:** se construyó a partir de `order_prods` (tabla de hechos) con cuatro `merge` de tipo `left` y `validate='m:1'`. Conserva las 4 545 007 filas y pasa de 4 a 14 columnas; un `assert` comprueba que no se perdieron ni crearon filas.
- **Producto más pedido:** Banana (66 050 veces, 33 274 clientes, tasa de recompra 0,844).
- **Departamentos:** `produce` es el que más artículos tiene (1 328 302; recompra 0,651), seguido de `dairy eggs` (759 842; 0,671) y `snacks` (404 850; 0,574).
- **Productos más fieles:** el producto más pedido no siempre es el más fiel. Entre los productos con al menos 1 000 pedidos (739), el de mayor tasa de recompra en lácteos es Whole Organic Omega 3 Milk (0,863).
- **Tabla por producto:** `ficha_productos.parquet` tiene 45 573 filas (una por producto pedido al menos una vez).

## 5. Archivos generados

Ver la tabla de `data/processed/` en el `README.md`. La tabla maestra (`instacart_maestra.parquet`, unos 104 MB) no se versiona por superar el límite de GitHub y se regenera ejecutando el cuaderno.

## 6. Conclusiones

1. Para obtener información confiable, el orden importa: primero se eliminan duplicados, después se corrigen tipos, se decide qué hacer con los nulos, se estandariza el texto y solo entonces se agrupa o se combina.
2. No todo nulo es un error: los de `days_since_prior_order` y `add_to_cart_order` tienen una causa explicable y se conservaron.
3. Las uniones se validan: `validate` evita duplicar filas en silencio e `indicator` permite ver qué se pierde.
4. La tabla de análisis por producto y la tabla maestra dejan los datos listos para estudiar los hábitos de compra por producto, departamento, día, hora y tipo de cliente.

## 7. Limitaciones

- Los segmentos de clientes se calculan con los pedidos que aparecen en este conjunto de datos, no con el historial completo de cada cliente.
- 28 906 pedidos no tienen productos registrados, así que no entran en los análisis de la tabla maestra.
- Los productos con nombre repetido y `product_id` distinto no se fusionaron; los análisis agrupan por `product_id`.

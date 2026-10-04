# Esquema de datos PEA-i, versión 3

Ambos programas usan CSV UTF-8 con encabezados exactos y `\n` como fin de registro. Los campos de texto pueden contener comas, comillas y saltos de línea mediante las reglas normales de CSV. Una carpeta completa contiene `manifest.csv`, cuatro archivos de entidades, tres de relaciones, `cola_validacion.csv` e `historial.csv`. Los archivos vacíos conservan el encabezado. Los guardados nuevos declaran la versión `3`; las carpetas de versiones `1` y `2` se pueden abrir y se convierten al guardar.

| Archivo | Campos, en orden |
|---|---|
| `manifest.csv` | `version,guardado` |
| `grupos.csv` | `id,nombre,codigo_gruplac,fecha_creacion,unidad,responsable,categoria,descripcion,objetivos,mision,vision,lineas,url,fuente,activo` |
| `investigadores.csv` | `id,nombre,codigo_cvlac,afiliacion,categoria,contacto,url,fuente,activo` |
| `productos.csv` | `id,titulo,anio,fecha,familia,tipologia,validacion,observacion,doi,url,fuente,activo` |
| `planes.csv` | `id,grupo_id,nombre,inicio,fin,objetivo,indicador,meta,actividad,activo` |
| `membresias.csv` | `grupo_id,investigador_id,rol,inicio,fin,activo` |
| `autorias.csv` | `producto_id,investigador_id,orden,rol,activo` |
| `grupos_productos.csv` | `grupo_id,producto_id,origen,activo` |
| `cola_validacion.csv` | `id,producto_id,motivo,creado` |
| `historial.csv` | `orden,snapshot` |

## Reglas

- `id` y nombres/títulos son obligatorios; los IDs de entidades son únicos y estables. Las relaciones se identifican por sus dos extremos. Todos los extremos deben existir.
- `activo` vale `1` o `0`. `validacion` vale `validado` o `rechazado` y la calcula el programa (ver [Validación automática](#validación-automática)); un valor escrito a mano o importado se recalcula. `observacion` es la nota libre de la revisión.
- Las fechas no vacías usan `AAAA-MM-DD`. El año del producto, si se conoce, está entre 1900 y el año siguiente al actual. Si hay fecha y año, deben coincidir.
- Un DOI no vacío se compara sin distinguir mayúsculas ni el prefijo `https://doi.org/`. También se exige unicidad de código GrupLAC/CvLAC no vacío. Los títulos iguales sin DOI se revisan manualmente: no se fusionan automáticamente.
- No hay borrado en cascada de entidades. Eliminar una entidad con vínculos, un grupo con planes o un producto con revisiones pendientes requiere resolver primero esas dependencias. Desactivar conserva la historia.
- Las listas dobles guardan entidades. Las multilistas guardan vínculos con recorrido por cualquiera de los dos extremos. Los índices auxiliares aceleran la búsqueda por ID y por código GrupLAC, CvLAC o DOI sin sustituir esas estructuras. La pila conserva hasta 30 acciones para deshacer; la cola atiende las revisiones en orden FIFO.
- Los totales institucionales cuentan IDs de producto activos una sola vez, incluso con varios autores o grupos. Las vistas de grupo/investigador solo usan relaciones activas. Un rango de años excluye productos sin año. El filtro de validación se aplica antes de agrupar.
- Solo grupos e investigadores tienen `categoria`. GrupLAC publica la clasificación del grupo, no la de cada producto, y la categoría oficial de un producto depende de índices externos que las fichas no incluyen. Por eso la versión 3 retira ese campo de `productos.csv`. La interfaz muestra las categorías solo en el módulo de grupos: la del grupo y la de sus integrantes.
- La carpeta `.backup` conserva los CSV previos al último guardado. La interfaz Tkinter trata `data/real` como muestra de solo lectura y requiere **Guardar como** para guardar cambios.

## Validación automática

Un producto queda `validado` cuando cumple todas las reglas siguientes y `rechazado` en otro caso. Ambos motores evalúan las mismas reglas en el mismo orden y devuelven sus códigos:

| Código | Regla incumplida |
|---|---|
| `anio` | Sin año de publicación. |
| `tipologia` | Sin tipología. |
| `autor` | Sin una autoría activa de un investigador activo. |
| `grupo` | Sin un vínculo activo con un grupo activo. |
| `doi` | El DOI no tiene la forma `10.NNNN/sufijo` (4 a 9 dígitos y un sufijo sin espacios; se admite el prefijo `https://doi.org/`). |
| `fuente` | Sin URL `http(s)://` ni DOI válido que permita verificar el producto. |

La validación se recalcula al crear, editar, eliminar, activar o desactivar el producto, sus autorías y vínculos con grupos, y al cambiar el estado de un investigador o grupo enlazado. También se recalcula al deshacer y al abrir una carpeta. No equivale a una evaluación oficial de Minciencias: comprueba que el registro está completo, enlazado y es verificable.

La cola atiende los productos que necesitan revisión. **Enviar rechazados** encola en una sola acción los productos activos rechazados que aún no están en la cola; su motivo se guarda como `reglas:` seguido de los códigos incumplidos (por ejemplo `reglas:autor,doi`). Cerrar una revisión exige una nota, que se guarda en `observacion`; el estado sigue dependiendo solo de las reglas.

## Compatibilidad y apertura en hojas de cálculo

Al abrir una carpeta de versión `1`, ambos motores admiten `sigla` en `grupos.csv` (se descarta), un `productos.csv` sin `validacion` y la ausencia de `cola_validacion.csv` (se inicia vacía). En las versiones `1` y `2`, `productos.csv` incluye `categoria`: se retira y, si tenía un valor, se conserva al final de `observacion` como «Categoría registrada antes de la versión 3: …». La validación se recalcula al abrir. Una carpeta de versión `3` exige todos los archivos y encabezados de la tabla. El historial de deshacer anterior se conserva y se adapta al usarlo.

Para evitar la interpretación de fórmulas al abrir los CSV persistidos en una hoja de cálculo, las versiones `2` y `3` anteponen una comilla simple a las celdas cuyo primer carácter es `=`, `+`, `-`, `@`, tabulación, retorno de carro o comilla simple. Ambos motores retiran exactamente esa comilla al cargar la carpeta; así se conserva el valor original. La opción **Archivo → Exportar para hoja de cálculo** produce una copia para consulta que neutraliza fórmulas y no incluye historial ni cola; no se debe usar como carpeta de trabajo.

## Entrada y salida

**Entradas compartidas:** registros de entidades y relaciones, carpeta persistida y archivos CSV por tipo. Python usa formularios Tkinter y consulta URL públicas HTTP/HTTPS de HTML, texto, CSV o PDF de texto. La consulta presenta primero el contenido y la fuente; la extracción automática de campos se limita a fichas GrupLAC/CvLAC o metadatos explícitos de artículos académicos. En GrupLAC puede incorporar el censo de integrantes con su período textual, el plan estratégico, productos fechados de secciones bibliográficas y técnicas reconocidas, vínculos al grupo y autorías cuyos nombres coincidan con el censo. Otros sitios permiten crear un registro tras completar y revisar el formulario. Un CSV remoto solo se incorpora si coincide con el esquema PEA-i. Por defecto, la ventana envía las operaciones de datos al [backend C++](protocolo_backend.md); la implementación Python independiente se activa con `--python-backend`. C++ también ofrece menús de consola e importa CSV.

**Salidas:** fichas consultables, relaciones por ambos extremos, revisiones pendientes, totales por vista y ventana, distribuciones por año, tipología, validación y reglas incumplidas, y carpeta CSV persistida. El dashboard Python dibuja gráficos con Tkinter Canvas; los campos vacíos siguen contando en el total y las barras incluyen «Sin dato» en su denominador. El histograma agrupa los años menores o iguales al año actual menos 20 en «20 años o más», sin modificar los años almacenados ni las estadísticas originales. La agrupación visual se aplica después del filtro; los años posteriores sin registros tienen frecuencia cero. La vista previa de tipologías puede agrupar las menos frecuentes; el desglose completo permanece en Gráficos. C++ presenta tablas y resúmenes en consola. `historial.csv` conserva el nombre de columna `snapshot` por compatibilidad: puede contener una instantánea anterior o un registro JSON de cambios reversibles (`__undo`). Ambos modos pueden leer y deshacer los dos formatos. C++ escribe cambios compactos; el modo Python independiente aún escribe instantáneas.

Las fichas muestran los campos con datos y resumen los vacíos en una línea «Sin registrar»; sus asociaciones están paginadas, incluso las inactivas. Las tablas ocultan las columnas que ningún registro ha llenado (por ejemplo afiliación o fechas de membresía en la muestra actual); reaparecen en cuanto un registro tiene ese dato. Los formularios conservan todos los campos para poder editarlos. Las estadísticas de la ficha cuentan productos activos y vínculos activos; abrir un producto no cambia los filtros de las listas. Los selectores guardan el ID estable, pero muestran nombres y buscan en todo el conjunto. La validación de formularios se realiza antes de cerrar la ventana, incluida la validación de integridad del motor activo.

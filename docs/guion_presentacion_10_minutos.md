# Guion de presentación de PEA-i UPC — 10 minutos

## 0:00–0:40 · Presentación — Integrante 1

[Los tres integrantes permanecen visibles durante todo el video. Mostrar el nombre del proyecto y de la Universidad Popular del Cesar.]

«Buenos días. Somos estudiantes de Ingeniería de Sistemas de la Universidad Popular del Cesar. Presentamos PEA-i, el Programa Estadístico de Análisis de Investigación desarrollado para el Taller 2 de Estructura de Datos. El sistema organiza grupos de investigación, investigadores, planes y productos obtenidos de fuentes públicas de Scienti. Construimos dos soluciones: una aplicación de consola en C++ y una aplicación Python con interfaz Tkinter. Durante estos diez minutos mostraremos el modelo, las estructuras exigidas y las operaciones principales».

## 0:40–1:30 · Arquitectura y dos soluciones — Integrante 1

[Mostrar los archivos `src/cpp/Taller2_REMR.cpp` y `src/python/Taller2_REMR.py`; abrir la aplicación Tkinter.]

«El archivo C++ funciona por sí solo mediante menús de consola. El archivo Python también tiene su propio motor de datos y puede ejecutarse de forma independiente. En el uso habitual, Tkinter inicia el proceso C++ y le envía solicitudes JSON por la entrada estándar; recibe las respuestas por la salida estándar. Así usamos la interfaz Python con las operaciones del motor C++, sin servidor ni segunda ventana. Si el ejecutable C++ no está disponible, la interfaz usa el motor Python. En Ayuda, “Acerca de PEA-i”, podemos comprobar cuál motor está activo».

## 1:30–2:20 · Datos, entradas y salidas — Integrante 2

[Mostrar el dashboard inicial y después las pestañas Grupos, Investigadores, Productos y Planes.]

«La entrada del modelo son registros de grupos, investigadores, productos y planes, además de sus relaciones. Los grupos incluyen nombre, código GrupLAC, unidad y categoría; los investigadores, nombre, código CvLAC y afiliación; los productos, título, año, tipología, DOI y URL. Los productos no tienen categoría porque GrupLAC solo clasifica al grupo; el estado de validación lo calcula el programa. También entran archivos CSV y, desde Python, URL públicas o PDF de texto. Como salida obtenemos fichas consultables, vínculos, revisiones pendientes, tablas, gráficos y archivos persistidos. La carpeta incluida contiene una captura de 66 grupos, 2.736 investigadores y 6.363 productos de fichas públicas; la procedencia y los límites de esa captura están documentados».

## 2:20–3:15 · Estructuras de datos e hipercubo — Integrante 2

[Mostrar brevemente las clases de estructuras en ambos archivos o el diagrama de la especificación técnica.]

«Las entidades se almacenan en listas doblemente enlazadas, para insertar, recorrer en ambos sentidos y eliminar registros. Las multilistas representan membresías, autorías y vínculos grupo-producto; permiten consultar la relación desde cualquiera de sus extremos sin duplicar la entidad. La pila guarda acciones reversibles para Deshacer. La cola atiende revisiones en orden FIFO: el primer producto en entrar es el primero en procesarse. Los índices auxiliares aceleran las búsquedas, pero las estructuras enlazadas conservan los datos. El hipercubo es lógico: consultamos el conjunto de productos por grupo, investigador o producto, y aplicamos dimensiones como año, tipología, categoría y validación. Calculamos estas vistas cuando se solicitan; no almacenamos todas las combinaciones».

## 3:15–4:10 · Estadísticas y vistas — Integrante 1

[En Dashboard, alternar entre Todos, Grupo e Investigador; aplicar una ventana de dos o cinco años. Abrir Gráficos.]

«Aquí observamos el total de productos y las distribuciones por año, tipología, estado de validación y reglas incumplidas. La vista cambia según el grupo, el investigador o el producto seleccionado. El filtro temporal permite observar, por ejemplo, los últimos dos o cinco años. Un producto relacionado con varios grupos o autores se cuenta una sola vez dentro de la vista correspondiente. Si un campo no aparece en la fuente, el programa lo deja vacío y lo representa como “Sin dato”. La validación es automática: un producto queda validado si tiene año, tipología, un autor y un grupo activos, y una URL o un DOI válido. En la muestra, 6.323 productos quedan validados y 40 rechazados. No es una clasificación oficial de Minciencias».

## 4:10–5:15 · Consulta, edición y relaciones — Integrante 1

[En Grupos, buscar un registro y abrir Información. Mostrar sus integrantes y productos; volver a la tabla y señalar Nuevo, Editar, Activar/desactivar y Eliminar. Abrir Relaciones.]

«La búsqueda funciona sobre el conjunto completo y las tablas muestran resultados por páginas. La ficha de un grupo permite consultar sus datos y los registros relacionados. Las mismas operaciones de creación, consulta, modificación, desactivación y eliminación están disponibles para grupos, investigadores, productos, planes y relaciones. Las reglas de integridad evitan IDs duplicados y vínculos hacia registros inexistentes. Si una entidad tiene relaciones o revisiones pendientes, el sistema impide borrarla directamente; primero hay que resolver esas dependencias. La desactivación conserva el registro y permite excluirlo de las estadísticas sin perder su información».

## 5:15–6:05 · Cola, validación y deshacer — Integrante 2

[Abrir Cola de revisión. Mostrar el frente y los controles para encolar y procesar. Señalar Editar → Deshacer. Si se ejecuta una modificación, hacerlo en una copia guardada fuera de `data/real`.]

«Los productos rechazados se envían a la cola con el motivo calculado, por ejemplo “Sin autor activo registrado”. Revisar el frente permite corregir el producto, y la validación se recalcula sola; cerrar la revisión exige una nota. El siguiente trabajo pasa a ser el frente: esta es la operación FIFO. Deshacer usa la pila LIFO para revertir la última acción reversible. La cola y el historial se conservan al guardar, de modo que no desaparecen al cerrar el programa».

## 6:05–7:00 · Importación de fuentes — Integrante 3

[Abrir el menú Importar y señalar URL pública, PDF de texto y Hoja de cálculo. Si la conexión está disponible, mostrar la vista previa de una ficha GrupLAC; no confirmar su importación durante la grabación.]

«Python puede consultar una URL pública, mostrar su contenido y proponer campos cuando reconoce una ficha GrupLAC o CvLAC; para otros sitios permite revisar el contenido y completar manualmente los datos. También admite PDF de texto y archivos CSV. La consola C++ importa CSV y muestra una vista previa con filas aceptadas y rechazadas antes de aplicarlas. No afirmamos que cualquier página web se transforme automáticamente en registros. Las consultas externas se limitan a direcciones públicas y a tamaños definidos; cada dato incorporado conserva su fuente cuando está disponible. La demostración de estadísticas utiliza los CSV ya incluidos y funciona sin Internet».

## 7:00–7:55 · Persistencia y consola C++ — Integrante 3

[Mostrar Archivo → Guardar como sin sobrescribir `data/real`. Enseñar los nombres de los CSV de una carpeta de trabajo. Abrir brevemente la consola C++ y entrar a Estadísticas.]

«Ambos motores comparten un esquema CSV UTF-8: archivos separados para entidades y relaciones, un manifiesto de versión, la cola y el historial. Podemos iniciar vacíos o cargar una carpeta existente. La muestra `data/real` se abre como solo lectura desde la interfaz; para modificarla se guarda una copia. Aquí vemos la segunda solución funcionando de manera autónoma: C++ ofrece menús para entidades, relaciones, importación, estadísticas, revisión, deshacer y guardado. Sus resultados se presentan como tablas y números, tal como pide el enunciado».

## 7:55–8:45 · Decisiones técnicas — Integrante 2

[Mostrar el esquema de datos y el protocolo en `docs/`, o un fragmento del código de listas y multilistas.]

«El diseño separa la información de cada entidad de sus vínculos. Eso evita repetir un producto cuando tiene varios autores o pertenece a más de un grupo. La interfaz solicita páginas de resultados en vez de copiar toda la base de datos a cada tabla. El lector CSV procesa registros de forma incremental y valida referencias y formato. Cada guardado crea archivos temporales y conserva una copia anterior para recuperación. Estas decisiones ayudan a trabajar con miles de registros sin depender de una tabla gigante ni recalcular todas las combinaciones del hipercubo».

## 8:45–9:30 · Entregables — Integrante 3

[Mostrar el repositorio y abrir `docs/especificacion_tecnica.docx`.]

«Entregamos los dos archivos fuente solicitados, los datos persistidos, el documento Word de especificación técnica y el repositorio GitHub con instrucciones de ejecución. La especificación explica las estructuras, el modelo de datos, las reglas, los casos de uso y la correspondencia entre requisitos y diseño. En Windows, el lanzador abre la interfaz con doble clic; el repositorio incluye el ejecutable C++ de 64 bits para el uso integrado. En Linux, el README indica los comandos de ejecución y compilación».

## 9:30–10:00 · Cierre — Integrante 1

[Volver al dashboard. Los tres integrantes continúan visibles.]

«En resumen, PEA-i reúne las operaciones CRUD, listas, multilistas, pila y cola en dos implementaciones que comparten datos. Permite consultar la investigación por grupo, investigador y producto, filtrar por años y visualizar estadísticas sin duplicar los productos. La interfaz Python y la consola C++ pueden utilizarse por separado o juntas. Muchas gracias».

# Verificación de interfaz — PEA-i UPC

Serie de mejoras de 2026-10-02; alcance y prioridades en [PLAN_UI_UX.md](../PLAN_UI_UX.md).

## Comportamiento esperado

| Requisito del taller | Recorrido verificable |
|---|---|
| 1–4: grupos, investigadores, productos, integrantes y planes | Abrir una ficha de grupo; revisar integrantes, planes y productos; abrir un investigador o producto asociado y consultar sus relaciones. |
| 9 y 11: editar, desactivar y eliminar | Crear y editar desde tablas o fichas; mantener abierto el formulario ante campos inválidos; desactivar/reactivar vínculos; resolver dependencias antes de eliminar. |
| 10: ventana de observación | Elegir últimos 2/5 años o escribir un rango personalizado; verificar que tarjetas, barras, histograma y tabla cambien automáticamente. |
| 12: vistas y gráficos | Elegir Todos/Grupo/Investigador/Producto por nombre; consultar histograma y barras de tipología, validación y reglas incumplidas con el mismo contexto. |
| Estructuras y persistencia | Añadir dos revisiones, atender la primera, deshacer y guardar una copia; reabrir para comprobar registros, asociaciones, cola e historial. |
| Inicio sin datos | Archivo → Iniciar vacío; comprobar métricas en cero, mensajes vacíos y creación de un primer registro. |

El corte «20 años o más» es año actual menos 20 (2006 en 2026). Solo agrupa
visualmente los productos dentro del filtro seleccionado; los CSV conservan el
año original. «Sin dato» conserva su frecuencia y forma parte del denominador.
Las categorías oficiales de productos no se infieren (el esquema 3 retiró ese campo) y la validación es automática: completitud, vínculos activos y fuente verificable.

## Pruebas automáticas

```bash
cmake -S . -B build
cmake --build build
PEA_TEST_CPP=build/pea_cpp python3 -m unittest discover -s tests/python -v
ctest --test-dir build --output-on-failure
```

`test_gui.py` verifica 17 recorridos con Python y repite los mismos con C++
cuando se define `PEA_TEST_CPP`. Incluye selección posterior a la primera página,
errores de dominio sin perder campos, filtros automáticos, agrupación de años,
ficha completa por clic, relaciones por nombre y por ambos extremos, planes del
grupo exacto, edición y deshacer, revisión FIFO, CSV con tipo detectado y
guardar/reabrir con cola e historial. No modifica la muestra UPC.

Una sesión gráfica es necesaria. Si no existe, las pruebas de interfaz se
omiten con una explicación. En un entorno Linux sin escritorio se puede usar
Xvfb para ejecutar las mismas pruebas. El programa básico no depende de Xvfb.

## Recorrido manual en Windows

1. Abrir `Iniciar PEA-i.pyw`. Comprobar nombre UPC, navegación seleccionada,
   mensaje de carga y botón Guardar copia. Repetir con `--python-backend`.
2. Elegir Investigador en Dashboard; buscar un nombre que esté después de la
   primera página y seleccionarlo. Comprobar contexto, tarjetas y productos.
3. Elegir Personalizado y escribir 2010–2010. El resultado se actualiza sin
   pulsar un botón. Probar rango invertido y corregirlo; el error debe desaparecer.
4. Pulsar un producto. Consultar Resumen, Autores, Grupos y Estadísticas. Cerrar
   la ficha: la lista y su búsqueda deben permanecer intactas.
5. Abrir un grupo y revisar Integrantes, Planes y Productos. Crear un plan,
   vincular un investigador, editar un vínculo, desactivarlo y deshacer.
6. Crear un producto con año incorrecto. Debe conservar el título y permitir
   corregir el año. Intentar un ID/DOI duplicado y corregirlo en el mismo formulario.
7. Añadir dos productos a revisión. Registrar una observación y revisar el
   siguiente; comprobar el orden. Deshacer y comprobar que se restaura el frente.
8. Importar un CSV con nombre arbitrario y encabezados PEA-i. Comprobar tipo
   propuesto, vista previa, aceptadas/rechazadas y cambios pendientes.
9. Guardar una copia en una carpeta nueva; cerrar y reabrir. Comprobar datos,
   cola e historial. La carpeta `data/real` debe conservarse intacta.
10. Reducir a 900×620, alternar tema y contraer/desplegar el menú. Comprobar
    desplazamiento del dashboard, controles de página y botones de formularios.

En PowerShell, las regresiones C++ pueden ejecutarse con
`$env:PEA_TEST_CPP = 'bin/windows/pea_cpp.exe'` y
`py -3 -m unittest discover -s tests/python -v`.

## Alcance de la evidencia

Resultado final: **74 pruebas Python correctas**, incluidas **34 de interfaz**
(17 con cada motor), y **4/4 CTest**. Compilación CMake, `py_compile` y
`git diff --check` correctos.

La serie se verifica en Linux con ambos motores, con datos temporales y con la
muestra pública de 6.363 productos para inspección visual en claro/oscuro y
1280×850/900×620. Se utiliza un display aislado para las capturas. La revisión
manual de estas nuevas pantallas en Windows real permanece pendiente; la
compatibilidad del binario incluido y de sus huellas se comprueba por las pruebas
existentes. No se han cambiado C++, el protocolo ni el contrato CSV.

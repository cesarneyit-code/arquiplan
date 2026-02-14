# ArquiPlan - Editor de Planos Interactivo

Editor visual de planos arquitectonicos para planificar remodelaciones de apartamentos. Aplicacion 100% client-side en un solo archivo HTML, sin dependencias externas.

## Inicio Rapido

```bash
open plano-editor.html
```

Solo abrir el archivo en cualquier navegador moderno (Chrome, Firefox, Safari, Edge).

## Estructura del Proyecto

```
arquiplan/
  plano-editor.html      # Aplicacion completa (HTML + CSS + JS)
  plano-original.xml      # Plano SVG fuente del apartamento (85.85 m2)
  README.md               # Este archivo
```

## Arquitectura Tecnica

### Principio: Bloques Agrupados

Cada elemento del plano es un `<g>` SVG que contiene todos sus sub-elementos (rectangulo, texto, decoraciones). Mover el grupo mueve todo junto, eliminando desalineaciones.

```
<g class="room-block" data-type="room" data-name="Cocina" data-rot="0" transform="translate(550,750)">
  <rect width="200" height="180" fill="#ffebee" .../>    <!-- rectangulo -->
  <text class="room-label" x="100" y="90">Cocina</text>  <!-- nombre centrado -->
</g>
```

El atributo `data-rot` almacena el angulo de rotacion del bloque. Cuando es distinto de 0, el transform incluye `rotate(deg, cx, cy)` centrado en el bloque.

### Funciones Factory

Cada tipo de elemento tiene su funcion constructora:

| Funcion      | Tipo       | Descripcion                              |
|------------- |----------- |----------------------------------------- |
| `mkRoom()`   | room       | Habitacion con nombre y subtitulo        |
| `mkWall()`   | wall       | Pared con patron de tramado              |
| `mkDoor()`   | door       | Puerta con arco de apertura              |
| `mkWindow()` | window     | Ventana con linea central                |
| `mkFurn()`   | furniture  | Mueble con etiqueta                      |
| `mkLabel()`  | label      | Texto libre                              |

### Capas SVG (orden de renderizado)

```
gridG    -> Grilla de medidas (toggleable)
blocksG  -> Todos los bloques del plano (habitaciones, paredes, muebles...)
dimG     -> Cotas y dimensiones automaticas
snapG    -> Guias de snap (verdes, temporales)
liveG    -> Dimensiones en vivo del elemento seleccionado
selG     -> Handles de seleccion y redimensionado
```

### Sistema de Escala

- ViewBox SVG: 800 x 1000 px
- Rectangulo exterior: 700 x 850 px
- Escala por defecto: 700px = 8.50m ancho, 850px = 10.10m alto
- Funciones: `px2mx()`, `px2my()` (px a metros), `m2px()`, `m2py()` (metros a px)
- Configurable desde la barra superior

### Smart Snap

Compara bordes (left, right, top, bottom, center) del bloque arrastrado contra todos los demas. Si un borde esta a menos de 8px de otro, lo alinea y muestra guia verde. Excluye todos los bloques seleccionados de los objetivos de snap.

Para puertas y ventanas, ademas aplica **Wall Snap**: detecta si el bloque esta cerca del borde de una habitacion (threshold 15px) y lo adhiere automaticamente al borde mas cercano.

### Sistema de Seleccion (v4)

- **Seleccion simple**: Click en un bloque lo selecciona
- **Multi-seleccion**: Shift+click agrega/quita bloques de la seleccion
- **Lasso**: Click y arrastrar en area vacia dibuja un rectangulo de seleccion; todos los bloques que intersectan se seleccionan
- **Estado**: `S.sels[]` array de elementos seleccionados. `S.sel` es getter de conveniencia que retorna `S.sels[0]`
- **Bounding box**: Cuando hay multiples seleccionados, se muestra un bounding box combinado con borde azul discontinuo

### Sistema de Rotacion (v4)

- **Handle circular**: Al seleccionar un bloque aparece un circulo verde arriba del bloque. Arrastrar el handle rota el bloque
- **Tecla R**: Rota 90 grados el bloque seleccionado (si no hay seleccion, activa la herramienta de habitacion)
- **Menu contextual**: Opcion "Rotar 90" disponible al hacer click derecho
- **Almacenamiento**: Angulo guardado en atributo `data-rot`, persistido via `blocksG.innerHTML`

### Clipboard (v4)

- **Ctrl+C**: Copia los bloques seleccionados (clona los nodos)
- **Ctrl+V**: Pega los bloques copiados con un offset de 20px para evitar superposicion
- Compatible con multi-seleccion (copia/pega todos los seleccionados)

### Drag & Drop (v4)

Los muebles en la biblioteca del panel derecho son arrastrables. Se pueden arrastrar directamente al canvas del plano usando HTML5 Drag & Drop API. El mueble se coloca en la posicion exacta donde se suelta.

### Edicion Inline (v4)

Doble click en cualquier bloque abre un campo de texto superpuesto para editar el nombre directamente en el plano, sin necesidad de usar el panel de propiedades. El input se posiciona sobre el texto del bloque usando coordenadas de pantalla calculadas desde las coordenadas SVG.

### Capas (Layers)

Sistema de capas para ocultar/mostrar grupos de elementos del plano:
- **Habitaciones**: rooms y perimetro
- **Paredes**: muros y tabiques
- **Puertas/Ventanas**: puertas y ventanas
- **Muebles**: todo el mobiliario
- **Textos**: etiquetas libres
- **Cotas**: dimensiones automaticas
- **Escala**: barra de escala grafica

Panel "Capas" en el sidebar derecho con checkboxes individuales. El estado se reaplica despues de undo/redo.

### Paredes Inteligentes

Al redimensionar una habitacion, si otra habitacion comparte el borde que se esta moviendo (threshold 5px), la habitacion adyacente se redimensiona automaticamente. Funciona en las 4 direcciones (norte, sur, este, oeste). Las posiciones originales de las habitaciones adyacentes se almacenan en `S.rszAdj[]` al iniciar el resize.

### Persistencia

- `localStorage` con clave `plano-v4`
- Guarda: `blocksG.innerHTML` + escala (WM, HM)
- Boton "Reset" para volver al plano original

## Funcionalidades Actuales (v4)

### Herramientas (Atajos)

| Tecla | Herramienta | Descripcion                          |
|------ |------------ |------------------------------------- |
| V     | Seleccionar | Click para seleccionar, arrastrar para mover |
| H     | Mover vista | Pan del canvas (tambien Alt+click o boton central) |
| R     | Habitacion / Rotar | Sin seleccion: dibujar habitacion. Con seleccion: rotar 90 grados |
| W     | Pared       | Dibujar pared con tramado            |
| D     | Puerta      | Dibujar puerta con arco              |
| N     | Ventana     | Dibujar ventana                      |
| T     | Texto       | Agregar etiqueta de texto            |
| M     | Cota        | Medir distancia entre 2 puntos       |
| E     | Borrador    | Click para eliminar elementos        |

### Atajos Globales

| Atajo         | Accion                        |
|-------------- |------------------------------ |
| Ctrl+Z        | Deshacer                      |
| Ctrl+Y        | Rehacer                       |
| Ctrl+C        | Copiar seleccion al clipboard |
| Ctrl+V        | Pegar desde clipboard         |
| Ctrl+D        | Duplicar seleccion            |
| R             | Rotar seleccion 90 grados     |
| Shift+Click   | Agregar/quitar de seleccion   |
| Doble click   | Editar nombre inline          |
| Delete        | Eliminar seleccion            |
| Escape        | Deseleccionar                 |
| Scroll        | Zoom in/out                   |
| Alt+Click     | Pan del canvas                |
| Click der.    | Menu contextual               |
| Drag (vacio)  | Lasso de seleccion            |

### Panel Derecho

- **Capas**: Toggles para ocultar/mostrar cada tipo de elemento.
- **Areas**: Lista de habitaciones con area individual y total automatico.
- **Propiedades**: Ancho, alto, area en metros. Posicion X/Y editable. Color de relleno. Nombre editable.
- **Biblioteca de muebles**: 18 muebles organizados en 5 categorias. Arrastrables directamente al canvas.
- **Colores rapidos**: Paleta de 16 colores para aplicar al elemento seleccionado.

### Biblioteca de Muebles (por categoria)

| Categoria   | Muebles                                    |
|------------ |------------------------------------------- |
| Sala        | Sofa, Mesa, Silla, TV                      |
| Dormitorio  | Cama, Armario, Mesa de noche, Escritorio   |
| Baño        | Inodoro, Ducha, Lavabo, Bañera             |
| Cocina      | Estufa, Nevera, Lavadora                   |
| Simbolos    | Escalera, Columna, Flecha Norte            |

Cada mueble tiene forma SVG real (paths/ellipses/circles) en vez de solo rectangulos. Las categorias son colapsables.

### Menu Contextual (Click derecho)

- Traer al frente
- Enviar atras
- Duplicar (Ctrl+D)
- Rotar 90 grados
- Eliminar (Del)

### Exportacion

- **SVG**: Exporta el plano limpio (sin handles ni guias)
- **PNG**: Exporta a 1600x2000px
- **PDF**: Abre vista de impresion con escala configurable (1:50, 1:100), tabla de areas, y metadatos. Usar "Guardar como PDF" del navegador.

### Restriccion de Movimiento

Mantener **Shift** presionado mientras se arrastra un elemento restringe el movimiento a un solo eje (X o Y), el que tenga mayor desplazamiento. Util para alinear elementos con precision.

### Estilo Visual Arquitectonico (v6)

- Paredes gruesas y oscuras (#3a3a3a) con sombra proyectada
- Colores de habitaciones suaves y calidos (paleta arquitectonica)
- Patron de piso sutil (grilla de azulejos) en habitaciones
- Perimetro con borde grueso (8px) y sombra exterior
- Puertas y muros con aspecto profesional oscuro
- Fondo calido off-white (#f0ede6)

### Imagen de Fondo (v7)

Importar una imagen (foto o scan de un plano existente) como fondo semitransparente para calcar encima:
- Boton "Fondo" en la barra superior abre el selector de archivos
- La imagen se posiciona dentro del area del plano (respetando aspect ratio)
- Control de opacidad deslizable (5% a 100%) en el panel derecho
- Boton para eliminar la imagen cuando ya no se necesita
- Se persiste en guardado rapido y en proyectos

### Multiples Proyectos (v7)

Sistema de proyectos con nombre para gestionar distintos planos:
- **Guardar**: Pide nombre y guarda el estado completo (bloques, escala, imagen de fondo)
- **Proyectos**: Modal con lista de todos los proyectos guardados
- Click en nombre para cargar, boton X para eliminar
- Indicador visual del proyecto activo
- Boton "Nuevo proyecto" para empezar desde cero
- Almacenados en localStorage (`ap-projects`)

### Botones de Alineacion (v7)

Seccion "Alinear" aparece automaticamente al multi-seleccionar (2+ elementos):
- **Alinear**: Izquierda, Centro H, Derecha, Arriba, Centro V, Abajo
- **Distribuir**: Horizontal y Vertical (espacio igual entre elementos)

### Pisos Contextuales (v7)

Las habitaciones muestran patrones de piso automaticos segun su nombre:
- **Bano**: Azulejos pequenos (`fBath`)
- **Cocina**: Patron de tablero sutil (`fKitchen`)
- **Habitacion/Sala/Dormitorio/Comedor**: Tablones de madera (`fWood`)
- **Otros**: Grilla de azulejos generica (`ftile`)

### Muebles Mejorados (v7)

- **Cama**: Cabecera, almohadas, linea de sabana, pliegue de cobija
- **Sofa**: Respaldo, brazos laterales, 3 cojines separados

### Modo Claro/Oscuro (v7)

Toggle en la esquina superior derecha (icono de luna). El modo claro usa:
- Fondos claros (#f5f5f5, #fff), textos oscuros (#333)
- Bordes suaves (#ddd), acentos azules (#3a7bf7)
- Persistido en localStorage (`ap-theme`)

### Reglas (Rulers) (v7)

Reglas de medicion en los bordes del canvas:
- **Horizontal**: Arriba del canvas, marcas cada 0.25m, etiquetas cada 1m
- **Vertical**: Izquierda del canvas, etiquetas rotadas
- Se actualizan con zoom/pan para reflejar metros reales
- Toggle "Reglas" en la barra superior para ocultar/mostrar
- Renderizadas con HTML5 Canvas para nitidez en cualquier DPI

### Asistente de Nuevo Proyecto (v8)

Boton "Nuevo" en la barra superior abre un wizard visual con:

**Plantillas predefinidas (6 opciones):**

| Plantilla       | Area    | Ambientes | Layout                                     |
|---------------- |-------- |---------- |------------------------------------------- |
| Estudio         | ~35 m²  | 4         | Sala/Dormitorio, Cocina, Bano, Lavadero    |
| 1 Habitacion    | ~50 m²  | 5         | Dormitorio, Sala-Comedor, Cocina, Bano, Pasillo |
| 2 Habitaciones  | ~70 m²  | 5         | 2 Dormitorios, Sala-Comedor, Bano, Cocina  |
| 3 Habitaciones  | ~85 m²  | 9         | Layout original del plano base             |
| Casa Pequena    | ~100 m² | 9         | 3 Hab, 2 Banos, Sala-Comedor, Cocina, Patio |
| Lienzo Vacio    | Variable| 0         | Solo perimetro, sin divisiones internas    |

Cada plantilla incluye habitaciones con colores, puertas, ventanas y puerta de entrada.

**Lienzo personalizado:**
- Inputs de ancho y alto en metros (2m a 30m)
- Genera perimetro vacio con la escala configurada
- Ideal para calcar sobre una imagen de fondo importada

### Manual de Uso Integrado (v8)

Boton "?" en la barra superior abre un manual interactivo con secciones colapsables (acordeon):

- **Primeros Pasos** - Guia paso a paso para principiantes (6 pasos numerados)
- **Herramientas** - Tabla con todas las herramientas, tecla de atajo y descripcion
- **Atajos de Teclado** - Referencia completa de todos los atajos
- **Panel Derecho** - Explicacion de cada seccion del panel (propiedades, capas, muebles, etc.)
- **Proyectos y Exportacion** - Como guardar, cargar y exportar planos
- **Funciones Especiales** - Snap, paredes inteligentes, rotacion, lasso, imagen de fondo, etc.

Se muestra automaticamente la primera vez que un usuario abre la aplicacion. Despues solo se abre manualmente con el boton "?". Estado persistido en localStorage (`ap-help-seen`).

---

## Historial de Versiones

### v8 (actual)
- Asistente de nuevo proyecto con 6 plantillas visuales predefinidas
- Lienzo personalizado con dimensiones a medida
- Boton "Nuevo" en barra superior para acceso rapido
- Manual de uso integrado con guia paso a paso y referencia de atajos
- Auto-muestra en primera visita

### v7
- Importar imagen de fondo con control de opacidad para calcar planos
- Multiples proyectos guardados con nombres (guardar/cargar/eliminar)
- Botones de alineacion: izquierda, centro, derecha, arriba, abajo, distribuir
- Pisos contextuales: patrones automaticos segun tipo de habitacion (bano, cocina, madera)
- Muebles mejorados: cama con almohadas/cabecera, sofa con cojines
- Modo claro/oscuro toggle con persistencia
- Reglas (rulers) en bordes del canvas con medidas en metros

### v6
- Estilo visual arquitectonico: paredes gruesas oscuras, colores suaves, patrones de piso, sombras
- Restriccion de movimiento con Shift (solo X o solo Y)
- Exportacion a PDF con escala imprimible (1:50, 1:100) y tabla de areas

### v5
- Capas (layers) con toggles por tipo de elemento
- Paredes inteligentes: redimensionar una habitacion ajusta las adyacentes
- 10 nuevos muebles: silla, TV, escritorio, armario, mesa de noche, lavadora, bañera, escalera, columna, flecha norte
- Biblioteca categorizada: Sala, Dormitorio, Baño, Cocina, Simbolos (colapsables)
- Formas SVG reales para todos los muebles
- Calculo automatico de area total por habitacion
- Escala grafica visual con regla dibujada

### v4
- Multi-seleccion con Shift+click y lasso
- Rotacion de elementos con handle circular y tecla R
- Snap a paredes para puertas y ventanas
- Copy/Paste (Ctrl+C/V)
- Drag & drop real desde la biblioteca de muebles
- Edicion inline con doble click

### v3
- Arquitectura de bloques agrupados (`<g>`)
- Smart snap con guias verdes
- Biblioteca de muebles (8 tipos)
- Panel de propiedades con medidas en metros
- Exportacion SVG/PNG
- Undo/Redo (60 estados)
- Menu contextual
- Dark theme UI

---

## Mejoras Planificadas

### Prioridad Alta - Interaccion

- [x] **Multi-seleccion** - Shift+click o lasso para seleccionar varios bloques
- [x] **Rotacion** de elementos con handle circular o tecla R
- [x] **Snap a paredes** - puertas y ventanas se adhieren al borde de habitaciones
- [x] **Copy/Paste** (Ctrl+C/V)
- [x] **Drag & drop** real desde la biblioteca de muebles
- [x] **Edicion inline** - doble click en texto para editar sin modal
- [x] **Restriccion de movimiento** - Shift para mover solo en X o solo en Y

### Prioridad Alta - Arquitectura

- [x] **Capas** (layers) - ocultar/mostrar grupos (estructura, muebles, cotas)
- [x] **Paredes inteligentes** - mover pared compartida redimensiona ambas habitaciones
- [x] **Calculo automatico de area total** sumando habitaciones
- [x] **Escala grafica** visual en el plano (regla dibujada)

### Prioridad Media - Biblioteca

- [x] **Mas muebles** - escritorio, silla, TV, lavadora, banera, armario, mesa de noche
- [x] **Formas SVG reales** en vez de solo rectangulos con texto
- [x] **Categorias** en la biblioteca (bano, cocina, dormitorio, sala)
- [x] **Simbolos arquitectonicos** correctos (escaleras, columna, flecha norte)

### Prioridad Media - Exportacion

- [x] **Exportar a PDF** con escala imprimible (1:50, 1:100)
- [x] **Tabla de areas** - resumen de m2 por habitacion
- [x] **Multiples proyectos** guardados con nombres
- [x] **Importar imagen de fondo** para calcar encima de un plano real

### Prioridad Baja - Visual/UX

- [x] **Modo claro/oscuro** toggle
- [ ] **Minimap** - vista miniatura para navegacion rapida
- [ ] **Zoom a seleccion**
- [x] **Reglas** (rulers) en los bordes del canvas
- [ ] **Guias personalizadas** arrastrables desde las reglas
- [ ] **Pisos/Niveles** - tabs para planta baja, primer piso, etc.

### Prioridad Baja - Tecnico

- [ ] **Touch support** - tablets con pinch-to-zoom
- [x] **Botones de alineacion** - alinear izquierda, centro, distribuir uniformemente
- [ ] **Historial visual** - timeline de cambios con preview
- [ ] **Atajos configurables**
- [ ] **Muebles favoritos/personalizados**

---

## Datos del Plano Base

El plano original (`plano-original.xml`) representa un apartamento de **85.85 m2** con:

| Habitacion              | Posicion (px) | Tamano (px)  |
|------------------------ |-------------- |------------- |
| Habitacion 2 (Estudio)  | 50, 80        | 350 x 300    |
| Habitacion 1 (Principal)| 400, 80       | 350 x 300    |
| Bano Privado            | 550, 380      | 200 x 120    |
| Bano Social             | 550, 500      | 200 x 120    |
| Patio / Lavadero        | 550, 620      | 200 x 130    |
| Habitacion 3 (Belen)    | 50, 380       | 280 x 250    |
| Pasillo                 | 370, 380      | 180 x 250    |
| Sala - Comedor          | 50, 630       | 500 x 300    |
| Cocina                  | 550, 750      | 200 x 180    |

Elementos adicionales: 1 muro closet, 4 ventanas, 1 puerta entrada, 1 isla de cocina.

## Notas de Desarrollo

- **Todo en un archivo**: La app es un solo HTML con CSS y JS embebido. No requiere build, bundler, ni servidor.
- **Sin dependencias**: Cero librerias externas. SVG nativo + vanilla JS.
- **Undo/Redo**: Guarda snapshots de `blocksG.innerHTML` (max 60 estados).
- **Multi-seleccion**: `S.sels[]` array con getter `S.sel` para compatibilidad hacia atras.
- **Rotacion**: Atributo `data-rot` en cada `<g>`, aplicado como `rotate()` en el transform SVG.
- **Wall Snap**: Puertas y ventanas detectan bordes de habitaciones cercanas (15px) y se adhieren.
- **Rendimiento**: Para planos grandes considerar virtualizar la grilla y limitar snap a elementos cercanos.
- **Navegador**: Probado en Chrome/Safari. Requiere ES6+.

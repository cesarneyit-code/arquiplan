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
<g class="room-block" data-type="room" data-name="Cocina" transform="translate(550,750)">
  <rect width="200" height="180" fill="#ffebee" .../>    <!-- rectangulo -->
  <text class="room-label" x="100" y="90">Cocina</text>  <!-- nombre centrado -->
</g>
```

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

Compara bordes (left, right, top, bottom, center) del bloque arrastrado contra todos los demas. Si un borde esta a menos de 8px de otro, lo alinea y muestra guia verde.

### Persistencia

- `localStorage` con clave `plano-v3`
- Guarda: `blocksG.innerHTML` + escala (WM, HM)
- Boton "Reset" para volver al plano original

## Funcionalidades Actuales (v3)

### Herramientas (Atajos)

| Tecla | Herramienta | Descripcion                          |
|------ |------------ |------------------------------------- |
| V     | Seleccionar | Click para seleccionar, arrastrar para mover |
| H     | Mover vista | Pan del canvas (tambien Alt+click o boton central) |
| R     | Habitacion  | Dibujar rectangulo de habitacion     |
| W     | Pared       | Dibujar pared con tramado            |
| D     | Puerta      | Dibujar puerta con arco              |
| N     | Ventana     | Dibujar ventana                      |
| T     | Texto       | Agregar etiqueta de texto            |
| M     | Cota        | Medir distancia entre 2 puntos       |
| E     | Borrador    | Click para eliminar elementos        |

### Atajos Globales

| Atajo       | Accion              |
|------------ |-------------------- |
| Ctrl+Z      | Deshacer            |
| Ctrl+Y      | Rehacer             |
| Ctrl+D      | Duplicar seleccion  |
| Delete      | Eliminar seleccion  |
| Escape      | Deseleccionar       |
| Scroll      | Zoom in/out         |
| Alt+Click   | Pan del canvas      |
| Click der.  | Menu contextual     |

### Panel Derecho

- **Propiedades**: Ancho, alto, area en metros. Posicion X/Y editable. Color de relleno. Nombre editable.
- **Biblioteca de muebles**: Sofa, mesa, cama, inodoro, ducha, lavabo, estufa, nevera.
- **Colores rapidos**: Paleta de 16 colores para aplicar al elemento seleccionado.

### Exportacion

- **SVG**: Exporta el plano limpio (sin handles ni guias)
- **PNG**: Exporta a 1600x2000px

---

## Mejoras Planificadas

### Prioridad Alta - Interaccion

- [ ] **Multi-seleccion** - Shift+click o lasso para seleccionar varios bloques
- [ ] **Rotacion** de elementos con handle circular o tecla R
- [ ] **Snap a paredes** - puertas y ventanas se adhieren al borde de habitaciones
- [ ] **Copy/Paste** (Ctrl+C/V)
- [ ] **Drag & drop** real desde la biblioteca de muebles
- [ ] **Edicion inline** - doble click en texto para editar sin modal
- [ ] **Restriccion de movimiento** - Shift para mover solo en X o solo en Y

### Prioridad Alta - Arquitectura

- [ ] **Capas** (layers) - ocultar/mostrar grupos (estructura, muebles, cotas)
- [ ] **Paredes inteligentes** - mover pared compartida redimensiona ambas habitaciones
- [ ] **Calculo automatico de area total** sumando habitaciones
- [ ] **Escala grafica** visual en el plano (regla dibujada)

### Prioridad Media - Biblioteca

- [ ] **Mas muebles** - escritorio, silla, TV, lavadora, banera, armario, mesa de noche
- [ ] **Formas SVG reales** en vez de solo rectangulos con texto
- [ ] **Categorias** en la biblioteca (bano, cocina, dormitorio, sala)
- [ ] **Simbolos arquitectonicos** correctos (escaleras, cortes, etc.)

### Prioridad Media - Exportacion

- [ ] **Exportar a PDF** con escala imprimible (1:50, 1:100)
- [ ] **Tabla de areas** - resumen de m2 por habitacion
- [ ] **Multiples proyectos** guardados con nombres
- [ ] **Importar imagen de fondo** para calcar encima de un plano real

### Prioridad Baja - Visual/UX

- [ ] **Modo claro/oscuro** toggle
- [ ] **Minimap** - vista miniatura para navegacion rapida
- [ ] **Zoom a seleccion**
- [ ] **Reglas** (rulers) en los bordes del canvas
- [ ] **Guias personalizadas** arrastrables desde las reglas
- [ ] **Pisos/Niveles** - tabs para planta baja, primer piso, etc.

### Prioridad Baja - Tecnico

- [ ] **Touch support** - tablets con pinch-to-zoom
- [ ] **Botones de alineacion** - alinear izquierda, centro, distribuir uniformemente
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
- **Rendimiento**: Para planos grandes considerar virtualizar la grilla y limitar snap a elementos cercanos.
- **Navegador**: Probado en Chrome/Safari. Requiere ES6+.

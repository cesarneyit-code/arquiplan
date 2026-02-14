# ArquiPlan - Referencia Tecnica para IA

## Que es ArquiPlan
Editor de planos arquitectonicos 2D en el navegador. Archivo unico HTML (~2500 lineas) sin dependencias. Genera SVG interactivo con habitaciones, puertas, ventanas, muros y muebles.

## Como la IA interactua con ArquiPlan

La IA genera archivos `.arquiplan.json` que el usuario importa. El archivo contiene SVG crudo (innerHTML) de todos los elementos del plano. La IA tambien puede leer un archivo exportado, analizarlo, y devolver uno modificado.

---

## Formato .arquiplan.json

```json
{
  "version": "1.0",
  "name": "Nombre del proyecto",
  "date": "2026-02-14T00:00:00.000Z",
  "data": {
    "blocks": "<SVG innerHTML aqui>",
    "wm": 8.5,
    "hm": 10.1,
    "bgImg": null,
    "bgOp": 0.3
  }
}
```

| Campo | Tipo | Descripcion |
|-------|------|-------------|
| `version` | string | Siempre "1.0" |
| `name` | string | Nombre del proyecto |
| `date` | string | ISO 8601 timestamp |
| `data.blocks` | string | innerHTML del grupo SVG `<g id="blocksG">` |
| `data.wm` | number | Ancho del plano en metros |
| `data.hm` | number | Alto del plano en metros |
| `data.bgImg` | string/null | Data URL de imagen de fondo (base64) |
| `data.bgOp` | number | Opacidad de imagen de fondo (0.05-1.0) |

---

## Sistema de coordenadas

### Canvas SVG
- viewBox: `0 0 800 1000` (ancho 800px, alto 1000px)
- Fondo: rect 800x1000, fill `#f0ede6`
- El area de dibujo es un subrecta centrada dentro del viewBox

### Area de dibujo
Las dimensiones del area de dibujo se calculan a partir de los metros:

```
maxW = 720, maxH = 880
aspectRatio = wm / hm

Si aspectRatio >= maxW/maxH:
  WPx = 720
  HPx = round(720 / aspectRatio)
Sino:
  HPx = 880
  WPx = round(880 * aspectRatio)

OX = round((800 - WPx) / 2)   // margen izquierdo
OY = round((1000 - HPx) / 2)  // margen superior
```

### Conversion metros <-> pixeles
```
pixelX = OX + (metros_x / WM) * WPx
pixelY = OY + (metros_y / HM) * HPx
anchoPixeles = (anchoMetros / WM) * WPx
altoPixeles = (altoMetros / HM) * HPx
```

### Ejemplo para plano 8.5m x 10.1m (default)
```
WPx = 700, HPx = 850
OX = 50, OY = 80
1 metro horizontal = 700/8.5 = 82.35px
1 metro vertical = 850/10.1 = 84.16px
```

### Tabla rapida de escalas comunes

| Metros | WPx | HPx | OX | OY | px/m horiz | px/m vert |
|--------|-----|-----|----|----|-----------|-----------|
| 5.8 x 6.0 | 720 | 745 | 40 | 128 | 124.14 | 124.17 |
| 7.0 x 7.2 | 720 | 741 | 40 | 130 | 102.86 | 102.92 |
| 8.0 x 8.8 | 720 | 792 | 40 | 104 | 90.00 | 90.00 |
| 8.5 x 10.1 | 700 | 850 | 50 | 75 | 82.35 | 84.16 |
| 10.0 x 10.0 | 720 | 720 | 40 | 140 | 72.00 | 72.00 |

---

## Elementos SVG

Todos los elementos van dentro de `data.blocks`. Cada elemento es un `<g>` (grupo SVG) con atributos de datos y un `transform` para posicion.

### Transform
Formato: `translate(x,y)` donde x,y son coordenadas en pixeles del viewBox.

Con rotacion: `translate(x,y) rotate(angulo,cx,cy)` donde cx=ancho/2, cy=alto/2.
Con flip H: agregar `translate(ancho,0) scale(-1,1)` despues.
Con flip V: agregar `translate(0,alto) scale(1,-1)` despues.

---

### 1. HABITACION (Room)

```xml
<g class="room-block" data-type="room" data-name="Sala" transform="translate(50,80)">
  <rect x="0" y="0" width="350" height="300" fill="#f5f2ed" stroke="#3a3a3a" stroke-width="3.5" class="room-rect" filter="url(#roomShd)"/>
  <rect x="0" y="0" width="350" height="300" fill="url(#fWood)" stroke="none" class="room-tile" pointer-events="none"/>
  <text x="175" y="154" text-anchor="middle" font-size="18" fill="#4a4a4a" font-weight="700" font-family="Inter,Arial,sans-serif" class="room-label">Sala</text>
</g>
```

| Atributo | Valor |
|----------|-------|
| class | `room-block` |
| data-type | `room` |
| data-name | Nombre de la habitacion |

**Hijos:**
1. `<rect>` principal: fill=color, stroke=`#3a3a3a`, stroke-width=`3.5`, filter=`url(#roomShd)`
2. `<rect>` patron de piso: fill=`url(#PATRON)`, class=`room-tile`
3. `<text>` nombre: text-anchor=`middle`, class=`room-label`
4. `<text>` sub-etiqueta (opcional): font-size=`11`, fill=`#888`, class=`room-sub`

**Patrones de piso automaticos (segun nombre):**

| Nombre contiene | Patron |
|----------------|--------|
| bano, baño | `url(#fBath)` (azulejos 10x10) |
| cocina | `url(#fKitchen)` (azulejos 20x20) |
| habitacion, sala, dormitorio, comedor | `url(#fWood)` (madera 40x8) |
| otro | `url(#ftile)` (grilla 18x18) |

**Colores tipicos de habitacion:**

| Tipo | Fill | Text color |
|------|------|------------|
| Sala/Comedor | `#f5f2ed` | `#4a4a4a` |
| Dormitorio | `#f2f0eb` | `#4a4a4a` |
| Cocina | `#f5edec` | `#8a3a3a` |
| Bano | `#eaf2f5` | `#3a6b8a` |
| Lavadero/Patio | `#edf5ef` | `#3a7a5a` |
| Pasillo | `#f0ede6` | `#aaa` |

---

### 2. PUERTA (Door)

```xml
<g class="door-block" data-type="door" transform="translate(200,76)">
  <rect x="0" y="0" width="80" height="10" fill="transparent" stroke="none"/>
  <line x1="0" y1="10" x2="80" y2="10" stroke="#333" stroke-width="2" class="door-part"/>
  <path d="M80,10 A80,80 0 0,1 0,-70" fill="none" stroke="#333" stroke-width="0.8" class="door-part"/>
</g>
```

**Horizontal** (puerta en borde superior/inferior, se abre verticalmente):
- Bounding rect: width=W, height=10
- Linea hoja: (0,10) a (W,10)
- Arco 90°: `M{W},10 A{W},{W} 0 0,1 0,{10-W}`

**Vertical** (puerta en borde lateral, se abre horizontalmente):
- Bounding rect: width=10, height=W
- Linea hoja: (10,0) a (10,W)
- Arco 90°: `M10,{W} A{W},{W} 0 0,0 {10+W},0`

Donde W = ancho de puerta en pixeles. Tipico: 55-80px (~0.7-1.0m).

---

### 3. VENTANA (Window)

```xml
<g class="win-block" data-type="window" transform="translate(100,76)">
  <rect x="0" y="0" width="100" height="8" fill="transparent" stroke="none"/>
  <line x1="0" y1="0" x2="100" y2="0" stroke="#333" stroke-width="2" class="win-part"/>
  <line x1="0" y1="8" x2="100" y2="8" stroke="#333" stroke-width="2" class="win-part"/>
  <line x1="0" y1="4" x2="100" y2="4" stroke="#333" stroke-width="0.6" class="win-part"/>
</g>
```

**Horizontal** (ventana en borde superior/inferior):
- Bounding rect: width=W, height=8
- Linea muro 1: (0,0) a (W,0), stroke-width=2
- Linea muro 2: (0,8) a (W,8), stroke-width=2
- Linea vidrio: (0,4) a (W,4), stroke-width=0.6

**Vertical** (ventana en borde lateral):
- Bounding rect: width=8, height=W
- Linea muro 1: (0,0) a (0,W), stroke-width=2
- Linea muro 2: (8,0) a (8,W), stroke-width=2
- Linea vidrio: (4,0) a (4,W), stroke-width=0.6

Tipico: 60-120px (~0.8-1.5m).

---

### 4. MURO/PARED (Wall)

```xml
<g class="wall-block" data-type="wall" data-name="Muro Closet" transform="translate(330,380)">
  <rect x="0" y="0" width="12" height="250" fill="#4a4a4a" stroke="#333" stroke-width="1"/>
  <rect x="0" y="0" width="12" height="250" fill="url(#hatch)" stroke="none"/>
  <text x="6" y="129" text-anchor="middle" font-size="10" fill="#ddd" font-weight="600" font-family="Inter,Arial,sans-serif" class="room-label" transform="rotate(-90,6,129)">Muro Closet</text>
</g>
```

Muros tipicos: 8-15px de grosor (~0.10-0.20m).

---

### 5. PERIMETRO (Boundary)

```xml
<g class="wall-block" data-type="boundary" data-name="Perimetro" transform="translate(50,80)">
  <rect x="0" y="0" width="700" height="850" fill="none" stroke="#2d2d2d" stroke-width="8" filter="url(#planShd)"/>
</g>
```

Siempre posicionado en (OX, OY) con dimensiones WPx x HPx. Debe ser el primer elemento despues del titulo.

---

### 6. MUEBLE (Furniture)

```xml
<g class="furn-block" data-type="furniture" data-name="Cama" data-furn="Cama" transform="translate(150,200)">
  <rect x="0" y="0" width="100" height="130" fill="#f0ece8" stroke="#555" stroke-width="1" rx="2"/>
  <!-- furn-detail elements aqui (varian por tipo) -->
  <text x="50" y="69" text-anchor="middle" font-size="11" fill="#333" font-weight="600" font-family="Inter,Arial,sans-serif" class="room-label">Cama</text>
</g>
```

**IMPORTANTE:** `data-furn` debe coincidir exactamente con el nombre para que el renderizado especial funcione al redimensionar.

---

### 7. ETIQUETA (Label)

```xml
<g class="furn-block" data-type="label" data-name="ENTRADA" transform="translate(300,920)">
  <text x="0" y="0" text-anchor="middle" font-size="12" fill="#3a3a3a" font-weight="600" font-family="Inter,Arial,sans-serif" class="room-label">ENTRADA</text>
</g>
```

---

## Catalogo completo de muebles

| Nombre exacto | W(px) | H(px) | Fill | Categoria | Uso tipico |
|---------------|-------|-------|------|-----------|------------|
| Sofa | 120 | 40 | #f0ece8 | Sala | Sala de estar |
| Mesa | 80 | 60 | #f0ece8 | Sala | Comedor, centro |
| Silla | 35 | 35 | #f0ece8 | Sala | Comedor, escritorio |
| TV | 80 | 8 | #e8e8e8 | Sala | Contra pared |
| Chimenea | 80 | 30 | #f0ece8 | Sala | Contra pared |
| Cama | 100 | 130 | #f0ece8 | Dormitorio | Doble/matrimonial |
| Armario | 100 | 45 | #ede9e5 | Dormitorio | Contra pared |
| Mesa de noche | 35 | 35 | #f0ece8 | Dormitorio | Junto a cama |
| Escritorio | 100 | 55 | #f0ece8 | Dormitorio | Oficina/estudio |
| Inodoro | 35 | 45 | #fff | Bano | Bano |
| Ducha | 60 | 60 | #fff | Bano | Bano |
| Lavabo | 40 | 30 | #fff | Bano | Bano |
| Banera | 60 | 130 | #fff | Bano | Bano grande |
| Estufa | 50 | 50 | #fff | Cocina | Cocina |
| Nevera | 45 | 55 | #fff | Cocina | Cocina |
| Lavadora | 45 | 45 | #f5f5f5 | Cocina | Lavadero/cocina |
| Escalera | 80 | 120 | #fff | Simbolos | Acceso entre pisos |
| Columna | 20 | 20 | #333 | Simbolos | Estructural |
| Flecha Norte | 30 | 40 | none | Simbolos | Orientacion |

Los muebles pueden redimensionarse. Las formas internas se regeneran proporcionalmente.

---

## SVG Defs requeridos

El `data.blocks` NO incluye los `<defs>`. Estos son generados automaticamente por la aplicacion al cargar:

```xml
<defs>
  <filter id="roomShd">...</filter>
  <filter id="planShd">...</filter>
  <pattern id="ftile" width="18" height="18" patternUnits="userSpaceOnUse">
    <rect width="18" height="18" fill="none" stroke="rgba(0,0,0,.03)" stroke-width=".3"/>
  </pattern>
  <pattern id="fBath" width="10" height="10" patternUnits="userSpaceOnUse">
    <rect width="10" height="10" fill="none" stroke="rgba(0,0,0,.06)" stroke-width=".3"/>
  </pattern>
  <pattern id="fKitchen" width="20" height="20" patternUnits="userSpaceOnUse">
    <rect width="20" height="20" fill="none" stroke="rgba(0,0,0,.04)" stroke-width=".4"/>
  </pattern>
  <pattern id="fWood" width="40" height="8" patternUnits="userSpaceOnUse">
    <line x1="0" y1="7.5" x2="40" y2="7.5" stroke="rgba(0,0,0,.05)" stroke-width=".4"/>
    <line x1="20" y1="0" x2="20" y2="8" stroke="rgba(0,0,0,.03)" stroke-width=".2"/>
  </pattern>
  <pattern id="hatch" width="6" height="6" patternUnits="userSpaceOnUse">
    <line x1="0" y1="6" x2="6" y2="0" stroke="#bbb" stroke-width=".5" opacity=".35"/>
  </pattern>
</defs>
```

La IA NO debe incluir estos en `data.blocks`. Son creados por `buildPlan()`.

---

## Guia para generar un plano desde cero

### Paso 1: Definir dimensiones en metros
Elegir `wm` y `hm` (el rectangulo exterior del plano).

### Paso 2: Calcular geometria de pixeles
```
maxW=720, maxH=880, ar=wm/hm
Si ar >= maxW/maxH: WPx=720, HPx=round(720/ar)
Sino: HPx=880, WPx=round(880*ar)
OX = round((800-WPx)/2)
OY = round((1000-HPx)/2)
```

### Paso 3: Funciones de conversion
```
mpx(m) = OX + (m / wm) * WPx      // metros a px posicion X
mpy(m) = OY + (m / hm) * HPx      // metros a px posicion Y
pw(m)  = (m / wm) * WPx            // metros a px ancho
ph(m)  = (m / hm) * HPx            // metros a px alto
```

### Paso 4: Construir blocks (en orden)
1. **Titulo** (label en posicion fija 400,45)
2. **Perimetro** (boundary en OX,OY, dimensiones WPx x HPx)
3. **Habitaciones** (rooms, NO deben solaparse ni salirse del perimetro)
4. **Muros interiores** (walls, divisiones adicionales)
5. **Puertas** (doors, en bordes de habitaciones)
6. **Ventanas** (windows, en bordes exteriores tipicamente)
7. **Muebles** (furniture, dentro de habitaciones)
8. **Etiquetas** (labels, como "ENTRADA")

### Paso 5: Ensamblar JSON
```json
{
  "version": "1.0",
  "name": "Apartamento 2 Habitaciones",
  "date": "2026-02-14T12:00:00.000Z",
  "data": {
    "blocks": "...SVG concatenado aqui...",
    "wm": 8.0,
    "hm": 8.8,
    "bgImg": null,
    "bgOp": 0.3
  }
}
```

---

## Ejemplo completo: Estudio 35m2

Dimensiones: 5.8m x 6.0m

```
Calculo:
ar = 5.8/6.0 = 0.967
maxW/maxH = 720/880 = 0.818
ar > 0.818, entonces WPx=720, HPx=round(720/0.967)=745
OX = round((800-720)/2) = 40
OY = round((1000-745)/2) = 128
```

Layout:
```
+----------------------------------+
|                    |  Cocina     |
|   Sala /           |  2.0x3.0m  |
|   Dormitorio       |------------|
|   3.8 x 6.0m      |  Bano      |
|                    |  2.0x2.0m  |
|                    |------------|
|                    | Lavadero   |
|                    | 2.0x1.0m  |
+------[ENTRADA]----+-----------+
```

JSON generado:
```json
{
  "version": "1.0",
  "name": "Estudio Moderno",
  "date": "2026-02-14T12:00:00.000Z",
  "data": {
    "wm": 5.8,
    "hm": 6.0,
    "blocks": "<g class=\"furn-block\" data-type=\"label\" data-name=\"Estudio - ~35 m2\" transform=\"translate(400,45)\"><text x=\"0\" y=\"0\" text-anchor=\"middle\" font-size=\"22\" fill=\"#2d3436\" font-weight=\"600\" font-family=\"Inter,Arial,sans-serif\" class=\"room-label\">Estudio - ~35 m2</text></g><g class=\"wall-block\" data-type=\"boundary\" data-name=\"Perimetro\" transform=\"translate(40,128)\"><rect x=\"0\" y=\"0\" width=\"720\" height=\"745\" fill=\"none\" stroke=\"#2d2d2d\" stroke-width=\"8\" filter=\"url(#planShd)\"/></g><g class=\"room-block\" data-type=\"room\" data-name=\"Sala / Dormitorio\" transform=\"translate(40,128)\"><rect x=\"0\" y=\"0\" width=\"472\" height=\"745\" fill=\"#f5f2ed\" stroke=\"#3a3a3a\" stroke-width=\"3.5\" class=\"room-rect\" filter=\"url(#roomShd)\"/><rect x=\"0\" y=\"0\" width=\"472\" height=\"745\" fill=\"url(#fWood)\" stroke=\"none\" class=\"room-tile\" pointer-events=\"none\"/><text x=\"236\" y=\"367\" text-anchor=\"middle\" font-size=\"18\" fill=\"#4a4a4a\" font-weight=\"700\" font-family=\"Inter,Arial,sans-serif\" class=\"room-label\">Sala / Dormitorio</text></g><g class=\"room-block\" data-type=\"room\" data-name=\"Cocina\" transform=\"translate(512,128)\"><rect x=\"0\" y=\"0\" width=\"248\" height=\"373\" fill=\"#f5edec\" stroke=\"#3a3a3a\" stroke-width=\"3.5\" class=\"room-rect\" filter=\"url(#roomShd)\"/><rect x=\"0\" y=\"0\" width=\"248\" height=\"373\" fill=\"url(#fKitchen)\" stroke=\"none\" class=\"room-tile\" pointer-events=\"none\"/><text x=\"124\" y=\"181\" text-anchor=\"middle\" font-size=\"16\" fill=\"#8a3a3a\" font-weight=\"700\" font-family=\"Inter,Arial,sans-serif\" class=\"room-label\">Cocina</text></g><g class=\"room-block\" data-type=\"room\" data-name=\"Bano\" transform=\"translate(512,501)\"><rect x=\"0\" y=\"0\" width=\"248\" height=\"248\" fill=\"#eaf2f5\" stroke=\"#3a3a3a\" stroke-width=\"3.5\" class=\"room-rect\" filter=\"url(#roomShd)\"/><rect x=\"0\" y=\"0\" width=\"248\" height=\"248\" fill=\"url(#fBath)\" stroke=\"none\" class=\"room-tile\" pointer-events=\"none\"/><text x=\"124\" y=\"128\" text-anchor=\"middle\" font-size=\"16\" fill=\"#3a6b8a\" font-weight=\"700\" font-family=\"Inter,Arial,sans-serif\" class=\"room-label\">Bano</text></g><g class=\"room-block\" data-type=\"room\" data-name=\"Lavadero\" transform=\"translate(512,749)\"><rect x=\"0\" y=\"0\" width=\"248\" height=\"124\" fill=\"#edf5ef\" stroke=\"#3a3a3a\" stroke-width=\"3.5\" class=\"room-rect\" filter=\"url(#roomShd)\"/><rect x=\"0\" y=\"0\" width=\"248\" height=\"124\" fill=\"url(#ftile)\" stroke=\"none\" class=\"room-tile\" pointer-events=\"none\"/><text x=\"124\" y=\"66\" text-anchor=\"middle\" font-size=\"14\" fill=\"#3a7a5a\" font-weight=\"700\" font-family=\"Inter,Arial,sans-serif\" class=\"room-label\">Lavadero</text></g><g class=\"win-block\" data-type=\"window\" transform=\"translate(102,124)\"><rect x=\"0\" y=\"0\" width=\"149\" height=\"8\" fill=\"transparent\" stroke=\"none\"/><line x1=\"0\" y1=\"0\" x2=\"149\" y2=\"0\" stroke=\"#333\" stroke-width=\"2\" class=\"win-part\"/><line x1=\"0\" y1=\"8\" x2=\"149\" y2=\"8\" stroke=\"#333\" stroke-width=\"2\" class=\"win-part\"/><line x1=\"0\" y1=\"4\" x2=\"149\" y2=\"4\" stroke=\"#333\" stroke-width=\"0.6\" class=\"win-part\"/></g><g class=\"win-block\" data-type=\"window\" transform=\"translate(326,124)\"><rect x=\"0\" y=\"0\" width=\"99\" height=\"8\" fill=\"transparent\" stroke=\"none\"/><line x1=\"0\" y1=\"0\" x2=\"99\" y2=\"0\" stroke=\"#333\" stroke-width=\"2\" class=\"win-part\"/><line x1=\"0\" y1=\"8\" x2=\"99\" y2=\"8\" stroke=\"#333\" stroke-width=\"2\" class=\"win-part\"/><line x1=\"0\" y1=\"4\" x2=\"99\" y2=\"4\" stroke=\"#333\" stroke-width=\"0.6\" class=\"win-part\"/></g><g class=\"door-block\" data-type=\"door\" transform=\"translate(506,562)\"><rect x=\"0\" y=\"0\" width=\"10\" height=\"87\" fill=\"transparent\" stroke=\"none\"/><line x1=\"10\" y1=\"0\" x2=\"10\" y2=\"87\" stroke=\"#333\" stroke-width=\"2\" class=\"door-part\"/><path d=\"M10,87 A87,87 0 0,0 97,0\" fill=\"none\" stroke=\"#333\" stroke-width=\"0.8\" class=\"door-part\"/></g><g class=\"door-block\" data-type=\"door\" transform=\"translate(226,868)\"><rect x=\"0\" y=\"0\" width=\"99\" height=\"10\" fill=\"transparent\" stroke=\"none\"/><line x1=\"0\" y1=\"10\" x2=\"99\" y2=\"10\" stroke=\"#333\" stroke-width=\"2\" class=\"door-part\"/><path d=\"M99,10 A99,99 0 0,1 0,-89\" fill=\"none\" stroke=\"#333\" stroke-width=\"0.8\" class=\"door-part\"/></g><g class=\"furn-block\" data-type=\"label\" data-name=\"ENTRADA\" transform=\"translate(275,888)\"><text x=\"0\" y=\"0\" text-anchor=\"middle\" font-size=\"12\" fill=\"#3a3a3a\" font-weight=\"600\" font-family=\"Inter,Arial,sans-serif\" class=\"room-label\">ENTRADA</text></g>",
    "bgImg": null,
    "bgOp": 0.3
  }
}
```

---

## Reglas de diseno de planos

### Distribucion de habitaciones
- Las habitaciones deben cubrir todo el area del perimetro sin solaparse
- Los bordes de habitaciones adyacentes deben coincidir exactamente (mismo valor de pixel)
- Dejar espacio para pasillos si es necesario (minimo 0.9m de ancho)

### Ubicacion de puertas
- Puertas se colocan en los bordes compartidos entre habitaciones
- Puertas exteriores (entrada) se colocan en el borde del perimetro
- Ancho tipico: 0.7-0.9m (puertas interiores), 0.8-1.0m (puerta principal)
- La puerta horizontal va en bordes superior/inferior, vertical en bordes laterales
- Posicionar la puerta SOBRE el borde (y del borde - 5px para horizontal)

### Ubicacion de ventanas
- Ventanas tipicamente van en bordes exteriores (coincidentes con el perimetro)
- Ancho tipico: 0.8-1.5m
- Posicionar la ventana SOBRE el borde (y del borde - 4px para horizontal)

### Ubicacion de muebles
- Muebles van DENTRO de las habitaciones correspondientes
- Respetar espacio de circulacion (no bloquear puertas)
- Camas: contra una pared, con mesitas de noche a los lados
- Sofa: frente al TV, dejando espacio
- Inodoro: contra una pared, con espacio frontal
- Lavabo: contra pared, puede ir junto al inodoro
- Ducha/banera: en esquina o contra pared
- Estufa/nevera: contra pared de cocina, con espacio entre ellos
- Mesa comedor: centro del area de comedor

### Proporciones recomendadas

| Tipo de espacio | Area minima | Area recomendada |
|-----------------|-------------|------------------|
| Dormitorio principal | 9 m2 | 12-15 m2 |
| Dormitorio secundario | 7 m2 | 9-12 m2 |
| Bano completo | 3.5 m2 | 4.5-6 m2 |
| Medio bano | 1.5 m2 | 2-3 m2 |
| Cocina | 5 m2 | 7-10 m2 |
| Sala-Comedor | 12 m2 | 18-25 m2 |
| Lavadero | 2 m2 | 3-4 m2 |
| Pasillo | 0.9m ancho min | 1.0-1.2m ancho |
| Estudio/Oficina | 4 m2 | 6-9 m2 |

---

## Capacidades de analisis

Al recibir un `.arquiplan.json` exportado, la IA puede:

1. **Parsear el SVG** - Extraer todos los `<g>` con sus data-type y data-name
2. **Calcular areas** - Leer width/height de cada room rect, convertir a metros
3. **Verificar distribucion** - Detectar solapamientos o espacios vacios
4. **Contar muebles** - Listar todos los furniture por habitacion
5. **Sugerir mejoras** - Ergonomia, feng shui, normativas
6. **Proponer decoracion** - Agregar muebles faltantes, reposicionar existentes
7. **Generar variantes** - Crear JSON modificado con cambios propuestos

### Como parsear el blocks
El campo `data.blocks` es HTML/SVG crudo. Para analizarlo:
1. Buscar todos los `<g` con regex o parser XML
2. Extraer `data-type`, `data-name`, `transform`
3. Extraer width/height del primer `<rect>` hijo
4. Convertir pixeles a metros con las formulas de conversion

---

## Formato de plantilla (para generar programaticamente)

Si la IA quiere definir un plano de forma estructurada antes de generar el SVG, puede usar este formato intermedio:

```json
{
  "wm": 8.0,
  "hm": 8.8,
  "title": "Apartamento 2 Habitaciones - ~70 m2",
  "rooms": [
    {"x": 0, "y": 0, "w": 4.5, "h": 4.0, "f": "#f5f2ed", "n": "Sala-Comedor"},
    {"x": 4.5, "y": 0, "w": 3.5, "h": 4.0, "f": "#f2f0eb", "n": "Habitacion 1"},
    {"x": 0, "y": 4.0, "w": 4.0, "h": 4.8, "f": "#f2f0eb", "n": "Habitacion 2"},
    {"x": 4.0, "y": 4.0, "w": 2.0, "h": 2.5, "f": "#eaf2f5", "n": "Bano"},
    {"x": 4.0, "y": 6.5, "w": 4.0, "h": 2.3, "f": "#f5edec", "n": "Cocina"},
    {"x": 6.0, "y": 4.0, "w": 2.0, "h": 2.5, "f": "#edf5ef", "n": "Lavadero"}
  ],
  "wins": [
    {"x": 0.5, "y": 0, "w": 1.2, "hz": true},
    {"x": 5.0, "y": 0, "w": 1.0, "hz": true}
  ],
  "doors": [
    {"x": 4.4, "y": 1.5, "w": 0.8, "hz": false},
    {"x": 1.0, "y": 3.95, "w": 0.8, "hz": true},
    {"x": 4.5, "y": 4.5, "w": 0.7, "hz": false}
  ],
  "ent": {"x": 2.5, "y": 8.75, "w": 0.9, "hz": true},
  "furniture": [
    {"name": "Sofa", "room": "Sala-Comedor", "x": 0.5, "y": 1.0, "w": 120, "h": 40},
    {"name": "Mesa", "room": "Sala-Comedor", "x": 1.5, "y": 2.5, "w": 80, "h": 60},
    {"name": "Cama", "room": "Habitacion 1", "x": 5.0, "y": 0.3, "w": 100, "h": 130},
    {"name": "Cama", "room": "Habitacion 2", "x": 0.5, "y": 4.5, "w": 100, "h": 130},
    {"name": "Inodoro", "room": "Bano", "x": 4.3, "y": 4.3, "w": 35, "h": 45},
    {"name": "Lavabo", "room": "Bano", "x": 5.2, "y": 4.3, "w": 40, "h": 30},
    {"name": "Estufa", "room": "Cocina", "x": 4.3, "y": 7.0, "w": 50, "h": 50},
    {"name": "Nevera", "room": "Cocina", "x": 5.5, "y": 6.8, "w": 45, "h": 55}
  ]
}
```

La IA puede definir el plano en este formato legible y luego convertirlo a SVG blocks siguiendo las reglas de conversion.

---

## Notas importantes

1. **Todas las coordenadas en `data.blocks` son en pixeles** del viewBox 800x1000
2. **Las comillas dentro del SVG deben escaparse** como `\"` en el JSON
3. **El orden de los elementos importa**: elementos posteriores se dibujan encima
4. **No incluir `<defs>`, filtros ni patrones** en blocks - la app los genera
5. **data-furn debe coincidir con nombres del catalogo** para renderizado especial
6. **font-family siempre**: `Inter,Arial,sans-serif`
7. **Los muebles se renderizan completos** solo al cargar. En el JSON solo necesitan el rect base + text label. La app regenera las formas internas (furn-detail) al redimensionar
8. **Para muebles custom** (no del catalogo): usar cualquier nombre en data-name/data-furn. Se renderizara como rectangulo con etiqueta

# Bitácora del proyecto — Tablero de ventas en vivo

Documento de trabajo. Registra qué se construyó, por qué, y cómo mantenerlo.
Escrito para alguien de marketing: si aparece una palabra de oficio, va
explicada al lado entre paréntesis.

---

## 1. Objetivo

Una pantalla para proyectar en la sala de la tienda que muestre las ventas
**en vivo**: cuántas van hoy, cuánto se facturó, qué se vende y dónde.

Condición de fondo: el tablero se juzga en su peor momento (cuando hay público
mirando y entran muchas ventas de golpe), no en su promedio.

---

## 2. La fuente de datos

Base **Supabase autoalojada** (una base de datos con una interfaz web para
consultarla; "autoalojada" = corre en un servidor propio, no en supabase.com).

| Dato | Valor |
|---|---|
| Dirección | `https://db.lanimalier.cl` |
| Esquema (grupo de tablas) | `lanimalier` |
| Tabla | `ventas` |
| Clave | anónima, **solo lectura** |
| Acceso a otras tablas (`orders`) | denegado con esta clave |

### Lo que se encontró al explorar

- ~270.000 filas y creciendo (entran ventas nuevas cada minuto).
- La venta más antigua es del **24-may-2026**.
- **Todas las filas son de prueba** (`simulado: true`). Sirve para construir y
  demostrar el tablero, pero no son ventas reales.
- El profesor controla el flujo: puede dejarlo en cero, en ritmo normal, o
  soltar 2.000 ventas en un segundo, sin avisar.

### Columnas que usa el tablero

| Campo | Qué contiene | Uso en el tablero |
|---|---|---|
| `ocurrido_en` | Fecha y hora (en horario universal UTC; Chile va 4 h atrás) | Agrupar "hoy" y "por hora" |
| `total` | Lo que pagó el cliente, en pesos chilenos enteros | Facturación y ticket promedio |
| `unidades` | Artículos en la venta | Unidades vendidas |
| `ciudad` | Comuna del cliente | Ranking de comunas |
| `fuente` | De dónde vino (orgánico, instagram, google ads…) | Reparto por fuente |
| `piezas` | Lista de productos del pedido (texto estructurado JSON) | Nombre del producto en la lista de ventas |
| `id` | Número único y creciente de cada venta | Control para no contar dos veces la misma |

---

## 3. Cómo funciona (arquitectura)

Es **un solo archivo** `index.html`. Se abre en el navegador, no hay nada que
instalar ni un servidor que mantener.

1. **Al abrir** hace una consulta normal a la base y trae las ventas de hoy.
   Así la pantalla siempre muestra algo, aunque el flujo esté en cero.
2. **Se conecta al canal en vivo** (Realtime: la base avisa al instante cada
   vez que entra una venta nueva) y va sumando sobre lo ya cargado.
3. **Amortiguador**: las ventas que llegan se juntan y la pantalla se
   redibuja **cada 0,5 segundos**, no una vez por venta. Esto es lo que
   aguanta el golpe de 2.000 ventas de una sola vez sin trabarse.
4. **Consulta de respaldo cada 20 segundos**: vuelve a preguntar a la base por
   lo que pudiera haberse perdido. Si el canal en vivo se cae, el tablero
   sigue funcionando con esto solo.
5. **Tope de memoria**: guarda únicamente las 15 últimas ventas para la lista.
6. **Cambio de día**: a medianoche (hora de Chile) reinicia los totales solo.

### Por qué así

- El canal en vivo da la sensación de "está vivo", pero puede perder mensajes
  en una ráfaga. La consulta de respaldo corrige cualquier diferencia sola.
- Redibujar cada 0,5 s (y no por cada venta) mantiene la pantalla fluida
  aunque entren miles de ventas juntas.

---

## 4. Qué muestra

**Cuatro cifras grandes arriba** (pensadas para leerse a cinco metros):
ventas de hoy · facturación de hoy · ticket promedio · unidades vendidas.

**Tres gráficos**:
- Facturación por hora (barras; en el eje solo la hora).
- Las cinco comunas que más compran.
- Reparto por fuente de tráfico (dona, máximo cinco porciones).

**Las últimas quince ventas**, actualizándose solas.

**Montos abreviados**: `$1,2 M` en vez de `$1.203.450`, porque un número de
nueve dígitos no se lee de lejos.

**Cuando entra una venta nueva** se nota un instante: la fila aparece con una
línea bronce a la izquierda que se desvanece en 1 segundo, y las cuatro cifras
se tiñen de bronce un momento. Es lo único que demuestra que está vivo.

**Con cero ventas**: fondo limpio y un texto que dice, con palabras, que
todavía no hay datos. Sin errores a la vista, sin gráficos vacíos, sin un cero
gigante sin explicación.

---

## 5. Estados y qué significan (esquina superior derecha)

| Indicador | Qué pasa |
|---|---|
| **En vivo** (punto bronce) | Todo normal, las ventas entran por el canal en vivo |
| **En vivo con retraso · datos cada 20 s** | El canal en vivo va lento; la consulta de respaldo cubre |
| **Reconectando · datos cada 20 s** | Se cortó el canal; reintenta solo y sigue con el respaldo |
| **Sin canal en vivo · datos cada 20 s** | El canal no acepta la suscripción; el tablero sigue con el respaldo |

Ninguno de estos es un error rojo. Si aparece "sin canal en vivo" de forma
permanente aunque el flujo esté en ritmo normal, hay que revisar el Realtime
de la base (avisar al responsable).

---

## 6. Sistema de diseño

Alineado con **[lanimalier.cl](https://lanimalier.cl)**:

| Elemento | Decisión |
|---|---|
| Tipografía de cifras y títulos | **Bodoni Moda** (serif de alto contraste, elegante) |
| Tipografía de texto y etiquetas | **Hanken Grotesk** |
| Fondo | Crudo cálido `#FFF8F6` |
| Color de datos y acento | Bronce `#775A19` |
| Esquinas | Rectas, sin redondear |
| Estructura | Líneas finas y aire; sin sombras ni degradados |

Las dos tipografías van **incrustadas dentro del archivo** (no se descargan de
internet). El tablero se ve igual aunque la conexión de la sala sea mala.

---

## 7. Publicación

- **Repositorio**: GitHub (código versionado).
- **Hosting**: Vercel. Detecta `index.html` y lo sirve en la raíz, sin
  configuración.
- **Flujo de actualización**: cada vez que se sube un cambio a GitHub
  (`git push`), Vercel vuelve a publicar el tablero solo.

### Archivos del repositorio

| Archivo | Para qué |
|---|---|
| `index.html` | El tablero completo (único archivo que se ejecuta) |
| `README.md` | Descripción corta del proyecto |
| `BITACORA.md` | Este documento |
| `.gitignore` | Lista de archivos que no se suben (basura del sistema) |

---

## 8. Cómo mantenerlo

- **Cambiar algo del tablero**: se edita `index.html`, se guarda, se sube a
  GitHub. Vercel lo publica en un minuto.
- **La clave de la base** está dentro de `index.html`. Es la clave anónima de
  solo lectura y los datos son simulados, por eso puede ir en un sitio
  público. Si algún día se cambia por una clave con más permisos, **no debe
  quedar en un sitio público** — hay que parar y replantear.
- **Si se quiere dejar de simular** y mostrar ventas reales, primero confirmar
  con el responsable de la base que la tabla `ventas` deje de traer
  `simulado: true`.

---

## 9. Límites conocidos

- Solo se probó con la tabla `ventas`. No hay acceso a `orders` ni a otras
  tablas con esta clave.
- "Comunas que más compran" cuenta **número de ventas**, no monto.
- El día se calcula en horario de Chile; en el cambio de hora de invierno/
  verano el corte de medianoche podría desviarse unos minutos una vez al año.

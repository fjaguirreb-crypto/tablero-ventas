# Tablero de ventas en vivo — L'Animalier

Pantalla única para proyectar en sala: muestra las ventas de la tienda en
tiempo real. Un solo archivo (`index.html`), sin build ni dependencias.

## Cómo funciona

- Al abrir hace una consulta normal a la base (Supabase autoalojada) y pinta
  el historial del día.
- Después se conecta al canal en vivo (Realtime, evento `INSERT` sobre
  `lanimalier.ventas`) y va sumando cada venta nueva.
- Junta lo que llega y redibuja cada 0,5 s. Guarda solo las 15 últimas ventas.
- Si el canal en vivo se cae, sigue con una consulta de respaldo cada 20 s y
  lo dice en pantalla.

## Contenido

- **Cuatro cifras**: ventas de hoy, facturación de hoy, ticket promedio,
  unidades vendidas.
- **Gráficos**: facturación por hora, cinco comunas que más compran, reparto
  por fuente de tráfico.
- **Últimas quince ventas**, actualizándose solas.

## Diseño

Alineado con el sistema visual de [lanimalier.cl](https://lanimalier.cl):
tipografías Bodoni Moda y Hanken Grotesk (incrustadas en el archivo), fondo
crudo, esquinas rectas, bronce como acento. Funciona sin conexión a fuentes
externas.

## Datos

Usa la clave anónima de solo lectura de la base. Los datos de la tabla son
simulados (`simulado: true`).

## Publicación

Sitio estático. En Vercel no necesita configuración: detecta `index.html` y
lo sirve en la raíz.

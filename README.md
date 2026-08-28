# Signal Room — L'Animalier

Dashboard de ventas en vivo pensado para sala y demo pública: muestra las
ventas de la tienda en tiempo real con una UI más tecnológica, construida en
un solo archivo (`index.html`) usando React y Recharts vía CDN.

## Cómo funciona

- Al abrir consulta la base (Supabase autoalojada) y arma el historial del día.
- Luego se conecta al canal en vivo (Realtime, evento `INSERT` sobre
  `lanimalier.ventas`) y suma las ventas nuevas.
- Amortigua el render cada `0,5 s`, mantiene solo las `15` últimas ventas y
  reconcilia contra la base cada `20 s`.
- Si el canal en vivo se cae, sigue funcionando con la consulta de respaldo y
  lo muestra en el estado del tablero.

## Contenido

- **Hero operacional** con estado del sistema, reloj y señales rápidas.
- **Cuatro métricas clave**: ventas de hoy, facturación, ticket promedio,
  unidades vendidas.
- **Tres visualizaciones Recharts**: facturación por hora, comunas top y mix
  de fuentes.
- **Feed de últimas quince ventas**, actualizándose solo.

## Diseño

Look frontier 2026: fondo oscuro atmosférico, paneles glass, glow controlado,
tipografía más expresiva y una jerarquía visual pensada para lectura a
distancia.

## Datos

Usa la clave anónima de solo lectura de la base. Los datos de la tabla son
simulados (`simulado: true`).

## Publicación

Sitio estático. En Vercel no necesita configuración: detecta `index.html` y
lo sirve en la raíz.

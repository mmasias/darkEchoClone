# Dark Echo — Prototipo Web v0.1

|![](/images/preview.webp)|Clon de inspiración del juego *Dark Echo* (RAC7, 2015).|
|:-:|:-:|

Juego de terror y sigilo donde el jugador es invidente y percibe el entorno exclusivamente mediante ondas de sonido visualizadas en pantalla.

## Concepto central

El mismo mecanismo que permite "ver" delata la posición del jugador. Caminar emite ondas amplias que revelan el entorno pero alertan a los enemigos. El sigilo emite ondas tenues que apenas revelan nada. Esta tensión entre información y vulnerabilidad es el núcleo del diseño.

## Estado actual (v0.1)

Archivo único: `dark_echo.html` — HTML5/Canvas vanilla, sin dependencias.

### Lo que funciona

- **Laberinto fijo** de 17×13 celdas (COLS × ROWS), representado como array 2D:
  - `0` = pasillo transitable
  - `1` = pared
  - `2` = salida (esquina inferior derecha)
- **Jugador** representado como punto blanco (radio 3px)
  - Click → caminar (ondas amplias, cyan, alcance 7 celdas)
  - Doble-click → sigilo (ondas tenues, verde, alcance 4 celdas)
  - Movimiento por pathfinding básico: dirección al objetivo con resolución de colisiones por eje
- **Sistema de ondas (raycast)**
  - `emitWave(loud)` genera N rayos radiales desde la posición del jugador
  - Cada rayo avanza 2px por iteración a velocidad `CELL * 8` unidades/segundo
  - Un rebote por rayo en paredes (reflexión por eje)
  - Al detectar un enemigo: el rayo se vuelve rojo, muere, alerta al enemigo
  - Fade por edad: `alpha = 1 - age / maxAge`
- **3 enemigos estáticos** que se activan al ser detectados por una onda
  - Estado `alerted`: persiguen al jugador durante `alertTimer` segundos (5s)
  - Velocidad de persecución: 2.5 celdas/segundo
  - Colisión con jugador → game over
- **Cursor personalizado**: retícula tenue (círculo + cruz, opacidad 0.35)
- **Línea guía** punteada desde jugador hasta destino
- **Pantalla completa**: botón + F11, canvas escalado por ratio

### Constantes clave

```javascript
const CELL = 48;          // px por celda (versión standalone)
const COLS = 17;
const ROWS = 13;
// Ondas ruidosas: 72 rayos, alcance 7 celdas, maxAge 1.2s
// Ondas sigilosas: 48 rayos, alcance 4 celdas, maxAge 0.8s
// Intervalo entre emisiones: 0.35s (ruido) / 0.8s (sigilo)
```

### Arquitectura del loop

```
requestAnimationFrame(loop)
  └── update(dt)
        ├── mover jugador hacia targetX/targetY
        ├── emitWave() según stepTimer
        ├── castRay() para cada rayo vivo
        └── updateEnemies()
  └── draw()
        ├── ondas (rayos con color/alpha)
        ├── jugador (punto blanco)
        ├── línea guía
        ├── cursor
        └── overlays (game over / win)
```

## Instrucciones de ejecución

Abrir `dark_echo.html` directamente en navegador. Sin servidor, sin dependencias. F11 o botón `[ FULLSCREEN ]` para pantalla completa.
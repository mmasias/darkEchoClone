# Clon del Dark Echo — Prototipo Web v0.3

Juego de terror y sigilo donde el jugador es invidente y percibe el entorno exclusivamente mediante ondas de sonido visualizadas en pantalla.

> ***[Jugar en el navegador](https://mmasias.github.io/darkEchoClone/dark_echo.html)***

|![](/images/preview.webp)|Clon de inspiración del juego *Dark Echo* (RAC7, 2015).|
|:-:|:-:|


## Concepto central

El mismo mecanismo que permite "ver" delata la posición del jugador. Caminar emite ondas amplias que revelan el entorno pero alertan a los enemigos. El sigilo emite ondas tenues que apenas revelan nada. Esta tensión entre información y vulnerabilidad es el núcleo del diseño.

## Estado actual (v0.3)

Archivo único: `dark_echo.html` — HTML5/Canvas vanilla, sin dependencias.

### Lo que funciona

- **Laberinto fijo** de 17×13 celdas (COLS × ROWS), representado como array 2D:
  - `0` = pasillo transitable
  - `1` = pared
  - `2` = salida (esquina inferior derecha)
- **Jugador** representado como punto blanco (radio 3px)
  - Click para mover; modo seleccionado mediante botones en la cabecera
  - **Sigilo** (verde): 1.2 c/s, 48 rayos, alcance 4 celdas, intervalo 0.8s
  - **Caminar** (blanco): 2.8 c/s, 72 rayos, alcance 7 celdas, intervalo 0.35s
  - **Correr** (naranja): 4.5 c/s, 90 rayos, alcance 10 celdas, intervalo 0.2s
  - Movimiento por pathfinding básico: dirección al objetivo con resolución de colisiones por eje
- **Sistema de ondas (raycast)**
  - `emitWave(loud)` genera N rayos radiales desde la posición del jugador
  - Cada rayo avanza 2px por iteración a velocidad `CELL * 8` unidades/segundo
  - Un rebote por rayo en paredes (reflexión por eje)
  - Al detectar un enemigo: el rayo se vuelve rojo, muere, alerta al enemigo
  - Fade por edad: `alpha = 1 - age / maxAge`
- **3 enemigos con patrulla autónoma** que emiten sus propias ondas de sonido
  - Cada enemigo sigue una ruta de waypoints en bucle a 1.5 celdas/segundo
  - Al ser detectado por una onda del jugador: abandona la patrulla y persigue al jugador durante `alertTimer` segundos (5s) a 2.5 celdas/segundo
  - Al expirar la alerta: retoma la patrulla desde el waypoint más cercano
  - Colisión con jugador (radio < 0.5 celdas) -> game over
  - **Ondas propias de enemigos** (naranja en patrulla, naranja-rojo en alerta):
    - Intervalo de emisión: 0.6s (patrulla) / 0.25s (alerta)
    - Rayos: 48 (patrulla) / 60 (alerta), alcance 4-5 celdas, maxAge 0.7-0.9s
    - No alertan a otros enemigos ni afectan al jugador
    - Solo son visibles si el jugador está a menos de 7 celdas del enemigo (fade lineal entre 4 y 7 celdas)
- **Cursor personalizado**: retícula tenue (círculo + cruz, opacidad 0.35)
- **Línea guía** punteada desde jugador hasta destino
- **Pantalla completa**: botón + F11, canvas escalado por ratio

### Rutas de patrulla (waypoints)

Verificadas celda a celda contra el mapa para garantizar movimiento sin bloqueos:

```
Enemigo 0 (zona central): (5,5) -> (11,5) -> (11,7) -> (5,7)  [bucle rectangular]
Enemigo 1 (zona derecha):  (11,1) <-> (15,1)                   [corredor horizontal]
Enemigo 2 (zona inferior): (1,9)  -> (5,9)  -> (5,11) -> (1,11) [bucle rectangular]
```

### Constantes clave

```javascript
const CELL = 48;          // px por celda (versión standalone)
const COLS = 17;
const ROWS = 13;
// Sigilo:   48 rayos, alcance 4 celdas,  maxAge 0.8s, intervalo 0.8s,  velocidad 1.2 c/s
// Caminar:  72 rayos, alcance 7 celdas,  maxAge 1.2s, intervalo 0.35s, velocidad 2.8 c/s
// Correr:   90 rayos, alcance 10 celdas, maxAge 1.5s, intervalo 0.2s,  velocidad 4.5 c/s
// Enemigo patrulla: 48 rayos, alcance 4 celdas, maxAge 0.9s, intervalo 0.6s
// Enemigo alerta:   60 rayos, alcance 5 celdas, maxAge 0.7s, intervalo 0.25s
// Percepción de enemigos: fade lineal entre 4 celdas (plena) y 7 celdas (nula)
```

### Arquitectura del loop

```
requestAnimationFrame(loop)
  |-- update(dt)
  |     |-- mover jugador hacia targetX/targetY
  |     |-- emitWave() según stepTimer
  |     |-- castRay() para cada rayo vivo (jugador)
  |     `-- updateEnemies()
  |           |-- patrulla: mover hacia waypoint actual con slide en colisiones
  |           |-- alerta: perseguir jugador, decrementar alertTimer
  |           |-- emitEnemyWave() según stepTimer del enemigo
  |           `-- castRay(..., isEnemy=true) para rayos de enemigos
  `-- draw()
        |-- ondas del jugador (cyan/verde con alpha por edad)
        |-- ondas de enemigos (naranja/naranja-rojo, solo si dist < 7 celdas)
        |-- puntos de enemigos (naranja, con fade por distancia)
        |-- jugador (punto blanco)
        |-- linea guia
        |-- cursor
        `-- overlays (game over / win)
```

## Jugar en el navegador

[https://mmasias.github.io/darkEchoClone/dark_echo.html](https://mmasias.github.io/darkEchoClone/dark_echo.html)

## Ejecución local

Abrir `dark_echo.html` directamente en navegador. Sin servidor, sin dependencias. F11 o botón `[ FULLSCREEN ]` para pantalla completa.

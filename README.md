# Clon del Dark Echo — Prototipo Web v0.5

Juego de terror y sigilo donde el jugador es invidente y percibe el entorno exclusivamente mediante ondas de sonido visualizadas en pantalla.

> ***[Jugar en el navegador](https://mmasias.github.io/darkEchoClone/dark_echo.html)***

|![](/images/preview.webp)|Clon de inspiración del juego *Dark Echo* (RAC7, 2015).|
|:-:|:-:|


## Concepto central

El mismo mecanismo que permite "ver" delata la posición del jugador. Caminar emite ondas amplias que revelan el entorno pero alertan a los enemigos. El sigilo emite ondas tenues que apenas revelan nada. Esta tensión entre información y vulnerabilidad es el núcleo del diseño.

## Estado actual (v0.5)

Archivo único: `dark_echo.html` — HTML5/Canvas vanilla, sin dependencias.

### Lo que funciona

- **Laberinto 81×41** generado proceduralmente con DFS (Recursive Backtracker) y PRNG con seed fija (reproducible). Salida en la esquina opuesta al spawn.
- **Cámara** centrada en el jugador, clampeada a los bordes del mapa. El viewport muestra una ventana de 17×13 celdas que se desplaza por el mundo.
- **Jugador** representado como punto blanco (radio 3px)
  - Click para mover; modo seleccionado mediante botones en la cabecera
  - **Sigilo** (verde): 1.2 c/s, 48 rayos, alcance 4 celdas, intervalo 0.8s
  - **Caminar** (blanco): 2.8 c/s, 72 rayos, alcance 7 celdas, intervalo 0.35s
  - **Correr** (naranja): 4.5 c/s, 90 rayos, alcance 10 celdas, intervalo 0.2s
  - Si el jugador queda completamente bloqueado por una pared, el destino se cancela automáticamente
- **Acciones instantáneas** (sin necesidad de moverse), con cooldown:
  - **SUSURRO**: onda puntual desde posición actual — 36 rayos, alcance 2.5 celdas, cooldown 0.8s. Para explorar esquinas sin moverse.
  - **TOS**: onda masiva desde posición actual — 100 rayos, alcance 12 celdas, cooldown 4s. Para crear señuelos y atraer enemigos.
  - El botón muestra la cuenta atrás mientras recarga (`TOS 3.2`, `TOS 1.0`...)
- **Sistema de ondas (raycast)**
  - `emitWave(mode)` genera N rayos radiales desde la posición del jugador
  - Cada rayo avanza 2px por iteración a velocidad `CELL * 8` unidades/segundo
  - Un rebote por rayo en paredes (reflexión por eje)
  - Al detectar un enemigo: el rayo se vuelve rojo, muere, alerta al enemigo
  - Fade por edad: `alpha = 1 - age / maxAge`
  - Culling de rayos fuera del viewport para rendimiento
- **3 enemigos** posicionados aleatoriamente lejos del spawn (distancia Manhattan >= 40)
  - Patrullan entre 2 waypoints generados aleatoriamente
  - Al ser detectados por una onda del jugador: persiguen durante 5s a 2.5 c/s
  - Al expirar la alerta: retoman la patrulla desde el waypoint más cercano
  - Colisión con jugador (radio < 0.5 celdas) -> game over
  - **Ondas propias** (naranja en patrulla, naranja-rojo en alerta):
    - Solo visibles si el jugador está a menos de 7 celdas (fade lineal entre 4 y 7 celdas)
    - No alertan a otros enemigos ni afectan al jugador

### Constantes clave

```javascript
const CELL = 48;          // px por celda
const COLS = 81, ROWS = 41;           // dimensiones del laberinto
const BASE_W = 17 * CELL;             // viewport: 816px
const BASE_H = 13 * CELL;             // viewport: 624px
// Sigilo:   48 rayos, alcance 4 celdas,  maxAge 0.8s, intervalo 0.8s,  velocidad 1.2 c/s
// Caminar:  72 rayos, alcance 7 celdas,  maxAge 1.2s, intervalo 0.35s, velocidad 2.8 c/s
// Correr:   90 rayos, alcance 10 celdas, maxAge 1.5s, intervalo 0.2s,  velocidad 4.5 c/s
// Susurro:  36 rayos, alcance 2.5 celdas, maxAge 0.6s, cooldown 0.8s  (instantáneo)
// Tos:     100 rayos, alcance 12 celdas,  maxAge 1.8s, cooldown 4.0s  (instantáneo)
// Enemigo patrulla: 48 rayos, alcance 4 celdas, maxAge 0.9s, intervalo 0.6s
// Enemigo alerta:   60 rayos, alcance 5 celdas, maxAge 0.7s, intervalo 0.25s
// Percepción de enemigos: fade lineal entre 4 celdas (plena) y 7 celdas (nula)
```

### Arquitectura del loop

```
requestAnimationFrame(loop)
  |-- update(dt)
  |     |-- updateCamera() — centra viewport en el jugador
  |     |-- tick cooldowns de susurro y tos
  |     |-- mover jugador hacia targetX/targetY (cancela si completamente bloqueado)
  |     |-- emitWave() según stepTimer
  |     |-- castRay() para cada rayo vivo (jugador)
  |     `-- updateEnemies()
  |           |-- patrulla: mover hacia waypoint actual con slide en colisiones
  |           |-- alerta: perseguir jugador, decrementar alertTimer
  |           |-- emitEnemyWave() según stepTimer del enemigo
  |           `-- castRay(..., isEnemy=true) para rayos de enemigos
  `-- draw()
        |-- ondas del jugador (cyan/verde con alpha por edad, culling por viewport)
        |-- ondas de enemigos (naranja, culling por viewport y distancia al jugador)
        |-- puntos de enemigos (naranja, fade por distancia)
        |-- jugador (punto blanco)
        |-- linea guia
        |-- cursor
        `-- overlays (game over / win)
```

## Jugar en el navegador

[https://mmasias.github.io/darkEchoClone/dark_echo.html](https://mmasias.github.io/darkEchoClone/dark_echo.html)

## Ejecución local

Abrir `dark_echo.html` directamente en navegador. Sin servidor, sin dependencias. F11 o botón `[ FULLSCREEN ]` para pantalla completa.

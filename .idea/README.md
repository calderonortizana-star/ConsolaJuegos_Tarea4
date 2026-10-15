# Consola de juegos en Java

Consola de texto con **dos juegos por turnos** para varios jugadores, y una pantalla de
**estadisticas** que muestra el **top 3** de cada juego.

| # | Juego | Quien lo hizo |
|---|-------|---------------|
| 1 | Tetris | propio |
| 2 | City Block (estilo Tower Bloxx) | propio |
| 3 | *(pendiente)* | otra persona, se adapta con la interfaz `Jugable` |

## Como compilar y ejecutar

Desde la carpeta raiz del proyecto (requiere Java 11 o superior):

```bash
# Linux / macOS
javac -d out $(find src -name "*.java")
java -cp out juegos.Main
```
```powershell
# Windows (PowerShell)
javac -d out (Get-ChildItem -Recurse src -Filter *.java).FullName
java -cp out juegos.Main
```

Usa una terminal real (Windows Terminal, PowerShell 7, macOS Terminal...) para ver los colores y la
animacion de City Block. En el menu, la opcion **Colores** los activa o desactiva.

## Estructura de paquetes

```
src/juegos/
├── Main.java                  la consola: menu, lanza juegos, registra estadisticas
├── comun/
│   ├── Jugable.java                 INTERFAZ que todo juego debe implementar
│   ├── PuntuacionComparable.java    INTERFAZ con el metodo comparar()
│   ├── Puntuacion.java              clase base de lo que devuelve start()
│   ├── Estadisticas.java            top 3 por juego (generico)
│   ├── Juego.java                   clase base: pide jugadores y maneja turnos
│   ├── Jugador.java / ResumenPartida.java / Color.java
├── juego1/   Tetris:      JuegoTetris, Pieza, Tablero, PuntuacionTetris
└── juego2/   City Block:  JuegoCityBlock, Grua, Torre, Bloque, PuntuacionCityBlock
```

## Como funcionan las estadisticas (diseno generico)

La idea es que la consola **no sepa nada de cada juego**, de modo que agregar un juego nuevo
no obligue a cambiar el codigo existente.

1. **`Jugable`** (interfaz): todo juego tiene `getNombre()`, `start()` y `puntuacionesIniciales()`.
   `start()` juega una partida completa y **devuelve una `Puntuacion`**.
2. **`Puntuacion`**: clase base del resultado (ganador, puntos, tiempo, movimientos).
   Cada juego hereda y crea la suya: `PuntuacionTetris`, `PuntuacionCityBlock`.
3. **`PuntuacionComparable`** (interfaz): tiene `int comparar(PuntuacionComparable otra)`.
   Devuelve **> 0** si esta puntuacion es mejor, **< 0** si es peor, **0** si es igual.
   **Cada juego decide como comparar**:
   - Tetris: mas puntos; si empatan, menos tiempo.
   - City Block: mas puntos; luego mas pisos; luego menos tiempo.
4. **`Estadisticas`** guarda una lista de 3 `PuntuacionComparable` por juego. Al terminar una partida,
   la consola llama `registrar(juego, puntuacion)`: recorre el top 3, usa `comparar()` para ubicar
   la nueva puntuacion, la inserta y saca la que queda de cuarta. Devuelve el puesto (1-3) o 0.
5. **Escenario justo:** el top 3 empieza con datos "quemados" (Pedrito, Juanito, Maria), definidos en
   `puntuacionesIniciales()` de cada juego, para que desde la primera partida se pueda comparar.

Flujo: `Main` -> `juego.start()` -> `Puntuacion` -> `estadisticas.registrar(...)` -> top 3.

## Como agregar un juego nuevo (por ejemplo el que te pasan)

1. Crear un paquete nuevo, por ejemplo `juegos.juego3`.
2. Crear `PuntuacionXxx extends Puntuacion` e implementar `comparar()` y `detalle()`.
3. Hacer que el juego implemente `Jugable` (o que extienda `Juego`, que ya pide los jugadores y
   los turnos), con `start()` devolviendo su `PuntuacionXxx`, y `puntuacionesIniciales()` con 3 datos.
4. En `Main.java` agregar **una sola linea**: `juegos.add(new JuegoXxx(entrada));`

El menu y las estadisticas se arman solos.

## Los juegos

**Tetris (por turnos):** cada jugador tiene su tablero de 10x14 y recibe 12 piezas. En su turno escribe
`R` para rotar o el numero de columna donde soltarla. Linea completa = 100 pts (300/500/800 por 2/3/4
a la vez) y 4 pts por pieza colocada.

**City Block:** la grua balancea un piso y se suelta con ENTER. El color indica donde caeria:
verde = perfecto (+15), amarillo = bien (+13), naranja = mal alineado (+10), rojo = cae al vacio
(-1 vida). Los pisos mal alineados suben el balanceo; si pasa de 14 se pierde el ultimo piso y una vida.
3 vidas, meta de 10 pisos.

## Uso de GitHub

Los cambios se van registrando con commits pequenos mientras se programa:

```bash
git init
git add .
git commit -m "Estructura inicial y clase base Juego"
git branch -M main
git remote add origin https://github.com/TU_USUARIO/consola-juegos.git
git push -u origin main
# despues, cada cambio:
git add .
git commit -m "Describe lo que cambiaste"
git push
```

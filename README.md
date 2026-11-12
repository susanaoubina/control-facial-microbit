# Control facial para micro:bit

👉 **Abrir la app:** https://susanaoubina.github.io/control-facial-microbit/

Aplicación web que usa la cámara para detectar la cara y envía sus gestos a la **BBC micro:bit** por **cable USB** o por **Bluetooth**. Con ella se puede controlar un robot de cartón, por ejemplo la [cabeza robot de LOFI Robot](https://cardboard.lofirobot.com/robot-head/): la cabeza gira cuando giras la tuya y la boca se abre cuando abres la tuya.

Es una versión propia y en español de la [Face Tracking App de LOFI Robot](https://cardboard.lofirobot.com/apps/face-tracking/). Usa el mismo formato de mensaje, así que sirve para sus mismos proyectos de MakeCode.

## Qué hace

- Detecta la cara con [MediaPipe Face Landmarker](https://ai.google.dev/edge/mediapipe/solutions/vision/face_landmarker) (de Google).
- Calcula 11 valores: posición, distancia, giro, inclinación, ladeo, boca, ojos, sonrisa y si hay una cara visible.
- Los envía a la micro:bit 10 veces por segundo.
- Enseña en pantalla el mensaje exacto que se envía, con cada dato en un color, para entender cómo funciona la comunicación.
- **Privacidad:** el vídeo se procesa en el propio navegador. Ninguna imagen se sube a internet.

## Requisitos

| | |
|---|---|
| Navegador | **Chrome** o **Edge**, en ordenador (Windows, macOS, Linux, ChromeOS) o en Android |
| Cable USB | Solo en ordenador (Chrome o Edge) |
| Bluetooth | Ordenador o Android. **No funciona en iPhone ni iPad** |
| micro:bit | V1 o V2. Con Bluetooth se recomienda la **V2** (la V1 tiene poca memoria) |
| Robot | Adaptador para micro:bit con alimentación propia para los servos y 2 servos de 180° |

## Montaje de la cabeza robot

| Servo | Función | Pin de la micro:bit |
|---|---|---|
| Servo 1 | Girar la cabeza | **P1** |
| Servo 2 | Abrir la boca | **P0** |

- Conecta los servos al adaptador, no directamente a la micro:bit. El adaptador les da la corriente que necesitan y así la micro:bit no se reinicia cuando se mueven.
- Cable de señal (naranja o amarillo) → pin; cable rojo → V+; cable marrón o negro → GND.
- Enciende la alimentación del adaptador (pilas o batería) antes de probar.

## Programa de MakeCode (cable USB)

1. Abre [makecode.microbit.org](https://makecode.microbit.org) y crea un proyecto nuevo.
2. Pasa a la pestaña **JavaScript**, borra lo que haya, pega este código y vuelve a **Bloques**.

```javascript
let datos = ""
let giro = 50
let boca = 0
let visible = 0
serial.setBaudRate(BaudRate.BaudRate115200)
basic.showIcon(IconNames.Square)

serial.onDataReceived(serial.delimiters(Delimiters.NewLine), function () {
    datos = serial.readUntil(serial.delimiters(Delimiters.NewLine))
    if (datos.length >= 19) {
        giro = parseFloat(datos.substr(6, 2))
        boca = parseFloat(datos.substr(10, 2))
        visible = parseFloat(datos.substr(18, 1))
    }
})

basic.forever(function () {
    if (visible == 1) {
        led.plotBarGraph(giro, 99)
        // Giro de la cara (20-80) → ángulo del servo (0-180)
        pins.servoWritePin(AnalogPin.P1,
            Math.constrain(Math.map(giro, 20, 80, 0, 180), 0, 180))
        // Boca (0-50) → servo de 90 (cerrada) a 150 (abierta)
        pins.servoWritePin(AnalogPin.P0,
            Math.constrain(Math.map(boca, 0, 50, 90, 150), 90, 150))
    } else {
        basic.showIcon(IconNames.Asleep, 0)
    }
})
```

3. Descarga el programa a la micro:bit.
4. **Cierra la pestaña de MakeCode** o desconéctalo de la micro:bit. Si no, la app no podrá usar el puerto USB.

**Ajustes:**
- Si la cabeza gira al revés, cambia `0, 180` por `180, 0`.
- Ajusta los rangos `20, 80` (giro) y `0, 50` (boca) mirando las barras de la app.
- Ajusta `90, 150` según cómo esté montada la boca.

### Versión Bluetooth

1. Añade la extensión **bluetooth**. Si MakeCode avisa de que no es compatible con *radio*, acepta quitar *radio*.
2. Pon `bluetooth.startUartService()` al principio del programa.
3. Cambia `serial.onDataReceived` por `bluetooth.onUartDataReceived` y `serial.readUntil` por `bluetooth.uartReadUntil`.
4. Ve al ⚙️ → **Configuración del proyecto** y elige **No Pairing Required**.

## Usar la app

1. Abre la app y pulsa **Activar la cámara**. Da permiso a la cámara.
2. Conecta la micro:bit por USB y pulsa **Cable USB**. En la lista, elige la micro:bit (aparece como *mbed Serial Port*, *BBC micro:bit* o similar).
3. Mueve la cabeza. El contador **Mensajes enviados** debe subir y la barra de LED de la micro:bit debe moverse.

Otros botones: **Cambiar cámara** (frontal o trasera), **Efecto espejo**, **Malla on/off**, **Pantalla completa** y una barra para cambiar cada cuánto se envían los datos.

## Formato del mensaje

Cada mensaje tiene **19 cifras** seguidas de un salto de línea (`\n`). Ejemplo: `0547997308423130471`

| Cifras (`substr`) | Dato | Rango |
|---|---|---|
| 0-1 (`0, 2`) | X: posición horizontal | 0-99 |
| 2-3 (`2, 2`) | Y: posición vertical | 0-99 |
| 4-5 (`4, 2`) | Distancia a la cámara | 0-99 |
| 6-7 (`6, 2`) | Giro (yaw), izquierda ↔ derecha | 0-99 |
| 8-9 (`8, 2`) | Inclinación (pitch), abajo ↔ arriba | 0-99 |
| 10-11 (`10, 2`) | Boca abierta | 0-99 |
| 12-13 (`12, 2`) | Ojo izquierdo abierto | 0-99 |
| 14-15 (`14, 2`) | Ojo derecho abierto | 0-99 |
| 16 (`16, 1`) | Ladeo (roll) | 0-9 |
| 17 (`17, 1`) | Sonrisa / beso | 0-9 |
| 18 (`18, 1`) | Cara visible | 0 ó 1 |

En MakeCode, cada dato se saca con `datos.substr(inicio, longitud)` y se convierte en número con `parseFloat` (bloque *convertir a número*).

**De la micro:bit a la app:** si la micro:bit envía `camera` seguido de salto de línea, la app cambia de cámara.

## ¿Qué pasa si me tapo parte de la cara?

Prueba a taparte un ojo o la boca con la mano. Verás que:

- **Con un ojo tapado**, su valor casi no cambia.
- **Con la boca tapada**, la sonrisa sube casi al máximo.

**No es un fallo de la app.** La inteligencia artificial que detecta la cara (MediaPipe) siempre dibuja una cara completa. Lo que no puede ver, **se lo inventa** a partir del resto de la cara y de todas las caras con las que aprendió. Fíjate en que la malla dibuja los labios o el ojo encima de tu mano.

- Si no ve el ojo, supone que está abierto, como en una cara normal.
- Si no ve la boca, se imagina unos labios normales, y con ellos la sonrisa sale alta.

La IA no sabe que algo está tapado: completa lo que falta con lo que ha aprendido, y lo hace con total seguridad aunque se equivoque.

**Sobre la sonrisa:** con la cara relajada ya marca 6 o 7. Baja hacia 0 al poner morritos (beso) y sube a 9 con una sonrisa grande. Mide cómo de estirada está la boca, no si estás contento.

## Solución de problemas

| Problema | Solución |
|---|---|
| No aparece la imagen de la cámara | Abre la app desde su enlace (https) y da permiso a la cámara. Cierra otras apps que estén usando la cámara (Meet, Teams…). |
| «No se pudo abrir el puerto» | Cierra MakeCode o cualquier otra pestaña conectada a la micro:bit. Desenchufa la micro:bit, vuelve a enchufarlo y prueba otra vez. |
| El contador sube, pero el robot no se mueve | Comprueba que el adaptador está encendido y que los servos están en P0 y P1. Comprueba que el programa de MakeCode está descargado en la micro:bit. |
| La micro:bit se reinicia | Los servos deben alimentarse desde el adaptador. Revisa las pilas o la batería. |
| No sale el botón Bluetooth o da error | Usa Chrome o Edge (no iPhone ni iPad). Elige **No Pairing Required** en MakeCode. |
| Cara sad y error 020 con Bluetooth | Es una micro:bit V1 sin memoria suficiente. Simplifica el programa o usa una micro:bit V2. |
| Los movimientos son muy bruscos o muy pequeños | Ajusta los rangos de `Math.map` en el programa. |

## Créditos

- Idea, formato de datos y diseño del robot: [LOFI Robot / Cardboard Robots](https://cardboard.lofirobot.com/).
- Detección facial: [MediaPipe](https://github.com/google-ai-edge/mediapipe) (Google, licencia Apache 2.0).
- Versión en español adaptada para el aula: **Susana Oubiña Falcón** ([@susanaoubina](https://github.com/susanaoubina)), con la ayuda de **Claude Opus 5.5** (Anthropic).

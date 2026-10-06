HUMANOS VS LLM: CREATIVIDAD APLICADA A LA SIMULACIÓN DE DATOS SERIALES CON MICRO:BIT
Equipo simulado: Mando Gamer (reportes HID)

Autora: Dafne Jael Moguel Benitez - No. de control: 22211616
Instituto Tecnológico de Tijuana


------------------------------------------------------------
1. HUMANOS VS LLM: ¿QUIÉN CREA?
------------------------------------------------------------

Margaret Boden distingue tres tipos de creatividad: combinatoria (unir
ideas conocidas), exploratoria (recorrer las reglas de un dominio) y
transformacional (cambiar esas reglas). Un LLM es muy bueno en las dos
primeras: aprendió millones de patrones y los recombina rápido. La
persona aporta lo que el modelo no tiene: la intención, el contexto
real y el criterio para decidir si algo tiene sentido.

En este proyecto, el LLM se usó como generador de la referencia técnica
(el formato real de un reporte HID de mando/gamepad), y yo, como
ingeniera, validé esa información contra la especificación oficial
de USB HID antes de programarla en el micro:bit.


------------------------------------------------------------
2. EL RETO
------------------------------------------------------------

El micro:bit simula un MANDO GAMER enviando datos por el puerto serial,
siguiendo el estándar real de un reporte HID Gamepad (USB HID Usage
Tables, Usage Page 0x01 "Generic Desktop", Usage 0x05 "Gamepad", con
botones definidos bajo Usage Page 0x09 "Button").

Un reporte típico de gamepad incluye:
  - Dos ejes analógicos (joystick izquierdo: X, Y)
  - Un bitmap de botones (cada bit representa un botón presionado/suelto)
  - Un "hat switch" o POV (dirección del D-pad, 0-7, o 8 = centrado)

ESTADO NORMAL (reporte en reposo, sin entradas):
  Trama: 80 80 00 08
  Interpretación:
    X = 0x80 (128)  -> joystick centrado
    Y = 0x80 (128)  -> joystick centrado
    Botones = 0x00  -> ningún botón presionado
    Hat = 0x08 (8)  -> D-pad centrado (sin dirección)

ESTADO CON FALLA (stick drift / botón atascado):
  Trama: 80 05 01 08
  Interpretación:
    X = 0x80 (128)  -> eje X correcto
    Y = 0x05 (5)    -> el eje Y se desvía solo hacia un extremo sin
                       que el jugador toque el stick (falla conocida
                       como "stick drift", muy común en controles
                       desgastados)
    Botones = 0x01  -> el bit 0 (botón A / X según el control) queda
                       encendido de forma continua (botón atascado)
    Hat = 0x08      -> D-pad centrado

Fuente para validar el formato: USB Implementers Forum, "HID Usage
Tables for USB", secciones de Usage Page Generic Desktop (0x01) y
Button Page (0x09). https://www.usb.org/hid


------------------------------------------------------------
3. ARQUITECTURA DE LA SOLUCIÓN
------------------------------------------------------------

Se reutilizó la misma arquitectura validada en un proyecto previo
(Whack-a-Mole con micro:bit):

  [Micro:bit + MicroPython]  --(USB Serial, 115200 baudios)-->  [C++ en PC]
     Genera y envía las             Recibe, interpreta y
     tramas del mando                clasifica (normal / falla)

LADO 1 - Micro:bit (MicroPython):
  Envía por serial, cada cierto intervalo, una trama en formato texto
  equivalente al reporte HID (ej: "80,80,00,08" para normal o
  "80,05,01,08" para falla), alternando entre ambos estados para
  poder demostrar la clasificación en el receptor.

LADO 2 - PC (C++):
  Abre el puerto COM correspondiente, lee línea por línea los reportes
  recibidos, los interpreta (separa X, Y, botones, hat) y clasifica
  el estado como NORMAL o FALLA según si los valores están dentro de
  rango esperado (ej: Y muy bajo sin input del usuario = posible
  stick drift; bit de botón sostenido por más tiempo del esperado =
  posible botón atascado).


------------------------------------------------------------
4. VALIDACIÓN (LLM vs HUMANO)
------------------------------------------------------------

El LLM propuso el formato de trama y el ejemplo de falla. Como
ingeniera validé:
  - Que el orden de campos (X, Y, botones, hat) corresponde a cómo
    un descriptor HID de gamepad real estructura su reporte.
  - Que "stick drift" y "botón atascado" son fallas documentadas y
    reales en controles físicos, no inventadas por el modelo.
  - Que los valores usados (0x80 como centro de un eje de 8 bits,
    0x08 como valor de "hat centrado" en una codificación de 0-7 más
    estado neutro) son consistentes con la especificación HID.

Decidir si estos datos eran correctos, y no solo aceptar lo que el LLM
entregó, fue el trabajo humano en este ejercicio.


------------------------------------------------------------
5. CIERRE
------------------------------------------------------------

El LLM generó en segundos una referencia técnica que de otra forma
hubiera tomado tiempo buscar en documentación oficial. Pero decidir
si esos datos eran correctos, armar la trama real y conectar el
hardware (micro:bit) con el estándar fue trabajo propio. Esa es la
competencia que esta actividad buscaba desarrollar.

------------------------------------------------------------
6. Evidencia
------------------------------------------------------------

<img width="1534" height="2048" alt="d93ca6ad-867a-4a19-95a4-249adfdd0b81" src="https://github.com/user-attachments/assets/e3f3160d-5563-484d-94a8-81ed906570d7" />
<img width="1640" height="953" alt="Captura de pantalla 2026-09-29 152849" src="https://github.com/user-attachments/assets/1664df7c-3d01-4e14-83d5-eb17b3c88274" />

PS C:\Users\DELL\Proyectos\Microbit-Lenguaje de Interfaz\JuegoDelTopo> .\JuegoDelTopo.exe
Juego iniciado. Tienes 20segundos.
¡Acierto! Puntos: 1 | Tiempo restante: 18s
¡Acierto! Puntos: 2 | Tiempo restante: 17s
¡Acierto! Puntos: 3 | Tiempo restante: 15s
¡Acierto! Puntos: 4 | Tiempo restante: 14s
¡Acierto! Puntos: 5 | Tiempo restante: 14s
¡Acierto! Puntos: 6 | Tiempo restante: 12s
¡Acierto! Puntos: 7 | Tiempo restante: 11s
¡Acierto! Puntos: 8 | Tiempo restante: 10s
¡Acierto! Puntos: 9 | Tiempo restante: 10s
¡Acierto! Puntos: 10 | Tiempo restante: 9s
¡Acierto! Puntos: 11 | Tiempo restante: 8s
¡Acierto! Puntos: 12 | Tiempo restante: 7s
¡Acierto! Puntos: 13 | Tiempo restante: 6s
¡Acierto! Puntos: 14 | Tiempo restante: 6s
¡Acierto! Puntos: 15 | Tiempo restante: 4s
¡Acierto! Puntos: 16 | Tiempo restante: 2s

¡Tiempo terminado! Puntaje final: 16
PS C:\Users\DELL\Proyectos\Microbit-Lenguaje de Interfaz\JuegoDelTopo> .\JuegoDelTopo.exe
Juego iniciado. Tienes 20segundos.
¡Acierto! Puntos: 1 | Tiempo restante: 20s
¡Acierto! Puntos: 2 | Tiempo restante: 19s
¡Acierto! Puntos: 3 | Tiempo restante: 18s
¡Acierto! Puntos: 4 | Tiempo restante: 17s
¡Acierto! Puntos: 5 | Tiempo restante: 16s
¡Acierto! Puntos: 6 | Tiempo restante: 16s
¡Acierto! Puntos: 7 | Tiempo restante: 15s
¡Acierto! Puntos: 8 | Tiempo restante: 14s
¡Acierto! Puntos: 9 | Tiempo restante: 13s
¡Acierto! Puntos: 10 | Tiempo restante: 12s
¡Acierto! Puntos: 11 | Tiempo restante: 12s
¡Acierto! Puntos: 12 | Tiempo restante: 11s
¡Acierto! Puntos: 13 | Tiempo restante: 10s
¡Acierto! Puntos: 14 | Tiempo restante: 9s
¡Acierto! Puntos: 15 | Tiempo restante: 8s
¡Acierto! Puntos: 16 | Tiempo restante: 8s
¡Acierto! Puntos: 17 | Tiempo restante: 7s
¡Acierto! Puntos: 18 | Tiempo restante: 6s
¡Acierto! Puntos: 19 | Tiempo restante: 5s
¡Acierto! Puntos: 20 | Tiempo restante: 4s
¡Acierto! Puntos: 21 | Tiempo restante: 4s
¡Acierto! Puntos: 22 | Tiempo restante: 3s
¡Acierto! Puntos: 23 | Tiempo restante: 2s
¡Acierto! Puntos: 24 | Tiempo restante: 1s

¡Tiempo terminado! Puntaje final: 24

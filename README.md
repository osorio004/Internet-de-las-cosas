# Internet-de-las-cosas
Diagram.JSON
{
  "version": 1,
  "author": "Juan Camilo Osorio Grisales",
  "editor": "wokwi",
  "parts": [
    { "type": "wokwi-arduino-uno", "id": "uno", "top": 0, "left": 0, "attrs": {} },
    { "type": "wokwi-led", "id": "ledVerde", "top": -150, "left": 150, "attrs": { "color": "limegreen", "label": "D2 Verde" } },
    { "type": "wokwi-led", "id": "ledRojo", "top": -150, "left": 230, "attrs": { "color": "red", "label": "D3 Rojo" } },
    { "type": "wokwi-resistor", "id": "r1", "top": -70, "left": 120, "rotate": 90, "attrs": { "value": "220" } },
    { "type": "wokwi-resistor", "id": "r2", "top": -70, "left": 200, "rotate": 90, "attrs": { "value": "220" } }
  ],
  "connections": [
    [ "uno:2", "r1:2", "green", [] ],
    [ "r1:1", "ledVerde:A", "green", [] ],
    [ "ledVerde:C", "uno:GND.1", "black", [] ],
    [ "uno:3", "r2:2", "red", [] ],
    [ "r2:1", "ledRojo:A", "red", [] ],
    [ "ledRojo:C", "uno:GND.1", "black", [] ]
  ],
  "dependencies": {}
}
SKETCH.INO
/*
  Control de dos LED por comandos seriales (UART) a 9600 baudios.
  Proyecto: Uso práctico de Arduino - Comunicación Bluetooth (Entrega 2).

  Conexiones:
    - LED verde -> D2 (con resistencia de 220 ohm a GND)
    - LED rojo  -> D3 (con resistencia de 220 ohm a GND)
    - Módulo HC-05 (montaje físico): TXD -> D0 (RX), RXD -> D1 (TX)
      (el RXD del HC-05 requiere divisor de tensión 1k/2k)
    - En la simulación (Wokwi), el monitor serial reemplaza al
      celular o terminal Bluetooth: el Arduino solo recibe bytes por UART.

  Comandos:
    V -> enciende el LED verde y apaga el rojo
    R -> enciende el LED rojo y apaga el verde
    A -> enciende ambos LED
    0 -> apaga ambos LED
    Cualquier otro carácter -> notifica comando inválido y no cambia los LED
*/

const int LED_VERDE = 2;   // pin digital D2
const int LED_ROJO  = 3;   // pin digital D3

// Aplica el estado a los LED y lo informa por el puerto serie
void aplicar(int verde, int rojo, const char* mensaje) {
  digitalWrite(LED_VERDE, verde);
  digitalWrite(LED_ROJO, rojo);
  Serial.println(mensaje);
}

void setup() {
  pinMode(LED_VERDE, OUTPUT);   // LED como salida
  pinMode(LED_ROJO, OUTPUT);
  Serial.begin(9600);           // misma velocidad por defecto del HC-05
  Serial.println("Listo: V, R, A, 0");
}

void loop() {
  if (Serial.available() > 0) {          // ¿llegó un byte?
    char c = toupper(Serial.read());     // acepta mayúsculas y minúsculas

    if (c == '\n' || c == '\r') return;  // ignora saltos de línea

    switch (c) {
      case 'V': aplicar(HIGH, LOW,  "OK: verde");    break;
      case 'R': aplicar(LOW,  HIGH, "OK: rojo");     break;
      case 'A': aplicar(HIGH, HIGH, "OK: ambos");    break;
      case '0': aplicar(LOW,  LOW,  "OK: apagados"); break;
      default:                               // validación de comandos incorrectos
        Serial.print("ERROR: comando invalido ");
        Serial.println(c);
    }
  }
}

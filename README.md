# Práctica 8: Comunicación Serie (UART)

Participantes Pascual Deza - Martí Vila 

## Introducción

En esta práctica estudiamos de la comunicación serie asíncrona mediante el protocolo **UART** (Universal Asynchronous Receiver Transmitter), ampliamente utilizado en sistemas embebidos y microcontroladores como la ESP32.

Durante las prácticas anteriores ya se ha estado utilizando UART para la comunicación con el monitor serie (por ejemplo, `Serial.println("...")`), pero en esta ocasión se profundiza en su funcionamiento a bajo nivel, implementando una comunicación bidireccional entre UART0 (PC) y UART2 del ESP32.


## Objetivos de la práctica

* Aprender sobre el funcionamiento de la comunicación UART en la ESP32.
* Implementar un "eco" entre UART0 y UART2.
* Visualizar el intercambio de datos entre terminales conectados mediante UART.
* Identificar la utilidad de UART en conexiones a módulos como GPS o GPRS/GSM.


## Ejercicio 1: Bucle de comunicación UART2

### Descripción

Se establece un bucle de comunicación en el que los datos que llegan al terminal UART0 (RXD0, que está conectado al PC) son enviados a UART2 (TXD2). Al mismo tiempo, la información recibida en UART2 (RXD2) se retransmite de vuelta al terminal UART0, haciendo que la comunicación sea visible en el monitor serie.

### Código fuente (sin modificar):

```c++
#include <Arduino.h>

#define BUF_SIZE 1024
uint8_t data_rx0[BUF_SIZE];
uint8_t data_rx2[BUF_SIZE];

void setup() {
  // Inicializar el monitor serial (UART0)
  Serial.begin(115200);  // UART0 para comunicación con el PC
  
  // Asegúrate de que se está configurando correctamente
  Serial.println("Iniciando la comunicación...");
  
  // Configuración de UART2 usando pines GPIO16 (RXD2) y GPIO17 (TXD2)
  const int RXD2_PIN = 16;
  const int TXD2_PIN = 17;
  Serial2.begin(115200, SERIAL_8N1, RXD2_PIN, TXD2_PIN);  // UART2 con RXD2_PIN y TXD2_PIN
  
  // Confirmación de que UART2 se ha iniciado correctamente
  Serial.println("UART2 configurado y listo para comunicarse.");
}

void loop() {
  // Leer datos de UART0 (RXD0) y reenviar a UART2 (TXD2)
  if (Serial.available()) {
    int len = Serial.readBytes(data_rx0, BUF_SIZE);
    if (len > 0) {
      Serial.println("Datos leídos desde UART0, reenviando a UART2...");
      Serial2.write(data_rx0, len);  // Redirigir datos a UART2
    }
  }

  // Leer datos de UART2 (RXD2) y reenviar a UART0 (TXD0)
  if (Serial2.available()) {
    int len = Serial2.readBytes(data_rx2, BUF_SIZE);
    if (len > 0) {
      Serial.println("Datos leídos desde UART2, reenviando a UART0...");
      Serial.write(data_rx2, len);  // Redirigir datos a UART0
    }
  }
}
```

#### Ejemplo de salida:

```
Iniciando la comunicación...
UART2 configurado y listo para comunicarse.
Datos leídos desde UART0, reenviando a UART2...
Datos leídos desde UART2, reenviando a UART0...
Hola mundo!
```

## Conclusiones

* La práctica permite entender y aplicar el uso del UART en microcontroladores como la ESP32.
* Se ha configurado el UART2 para redirigir la entrada y salida de datos, formando un bucle de eco.
* UART es fundamental en aplicaciones como lectura de módulos GPS o conexión a modems GSM/GPRS, por ejemplo, demostrando su versatilidad y uso extendido en sistemas embebidos.



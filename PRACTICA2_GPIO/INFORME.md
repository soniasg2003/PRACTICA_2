# Practica 2.A: Interrupción por GPIO

## **Introducción**

Una interrupción es una señal que interrumpe la actividad nomral de nuestro microprocesaor y salta a atenderla.
En la primera parte de la segunda práctica realizaremos una interrupción mediante un evento hardware.
En el ESP32, podemos 

## **Material**

- ESP32
- ProtoBoard
- Pulsador
El montaje a realizar es el siguiente:
![](practica2_gpio.PNG)

## **Funcionamiento**

#### **Adjuntar la interrupción a un PIN GPIO**
En el IDE de Arduino, usamos una función llamada attachInterrupt() para establecer una interrupción con base en un pin por pin. La sintaxis es la siguiente:
```cpp
attachInterrupt(button1.PIN, isr, FALLING);
```
#### **Desconectar la interrupción de un GPIO**
La función de separación de interrupciones sigue la sintaxis siguente:
```cpp
detachInterrupt(GPIOPin);
```

#### **Estructura de un pulsador**
 La estructura para el pulsador contiene una variable booleana que mantiene el valor dependiendo de si está accionado o no.
 También contiene una variable (contador) para guardar el número de veces que se ha accionado el pulsador. 
 Finalmente se define un constructor por defecto que será el pin 18 con la variable contable a 0 y sin pulsar.
```cpp
struct Button 
{
  const uint8_t PIN;
  uint32_t numberKeyPresses;
  bool pressed;
};

Button button1 = {18, 0, false};
```
#### **Rutina de interrupción**
Esta función se lleva a cabo quando se produce una interrupción. El booleano retorna un "true" y por lo tanto el contador del pulsador suma uno.
En conclusión, esta acción contará las veces que se acciona el pulsador y que servirá para ver si ha sido accionado o no.

```cpp
void IRAM_ATTR isr() 
{
  button1.numberKeyPresses += 1;
  button1.pressed = true;
}
```

#### **Estructura del Setup**
Se inicializan los pines como entrada y con la función AttachInterrupt se producirá la interrupción.
```cpp
void setup()
{  
  Serial.begin(115200);
  pinMode(button1.PIN, INPUT_PULLUP);
  attachInterrupt(button1.PIN, isr, FALLING);
}
```

### **Código completo**
```cpp
#include <Arduino.h>
/*p2_parte1*/
struct Button {
  const uint8_t PIN;
  uint32_t numberKeyPresses;
  bool pressed;
};

Button button1 = {18, 0, false};

void IRAM_ATTR isr() {
  button1.numberKeyPresses += 1;
  button1.pressed = true;
}

void setup() {
  Serial.begin(115200);
  pinMode(button1.PIN, INPUT_PULLUP);
  attachInterrupt(button1.PIN, isr, FALLING);
}

void loop() {
  if (button1.pressed) {
      Serial.printf("Button 1 has been pressed %u times\n", button1.numberKeyPresses);
      button1.pressed = false;
  }

  //Detach Interrupt after 1 Minute
  static uint32_t lastMillis = 0;
  if (millis() - lastMillis > 60000) {
    lastMillis = millis();
    detachInterrupt(button1.PIN);
     Serial.println("Interrupt Detached!");
  }
}
```
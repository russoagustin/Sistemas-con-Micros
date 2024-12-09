# 8-Entradas y Salidas Digitales.

En un MCU habitualmente un mismo terminal puede tener entre cuatro a ocho funciones diferentes

En el LPC4337 los terminales físicos son totalmente independientes de los puertos GPIO.

---

Pin name:  ej. P2_12  →  puerto 2  |  terminal/pin 12

en la tabla del fabricante buscamos cual GPIO Y BIT  es

P2_12 → GPIO1 [12]  * Bit 12 del GPIO1

# Configuración de Terminales

## Caso de Reset

Cuando ocurre un reset queda configurado

GPIO → Entrada

con resistencia de Pull up

puerto de entrada → inhabilitado.

## Registros SFSPx_b

**x → puerto |   b→ terminal**

Sirve para establecer la conexión eléctrica del pin dentro del micro.

```nasm
bit:          31...8          7     6     5     4      3     2:0
        +----------------------------------------------------------+
        |  RESERVADOS      | ZIF | EZI | EHS | EPUN | EPD |  MODE  |
        +----------------------------------------------------------+
```

**MODE**: selecciona la función del pin (0x0 a 0x7) |  **0x0 = GPIO**

**EPD**: Habilita o deshabilita Resistencia de pull down → **habilitado = 1**

**EPUN**: Habilita o deshabilita Resistencia de pull up → **habilitado = 0**

**EHS**:  0 → Salida de pendiente media |  1 → Salida de alta velocidad

**EZI**: 0 → Entrada desconectada  |  1 → Entrada conectada

**ZIF**: 0 → Filtro de ruido habilitado  |  1 → Sin filtro de ruido

## Puertos de Entrada / Salida GPIO

El procesador soporta hasta 8 puerto de GPIO

Cada puerto GPIO puede tener hasta 32 líneas 

El uC LPC4337 posee 83 líneas de entrada/salida digitales

Este tiene 5 puertos GPIO

- Los puertos GPIO0 A GPIO3  tienen 16 líneas cada uno.
- El puerto GPIO4 tiene una sola.
- El puerto GPIO5 tiene 18 líneas.

## Registros de Control y Estado de líneas digitales

Hay un registro de estos **por cada puerto GPIO.**

**GPIO_DIR**: Determina si la línea es entrada o salida. Entrada = 0  Salida = 1   

**GPIO_PIN**: Devuelve el estado actual de cada línea y fija el mismo en las salidas.

**GPIO_MASK**: Almacena la mascara para operaciones en bits particulares en el puerto.

**GPIO_MPIN**: Realiza una lectura o escritura solo de los
pines enmascarados del puerto

**GPIO_SET**: Fija las salidas en un valor alto. 

**GPIO_CLR**: Fija las salidas en un valor bajo. 

**GPIO_NOT**: Cambia las salidas al valor contrario al valor actual.
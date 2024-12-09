# 7-Interrupciones Resumen

## NVIC (Nested vector Interrupt priority)

Es un dispositivo externo al procesador encargado de manejar las prioridades.

permite enmascarar cada dispositivo que pide la interrupción.

Está incorporado en la misma pastilla que el CPU.

![image.png](7-Interrupciones%20Resumen%2013ce88c4d1e2804c9859e44485ec95e7/image.png)

Puede manejar hasta 240 priorizables. 

Hay 3 con prioridad fija.

## Tipos de Excepciones

ARM diferencia 2 tipos internas y externas

### Internas

Son causadas por el núcleo ARM

Se les llama *Traps*

Son en total 15

Las primeras 3 Tienen prioridad fija

1. Reset. → Prioridad = -3
2. NMI. → Prioridad = -2
3. Hard Faul (falla de Hardware no especificada) → Prioridad = -1

Las siguientes Traps son para otros tipos de *Faults* como Hard Faul

SVCall: Pedidos al SO

SysTick Interrupt: timer interno del núcleo de ARM

### Externas

El NVIC puede manejar hasta 255 excepciones, de las cuales 15 son internas (traps) por lo tanto quedan hasta 240 interrupciones, A las cuales le podemos modificar su prioridad

Cada excepción se identifica con un número, a veces utilizamos número de excepción y otras de interrupción.

#Interrupción = #Excepción -16

## Vector de interrupciones VTOR

Es un vector donde en la posición correspondiente al número de excepción tiene la dirección de memoria de la rutina de servicio de dicha interrupción.

```nasm
.section .isr // Define una seccion especial para el vector
.word stack // 0: Initial stack pointer value
.word reset+1 // 1: Initial program counter value
.word handler+1 // 2: Non mascarable interrupt service routine
.word handler+1 // 3: Hard fault system trap service routine
.word handler+1 // 4: Memory manager system trap service routine
.word handler+1 // 5: Bus fault system trap service routine
.word handler+1 // 6: Usage fault system tram service routine
.word 0 // 7: Reserved entry
.word 0 // 8: Reserved entry
.word 0 // 9: Reserved entry
.word 0 // 10: Reserved entry
.word handler+1 // 11: System service call trap service routine
.word 0 // 12: Reserved entry
.word 0 // 13: Reserved entry
.word handler+1 // 14: Pending service system trap service routine
.word handler+1 // 15: System tick service routine
.word handler+1 // 16: IRQ 0: DAC service routine
.word handler+1 // 17: IRQ 1: M0APP service routine
.word handler+1 // 18: IRQ 2: DMA service routine
.word 0 // 19: Reserved entry
.word handler+1 // 20: IRQ 4: FLASHEEPROM service routine
.word handler+1 // 21: IRQ 5: ETHERNET service routine
.word handler+1 // 22: IRQ 6: SDIO service routine
.word handler+1 // 23: IRQ 7: LCD service routine
.word handler+1 // 24: IRQ 8: USB0 service routine
.word handler+1 // 25: IRQ 9: USB1 service routine
.word handler+1 // 26: IRQ 10: SCT service routine
.word handler+1 // 27: IRQ 11: RTIMER service routine
.word timer_isr+1 // 28: IRQ 12: TIMER0 service routine
.word handler+1 // 29: IRQ 13: TIMER1 service routine
.word handler+1 // 30: IRQ 14: TIMER2 service routine
.word handler+1 // 31: IRQ 15: TIMER3 service routine
.word handler+1 // 32: IRQ 16: MCPWM service routine
.word handler+1 // 33: IRQ 17: ADC0 service routine
.word handler+1 // 34: IRQ 18: I2C0 service routine
.word handler+1 // 35: IRQ 19: I2C1 service routine
.word handler+1 // 36: IRQ 20: SPI service routine
.word handler+1 // 37: IRQ 21: ADC1 service routine
.word handler+1 // 38: IRQ 22: SSP0 service routine
.word handler+1 // 39: IRQ 23: SSP1 service routine
.word handler+1 // 40: IRQ 24: USART0 service routine
.word handler+1 // 41: IRQ 25: UART1 service routine
.word handler+1 // 42: IRQ 26: USART2 service routine
.word handler+1 // 43: IRQ 27: USART3 service routine
.word handler+1 // 44: IRQ 28: I2S0 service routine
.word handler+1 // 45: IRQ 29: I2S1 service routine
.word handler+1 // 46: IRQ 30: SPIFI service routine
.word handler+1 // 47: IRQ 31: SGPIO service routine
.word handler+1 // 48: IRQ 32: PIN_INT0 service routine
.word handler+1 // 49: IRQ 33: PIN_INT1 service routine
.word handler+1 // 50: IRQ 34: PIN_INT2 service routine
.word handler+1 // 51: IRQ 35: PIN_INT3 service routine
.word handler+1 // 52: IRQ 36: PIN_INT4 service routine
.word handler+1 // 53: IRQ 37: PIN_INT5 service routine
.word handler+1 // 54: IRQ 38: PIN_INT6 service routine
.word handler+1 // 55: IRQ 39: PIN_INT7 service routine
.word handler+1 // 56: IRQ 40: GINT0 service routine
.word handler+1 // 56: IRQ 40: GINT1 service routine
```

## Registros Especiales (CPU)

---

Dentro del Registro de control Existe el IPSR (*Interrupt Program Status Register*)

Este registro guarda el número de interrupción activa. Este valor permite al sistema conocer qué excepción o interrupción está siendo atendida actualmente.

El IPSR ocupa 9 bits dentro de un registro de 32 bits

```nasm
	  Program Estatus Register		 
+---------------------------------+
|                        |  IPSR  |
+---------------------------------+
#bit									    8       0
```

### PRIMASK

Registro especial que nos permite habilitar o deshabilitar todas las interrupciones enmascarables, es decir, aquellas con prioridad inferior a la de las excepciones críticas NMI o Hard Fault, que no pueden ser enmascaradas.

- Tamaño:  1 bit
- Función
    - PRIMASK = 0 → Todas las interrupciones están habilitadas (por defecto).
    - PRIMASK = 1 → Todas las interrupciones enmascarables están deshabilitadas.

Para modificar este registro existen dos instrucciones

**CPSIE I** → Clear PRIMASK, (Habilita las interrupciones).

**CPSID I** → Set PRIMASK, (Inhabilita las interrupciones.)

### BASEPRI

Registro especial que nos permite establecer un umbral de prioridad para las interrupciones. Su función es deshabilitar todas las interrupciones con una prioridad igual o inferior al valor configurado

- Tamaño: 8 bits
- Función:
    - Cuando BASEPRI contiene un valor distinto de cero, el procesador bloque todas las interrupciones cuya prioridad es **igual o mayor** (Refiriéndose al número) al de BASEPRI.
    - Si se establece en 0, **Todas las interrupciones están habilitadas,** excepto aquellas deshabilitadas por el registro PRIMASK.

Para modificar este registro se utiliza la instrucción MSR

```nasm
/*GUARDA EN BASEPRI EL VALOR DE R0*/
MOV R0,#PRIORIDAD 
MSR BASEPRI, R0
```

Para obtener el valor actual de BASEPRI se utiliza la instrucción MRS

```nasm
/*GUARDA EN R0 EL VALOR DE BASEPRI*/
MOV R0,#PRIORIDAD 
MRS BASEPRI, R0
```

---

## Registros para habilitar interrupción individualmente. (NVIC)

Estos registros son parte del NVIC estos están mapeados a direcciones de memoria.

### ISER _x

**ISER _x, x= 0,1,… ***Interrupt set enable register*  Cada uno de estos registros **permite habilitar 32 interrupciones, donde cada bit corresponde a una interrupción**.

- Función: Escribiendo un 1 en el bit n del registro se habilita la interrupción #n.

### ICER _X

**ICER _x, x=0,1,…** *Interrupt Clear Enable Register* Cada uno permite deshabilitar 32 interrupciones, donde cada bit corresponde a una interrupción.

- Función: Escribiendo un 1 en el bit n del registro se deshabilita la interrupción #n.

## Registros de Prioridad para Interrupciones.

### **IPR_ x**

**IPR _x, x=1,2,… -** *Interrupt Priority Registers x* 

Establece la prioridad de una interrupción, están divididos en campos de 8 bits, cada campo determina una prioridad.

La mayor prioridad es 0.

#reg = #IRQ/4  Ya que cada registro tiene 4 Bytes

#Byte = #IRQ Mod(4)

Se emplean los 3 bits más significativos del campo.

El resto se puede configurar como subnivel.

## Registros de Prioridad para Traps

### **SHPR_ x**

SHPR_ x, x=1,2,… - *System Handler Priority Registers* 

Mapeados a direcciones de memoria pero prácticamente parte del CPU ya que están incluidos en el bloque de System Control Space, que es un área de memoria dedicada para la configuración de excepciones y funciones del sistema.

Son 3 registros 

Guardan la prioridad en un Byte

Se utilizan los 3 bits mas significativos

## Registros de Pendientes.

### ISPR_ n

**ISPR _n, n=1,2,…** *Interrupt Set-Pending Registers ,* forman parte del NVIC

Permite establecer una interrupción como pendiente, al escribir un 1 en el bit correspondiente a una interrupción en el registro ISPR _x el sistema marca esa interrupción como pendiente, lo que hace que el NVIC programe su atención cuando sea posible (de acuerdo a las prioridades y si las interrupciones están habilitadas.) 

**Sirve para provocar una interrupción por software.**

Cuando se atiende la interrupción el bit se establece en 0 automáticamente.

### ICPR_ n

ICPR_ n, n=1,2,… *Interrupt Clear-Pending Registers,* también forman parte del NVIC

Permite eliminar una interrupción pendiente para que no se ejecute su rutina de servicio. Cada bit corresponde a un numero de interrupción 

Consultar: Estos registros del NVIC contienen todas las excepciones o solo para las interrupciones (#excepción >15)

# SysTick Timer

Es un temporizador de propósito general integrado en los núcleos ARM Cortex-M 

Características:

- **Tamaño**: Es un contador de 24 bits
- **Frecuencia de reloj:** puede configurarse para usar el mismo reloj del sistema o un reloj de referencia más lento, según la configuración del microcontrolador.
- **Recarga automática:** Al alcanzar el valor cero, el contador se recarga automáticamente con el valor configurado, reiniciando el ciclo sin necesidad de intervención manual.

## Registros del SysTick.

### SYST_CSR **Control Status Register**

```nasm
bit:     31-24   23-17    16	   15-3       2         1        0
        +----------------------------------------------------------+
        |  0  |    0    |COUNT|    0    | CLK_SRC | INTEN | ENABLE |
        +----------------------------------------------------------+
```

**ENABLE:** Para arrancar el systick | 1 = enable

**ITEN:** Habilita las interrupciones con 1

**CLK_SRC:** Para elegir que clock utiliza

**COUNT:** Es un bit de solo lectura, si se lo escribe no hace nada. se pone en 1 cuando el contador llega a 0.

### SYST_RVR Reload Value Register

Es un registro q utiliza 24 bits para el valor de recarga del contador.

### SYST_CVR Current Value Register

Registro de 24 bits que contiene el valor actual del contador

Al escribir cualquier valor  lo pone en 0.

---

### Inicialización: 5 pasos

ENABLE = 0  en **CSR**

Escribir el valor de RELOAD  en **RVR**

Resetear el Contador.  Se escribe cualquier valor en **CVR**

CLK_SRC = 1 en **CSR**

Que cuente → ENABLE = 1 y habilitar interrupción con INTEN =1 en **CSR**

**Ejemplo**

```nasm
systick_init:
 // Inhabilitar SysTick antes de configurar
 LDR R1,=SYST_CSR
 MOV R0,#0
 STR R0,[R1] // Quitar ENABLE
 // Configurar el desborde para un periodo de 0,1 Segundos
 LDR R1,=SYST_RVR
 LDR R0,=#(4800000-1)
 STR R0,[R1] // Especificar valor RELOAD
 // Incializar el valor actual del contador
 // Escribir cualquier valor en CVR limpia el contador
 LDR R1,=SYST_CVR
 STR R0,[R1] // Clear COUNTER y flag COUNT
 // Habilitar el SysTick con el reloj del nucleo
 LDR R1,=SYST_CSR
 MOV R0,#0x05
 STR R0,[R1] // Fijar ENABLE y CLOCK_SRC, sin INTEN
 BX LR
```

---

## Latencia de la Interrupción

Es el tiempo que demora el CPU desde que llega el pedido hasta que lo atiende, suponiendo que el pedido no quedó pendiente.

En Cortex-M4, en general la latencia será de 12T, puede llegar a 23T si se utiliza la unidad de punto flotante.

Si hay anidamiento también será de 12T, pero puede ser menor en interrupciones simultáneas
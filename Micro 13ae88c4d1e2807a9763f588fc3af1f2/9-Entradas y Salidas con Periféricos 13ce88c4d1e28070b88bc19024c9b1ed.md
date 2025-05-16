# 9-Entradas y Salidas con Periféricos

## Interrupciones Grupales en GPIO

Se puede crear una Interrupción a partir de varios pines de un GPIO.

puede ser un OR o AND entre los pines para generar el pedido.

Se pueden armar hasta dos grupos por GPIO. Cada grupo un único pedido.

**Detección de eventos**: Las interrupciones grupales permiten configurar el tipo de evento que dispara la interrupción, tales como:

- Flanco ascendente
- Flanco descendente
- Nivel bajo o alto

## Interrupciones Individuales en GPIO

Se pueden configurar hasta 8 pedidos distintos de Interrupción

Cada interrupción se puede asociar a una línea cualquiera de GPIO

**Detección de eventos**: Las interrupciones  permiten configurar el tipo de evento que dispara la interrupción, tales como:

- Flanco ascendente
- Flanco descendente
- Nivel bajo o alto

## Registros de Configuración

### Selección de la fuente de interrupción

Para  seleccionar la fuente hay 2 registros por GPIO, con estos configuramos las fuentes de interrupción de los pines GPIO, estos seleccionan qué pin específico del GPIO se utilizará para generar una interrupción.

**PINTSEL0**  (pertenecen al SCU)

Para las interrupciones 0 a 3 

**PINTSEL1**

Para las interrupciones 4 a 7

Se ocupa un Byte por cada fuente interrupción 

```nasm
                            ... |      INT 1     |     INT 0      |
+-----------------------------------------------------------------+
|           ... |**PORTSEL1**|**INTPIN1**|**PORTSEL0**|**INTPIN0**|
+-----------------------------------------------------------------+
BIT:            |15        13|12        8|7          5|4        0 |
```

**INTPIN n**  N° de bit dentro del GPIO 0 (0 a 32)

**PORTSEL n** N° de GPIO (0 a 7)

---

*—-—-Los siguientes registros pertenecen al GPIO no al SCU—————*

### ISEL - Selección de Flanco o Nivel

Existe un registro **ISEL** para configurar las interrupciones por flanco o nivel

Se usan solo 8bits, uno por cada interrupción.

Cada bit indica el modo de funcionamiento:

- 0 → Operación por **flanco**.
- 1 → Operación por **nivel** (mientras esté en nivel alto al terminar la ISR se va a mandar una nueva interrupción).

Chat GPT————————————————————————————-

El valor de cada campo en el registro **ISEL** determina el tipo de evento que generará la interrupción. Por lo general, los valores posibles son:

- **00**: Flanco ascendente (rising edge).
- **01**: Flanco descendente (falling edge).
- **10**: Nivel bajo (low level).
- **11**: Nivel alto (high level).

Entonces es un bit o dos bit

---

### SIENR - Set Interrupt Enable

… Continuar con los registros de configuración

## Métodos de E/S

---

### Polling (Encuestas)

Consiste en testear permanentemente una señal de status, hasta que esta tome un valor. 

- Requiere que el programa principal se ocupe de testear.
- Puede testear continuamente y no hacer nada más.
- Puede testear periódicamente sin que pare el programa principal.
- Si está bien diseñado, atiende más rápido una señal.

### Interrupción

Cuando la señal de status toma un valor, el programa se interrumpe para manejar la E/S.

- Mucho más complicado de reproducir para probar.
- El programa principal está libre para realizar otras tareas.
- No hay desperdicio de tiempo testeando.
- Mayor sobrecarga (overhead) pero solo cuando es necesario.

---

## Conceptos

### Latencia de Interrupción:

- 12T Tiempo total de cambio de Contexto, incluye el vaciado del pipeline.
- 10T Retorno de Interrupción

*Lo anterior es si no estoy utilizando punto flotante.

Si hay un pendiente mientras se ejecuta una rutina de servicio, cuando termine la rutina de servicio actual y pase a la otra, no se recupera el estado del programa  principal y vuelve a cambiar de contexto, sino que directamente pasa a la rutina de servicio de la interrupción pendiente por lo tanto se tiene una **latencia de 0**

# Tasa de Transferencia

Si el periférico es mas lento que el CPU por lo tanto limita la tasa de transferencia → **I/O bounded** 

Si el mas lento el CPU, este limita la tasa de transferencia → **CPU bounded**

Supondremos que si estamos en **Polling nos lleva 16T** transferir un Byte.

Y que en **Interrupción nos lleva 53T** (overhead)

## Tasa Máxima de Transferencia

La tasa máxima se da cuando el dispositivo puede transferir datos al MCU sin perder ninguno.

El tiempo que tarda el MCU en procesar un datos es **t = To + Td**

- **To** → Tiempo de Dato, es el tiempo para poner un dato en memoria (idealmente un ciclo de M → 2T)
- **Td** → Tiempo de overhead, es el tiempo que se tarda en detectar la señal, saltar al código adecuado, manejar la línea READY, mantener los punteros, buscar instrucciones, etc

### Tasa máxima en Polling

En caso de tasa máxima, el lazo de detección solo se ejecuta una vez por cada dato que llega.

Ejemplo 

```nasm
//*********LAZO POLLING************
Pol0 STR R6,[R4] //2T
Pol1 LDR R5,[R3] //2T
TST R5,#1 //1T
BNE Poll //1T
STR R6,[R3] //2T
STR R7,[R4] //2T
LDRB R5,[R0] //2T
STRB R5,[R1],#1 //2T
SUBS R2,#1 //1T
BGE Pol0 //1T 
```

**Td = 4T** (por el LOAD y STORE)

**To = 12T** (es el total menos Td)

total = 16T

→ Eficiencia = 4/16 = 25% (el 25% es util (Transferir datos) el resto desperdicio)

Si F(bus) = 32MHz

T = 31,25 ns

Entonces podemos calcular los Bytes por segundos

**16T→1B   |    1T → 31,25ns    |    16T = 500ns**

500ns → 1Byte

1s → **x= 2x10^6 = 2MB/s**

### Tasa Máxima con Interrupciones

con tasa máx. termina una rutina de interrupción y comienza otra (tail-chain).

Con Corex-M4 latencia IRQ: 6 ciclos. → **tail-chain = 6T**

```nasm
In_I LDR R0,=IST     //2T Puntero IST
MOVE R1,#1           //1T
STR R1,[R0]          //2T IST[0]=0 – no cambia el resto
LDR R3,=Ready        //2T Puntero Ready
MOVE R2,#0           //1T
STR R2,[R3]          //2T RDY=0
LDR R0,=MPIN_GPIO_0  //2T
LDRB R2,[R0]         //2T Ingresa Datos
LDR R0,=puntero      //2T R0 dirección puntero
LDR R1,[R0]          //2T R1 apunta a Datos
STRB R2,[R1],#1      //2T guardar dato e incr puntero
STR R2,[R0]          //2T guardar puntero
LDR R0,=contador     //2T apunta a contador
LDR R2,[R0]          //2T
SUBS R2,#1           //1T decr. Contador y cambia flags
STR R2,[R0]          //2T guardar contador
IT NEQ               //1T calcula condición
STRBNEQ R1,[R3]      //2T Rdy=1 (no se cuenta su tiempo?)
Fin: BX LR           //1T (predice) Retorna
```

Duración **31T** en forma sostenida. No sostenida: sumar 12+10T → 53T.

Los tiempos suponiendo que solo atiendo una instrucción tras otra son:

**Td = 4T** 

**To = 33T** overhead (37T - 4T)

total = 31T - (total de T de la rutina de servicio) + 6T (tail-chain)

**total = 37T**

Si F(bus) = 32MHz

T = 31,25 ns

37T = 1,156 us

Tasa máxima = 0,86 MB/s

Eficiencia = 4 / 37 = **10,81%**

## Modo Bloque

En una interrupción no transfieren un dato, sino que transfieren **n datos**

aumentando así la eficiencia y tasa de transferencia. se paga con latencia.

**Implementación:**

En la ISR se escribe un lazo para todo el bloque (Similar al de Polling). Se ejecuta n veces, n= cantidad de datos.

**Consideraciones calculo de performance:**

Todo lo que se ejecuta una única vez es despreciable frente a n veces del lazo

resultado igual al poling

**Desventaja**: ISR largas.

## Multiples dispositivos lentos.

Sean n dispositivos que transfieren datos

**suponiendo:**

**Todos a igual tas: fdisp**

**todos consumen igual tiempo de CPU Td = D**

**Performance con multiples dispositivos:**

Sin interrupciones T = Tprog-prin.

Con interrupciones T’ = Tprog-prin + n * T disp. 

![image.png](9-Entradas%20y%20Salidas%20con%20Perife%CC%81ricos%2013ce88c4d1e28070b88bc19024c9b1ed/image.png)

**Performance del programa : T /  T’ = 1 - n * (T disp. / T’) = 1-n * D *f disp.**

# Consultar diapo 41 tema 9, no entiendo nada.

**Ejemplo:**

Para interrupciones:

MCU a 32 MHz y 20 dispositivos de igual tasa de transferencia = 10KB/s

En el mejor caso: todas las interrupciones llegan juntas.

- f disp. :10 KB/s → 10KHz
- T disp. : 37 T → 1,15us
- T / T’ = 1 - n * f disp. * Tdisp

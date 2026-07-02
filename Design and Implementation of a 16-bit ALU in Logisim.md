# Diseño e Implementación de una ALU de 16 bits en Logisim

## i) Sumador/Restador Base de 16 bits (Complemento a 2)

### Full_Adder_1bit (Sumador Completo de 1 bit)

Sumar 2 bits presenta cuatro escenarios posibles:

1. `0 + 0 = 0`
2. `1 + 0 = 1`
3. `0 + 1 = 1`
4. `1 + 1 = 10`

Cuando se suman dos bits, una sola posición de bit solo puede almacenar un valor máximo de 1. Para preservar el valor posicional de la operación, debemos tener en cuenta el desbordamiento matemático (*overflow*). Al almacenar y transmitir una señal de Acarreo (*Carry*) al siguiente bit más significativo en la cadena del sumador, el sistema propaga con éxito este desbordamiento a lo largo de todo el cálculo, evitando la pérdida de datos.

Para resolver esto, el `Full_Adder_1bit` realiza las siguientes operaciones booleanas:

* `Sum = A XOR B XOR Cin`
* `Cout = (A AND B) OR (Cin AND (A XOR B))`

Esto se puede lograr con el hardware que se muestra a continuación:

* 3 pines de 1 bit
* 2 compuertas XOR
* 2 compuertas AND
* 1 compuerta OR

### AdderSubtractor_1bit (Sumador/Restador de 1 bit)

Realizar tanto la suma como la resta en hardware requiere matemáticas binarias. Los circuitos digitales restan sumando el complemento a 2 del segundo número al primero. La regla para el complemento a 2 es: Invertir los bits del número que se desea restar (NOT) y luego sumar 1.

El `AdderSubtractor_1bit` utiliza una capa de inversión antes de cualquier operación aritmética. Usa una señal de control (`Op`) para decidir qué hacer con la entrada `B`:

* Cuando `Op = 0` (Suma): La entrada B pasa tal como se introdujo.
* Cuando `Op = 1` (Resta): La entrada B se invierte (NOT).

Para resolver esto, el `AdderSubtractor_1bit` realiza las siguientes operaciones booleanas:

* `B_internal = B XOR Op`
* `Sum = A XOR B_internal XOR Cin`
* `Cout = (A * B_internal) + (Cin * (A XOR B_internal))`

Esto se puede lograr con el hardware que se muestra a continuación:

* 4 pines de entrada de 1 bit (A, B, Cin, Op)
* 2 pines de salida de 1 bit (Sum, Cout)
* 1 compuerta XOR (La Capa de Inversión)
* 1 subcircuito Full_Adder_1bit (El Motor Matemático)

### AdderSubtractor_16bit (Sumador/Restador de 16 bits)

Para procesar enteros de 16 bits, debemos conectar en cascada múltiples operadores de 1 bit en una arquitectura unificada.

Para resolver esto, el `AdderSubtractor_16bit` utiliza una arquitectura de "Acarreo en Cascada" (*Ripple Carry*). Al conectar dieciséis módulos `AdderSubtractor_1bit` en serie, el sistema permite que el Acarreo de salida (`Cout`) de un bit se propague físicamente hacia el Acarreo de entrada (`Cin`) del siguiente, preservando el valor posicional en los 16 bits. Además, para completar el requisito de "+ 1" de la resta en complemento a 2, la señal de control `Op` se enruta directamente al `Cin` del primerísimo bit (Bit 0).

Esto se puede leer como:

* Para cada bit *i* (de 0 a 15): `Result[i] = A[i] ± B[i]`
* `Cin del Bit 0 = Op` (Introduce el +1 requerido al restar)
* `Cin del Bit i = Cout del Bit i-1` (El Acarreo en Cascada)

Esto se puede lograr con el hardware que se muestra a continuación:

* 2 pines de entrada de 16 bits (A, B)
* 1 pin de entrada de 1 bit (Op)
* 1 pin de salida de 16 bits (Result)
* 2 Divisores (*Splitters*) (de 16 bits de ancho) para desempaquetar y reempaquetar los buses de datos
* 16 AdderSubtractor_1bit

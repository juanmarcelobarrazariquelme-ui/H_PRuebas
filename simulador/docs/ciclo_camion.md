# Contrato del ciclo de un camión

## Escenario didáctico

Desde E1, el despachador asigna P1. Ruta de ida vacío: E1–T1–I1–T3–I2–R1–P1. Los tiempos nominales por tramo son 5, 10 y 4 ticks, total 19 ticks.

Una vez que obtiene posición en P1, el camión espera FIFO y realiza la carga en una sola posición de servicio. El tiempo se muestrea según una lognormal de media 4,41 y desviación 2,54; las toneladas, mediante lognormal de media 331 y desviación 16,06. Para ilustrar la cronología, se utiliza una carga de 6 ticks y 330 toneladas, sin afirmar que son un muestreo ejecutado.

El despachador asigna S1 **antes** de liberar la posición en P1. La ruta de regreso cargado es P1–R1–I2–T3–I1–T2–S1. Sus tiempos son ceil(4×1,3)=6, ceil(10×1,3)=13 y ceil(8×1,3)=11, total 30 ticks. La descarga se modela con una lognormal de media 1,84 y desviación 1,38; el ejemplo usa 2 ticks. Sin espera adicional, el ciclo suma 19+6+30+2=57 ticks.

## Secuencia causal de recursos

1. Solicitar destino y ruta desde estacionamiento.
2. Entrar/salir de T1 y T3 en orden FIFO; adquirir R1 para subir.
3. Obtener una plaza en P1 antes de liberar R1; registrar llegada a cola.
4. Esperar servicio, comenzar y completar la carga.
5. Obtener asignación a S1 y asegurar R1 en dirección de bajada antes de abandonar P1 (o usar zona de salida explícita si se eligió esa alternativa).
6. Recorrer R1, T3 y T2 cargado, manteniendo la disciplina FIFO.
7. Ingresar a la cola ilimitada de S1, descargar y contabilizar solo si se termina dentro de 720 ticks.
8. Pedir nueva pala antes de liberar la posición de descarga y repetir hasta fin del turno.

## Estados observables

`ViajandoVacío`, `EnColaPala`, `Cargando`, `Transportando`, `EnColaStockpile`, `Descargando`, `Esperando(recurso)` y `EnFalla`. En el ciclo principal, el `select` habilita canales según estado; los casos no aplicables se desactivan mediante canales `nil`. Los tiempos simulados no requieren sincronización global entre camiones.

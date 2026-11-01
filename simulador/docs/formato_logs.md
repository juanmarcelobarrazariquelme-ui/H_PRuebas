# Especificación del formato CSV de eventos

## Contrato

Formato UTF-8, separador coma, encabezado uniforme, sin estructuras anidadas. Todos los campos no aplicables usan `na` de forma explícita. Cada entidad escribe su archivo de manera exclusiva. Un registro CSV es una observación de un evento desde el punto de vista de una entidad y, por tanto, tiene su propio `seq`.

```csv
seq,tick,evento,entidad,contraparte,ubicacion,estado,recurso,recursos_retenidos,toneladas,evento_relacionado,detalle
```

| Campo | Regla |
|---|---|
| seq | Entero positivo global único, estrictamente creciente en el orden de las transiciones confirmadas |
| tick | Entero de 0 a 720 en que se constata el evento |
| evento | Nombre normalizado en mayúsculas con `_` |
| entidad | Identificador de camión, pala o despachador autor del registro |
| contraparte | Entidad participante, si corresponde; `na` en otro caso |
| ubicacion | Nodo o `TRAMO:origen>destino` |
| estado | Estado actual del camión; `na` en archivos de otras entidades |
| recurso | Identificador del recurso de interés |
| recursos_retenidos | Lista simple separada por `+`, ordenada lexicográficamente, o `na` |
| toneladas | Número con punto decimal o 0 cuando el evento no contabiliza mineral |
| evento_relacionado | ID de correlación común entre archivos para una misma operación |
| detalle | Texto plano breve, sin comas, con información contextual verificable |

## Orden y atomicidad

El contador global de secuencia se asigna **al confirmar cada transición en su sección crítica**, no posteriormente al imprimirla. Los eventos relacionados entre entidades reciben secuencias distintas y comparten `evento_relacionado`. El log de una misma entidad se ordena por `seq`. El consumidor del Certamen III intercala los archivos por `seq`; un empate indicaría un error de formato. La fecha/tick por sí sola no permite reconstruir el orden de eventos simultáneos.

## Catálogo de eventos

`ASIGNACION_PALA`, `ASIGNACION_STOCKPILE`, `CAMBIO_ESTADO`, `ENTRADA_TRAMO`, `SALIDA_TRAMO`, `ADQUIRIR_RAMPA`, `LIBERAR_RAMPA`, `RESERVA_PALA`, `CANCELAR_RESERVA`, `INGRESO_COLA_PALA`, `INGRESO_COLA_STOCKPILE`, `INICIO_CARGA`, `FIN_CARGA`, `INICIO_DESCARGA`, `FIN_DESCARGA`, `ADQUIRIR_LUGAR_PALA`, `LIBERAR_LUGAR_PALA`, `ESPERA_RECURSO`, `ESPERA_PROLONGADA`, `FALLA_TEMPORAL`, `REPARACION`, `FALLA_PERMANENTE`, `PANIC_RECUPERADO`, `FIN_TURNO`.

## Semántica de toneladas

`FIN_CARGA` registra carga adquirida por el vehículo y `FIN_DESCARGA` registra la carga entregada al stockpile. El total transportado es la suma de descargas completadas dentro de turno; la carga que no se descarga no suma producción.

## Escenarios de ejemplo

- `caso_basico`: dos camiones y una pala, una cronología sin fallas y con secuencias repartidas entre archivos.
- `caso_fallas`: otra ejecución manual independiente, con camiones y pala, fallas, reparación, espera prolongada, reserva/cancelación y recuperación de panic.

Los archivos separados representan trazas de **ejemplo escritas a mano**. Los números de secuencia se reinician únicamente entre ejecuciones distintas, no entre entidades de una misma ejecución. Los eventos están concebidos para ilustrar la semántica que deberán implementar los logs reales.

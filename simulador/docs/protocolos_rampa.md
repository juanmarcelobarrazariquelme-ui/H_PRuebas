# Protocolos de acceso a rampas

## Invariantes

1. Una rampa tiene sentido `libre`, `A→B` o `B→A`.
2. Si tiene ocupantes, todos comparten sentido.
3. Todo ingreso o salida cambia ocupación/sentido de forma indivisible.
4. La salida desde un tramo respeta el orden FIFO de su sentido.
5. Un camión espera sin perder la capacidad de detectar cancelación de turno y de emitir `ESPERA_PROLONGADA`.

## Protocolo ingenuo (pseudocódigo)

```text
AL SOLICITAR una rampa R en sentido S:
    REPETIR hasta obtener acceso o terminar el turno:
        solicitar al controlador permiso atómico de ingreso
        SI R está vacía O el sentido ocupado es S:
            registrar adquisición y sumar ocupante
            autorizar ingreso
        SI vence el límite de espera:
            registrar recurso esperado y recursos retenidos
    recorrer R respetando FIFO
    AL LLEGAR al extremo:
        solicitar plaza del nodo de destino
        SI está lleno:
            mantener R ocupada mientras se espera
            registrar cada espera prolongada definida
        AL OBTENER plaza:
            transferir posición al nodo y liberar R
```

## Por qué puede bloquearse

Una pala llena conserva una plaza de carga y sus espacios de espera; algunos camiones ya están en la rampa subiendo y aguardan una plaza. El camión cargado conserva una plaza de pala mientras pide la rampa de bajada, incompatible con la subida. Los de la rampa esperan la pala y el de la pala espera la rampa. Se forma un ciclo de espera.

## Protocolo de prevención (pseudocódigo de adquisición conjunta)

```text
AL SOLICITAR subir por rampa R hacia pala P:
    enviar una sola solicitud (camión, R, sentido_subida, P)
    el coordinador examina en una misma transición indivisible:
        compatibilidad del sentido de R
        existencia de una plaza no ocupada ni reservada en P
        política de prioridad para las salidas de P
    SI ambas condiciones son aceptables:
        reservar una plaza única de P
        conceder ingreso y ocupar R en el sentido correspondiente
        registrar ambos compromisos y responder autorización
    SI NO:
        no retener reserva parcial alguna
        encolar la solicitud y esperar con límite y cancelación
    AL ARRIBAR a P:
        convertir reserva en ocupación de plaza
        liberar R según FIFO
    SI ocurre cancelación antes de completar:
        deshacer conjuntamente toda reserva/adquisición concedida
```

Un controlador único puede serializar las decisiones sobre la rampa y las reservas de la pala. La operación conjunta no es una comprobación seguida de una reserva independiente; ambos compromisos suceden como una unidad coordinada. Las salidas de la pala deben poder reservar o adquirir la dirección contraria según una política que no deje reservas parciales retenidas por solicitudes de subida. Un controlador debe garantizar progresión cuando desaparezca el conflicto y no permitir que una preferencia interminable de un sentido provoque inanición.

## Alternativa del enunciado

Una zona de salida ilimitada permite al camión finalizar la carga y **liberar primero** su lugar en la pala; luego espera la rampa de bajada fuera de la capacidad limitada. Rompe la condición de retención y espera para ese caso, a costa de añadir un lugar explícito al modelo.

## Condiciones de Coffman

- Exclusión mutua: se mantiene en el lugar de pala y el sentido incompatible de rampa.
- Retención y espera: el diseño previene que una solicitud retenga parcialmente una plaza sin acceso utilizable; la alternativa de zona de salida lo rompe directamente.
- No expropiación: se mantiene normalmente para recursos ya ocupados.
- Espera circular: el controlador evita construir el ciclo pala↔rampa en las admisiones que autoriza; la prueba final depende de que no existan otros ciclos de recursos en el modelo.

## Evidencia de espera prolongada

En ambos protocolos la adquisición admite un límite configurado y se registra al vencer, pero la goroutine sigue esperando o reintenta. Un camión atrapado tras falla permanente no constituye por sí mismo deadlock: espera a otro camión que no está esperando otro recurso. En cambio, el ciclo pala↔rampa del ingenuo sí produce espera circular. Las propiedades de prevención deben comprobarse con escenarios de concurrencia reales.

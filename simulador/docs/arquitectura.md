# Diseño de componentes y comunicación

## Modelo de dominio

La mina es un grafo no dirigido. Cada nodo se identifica por un ID único y almacena tipo (`estacionamiento`, `interseccion`, `pala`, `stockpile`) y, en el caso de palas, capacidad total positiva. Cada tramo posee ID único, extremos existentes, clase (`doble`, `rampa`) y un tiempo base positivo. El grafo se consulta mediante adyacencias en ambos sentidos.

## Componentes concurrentes

| Componente | Estado propio | Entrada / salida | Garantía |
|---|---|---|---|
| Camión (goroutine por camión) | Identidad, ruta inmutable en viaje, estado, ubicación, carga, recursos retenidos | Solicitudes al despachador, órdenes de pala/stockpile, autorizaciones de tramo/rampa | Máquina de estados y `select` con guardas de canales habilitados por estado |
| Pala (goroutine por pala) | Cola FIFO, cupos reservados/ocupados, puesto de carga | Solicitud de ingreso, atención de carga, liberación de cupos | No superar la capacidad; 1 carga a la vez |
| Stockpile (goroutine por stockpile) | Cola FIFO y puesto de descarga | Solicitud de ingreso y descarga | Cola ilimitada; 1 descarga a la vez |
| Despachador (goroutine) | Política de destinos y conocimiento consistente de fallas permanentes | Solicitudes de camiones; respuesta destino + ruta | No asignar rutas que crucen tramos con fallas permanentes |
| Control de tramos | Colas FIFO por sentido, ubicación/orden de vehículos, fallas | Solicitudes de entrada/salida y notificaciones | Sin adelantamiento; caída de un camión bloquea a quienes vienen detrás |
| Control de rampas | Sentido vigente, número de ocupantes, solicitudes pendientes | Reservas e ingresos; confirmación de salida | Direcciones opuestas nunca coexisten |
| Observador de estado | Instantánea coherente de camiones, colas y rampas | Lectura de estado | No mostrar actualizaciones parciales |
| Registro de eventos | Archivos separados por entidad y secuencia global | Emisión de eventos | Una goroutine es autora de cada archivo |

## Canales conceptuales

`solicitudDespacho`: camión → despachador. `respuestaDespacho`: despachador → camión solicitante. `solicitudIngreso`: camión → controlador de recurso. `autorizacion`: controlador → camión. `solicitudCarga` / `cargaFinalizada`: camión ↔ pala. `solicitudDescarga` / `descargaFinalizada`: camión ↔ stockpile. `cancelacion`: notificación por término de turno o solicitud abandonada.

Cada solicitud tiene identificador de correlación para que una respuesta tardía no autorice a un camión cancelado. El cierre global propaga cancelación y espera el fin ordenado de los trabajadores sin que un emisor escriba en un canal ya cerrado.

## Conservación de recursos

Se debe adquirir el recurso siguiente antes de liberar el anterior; por ejemplo, un camión al final de una rampa debe asegurar un lugar de pala para abandonar la rampa. Una asignación del despachador se obtiene antes de salir del puesto de carga/descarga, porque define el siguiente recurso. La coordinación de múltiples recursos no se reduce a comprobar dos booleanos sucesivos: exige una operación atómica o un protocolo con reversión segura.

## Reglas temporales

El instante cero es común. `tick = floor((ahora − inicio) / Δ)`. Cada goroutine establece sus propios temporizadores para viajes y operaciones; no se usa una barrera por tick. Los valores lognormales se calculan con `σ² = ln(1+s²/m²)`, `μ = ln(m)−σ²/2`, `X = exp(μ+σZ)` y redondeo al tick más cercano con mínimo de 1.

## Falla y término

Solo hay fallas aleatorias en tramos dobles, por tick de recorrido, con probabilidades `p_f` y `p_perm`. Una temporal suspende el camión y luego lo reanuda; una permanente bloquea el sentido hasta fin del turno. Camiones en tránsito no replanifican; nuevas rutas excluyen fallas permanentes. `defer` y `recover` en cada camión convierten un `panic` en falla permanente, liberan recursos limitados que retuviera y cierran el log. El turno termina en tick 720, se cancelan esperas y se verifica la finalización de las goroutines.

## Responsabilidad de implementación

Los tres integrantes deben ser autores de componentes concurrentes identificables en el historial; ninguno puede limitarse a documentación. La integración requiere ramas, revisión por un compañero y pull requests a `main`.

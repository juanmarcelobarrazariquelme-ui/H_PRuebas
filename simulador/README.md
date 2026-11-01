# Simulador concurrente de una mina — H1

## Objetivo

Modelar una red de rutas, camiones, palas y stockpiles; en la implementación final, cada camión, pala, stockpile y despachador utilizará su propia goroutine. Las rampas son de sentido exclusivo y los tramos mantienen FIFO por sentido.

## Diseño

El desglose formal de los componentes se encuentra en `docs/arquitectura.md`; los protocolos y su argumento de prevención en `docs/protocolos_rampa.md`; el contrato del lector en `docs/lector_mina.md`; la descripción del recorrido en `docs/ciclo_camion.md` y el formato CSV en `docs/formato_logs.md`.

## Mina de referencia

El archivo `mina_ejemplo.txt` contiene seis nodos y cinco tramos: E1, I1, I2, P1, P2, S1 y T1, T2, T3, R1, R2. Es el ejemplo del enunciado y no sustituye el anexo específico de cada grupo.

## Salidas documentadas

Los directorios `ejemplos_logs/caso_basico` y `ejemplos_logs/caso_fallas` contienen eventos **escritos manualmente** que ilustran el formato de intercambio previsto para Certamen III; no son salidas ejecutadas por una aplicación.

## Reglas de simulación

- Duración: 720 ticks; un tick equivale a 1 minuto simulado y a un intervalo configurable Δ en tiempo real.
- Recorrido vacío: tiempo base; cargado: `ceil(1,3 × tiempo base)` por tramo.
- Rampa: ingreso compatible con el sentido ocupado; la exclusión es atómica.
- Pala: un lugar de carga y `capacidad − 1` lugares de espera.
- Stockpile: una posición de descarga y cola de espera ilimitada.
- Adquisición antes de liberación, salvo alternativa de zona de salida explícitamente modelada.
- Las adquisiciones limitadas registran una espera prolongada al exceder un límite configurado.
- Los eventos de logs reciben secuencia global dentro de la sección crítica del evento, conservando orden causal.

## Código y verificación

La implementación en Go, el Makefile funcional, las pruebas de concurrencia, el cumplimiento de `go run -race` y los experimentos pertenecen al desarrollo propio de los tres integrantes y a los hitos posteriores. Ninguna ejecución se atribuye a este material documental.

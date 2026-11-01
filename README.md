# Certamen II — Hito H1

## Alcance

Material documental del H1 para una simulación concurrente de carguío y transporte en Go: diseño de componentes, pseudocódigo de los protocolos de rampas, contrato de lectura del archivo de mina y especificación y ejemplos manuales de logs. Los ejemplos emplean el grafo didáctico publicado en el enunciado.

## Índice

- `simulador/docs/arquitectura.md`: componentes, estado compartido, goroutines y secuencia de recursos.
- `simulador/docs/protocolos_rampa.md`: protocolos ingenuo y de prevención, condiciones de Coffman y garantías.
- `simulador/docs/lector_mina.md`: gramática, estructuras, validación y pruebas de aceptación.
- `simulador/docs/ciclo_camion.md`: trazado verificable de un ciclo de transporte.
- `simulador/docs/formato_logs.md`: contrato CSV y reglas de secuencia.
- `simulador/ejemplos_logs/`: eventos manuales ilustrativos organizados por escenario.
- `simulador/mina_ejemplo.txt`: grafo público de referencia del enunciado.
- `DEVLOG.md`, `AI_USAGE.md` y `COMPROMISO.md`: documentos de trazabilidad y registro.

## Entrega académica

El H1 requiere también un lector real en Go y la ejecución real de un camión que complete el ciclo pala–stockpile; esos componentes solo pueden ser implementados por el equipo, conforme a la política de IA indicada en el enunciado. No se incluyen aquí programas Go generados por IA ni se declaran pruebas de ejecución que no se han realizado. Los archivos de ejemplo se identifican como registros **manuales**.

La entrega oficial se evalúa en el repositorio registrado y en la rama `main`, con las reglas de trazabilidad, autoría y revisión establecidas por el profesor.

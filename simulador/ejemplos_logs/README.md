# Ejemplos manuales de eventos

Los archivos en `caso_basico/` y `caso_fallas/` ilustran dos escenarios **independientes**. En cada escenario, los números `seq` son únicos y ordenan los eventos entre archivos; no deben concatenarse ambos escenarios como una ejecución única.

`caso_basico`: lectura de una trayectoria completa del camión C01 y de C02 por la mina de ejemplo. La pala P1 registra los comienzos y fines de carga con los mismos identificadores de correlación que sus camiones. La cronología presenta tiempos nominales y valores de mineral escogidos para ilustración.

`caso_fallas`: eventos de fallo temporal, reparación, fallo permanente, espera sin ciclo, reserva y cancelación y `panic` recuperado. C03 y C04 aparecen como entidades de situaciones ilustrativas específicas; no se pretende representar sus viajes completos.

Los ejemplos no reemplazan la futura producción de logs reales; el formato y el catálogo completos se especifican en `../docs/formato_logs.md`.

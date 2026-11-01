# Especificación del lector del archivo de mina

## Gramática de entrada

Cada línea significativa define un nodo o un tramo, con campos separados por espacios o tabulaciones. Se ignoran líneas en blanco y comentarios cuyo primer carácter no blanco es `#`.

```text
NODO <id> estacionamiento
NODO <id> interseccion
NODO <id> stockpile
NODO <id> pala <capacidad_entera_positiva>
TRAMO <id> <nodo_a> <nodo_b> doble <tiempo_entero_positivo>
TRAMO <id> <nodo_a> <nodo_b> rampa <tiempo_entero_positivo>
```

Los IDs son cadenas no vacías sin espacios; deben ser únicos dentro de su categoría. Los extremos de cada tramo han de referirse a nodos declarados. Una arista conecta ambos sentidos; la distinción `doble`/`rampa` cambia la política de circulación, no la topología.

## Tabla de símbolos recomendada

- `nodos[id]`: tipo, capacidad y atributos del nodo.
- `tramos[id]`: extremos, tipo, tiempo base y atributos del tramo.
- `adyacencias[idNodo]`: incidencias con identificador de tramo, nodo vecino y sentido.
- Estado dinámico separado: fallas, ocupación, FIFO y reservas; su acceso concurrente requiere consistencia.

## Pseudocódigo de lectura

```text
iniciar tablas vacías de nodos y tramos
para cada línea numerada del archivo:
    limpiar espacios exteriores
    si está vacía o inicia con #: continuar
    dividir en tokens
    según primer token:
        NODO: validar aridad, tipo, capacidad y unicidad; guardar
        TRAMO: validar aridad, tipo, tiempo y unicidad; guardar
        otro: error indicando línea y contenido
al terminar la lectura:
    para cada tramo:
        validar existencia de extremos y que sean distintos
        incorporar cada sentido en las adyacencias
    validar existencia de al menos un estacionamiento, una pala y un stockpile
    devolver grafo completo o lista de errores
```

La validación diferida de extremos permite que un tramo aparezca antes de sus nodos. Se recomienda rechazar líneas parciales y números fuera de rango; no continuar con un grafo inconsistente.

## Matriz de aceptación

| Caso | Resultado |
|---|---|
| Archivo de ejemplo | 6 nodos, 5 tramos; P1 y P2 capacidad 3 |
| Comentarios y blancos | Sin entidades añadidas |
| ID repetido de nodo o tramo | Error con línea |
| Pala sin capacidad / capacidad 0 | Error de aridad o dominio |
| Tramo con extremo no declarado | Error de integridad |
| Tipo de tramo desconocido | Error de dominio |
| Tiempo 0 o negativo | Error de dominio |
| No existe stockpile o pala | Error de mina inválida |

## Datos esperados del grafo de ejemplo

E1↔I1 por T1 (5); I1↔S1 por T2 (8); I1↔I2 por T3 (10); I2↔P1 por R1 (4); I2↔P2 por R2 (6). Para camión cargado cada tiempo se calcula por tramo con `ceil(tiempo × 1,3)`.

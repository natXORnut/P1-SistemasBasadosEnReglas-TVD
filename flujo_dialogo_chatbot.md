# Flujo de diálogo del chatbot de viviendas

Propuesta de la nueva versión del asistente (ejercicio 7). Los diagramas están en sintaxis Mermaid, que se visualiza en GitHub, VS Code (con la extensión de Mermaid) y la mayoría de editores de Markdown.

## 1. Cambios respecto a la versión actual

| Cambio | Antes | Ahora |
|---|---|---|
| Planta mínima | Se preguntaba casi al final | Se pregunta justo después del bloque de uso comercial, y todas las ramas convergen en un único nodo |
| Búsqueda | Función de búsqueda exacta, más menú de relajación, más «casas más próximas» | Una sola función de puntuación: las casas con 0 desajustos son las exactas |
| Resultados | Al ver una casa, la conversación terminaba | Se puede ver el detalle de varias casas y volver siempre a la lista |
| Presupuesto al alquilar | Se elegía entre precio máximo, ingresos o abierto | Se ofrece primero ver el precio máximo recomendado según el sueldo (35 %), que el usuario puede aceptar o modificar; si no quiere verlo, se le pregunta si quiere fijar un precio límite |
| Relajar requisitos | Menú obligatorio con sintaxis `1,3` / `2:any` cuando no había resultados | Opcional: el usuario lo pide cuando quiere, un requisito cada vez |

## 2. Fase 1: preguntas

```mermaid
flowchart TD
    A(["Mensaje de bienvenida"]) --> B{"¿Comprar o alquilar?"}
    B -->|comprar| P["Precio máximo"]
    B -->|alquilar| V{"¿Quieres ver el precio máximo<br/>recomendado según tu sueldo?"}
    V -->|sí| I["Ingresos mensuales"]
    I --> RC["Calcular y mostrar: se recomienda no superar<br/>el 35 % del sueldo, en este caso X"]
    RC --> MOD{"¿Quieres modificarlo?"}
    MOD -->|no| AC["price = precio recomendado"]
    MOD -->|sí| P
    V -->|no| LIM{"¿Quieres establecer<br/>un precio límite?"}
    LIM -->|sí| P
    LIM -->|no| O["Sin límite de precio"]
    P --> C
    AC --> C
    O --> C
    C{"¿Uso comercial?"}
    C -->|no| F
    C -->|sí| CF{"¿Te encaja planta baja?"}
    CF -->|sí| H
    CF -->|no| F
    F["Planta mínima"] --> H
    H["Habitaciones"] --> W["Baños"]
    W --> S["Metros cuadrados"]
    S --> L["Ubicación"]
    L --> T["Terraza"]
    T --> E["Ascensor"]
    E --> R(["Resumen y puntuación de casas"])
```

### Orden y condiciones de las preguntas

| Orden | `answer_key` | Pregunta | Se hace si… |
|---|---|---|---|
| 1 | `type` | ¿Comprar o alquilar? | Siempre |
| 2 | `see_recommendation` | ¿Quieres ver el precio máximo recomendado según tu sueldo? | Alquila |
| 3 | `monthly_income` | Ingresos mensuales | Alquila y quiere ver la recomendación |
| 4 | `modify_price` | Se muestra el mensaje con el 35 % calculado y se pregunta: ¿quieres modificarlo? | Alquila y quiere ver la recomendación |
| 5 | `set_limit` | ¿Quieres establecer un precio límite? | Alquila y no quiere ver la recomendación |
| 6 | `price` | Precio máximo | Compra; o alquila y quiere modificar el recomendado; o alquila y quiere fijar un límite |
| 7 | `commercial_use` | ¿Uso comercial? | Siempre |
| 8 | `commercial_floor` | ¿Te encaja una planta baja? | Uso comercial = Sí |
| 9 | `floor` | Planta mínima | Siempre, salvo uso comercial = Sí y acepta planta baja |
| 10 | `bedrooms` | Habitaciones | Siempre |
| 11 | `bathrooms` | Baños | Siempre |
| 12 | `square_meters` | Metros cuadrados | Siempre |
| 13 | `location` | Ciudad o barrio | Siempre |
| 14 | `terrace` | Terraza | Siempre |
| 15 | `elevator` | Ascensor | Siempre |

Notas:

- El precio recomendado se calcula al momento, en cuanto el usuario da sus ingresos, y se muestra con un mensaje del tipo: «Recomendamos que el precio máximo no supere el 35 % de tu sueldo; en tu caso sería X €. ¿Quieres modificarlo?».
- En `monthly_income` no se acepta `any` ni «no lo sé»: el usuario ya aceptó ver la recomendación, así que el bot insiste en pedir un número. En `price` (precio máximo libre) `any` sí vale y significa «sin límite».
- Si el usuario no modifica la recomendación, `price` se guarda con el valor calculado y no se vuelve a preguntar el precio.
- Si quiere modificarlo, se le pide el precio máximo con la pregunta `price` y se usa ese valor.
- Si no quiere ver la recomendación y tampoco quiere fijar un límite, `price` se guarda como `any`.
- Si rechaza la planta baja, el bot lo comenta («Alright, then let's choose the floor yourself.») y pregunta la planta mínima con normalidad.
- La pregunta de ubicación avisa de que, para ver las ubicaciones disponibles, se escribe `options`. El bot las lista y vuelve a esperar la respuesta, sin avanzar de pregunta.
- Las preguntas 2 a 8 pueden cambiar las que vienen después, así que el flujo se reconstruye tras responderlas (`RECOMPUTE_FLOW_ON`).

## 3. Fase 2: puntuación y presentación de resultados

```mermaid
flowchart TD
    S["Puntuar casas<br/>(tras filtrar requisitos duros)"] --> D{"¿Hay casas con 0 desajustos?"}
    D -->|sí| E["Mostrar todas las exactas<br/>id, zona y precio"]
    D -->|no| N["Avisar: no hay coincidencia exacta"]
    E --> QE{"¿Te gusta alguna o prefieres alternativas?"}
    N --> A
    QE -->|"número"| DE["Detalle completo de la casa"]
    DE --> QE
    QE -->|alternatives| A
    QE -->|relax| R
    QE -->|no| FIN
    A["Mostrar las 3 mejores con desajustos"] --> QA{"¿Qué quieres hacer?"}
    QA -->|"número"| DA["Detalle completo de la casa"]
    DA --> QA
    QA -->|more| A
    QA -->|relax| R
    QA -->|no| FIN
    R["Relajar un requisito"] --> M{"¿Relajar algo más?"}
    M -->|sí| R
    M -->|no| S
    FIN(["Fin del diálogo"])
```

### Opciones del usuario en cada lista

| Respuesta | Efecto |
|---|---|
| Un número | Muestra el detalle completo de esa casa y vuelve a la misma lista |
| `alternatives` (solo en la lista de exactas) | Pasa a las 3 mejores casas con desajustos |
| `more` | Muestra las 3 siguientes de la lista ordenada |
| `relax` | Pasa a la fase 3 |
| `no`, `done`, «that's all» | Termina el diálogo |

Si no hay ninguna casa exacta, el bot lo dice y pasa directamente a las alternativas, sin preguntar si quiere verlas.

### Requisitos duros (filtran, no puntúan)

Una casa que no cumpla alguno de estos requisitos no entra en la lista:

- `type` (comprar o alquilar).
- `commercial_use = Yes`: solo casas con uso comercial permitido.
- Planta baja, si es uso comercial y el usuario aceptó la recomendación.

### Desajustos y gravedad

Cada requisito que una casa no cumple es un desajuste, con una gravedad entre 0 (leve) y 1 (grave).

| Requisito | Se considera desajuste si… | Gravedad |
|---|---|---|
| Presupuesto | El precio supera el tope (precio directo o 35 % de ingresos) | `min(1, (precio - tope) / tope)` |
| Metros cuadrados | La casa se sale del margen: 10 m² menos o 25 m² más de lo pedido | Exceso sobre el margen / 25, máximo 1 |
| Planta mínima | La casa está por debajo de la planta pedida | `(pedida - real) / max(pedida, 1)`, máximo 1 |
| Habitaciones y baños | El número no coincide exactamente | `\|diferencia\| / 2`, máximo 1 |
| Ubicación | La zona es distinta | 0,6 fijo |
| Terraza y ascensor | Los pidió y la casa no los tiene | 0,3 fijo |

Una casa es exacta cuando no tiene ningún desajuste. El resto se ordena por:

1. Número de desajustos (menos es mejor).
2. Suma de gravedades.
3. Id de la casa, para desempatar.

## 4. Fase 3: relajar requisitos

El usuario escribe `relax` desde cualquier lista de resultados.

```mermaid
flowchart TD
    A["El bot lista los requisitos activos<br/>con las casas extra que aparecerían"] --> B["El usuario elige uno"]
    B --> C["Se aplica el cambio a las preferencias"]
    C --> D{"¿Relajar algo más?"}
    D -->|sí| A
    D -->|no| E(["Volver a puntuar todas las casas"])
```

| Requisito | Al relajarlo |
|---|---|
| Presupuesto | Sube el tope un 10 % (`BUDGET_RELAXATION_PERCENT`) |
| Habitaciones, baños, ubicación | Pasa a `any` |
| Metros cuadrados | Pasa a `any` |
| Planta mínima | Pasa a `any` |
| Terraza, ascensor | Pasa a `any` |
| Planta baja (uso comercial) | Pasa a «cualquier planta» |
| Tipo de operación, uso comercial = Sí | No se pueden relajar |

Junto a cada requisito se indica cuántas casas más aparecerían: son las que solo fallan en ese campo, y se obtienen de la misma lista de desajustos sin volver a buscar. Los cambios son acumulativos: lo ya relajado sigue relajado en las rondas siguientes.

## 5. Comandos disponibles en cualquier momento

| Comando | Efecto | Dónde |
|---|---|---|
| `back`, «go back», «I made a mistake» | Pide confirmación y vuelve a la pregunta anterior | Solo en la fase de preguntas |
| `q`, `quit`, `exit` | Pide confirmación y sale del chat | Todas las fases |
| `any`, «I don't care», «no preference» | Guarda la preferencia como `any` | Fase de preguntas, salvo en las preguntas binarias del flujo (ver abajo) |
| «I don't know», `idk`, «not sure» | Guarda `any` y se lo dice al usuario | Fase de preguntas, salvo en las preguntas binarias |

En `monthly_income` y en las preguntas binarias del flujo (`type`, `see_recommendation`, `modify_price`, `set_limit`, `commercial_use`, `commercial_floor`) no se acepta `any` ni «no lo sé» (en las binarias `any` equivaldría a «No»): el bot pide una respuesta clara y no sugiere `any` en los mensajes de error. `terrace` y `elevator` sí lo aceptan («I don't mind», «whatever», «no preference»...), y se guarda como `any`; «I don't want any terrace» se entiende como «No».

## 6. Decisiones pendientes

1. Si el usuario relaja el presupuesto varias veces, ¿cada vez sube otro 10 % o solo se permite una vez?
2. Tras ver las alternativas, ¿debe poder volver a la lista de casas exactas? Propuesta: sí, con la palabra `exact`.
3. Cuando se acaban las alternativas con `more`, ¿basta con avisar y volver a la lista, o se ofrece relajar?
4. ¿Debe funcionar `back` también en la fase de resultados? Hoy solo funciona en la de preguntas.

## 7. Idioma

El bot habla y entiende solo inglés: todos los diccionarios de palabras y frases (afirmaciones, negaciones, `any`, `back`, `relax`, `alternatives`, `done`, salida...) están en inglés. Este documento está en español solo como documentación.

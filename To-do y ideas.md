# TO-DO

## A. Parsing flexible de cada respuesta (lo que hablamos)
- [ ] Limpiar el número extraído (`'2000€'` → `2000`, `'35k'` → `35000`), en vez de devolver el token tal cual.
- [ ] Detectar comparador por palabras de contexto cerca del número ("max", "at least", "around" → `<=`, `>=`, `==` por defecto).
- [ ] Sinónimos + negación para las preguntas Sí/No (terraza, ascensor, uso comercial): aceptar "yeah", "I don't want a terrace", no solo `Yes`/`No` exactos.
- [ ] Fuzzy matching en `location` (`difflib`) para errores de escritura y mayúsculas.
- [ ] Aceptar números en palabra para habitaciones/baños ("one", "two"), ya que el rango es pequeño (1-5 / 1-3).
- Añadir más frases y sinonimos

## B. Sensación de conversación natural (interfaz)
- [ ] Confirmación corta tras cada respuesta ("Got it, 3 bedrooms.") antes de pasar a la siguiente pregunta.
- [ ] 2-3 variantes de cada mensaje del agente elegidas al azar, para que no suene siempre igual.
- [ ] Resumen de todo lo elegido justo antes de buscar ("Buying, 3 bedrooms, Barcelona, up to 300k — searching…").
- [ ] Mensaje de error progresivo si el usuario no da una respuesta válida (1r intento: pista; 2n: ejemplo concreto de los datos; 3r: opción de poner `any` en vez de quedarse atascado).
- [ ] `back` para volver a la pregunta anterior sin reiniciar todo.

## C. Ramificaciones del propio sistema (no las decide el usuario, las decide el agente según lo ya respondido)
- [ ] Si `type = rent` → pregunta de ingresos y tope del 35%; si `type = sale` → se salta.
- [ ] Si `commercial_use = Yes` → sugerir/filtrar planta baja.
- [ ] Si la planta resultante es alta (≥3) y la casa no tiene ascensor → aviso en el resultado, no antes.

## D. Búsqueda y resultados (bugs reales, no opcionales)
- Mirar si se me ocurren más ideas

## E. Pruebas
- [ ] Tabla de frases con distintas formas de decir lo mismo (10-15 frases) para comprobar que el parser flexible funciona, y qué porcentaje acierta.

## F. Opcional / baja prioridad
- [ ] Datos sintéticos y más campos.
- [ ] Usuarios simulados.




# Problemas que he solucionado en el ejercicio 7
- Que acepte sinonimos dentro de las respuestas
- que al escribir la localización, no requiera que esté escrita tal cual

# Revisar
- que si dices any con sale and rent, te pida las dos opciones de precio
- Revisar que los caminos sean correctos
- fuzzy maching en preguntas que no tengan si y no
- preguntar si el precio del alquiler lo quieren calcular a partir de su sueldo o dar un precio limite.
- hacer que el debug se active con un parametro

🏠 No exact match, and no single change unlocks anything. These are the closest houses:
1. House 18 · Barcelona · 850€/month — differs in: size (50 vs ~90 m²), bedrooms (1 vs 3)
Type the number to see the full details, or 'no' to finish.

- A lo mejor habria que hacer mas natural lo que dice y la dinamica en la segunda ronda

- Acabar de revisar como salen las cosas si solo cambias un factor, dos o tres.

- lo de ir hacia atrás hay que revisarlo y si el usuario dice que se ha equivocado, tambien ir hacia atrás, no sé si poner una pregunta de confirmacion...

- Que se pueda cambiar algo concreto con change

- que si has dicho que sea de proposito comercial, donde se pone el filtro de piso 0, entonces te pregunta igualmente el piso y eso hace que sea contradictorio...

- Cuando preguntamos que metodo queremos utilizar para estimar el precio de rent, me gustaria que el chat de tijera, podemos poner un precio limite, estimar una renta o dejar el campo habierto si no tienes un presupuesto en mente. 

- que te pregunte si estas seguro de que quieres abandonar el chat cuando haces q o quit

- faltan limites en las preguntas numericas

- si te equivocas en la ultima pregunta no va hacia atras porque te busca los pisos directamente

- boton para nueva conversacion?

# Resuelto

# Ejercicio 7: resumen de cambios

## 1. Entrada del usuario

### 1.1 Sí/No con sinónimos y negación
- Preguntas afectadas: `terrace`, `elevator`, `commercial_use`.
- Se aceptan frases libres: "yeah", "sure", "nah", "I don't want a terrace", "no thanks".
- La negación tiene prioridad: si aparece una palabra negativa, la respuesta es `No`. Si no, y hay una afirmativa, es `Yes`.
- También se reconocen palabras clave: "balcony" o "lift" para terraza y ascensor, "shop" o "business" para uso comercial.

### 1.2 Números en palabra
- "one", "two"... "ten", además de "single", "couple" y "ground" (planta 0).
- Se usan como alternativa cuando no hay dígitos en la respuesta.

### 1.3 Sinónimos para "any"
- Palabras sueltas, comprobadas como palabra completa: `any`, `whatever`, `indifferent`, `flexible`.
- Frases de 2 o más palabras, comprobadas como subcadena: "no preference", "don't care", "doesn't matter", "not fussed", "either way", "up to you", "you decide", etc.
- Se separa así a propósito para evitar falsos positivos. Por ejemplo, "any" está dentro de "many", y "not really" tiene que seguir siendo un `No` real.

### 1.3b Sinónimos de otras preguntas
- `type`: buy, purchase, lease, renting...
- `budget_method`: income o salary frente a budget, price o max.

### 1.4 Comando `back`
- Vuelve a la pregunta anterior sin reiniciar la conversación.
- Antes de aplicar cada respuesta se guarda una copia de (preguntas, paso, preferencias).
- También funciona cuando el flujo cambia a mitad (por ejemplo, al cambiar de `rent` a `sale`).

---

## 2. Mensajes del agente

- **Confirmación tras cada respuesta** ("Got it — 3 bedroom(s)."), con 4 plantillas elegidas al azar.
- **Variantes de preguntas:** 3 variantes por pregunta, elegidas al azar.
- **Resumen antes de buscar:** "Renting, 3 bedroom(s), Barcelona, up to 1000€, with terrace — searching...".
- **Formato del resultado final:**
  `House 3 · Santa Coloma de Gramenet · 800€/month — 2 bedrooms, 1 bathrooms, 80 m², floor 8, elevator: Yes, terrace: No, commercial use: No`
  En alquiler el precio sale como `800€/month` y en compra como `250k€`.

---

## 3. Flujo de preguntas

- **Compra (`sale`):** se pregunta directamente el precio máximo.
- **Alquiler (`rent`):** primero se pregunta si quiere calcular el presupuesto a partir de sus ingresos o si ya tiene un precio máximo en mente.
  - `income`: se pregunta por los ingresos y el tope es el 35% de ese valor.
  - `budget`: se pregunta el precio máximo.
  - Solo se hace una de las dos preguntas, nunca las dos.
- El flujo se reconstruye tras responder `type` o `budget_method`.
- **Uso comercial = Yes:**
  - Al llegar a la pregunta de planta, el bot avisa de que se priorizará la planta baja.
  - La búsqueda filtra por planta 0.
- **Corrección:** `commercial_use = No` ya no excluye casas que permiten uso comercial. Solo el `Yes` restringe.

---

## 4. Reglas de búsqueda

| Campo | Regla en la primera búsqueda |
|---|---|
| Precio | Tope estricto, sin margen, en compra y en alquiler. Se calcula con `effective_budget_cap`: un precio directo manda sobre el 35% de ingresos. |
| m² | Margen asimétrico: `SQM_MARGIN_BELOW = 10` y `SQM_MARGIN_ABOVE = 25`. Pedir 80 m² busca entre 70 y 105 m². Es preferible pasarse que quedarse corto. |
| Habitaciones y baños | Coincidencia exacta. |
| Planta | Planta mínima. Se ignora si hay uso comercial (planta 0). |
| Terraza y ascensor | Solo restringen si se pidieron con `Yes`. |

---

## 5. Negociación (segunda ronda)

Si no hay resultados exactos, el bot calcula qué campos, relajados de uno en uno, desbloquean casas. Los muestra todos a la vez y el usuario elige con `1`, `1,3` o `2:any`. Sale con `no`.

| Campo | Qué se ofrece |
|---|---|
| Presupuesto | Subir el tope un 10% (`BUDGET_RELAXATION_PERCENT`). Funciona igual si viene de un precio directo o de los ingresos. |
| Habitaciones, baños, ciudad | Valores alternativos que desbloquean casas. |
| Terraza, ascensor | "required -> not required". |
| m² | Cualquier tamaño. |
| Planta mínima | Cualquier planta. |

No se negocian nunca: `type` (comprar o alquilar) y `commercial_use = Yes`. Se consideran necesidades de fondo.

---

## 6. Casas más cercanas

- Se activan cuando ninguna relajación individual desbloquea nada.
- Para cada casa se cuentan los campos en los que no encaja y se puntúa su gravedad de 0 a 1.
- Se muestran solo las casas con el mínimo de fallos, ordenadas por gravedad total, hasta 3.
- El bot no elige por el usuario: lista las casas y él escribe el número para ver la ficha completa.
- Ejemplo:
  `1. House 18 · Barcelona · 850€/month — differs in: size (50 vs ~90 m²), bedrooms (1 vs 3)`
- Los pesos son constantes ajustables: terraza y ascensor 0.3, ubicación 0.6, habitaciones y baños según la diferencia.

---

## 7. Organización del código

- Nueva clase **`DialogEngine`**: sustituye al diccionario global `dialog_state` y a las funciones que lo modificaban. Guarda el estado (pregunta actual, preferencias, historial para `back`, modo) y tiene métodos como `handle_answer`, `go_back`, `finish_dialog`, `offer_relaxation_options` y `offer_closest_houses`.
- `chatbot_v7()` crea una instancia nueva cada vez.
- Una función `handle_answer` a nivel de módulo delega en la instancia actual, así que los widgets se registran una sola vez.
- Las funciones sin estado siguen siendo funciones independientes: el parsing de respuestas, `find_suitable_houses`, `build_question_flow`, etc.

---

## 8. Limitaciones conocidas

1. **Relajación de uno en uno.** Cuando el problema son varios filtros a la vez, ninguno aparece por separado. En ese caso entra el mecanismo de casas cercanas.
2. **Ciudad en la negociación.** La lista de ciudades sale en orden alfabético y `1` aplica solo la primera. Hay dos arreglos posibles: ordenar por cercanía geográfica y dejar elegir la ciudad.
3. **Casas cercanas.** Solo se muestra el grupo con menos fallos, así que a menudo saldrá una única casa. Los pesos de gravedad son una decisión de diseño, no algo objetivo.
4. **Variación de mensajes.** Solo varían las preguntas y las confirmaciones. Los mensajes de bienvenida, despedida y error son fijos.
5. **Detección de negación.** Es por palabras sueltas, así que frases ambiguas como "not sure" se leen como `No`.
6. **Pruebas.** La lógica se probó ejecutando las celdas con sustitutos de `ipywidgets` y de `nltk`. No se ha ejecutado en un Jupyter real, así que conviene lanzar el notebook completo una vez.

---

## 9. Casos de prueba (respuestas en orden de pregunta)

| Qué prueba | Respuestas | Resultado |
|---|---|---|
| Compra exacta | `sale, 300k, no, 3, 2, any, Barcelona, any, no, no` | Casa 23 |
| Alquiler por ingresos | `rent, income, 3000, no, one, one, any, any, any, any, any` | Casas 5, 9, 17, 18 |
| Uso comercial | `sale, 300k, yes, 4, 2, any, any, any, no, no` | Casa 2 |
| Negociar presupuesto | `sale, 330000, no, 3, 2, any, any, any, yes, no` y luego `1` | Casa 20 |
| Negociar baños | `sale, 500000, no, 4, 3, any, any, any, yes, no` y luego `1` | 5 casas |
| Negociar ascensor | `rent, budget, 900, no, 1, 1, any, Barcelona, 5, any, yes` y luego `1` | Casa 17 |
| Varias opciones | `rent, budget, 1200, no, 3, 1, any, Santa Coloma, any, any, any` | Baños, habitaciones o ciudad |
| Casas cercanas | `rent, budget, 1000, no, 3, 1, 90, Barcelona, 4, yes, yes` | Casa 18 |
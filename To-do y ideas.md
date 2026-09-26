# TO-DO

## A. Parsing flexible de cada respuesta (lo que hablamos)
- [ ] Limpiar el número extraído (`'2000€'` → `2000`, `'35k'` → `35000`), en vez de devolver el token tal cual.
- [ ] Detectar comparador por palabras de contexto cerca del número ("max", "at least", "around" → `<=`, `>=`, `==` por defecto).
- [ ] Sinónimos + negación para las preguntas Sí/No (terraza, ascensor, uso comercial): aceptar "yeah", "I don't want a terrace", no solo `Yes`/`No` exactos.
- [ ] Fuzzy matching en `location` (`difflib`) para errores de escritura y mayúsculas.
- [ ] Aceptar números en palabra para habitaciones/baños ("one", "two"), ya que el rango es pequeño (1-5 / 1-3).

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
- [ ] `price` se compara con `==` en vez de `<=`: casi nunca hay coincidencia. Hay que arreglarlo si o sí para que el sistema devuelva algo.
- [ ] `bedrooms`, `bathrooms`, `square_meters` igual: deberían ser `>=`, no `==`.
- [ ] Mensaje de "sin resultados" con explicación de por qué (qué filtro falló), en vez de solo "lo siento". No hace falta que ofrezca alternativas para elegir, solo que explique.

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

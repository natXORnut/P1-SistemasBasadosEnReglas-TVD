# To-do del ejercicio 7

Está organizado siguiendo el método del enunciado (Exemples → Camins → Prototipat → Proves). Lo marcado como *(opcional)* se puede dejar para el final si falta tiempo.

## 0. Decisiones previas
- [ ] Decidir el alcance: qué entra sí o sí (extractor, gestor de diálogo, ranking y relajación) y qué queda como extra (datos sintéticos, usuarios simulados).
- [ ] Repartir el trabajo con tu compañera: por ejemplo, una hace el extractor y la otra el gestor de diálogo y la búsqueda.
- [ok] Decidir el idioma del agente. Lo más simple es inglés, como el JSON. - Inglés

## 1. Exemples (diálogos de ejemplo)
- [ ] Escribir 4 diálogos en una celda markdown:
  - **Camino feliz:** el usuario responde pregunta a pregunta.
  - **Todo en una frase:** "I want a 3 bedroom flat with a terrace under 200k".
  - **Usuario vago:** "something cheap near Barcelona".
  - **Sin resultados:** el agente relaja una restricción y lo explica.
- [ ] Anotar en cada diálogo qué datos debería extraer el sistema y cuál es la siguiente pregunta.
- [ ] Redactar los mensajes del agente: saludo, confirmación, "¿algo más importante para ti?", disculpa, despedida.

## 2. Camins (flujo del diálogo)
- [ ] Dibujar el diagrama de flujo (papel, mermaid o el Visualizer) con las ramas principales:
  - Compra o alquiler: en alquiler se piden ingresos; en compra no.
  - Uso comercial → sugerir planta baja.
  - Planta alta sin ascensor → avisar o preguntar.
  - Sin resultados → relajar restricciones.
  - Tras los resultados → refinar o empezar de nuevo.
- [ ] Definir los comandos globales: `quit`, `any`, `back`, `restart`.
- [ ] Clasificar los campos en restricciones duras (tipo, uso comercial, tope del 35%) y blandas (habitaciones, metros, terraza, planta...).

## 3. Prototipo
**Base de datos y normalización**
- [ ] Función para pasar los precios a número (`"250k"` → 250000).
- [ ] Convertir Yes/No a booleano y los strings numéricos a `int`.
- [ ] Calcular desde los datos las opciones disponibles por campo, con su recuento (para los ejemplos del agente).

**Frame de slots**
- [ ] Diccionario con todos los campos (`None` / `"any"` / valor).
- [ ] El estado se conserva entre turnos y no se reinicia.

**Extractor por reglas (NLTK, sin deep learning)**
- [ ] Números asignados a un slot según la palabra de contexto ("3 bedrooms", "under 200k", "at least 80 m²", "2nd floor").
- [ ] Ubicación con fuzzy matching (`difflib.get_close_matches`), que tolere mayúsculas y typos.
- [ ] Diccionario de sinónimos para terraza, ascensor, uso comercial, compra y alquiler.
- [ ] Negaciones en una ventana de 2-3 tokens ("no elevator", "without a terrace").
- [ ] Detectar "any", "no me importa" y expresiones similares.

**Gestor de diálogo**
- [ ] Elegir la siguiente pregunta según los slots que faltan, sin repetir lo que el usuario ya dijo.
- [ ] Implementar las ramificaciones del apartado 2.
- [ ] Confirmación implícita antes de buscar: "So you want a rental, 2 bedrooms, in Barcelona, right?".
- [ ] Paso "¿algo más importante?" con ejemplos generados desde la base de datos ("we also have houses with a terrace (9), elevator (14)...").
- [ ] *(Opcional)* Pregunta dinámica: elegir la pregunta que mejor divide las casas candidatas que quedan.

**Búsqueda**
- [ ] Filtro de restricciones duras y puntuación de las blandas.
- [ ] Mostrar el top 3 con el porqué (✓ / ✗ por criterio).
- [ ] Relajación guiada cuando no hay resultados, avisando de qué restricción se ha relajado.
- [ ] Bucle de refinamiento tras los resultados ("cheaper", "another option", "start over").

**Robustez**
- [ ] Manejar entradas vacías, no reconocidas y en mayúsculas o minúsculas, sin que el programa se caiga.
- [ ] Máximo de reintentos por pregunta y mensaje de ayuda cuando no se entiende al usuario.

## 4. Proves
- [ ] Tabla de 15-20 frases de prueba con el resultado esperado del extractor y si acierta o no; calcular el porcentaje de acierto.
- [ ] Corregir los fallos típicos (negaciones, números ambiguos, typos en la ubicación) y anotar qué se cambió y por qué.
- [ ] Repetir los 4 diálogos del apartado 1 contra el prototipo y comparar el resultado real con lo previsto.
- [ ] Prueba con 2-3 personas ajenas a la práctica; anotar dónde se atascaron y qué mejorasteis.
- [ ] *(Opcional)* Usuarios simulados: perfiles aleatorios que verifican que nunca hay un callejón sin salida y que las conversaciones acaban en menos de N turnos.

## 5. Datos sintéticos *(opcional)*
- [ ] Generar 150-200 casas con semilla fija y correlaciones realistas (precio ≈ metros × precio/m² según zona, ascensor más probable en plantas altas).
- [ ] Añadir 3 o 4 campos nuevos (parking, mascotas, amueblado...) solo si el agente los usa en alguna pregunta.
- [ ] Mantener las 25 casas originales y actualizar `questions` en el JSON si se añaden campos.

## 6. Entrega
- [ ] Celdas markdown con el razonamiento de cada fase (ejemplos, diagrama, decisiones de diseño, resultados de las pruebas, limitaciones).
- [ ] Ejecutar el notebook de principio a fin y comprobar que corre sin errores.
- [ ] Comprobar que el agente no usa deep learning ni MLP y dejarlo explícito en la memoria.

## Orden recomendado
1 → 2 → normalización y frame → extractor → gestor de diálogo → búsqueda → pruebas → extras.

Si quieres, te lo paso como fichero `.md` para compartirlo con tu compañera, o te preparo el esqueleto del extractor.
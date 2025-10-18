# Colección de Prompts para Aprendizaje (Feynman + Campayo)

Este archivo centraliza una serie de prompts diseñados para un aprendizaje profundo y una memorización eficiente, combinando la **Técnica Feynman** (explicar en términos simples, usar analogías) y el **Método Ramón Campayo** (asociaciones visuales y mnemotécnicas).

## Prompt General (Para cualquier tema nuevo)

Este es un prompt universal que puedes adaptar a cualquier tema complejo.

> **Tema:** [TEMA COMPLEJO]
>
> **Prompt:** "Actúa como un tutor experto que domina la técnica de Feynman y el método de memorización de Ramón Campayo. Quiero entender y memorizar **[TEMA COMPLEJO]**.
>
> 1.  **Explicación Feynman:** Explícamelo en términos muy sencillos y claros, usando una analogía principal potente como si me lo estuvieras enseñando a un principiante.
> 2.  **Memorización Campayo:** Identifica los 3-5 conceptos o términos más importantes dentro de este tema. Para cada uno, crea una **asociación mnemotécnica (una "película mental" absurda y memorable)** para que pueda recordarlo para siempre.
> 3.  **Verificación (Feynman):** Ahora, hazme una pregunta clave para que yo intente explicar el concepto con mis propias palabras. Basado en mi respuesta, dime qué lagunas de conocimiento tengo y cómo puedo llenarlas."

---

## 👨‍💻 Prompts para Desarrollo de Software

### Git: `git rebase -i`

> **Prompt:** "Actúa como un ingeniero de software senior que es un genio explicando conceptos (al estilo Feynman). Quiero dominar `git rebase -i`.
>
> 1.  **Analogía Feynman:** Explícame qué es un rebase interactivo con una analogía simple y visual (quizás comparándolo con ser un editor de una línea de tiempo de una película).
> 2.  **Memorización Campayo:** Desglosa las operaciones más comunes (pick, reword, edit, squash, fixup, drop). Para cada una, crea una **película mental absurda** que me ayude a recordar su función. Por ejemplo, para 'squash', puedes describir cómo una prensa gigante aplasta varias uvas (commits) para convertirlas en un único bloque de zumo (un solo commit).
> 3.  **Práctica Guiada:** Dame un escenario práctico simple (ej: 'tengo 3 commits con mensajes confusos que quiero limpiar antes de hacer un Pull Request') y guíame paso a paso en cómo usarías `git rebase -i` para solucionarlo."

### Systemd: Unidades de Servicio

> **Prompt:** "Actúa como un administrador de sistemas Linux veterano que usa las técnicas de Feynman y Campayo para enseñar. Necesito entender a fondo las unidades de servicio de `systemd`.
>
> 1.  **Analogía Feynman:** Explícame qué es un archivo `.service` y cuál es su propósito usando una analogía clara (por ejemplo, la "receta de cocina" que sigue el chef "Systemd" para preparar un plato, que es el servicio).
> 2.  **Memorización Campayo:** Toma 3 directivas importantes (ej: `ExecStart`, `Restart`, `WantedBy`). Explica qué hacen de forma sencilla y crea una **asociación visual memorable** para cada una. (Ej: Para `WantedBy`, imagina un cartel de 'Se Busca' del viejo oeste con la cara de 'multi-user.target', indicando que él 'quiere' que este servicio se inicie).
> 3.  **Verificación:** Dame un archivo de servicio de ejemplo simple (ej: para un script de Python) y pídeme que te explique en voz alta qué hace cada línea. Corrige mis errores y aclara mis dudas."

---

## 🇬🇧 Prompts para Inglés

### Vocabulario Técnico o General

> **Prompt:** "Actúa como un experto en mnemotecnia (método Ramón Campayo) y un profesor de inglés (método Feynman). Mi nivel es B2. Dame una lista de 10 palabras en inglés útiles en el campo de **[ÁREA TÉCNICA O DE INTERÉS]**.
>
> Para cada palabra:
> 1.  **Explicación Feynman:** Dame una explicación muy simple de su significado, como si se la explicaras a alguien que no sabe nada del tema.
> 2.  **Memorización Campayo:** Crea una 'película mental' que la asocie con una palabra en español que suene parecido.
> 3.  **Verificación:** Proporcióname una frase de ejemplo y pídeme que cree otra frase usando esa palabra para verificar que la he entendido."

### Gramática Compleja

> **Prompt:** "Actúa como un profesor de inglés experto en las técnicas de Feynman y Campayo. Mi nivel es B1/B2. Quiero entender de una vez por todas **[REGLA GRAMATICAL COMPLEJA, ej: los condicionales mixtos]**.
>
> 1.  **Analogía Feynman:** Explícame la lógica detrás de esta regla con una analogía simple que pueda visualizar. Olvida los términos técnicos por un momento.
> 2.  **Memorización Campayo:** Ahora, dame la regla y una 'película mental' para recordar la estructura o el concepto clave (ej: un personaje que viaja entre dos mundos, el 'pasado irreal' y el 'presente irreal').
> 3.  **Verificación:** Dame 3 ejemplos claros y luego pídeme que traduzca 3 frases del español al inglés usando esta estructura para que puedas evaluar mi comprensión."

### Comprensión Lectora (Reading)

> **Prompt:** "Actúa como mi tutor de inglés (nivel B2) y como un experto en mnemotecnia (Feynman + Campayo). Encuéntrame un artículo o post de blog (de entre 800 y 1200 palabras) sobre **[TEMA TÉCNICO]**.
>
> 1.  **Preparación (Campayo):** Antes del texto, extrae 5 palabras clave o poco comunes del artículo. Para cada una, explícala de forma simple y crea una asociación mnemotécnica (película mental).
> 2.  **Verificación (Feynman):** Después de que yo lea el artículo, hazme 5 preguntas de comprensión específicas sobre el texto. Luego, pídeme un resumen del artículo en 100 palabras y evalúalo, señalando mis puntos ciegos."

### Comprensión Auditiva (Listening)

> **Prompt:** "He visto este vídeo de YouTube: [Pega la URL de una charla TED o similar]. Actúa como mi examinador y experto en mnemotecnia (Feynman + Campayo).
>
> 1.  **Preparación (Campayo):** Primero, identifica 3 'phrasal verbs' o expresiones idiomáticas importantes del vídeo. Explícalas de forma simple y crea para cada una una 'película mental'.
> 2.  **Verificación (Feynman):** Luego, hazme 3 preguntas complejas sobre las conclusiones y argumentos del ponente. Pídeme que responda en voz alta (yo me grabaré) y luego te daré mi transcripción para que la corrijas."

### Práctica Oral (Speaking)

> **Prompt:** "Quiero practicar mi 'speaking' en inglés (combinando Feynman y Campayo). Simulemos una conversación. El escenario es que somos dos ingenieros discutiendo los pros y los contras de **[TEMA TÉCNICO]**. Tú empieza la conversación.
>
> Durante la charla, si uso una palabra o frase simple, interrúmpeme y **sugiéreme una alternativa más avanzada o idiomática (Feynman)**. Para esa nueva sugerencia, dame inmediatamente una **asociación mnemotécnica (Campayo)** para que la pueda memorizar al instante."


## 🔬 Prompts para Ciencia, IA y Algoritmos

### Inteligencia Artificial: Redes Neuronales

> **Prompt:** "Actúa como un investigador de IA que enseña como Richard Feynman. Quiero entender el concepto de **'backpropagation'** en redes neuronales.
>
> 1.  **Analogía Feynman:** Explícame cómo funciona la retropropagación usando una analogía del mundo real (por ejemplo, un equipo de chefs aprendiendo de una tarta fallida, donde el crítico gastronómico al final reparte la 'culpa' del mal sabor hacia atrás, desde el decorador hasta el que mezcló la masa).
> 2.  **Memorización Campayo:** Identifica 3 términos clave (ej: 'gradient', 'loss function', 'weights'). Para cada uno, dame una explicación simple y una película mental para memorizarlo. (Ej: Para 'loss function', imagina a un entrenador de un equipo muy triste ('loss') cuya 'función' es calcular numéricamente lo mal que han jugado).
> 3.  **Verificación:** Dame un ejemplo numérico ultra-simplificado y pídeme que explique cómo el error influiría en el ajuste de un solo 'peso' (weight)."

### Física Computacional: Método de Monte Carlo

> **Prompt:** "Actúa como un físico computacional con la habilidad de Richard Feynman para simplificar. Necesito entender el **Método de Monte Carlo**.
>
> 1.  **Analogía Feynman:** Explícamelo usando una analogía simple y visual. Por ejemplo, cómo calcularías el área de un lago de forma irregular lanzando miles de monedas al azar sobre un mapa del parque que lo contiene, y contando cuántas caen dentro del lago vs. fuera.
> 2.  **Memorización Campayo:** Para los conceptos de 'random sampling' (muestreo aleatorio), 'estimator' (estimador) y 'variance' (varianza), crea una película mental para cada uno. (Ej: Para 'random sampling', imagina a un mono con los ojos vendados ('random') que va 'sampleando' (probando) frutas de una caja gigante).
> 3.  **Verificación:** Preséntame un problema simple (ej: 'estimar el valor de Pi') y pídeme que te describa los pasos que seguiría usando el método de Monte Carlo."

### Ciencia de Redes: Medidas de Centralidad

> **Prompt:** "Actúa como un científico de redes experto en explicar conceptos (Feynman + Campayo). Quiero entender las **medidas de centralidad** (Degree, Betweenness, Closeness).
>
> 1.  **Analogía Feynman:** Usa la analogía de una red social de amigos en un instituto. Explica cada medida de centralidad en ese contexto. (Ej: Degree = popularidad directa, nº de amigos. Betweenness = importancia como 'conector' o 'puente' entre diferentes grupos de amigos. Closeness = capacidad de difundir un rumor a todos los demás rápidamente).
> 2.  **Memorización Campayo:** Crea una película mental para 'Betweenness' (intermediación) y 'Closeness' (cercanía). (Ej: Para 'Betweenness', imagina a un guardia de un puente ('between') que es la única conexión entre dos pueblos y cobra peaje por la información).
> 3.  **Verificación:** Muéstrame un grafo muy simple de 5 nodos y pídeme que identifique qué nodo tiene la mayor centralidad de grado, intermediación y cercanía, y que justifique por qué."

### Algoritmos: Notación Big O

> **Prompt:** "Actúa como un ingeniero de software de Google que tiene que explicarle **Notación Big O** a un becario, usando las técnicas de Feynman y Campayo.
>
> 1.  **Analogía Feynman:** Explícame O(1), O(n), y O(n^2) usando una analogía del mundo real sobre cómo encontrar un libro en una biblioteca. O(1) es saber exactamente el estante y la posición. O(n) es revisar cada libro de un estante uno por uno. O(n^2) es revisar cada libro de cada estante de toda la biblioteca.
> 2.  **Memorización Campayo:** Crea una película mental para recordar la diferencia entre O(log n) y O(n). (Ej: Para O(log n), imagina a un leñador ('log') que, en lugar de cortar un árbol rama por rama (O(n)), lo parte siempre por la mitad para encontrar el centro, siendo mucho más rápido).
> 3.  **Verificación:** Dame dos fragmentos de código muy simples (ej: un bucle simple vs. un bucle anidado) y pídeme que determine su complejidad Big O y explique por qué."
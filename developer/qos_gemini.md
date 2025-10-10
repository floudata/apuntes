# Informe Exhaustivo sobre Calidad de Servicio (QoS) en Redes Cisco: De los Fundamentos a la Implementación Avanzada

## Introducción

En la era de las redes convergentes, donde una única infraestructura de Protocolo de Internet (IP) transporta simultáneamente datos, voz y vídeo, la gestión del ancho de banda ha evolucionado de ser una simple cuestión de capacidad a una estrategia arquitectónica crítica. La Calidad de Servicio (QoS, por sus siglas en inglés) ya no es una característica opcional, sino un componente fundamental para alinear el rendimiento de la red con los objetivos empresariales. Sin una gestión inteligente, todas las aplicaciones, desde una llamada de Voz sobre IP (VoIP) hasta una transferencia de archivos masiva, compiten por los recursos de la red en igualdad de condiciones, lo que inevitablemente degrada el rendimiento de las aplicaciones más sensibles y críticas para el negocio.[1, 2, 3]

El problema central reside en la naturaleza finita de los recursos de red. Todos los dispositivos de infraestructura, como routers y switches, tienen límites en la cantidad de tráfico que pueden procesar y transmitir en un momento dado.[4] Cuando la demanda de ancho de banda supera la capacidad disponible, se produce una congestión. En estos momentos críticos, los paquetes de datos se retrasan en búferes o, peor aún, se descartan. La Calidad de Servicio es el conjunto de mecanismos y tecnologías diseñados para controlar cómo se prioriza y maneja el tráfico durante estos períodos de congestión, garantizando que las aplicaciones esenciales reciban los recursos que necesitan para funcionar de manera óptima.[5, 6]

Este informe ofrece un análisis exhaustivo de la Calidad de Servicio en el ecosistema de Cisco. Se ha estructurado en tres partes progresivas para guiar al lector desde los conceptos fundamentales hasta las técnicas de implementación avanzadas. La Parte I establece los fundamentos, respondiendo al "porqué" de la necesidad de QoS. La Parte II detalla las herramientas y mecanismos específicos, abordando el "qué" y el "cómo" de su funcionamiento. Finalmente, la Parte III integra estos mecanismos en escenarios prácticos y configuraciones del mundo real, proporcionando ejemplos concretos que demuestran la aplicación de la teoría en redes operativas. El objetivo es proporcionar una guía definitiva que sirva tanto como recurso de aprendizaje fundamental como de referencia práctica para la implementación.

-----

## Parte I: Fundamentos de la Calidad de Servicio (Nivel Básico)

Esta sección establece el problema fundamental que la Calidad de Servicio resuelve y el marco conceptual para su solución. Antes de explorar las herramientas específicas, es imperativo comprender las deficiencias de la red que QoS busca mitigar y el modelo arquitectónico que permite una solución escalable.

### 1.1. El "Porqué" de QoS: Comprendiendo la Congestión y sus Efectos

La Calidad de Servicio es un conjunto de técnicas y tecnologías diseñadas para optimizar el rendimiento de la red de una organización, garantizando que las aplicaciones de alta prioridad y sensibles al rendimiento puedan seguir funcionando eficazmente incluso con un ancho de banda limitado y una alta congestión de la red.[1, 6] Para implementar QoS de manera efectiva, primero se deben comprender los problemas que pretende solucionar, a menudo denominados los "enemigos" del rendimiento de la red.

Estos impedimentos no son problemas aislados; a menudo están interrelacionados, creando un ciclo de degradación del rendimiento durante los períodos de congestión.[7]

  * **Latencia (Retardo):** Se refiere al tiempo que tarda un paquete de datos en viajar desde su origen hasta su destino.[1, 8] Una latencia elevada puede ser causada por la distancia física (retardo de propagación), el tiempo de procesamiento en los routers y, lo más importante, el tiempo de espera en las colas de los dispositivos de red congestionados. Es especialmente perjudicial para las aplicaciones interactivas como VoIP, videoconferencias y sesiones de escritorio remoto, donde un retardo notable puede hacer que la aplicación sea inutilizable.[3, 9]
  * **Jitter (Variación del Retardo):** Es la variación en el tiempo de llegada de paquetes consecutivos dentro del mismo flujo de datos.[1, 10] El jitter es el principal adversario de los flujos de medios en tiempo real como la voz y el vídeo. Causa que el audio suene entrecortado y que el vídeo se vea pixelado o se congele, ya que los paquetes no llegan a un ritmo constante y predecible. Para compensar el jitter, los dispositivos receptores deben utilizar un búfer (conocido como búfer de jitter) para reordenar los paquetes y reproducirlos a una velocidad constante, lo que, a su vez, añade latencia general a la comunicación.[8, 11] Para aplicaciones de VoIP, se recomienda mantener el jitter por debajo de 30 milisegundos (ms) para garantizar una alta calidad.[8, 12]
  * **Pérdida de Paquetes (Packet Loss):** Ocurre cuando los dispositivos de red, típicamente routers, se quedan sin espacio en sus búferes (colas) durante un período de congestión y se ven obligados a descartar los paquetes entrantes.[7, 13] Las aplicaciones que utilizan el Protocolo de Control de Transmisión (TCP), como la navegación web y las transferencias de archivos, pueden recuperarse de la pérdida de paquetes retransmitiéndolos. Sin embargo, este proceso de retransmisión introduce un retardo significativo y reduce drásticamente el rendimiento.[13] Para las aplicaciones que utilizan el Protocolo de Datagramas de Usuario (UDP), como VoIP o los juegos en línea, los paquetes perdidos se han ido para siempre, lo que resulta en sílabas perdidas en una conversación o fallos visuales en un juego.[9, 13]

La siguiente tabla resume el impacto de estos impedimentos en aplicaciones comunes, conectando las métricas técnicas con la experiencia tangible del usuario.

**Tabla 1: Análisis de los Enemigos de la Red**

| Métrica de Red | Descripción y Causas Comunes | Impacto en Aplicaciones (Ejemplos Reales) |
| :--- | :--- | :--- |
| **Latencia** | Tiempo de viaje de un paquete de origen a destino. Causado por la distancia, el procesamiento del router y, principalmente, el tiempo de espera en colas congestionadas. | **VoIP:** Efecto "walkie-talkie" donde los interlocutores se interrumpen. **Juegos en línea:** Retraso notable entre la acción del jugador y la respuesta del juego ("lag"). **Escritorio Remoto:** Retraso en la respuesta del cursor y la actualización de la pantalla. |
| **Jitter** | Variación en el retardo de llegada entre paquetes consecutivos de un mismo flujo. Causado por fluctuaciones en la congestión y las rutas de la red. | **VoIP:** Audio entrecortado o robótico. **Videoconferencia:** Vídeo pixelado, congelado o desincronizado con el audio. **Streaming de vídeo:** Pausas para el almacenamiento en búfer ("buffering"). |
| **Pérdida de Paquetes** | Descarte de paquetes cuando las colas de los routers se llenan. Causado principalmente por la congestión de la red. | **VoIP:** Sílabas o palabras enteras que desaparecen de la conversación. **Videoconferencia:** Artefactos visuales o congelación momentánea de la imagen. **Transferencia de archivos (TCP):** Reducción drástica de la velocidad de transferencia debido a las retransmisiones. |

Es crucial entender que estos tres "enemigos" no son variables independientes, sino que forman parte de un ciclo de retroalimentación negativa que se intensifica durante la congestión. Cuando un enlace de red se congestiona, la cola de salida de un router comienza a llenarse. Los paquetes que llegan deben esperar, lo que *aumenta directamente la latencia*. Dado que el tráfico de la red llega en ráfagas, la longitud de esta cola fluctúa rápidamente, y esta variación en el tiempo de espera es, por definición, *jitter*. Si la cola se llena por completo, el router no tiene más opción que descartar los paquetes recién llegados, lo que provoca *pérdida de paquetes*. En el caso de las aplicaciones basadas en TCP, la pérdida de paquetes desencadena retransmisiones. Estos paquetes retransmitidos se suman a la carga de tráfico general, lo que puede *empeorar la congestión inicial* y, a su vez, provocar más latencia, jitter y pérdida.[7, 10, 13] Por lo tanto, el objetivo principal de QoS no es solo gestionar una métrica, sino romper este círculo vicioso mediante la gestión inteligente de la causa raíz: el comportamiento de las colas durante la congestión.

### 1.2. El Modelo Arquitectónico: Servicios Diferenciados (DiffServ)

Los primeros modelos de QoS, como los Servicios Integrados (IntServ), intentaban resolver el problema de la calidad de la red exigiendo que cada router en la ruta reservara recursos (como ancho de banda) para cada flujo de tráfico individual. Este modelo, que utilizaba el Protocolo de Reserva de Recursos (RSVP), requería que los routers mantuvieran información de estado para cada conversación, lo que resultaba inviable y no escalaba en redes grandes como Internet.[14]

Como respuesta a esta limitación, se desarrolló el modelo de Servicios Diferenciados (DiffServ), que se ha convertido en el estándar de la industria para implementar QoS de manera escalable.[14] En lugar de gestionar miles de flujos individuales, DiffServ agrupa el tráfico en un pequeño número de clases o categorías de comportamiento. El concepto central de DiffServ es el **Comportamiento por Salto (Per-Hop Behavior, PHB)**.[15, 16]

Bajo este modelo, cada router a lo largo de una ruta toma sus propias decisiones de reenvío de forma independiente, basándose en una marca o etiqueta en la cabecera del paquete (conocida como el valor DSCP). No hay necesidad de una señalización de extremo a extremo ni de que los routers mantengan un estado por flujo. El tráfico se clasifica y se marca en el borde de la red, y luego los routers del núcleo simplemente "honran" esa marca aplicando el PHB preconfigurado correspondiente, como colocar el paquete en una cola de alta prioridad. Esta simplicidad es lo que hace que DiffServ sea tan escalable y eficiente.[14, 17] El objetivo de un diseño de QoS de extremo a extremo es configurar un PHB coherente en todos los dispositivos de la red para lograr la calidad de servicio deseada.

-----

## Parte II: Mecanismos y Herramientas de QoS (Nivel Intermedio)

Esta sección profundiza en las herramientas y técnicas específicas utilizadas para implementar el modelo DiffServ. Se abordan los mecanismos centrales que permiten a los administradores de red identificar, priorizar y controlar el tráfico de manera efectiva.

### 2.1. Clasificación y Marcado: El Punto de Partida de QoS

La primera acción en cualquier política de QoS es un proceso de dos pasos: identificar el tráfico de interés (Clasificación) y luego etiquetarlo (Marcado) para que otros dispositivos en la red puedan reconocer fácilmente su nivel de prioridad y aplicar el tratamiento adecuado.[2, 13, 18]

#### Clasificación (Identificación)

La clasificación es el proceso de inspeccionar un paquete para determinar a qué aplicación o tipo de tráfico pertenece. Existen varios métodos para lograr esto en los dispositivos Cisco:

  * **Uso de Listas de Control de Acceso (ACLs):** Este es el método más tradicional y extendido. Las ACLs pueden identificar el tráfico basándose en la información de la Capa 3 (direcciones IP de origen y destino) y la Capa 4 (puertos TCP y UDP).[19, 20, 21] Por ejemplo, el tráfico web (HTTP/HTTPS) puede ser clasificado haciendo coincidir los puertos TCP 80 y 443, mientras que el tráfico de señalización de VoIP (SIP) puede ser identificado por el puerto UDP 5060.
  * **Uso de Network-Based Application Recognition (NBAR):** NBAR es una tecnología más avanzada de Cisco que realiza una inspección profunda de paquetes (DPI) para identificar aplicaciones. A diferencia de las ACLs, NBAR puede reconocer aplicaciones que utilizan puertos dinámicos o que intentan ofuscar su identidad, como Skype o BitTorrent.[18, 22, 23] La versión más reciente, NBAR2, puede reconocer más de mil aplicaciones y permite la clasificación basada en atributos como la "relevancia empresarial" o la categoría (por ejemplo, `match protocol attribute category voice-and-video`), lo que simplifica enormemente la creación de políticas.[24, 25]

#### Marcado (Etiquetado)

Una vez que un paquete ha sido clasificado, se escribe un valor en su cabecera. Esta marca es la que los dispositivos posteriores en la ruta utilizarán para aplicar el PHB correcto.

  * **Marcado de Capa 2 (Class of Service - CoS):** El marcado de CoS utiliza un campo de 3 bits en la cabecera de la etiqueta VLAN 802.1Q. Esto permite 8 niveles de prioridad (de 0 a 7). Su principal limitación es que solo es significativo dentro de un segmento de red conmutada (LAN), ya que la etiqueta VLAN, y por lo tanto el valor de CoS, se elimina cuando el paquete es enrutado a otra red.[18, 26]
  * **Marcado de Capa 3 (IP Precedence y DSCP):** Este marcado reside en el campo de 8 bits de Tipo de Servicio (ToS) de la cabecera IP y, a diferencia del CoS, persiste de extremo a extremo a través de redes enrutadas.
      * **IP Precedence (IPP):** El estándar original que utilizaba los 3 primeros bits del byte ToS, permitiendo 8 niveles de prioridad (de 0 a 7).[27]
      * **Differentiated Services Code Point (DSCP):** El estándar moderno, fundamental para el modelo DiffServ, que redefine el campo ToS. Utiliza los 6 primeros bits para proporcionar 64 niveles de prioridad distintos (de 0 a 63), ofreciendo una granularidad mucho mayor que IPP.[18, 27, 28] DSCP es retrocompatible con IPP, lo que significa que los dispositivos que solo entienden IPP pueden interpretar correctamente los valores DSCP que se alinean con el antiguo esquema.

#### La Frontera de Confianza (Trust Boundary)

Este es un concepto de diseño y seguridad fundamental en QoS. La frontera de confianza es el punto en la red más allá del cual ya no se confía en las marcas de QoS aplicadas por los dispositivos finales.[15, 18] Típicamente, esta frontera se establece en el puerto del switch de acceso donde se conecta un PC de usuario. La razón es que un usuario malintencionado o una aplicación mal configurada podría marcar su tráfico de baja prioridad (por ejemplo, una descarga de archivos) con el valor de prioridad más alto (EF), lo que le permitiría acaparar los recursos de la red y perjudicar a las aplicaciones críticas.[18, 29, 30]

Por lo tanto, la mejor práctica es que el switch de acceso ignore cualquier marca entrante de un dispositivo no confiable (como un PC), clasifique el tráfico él mismo y aplique la marca correcta según la política corporativa. Una excepción común son los teléfonos IP de Cisco. Estos dispositivos están diseñados para marcar correctamente su propio tráfico de voz (con DSCP EF) y de señalización (con DSCP CS3), por lo que se pueden configurar como dispositivos confiables en el puerto del switch, permitiendo que sus marcas sean aceptadas.[26, 31]

La decisión sobre dónde colocar la frontera de confianza representa un equilibrio fundamental. Confiar en los dispositivos finales simplifica la configuración de la red pero introduce un riesgo de seguridad y abuso de políticas. Por otro lado, hacer que la red (el switch de acceso) realice todo el marcado garantiza la aplicación de la política y la seguridad, pero a costa de una mayor complejidad de configuración en la infraestructura de red.

La siguiente tabla sirve como una guía de referencia esencial para aplicar marcas DSCP estandarizadas y lógicas, basadas en las recomendaciones de RFC 4594.

**Tabla 2: Guía de Marcado DSCP Recomendada (Basada en RFC 4594)**

| Clase de Tráfico | PHB Recomendado | Valor DSCP | Valor Decimal | Descripción del Tratamiento |
| :--- | :--- | :--- | :--- | :--- |
| **Voz (RTP)** | Expedited Forwarding | EF | 46 | Latencia, jitter y pérdida mínimos. Tratamiento de prioridad estricta. |
| **Videoconferencia** | Assured Forwarding | AF41, AF42, AF43 | 34, 36, 38 | Ancho de banda garantizado con baja probabilidad de descarte. |
| **Streaming de Vídeo** | Assured Forwarding | AF31, AF32, AF33 | 26, 28, 30 | Ancho de banda garantizado, pero con mayor tolerancia a la pérdida que la videoconferencia. |
| **Señalización de Llamada** | Class Selector | CS3 | 24 | Importante para el establecimiento de llamadas, requiere un tratamiento mejor que el predeterminado. |
| **Datos Transaccionales** | Assured Forwarding | AF21, AF22, AF23 | 18, 20, 22 | Datos críticos para el negocio que necesitan una entrega asegurada. |
| **Tráfico Masivo / Correo** | Assured Forwarding | AF11, AF12, AF13 | 10, 12, 14 | Tráfico de fondo importante pero no sensible al tiempo. |
| **Scavenger / Baja Prioridad** | Class Selector | CS1 | 8 | Tráfico que debe ser descartado primero durante la congestión (p. ej., descargas P2P). |
| **Mejor Esfuerzo (Default)** | Default Forwarding | DF (o CS0) | 0 | Tráfico sin requisitos especiales de QoS. |

### 2.2. Gestión de la Congestión: Un Análisis Comparativo de las Colas (Queuing)

Los algoritmos de encolamiento son el corazón de la gestión de la congestión. Cuando una interfaz de salida de un router está congestionada (es decir, recibe paquetes más rápido de lo que puede enviarlos), los paquetes se almacenan temporalmente en un búfer o cola. El algoritmo de encolamiento determina el orden en que se atienden estos paquetes.[32]

  * **First-In, First-Out (FIFO):** Es el método más simple y el predeterminado en las interfaces de alta velocidad de Cisco.[33, 34] Los paquetes se transmiten en el mismo orden en que llegan, sin ninguna priorización.[32, 35]
      * **Ventajas:** Es rápido y consume muy pocos recursos de CPU.[36]
      * **Desventajas:** Es inherentemente injusto. Un único flujo de gran ancho de banda, como una transferencia de archivos grande, puede llenar la cola y provocar una alta latencia y pérdida de paquetes para flujos pequeños y sensibles al tiempo, como los de VoIP o Telnet, que quedan atrapados detrás. Este fenómeno se conoce como "inanición" (starvation).[36, 37]
  * **Weighted Fair Queuing (WFQ):** Es un mecanismo automatizado que es el predeterminado en las interfaces de baja velocidad de Cisco (≤ 2.048 Mbps).[35, 38] WFQ clasifica dinámicamente el tráfico en "flujos" (basándose en una combinación de direcciones IP, puertos y protocolo) y asigna una cola separada para cada flujo.[35, 39] Luego, atiende estas colas de manera justa, dando preferencia a los flujos de bajo volumen e interactivos sobre los flujos de alto volumen, al tiempo que comparte el ancho de banda restante de forma equitativa.[40]
      * **Ventajas:** Es mucho más justo que FIFO y protege a los flujos interactivos de la inanición.
      * **Desventajas:** No ofrece al administrador un control directo para garantizar una cantidad específica de ancho de banda a una aplicación concreta.[35]
  * **Class-Based Weighted Fair Queuing (CBWFQ):** Es la piedra angular de las configuraciones modernas de QoS en Cisco. CBWFQ permite al administrador definir sus propias clases de tráfico (utilizando `class-map`) y luego garantizar una cantidad mínima de ancho de banda a cada clase durante los períodos de congestión.[32, 41]
      * **Funcionamiento:** Cada clase definida obtiene su propia cola FIFO.[42, 43] Un programador (scheduler) atiende estas colas basándose en el "peso" derivado del ancho de banda configurado para cada una, asegurando que cada clase reciba su parte garantizada.[44, 45] Si una clase no está utilizando todo su ancho de banda garantizado, el ancho de banda no utilizado se puede compartir proporcionalmente entre las otras clases que lo necesiten.[46, 47]
      * **La Regla del 75%:** Por defecto, Cisco IOS solo permite reservar hasta el 75% del ancho de banda de una interfaz para las colas definidas por el usuario. El 25% restante se reserva para el tráfico de control de la red (como actualizaciones de enrutamiento) y para el tráfico no clasificado que cae en la clase por defecto. Este límite se puede modificar, pero no se recomienda hacerlo sin una cuidadosa consideración.[40, 41]

### 2.3. Control de Tasa: La Dicotomía entre Policing y Shaping

Tanto el policing como el shaping son mecanismos utilizados para limitar el tráfico a una velocidad configurada, conocida como la Tasa de Información Comprometida (CIR, por sus siglas en inglés). Sin embargo, su forma de tratar el tráfico que excede esta tasa es fundamentalmente diferente y dicta sus casos de uso.[48]

  * **Policing (Vigilancia):** Actúa como un estricto guardia de seguridad. Cuando el tráfico excede la CIR, los paquetes excedentes son descartados inmediatamente o, alternativamente, se les cambia la marca a una prioridad inferior (por ejemplo, de AF21 a AF11).[49, 50, 51] El policing no almacena en búfer el tráfico excedente, por lo que no introduce latencia ni jitter adicionales.[52, 53]
  * **Shaping (Modelado):** Actúa como un regulador de tráfico. Cuando el tráfico excede la CIR, los paquetes excedentes se colocan en un búfer (una cola) y se transmiten más tarde, cuando haya ancho de banda disponible.[49, 50, 51] Este enfoque evita la pérdida de paquetes, lo que es beneficioso para las aplicaciones TCP, pero lo hace a costa de introducir latencia y jitter, ya que los paquetes deben esperar en la cola.[5, 54]

Una restricción técnica clave es la dirección en la que se pueden aplicar. El policing se puede aplicar tanto al tráfico que entra (entrada/inbound) como al que sale (salida/outbound) de una interfaz. El shaping, debido a que depende de una cola de salida, solo se puede aplicar al tráfico de salida.[48, 55]

Los casos de uso en el mundo real ilustran claramente esta diferencia:

  * **Caso de Uso del Policing:** Es más comúnmente utilizado por los proveedores de servicios de Internet (ISP) para hacer cumplir un Acuerdo de Nivel de Servicio (SLA). Si una empresa contrata una conexión de 100 Mbps, el ISP aplicará un *policer* de entrada en su router de borde para descartar cualquier tráfico proveniente del cliente que supere los 100 Mbps. Esto protege la red del ISP y garantiza que el cliente no consuma más recursos de los que ha pagado.[48, 53, 56, 57]
  * **Caso de Uso del Shaping:** Es más comúnmente utilizado por los clientes empresariales para cumplir con ese mismo SLA. La empresa aplicará un *shaper* de salida en su router de borde WAN, configurando la velocidad ligeramente por debajo de la tasa contratada (por ejemplo, a 98 Mbps para un circuito de 100 Mbps). Esto suaviza las ráfagas de tráfico y garantiza que el flujo de datos que sale de su red nunca supere el límite del *policer* del ISP, evitando así la pérdida de paquetes.[48, 58, 59]

La siguiente tabla resume las diferencias clave entre estos dos mecanismos, sirviendo como una guía para decidir cuál utilizar en un escenario determinado.

**Tabla 3: Policing vs. Shaping: Un Resumen Comparativo**

| Característica | Policing (Vigilancia) | Shaping (Modelado) |
| :--- | :--- | :--- |
| **Acción sobre Tráfico Excedente** | Descarta o re-marca los paquetes. | Almacena los paquetes en un búfer para enviarlos más tarde. |
| **Dirección de Aplicación** | Entrada (Inbound) u Salida (Outbound). | Solo Salida (Outbound). |
| **Impacto en Latencia/Jitter** | Bajo. No introduce retardo adicional. | Alto. Introduce tanto latencia como jitter. |
| **Uso de Recursos del Router** | Bajo (CPU). | Alto (CPU y Memoria para el búfer). |
| **Caso de Uso Principal** | **Proveedor de Servicios:** Hacer cumplir un SLA. | **Cliente Empresarial:** Cumplir con un SLA. |
| **Impacto en TCP** | Causa pérdida de paquetes, lo que provoca retransmisiones y una reducción brusca de la ventana de TCP. | Suaviza el flujo de tráfico, lo que generalmente resulta en un mejor rendimiento para TCP al evitar pérdidas. |

-----

## Parte III: Implementación y Escenarios Avanzados (Nivel Avanzado)

Esta sección final integra los mecanismos de la Parte II en soluciones cohesivas y de extremo a extremo para problemas del mundo real. Se demuestran configuraciones avanzadas y principios de diseño que son cruciales para una implementación exitosa de QoS en redes empresariales complejas.

### 3.1. Low Latency Queuing (LLQ): Prioridad Absoluta para Tráfico en Tiempo Real

Aunque CBWFQ es excelente para garantizar un ancho de banda mínimo a las clases de tráfico, sigue siendo un sistema de encolamiento "justo", lo que significa que todas las colas definidas tienen su turno para enviar paquetes. Para el tráfico de voz, que es extremadamente sensible al retardo, incluso una pequeña espera para su "turno" durante la congestión puede degradar la calidad de la llamada.[60]

La solución a este problema es el **Low Latency Queuing (LLQ)**. LLQ es una mejora de CBWFQ que introduce una única y estricta **Cola de Prioridad (Priority Queue, PQ)**.[61, 62] El tráfico que se coloca en esta cola de prioridad (utilizando el comando `priority` dentro de un `policy-map`) *siempre* se atiende antes que cualquiera de las colas estándar de CBWFQ. Esto garantiza la menor latencia posible y es el mecanismo preferido para el tráfico de voz y, a menudo, para el de videoconferencia interactiva.[47, 60]

Un problema potencial con una cola de prioridad tradicional es que, si está constantemente llena de tráfico, podría acaparar todo el ancho de banda de la interfaz, dejando a las otras colas sin servicio (inanición). LLQ resuelve este problema de manera elegante al incluir un *policer* implícito e integrado. La cola de prioridad tiene garantizado un cierto ancho de banda, pero también está limitada a esa cantidad durante los períodos de congestión. Esto evita que consuma todo el ancho de banda de la interfaz y asegura que las colas de CBWFQ también reciban servicio, combinando lo mejor de la prioridad estricta con la protección contra la inanición.[63, 64]

### 3.2. Ejemplo Real 1: Diseño de QoS en una Red Corporativa con VoIP

**Escenario:** Una oficina tiene un enlace WAN de 10 Mbps. La red transporta llamadas VoIP, tráfico hacia una aplicación web interna crítica para el negocio y navegación general por Internet. El objetivo es garantizar una calidad de llamada perfecta, que la aplicación crítica sea siempre receptiva y que la navegación web utilice el ancho de banda restante.

**Estrategia de Configuración de Extremo a Extremo utilizando la CLI de QoS Modular (MQC):**

1.  **Paso 1: Clasificación (en el Router de Borde):** Se crean `class-map` para identificar los diferentes tipos de tráfico.

      * `class-map VOICE`: Se configura para que coincida con el tráfico que ya ha sido marcado con DSCP EF (asumiendo que los teléfonos IP son confiables y marcan sus propios paquetes con el valor 46).[61, 65]
      * `class-map CRITICAL-APP`: Se configura para que coincida con el tráfico hacia y desde la dirección IP y el puerto del servidor de la aplicación web (por ejemplo, utilizando una ACL).[65]
      * El tráfico restante (navegación web, etc.) caerá automáticamente en la clase por defecto, `class-default`.

2.  **Paso 2: Creación de la Política (en el Router de Borde):** Se crea un `policy-map` para definir el comportamiento de encolamiento.
    \! Creación de las clases de tráfico
    class-map match-any VOICE
    match ip dscp ef
    \!
    class-map match-any CRITICAL-APP
    match access-group name ACL\_CRITICAL\_APP
    \!
    \! Creación de la política de QoS
    policy-map WAN-EGRESS-POLICY
    class VOICE
    priority percent 30\! LLQ: 30% (3 Mbps) de prioridad estricta para la voz
    class CRITICAL-APP
    bandwidth percent 40\! CBWFQ: 40% (4 Mbps) de ancho de banda garantizado para la app
    class class-default
    fair-queue       \! WFQ: El resto del tráfico comparte el ancho de banda de forma justa

    ```
    *   `class VOICE`: El comando `priority percent 30` crea la LLQ, garantizando 3 Mbps para la voz y dándole prioridad estricta sobre todo lo demás.[61, 63]
    *   `class CRITICAL-APP`: El comando `bandwidth percent 40` crea una cola CBWFQ, garantizando un mínimo de 4 Mbps para la aplicación crítica durante la congestión.[65]
    *   `class class-default`: El comando `fair-queue` asegura que todo el tráfico restante (el 30% restante, o 3 Mbps) se coloque en esta clase y se atienda utilizando WFQ, lo que garantiza la equidad entre las diferentes sesiones de navegación web.[65, 66]

    ```

3.  **Paso 3: Aplicación (en el Router de Borde):** La política se aplica a la interfaz WAN en la dirección de salida.

    ```
    interface Serial0/0/0
     description Enlace WAN a la Sucursal
     service-policy output WAN-EGRESS-POLICY
    ```

    Este comando activa la política de QoS para todo el tráfico que sale de la red a través del enlace WAN.[61, 65]

4.  **Verificación:** El comando `show policy-map interface Serial0/0/0` es esencial para verificar el funcionamiento. Muestra contadores de paquetes para cada clase, confirmando que el tráfico se está clasificando correctamente. Durante la congestión, se puede observar qué colas están acumulando paquetes y si se están produciendo descartes (drops), que deberían ocurrir en las colas de menor prioridad.[46, 61]

### 3.3. Ejemplo Real 2: Interconexión WAN y Cumplimiento de SLA con un ISP

**Escenario:** La misma empresa del Ejemplo 1 actualiza su conexión WAN. Ahora tienen una interfaz física de fibra de 1 Gbps, pero solo pagan a su ISP por un servicio de 100 Mbps. El ISP hace cumplir este límite con un *policer* en su extremo, que descarta sin piedad todo el tráfico que supere los 100 Mbps.[58, 67]

**El Problema:** La interfaz de 1 Gbps del router del cliente nunca experimentará congestión por sí misma, ya que su capacidad física es 10 veces mayor que la tasa contratada. Como resultado, la política de QoS del Ejemplo 1 (que se activa solo durante la congestión) nunca se activará. El router enviará ráfagas de tráfico de hasta 1 Gbps, que luego serán descartadas indiscriminadamente por el *policer* del ISP. Esto destruirá el rendimiento de todas las aplicaciones, incluida la voz, ya que los paquetes de voz se descartarán junto con todo lo demás.[67, 68]

**La Solución con QoS Jerárquico (Política Anidada):**

La solución es crear un punto de congestión *artificial* dentro del propio router del cliente, que refleje el límite del contrato del ISP. Esto se logra con una política de dos niveles (padre e hijo).

1.  **Paso 1: Crear la Política Hijo:** La política `WAN-EGRESS-POLICY` del Ejemplo 1, que contiene la lógica de encolamiento (LLQ y CBWFQ), se convierte en nuestra política "hijo".

2.  **Paso 2: Crear la Política Padre:** Se crea un nuevo `policy-map` que actuará como el "padre". Su única función es modelar (shape) todo el tráfico a la velocidad contratada por el ISP.

    ```
    ```

\! La política hijo (del ejemplo anterior) contiene la lógica de priorización
policy-map WAN-EGRESS-POLICY
class VOICE
priority percent 30
class CRITICAL-APP
bandwidth percent 40
class class-default
fair-queue
\!
\! La política padre aplica el modelado y anida la política hijo
policy-map WAN-SHAPER
class class-default
shape average 98000000\! Modela todo el tráfico a 98 Mbps
service-policy WAN-EGRESS-POLICY\! Anida la política hijo aquí
\`\`\`
\*   El comando `shape average 98000000` modela todo el tráfico a 98 Mbps (ligeramente por debajo de la CIR para tener en cuenta la sobrecarga y las ráfagas).[67, 69]
\*   El comando \`service-policy WAN-EGRESS-POLICY\` anida la política hijo dentro de la padre. Este es el paso clave.

3.  **Paso 3: Aplicación:** Solo la política padre se aplica a la interfaz WAN de salida.

    ```
    interface GigabitEthernet0/0/1
     description Enlace WAN al ISP
     service-policy output WAN-SHAPER
    ```

    [69]

**Análisis del Impacto:** El *shaper* en la política padre crea un punto de congestión artificial dentro del router. Cuando el router intenta enviar más de 98 Mbps, el *shaper* retiene los paquetes excedentes en su búfer. Ahora que existe congestión *dentro del router*, la política hijo anidada (la lógica de encolamiento) se activa. Esta política puede ahora reordenar y priorizar inteligentemente los paquetes retenidos en el búfer del *shaper* *antes* de que se envíen al ISP. Esto asegura que la voz salga primero, las aplicaciones críticas en segundo lugar, y que la tasa total nunca exceda el límite del ISP, evitando así por completo los descartes del proveedor.[59, 68]

Esta técnica de QoS jerárquico es fundamental para cerrar la brecha entre los contratos de servicio lógicos (SLAs) y la infraestructura de red física. Es la solución práctica al modelo de servicio de "sub-tasa" que es omnipresente en las redes WAN modernas. Sin ella, cualquier otra política de encolamiento en un escenario de este tipo es, en la práctica, inútil.

### 3.4. Tópicos Avanzados y Mejores Prácticas de Diseño

  * **Gestión vs. Prevención de la Congestión:**
      * El encolamiento (FIFO, WFQ, CBWFQ, LLQ) es una técnica de **gestión de la congestión**. Es reactiva; decide qué hacer con los paquetes *después* de que la congestión ya ha ocurrido.[70]
      * **Weighted Random Early Detection (WRED)** es una técnica de **prevención de la congestión**. Es proactiva. A medida que una cola comienza a llenarse, WRED empieza a descartar *aleatoriamente* algunos paquetes de baja prioridad *antes* de que la cola esté completamente llena.[40, 70] Este descarte temprano sirve como una señal para los remitentes de TCP para que reduzcan su velocidad de transmisión, evitando así los descartes masivos y sincronizados (conocidos como "tail drop") que ocurren cuando una cola se desborda. Esto previene un fenómeno llamado "sincronización global de TCP", que puede degradar gravemente el rendimiento.[71, 72]
  * **Principios de Diseño de Extremo a Extremo:** Es vital recordar que QoS no es una configuración que se realiza en un solo dispositivo. Para que sea verdaderamente efectiva, las marcas deben ser coherentes y se debe aplicar una política cohesiva en cada punto potencial de congestión a lo largo de toda la ruta de la red. Un diseño de QoS exitoso requiere una planificación holística que abarque desde el switch de acceso en la LAN hasta el router de borde de la WAN y a través de la red del proveedor de servicios.[15, 29, 73]

## Conclusión

Este informe ha recorrido el panorama de la Calidad de Servicio en redes Cisco, desde la comprensión de los impedimentos fundamentales de la red, como la latencia, el jitter y la pérdida de paquetes, hasta la implementación de políticas jerárquicas sofisticadas capaces de gestionar entornos de red complejos. Se ha demostrado que QoS es un conjunto de herramientas interdependientes —clasificación, marcado, encolamiento, vigilancia y modelado— que deben ser aplicadas de manera cohesiva para lograr los resultados deseados.

La clave del éxito con QoS no reside en la memorización de comandos, sino en la comprensión de los principios subyacentes. La clasificación y el marcado establecen la base de la política, la gestión de la congestión a través de mecanismos de encolamiento como CBWFQ y LLQ proporciona la priorización durante los períodos de contención, y las herramientas de control de tasa como el policing y el shaping aseguran el cumplimiento de los acuerdos de servicio.

Finalmente, es crucial entender que la Calidad de Servicio no es un proyecto de "configurar y olvidar". Es un ciclo de vida que requiere un diseño inicial cuidadoso basado en los requisitos del negocio, una implementación meticulosa, un monitoreo continuo del rendimiento y un ajuste constante para adaptarse a las nuevas aplicaciones y a los patrones de tráfico cambiantes.[5, 74] Al adoptar este enfoque, los administradores de red pueden transformar la red de un simple sistema de entrega de paquetes de "mejor esfuerzo" a un activo estratégico para el negocio, capaz de soportar de manera fiable las aplicaciones de misión crítica que impulsan a la empresa moderna.

```
```
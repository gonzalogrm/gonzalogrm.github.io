---
layout: post
title:  "Generación Dinámica de Textos mediante cadenas de Markov"
date:   2020-11-19 14:05:31 +0100
categories: text generation
---
### I. Redes neuronales y cadenas de Markov

La generación de contenidos de forma procedural —generación por procedimientos o procedimental— se ha impuesto como uno de los paradigmas creativos de mayor relevancia en múltiples disciplinas relacionadas con la programación y el diseño. La generación procedural permite crear recursos de forma dinámica a partir de un algoritmo, obteniendo una infinidad de posibles resultados en tiempo real mediante un único conjunto de instrucciones, en lugar de trabajar manualmente en cada recurso uno por uno. Esta automatización —que puede ser controlada por medio de parámetros— también puede tener un componente de aleatoriedad; de modo que nunca sepamos con antelación cuál será el próximo resultado de la iteración del algoritmo, permitiendo que esta indeterminación en los resultados forme parte de las mecánicas internas del proyecto que implemente la generación procedural.

Ejemplos concretos de implementación de este tipo de algoritmos vienen dándose desde hace décadas en la generación aleatoria de mapas en videojuegos. Dicha generación suele estar asociada a un número identificador conocido como semilla. La elección de la semilla se hace de forma aleatoria, garantizando variedad en los mapas; no obstante, una semilla concreta siempre debe generar el mismo mapa, garantizando el control en la automatización. En computación gráfica, la generación procedural se utiliza para generar texturas, animaciones y modelos tridimensionales que requieran una gran variación o necesiten ser generados bajo demanda con unas condiciones que no se conocen hasta el momento de la petición, haciendo imposible que la textura o modelo estuviese generado con antelación.

Recientemente, se ha comenzado a popularizar la idea de emplear la generación por procedimientos y otras técnicas de programación —NLP o Natural Language Processing— como mecanismo de producción de textos que imitan el lenguaje humano. Uno de los arquetipos más populares es el uso de redes neuronales y machine learning [1], muy señaladamente los modelos GPT-2 [2] y GPT-3 [3] de la organización OpenAI. Este enfoque, aun siendo el más prometedor, tiene una complejidad inasumible para la mayoría de programadores que buscan desarrollar por cuenta propia y desde cero sus propios algoritmos y sólo está al alcance de grupos de investigación —el modelo GPT-3 está firmado por 31 autores, 621 participantes en el control de calidad de los resultados y, según distintas fuentes, el proceso de entrenamiento tuvo un coste cercano a los 12 millones de dólares—.

Otra posibilidad para la generación de textos se basa en el uso de cadenas de Markov. Los métodos de escritura procedural basados en procesos de Markov —procesos estocásticos sin memoria— permiten generar una familia de algoritmos muy flexibles a partir de los cuales se obtienen textos generados dinámicamente en base a un texto de entrada con el que se entrena el algoritmo. Una de las principales ventajas de los algoritmos basados en procesos de Markov es que se pueden obtener textos moderadamente legibles con muy pocas instrucciones de código, haciendo que sea un buen punto de entrada para comenzar a indagar en el campo de la generación procedural de textos. Por contra, esta simplicidad hace que los textos generados, aunque sean gramaticalmente correctos en la mayoría de los casos, no tengan una semántica clara y en muchos casos carezcan de sentido.

<br>

### II. Implementación básica de un algoritmo generador de textos

Los casos más sencillos de implementación de cadenas de Markov se basan en una estructura de datos tipo ``Map<Key, List<Values>>``, es decir, un mapa —o diccionario, en función del lenguaje de programación— que asocia a una clave, Key, una lista de valores, ``List<Values>>``, siendo tanto Key como Values cadenas de caracteres (String). La elección de la estructura de la clave es el primer paso para la generación de cadenas de Markov. Generalmente, la clave se forma con n-gramas compuestos por una o más palabas del texto de entrenamiento. Para esta primera implementación utilizamos un n-grama de primer orden, es decir, las claves están formadas únicamente por una palabra; pero también podríamos utilizar claves formadas por dos o más palabras. Generalmente, no es recomendable ir más allá de tercer orden dado que el mapa resultante puede llegar a tener demasiadas claves, y los textos generados pueden parecerse demasiado al texto original, en función de la extensión de éste.

Al ejecutar el algoritmo se recorre secuencialmente el texto de entrenamiento palabra por palabra para construir el mapa de correspondencias: a cada n-grama clave se le asocia la lista de palabras que le suceden. Por ejemplo, utilizando como texto de entrenamiento el siguiente extracto del Génesis centrándonos en la palabra “Dios”: “En el principio creó Dios los cielos y la tierra. La tierra era caos y confusión y oscuridad por encima del abismo, y un viento de Dios aleteaba por encima de las aguas. Dijo Dios: «Haya luz», y hubo luz. Vio Dios que la luz estaba bien, y apartó Dios la luz de la oscuridad; y llamó Dios a la luz «día», y a la oscuridad la llamó «noche». Y atardeció y amaneció: día primero”. El n-grama formado por la clave “Dios” tendrá asociado los valores: los, aleteaba, que, la, a. Es decir, {Dios:[los, aleteaba, que, la, a]}. Nótese que se ha omitido una ocurrencia de la palabra Dios, en Dijo Dios: «Haya luz» ya que, en este caso, el algoritmo no lee “Dios”, sino “Dios:” y se consideran cadenas de caracteres distintas. En este punto, el algoritmo ha analizado el texto y comprende que, históricamente, siempre que ha aparecido la palabra “Dios”, ha sido sucedida por alguna de las palabras contenidas en ``List<Values>``; por tanto, si el algoritmo estuviese en la fase de escritura de texto, sabría qué palabras puede escoger cuando apareciese este n-grama clave en concreto.

En este caso, las únicas combinaciones disponibles para generar serían: Dios los, Dios aleteaba, Dios que, Dios la, Dios a. Supongamos que el algoritmo decide escribir “Dios la”. Repitiendo el proceso, ahora el n-grama clave sería “la”. Analizando el texto, se obtiene el siguiente mapa: ``{la:[tierra, luz, luz, oscuridad, luz, oscuridad, llamó]}``. 

Es importante que las palabras puedan aparecer repetidas en la ``List<Values>``, por lo que hay que evitar estructuras tipo Set —todas aquellas que no admiten valores repetidos—. De la lista de valores asociados a la Key “la” concluimos que, estadísticamente, la palabra más frecuente ha sido “luz”; supongamos que el algoritmo en fase de escritura escoge dicha palabra, la frase escrita hasta este punto quedaría como: “Dios la luz”. Así, continuaríamos el proceso hasta obtener una frase de la longitud deseada. Naturalmente, cuanto más variado y extenso sea el texto de entrada, mejor será el resultado de las frases obtenidas. Como aplicación real del extracto del Génesis sobre el algoritmo podríamos obtener frases tales como: “Dijo Dios: «Haya luz», y confusión y oscuridad por encima del abismo, y apartó Dios la luz”.

<br>

### III. Mapas anidados

Con esta implementación sencilla, tenemos una lista de salida que permite conocer qué palabras emergen de cada n-grama clave. Es evidente que ``List<Values>`` es en realidad una lista de palabras sucesoras asociadas a una clave: ``List<Sucesoras>``. Sin embargo, no tenemos una lista de entrada que nos permita saber qué palabras le han precedido. En otras palabras, podemos conocer el conjunto de palabras candidatas a suceder después de la aparición de cada clave ``(Key -> List<Sucesoras>)``, pero no conocemos el conjunto de palabras que han conducido previamente a cada clave ``(List<Predecesoras> -> Key)``. Nuestra propuesta de modelo busca mitigar esta y muchas otras carencias asociadas las implementaciones básicas de algoritmos que utilizan cadenas de Markov. Concretamente, contempla la posibilidad de obtener el flujo completo asociado a cada n-grama clave: ``List<Predecesoras> -> Key -> List<Sucesoras>``. Para ello, reemplazamos la estructura de datos tipo ``Map<Key, List<Values>>`` por otra más compleja:


`Map<MK, Map<PK, List<SV>>>`



Donde MK hace referencia a Main Key o n-grama clave; PK hace referencia a Predecesor Key o clave predecesora; es decir, alguna de las palabras contenida en la anterior ``List<Predecesoras>``; y SV hace referencia a Successor Values: alguno de los posibles resultados de la combinación (PK, MK) —al igual que antes, PK, MK y SV son cadenas de caracteres (String)—. Tal y como hemos indicado, cada elemento del mapa es a su vez otro mapa que permite generar correspondencias tipo ``(PK, MK) -> List<SV>``. Es decir, a cada pareja formada por un n-grama clave y alguna de las palabras predecesoras le corresponde de forma unívoca una única lista de palabras sucesoras.

Por otro lado, la elección de la próxima palabra en el proceso de escritura —sin duda, la parte más importante de la generación del texto— no es meramente probabilística, sino que se basa en una función de elección —Choice Function o CF— que permite, entre otras cuestiones, suprimir en la medida de lo posible una lista de palabras censuradas, forzar el uso de una lista de palabras privilegiadas, hacer que el texto comience por una palabra o frase determinada, sustituir en tiempo real unas palabras por otras, introducir imprecisiones y realizar inserciones en tiempo de ejecución de nuevas correspondencias ``(PK,MK) -> List<SV>``. Estas y otras funcionalidades provistas por la CF permiten amortiguar el carácter puramente estadístico de las cadenas de Markov, haciendo que sea posible tener cierto control sobre el texto generado, reduciendo la aleatoriedad.

La CF recibe como parámetro de entrada la ``List<SV>`` de palabras sucesoras candidatas asociadas a la pareja (Predecesor Key, Main Key), así como dos ``Set<String>`` de palabras privilegiadas y palabras censuradas —una estructura tipo HashSet, o el equivalente del lenguaje de programación a utilizar, sería la opción recomendable dado que las palabras no están repetidas y se quiere tener un acceso inmediato—. Con estos datos, la CF filtra la ``List<SV>`` eliminando —en el caso de que sea posible eliminarlas sin resultar en una lista vacía de palabras candidatas— aquellas palabras que estén censuradas. Si entre la lista de palabras candidatas está alguna del ``Set<String>`` de palabras privilegiadas, la CF creará una lista en paralelo con dichas palabras, de tal modo que se garantizará la elección de alguna de las palabras privilegiadas. Por otra parte, al conocer ``List<Predecesoras> -> Key -> List<Sucesoras>``, la CF puede buscar a segundo orden; es decir, puede elegir una palabra que no siendo privilegiada conduzca a una que sí lo sea, haciendo posible que la próxima iteración de la CF dé como resultado la palabra buscada. Las condiciones de búsqueda de la CF —los dos ``Set<String>``— se pueden modificar en tiempo de ejecución tanto por el usuario como por el propio algoritmo sin modificar el mapa de correspondencias con un impacto mínimo en el rendimiento.

Es importante remarcar que la sustitución y forzamiento de palabras en tiempo de ejecución no es trivial. Es decir, no podemos añadir forzadamente palabras en el texto ni se trata una sustitución superficial de una palabra por otra del tipo:

String x = «A B»;        x = x.replace(«B», «D»);

No podemos hacer este tipo de sustituciones y forzamientos debido a que el algoritmo funciona mediante correspondencias y al añadir una palabra forzadamente podríamos estar generando correspondencias que no existen en el mapa. Por ejemplo, si el texto fuese “A B C” y quisiéramos forzar la aparición de la palabra “D”, no sería adecuado añadirla sin más ya que podríamos estar generando una correspondencia inexistente ``(C,D) -> ?``, y el algoritmo no sabría cómo continuar al no tener listas de palabras candidatas con D como Main Key y C como Predecesor Key. Es por ello que las sustituciones y forzamientos de palabras en tiempo de ejecución han de realizarse mediante métodos que respeten la congruencia de las correspondencias. De forma análoga, hacer que el texto comience con una palabra concreta no es un proceso inmediato; la palabra debe existir como elemento del mapa y, a partir de ella, se debe buscar una correspondencia adecuada para continuar el proceso de escritura: ``(A,B)->C, (B,C)->D …``

Respecto a la inserción de nuevas correspondencias en el mapa una vez éste ya ha sido generado, se ha de tener en cuenta que el algoritmo utiliza condiciones de contorno periódicas (PBC) para generar el mapa inicial. Es decir, si el texto de entramiento fuese “A B C D E F” y el algoritmo trabajara con n-gramas de primer orden; se añadirían dos palabras adicionales para tener un texto cíclico “A B C D E F A B” y garantizar que el mapa de correspondencia no tenga ninguna palabra que no conduzca a otra. Se puede comprobar que los casos límites quedan cubiertos: ``(E,F)->A, (F,A)->B``. Teniendo esto en cuenta, si queremos añadir nuevo texto al mapa con n-gramas de primer orden, habría que mantener las condiciones PBC:     

> Valores iniciales: A B
> 
> Valores finales: E F
> 
> Texto a insertar: G H I J
> 
> Resultado con PBC: E F \| G H I J \| A B

Una vez el texto a insertar incluye los valores del PBC, se puede actualizar el mapa que contiene las correspondencias asociadas al primer texto de entrenamiento con las correspondencias que puedan surgir del nuevo bloque de texto. Nótese que al haber aplicado las PBC, todas las condiciones límite en los extremos están contempladas, y no hay ninguna palabra que quede sin formar parte de alguna correspondencia. Como mínimo se tendrá: ``(E,F)->G, (F,G)->H … (J,A)->B``. Por tanto, la nueva cadena de texto queda integrada en el mapa de correspondencias inicial y el algoritmo puede usarlas para generar textos que incluyan el nuevo bloque de entrada.

Otro aspecto importante relativo a nuestro modelo de generación de textos, es que el algoritmo presta atención a lo que ha escrito con anterioridad y lo tiene en cuenta en la CF para elegir las palabras siguientes. Es decir, el algoritmo intenta mantener una serie de temáticas basadas en palabras clave que ha ido utilizando a lo largo de todo el texto, permitiendo obtener un resultado mucho más coherente respecto a modelos sencillos que no implementan esta funcionalidad.

Finalmente, mientras el algoritmo construye el texto, vigila las palabras que ha incluido en la frase actual para intentar no repetir —o repetir lo mínimo en la medida de lo posible— palabras que ya haya escrito en la misma frase, consiguiendo que la frase sea más natural. La estructura general en pseudocódigo de la CF quedaría en la forma:

{% highlight java %}
String ChoiceFunction(){
 //-------- Tratar de terminar la frase      
             if(frase actual comienza a ser demasiado larga)
                         buscar entre List<SV> un final de frase
                         return candidato
              
 //-------- Filtrar lista de candidatos
 if(frase actual es excesivamente corta){
             eliminar finales de frase de List<SV>
             eliminar palabras censuradas de List<SV>
             eliminar palabras que ya se hayan repetido en la frase actual de List<SV>
             comprobar que de List<SV> no ha quedado vacía
             if(List<SV> ha quedado vacía)
                         reducir las restricciones y volver a intentar
 }else (frase actual tiene longitud adecuada){
             eliminar palabras censuradas de List<SV>
             eliminar palabras que ya se hayan repetido en la frase actual de List<SV>
             comprobar que de List<SV> no ha quedado vacía
 if(List<SV> ha quedado vacía)
             reducir las restricciones y volver a intentar
 }
              
 //-------- Operaciones con la List<SV> filtrada
 Buscar palabra privilegiada a primer orden
             if(List<SV> filtrada contiene palabras privilegiadas)
             return privilegiada
   
 Buscar palabra privilegiada a segundo orden
 if(List<SV> filtrada contiene palabras no-privilegiadas que conducen a alguna privilegiada en próxima iteración)
             return no-privilegiada
   
 Centrar la atención en las frases anteriores
 if(List<SV> filtrada contiene alguna palabra clave ya escrita en alguna frase anterior)
             return atención
   
 //Si no se ha encontrado ninguna palabra privilegiada ni le ha llamado la atención ninguna palabra clave de frases anteriores, devolver algún candidato
     return candidato
 }
 {% endhighlight %}

Donde cada uno de los filtros, operaciones, búsquedas a primer y segundo orden, y demás instrucciones de la secuencia de pseudocódigo se realizan a su vez mediante funciones específicas para cada caso cuya implementación no discutiremos aquí.

Como se puede observar, la CF está estructurada por subprocesos que se ordenan de mayor a menor prioridad. En primer lugar, la CF intentará buscar un final de frase si ésta comienza a ser demasiado larga. En segundo lugar —si la frase tiene una longitud adecuada—, intentará filtrar la lista de palabras candidatas teniendo en cuenta si mantiene o no los finales de frase, las palabras repetidas en la frase actual y retirando palabras censuradas. Una vez la lista de candidatos ha sido filtrada, el siguiente paso en términos de prioridad es buscar a primer orden una palabra privilegiada; si no la encuentra, tratar de buscarla a segundo orden. Posteriormente, si no ha encontrado ninguna palabra privilegiada intenta centrar la atención en palabras clave anteriormente utilizadas para mantener la coherencia en el texto. Finalmente, si entre la lista de candidatos no se ha encontrado ninguna palabra que satisfaga alguno de los filtros, la CF escogerá de forma probabilística una de las palabras candidatas de List<SV>; siendo más probable que escoja una palaba popular respecto a otras poco usadas —que también podrían ser escogidas con menor probabilidad—.

<br>

### IV. Alternativas a los mapas anidados

La motivación de nuestro modelo era construir un algoritmo que, partiendo de la lógica de modelos de Markov, pudiera:
- Generar el flujo completo asociado a cada n-grama clave: ``List<Predecesoras> -> Key -> List<Sucesoras>``
- Generar correspondencias tipo ``(PK, MK) -> List<SV>``

Para ello se ha propuesto el uso de mapas anidados ``Map<MK, Map<PK, List<SV>>>``. No obstante, el uso de mapas no es gratuito; de hecho, el uso a gran escala de mapas tiene un impacto negativo en la utilización de memoria. En nuestro caso, se estaría generando un mapa interno asociado a los distintos Predecessor Key (PK) por cada una de las Main Key (MK) del mapa externo. Una alternativa sería implementar un único mapa sin mapas internos que usara como clave directamente las correspondencias. Es decir:

{% highlight java %}
Map<(PK,MK), List<SV>>
{% endhighlight %}

Aunque esta estructura es más sencilla —al no utilizar mapas anidados— y tiene un menor impacto en el uso de memoria, muchas de las operaciones que nuestra propuesta con mapas anidados realiza sin problemas pasan a ser menos eficientes en esta segunda configuración. Por ejemplo, si en nuestro modelo queremos conocer todas las listas List<SV> asociadas a una Main Key sin centrarnos en ninguna PK; es decir, buscamos realizar la operación ``(MK) -> [List<SV>]`` —donde los corchetes indican conjunto o colección de más de una lista—, sólo tenemos que unir las listas del mapa interno asociado a MK sin preocuparnos por ninguna PK; es decir, obtener el conjunto de valores del ``Map<PK, List<SV>>`` con una operación tipo map.values()->[List<SV>]. Sin embargo, para hacer esta misma operación utilizando un ``Map<(PK,MK), List<SV>>`` tendríamos que recorrer todas las correspondencias que actúan como claves del mapa e ir verificando una por una cuales contienen la MK buscada y, una vez encentradas todas, unir las ``List<SV>`` asociadas. Dicha búsqueda podría ser potencialmente lenta dado que pueden existir muchas correspondencias con la misma MK: (PK_1, MK), (PK_2, MK), (PK_3, MK) …, y este proceso de búsqueda desaprovecha el rendimiento que se puede obtener de una estructura tipo Map, ya que ésta permite acceder a los elementos mediante hashing sin tener que recorrerlos secuencialmente.

### V. Resultados

En esta sección pasamos a mostrar resultados concretos de nuestra implementación del algoritmo con n-gramas de primer orden. Una de las dificultades presentes en modelos entrenados a partir de textos preexistentes es, precisamente, el propio proceso de entrenamiento del algoritmo. El texto de entrada tiene que estar pulido: no presentar inconsistencias ni faltas de ortografía ya que el algoritmo sólo va a saber escribir a partir de los patrones que deduzca del texto de entrenamiento. Para hacernos una idea de la escala de este proceso, cabe destacar que el modelo GPT-2 usa como textos de entrenamiento una colección de 8 millones de documentos con un tamaño total de 40 GB; mientras que su sucesor, el modelo GPT-3 parte de 45 TB filtrados hasta obtener 570 GB de texto plano.

En nuestro caso, por limitaciones en la extensión de los textos de entrada a nuestra disposición, vamos a examinar la situación contraria; es decir, tratamos de generar textos partiendo de ficheros de tamaño muy reducido. Para la primera producción de resultados utilizamos varios capítulos de dos textos que contienen extractos sobre la vida y obra de los alquimistas más conocidos: Memoirs of Extraordinary Popular Delusions and the Madness of Crowds (1852) y The Story of Alchemy and the Beginnings of Chemistry (1902), con una extensión total de 0.49 MB:

**Ejemplo 1**

|I have two failings: my poverty and destitution. The golden or silver reduced to two serpents. The fixed nature is spiritual, it would have done already. Mention is made in the nature of man, and desired nothing better; Muhlenfels, being provided with some difficulty. He found that he made gold enough to have some description of our chagrin, especially of the day. Agrippa needlessly ran foul of this description was, for a long conversation with him I had obtained.When I had made of the silver he had found in tin and lead is changed into fire. That interpretation was perfectly reasonable at a time when he found that Raymond Lulli resided for some time, and made him one of gold would cure all diseases. Secondly. That they sholden discovers it unto noon, Ne in no book it write in no wise loath, and promised him his golden box, containing two pieces of gold made out of him. He provided Borri with the book of Rhazes continually before him. The star of his changed fortune, he had, like a dog without trial, than plead either guilty or not guilty before such contemptible miscreants. But his confidence forsook him as they began writing down his discourse.|



En este primer ejemplo se han marcado con distintos colores las palabras a las que el algoritmo ha ido prestando atención volviendo recurrentemente sobre ellas con el fin de mantener una coherencia semántica a lo largo del proceso de escritura. Se puede comprobar que ha tratado de mantener la misma persona —he, him, his— a lo largo del texto, así como el uso reiterado de determinadas palabras que ha considerado relevantes. Debido a la escasa extensión del texto de entrada —0.49 MB frente a los ≈570.000 MB que puede llegar a usar un modelo tipo GPT-3— el texto carece de la precisión semántica que cabría esperar de algoritmos más sofisticados. No obstante, a pesar de la limitada longitud de nuestro texto de entramiento, el resultado obtenido con n-gramas de primer orden es muy aceptable y cabe esperar mejoras notables con modificaciones sencillas tales como aumentar la extensión del texto de entrenamiento hasta unos ≈5-10 MB y mejorar el proceso de filtrado.

Para el segundo ejemplo, utilizamos una colección de textos de Pablo Mogica Ortuño con una extensión todavía menor: 0.12 MB. En este caso el texto está escrito en castellano, lo cual presenta una dificultad adicional dado que los verbos se conjugan de una forma más complicada que en lengua inglesa y los adjetivos no son invariantes.

**Ejemplo 2**

|Morir a la mitad del camino de la casa en la rutina. Las tres líneas que seguían este momento hasta la mente del desconocido. Que la casa de su belleza, como no podía ser, tomara decisiones basadas en el mundo. Tampoco hay que hacer tiempo antes de perder el tiempo. Tienes que hacer tiempo porque cualquier texto es posible contaminarlo con un agujero en una de las vistas. ¿Es que acaso no piensas que empezar la carrera, se interesaba por todo? Sabía usar Photoshop, sabía tocar la viola, sabía cosas de este ser impronunciable. El lag de la habitación 201, me dijo: hice lo que podía escribir, siempre con límites en estas cosas como un boomer. Dame un momento. Casi se vio cómo se esparcía su posición en la rutina. Las tres líneas que seguían este párrafo han sido siempre el enemigo de las vistas. ¿Es que acaso no piensas que yo simplemente lo usaba para guardar la bandeja ésta con raíles que viene debajo de algunas mesas? Algo que recuerdo que en estas cosas no llegaste a pensar nunca en quién va a sus hijos. La distancia que los trabajadores pudieran acceder al barrio rico no llegaste a pensar nunca. El esquema de fondo aquí está claro. Una persona se levanta. Las tres líneas que seguían este párrafo han sido siempre el enemigo de mi casa. Gasolinera que viene hacia mí, y eso significa que eres bueno y nadie debe salir ahora. No haces incursiones porque ha hecho frío fuera, echabas de menos a la gasolinera. Llegamos a la distancia que separa a casa, nos quedamos solos en Madrid y aquí es donde resalta su propia validez, en la lluvia.|

De nuevo, un análisis sencillo del resultado muestra que el algoritmo no escribe aleatoriamente, sino que trata de escoger palabras, conjuntos de palabras y pequeñas frases que considera relevantes —generando un resultado extrañamente onírico en este segundo ejemplo—, intentando colocarlas de nuevo siempre que le sea posible; o, dicho de otra manera, presta atención a lo que está escribiendo para mantener los mismos temas. Una vez más, dada la escasa extensión del texto de entrada y a la dificultad adicional de la gramática del castellano, los resultados no son tan precisos como los que podríamos obtener con el mismo algoritmo usando un mayor volumen de texto de entrenamiento. En frases largas se corre el riesgo de que el texto comience a perder sentido; que los tiempos verbales no sean consistentes ni tampoco el género y el número de los sustantivos y adjetivos. Por este motivo, los textos generados en inglés —donde la conjugación de los verbos es sencilla y los adjetivos son invariantes— se obtienen resultados muy superiores a los textos en castellano. No obstante, los resultados preliminares obtenidos con la versión actual del modelo indican que es posible comenzar a producir textos de una legibilidad aceptable incluso con volúmenes de entrada muy reducidos.

Los ejemplos mostrados hasta ahora han sido construidos con MK formados por n-gramas de primer orden. Para usar n-gramas de segundo orden se necesitan textos de mayor extensión con el objetivo de evitar que los resultados se parezcan demasiado al texto de entrada. Utilizando una versión de la King James Bible, con una extensión de 3.9 MB, obtenemos resultados a segundo orden similares al siguiente:

**Ejemplo 3**

|Spakest unto thy servant David my father that which thou spreadest forth to be baptized of him, O generation of vipers, how can ye escape the damnation of hell? Wherefore, behold, I send you forth as lambs among wolves. Carry neither purse, nor scrip, nor shoes: and salute no man by lying with him, keep alive for yourselves. And do ye abide without the camp seven days: and they kept other seven days with gladness. For Hezekiah king of Judah: so that he gave him, all people, nations, and languages, trembled and feared before him: whom he would he kept alive; and whom he shall anoint, and whom he justified, them he also glorified. What shall we do unto the queen Vashti refused to come at this time; but he will call to remembrance my song in the house of Israel. So I came to them Tatnai, governor on this side Jordan may be our’s. And they took him, and commanded him to be my priest, to offer upon mine altar, to burn incense, to wear an ephod before me? and did I give unto them for whom it is given. For there are three that bear witness of me, that will I do unto thee? O Judah, what shall I do then with Jesus which is called Bethlehem; (because he was of the prophets). He saith unto him, How can a man then understand his own way? It is a good land and a large, unto a land not inhabited. Thus saith the great king, the king of Judah, unto the carrying away into Babylon are fourteen generations; and from David until the carrying away into Babylon unto Christ are fourteen generations. Now the birth of Jesus Christ from the rudiments of the world, how he may please him.|

En este caso, habiendo aumentado la extensión del texto de entrenamiento y el orden del n-grama, los resultados obtenidos no muestran imprecisiones gramaticales graves y dan lugar a un texto muy legible —salvando las distancias con lo arcaico del inglés—. El resultado mantiene el tono del original y, una vez más, intenta volver recurrentemente a palabras relevantes que han aparecido durante la escritura para tratar de mantener una coherencia global.

### VI. Conclusiones

A lo largo de este artículo hemos examinado el panorama general de las técnicas de generación dinámica de textos mediante algoritmos, centrándonos en modelos basados en procesos de Markov. Basándonos en el paradigma más extendido de generación de textos mediante procesos sin memoria, proponemos un modelo que amplía las posibilidades de los anteriores, alejándonos de aquellos cuya Choice Function es meramente probabilística para utilizar métodos más sofisticados que prestan atención al texto que está escribiendo, obteniendo resultados que presentan una coherencia muy superior a la de modelos más sencillos.

En líneas generales, nuestro modelo permite modificar las condiciones de escritura en tiempo de ejecución —después de haber construido el mapa de correspondencias, y sin necesidad de volver a generarlo—, de tal forma que se puede privilegiar el uso de determinadas palabras, así como tratar de suprimir en la medida de lo posible una lista de palabras censuradas, pudiendo alterar ambos conjuntos en cualquier momento. Por otra parte, dado que nuestro modelo permite conocer el flujo completo List<Predecesoras> -> Key -> List<Sucesoras> asociado a cada n-grama clave, podemos realizar inserciones en tiempo de ejecución de nuevas correspondencias (PK,MK) -> List<SV>, haciendo posible la ampliación del mapa de correspondencias utilizado para generar texto en tiempo real. Es decir, podemos añadir nuevos fragmentos al texto de entrada, conservando el mapa inicial, y generar las nuevas correspondencias asociadas al texto ampliado manteniendo las anteriores.

Respecto a las limitaciones de nuestra propuesta, los algoritmos basados en procesos de Markov presentan inconvenientes inherentes a la propia naturaleza sin memoria de los mismos. En frases largas existe el riesgo de que el texto comience a perder sentido y que los tiempos verbales y las conjugaciones sean imprecisos. Para mitigar en la medida de lo posible esta situación, el algoritmo presta atención a los pronombres utilizados, de modo que se intenta forzar la utilización de los mismos pronombres a lo largo del texto, tratando de mantener la consistencia en género y número en las frases.

A pesar de ser un modelo relativamente sencillo —una persona con conocimientos generales de programación orientada a objetos y estructuras de datos debería ser capaz de poder implementarlo sin problemas—, se han obtenido unos resultados preliminares más que satisfactorios, incluso en condiciones de entrenamiento con textos de extensión reducida —0.12-0.49 MB—. En el momento de la escritura de este artículo (junio 2020), el algoritmo se ha ido actualizando y actualmente la Choice Function utiliza —además de lo mencionado en la Sección III— un sistema de puntos con el que evalúa las palabras clave a las que está prestando atención durante la generación del texto, así como la implementación de un sistema de atención a segundo orden, obteniendo resultados todavía mejores que se discutirán en sucesivos artículos.

### VII. Referencias

1. Attention Is All You Need. Ashish Vaswani, Noam Shazeer, Niki Parmar, Jakob Uszkoreit, Llion Jones, Aidan N. Gomez, Lukasz Kaiser, Illia Polosukhin  https://arxiv.org/pdf/1706.03762.pdf
2. Language Models are Unsupervised Multitask Learners. Alec Radford, Jeffrey Wu, Rewon Child,  David Luan,  Dario Amodei, Ilya Sutskever
3. Language Models are Few-Shot Learners. Tom B. Brown, Benjamin Mann, Nick Ryder, Melanie Subbiah, Jared Kaplan, Prafulla Dhariwal, Arvind Neelakantan, Pranav Shyam, Girish Sastry, Amanda Askell, Sandhini Agarwal, Ariel Herbert-Voss, Gretchen Krueger, Tom Henighan, Rewon Child, Aditya Ramesh, Daniel M. Ziegler, Jeffrey Wu, Clemens Winter, Christopher Hesse, Mark Chen, Eric Sigler, Mateusz Litwin, Scott Gray, Benjamin Chess, Jack Clark, Christopher Berner, Sam McCandlish, Alec Radford, Ilya Sutskever, Dario Amodei. https://arxiv.org/abs/2005.14165
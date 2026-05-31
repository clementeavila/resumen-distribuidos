# Primer final (whatsapp 1)

## Preguntas teóricas 

**1- ¿En que consiste el modelo Work-Span para análisis de paralelización de tareas y camino crítico?**

El modelo work-span nos brinda una cota superior e inferior para el Speedup, teniendo en cuenta la cantidad de procesos, el tiempo de ejecución secuencial de todas las tareas y el tiempo de ejecución del camino critico si tuvieramos infinitos procesadores.

Sea P la cantidad de procesadores, Work (T1) el tiempo de ejecución secuencial de todas las tareas y Span (Tinf) el tiempo de ejecución del camino critico si tuvieramos infinitos procesadores, las cotas se pueden calcular de la siguiente manera

Cota superior= min(P, T1/Tinf)
Cota inferior = (T1 - Tinf)/P + Tinf

Este modelo nos brida una mejor estimación que Amdhal ya que no asume paralelización perfecta (se ve en la cota inferior)

**2- ¿Que se entiende por atomicidad de mensajes en el contexto de grupos de comunicación? ¿Con que estrategias es posible garantizarla?**

En la comunicación de grupos, el concepto de atomicidad de mensajes se entiende a que si un proceso envía un mensaje a varios procesos del grupo, este mensaje lo reciben todos los procesos o no lo recibe ninguno.

Una forma de garantizar esto es haciendo que el envío de mensaje sea centralizado, es decir, que un nodo se encargue de enviar el mensaje a todos los demás y esperar a recibir los ACK de todos, en el caso de que alguno no responda dentro de un cierto timeout, entonces el mensaje no le llego a todos y se manda un ABORT a los procesos.

Para poder realizar esot, tambien se puede utilizar una holdback queue, que no entrega el mensaje hasta que se le avise.

Tambien hay que tener en cuenta realizar reintentos ante la perdida de mensajes o si se cae el que envio el mensaje o alguno que lo recibe.

**3- Defina 2 protocolos distintos para la transmision de paquetes considerando un registro que posee un int y un arreglo de floats? Compare ventajas y desventajas de cada protocolo**

Como el registro posee un int (largo fijo) y un arreglo de float (largo variable), un protocolo, el cual usa formato binario, podría ser de la siguiente forma: 

| int (4bytes) | largo array  (4bytes) | floats | 

En donde, se usarían 4 bytes para el largo del array ya que permite que el array sea de hasta 2^32 de largo

Otro protocolo puede ser usando texto plano (formato JSON). En este caso, el paquete sería de la siguiente forma:

	{
	   int: <int>
	   array: [1.0, 2.0, ...]
	}

La ventaja del primer protocolo es que tiene un tamaño de mensaje mucho más eficiente que el que usa JSON, pero tiene la desvenataja de que para poder leerlo facilmente, se debe serializar el paquete, a diferecia del que usa JSON el cual permite debuggearlo y leerlo más facilmente.

Otra ventaja del primero es que al no agregar un Overhead por usar formato binario, es más rápido para enviarse.

**4- Explique HDFS (Hadoop Distributed File System) y desarrolle los conceptos claves utilizados como factores de diseño.**

HDFS es un File system distribuido pensado para realizar operaciones sobre grandes cantidades de datos y usar hardware debajo costo. La idea detrás de este es que hay 2 tipos de nodos:

- Namenode: se encarga de guardar la metadata de los archivos y sabe en que datanode estan ubicados
- Datanode: se encarga de almacenar los archivos en bloques de tamaño fijo

Si un cliente quiere acceder a un archivo, este tiene que hablar con el namenode más cercano, el cual le va a decir que que datanode se encuentra para que pueda acceder al archivo. Además, en el caso de que se quieran hacer consultas sobre la metadata, esta es directamente respondida por el namenode ya que guarda la metadata.

Otra cosa que hace Hadoop es replicar la información de los archivos en distintos datanodes para tenerla disponible en caso de que falle alguno y balancear la carga.

Los factores de diseño se Hadoop son:
- Favorecer las lecturas sobre las escrituras, esto implica que es mucho más costoso crear o modificar un archivo que leerlo.
- Portabilidad a cualquier tipo de hardware
- Busca adaptarse a los fallos ya que los fallos son normales y es más económico adaptarse 
- Favorece operaciones sobre streaming o grandes conjuntos de datos


**5- Explique un algoritmo de exclusión mutua distribuida que no utilice servidor central** 

Un algoritmo de exclusión mutua distribuida que no utiliza un servidor central es Token Ring. Para usar este algoritmo se necesita una topología de anillo, donde cada nodo solo se comunica con su vecino, establecer un orden para el envío de mensajes y un Token.

La idea es que los nodos se vayan pasando el token entre ellos y solamente el nodo que tenga el token puede acceder a la sección critica, lo que garantiza que solo 1 va a estar en la sección critica en todo momento ya que solamente hay 1 token. En el caso de que un nodo no necesite acceder a la sección crítica, lo pasa al nodo vecino.

Este algoritmo si bien es simple y no tiene un servidor central, hay que tener cuidado con 2 cosas:
- En el caso de que se caiga un proceso, como se va a rearmar el anillo
- Que hacer en el caso de que se pierda el token.

------------------- Otra respuesta -------------------

Otro algoritmo es el de Ricart & Argawala, en el cual se requiere que los nodos tenga un ID único y utiliza timestamps. Este algoritmo usa una topología de todos contra todos, en donde si un nodo quiere acceder a una sección critica, le envia a todos los demás el mensaje indicando eso junto con su ID y un timestamp.

El proceso que recibe el mensaje puede hacer los siguiente:
- Si no quiere acceder a la sección critica, responde OK
- Si ya esta en la sección critica, encola el mensaje para responderlo cuando salga de la sección critica 
- Si quiere entrar en la sección critica, compara el timestamp. Si el timestamp del mensaje recibido es menor al del proceso, entonces responde OK. Caso contrario, encola el mensaje. 

Las desventajas de este algoritmo es que requiere que haya conexiones con todos los nodos y tiene un chatiness alto.


**6- Explique como se relacionan los conceptos de fallo, error y avería. Indique luego la importancia de esta relacion para el estudio de sistemas tolerantes a fallos**

Un Fallo es una condicion del sitema que no se cumplió y el cual puede llegar a generar un Error. Por ejemplo un bit defectuoso en la memoria. 

El Error ocurre porque el estado del sistema pasa a ser incorrecto y puede llegar a derivar en una avería.

La avería sucede cuando hay un comportamiento erróneo del sistema debido a un error

La importancia de esto es que en la tolerancia a fallos se busca procesar y tratar los fallos para evitar llegar a una avería del sistema. Esto implica de intentar solucionar estos fallos antes de que lleguen a afectar el estado del sistema través de distintan tecnicas como replicación, consenso, etc.

**7- Explique un algoritmo de consenso para que 4 procesos lleguen a un acuerdo sobre cierta variable que ellos definen. ¿Cómo actúa el algoritmo al caerse 1 proceso? ¿ Y al caerse 2 procesos?**

Un algoritmo para resolver esto consiste en tener un arreglo de valores, que inicialmente solo contiene el valor que propone el proceso,  un numero F de rondas y una función de agregación deterministica para poder determinar el resultado.

El algoritmo en cada ronda hace lo siguietne:
- El proceso broadcastea su arreglo de valores y asigna que el arreglo de valores para la ronda siguiente va a ser al actual.
- Luego se queda esperando (mientras este abierta la ronda) a los demás procesos envíen sus valores y los suma a los de la ronda siguiente (Hace una union entre los arreglos)

Una vez se realiza la ultima ronda, se aplica la función de agregación al arreglo de la ultima ronda para obtener el valor acordado.

En el caso de que 1 proceso se caiga, no va a haber problema porque sigue habiendo más de la mitad de procesos para acordar el valor. Pero en el caso de que se caigan 2 procesos, el algoritmo no se debería ejecutar ya que no hay una mayoría para decidir el valor de la variable.

## Preguntas practicas 

**8- Detalle un algoritmo de consenso que permita que 3 procesos concuerden el valor de la altitud de un dron en pleno vuelo. Muestre el paso a paso de como funciona el algoritmo considerando que los procesos A,B miden 10m y el proceso C mide 9m** 

--- Esto con paxos es medio quilombo, ver como hacer ---- (No se si esta bien usar este algoritmo aca)

En este caso voy a usar un algoritmo de consenso que utiliza un nodo central parar aplicar una función de agregación, por ejemplo un promedio entre los valores que marcan los procesos. El algoritmo consiste en que los procesos envía los valores que obtuvieron al nodo central, en donde, una vez todos los procesos envíen sus datos va a ejecutar una función para calcular el promedio entre los valores y le va responder con el resultado a cada proceso.  En el caso de que se caigan 2 de los procesos, este algoritmo no se va a poder ejecutar ya que no va a haber una mayoría como para llegar al consenso (se requiere un mínimo de 3)

Paso 1: Los procesos le envían las valores al nodo central. El nodo central espera a que estén la mayoría de procesos antes de continuar

Paso 2: El nodo central ejecuta el algoritmo y se queda con el mayor valor

Paso 3: El nodo central responde a todos los procesos con el valor promedio


**9- Dada una libreria basada en request-reply, realice un pequeño sistema que realice la tarea de calcular el hash de un archivo binario leyendo sus bloques de 4 bytes y aplicando sucesivamente la operación XOR a medida que se leen más bloques. Distribuya la carga entre N servidores conocidos. Utilice pseudocódigo o diagrama UML (Secuencia, colaboracion, actividades) para explicar la solucion e indique supuestos de ser necesario** 

==(No estoy seguro como hacer para tener en cuenta los N servidores para distribuir la carga. Si es uno solo, es mas facil ya que es envio y espero la respuesta.==
==Si pudiera asumir que el archivo entra en memoria, puedo hacer un fork-join sin complicarme la vida)==

(Se puede mezclar UML con algo de psedocódigo para esto?)

Supuestos utilizados:
- Comunicación reliable entre el cliente y el servidor.
- El cliente cuenta con suficiente memoria para guardar el archivo completo en memoria

La idea del sistema es la siguiente:
- El cliente divide el archivo en chunks segun la cantidad de procesos
- El cliente hace un fork de N procesos y le asigna los chunks correspondientes
- Cada sub procesos envia el chunk al servidor
- El servidor aplica la función XOR al chunk recibido y envia el resultado parcial al cliente 
- El cliente recibe los resultados parciales de los servidores y le va aplicando la funcion XOR. Esto se hace para no ir guardando todos los resultados parciales (solo me guardo el acumulado)
- Una vez el cliente recibió todos los datos, se joinean los procesos y se aplica la función XOR entre los datos de los servidores y se obtiene el resultado final 

!(img)[images/Diagramas-practica-examen-ej9-FINAL1.png]

(Falta el diagrama del servidor, pero es muy simple)

(Este ejercicio puede mejorarse si no se carga todo el archivo en memoria y se tiene un proceso que va leyendo el archivo y mandando por un canal los chunks haciendo round robin)

**10- Diseñe una arquitectura que asegure la escalabilidad para un sistema con los siguiente requisitos:** 
- **Permitir la asignacion de Aires Acondicionados inteligentes en LATAM para la captura de datos: fecha de ultima medición, minutos en uso desde ultima medición, temperatura medida, ubicación aproximada (latencia, longitud)**
- **Asegurar la captura de datos con una frecuencia de 10 minutos**
- **Permitir la consulta de un reporte con ciudades conocidas y temperatura min, max, avg medidas en las ultimas 24 horas**
- **Permitir la generación de una alarma y reporte por email al dueño del dispositivo si se detecta uso excesivo: mas de 12 horas de uso durante las ultima 24 horas.** 
**Resuelva el problema mediante analisis de volumen, endpoints y vista física con su explicación**

De los requerimientos obtengo que:

- Tengo un paquete que contiene fecha de ultima medición, minutos en uso desde ultima medición, temperatura medida, latencia, longitud. Si bien no conozco el tamaño especifico en bytes para la fecha, latencia y longitud, puedo estimar que el paquete no va a ser mayor a 1KB. Tambien necesito un endpoint para poder cargar estos datos
- Cada aire tiene un UUID único asociado para poder identificar quien lo compró
- Como debo permitir realizar consultas sobre las temperaturas medidas en las últimas 24 horas, debo tener un endpoint y guardar los datos de las últimas 24 horas. Con el fin de no tener que actualizar cada vez que llega una consulta, se actualizan los datos cada 1 hora
- Como debo enviar cada 10 minutos, en 1 hora envío 6 veces y en un día envío 6x24 =144 por cada aire acondicionado 
- Por donde suena la alarma? una app o el aire acondicionado? -> Duda. Asumo que hay un servicio para eso y se avisa por una notificacion.
- Necesito poder usar un servicio de mailing para enviar el reporte al haber uso excesivo. Para conocer el mail del usuario voy a asumir que puedo obtenerlo de algún servicio externo a este sistema.

Análisis de Endpoints:

POST /medicion -> aca es donde se van a cargar los datos de los aires acondicionados. 
Fecha de ultima medición, minutos en uso desde ultima medición, temperatura medida, latencia, longitud.

GET /consulta ->  aca es donde se hacen las consultas sobre las temperaturas de las últimas 24 horas.

Análisis de volumen de datos:

Para calcular el tamaño del paquete tengo:
- Tiempo de la medición: va a ser un timestamp por lo que no creo que ocupe más de 20 Bytes 
- Minutos en uso desde la ultima medición: con 1 byte alcanza porque cada medición se hace cada 10 minutos por lo que el numero no puede ser mayor a 10
- Temperatura medida: con 1 byte alcanza ya que la temperatura más baja a al que llega el aire es de alrededor de 16 grados y la máxima no llega a mucho más de 30.
- Para la ubicación, teniendo en cuenta latitud y longitud, no se supera más de los 18 bytes
- UUID del aire para poder identificarlo, lo cual es 16 Bytes

Entonces, el tamaño del paquete va a ser de alrededor de 60 bytes.

Viendo que los aires se venden por todo LATAM, debo estimar cuantos aires/usuarios hay.  Asumiendo que en LATAM hay 500 millones de personas, tambien que hay 1 aire acondicionado por casa (no es si o si el inteligente) y que en cada casa vive en promedio una familia de 4, entonces voy a tener:

500 mill / 4 = 125 mill aires 

Y de todos esos asumo que un 5% como mucho va a tener un aire inteligente, entonces:

125 mill * 0.05 = 6.25 mill

Por lo tanto asumo que hay 6.25 millones de aires inteligentes

Como cada aire envía cada 10 min el paquete, en 1 hora cada aire envía 6 paquetes y en 24 horas cada uno envía:

6 * 24 = 144 

Por lo que cada aire envía 144 paquetes por día 

Entonces el volumen de datos por día es de:

6.000.000 * 144 bytes * 60 = 6.000.000 * (240 + 2400 + 6000) = 6.000.000 * 9000 =~ 60.000.00 bytes por día

Por lo tanto por día se recibe un total de 6.000.000.000 de bytes = 60 GB 

Esos datos los voy a tener que almacenar y cuando pasan 24 horas, debo borrar los datos viejos ya que no me sirven más.

Vista física: 

Debido a que el servicio se ofrece en todo LATAM, para reducir la latencia, debo tener servidores en los distintos países, los cuales se comunican con un servidor central para enviar solo los datos procesados para las ciudades más conocidas de los respectivos países (otra opcion es enviarle todos los datos, pero no hace falta).

Por un lado, tengo que enviar la temperatura registrada,  ubicación y el timestamp a un nodo que se va a encargar de obtener la ciudad a partir de la ubicación del aire. Una vez obtiene la ciudad, este va a enviar tanto la ciudad como la temperatura y el timestamp a un filtro para ver si es una ciudad conocida o no. Y los que pasan ese filtro se los guarda en una DB particionada según el nombre de la ciudad.

Para poder calcular el max, min y avg de temperaturas de las ultimas 24 horas, va a haber un nodo que se va a despertar cada 1 hora para leer los datos de la base y calcular esos datos.


Para poder calcular la cantidad de tiempo de uso en las últimas 24 horas, envio el timestamp junto con el tiempo registrado en la ultima medición y el ID del aire a un nodo y  lo guardo en una DB

Voy a tener N nodos que van a leer de esa DB (cada una va a leer una particion como minimo, por lo que no debería haber más particiones que nodos) cada 10 minutos (puede cambiarse, pero lo hago asi porque recibo registros cada 10 min) y va a calcular el tiempo acumulado de uso en las ultimas 24 horas. En el caso de que alguno supere las 12 horas, se va a usar un servicio de mailing para enviar un mail avisando esto y se va a activar la alarma

Voy a necesitar tambien un proceso que se encargue de borrar los registros de todas las  DB que ya tengan más de 24 horas para no tener datos guardados que no voy a usar


# Segundo Final (whatsapp 2)

## Preguntas teóricas

**1- Suponga que debe implementar un protocolo de health-check de server a clientes mediante sockets ¿Que condiciones se deberían cumplir para que sea más conveniente utilizar UDP frente a TCP? Y cuando es mas conveniente usar TCP que UDP?**

Usaría UDP si:
- No necesito reliability, es decir, no me importa si el paquete que envio se perdio o no o si no recibo una respuesta inmediatamente ya que podría enviar paquetes cada cierto tiempo y tener un timeout y polita de reintentos para ver desconexiones
- Necesito que el envio de mensajes sea rápido y más eficiente ya que UDP agrega un bajo Overhead de memoria
- Si no hay necesidad de mantener múltiples conexiones abiertas que estén idle la mayoría del tiempo
- Si hay muchos clientes, UDP es mas eficiente ya que hace que el servidor utilice menos recursos 

Usaría TCP si:
- Necesito reliabilty, es decir, es importante garantizarme que el paquete y la respuesta lleguen si o si.
- Quiero poder detectar si un proceso esta caido de forma más rápida. Esto pasa porque se cae la conexión y generalmente levanta un error
- El numero de clientes no es muy grande ya que TCP consume más recursos que utilizar UDP

**2- Explique la utilidad de los algoritmos de relojes lógicos de Lamport y de vectores de relojes explicados en clase. ¿Para que sirve cada uno?**

Ambos algoritmos sirven para poder ordenar los distinitos eventos del sistema y ver si estos eventos son concurrentes o si uno ocurrio antes que otro.

En el caso del algoritmo de Lamport, este permite detectar eventos concurrentes (solo si los relojes tienen el mismo valor) y nos garantiza que si el evento S sucedió antes que el evento T, entonces el reloj logico de S va a ser menor al de T. El problema que tiene este algoritmo es que no puede garantizar la inversa ya que le hace falta más información.
Este algoritmo se usa como base para el de vectores.

El algoritmo de vectores sirve para establecer relaciones de causalidad entre estados. Permite garantizar lo mismo que el algoritmo de Lamport asi como tambien nos garantiza que si C(S) < C(T), entonces S->T cosa que lamport no podía garantizar. Esto no solo permite detectar eventos que sucedieron antes que otros sino que tambien nos permite detectar si los eventos son concurrentes (si los vectores son iguales o intercalan en vez de ser uno mayor que otro)


**3- Describa como implementaría un middleware que garantice la comunicación de grupo con las siguiente condiciones:**
- **Solo existen 3 grupos**
- **Los miembros del grupo se pueden unir en cualquier momento para recibir mensajes**  
- **Solo se permite enviar mensajes a todos los miembros de tu grupo**

(Ni idea si esto esta bien)

Para poder garantizar eso, se tienen las siguientes primitivas:
- Un proceso puede unirse a un grupo mediante el metodo *subscribe_group(group_id)*. El middleware se encargaría de determinar cuales son los miembros del grupo. Tambien valida de que el grupo exista y si ya esta en un grupo, lo desuscribe
- Un proceso puede salirse de un grupo mediante el metodo *unsuscribe_group(group_id)*. Esto puede fallar si  el proceso no esta en ningún grupo
- *broadcast(msg)* para poder enviar un mensaje a todos los miembros del grupo. Para poder usarlo, el proceso debe estar en un grupo. Esto revisa en que grupo se encuentra el proceso y le envía el mensaje a todos ellos 

Como el middleware mantiene una lista de procesos en cada grupo, debe tener un mutex por si esta enviando un mensaje y justo alguien quiere salirse/unirse al grupo.


(Duda a que se refiere con garantizar y que tan en profundidad. Tema de fallo de un proceso, etc)

En el caso de que falle un proceso, el middleware se encarga de enviar heartbeats para detectar la caida y sacarlo del grupo en ese caso.

Middleware mantiene cuales son los grupos y los miembros de los grupos en un diccionario en memoria (asumo que no van a ser muchisimos procesos en los grupos)

**4- Explique el beneficio de la replicación de datos y desarrolle una estrategia de implementación** 

La replicación de datos brinda muchos beneficios como por ejemplo:
- Poder tener la informacion en varios lugares distintos lo que permite reducir latencias y hacer un balanceo de carga a la hora que se quieran consultar los datos 
- Permite que se puedan recuperar los datos en caso de que haya una falla de hardware

--- Son las estrategias de unico lider, multi-leader, leaderless ---

Una de las estrategias implica tener 1 nodo lider y varias réplicas, donde solo el lider puede recibir escrituras pero todos los nodos pueden recibir lecturas. El nodo lider cuando recibe una escritura lo que hace es guardarla y enviarla a todas las réplicas para que hagan los mismo.

El problema que tiene esta estrategia es que al realizar una consulta en una de las réplicas, puede que los datos no estén actualizados

--- Otras estrategias --- 

Otra estrategia es la de Multi-lider. En este caso no hay un unico lider, sino que hay más de 1. Ambos lideres pueden recibir tanto lecturas como escrituras y las replicas solo pueden recibir lecturas. Cuando alguno de los lideres recibe una escritura, este se lo debe comunicar tanto a sus réplicas como al otro lider para que actualizen la información.

El problema de esta estrategia, además de que se puede llegar a responder con datos desactualizados, es que ambos lideres pueden estar modificando un mismo registro, por lo que todos esos conflictos deben poder resolverse.

Por último una estrategia es la de leaderless, en este caso no hay lideres, sino que todos son replicas que pueden recibir lecturas y escrituras. Esto implica que cuando uno recibe una escritura se lo debe enviar a los demás nodos, lo cual puede generar conflictos cuando 2 nodos queiren modificar el mismo registro.


**5- Explique el algoritmo de Distributed Shared Memory (DSM) donde se utilice la replicación de páginas. Indique un caso de uso donde el algoritmo elegido sea ventajoso y otro donde sea perjucicial**

DSM consiste en que varios procesos que se encuentran distribuidos compartan la memoria como si estuvieran en una única computadora. Uno de los algoritmos que utiliza la replicación de páginas es el siguiente:

Replicación de páginas (solo lectura)

En este caso se tiene un servidor que es donde van a estar guardado los recursos que se quieran compartir. Un cliente que quiere adquirir un recurso lo puede hacer para leer o para escribirlo. En el caso que un cliente quiera escribir, se la da el recurso y en caso de que quiera leer, se le da una replica de la página.  Solo un proceso puede pedir la pagina para escribir al mismo tiempo, por lo que si un proceso quiere una página que esta tomada, debe esperar.

Si el proceso que pidio la pagina para escribir la modifica y la envía al servidor para guardarla, el servidor anula todas las replicas de lecturas de esa página ya que no serían válidas.

Este  algoritmo puede ser ventajoso en situaciones donde se requieran hacer muchas lecturas y muy pocas esccrituras y sería perjudicial en el caso contrario

(Buscar un ejemplo específico para esto.)

Replicacion de páginas (lectura-escritura) ??? -> repasar esto

Es muy similar, solo que tambien se permite replicar páginas para modificarlas. En este caso el servidor funciona como un secuenciador de operaciones y cuando uno modifica una página, debe enviar esos cambios a los demás clientes.


**6- ¿ En que consiste la propiedad de Elasticidad de un sistema distribuido? ¿ Que elementos mínimos son necesarias para garantizarlas?**

La propiedad de elasticidad es la capacidad que tiene le sistema en poder adaptarse de forma dinámica a los cambios en los patrones de carga del sistema (start slow-grow fast, predictable burst, unpredictable burst, procesamiento periodico).

Para poder garantizar esta propiedad se necesita:
- Monitoreo automatico que nos brinda datos sobre la carga del sistema 
- Autoscaler para poder aumentar/reducir la cantidad de instancias del sistema de acuerdo a las metricas
- Un load balancer para poder asignar trafico a las instancias levantadas y hacer que no le llegue trafico a las instancias caidas.


**7- El estudio de sistemas confiables (dependable) implica la revision de varias propiedades que impactan en la tolerancia a fallos del sistema. Detalle los conceptos claves de al menos alguna de ellas**

Las propiedades que impactan la tolerancia a fallos son:
- Disponibilidad
- Confiabilidad
- Seguridad
- Mantenibilidad
- Durabilidad

En el caso de disponibilidad es la probabilidad que tiene el sistema de responder consultas. Esto no implica que las responda correctamente, sino que implica que el sistema debe seguir ofreciendo servicio, aunque se reducido. La disponibilidad se mide en base a los 9's de acuerdo a la probabilidad de que este este disponible o no.

En el caso de confiabilidad implica la capacidad del sistema de responder de forma correcta las consultas, es decir, que si yo recibo una consulta no respondo cualquier cosa.

Debido al tema de replicación, siempre hay un trade-off entre disponibilidad y confiabilidad ya que tener más replicas me permite aumentar la disponibilidad del sistema ante la caida de nodos, pero la confiabilidad baja ya que puede que las respuestas no esten actualizadas.

## Preguntas practicas

**8- Utilizando blocking-queues, implemente una barrera para que 3 procesos llamados workers puedan sincronizarse en cierto punto del algoritmo que ejecutan. Detalle supuestos de ser necesario. Utilice pseudocodigo o diagrama UML.**

Toma los siguientes supuesto:
- Hay un cuarto proceso que se encarga de mantener la barrera.

La idea es tener 2 blocking queues por proceso, una en donde el proceso avisa que llegó a la barrera y otra para que el proceso quede bloqueado hasta que los 3 procesos lleguen a la barrera.

	_____________                       ____________
	|           |   ----> |cola| ---->  |          |
	|  Proceso  |                       |  Barrera |
	|___________|  <---- |cola| <----   |__________|

Cada proceso haría algo asi: 

```
 -- Algoritmo ejecutando--- 
		   .....
	queue.push(msg)
	queue.pop() #quedo bloqueado aca hasta que el proceso que mantiene la barrera mande un mensaje por la cola 

	--- Sigue algoritmo ---
```


En cambio el proceso de la barrera hace lo siguiente: 

```
N = 3 # cantidad maxima de procesos que se bloquean 
i = 0 # cantidad de procesos bloquedos
receiving_queues = [recv_queue1, recv_queue2, recv_queue3]
sending_queues = [send_queue1, send_queue2, send_queue3]

while True:
	for queue in receiving_queues:
		if not queue.empty():
			queue.pop()
			i += 1
			if i == N:
				envío mensaje por las sending_queues a cada proceso 
				i = 0 	

```



**9- Defina con pseudocodigo los pasos para consultar el estado de una subscripción de Netflix con numero NF-1234, obtener todos los serial numbers de dispositivios permitidos en la subscripción y darlos de alta en otra subscripción número NF-12345 asumiendo la existencia de una plataforma de Distributed Objects. Realice un grafico que ejemplifique dicha arquitectura y el estado de los objetos** 

Asumo que netflix guarda las subscripciones en objetos que contienen lo siguiente:

Subscripcion {
	ID: Int
	dispositivos_permitidos: Array
	...
}
tiene los metodos:

get_subscription(sub_ID) -> funciona como si fuese el INIT y se encarga de buscar que el objeto exista
get_devices() -> return dispositivos_permitidos
add_device(serial_number) -> dispositivos_permitidos.append(serial_number) && actualizar_objeto()



Y los objetos tienen metodos para agregar dispositivos a la lista de Dispositivos_permitidos

Entonces los pasos a seguir serian:

```
#Incio el middleware 

netflix = Middleware()

# Obtengo referencias a ambas subscripciones
old_sub = netflix.get_subscription(NF-1234) 
new_sub = netflix.get_subscription(NF-12345)

# Obtengo los dispositivos permitidos de la primera sub
dispositivos_permitidos = old_sub.get_devices()

# Agrego los dispositivos de la subscripcion vieja en la nueva
for serial_number in dispositivos_permitidos:
	new_sub.add_device(serial_number)


```

??? Ver como se usa el tema de Distributed Objects porque nunca vimos ningun ejemplo creo. Y con grafico de arquitectura y estado de los objetos a que se refiere? Arquitectura de Distrib Object y el estado

Hace falta el pseudo codigo del lado del servidor?

(Todo esto esta en la clase del video RMI. Asumir que hay discovery de objetos y mostrarlo en diagrama y que tambien hay un middleware centralizado que guarda los objetos y tiene un dispatcher que se encarga de ver donde estan los objetos y modificarlos). En el diagrama tambien mostrar que esta la librería del cliente para consultar.

(Del objeto debo meter las variables que guarda y los metodos que necesito. No se si hace falta tambien mostrar como queda cada clase)

**10- Diseñe una arquitectura que asegure la escalabilidad para un sistema con los siguintes requisitos:**
- **Permitir el reporte ciudadano de infracciones de transito presenciadas en la via publica en CABA**
- **El reporte consta de una foto e indica tipo de infraccion presenciada ("vehiculo mal estacionado", "cruce en rojo", "choque con fuga") y una descrición de lo ocurrido
- **Reconocer el vehiculo del reporte y los datos de su titular, dejando registro de los mismos** 
- **Analizar automáticamente la fota para el tipo "cruce en rojo" y emitir una multa por email al titular en caso de validar la infracción** 
- **Permitir la consulta para la evaluación manual de los tipos restantes y para auditoría general por 10 años** 
- **Notificar por email al ciudadano sobre el estado de su reporte (hubo/no hubo infraccion) a los 7 días.**
**Resuelva el problema mediante analisis de volumen, endpoint  y vista fisica con su explicación**

Viendo los requisitos, obtengo la siguiente información:
- Solo en CABA, por lo que puedo tener servidor central 
- Endpoint para reportar infracciones. Este recibe una foto, tipo de infraccion y descripción de lo ocurrido.
- Reconocer el vehiculo del reporte y obtener los datos del titular. Estos datos debo guardarlos. Los datos del titular lo obtengo de un servicio externo 
- Analisar si tipo multa es "cruce en rojo", validar infraccion y si lo es emito multa por email 
- Endpoint para consultar tipos de multas restantes ("vehiculo mal estacionado", "choque con fuga") por 10 años.
- Notificación por mail del estado del reporte a los 7 días. Debo guardar un timestamp del reporte para saber cuando pasan 7 días y debo guardarme el email de quien reportó
- Debo guardarme los reportes de los reportes que no fueron infracciones? Asumo que si por la auditoría.


Analisis de Endpoints:

- POST /infraccion -> Permite reportar las infracciones
	 body = {foto, tipo multa, descripción, timestamp, email_reportante}
     retorna {UID_reporte}

- GET /reporte/:UID -> Permite ver los reportes para evaluar manualmente. Permite filtrar por ID de infraccion y fecha. El ID de la infracción se debe obtener de otro lado.

 Responde con {ID_infraccion, foto, tipo multa, descripción, dni_dueño}.
 
 (Agregar un endpoint más por el tema de evalución manual de la multa? - Tengo uno para obtener, pero no uno que permita modificar el estado)

Análisis de volumen:

Por cada reporte, yo me guardo :
- La foto, la cual para poder verse solo se envía con blanco y negro y de resolución 600 * 400
- Tipo de multa podría representarse con int,(1,2 o 3), en ese caso necesitaría 1 byte.
- La descripción va a ser de aproximadamente 200 bytes (depende del limite que se imponga)
- El timestamp es de no más de 15 Bytes 
- El mail del reportante de 100 Bytes (No se cuanto es el largo máximo)
- Nombre del titular (200 Bytes) y mail 
- Estado (indefinido, hubo infraccion o no hubo) -> 2 Bytes

En este caso, se nota que el tamaño del repote va a estar guiado por el tamaño de la imágen. En este caso, la imágen es de 600 x 400 = 240 kB

Por lo tanto, cada reporte va a ocupar algo del orden de 250KB

Para estimar la cantidad de reportes que puede haber por día, tengo que CABA tiene 3 millones de personas, de las cuales solo un 10% tiene vehiculo, por lo que tenemos 300.000 vehiculos. Teniendo en cuenta que los vecinos son los que hacen reportes, asumo que va a haber un 1% de reportes comparado con el total del vehiculos.

Entonces hay un total de 3000 reportes por día.

Por lo tanto, en un día hay un volumen de 1/4 MB x 3000 =  1 MB x  750 = 750 MB 

En 1 año, tenemos un total de 750 MB * 365 =~ 300 GB
Y en 10 años tenemos un total de 3000 GB = 3 TB

Como cuando un reporte supera los 10 años desde que se hizo se eliminan de la base, no se va a necesitar mucho más espacio, por lo que es factible.

Análisis físico.

En este caso, como el sistema es solo para CABA, voy a tener 1 servidor en cada cómuna que se encarga de enviar todo a un servidor central donde se van a procesar los datos.

[Diagrama de la distribucion fisica - Servidores de comuna y uno central]

Como solo debo guardar los datos durante 10 años, necesito un cleaner que se encargue de eliminar los registros viejos 

En este caso, los servidores de cada comuna se encargan de detectar la patente de la foto y de obtener los datos del dueño del auto (Nombre y email) antes de pasarlo al servidor central 

[Diagrama de robustez]


# Tercer final (wassap) 

## Preguntas teóricas

**1- ¿En que casos se utiliza la estrategia de throttling o descarte de paquetes?¿ Que impacto negativo produce?**

Throttling = limitar capacidad de la red

Estas estrategia se utiliza cuando las redes están congestionadas para evitar colapsos de la red así como también se utilizan para descartar paquetes maliciosos. 

El impacto negativo que producen es que se deben hacer reintentos ya que cualquier paquete puede ser descartado y se aumenta la latencia.

**2- Defina el concepto de Orden Total. De un ejemplo donde se cumpla dicho orden para cierto conjunto de eventos** 

Orden total nos dice que dado varios procesos que se envían mensajes, si el Proceso 1 envía un mensaje y el Proceso 2 también, entonces si un proceso recibe primero el mensaje del proceso 2 y luego el del proceso 1, todos los procesos deben recibir los mensajes en ese mismo ordenar

Ejemplo: (Es mostrar las lineas temporales usando 3 procesos, donde los 3 envían mensajes. No se si se referira a otro tipo de ejemplo?)

**3- Describa los conceptos fundamentales de la arquitectura "Pipes & Filters". Explique las estrategias de procesamiento Workers per Filter y Workers per Item**

Pipes & Filters es una arquitectura que utiliza el concepto de pipeline, es decir, que un conjunto de datos va a ir pasando por un conjunto de etapas, donde se le van a aplicar distintas operaciones de forma secuencial. Las etapas donde se aplican esas operaciones se los conoce como Filters.

Cada filter puede tener varios procesos corriendo de forma concurrente para poder procesar más items al mismo tiempo 

Worker per filters consiste en tener un proceso por cada filtro, en donde un item entra, se le aplica una transformación y luego se envía al siguiente filtro. En este caso, cada filtro tiene por lo menos 1 proceso 

En cambio, workers per item lo que hace es agarrar un item y llevarlo por todo el pipeline, es decir, que tener un proceso que se encarga de aplicarle todos los filtros al item.

**4- Explique los factores de diseño de la arquitectura BigTable. ¿Como logra escalar dicho sistema?¿Que restricciones presenta a los usuarios?**

Big Table esta pensado para guardar grandes volumenes de datos en formato clave-datos, donde los conjuntos de datos son conocidos como columnas. 

BigTable fue pensado para favorecer las escrituras de muchos datos, teniendo asi un velocidad constante, lo cual termina siendo lento para pocos datos, pero rápido para muchos datos

Este esta compuesto por Tablets (guardan las filas de datos según claves de forma particionada, es decir, si las claves son similares, van a estar en la misma tablet) y la arquitectura usa una jerarquía de 3 niveles de tablets

El primer nivel guarda la metadata para saber en que tablet va a estar los datos.El segundo nivel por lo general guarda tablets (las del 3er nivel) y el tercer nivel se encarga de guardar datos

Este sistema para escalar lo que hace es detectar que una tablet tiene muchos datos y lo particiona horizontalmente segun claves: "parte la tabla a la mitad". El problema que tiene es que si las claves no estan distribuidos de manera uniforme y son similares, al hacer el particionamiento va a seguir quedando un tablet casi llena y otra que va estar muy vacía.

Las restricciones para los usuarios son las siguientes:
- Las claves que se usen deben estar distribuidas de forma aleatoria ya que si las claves son muy similares, puede generar problemas a la hora de particionar
- Las escrituras van a ser rápidas si son muchos datos, pero en el caso de usar pocos datos, van a ser lentas

**5- ??? No hay 5**

**6- Calcule los 9s de disponibilidad de un sistema con un nodo único de ingreso con P(availability)=0.99 que balancea la carga en 3 nodos identicos de backend con P(availability)=0.95 para cada uno.

Lo que tenemos es los siguiente:

| Load balancer | ---- | Cluster de 3 nodos backend| 

P(Availability-cluster) = 1 - P(fail-nodo) x P(fail-nodo) x P(fail-nodo) = 1 - 0.05 x -0.05  x -0.05 = 1 - (5 x 10^-2) x  (5 x 10^-2) x  (5 x 10^-2) =1- 25 x 5 x  = 1 - 125 x 10^-6 = 
= 1 - 0.000125 = 0.999875 =~ 0.999

Entonces, la probabilidad de que el sistema este disponible es 

P(sys-availability) = P(Availability-balancer) x P(Availability-cluster) = 0.99 x 0.999 =
  99 x 10^-2 x 999 x10^-3 = 99 x 999 x 10^-5 = (81 + 810 +8100 + 810 + 8100 + 81000) x 10^-5 = (81000 + 16200 + 1620 + 81)  x 10^-5 = 98901 x 10^-5 = 0.98901 

Por lo tanto, el sistema tiene un solo 9 de disponibilidad ya que P(availavility) = 0.98901


**7 - ¿Que es un sistema de Tiempo Real? Brinde algún ejemplo**

Un sistema real time es un sistema donde se tienen requerimientos temporales que deben cumplirse. Un sistema RT funciona correctamente solo si da las respuestas correctas en el timepo correcto. 

Los sistemas RT deben se previssibles y tolerantes a fallos, especialmente respecto a los tiempos.

Este tipo de sistema puede ser Hard o Soft dependiendo de si al no cumplirse un requerimiento temporal se lo toma como una falla catastrófica o si se permite que no lo cumpla ocasionalmente, a costa de que el resultado sea menos útil.

Un ejemplo de sistema realtime es el marcapasos. En este caso es un sistema Real time Hard ya que un error en cuanto al tiempo implica que el sistema no esta andando correctamente.


## Preguntas practicas

**8- Ejemplifique el uso de DAGs al diseñar el procesamiento de datos las transacciones en cuentas de una billetera virtual. Cada transaccion posee: ID, Num cuenta orige, Num cuenta destino, Tipo cuenta origen, tipo cuenta destino, monto y comentario. Se pretende obtener:**
- **10 comercios que más dinero recibieron**
- **Monto promedio por transferencias entre personas fisicas**
- **Comercios que recibieron al menos 3 transferencias de al menos $1M**

[DAG en carpeta]

**9- Ejemplifique el uso de Map-Reduce al deseñar el procesamiento de todos los mensajes de WhatssApp recibidos por una plataforma de delivery durante el día. Cada mensaje posee: numero de orige, texto, HH:MM:SS y quien lo respondió (nombre de usuario humano, robot, aun no respondido). Se pretende obtener:**
**a) la cantidad de respuestas realizadas por cada usuario humano**
**b)  la cantidad de respuestas realizadas por usuario "robot"**
**c) El numero telefonico que más palabras envió**

(Asumo que es solo completar las funciones de map y reduce. No se si hace falta agregar un seguimiento de como funciona map reduce o no.)

a)

```
#Filtro por quien_respondio != robot o aun no respondido
# Clave nombre usuario, valor = 1

map(key, value):

	if key != "robot" || key != "aun no respondido":
		emitIntermediate(key, 1)

reduce (key, values):
	# Key is username
	# Values is an array of 1s

	emit((username, len(values)))


```

b)

``` 
# Usuario debe ser robot

def map(key, value):
	# Key = quien escribio el mensaje 
	# Value = no importa que es, no lo uso en este caso 

	if key == "robot":
		emitIntermediate(key, 1)

def reduce(key, values):
	# Key = "robot"
	# Value array of 1s
	
	emitResult(len(values))

```


c)

Para este caso necesito hacer 2 map-reduce ya que sino no podría calcular el máximo

```
# Numero que más palabras envió 

def map(key, value):
	# key = nro_telefono de origen 
	# value = texto
	cant_palabras = len(texto.split(" "))

	emitIntermediate(key, cant_palabras)

def reduce(key, value):
	# key = telefono de origen 
	# values = array de cantidades de palabras 

	cant_palabras = sum(values)
	# como consigo el maximo ??? -> hago otro map reduce 
	emit(key, cant_palabras)

def second_map(key, value)
	# Key es el numero de telefono
	# Value es la cantidad de palabras 
	emitIntermediate("", (key,value))

def second_reduce(key, values)
	# Key = ""
	# Value = array de tuplas (numero origen, cant_palabras enviadas)

	tupla_maxima = max(values, key=lambda tupla: tupla[1])
	emit(tupla_maxima[0], tupla_maxima[1]) 

```

**10- Diseñe una arquitectura que asegure la escalabilidad para un sistema con los siguientes requisitos:**
- **Permitir que todo smartphone que acepte los terminos y condiciones de "rastreo y busqueda" reporte periodicamente los IMEIs de los smartphones proximos detectados**
- **Recolectar reportes de los smartphones con timestamp, latitud, longitud, e IMEIs proximos cada 1 hora** 
- **Registrar un IMEI como propio y asociarlo a un email para permitir reportes de perdidos**
- **Consultar hora y ultima ubicacion conocida (calle, altura y ciudad) de un IMEI reportado perdido** 
- **Notificar por email al titular si un IMEI perdido vuelve a conectarse** 
**Resuelva el problema mediante un análisis de volumen, endpoints y vista física con su explicación.**


----- notas ----

De los requisitos obtengo que:
- La 1ra condicion no tiene mucha importancia a la hora del modelado de la solución ya que solo me llegan reportes de los telefonos que lo permitieron. Asumo que en algún lugar puedo consultar si el IMEI lo permitió o no.
- Recolecto reportes cada 1 hora que tienen: (timestamp, latitud, longitud y array de IMEIs cercanos). Esto implica un endpoint para cargar reportes
- Asociar IMEI a un email. Para esto necesito un endpoint para "registrarse"
- Consultar ultima ubicacion (calle, altura, ciudad) de un IMEI reportado perdido. Esto implica endpont para consultas y un nodo que permita obtener la ubicación. Si un celu no fue reportado perdido, no permito consultas. Necesito nodo que permita obtener la calle, numero y ciudad a partir de latitud y longitud.
- Si detecto conexión de IMEI perdido, notifico por mail. Cuando me llega un registro de un telefono, me fijo si algun IMEI cercano está perdido 

Asumo que es solo para argentina ya que no dice ninguna ubicación (si es mundial o no)
Asumo tambien que guarda indefinidamente los datos de los que se registraron (No se pueden dar de baja)
Asumo que si fue reportado perdido me lo dan desde otro lado? o debería tener endpoint?

---- Solucion 

Análisis de endpoints.

En este caso, voy a necesitar los siguientes endpoints:

**POST /reporte** -> los celulares envían los reportes a este endpoint

body = {timestamp, longitud, latitud, array de IMEIs cercanos}
return status 200

**POST /registrar** -> Para asociar IMEI a email 

body = {email, IMEI}
return = status 200 o status 400 (si ya esta registrado) 

**GET /consultar/:IMEI** -> Permite consultar ultima ubicación de un smartphone perdido de acuerdo al IMEI 

body = {}
return status 200 body = {hora, calle, altura, ciudad} || status 400 (si no se reporto como perdido)


Analisis de volumenes.

En este caso, solo me interesa guardarme los datos de los IMEIs que fueron registrados, ya que tengo que guardar el email cuando se registra y son los únicos que pueden ser detectados como perdidos.

El problema es que si uno no esta perdido, pero esta registrado, no debería andar avisando nada

En este caso, el storage va a tener que guardar:
- IMEI  -> 20 bytes
- EMAIL asociado -> 200 bytes
- Ubicación (calle, altura, ciudad) -> 200 bytes (calle pueden tener nombres largos y lo mismo para email, pero la altura nunca es más de 5 bytes)
- Timestamp -> 20 bytes 

Por lo tanto, cada registro ocupa alrededor de 0.5 KB

Asumo que cada persona tiene 1 solo telefono y esto solo es en argentina, por lo tanto en toda la argentina hay un total de 50M de telefonos. De estos 50M, asumo que todo el país se asoció un IMEI al email para poder reportar si se perdió. Y de todos esos asumo que estamos en el peor de los casos, por lo que tengo 50M de telefonos perdidos.

==En este caso yo no me guardo los reportes, sino que me guardo la info anterior por cada IMEI distinto, por lo que no hace falta estimar cuantos reportes tengo por hora? (o lo estimo igual aunque no lo guarde?)==

Entonces, constantemente estaría guardando un total de:

50.000.000 x 0.5 KB = 2.500.000 KB = 2.500 MB = 2.5 GB 


Análisis físico

Por un lado, al ser en toda la argentina voy a tener un repetidor de información en cada provincia los cuales se van a comunicar con un servidor central 


[Esquema de siempre 4 nodos unidos a uno central]


[Diagrama de robustez en la carpeta]

En este caso, el repetidor que esta en cada comuna se encarga de:
- Obtener la calle, altura y ciudad del reporte a partir de la longitud y latitud. Esto se hace en el nodo Encontrar Ubicacion 
- A partir del arreglo de IMEIs, se encarga de obtener cuales de esos fueron reportados como perdidos usando un servicio externo,  actualiza la DB con la ubicación y el timestamp del reporte, asi como también envía la informacion filtrada al nodo que se encarga de notificar por mail.

El nodo "Notificar por email",  a partir de la informacion que le llega de las colas, obtiene por cada IMEI que recibe el mail asociado de la DB y envía la notificación con los datos del reporte 

Al asociar un email a un IMEI, pasa por un nodo que se encaga de verificar que el mail y el IMEI no hayan sido registrados anteriormente. Si no fue registrado, actualiza la DB con el mail asociado 

Por utimo, al consultar por un IMEI, pasa por un nodo que se encarga de ver si el IMEI fue esta en la DB donde estan los perdidos y envía tanto la ubicacion como la hora en la que fue visto por ultima vez.

(Ahora que lo pienso mejor, debería tener un ENDPOINT para que reporten si se perdió el telefono y esa info guardarla en la DB ya que simplificaría ciertas cosas del diseño. En este caso, la deteccion de IMEI perdidos se haria en el servidor central ya que ahi esta la DB con la info y el que obtiene la ubicación guarda los datos en la DB, el resto queda muy parecido.)

# 4to final (wasa)

## Preguntas teoricas 

**1- Suponga que debe implementar un protocolo de health-check de server a clientes mediante sockets ¿Que condiciones se deberían cumplir para que sea mas conveniente utilizar TCP frente a UDP?**

Para que sea más conveniente usar TCP sobre UDP al realizar un protocolo de healthcheck, se debería cumplir:
- Necesidad de que el protocolo sea reliable, ya sea porque es importante que todos los mensajes que mando se respondan
- No hay gran cantidad de clientes ya que eso implica mantener muchas conexiones, lo que aumenta bastante el consumo de recursos por parte del servidor 

(Pensar si puedo agregar algo más.)

**2- Explique la utilidad de los algoritmos de relojes Lógicos de Lamport y vectores de relojes explicados en clase ¿Para que sirve cada uno?**

La utilidad de estos algoritmos es que nos permiten ordenar estados del sistema y en el caso del algoritmo de vectores tambien nos permite ver la causalidad entre los distintos procesos.

En el caso del algoritmo de Lamport, solo puede establecer bien la causalidad de eventos si se utiliza solo con 1 proceso, en el caso de usar vario procesos el algoritmo no me garantiza que C(s) < C(t), s -> t, (siendo C(x) el valor del reloj lógico y s y t 2 estados del proceso) ya que le falta información. Este algoritmo se usa como base en el algoritmo de vectores.

En el caso del algoritmo de vectores de relojes, este nos permite obtener la causalidad entre estados, ya sea si un estado sucedió antes que otro o si son concurrentes. Para eso se comparan los vectores de relojes de los procesos. Si el vector de s es <= en todos los valores que t y para 1 solo es <, entonces se puede garantizar que s -> t, caso contrario son estados concurrentes.


**3- ¿Que se entiende por Atomicidad de mensajes en el contexto de grupos de comunciacion? ¿Con que estrategias es posible garantizarla?**

(No estoy seguro sobre la estrategia)

En el contexto de grupos de comunicación, se entiende a la atomicidad de mensajes a que si un miembro del grupo envía un mensaje, este lo reciben todos los demás miembros del grupo o no lo recibe nadie.

Para poder garantizar esto hay varias estrategia, una de ellas implica que el envío se haga de forma centralizada y que si a los procesos les llegó el mensaje respondan con un ACK.
En el caso de que uno no haya respondido con el ACK, entonces se tendría que enviar un mensaje para abortar. Algo útil para esto es utilizar una holdback queue 

Si todos recibieron el mensaje y enviaron el ACK, entonces se puede enviar un mensaje para que se proceda.

Estas estrategias requieren de tener canales de comunicación reliable para evitar la perdida de mensajes o podría contar con un sistema de timeouts y retries.

==**4- Asumiendo la existencia de un equipo de hasta 10 agentes de atención al cliente en una empresa telefónica, defina como implementar in middleware distribuido que permita a los agentes llamar al siguiente numero en fila. ¿ Donde almacenaría la información relacionada al ultimo numero llamado?**==

(Como es un middleware distribuido, no puedo tener un broker centralizado que guarde la información y envie por colas a cada agente 1 numero. El broker en este caso esta dentro de cada nodo/"agente")

(Asumo que hay algun servidor o algo donde estan guardados los numeros)

?'?????



**5- Describa 2 estrategias útiles para garantizar tolerancia a fallos en una arquitectura distribuida. De un ejemplo de arquitectura donde dichas estrategias deben ser utilizadas.**

Una estrategia implica usar redundancia de nodos y algoritmos de consenso. En este caso, lo que se hace es tener replicas de un nodo y se utiliza al algoritmo de consenso para poder acordar quien es el líder. Este tipo de estrategias se puede usar en arquitecturas donde hay un nodo central, el cual es un único punto de falla.

Otra estrategia implica ??? (Sistema que levanta nodos caidos + recuperación)


**6- El estudio de sistemas confiables implica la revisión de varias propiedades que impactan en la tolerancia a fallos del sistema. Detalle los conceptos claves de al menos una de ellas.**

Las propiedades que impactan en la tolerancia a fallos del sistema son:
- Disponibilidad: es la probabilidad de que un sistema este funcionando de forma correcta, es decir, que ofrezca servicios. Esto se mide con 9s de probabilidad
- Confiabilidad: es la capacidad del sistema de responder de forma correcta los pedidos que le llegan
- Seguridad: en un sistema tolerante a fallos, no debe suceder nada catastrófico ante errores. Esto implica que el sistema pueda recuperarse ante cualquier tipo de fallo.
- Mantenibilidad: es la velocidad con la que se puede actualizar o arreglar el sistema 

En el caso de la disponibilidad, se puede mejorar agregando redundancia de nodos a costa de que, dependiendo del caso, no se responda con los datos más actualizados (reduce la reliability). Que el sistema este disponible implica que ofrezca un servicio adecuado (aunque sea mínimo) ante la presencia de fallos

(Creo que refiere a esto.)

**7- Explique una arquitectura de DFS que conozca. Indique un caso de uso donde elegir dicho mecanismo sería ventajoso y otro donde sería perjudicial** 

Una arquitectura de DFS conocida es la que usa Hadoop. En este caso se cuenta con 2 tipos de nodos:
- Namenode: se encarga de guardar la metadata y sabe ubicar los datos dentro del datanode 
- Datanode: se encarga de almacenar los datos. Esta compuesto por bloques de alrededor de 128 MB.

[Agregar dibujo de arquitectura]

Cuando un cliente quiere consultar, va directo al namenode el cual le dice en que datanode se encuentra y luego el cliente obtiene los datos del datanode. 

Los factores de diseño de Hadoop fueron los siguientes:
- Tolerar los fallos ya que son comunes y es más economico tolerarlos 
- Favorecer lectura sobre la escritura, lo que implica que es costoso escribit/modificar/crear archivos 
- Favorecer realizar operaciones sobre grandes cantidades de datos o streaming por sobre pocos datos 
- Que sea portable a cualquier tipo de hardware 

Por lo tanto, esto sería ventajoso usarlo en el caso de que haya que guardar grandes cantidades de datos para hacerle operaciones (ejemplo data science), pero no es bueno utilizarlo cuando no tengo que guardar grandes cantidades de datos

## Preguntas practicas 


**8- Utilizando request-reply, implemente una barrera para que 3 procesos llamados workers puedan sincronizarse en cierto punto del algoritmo que se ejecutan. Detalle supuestos de ser necesarios. Utilice pseudocódigo o diagrama UML (secuencia, colaboracion o actividades)**

Tomo como supuestos:
- Hay un servidor que se encarga de recibir las request de los procesos, el cual es tolerante a fallos 
- La comunicación es reliable
- En caso de que un proceso se caiga mientras espera la barrera, este se vuelve a levantar eventualmente y envía la request nuevamente (mantiene le ID)
- Se usa un request-reply sincronico, permitiendo que los procesos se queden bloqueados hasta recibir una respuesta
- Se tiene un middleware que se encarga de envia los mensajes al servidor 

Cada cliente al querer sincronizarse, hace lo siguiente:

```
def main():

-- Ejecuto algoritmo --

middleware.request_barrier_wait("barrera") #queda bloqueado 

-- Sigue algoritmo --

```


Por otro lado, el servidor tiene el siguiente código:

```
El servidor guarda el siguiente estado:

def main():
	N = 3 # Cantidad de procesos máxima a bloquearse 
	cant_bloqueados = 0
	ids_procesos_bloqueados = [] #es un set que no admite repetidos

	mientras el servidor este activo:
		
		id_proceso = middleware.obtener_request()

		cant_bloqueados += 1 
		ids_procesos_bloqueados.append(id_proceso)

		if cant_bloqueados == N:
			for processID in ids_procesos_bloqueados: 
				middleware.reply(processID, "unlock")
			
			cant_bloqueados = 0
			ids_procesos_bloqueados.empty()

```


[Se puede agregar diagramas de actividad o de secuencia para mostrar el tama de que se queda bloqueado o que hace el servidor, pero con esto creo que alcanza]

(No se si se espera hacer algo asi, o no se permite tener un proceso servidor que se encargue de eso. En ese caso, debería enviar el reques a todos los procesos lo cual sería raro.)

**9- Utilizando map-reduce, procese los eventos de mensajes recibidos en un chatbot mediante un listado de tuplas (messageID, clientPhoneNumber, text, date, time) y calcule:
a) Cantidad de clientes atendidos por día del mes (1, 2, ..., 30, 31)
b) Listado de teléfonos con mas de 10 mensajes recibidos en un mismo día. Utilice pseudocódigo** 

a) 

```
def map (key, value):
	# Key = messageID -> 1
	# Value = Date -> 12-01-2024

	day = value.day
	emitIntermediate(day, 1) # -> 12, 1

def reduce(key, values):
	# Key = numero de día 
	# Values = array de 1s 

	total_clients = len(values)
	emit(key, total_clients)

```

b)

```
def map(key, value):
	# Key = clientPhoneNumber -> 2364123456
	# Value = Date -> 12-01-2024

	day = value.day
	emitIntermediate(key, day) # -> 2364123456, 12

def reduce(key, values) 
	# Key = clientPhoneNumber -> 2364123456
	# Values = array de días. Si hay repeticiones, es porque hubo mas de un mensaje en ese día -> [12, 31, 15, 12, ...]

	msg_per_day = {}

	for day in values:
		if day not in msg_per_day:
			msg_per_day[day] = 0
		msg_per_day[day] += 1


		if msg_per_day[day] >= 10: # hay más de 10 en un día, entonces lo guardo al numero
			emit(key, "") # 2364123456, ""
			return 

```



**10- Diseñe una arquitectura que asegure la escalabilidad para un sistema con los siguientes requisitos:**
- **Tomar los pedidos de cafe y cookies en terminales AUTOSERVICIO de una cadena multinacional con sucursales en las capitales de todos los países, para que lo empleados preparen y cobren pedidos en caja**
- **Emitir una comanda de pedido con cantidad y producto en la terminales COCINA y un ticket de cobro con precio unitario y total en las terminales CAJA de la sucursal en cuestión**
- **Recibir de forma centralizada la lista de productos y fotos a vender en la terminales de cada sucursal** 
- **Recibir de forma centralizada los pedidos de cambio de precio unitario para cada sucursal**
- **Permitir consultar ventas totales por día. por día y por sucursal**
**Resuelva el problema mediante analisis de volumen, endpoints y vista física con su explicación**

(DUDA: Solo modelo el servidor central o también toda la estructura de las sucursales. Esto ultimo implica endpoints para manejar el tema de los pedidos)
(DUDA2: Hace falta guardarme los pedidos que se hacen?)
(DUDA 3: la sucursal guarda alguna info? porque como dice que recibe la lista de forma centralizada y tambien los cambios de precios no estoy seguro. Asumo que sí.)


--- Notas ---
- Cada sucursal tiene las siguientes terminales: COCINA , CAJA, AUTOSERVICIO
- En las terminales de AUTOSERVICIO se toman los pedidos (producto y cantidad)
- En las terminales COCINA, se emite la comanda (Pedidos y cantidades) para preparalo
- En las terminales CAJA se emite ticket de cobro (precios uniarios y total) (cantidad hace falta?)
- El servidor central va a responder todo lo que tiene que ver con "consultas centralizadas"
- El servidor central envía a las terminales la lista de productos y sus respectivas fotos 
- El servidor central envía los cambios del precio unitario de los productos 
- El servidor permite consultar ventas totales por día y las ventas totales por día y sucursal ?? esto no estoy seguro

El server central se encarga de guardar para cada producto: (producto, foto del producto, precio unitario) y envía estos a las sucursales. Como esta en todo el mundo,  lo mejor es que todos las sucursales tengan toda esta información y el servidor central avise de cambios.
También se necesita un conversor de precios ya que cada país se maneja con distintas monedas

El server central tambien debe tener un endpoint para realizar cambios de precios?

El server central tiene productos que pueden ser café o cookies (distintos tipos?)

------

**Análisis de endpoints:**

Cada sucursal tiene los siguientes ENDPOINTS:

**POST /pedido** -> aca se reciben los pedidos. Se comunica con la terminal de AUTOSERVICIO 

body = {ID_pedido, [(producto, cantidad), ...]}
return: status 200, ID_pedido

**GET /productos** -> Obtener lista de productos

return: status 200, body {(producto, imagen, precio_unitario), ...}

**GET /comanda** ->de aca recibe la terminal de COCINA 

return: status 200, body = {ID_pedido, [(producto, cantidad), ...]}

**GET /ticket** -> de aca recibe la terminal de CAJA 

return: status 200, body = {ID_pedido,  [(producto, precio_unitario), ...], precio_total }


**POST /producto/:nombre_producto** -> el server central actualiza el precio del producto 

body = {precio_unitario}
return status 200 

**POST /productos** -> el server central envia la lista de productos aca

body = {nombre_producto, foto, precio}
return status 200 


El servidor central tiene los siguientes endpoints:


**GET /ventas** -> Permite consultas por día y sucursal. Asumo que el ID de la sucursal se obtiene de otro lugar

body = {día, ID_sucursal}
return: status 200, body = {cant_ventas}

POST /venta -> confirma que se vendío un producto 

body = {fecha, sucursal, monto}
return: status 200


**Analisis de Volumen:**

Cada sucursal va a tener que guardar los siguiente por cada producto:
- Nombre -> 300 bytes
- Foto 
- Precio unitario ->  si lo guardo en float no es más de 4 bytes


En este caso el tamaño del nombre y precio unitario es despreciable frente a la foto, por lo que el tamaño va a estar dado por la imagen.

Al ser foto de un producto la calidad debe ser buena y debe ser RGB, por lo que se puede usar una foto de 1300 x 700 (HD) pixeles donde cada pixel es de 3 byes. Por lo tanto el tamaño de cada imágen es de: 

1300 x 700 x 3 = 13 x 7 x 3 x 10^4 = (70 + 21) x 3 x 10^4 = 91 x 3 x 10^4 = 2.730.000 bytes =~ 3MB

Al solo vender cookies y cafe, no debería haber más de 100 productos (10 me parece que es poco, pero mas de 100 ya es mucho), entonces se debe guardar alrededor de 

Entonces, cada sucursal guarda alrededor de:
100 x 3MB = 300MB de informacion.

Además de esto, debo guardarme lo siguiente en el servidor central para permitir consultas:
- Fecha -> 10 Bytes
- Sucursal -> 200 bytes (asumo que es el nombre de la ciudad)
- Monto_vendido -> 8 bytes 

Entonces cada registro va a ser del orden de 200 Bytes

El tamaño de esto va a depender de la cantidad de sucursales y de cuanto tiempo guardo estos registros.

Asumo que hay alrededor de 200 países, asumiendo que todos tienen 1 capital, entonces por día tendría guardado:

200 Bytes x 200 sucursales = 40000 Bytes = 40 KB

En un año tendría guardado 

40 KB x 365 = 200 + 2400 + 12000 = 36600 KB =~ 40 MB

Y en 10 años guardaría un total de 400 MB, por lo que no requiero de mucha memoria.

Entonces, en total estaría guardando alrededor de 1GB de información en el servidor central, teniendo en cuanta 10 años.

Analisis físico:

[Diagrama de componentes]

Los servidores de cada sucursal se comunican con el servidor central solo en el caso de que se confirme alguna venta. Esto se hace para que se pueda mantener actualizada la cantidad de ventas en el dia por sucursal.

Tambien, se tiene al servidor central que le va a enviar la lista deproductos a cada sucursal y tambien le va a avisar si hay algun cambio en los precios de los productos.

[Diagrama de robustez]

Cada sucursal va a guardar los datos de cada producto dentro de una DB particionada por el nombre del producto (asumo que no hay productos distintos con mismo nombre), para hacer que la consultas por precios de un producto sea más rapida.

El nodo de procesar pedido, lo que va a hacer es agarrar los pedidos y enviarlo a las terminales de COCINA y CAJA. Este nodo se comunica con la DB con el fin de poder obtener el precio unitario de cada producto y calcular el total. 

Los endpoint mediante los cuales la sucursal recibe la actualización de precios o la lista de productos (con precios), pasan por un conversor de moneda y se lo guarda/actualiza en la base de datos. Esto se hace porque no todos los países usan la misma moneda, por lo que es más sencillo guardarlo y no tener que realizar la conversión a cada rato.

Por último, el servidor central contiene una DB particionada por fechas, en donde guarda cual ve la cantidad de ventas para cada sucursal por fecha. Esta DB tiene que favorecer la escritura por sobre la lectura ya que va a recibir constantemente ventas de las sucursales. Esto se puede cambiar en el caso de que la no se me avise sobre cada venta, sino que al finalizar el día se reciba el monto total de cada sucursal.


# 5to final (pdf1)

## Preguntas teóricas 

**1) Indique las ventajas y desventajas de utilizar comunicación por sockets TCP frente al pasaje de mensajes por colas mediante MOM en un sistema distribuido típico.**

Ventajas:
- El envío de mensajes no pasa por un broker intermedio, reduciendo latencia
- Es facil saber si el proceso del otro lado se desconectó (En MOM con cola no se puede saber)
- Permite comunicación bidireccional con una sola conexion, mientras que el MOM necesita 2 colas para esto (una para enviar y otra para recibir)

Desventajas:
- En caso de que se desconecte uno de los procesos que se están comunicando, se debe esperar a que se vuelva a conectar para reintentar el envío del mensaje. Con los MOM el mensaje se puede persistir en la cola haciendo que no se deba reintentar el envío del mensaje frente a una desconexion
- TCP requiere que ambas partes conozcan sus direcciones y estén activas al mismo tiempo para comunicarse, mientras que MOM no requiere que estén conectadas ya que los mensajes se guardan en las colas.


**2) ¿A qué se refiere el concepto de drift de un reloj? Explique algún algoritmo para mitigar su efecto.**

El drift del reloj se refiere a que un reloj físico puede descalibrarse por cambios en la presión, temperatura, humedad, etc. por lo que hay que calibrarlo cada cierto tiempo. Una forma de poder mitigar el efecto es sincronizándolo contra un servidor. Esto requiere tener en cuenta los delays de la red, por lo que se hace utilizando el algoritmo de cristian.

El algoritmo de cristian lo que propone es calcular el tiempo al que hay que ajustarse teniendo en cuenta los delays de la red. Por este motivo, propone la siguiente formula:

T = T_server + (RTT)/2


**3) En qué se diferencia la exigencia de Orden Total vs Orden Causal para una serie de eventos distribuidos. Dé un ejemplo donde se respete el primero y a su vez se viole el segundo.**

La diferencia es que el Orden Total implica que si tengo varios procesos que se envían mensajes entre ellos, a todos les llega los mensajes en el mismo orden, es decir, si el proceso 1 envia el mensaje M1 y P2 envia el mensaje M2 a todos los procesos, si uno recibe el mensaje M2 primero, entonces todos los procesos deben recibir el mensaje M2 antes que M1.

Orden Causal hace referencia a que si un proceso envía un mensaje (M1), el cual genera otro mensaje (M2), entonces todos los procesos deben recibir el mensaje M1 antes que M2.

El ejemplo donde se viola orden Causal pero no Total es el siguiente:

![[Pasted image 20250206160738.png]]

En este caso se respeta el orden total ya que todos los procesos "ven" lo mismo: primero m1, liego m2 y por ultimo m2. Sin embargo no se cumple el orden causal ya que el mensaje M2 precede causalmente al broadcast de M3 pero se recibe M3 antes que M2.

Tambien por las dudas hacer el ejemplo contrario para practicar

[Misma carpeta]

En este ejemplo el mensaje M1 genera a M2 y se recibe siempre M1 antes que M2, sin embargo el P3 tiene un mensaje M3 que no fue causado por nadie, el cual P1 lo recibe antes que M2 y P2 lo recibe despues de M2, rocmpiendo el orden total.

**4) Explique cómo funciona el modelo RPC para arquitecturas distribuidas incluyendo el flujo de comunicaciones para un ejemplo propuesto.**

RPC en arquitecturas distribuidas se usa para cuando una computadora no tiene recursos suficientes para ejecutar cierta función (pueden ser otros motivos), por lo que llama a un servidor que contiene un conjunto de funciones/procedimientos con el fin de que ejecute la función y le devuelva el resultado.

En un ejemplo sería el siguiente:
- El proceso quiere ejecutar la función A, por lo que se comunica con el servidor para que ejecute la función. El proceso se encarga de enviarle los argumentos (no pueden ser punteros o referencias)
- Cuando el proceso llama a la funcion para ejecutarla, esto pasa por un stub que se encarga de serializar la información y se envía mediante el modulo de comunicación al server
- El servidor recibe el mensaje para ejecutar la función A, el cual pasa por un STUB para deserializar la informacion y luego ejecuta la función A.
- Una vez se ejecutó la funcion, se hace el camino inverso para enviarle el resultado de la operación al cliente 

Cabe destacar que RPC es un modelo stateless, por lo que no guarda informacion alguna de lo que envió el cliente y las operaciones del cliente no afecta a los resultados dados

**5) ¿Qué componentes requiere una arquitectura para brindar elasticidad? Describa una situación donde este comportamiento sea deseable.**

Para brindar elasticidad, se requieren los siguientes componentes:
- Load balancer: se encarga de enviarle datos a las instancias activas y de hacer que no le lleguen más datos a instancias que fueron removidas o que se cayeron
- Auto-scaler: sabe hacer el deploy de instancias y aumenta/disminuye las instancias de acuerdo a las metricas
- Monitoring: envía metricas sobre el sistema

Este comportamiento de adaptar el los recursos del sistema dependiendo de la carga puede ser util en escenarios donde tengo una carga del tipo "unpredictable burst", donde puede ser que un día tengo mucha carga de request pero otros días que no, permitiendo ahorrar costos.

OBS -> Siempre hay que limitarlo igual porque si no lo haces, cuando tengas mucha carga se van a disparar los precios 

**6) Considerando un escenario de particionamiento de datos en un sistema distribuido, explique al menos dos alternativas de enrutamiento de una consulta para poder ubicar la información deseada.**

Teniendo en cuenta el particionamiento de datos, hay varias formas de enrutamiento:
- El cliente se comunica con cualquier partición para preguntarle si tiene lo que busca. En el caso de que lo tenga, se lo da, sino le dice en que particion esta guardado. Luego el cliente va a buscarlo a dicha particion 
- Tener un Nodo que sabe ubicar los datos (guarda la metadata) se encarga de recibir los pedidos del cliente para ubicar los archivos y le responde en que particion se encuentra.

**7) Explique un algoritmo de exclusión mutua distribuida indicando sus ventajas y desventajas.**

Un algoritmo de exclusión mutua es el de Token Ring. Este algoritmo requiere que cada nodo tenga un ID único, y establecer un orden entre los procesos. La idea es la siguiente:

Se arma una topología de anillo con los distintos procesos, donde cada proceso solo se comunica con el nodo siguiente (necesito tener un orden designado para esto). Una vez se tiene la topología, se genera un token que se va pasando entre los procesos.

El proceso que tenga el token va a poder acceder a la seccion critica. Si el proceso no quiere acceder a la seccion critica, entonces pasa el token.

Como solo el proceso que tiene el token puede acceder a la seccion critica, entonces se garantiza que solo 1 proceso va a poder acceder al mismo tiempo.

Las ventajas de este algoritmo es que no usa un servior central que trae un unico punto de falla y es fácil de entender e implementar.

Las desventajas son que:
- Si se cae algun nodo, se debe volver a armar el anillo lo cual a veces no es tan sencillo 
- Es dificil generar el anillo de forma dinámica
- El token se puede perder, por lo que hay que tenerlo en cuenta. 

## Preguntas practicas

**8- Utilizando una arquitectura Request-Reply, ejecute de forma remota la productoria de un arreglo de enteros de manera asincrónica. Mientras la ejecución transcurre, calcule la  sumatoria. Por último, imprima ambos resultados. Utilice pseudocódigo o diagrama UML (secuencia, colaboración o actividades).**


Al ser un request-reply asincronico, la idea seria:
- Cliente envia la request junto con el array de enteros al servidor 
- El servidor envía un ACK y comienza a realizar la productoria 
- Cliente recibe el ACK (junto con id de request) y empieza a calcular la sumatoria 
- Cliente termina de calcular sumatoria y hace polling para obtener los resultados 
- Cliente obtiene el resultado de la productoria e imprime ambos 

Supuestos:
- Se asume uso de canales reliables
- Hay 2 request que puede llamar el cliente: calcular_productoria(array) y obtener_resultados(request_ID)
- Hay un middleware que se encarga de comunicar al cliente y el servidor


```
# Cliente 


def main():
	mid = Middleware()
	number_array = [1, 2, ...]

	request_ID = mid.calcular_productoria(number_array) #send request to server 

	sumatoria = 0
	for number in number_array:
		sumatoria += number 

	while True:

		respuesta = mid.obtener_resultados(request_ID)
		if respuesta != "NOT FINISHED":
			break 
		sleep(POLLING_SLEEP_TIME)

	print(f"sumatoria: {sumatoria}, productoria: {respuesta}")

```

```
#Server 

def handle_productoria(request_ID, array):
	productoria = 1 

	for number in array:
		productoria *= number 

	save_request_result(request_ID, productoria) # guarda el resultado para devolverlo luego 

def handle_results(request_ID)
	
	request_state = get_request_state(request_ID) #se fija si request tiene resultado o se esta procesando todavia

	if request_state == "NOT FINISHED":
		send_request_not_finished() # respondo al cliente que todavia no termino

	result = get_request_result(request_ID) #obtengo el resultado

	send_request_response() #respondo con el resultado de la request

```


**9- Ejemplifique el uso de Direct Acyclic Graphs (DAGs) al diseñar el procesamiento de datos de todas las ventas diarias de una cadena de tiendas de ropa. Cada venta posee ID de tienda y un arreglo con: monto unitario, cantidad y código de producto. 
Se pretende obtener los 10 códigos de producto más vendidos, el monto promedio de venta y la cantidad de ventas con más de 3 productos.**

Calculo que la idea es hacer un DAG. Para cumplir con las request, necesito los siguientes "nodos":
- Sum segun codigo de producto. Suma la cantidad (1)
- Top 10 segun codigo de producto (1)
- Suma de total por venta (2)
- AVG de totales segun ID de venta (2)
- Filtro segun cantidad de productos. (3) # se encarga de contar y filtrar? o hago algo que cuente
- Counter de total ventas (3)

[DAG en carpeta]


**10- Diseñe una arquitectura que asegure la escalabilidad para un sistema con los siguientes requisitos:**
- **Recolectar la información de ventas de pasajes de trenes de larga distancia de toda Argentina, desde las distintas estaciones que funcionan como puntos de venta para los servicios que prestan.**
- **Recibir listado actualizado de servicios a prestar indicando origen, destino, fecha y Cant. de asientos.**
- **Recibir pedido de compra con destino, fecha de partida, nombre de cada pasajero y foto de cada DNI.**
- **Permitir consultar fotos de DNI de pasajeros incluyendo origen y destino si se posee la fecha de partida.**
**Resuelva el problema mediante un diagrama de vista física y su explicación.****


-- Notas --

El primero de todos me dice un resumen de lo que se quiere

De los requisitos obtengo que:
- Endpoint para recibir listado de servicios a prestar (origen, destino, fecha, cantidad_asientos). Este endpoint puede ser un POST /servicios
- Endpoint para pedidos de compras (destino, fecha de partida, nombre de pasajeros y fotos del DNI). El endpoint puede ser POST /pasaje 
- Endpoint de consultas de foto de DNI, origen y destino (REQUIERE FECHA DE PARTIDA). El endpoint puede ser GET /pasaje.
- Debido a las consultas, la info que debo guardar es: Foto DNI, origen, destino y fecha de partida. Puedo particionar por fecha ya que las consultas deben tener eso.
- Debido a la compra de pasajes, debo guardar la info de los servicios (origen, destino, fecha, cantidad_asientos)
- Debido que abarca todo el país, me conviene tener replicadores de información en cada provincia

-- respuesta 


**Analisis de ednpoints**

POST /servicio -> recibe el listado de servicios a prestar 

recibe: Body = { [(origen, destino, fecha, cantidad_asientos), ...]} # array de servicios
retorna: status 200

POST /pasajes -> Permite realizar pedidos de compra de pasajes 
	# creo que falta el origen, pero por algun motivo no esta. Compras pasaje para ir a x lugar, pero no indicas de donde? 

Body = {destino, fecha_partida, [(nombre pasajero1, foto DNI), ...]}

retorna: status 200, body = {id_pasajes: array} o status = 400 si no pudo conseguir algun pasaje 

GET /pasaje/:id_pasaje 

Body = {fecha_partida} 

return: status 200, body = {foto_DNI, origen, destino} 


**Análisis de volumen**

Por un lado, me tengo que guardar los servicios ofrecidos. Cada servicio tiene lo siguiente: 
- ID -> Asumo que es un UUID por lo que ocupa 4 bytes 
- Origen -> 200 BYTES 
- Destino -> 200 BYTES 
- Fecha de salida -> 15 BYTES
- Cantidad de asientos -> 2 BYTES 

Por lo tanto, necesito guardar alrededor de 500 BYTES por cada servicio 

	 # es dificil estimar el volumen de esto porque no tengo ni idea cuantos trenes puede haber por día (seria alrededor de los que van al AMBA)

Asumo que hay 10 trenes por hora que salen en todo el país, por lo que en un día tengo:
10x24 = 240 trenes.

Lo cual es un total de 240 x 0.5 KB = 120KB por día

y asumiendo que nunca borro los datos de los revicios ofrecidos, en un año guardo: 

120 KB x 365 = 7300 + 36500 = 40.000 KB =~ 50MB por año

Por otro lado, me tengo que guardar la información de los pasajes. Cada pasaje tiene los siguiente:
- ID_PASAJE -> 4 Bytes
- ID_SERVICIO -> 4 Bytes
- Foto DNI -> 1MB
- Nombre -> 200 Bytes

Para la foto de DNI voy a tener que decidir un tamaño y si lo guardo a color o no. En este caso, voy a guardar con color y un tamaño de 600 x 400, por lo que cada imagen pesa:

600 x 400 x 3 = 600 x 1200 = 720.000 =~ 1MB 

Teniendo en cuenta que los demas datos a guardar son de tamaños muy pequeños, tomo cada pasaje pesa algo del orden de 1MB 

Debido al analisis anterior, se que por día tengo 240 trenes que salen. Asumiendo que cada tren tiene alrededor de 200 asientos, entonces puede haber un total de 48.000 pasajeros por día.

Si asumo que todos los trenes se llenan todos los días, entonces tengo un total de 48.000 pasajes que guardar. Por lo tanto, por día guardo 

1MB x 48.000 = 48 GB

y mes, guardo un total de: 

48 x 30 = 1440 GB =~ 2 TB

En el caso de tener que guardar estos datos indefinidamente, no es rentable ya que por año terminas guardando 24TB de información. Pero si solo guardo por año o por mes los datos de los pasajes, esta bien ese tamaño. 

**Análisis Fisico**

Por un lado, como esto abarca todo el país, yo tendría repetidores de información por provincia (y otro en CABA) que se encargan de enviar los datos al servidor central.

[Diagrama de componentes para esto]

[Diagrama de robustez]

En el diagrama de robustez, tenemos los siguientes nodos:
- Validar info: valida que la info del cliente sea correcta 
- Verificar disponibilidad: verifica que el servicio tenga asientos disponibles para el pasaje 
- Actualizar servicios: se encarga de agregar o actualizar servicios en la DB 
- Validar fecha: se encarga de verificar que la fecha de salida sea correcta antes de devolver la informacion de la consulta 


# Sexto final 

## Preguntas teoricas 

**1- ¿Que diferencia una arquitectura multithreading de una multiprocessing? Indique ventajas**

La diferencia es que la arquitectura multithreading usa threads, los cuales comparten memoria (heap, data segment, etc.) mientras que la multiprocessing usa procesos que no comparte memoria.

La ventaja de multithreading es:
- Es sencillo compartir datos entre threads
- Los threads son mas livianos que los procesos

La ventaja de multiprocessing es:
- No hay alto acoplamiento como pasa con los threads
- El fallo de un proceso no deja inutilizable a todo el sistema
- Los procesos estan aislados, lo que los hace más seguros

**2- Explique como garantizar un corte consistente al capturar un estado global de cierto sistema distribuido. Indique pasos del algoritmo propuesto.** 

==(Revisar por las dudas la condicion de corte y si envían su estado a los demas o no.)==

Un corte consistente es cuando dado un evento E que esta en el corte, si el evento A genero a E, entonces A tambien esta en el corte. Un algoritmo usado para esto es el de Chandy & Lamport:

En este algoritmo todos los procesos tienen un lugar donde guardan el estado, otro para guardar el corte y usan canales de comunicación reliables (uno para enviar y otro para recibir mensajes). Tambien cualquier proceso puede iniciar el algoritmo.

Algunos de los supuestos que toma son:
- Canales de comunicacion reliable, unidireccionales y orden FIFO

Los pasos son los siguientes:
- El proceso que inicia el algoritmo guarda su estado dentro del corte y envía un marcador a todos los procesos.
- El proceso que recibe el marcador por primera vez, agrega su estado al corte e inicia un log para los mensajes que lleguen durante el algoritmo. Además, envía un marcador a los demás procesos.
- En el caso de que ya haya recibido un marcador, guarda el estado del log dentro del corte.

Este algoritmo termina cuando todos los procesos recibieron los marcadores de los demás procesos.

**3- Explique que estrategias puede implementar un MOM para garantizar tolerancia a fallos. Ejemplifique con algún MOM que conozca o haya implementado.**

Un MOM puede implementar varias cosas para garantizar tolerancia a fallos:
- Hacer que el MOM sea distribuido, garantizando que no haya un unico punto de falla 
- Si el middleware es centralizado, tener replicas para evitar tener un unico punto de falla
- Usar colas de mensajes, las cuales permite guardar los mensajes en el caso de que se haya caido un nodo para que lo reciba después.

Un MOM que conozco es RabbitMQ. Este es un MOM centralizado, al cual se le pueden agregar replicas para evitar que sea un unico punto de falla y utiliza colas de mensajes para la comunicación entre procesos. Estas colas de mensajes se les puede indicar  para que guarden los mensajes en el caso de que no haya nadie recibiendo/agregando nuevos mensajes (siempre y cuando haya espacio disponible). Tambien RabbitMQ viene con un monitor para poder ver metricas, las colas que hay y los mensajes en vivo. 

**4- Explique DSM desarrollando al menos una alternativa de implementación. Indique ventajas y desventajas de dicha arquitectura** 

La idea de Distributred Shared Memory (DSM) es que varios procesos compartan datos a través de una memoria centralizada, sin conocerse entre ellos. Una de las alternativas de implementación implica utilizar replicas de paginas para que los procesos puedan leerlas o modificarlas. La idea de esa implementación es la siguiente:
- El servidor se encarga de guardar las páginas con los datos 
- Un proceso puede pedir replicas de la página para lectura. O la página en si para escritura
- Si un proceso modifica una página, le avisa al server y el server se encarga de invalidar el resto de páginas que tienen los demás procesos. (en otros casos, aplica los cambios a las demás replicas)

La ventaja de esto es:
- Favorece las lectura por las escrituras, por lo que es bueno si hay muchos procesos que leen y pocas modificaciones

Las desventajas son:
- Si se necesita modificar paginas a cada rato, entonces se invalidan las replicas muchas veces y los procesos deben volver a pedirlas 
- Unico punto de falla, salvo que se implemente replicación del servidor. Por este motivo, tambien puede haber cuello de botella.

**5- Describa al menos 2 patrones de carga de apps web e indique como el escalamiento "Horizontal" permite mitigar efectos negativos.**

Algunos patrones de carga de las aplicaciones web son:
- Predictable Burst: se sabe que va a haber un pico de carga en determinadas fechas, por ejemplo en black friday.
- Unpredictable Burst: se sabe que va a haber picos, pero no se sabe cuando. Por lo general esto implica hacer un análisis para saber cuantos recursos se necesitan.

El escalamiento horizontal implica agregar más instancias del servidor de la app. Lo que permite esto es que la carga se distribuya entre los distintos servidores, reduciendo problemas de latencia y throughput, asi como tambien reduce la carga que recibe cada servidor.

**6- Indique diferencias entre los modelos de Replicación Activa y Pasiva. Indique ventajas de cada uno.** 

(Repasar y ver tema de ventajas de cada uno)

La diferencia entre ellos es que:
- La replicación activa cuenta con un load balancer que se encarga de tomar las request y enviarsela a todos los servidores para que la ejecuten. Luego, los servidores envía la respuesta al load balancer y este se encarga de ver que no haya errores. La ventaja de este es que como todos ejecutan lo mismo, en el caso de que el load balancer detecte que alguna de las respuestas no es la esperada, puede detectar que hubo un error. Tambien garantiza orden Total

- La replicación pasiva cuenta con varios nodos donde uno solo es líder y los demás son followers. La idea es que cuando se quiere ejecutar una request, esta la recibe el lider y la ejecuta. Luego le pasa los cambios de estado a las replicas. En el caso de que se caiga el lider, se elige a una de las replicas como nuevo lider. La ventaja de este es que no hay un único punto de falla como en los demas modelos, lo cual permite seguir ofreciendo servicios ante la caida de nodos.

**7- Explique un algoritmo de consenso para que varios procesos puedan llegar a un acuerdo sobre cierta variable que ellos definen. Indique cuantas replicas son necesarias para garantizar que el algoritmo funcione si: a) se cae 1 proceso, b) se caen 2 procesos**

Uno de los algoritmos de consenso que vimos fue el "Algoritmo de consenso simple". Este algoritmo tiene lo siguiente:
- Todos los procesos tiene un ID unico y tienen una variable donde van a guardar los valores de las rondas y los procesos van proponiendo su valor en las rondas
- En la primera ronda, cada proceso propone su valor 
- Requiere de una función de agregación

Luego se ejecuta durante N rondas lo siguiente: 
- El valor de la ronda siguiente pasa a ser el de la ronda actual
- Se envia a todos los nodos los nuevos valores de la ronda (que no estaban en la ronda anterior) 
- Mientras la ronda esta abierta, el nodo se queda esperando por los valores de los demás procesos. En caso de recibir alguno, a los valores de la siguiente ronda se le hace la Union con los recibidos 

Una vez terminan todas las rondas, los procesos ejecutan la función de agregación para determinar el valor acordado.

La formula de quorum para algoritmos de consenso (suuponiendo que no tolera fallas bizantinas) es de N >= 2F+1, siendo N la cantidad de procesos necesarios y F la cantidad de procesos que fallan. Esta formuala sale de que se necesita una mayoría para ejecutar un algoritmo de consenso.

a) N >= (2 x 1 ) +1 = 3 procesos. Por lo que se necesitan al meneos 3 replicas
b) N >= (2 x 2 ) +1 = 5 procesos. Por lo que se necesitan al meneos 5 replicas



## Preguntas practicas 


**8- Utilizando una arquitectura publisher-subscriber, calcule la suma de todos los enteros incluidos en un vector de tamaño N, optimizando el uso de P procesos conocidos como disponibles. Utilice pseudocódigo o diagrama UML.**

En este caso uso publisher-subscriber por cola, donde hay una cola para obtener resultados y una cola por proceso para enviar los numeros del vector.

Cliente (tiene el vector)

```
# el cliente conoce cantidad de procesos disponibles

def main():

	vector = [1, 5, 84, 3, ...] # no se si puedo asumir que lo tengo o me lo pasan por parametro
	
	conn = innit_connection('broker')
	conn.create_and_subscribe_queue('results')

	for i in P:
		conn.create_queue(f'queue_{i}')

	for i in range(N):
		queue_name = f'queue_{i%P}'
		conn.publish(queue, vector[i])

	for i in range(P):
		conn.publish(f'queue_{i}', 'END')

	sum = 0
	i = 0

	while i != P:

		result = conn.recv_message_from_queue('results')

		sum += results
		i++

	return sum 


```


Process: 

- Cada procesos tiene un ID unico asignado.

```

ID = get_env('ID')
def main():

	conn = init_connection('broker')
	conn.create_and_subscribe_queue(f'queue_{ID}')
	conn.create_queue(f'results')

	acum_sum = 0 

	while not conn.is_closed():

		msg = conn.recv_message_from_queue(f'queue_{ID}')

		if msg == 'END':
			conn.publish('results', acum_sum)
			acum_sum = 0
			
		sum += msg

```

**9- Dado el siguiente diagrama de tiempo, calcule el valor del vector de relojes logicos para cada evento. Utilizando unicamente los vectores de relojes, establezca la relacion entre b y g, entre d y g. Justifique.**

![[Pasted image 20250209144438.png]]

El valor de los vectores son:
- a: (0,1,0)
- b: (0,1,1)
- c: (1,1,1)
- d: (0,1,2)
- f: (2,1,1)
- g: (2,2,1)
- h: (2,2,3)

La relación entre b y g se puede decir que como g es mayor en 2 valores e igual en el tercer campo del vector, entonces se puede decir que b -> g

b: (0,1,1)
g: (2,2,1)

En cuanto a la relación entre d y g como g tiene 2 valores que son mayores al vector de d, pero d tiene un valor mayor al de g, entonces se puede decir que los eventos son concurrentes

- d: (0,1,2)
- g: (2,2,1) 


**10- Diseñe una arquitectura que asegure la escalabilidad para un sistema con los siguientes requisitos:**
- **Recibir usuario, timestamp y cambios de canales realizados en Argentina y Brasil por usuarios que poseen SmartTV de la marca LG y aceptó los terminos y condiciones que incluyen monitoreo de uso.**
- **Consultar la cantidad de usuarios actualmente sintonizando cierto canal** 
- **Consultar los 5 canales más visualizados por usuarios (mayor cant. de minutos) en cierto rango de días.**
- **Consultar la cantidad de usuarios con mas de 1 visualización por día.**
**Resuelva el problema mediante un diagrama de vista física y su explicación.** (hago tambien endpoints y volumen porque normalmente piden eso tambien.)

-- Notas --

- Recibo usuario, timestamp, canal_sintonizado de los TVs LG que aceptaron.
- Consulta de cantidad de usuario por canal: endpoint, y debo poder calcular la cantidad por canal 
- Consulta de top 5 canales mas vistos(en cant de minutos) en rango de días. Necestio endpoint, guardar por usuario los timestamps (si se apaga la tele, como calculo los minutos?) y proceso que calcule por canal los minutos y otro que saque el top 5 
- Cosulta cant usuarios con +1 visualizacion por día (más de 2 canales vistos?). Endpoint, filtrar por usuarios que tengan m'as de 1 y luego contarlos.

-- Resolución 

**Analisis de endpoint**

POST /sintonización

body = {usuario, timestamp, canal}

return status 200 

GET /canales/:canal 

return status 200, body = {cantidad: int}

GET canales/mas_vistos

body = {rango_de_días}

return status 200, body = {[canal1, canal2, canal3, canal4, canal5]}

GET /usuarios/cantidad_visualizacion

body = {}

return status 200, body = {cantidad_usuarios: int}


-- Análisis de volumen -- 

En la DB me tengo que guardar lo siguiente:
- Usuario -> 200 Bytes
- Timestamp -> 50 Bytes
- Canal -> 100 Bytes

El volumen de esto es del orden de 1/2 KB. 

Como es en argentina y brasil, hay aproximadamente un total de 300 millones de personas. Como no se de cuanto es el porcentaje de personas que tiene SmarTV LG, asumo que el 50% solo tiene.

Tambien asumo que por día, cada persona cambia de canal 5 veces. Entonces, por día se debe guardar:

150.000.000 x 5 x 1/2 KB =  75.000.000 x 5 x 1KB = 375.000.000 KB = 375 GB

En un mes guardo alrededor de: 375 x 30 = .(150 + 2100 + 9000) =~ 11.000 GB = 11 TB

Como no dice nada, asumo que debo guardar los datos indefinidamente, por lo que no es muy rentable ya que el volumen de almacenamiento es muy grande


-- Análisis físico --

Como es en argentina y brasil, debo tener repetidores de información en cada provincia/estado que se encarguen de calcular los minutos dado el timestamp y enviarlos al servidor central 

[Diagrama fisico de repetidores y server central]

[Diagrama de robustez]

En esta caso se tienen 4 endpoints, 1 para que los tvs suban la información de los canales sintonizados para cada usuario, y otros 3 para as consultas en el servidor central. 

Los repetidores de información se usan para reducir las latencias debido a las grandes distancias entre los países

La DB esta particionada por timestamp para hacer más rápidas las consultas debido a que todas requieren del tiempo ya sea los sintonizados actuales, los canales con m´s visitas en un rango de fechas y la cantidad de usuarios con x visualizaciones en el día.

Para obtener la cantidad de usuarios sintonizando un canal, se toman los registro de la DB y se obtiene los usuarios activos de un canal x y luego se calcula cuantos hay en ese canal.

Por otro lado, para obtener los canales con más visitas en determinados días, primero filtro por el rango de días a los registros de la DB, calculo los minutos que se pasaron en cada canal y por ultimo calculo un TOP 5 en base a la cantidad de minutos

Por último, se tiene el endpoint de "usuarios activos" que son los qeu sintonizaron m´sa de 1 canal en el día. Para esto primero obtengo los registro del día de la DB, luego cento por cada user la cnatidad de canales sintonizados, filtro por los que tienen más de 1 y luego cuento la cantidad total de usuarios.

### Escuela Colombiana de Ingeniería
### Arquitecturas de Software - ARSW

## Escalamiento en Azure con Maquinas Virtuales, Sacale Sets y Service Plans

### Dependencias
* Cree una cuenta gratuita dentro de Azure. Para hacerlo puede guiarse de esta [documentación](https://azure.microsoft.com/en-us/free/search/?&ef_id=Cj0KCQiA2ITuBRDkARIsAMK9Q7MuvuTqIfK15LWfaM7bLL_QsBbC5XhJJezUbcfx-qAnfPjH568chTMaAkAsEALw_wcB:G:s&OCID=AID2000068_SEM_alOkB9ZE&MarinID=alOkB9ZE_368060503322_%2Bazure_b_c__79187603991_kwd-23159435208&lnkd=Google_Azure_Brand&dclid=CjgKEAiA2ITuBRDchty8lqPlzS4SJAC3x4k1mAxU7XNhWdOSESfffUnMNjLWcAIuikQnj3C4U8xRG_D_BwE). Al hacerlo usted contará con $200 USD para gastar durante 1 mes.

### Parte 0 - Entendiendo el escenario de calidad

Adjunto a este laboratorio usted podrá encontrar una aplicación totalmente desarrollada que tiene como objetivo calcular el enésimo valor de la secuencia de Fibonnaci.

**Escalabilidad**
Cuando un conjunto de usuarios consulta un enésimo número (superior a 1000000) de la secuencia de Fibonacci de forma concurrente y el sistema se encuentra bajo condiciones normales de operación, todas las peticiones deben ser respondidas y el consumo de CPU del sistema no puede superar el 70%.

### Parte 1 - Escalabilidad vertical

1. Diríjase a el [Portal de Azure](https://portal.azure.com/) y a continuación cree una maquina virtual con las características básicas descritas en la imágen 1 y que corresponden a las siguientes:
    * Resource Group = SCALABILITY_LAB
    * Virtual machine name = VERTICAL-SCALABILITY
    * Image = Ubuntu Server 
    * Size = Standard B1ls
    * Username = scalability_lab
    * SSH publi key = Su llave ssh publica

![Imágen 1](images/part1/part1-vm-basic-config.png)

2. Para conectarse a la VM use el siguiente comando, donde las `x` las debe remplazar por la IP de su propia VM.

    `ssh scalability_lab@xxx.xxx.xxx.xxx`

3. Instale node, para ello siga la sección *Installing Node.js and npm using NVM* que encontrará en este [enlace](https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/).
4. Para instalar la aplicación adjunta al Laboratorio, suba la carpeta `FibonacciApp` a un repositorio al cual tenga acceso y ejecute estos comandos dentro de la VM:

    `git clone <your_repo>`

    `cd <your_repo>/FibonacciApp`

    `npm install`

5. Para ejecutar la aplicación puede usar el comando `npm FibinacciApp.js`, sin embargo una vez pierda la conexión ssh la aplicación dejará de funcionar. Para evitar ese compartamiento usaremos *forever*. Ejecute los siguientes comando dentro de la VM.

    `npm install forever -g`

    `forever start FibinacciApp.js`

6. Antes de verificar si el endpoint funciona, en Azure vaya a la sección de *Networking* y cree una *Inbound port rule* tal como se muestra en la imágen. Para verificar que la aplicación funciona, use un browser y user el endpoint `http://xxx.xxx.xxx.xxx:3000/fibonacci/6`. La respuesta debe ser `The answer is 8`.

![](images/part1/part1-vm-3000InboudRule.png)

7. La función que calcula en enésimo número de la secuencia de Fibonacci está muy mal construido y consume bastante CPU para obtener la respuesta. Usando la consola del Browser documente los tiempos de respuesta para dicho endpoint usando los siguintes valores:
    * 1000000
    * 1010000
    * 1020000
    * 1030000
    * 1040000
    * 1050000
    * 1060000
    * 1070000
    * 1080000
    * 1090000    

8. Dírijase ahora a Azure y verifique el consumo de CPU para la VM. (Los resultados pueden tardar 5 minutos en aparecer).

![Imágen 2](images/part1/part1-vm-cpu.png)

9. Ahora usaremos Postman para simular una carga concurrente a nuestro sistema. Siga estos pasos.
    * Instale newman con el comando `npm install newman -g`. Para conocer más de Newman consulte el siguiente [enlace](https://learning.getpostman.com/docs/postman/collection-runs/command-line-integration-with-newman/).
    * Diríjase hasta la ruta `FibonacciApp/postman` en una maquina diferente a la VM.
    * Para el archivo `[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambie el valor del parámetro `VM1` para que coincida con la IP de su VM.
    * Ejecute el siguiente comando.

    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```

10. La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure diríjase a la sección *size* y a continuación seleccione el tamaño `B2ms`.

![Imágen 3](images/part1/part1-vm-resize.png)

11. Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9.
12. Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo.
13. Vuelva a dejar la VM en el tamaño inicial para evitar cobros adicionales.

**Preguntas**

1. ¿Cuántos y cuáles recursos crea Azure junto con la VM?
	
	Azure crea 6 recursos junto con la VM los cuales son:
		- Red Virtual.
		- Disco.
		- Dirección IP Pública.
		- Interfaz de Red.
		- Grupo de Seguridad de Red.
		- Cuenta de almacenamiento.
	
2. ¿Brevemente describa para qué sirve cada recurso?
	
	- *Red Virtual:*  es el bloque de creación fundamental de una red privada en Azure. VNet permite muchos tipos de recursos de Azure, como Azure Virtual Machines (máquinas virtuales), para comunicarse de forma segura entre usuarios, con Internet y con las redes locales.
	
	- *Disco:* los discos administrados de Azure son volúmenes de almacenamiento de nivel de bloque que administra Azure y que se usan con Azure Virtual Machines.

	- *Dirección IP Pública:* se usan para la comunicación entrante y saliente [sin traducción de direcciones de red (NAT)] con Internet y otros recursos de Azure no conectados a una red virtual. La asignación de una dirección IP pública a una NIC es opcional.
	
	- *Interfaz de Red:* es la interconexión entre una máquina virtual y una red virtual (VNet). Una máquina virtual debe tener al menos una NIC, pero puede tener varias,
		   en función del tamaño de la máquina virtual que se cree. Obtenga información sobre cuántas NIC admite cada tamaño de máquina virtual, consulte Tamaño de máquina virtual.
	
	- *Grupo de Seguridad de Red:* contiene una lista de reglas de la lista de control de acceso (ACL) que permiten o deniegan el tráfico de red a subredes, NIC, o ambas.
	
	- *Cuenta de almacenamiento:* una cuenta de Azure Storage contiene todos los objetos de datos de Azure Storage: blobs, archivos, colas, tablas y discos. La cuenta de almacenamiento proporciona un espacio de nombres único para los datos de Azure Storage que es accesible desde cualquier lugar del mundo a través de HTTP o HTTPS.
		
3. ¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`? ¿Por qué debemos crear un *Inbound port rule* antes de acceder al servicio?
	
	* Porque cuando se cierra la conexión de SSH, todos los servicios que esten corriendo en ese momento de manera remota se terminarán en ese momento exacto.
	
	* Se crea porque al abrirse un nuevo puerto (3000) permitimos que asi se cierre o se caiga la conexión SSH. De igual manera se siga ejecutando el servicio y pase el trafico por ese puerto.
	
4. Adjunte tabla de tiempos e interprete por qué la función tarda tando tiempo.

## Tabla
| N | B1ls(seg)| B2ms(seg)|
|---------|--------|--------|
| 1000000 | 29.61 | 19.19 |
| 1010000 | 28.00 | 19.62 |
| 1020000 | 25.13 | 20.41 |
| 1030000 | 34.33 | 20.96 |
| 1040000 | 23.82 | 20.80 |
| 1050000 | 24.51 | 21.19 |
| 1060000 | 25.91 | 21.43 |
| 1070000 | 25.88 | 21.60 |
| 1080000 | 26.21 | 22.65 |
| 1090000 | 27.04 | 23.25 |


	
5. Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.

	* B1ls
	
		![image](https://github.com/csarssj/ARSW-LAB-8/blob/main/images/part1/CPU1.png)
		
	* B2ms
	
		![image](https://github.com/csarssj/ARSW-LAB-8/blob/main/images/part1/CPU2.png)
		
	Consume una gran cantidad de la CPU con diferentes picos. Ya  que se ejecutan muchas operaciones en un tiempo corto y al existir poca capacidad produce un consumo alto de la CPU con una mala practica se recurre a repetir iteraciones por lo que conlleva el mismo procedimiento varias veces.
	Y al agregarse más capacidad a la máquina, los tiempos y el consumo se ve disminuido. Sin embargo al aumentar se evidencia la misma problematica de repetición de operacions e/o iteraciones.

6. Adjunte la imagen del resumen de la ejecución de Postman. Interprete:
    * Tiempos de ejecución de cada petición.
    
		* B1ls
		
		![image](https://github.com/csarssj/ARSW-LAB-8/blob/main/images/part1/postman1.png)
		
		* B2ms
			
		![image](https://github.com/csarssj/ARSW-LAB-8/blob/main/images/part1/postman2.png)
		
    * Si hubo fallos documentelos y explique.
	
		Si existieron errores debido a las grandes cantidades de iteraciones al intentar acceder al recurso en un mismo instante. Lo que demuestra la poca eficiencia del programa y produce fallas.
		Se ve una leve mejoria en los tiempos con la mayor capacidad, pero los fallos persisten ya que se debe a un problema más de concurrencia.
		
7. ¿Cuál es la diferencia entre los tamaños `B2ms` y `B1ls` (no solo busque especificaciones de infraestructura)?
	
	* Mayor RAM
	* Mayor Almacenamiento
	* Mayor cantidad de discos de datos
	* Mayor cantidad de procesadores virtuales implica mayor potencia.
	
8. ¿Aumentar el tamaño de la VM es una buena solución en este escenario?, ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM?
	
	No considero que sea una buena solución debido a que solo se observa una leve mejora en los tiempos, pero en cuestion de consumo y fallos siguen ocurriendo casi con la misma frecuencia.
	FibinacciApp sigue comportandose de manera similar debido a sus errores de concurrencia, solo es mas rapido porque consume los mismos recursos con un tamaño mayor o menor dependiendo la máquina.
	
9. ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?

	Aumenta la capacidad y el procesamiento de la máquina pero para ello se tiene que reiniciar, lo cual interrumpe el servicio que se ejecuta y no es conveniente en terminos de 
	disponibilidad.
	
10. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?

	Si hubo mejoria pero muy leve, ya que el cambio ofrece un mayor tamaño, almacenamiento y RAM. Lo que implica que se puede ofrecer mayor potencia y mejoria en el procesamiento ante las cantidades de iteraciones que se realizan en el mismo tiempo.

11. Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor?
	
	* ![image](https://github.com/csarssj/ARSW-LAB-8/blob/main/images/part1/15.png)
	
	El comportamiento porcentualmente es casi igual, no se ven muchas diferencias.

### Parte 2 - Escalabilidad horizontal

#### Crear el Balanceador de Carga

Antes de continuar puede eliminar el grupo de recursos anterior para evitar gastos adicionales y realizar la actividad en un grupo de recursos totalmente limpio.

1. El Balanceador de Carga es un recurso fundamental para habilitar la escalabilidad horizontal de nuestro sistema, por eso en este paso cree un balanceador de carga dentro de Azure tal cual como se muestra en la imágen adjunta.

![](images/part2/part2-lb-create.png)

2. A continuación cree un *Backend Pool*, guiese con la siguiente imágen.

![](images/part2/part2-lb-bp-create.png)

3. A continuación cree un *Health Probe*, guiese con la siguiente imágen.

![](images/part2/part2-lb-hp-create.png)

4. A continuación cree un *Load Balancing Rule*, guiese con la siguiente imágen.

![](images/part2/part2-lb-lbr-create.png)

5. Cree una *Virtual Network* dentro del grupo de recursos, guiese con la siguiente imágen.

![](images/part2/part2-vn-create.png)

#### Crear las maquinas virtuales (Nodos)

Ahora vamos a crear 3 VMs (VM1, VM2 y VM3) con direcciones IP públicas standar en 3 diferentes zonas de disponibilidad. Después las agregaremos al balanceador de carga.

1. En la configuración básica de la VM guíese por la siguiente imágen. Es importante que se fije en la "Avaiability Zone", donde la VM1 será 1, la VM2 será 2 y la VM3 será 3.

![](images/part2/part2-vm-create1.png)

2. En la configuración de networking, verifique que se ha seleccionado la *Virtual Network*  y la *Subnet* creadas anteriormente. Adicionalmente asigne una IP pública y no olvide habilitar la redundancia de zona.

![](images/part2/part2-vm-create2.png)

3. Para el Network Security Group seleccione "avanzado" y realice la siguiente configuración. No olvide crear un *Inbound Rule*, en el cual habilite el tráfico por el puerto 3000. Cuando cree la VM2 y la VM3, no necesita volver a crear el *Network Security Group*, sino que puede seleccionar el anteriormente creado.

![](images/part2/part2-vm-create3.png)

4. Ahora asignaremos esta VM a nuestro balanceador de carga, para ello siga la configuración de la siguiente imágen.

![](images/part2/part2-vm-create4.png)

5. Finalmente debemos instalar la aplicación de Fibonacci en la VM. para ello puede ejecutar el conjunto de los siguientes comandos, cambiando el nombre de la VM por el correcto

```
git clone https://github.com/daprieto1/ARSW_LOAD-BALANCING_AZURE.git

curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash
source /home/vm1/.bashrc
nvm install node

cd ARSW_LOAD-BALANCING_AZURE/FibonacciApp
npm install

npm install forever -g
forever start FibonacciApp.js
```

Realice este proceso para las 3 VMs, por ahora lo haremos a mano una por una, sin embargo es importante que usted sepa que existen herramientas para aumatizar este proceso, entre ellas encontramos Azure Resource Manager, OsDisk Images, Terraform con Vagrant y Paker, Puppet, Ansible entre otras.

#### Probar el resultado final de nuestra infraestructura

1. Porsupuesto el endpoint de acceso a nuestro sistema será la IP pública del balanceador de carga, primero verifiquemos que los servicios básicos están funcionando, consuma los siguientes recursos:

```
http://52.155.223.248/
http://52.155.223.248/fibonacci/1
```

* ![image](https://github.com/csarssj/ARSW-LAB-8/blob/main/images/part2/VMALL.png)

*Todas las maquinas y el balanceador de carga funcionando correctamente*

2. Realice las pruebas de carga con `newman` que se realizaron en la parte 1 y haga un informe comparativo donde contraste: tiempos de respuesta, cantidad de peticiones respondidas con éxito, costos de las 2 infraestrucruras, es decir, la que desarrollamos con balanceo de carga horizontal y la que se hizo con una maquina virtual escalada.

	* Parte 1:
	
		![image](https://github.com/csarssj/ARSW-LAB-8/blob/main/images/part1/postman1.png)
		
	* Parte 2:
	
		![image](https://github.com/csarssj/ARSW-LAB-8/blob/main/images/part2/newman2.png)
		
	Se ve la comparación de entre las dos infraestructuras y vemos que en relación beneficio/costo comenzamos a notar que la escalamiento horizontal es mejor, resultando en 10/10 peticiones exitosas y con mejor costo que la vertical.

3. Agregue una 4 maquina virtual y realice las pruebas de newman, pero esta vez no lance 2 peticiones en paralelo, sino que incrementelo a 4. Haga un informe donde presente el comportamiento de la CPU de las 4 VM y explique porque la tasa de éxito de las peticiones aumento con este estilo de escalabilidad.

```
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
```
* VM1:
	
	![image](https://github.com/csarssj/ARSW-LAB-8/blob/main/images/part2/vm1.png)
		
* VM2:
	
	![image](https://github.com/csarssj/ARSW-LAB-8/blob/main/images/part2/vm2.png)
		
* VM3:
	
	![image](https://github.com/csarssj/ARSW-LAB-8/blob/main/images/part2/vm3.png)
		
* VM4:
	
	![image](https://github.com/csarssj/ARSW-LAB-8/blob/main/images/part2/vm4.png)

**Preguntas**

* ¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian?, ¿Qué es SKU, qué tipos hay y en qué se diferencian?, ¿Por qué el balanceador de carga necesita una IP pública?
	
	- *Tipos balanceadores:* 
	
		- Un equilibrador de carga público puede proporcionar conexiones de salida para máquinas virtuales dentro de la red virtual. Estas conexiones se realizan mediante la traducción de sus direcciones IP privadas a direcciones IP públicas. Las instancias públicas de Load Balancer se usan para equilibrar la carga del tráfico de Internet en las máquinas virtuales.
					
		- Un equilibrador de carga interno (o privado) se usa cuando se necesitan direcciones IP privadas solo en el front-end. Los equilibradores de carga internos se usan para equilibrar la carga del tráfico dentro de una red virtual. También se puede acceder a un servidor front-end del equilibrador de carga desde una red local en un escenario híbrido.
	
	- *SKU:* Load balancer admite SKU estándar y básicas. Estas SKU difieren en la escala, las características y los precios del escenario. Cualquier escenario que sea posible con Basic Load Balancer se puede crear con Standard Load Balancer.
	
	- *IP pública:* Para obtener acceso a la aplicación en Internet, necesita una dirección IP pública para el equilibrador de carga.

* ¿Cuál es el propósito del *Backend Pool*?

	- *Backend Pool:* El grupo de máquinas virtuales o instancias de un conjunto de escalado de máquinas virtuales que van a atender la solicitud entrante.
	
* ¿Cuál es el propósito del *Health Probe*?

	- *Health Probe:* se usan para determinar el estado de mantenimiento de las instancias del grupo de back-end. Durante la creación del equilibrador de carga, configure un sondeo de estado para que lo use el equilibrador de carga. Este sondeo de estado determinará si una instancia está en buen estado y puede recibir tráfico.
	
* ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?.
	
	- *Load Balancing Rule:*  se usan para definir cómo se distribuye el tráfico entrante a todas las instancias del grupo de back-end. Las reglas de equilibrio de carga asignan una configuración de dirección IP de front-end y un puerto dados a varios puertos y direcciones IP de back-end.
	
	- *None (hash-based):* Especifica que las solicitudes sucesivas del mismo cliente pueden ser manejadas por cualquier máquina virtual.
	
    - *Client IP (source IP affinity 2-tuple):* Especifica que las peticiones sucesivas de la misma dirección IP del cliente serán gestionadas 	por la misma máquina virtual.
	
    - *Client IP and Protocol (Source IP affinity 3-tuple):* Especifica que las solicitudes sucesivas de la misma combinación de dirección IP de cliente y protocolo serán tratadas por la misma máquina virtual.

* ¿Qué es una *Virtual Network*? ¿Qué es una *Subnet*? ¿Para qué sirven los *address space* y *address range*?
	
	- *Virtual Network:* es el bloque de creación fundamental de una red privada en Azure. VNet permite muchos tipos de recursos de Azure, como Azure Virtual Machines (máquinas virtuales), para comunicarse de forma segura entre usuarios, con Internet y con las redes locales.
	
	- *Subnet:* Las subredes le permiten segmentar la red virtual en una o varias subredes y asignar una parte del espacio de direcciones de la red virtual para cada subred.
	
	- *address space:* tiene que especificar un espacio de direcciones IP privado personalizado mediante direcciones públicas y privadas, Azure asigna a los recursos de una red virtual una dirección IP privada desde el espacio de direcciones que asigne.
	
	- *address range:* El rango de direcciones que defina puede ser público o privado (RFC 1918). Ya sea que defina el rango de direcciones como público o privado, solo se puede acceder al rango de direcciones desde la red virtual, desde redes virtuales interconectadas y desde cualquier red local que haya conectado a la red virtual.

* ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas?. ¿Qué significa que una IP sea *zone-redundant*?

	- *Availability Zone*. Una zona de disponibilidad es una zona separada físicamente dentro de una región de Azure. Hay tres zonas de disponibilidad por cada región de Azure admitida.
	
	- *Zonas*. Cada zona de disponibilidad tiene una fuente de alimentación, una red y un sistema de refrigeración distintos. Si diseña las soluciones para que utilicen máquinas virtuales replicadas en zonas, podrá proteger sus datos y aplicaciones frente a la pérdida de un centro de datos.
	
	- *zone-redundant* Es una dirección IP-Pública que si se produce un error en una región, el tráfico se enruta al siguiente equilibrador de carga regional correcto más cercano.

* ¿Cuál es el propósito del *Network Security Group*?

	- *Network Security Group:* para filtrar el tráfico de red hacia y desde los recursos de Azure de una red virtual de Azure. Un grupo de seguridad de red contiene reglas de seguridad que permiten o deniegan el tráfico de red entrante o el tráfico de red saliente de varios tipos de recursos de Azure. Para cada regla, puede especificar un origen y destino, un puerto y un protocolo.
	
* Informe de newman 1 (Punto 2)
	*SOLUCIONADO ARRIBA*
* Presente el Diagrama de Despliegue de la solución.

	![image](https://github.com/csarssj/ARSW-LAB-8/blob/main/images/part2/despliegue.png)





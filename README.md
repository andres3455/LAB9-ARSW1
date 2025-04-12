### Escuela Colombiana de Ingeniería
### Arquitecturas de Software - ARSW

## Escalamiento en Azure con Maquinas Virtuales, Sacale Sets y Service Plans

## Integrantes:
* Santiago Guerra Penagos
* Andrés Felipe Rodríguez

### Dependencias
* Cree una cuenta gratuita dentro de Azure. Para hacerlo puede guiarse de esta [documentación](https://azure.microsoft.com/es-es/free/students/). Al hacerlo usted contará con $100 USD para gastar durante 12 meses.

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

2. Para conectarse a la VM use el siguiente comando, donde las `x` las debe remplazar por la IP de su propia VM (Revise la sección "Connect" de la virtual machine creada para tener una guía más detallada).

    `ssh scalability_lab@xxx.xxx.xxx.xxx`

3. Instale node, para ello siga la sección *Installing Node.js and npm using NVM* que encontrará en este [enlace](https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/).
4. Para instalar la aplicación adjunta al Laboratorio, suba la carpeta `FibonacciApp` a un repositorio al cual tenga acceso y ejecute estos comandos dentro de la VM:

    `git clone <your_repo>`

    `cd <your_repo>/FibonacciApp`

    `npm install`

5. Para ejecutar la aplicación puede usar el comando `npm FibinacciApp.js`, sin embargo una vez pierda la conexión ssh la aplicación dejará de funcionar. Para evitar ese compartamiento usaremos *forever*. Ejecute los siguientes comando dentro de la VM.

    ` node FibonacciApp.js`

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

### Desarrollo Lab

* Prueba del endpoint Answer 8

![Screenshot 2025-04-10 081634](https://github.com/user-attachments/assets/014a1b59-1ce8-43a0-9fba-cb9ce78ad0b8)

* 1000000
  ![Screenshot 2025-04-10 082204](https://github.com/user-attachments/assets/a6ad83da-35eb-4ccf-8dde-197fafcb3aa4)

* 1010000
  ![Screenshot 2025-04-10 082342](https://github.com/user-attachments/assets/80936f2a-debb-48a3-b9aa-b057081ef290)

* 1020000

  ![Screenshot 2025-04-10 082407](https://github.com/user-attachments/assets/44b0f991-dd71-4868-89f4-6807d9f1c710)
* 1030000

  ![Screenshot 2025-04-10 082430](https://github.com/user-attachments/assets/5e0c54fa-153d-40f9-b282-f8d9fbe05cf6)

* 1040000

  ![Screenshot 2025-04-10 082455](https://github.com/user-attachments/assets/20447fc7-77f7-4a85-8592-6d160bc9e053)

* 1050000

  ![Screenshot 2025-04-10 082614](https://github.com/user-attachments/assets/dc3f3056-8771-4f39-b31b-3e74588de018)

* 1060000

  ![Screenshot 2025-04-10 082642](https://github.com/user-attachments/assets/66583e66-4b47-4577-885c-87986119dfc6)

* 1070000

  ![Screenshot 2025-04-10 082937](https://github.com/user-attachments/assets/580ec980-2783-42de-8b43-413572c3fbbe)


* 1080000

  ![Screenshot 2025-04-10 082957](https://github.com/user-attachments/assets/f39afb59-388b-49c5-b9a8-c31617068130)

* 1090000

  ![Screenshot 2025-04-10 083355](https://github.com/user-attachments/assets/a5488084-7317-42b1-baf2-144a294b956c)


8. Dírijase ahora a Azure y verifique el consumo de CPU para la VM. (Los resultados pueden tardar 5 minutos en aparecer).

![Imágen 2](images/part1/part1-vm-cpu.png)

## Consumo en Azure

![Screenshot 2025-04-10 085320](https://github.com/user-attachments/assets/d4ba6889-08bb-4dc2-a582-1b47599f0c7e)



9. Ahora usaremos Postman para simular una carga concurrente a nuestro sistema. Siga estos pasos.
    * Instale newman con el comando `npm install newman -g`. Para conocer más de Newman consulte el siguiente [enlace](https://learning.getpostman.com/docs/postman/collection-runs/command-line-integration-with-newman/).
    * Diríjase hasta la ruta `FibonacciApp/postman` en una maquina diferente a la VM.
    * Para el archivo `[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambie el valor del parámetro `VM1` para que coincida con la IP de su VM.
    * Ejecute el siguiente comando.

    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```
   ![](images/part1/newman.png)

   Después de probar, el servicio falla muchas veces:
   ![](images/part1/newman%20results.png) 
10. La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure diríjase a la sección *size* y a continuación seleccione el tamaño `B2ms`.

![Imágen 3](images/part1/part1-vm-resize.png)

11. Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9.
12. Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo.
* ![image](https://github.com/user-attachments/assets/0a3e5604-2521-4358-af26-d5f35966fdda)

14. Vuelva a dejar la VM en el tamaño inicial para evitar cobros adicionales.

**Preguntas**

1. ¿Cuántos y cuáles recursos crea Azure junto con la VM?
* Respuesta: Aproximadamente 7 Recursos
  ![image](https://github.com/user-attachments/assets/4be9fb55-f7b2-4390-9772-cbdd7f608c52)

2. ¿Brevemente describa para qué sirve cada recurso?
* Respuesta:
  * Maquina Virtual: Es el recurso principal: la máquina virtual en sí. Aquí se ejecuta el sistema operativo (Linux, en este caso) y es donde nos conectamos vía SSH.
  * Dirección Ip publica: Es la IP pública que nos permite conectar remotamente desde un equipo local a la VM a través de Internet.
  * Grupo de Seguridad: Es un firewall a nivel de red que controla el tráfico de entrada y salida. Define qué puertos están abiertos.
  * Red Virtual:Es la red interna en la que se encuentra la máquina virtual. Sirve para conectar recursos entre sí dentro de Azure de forma segura
  * Interfaz de red:Es la tarjeta de red virtual que conecta la VM con la red virtual (VNet) y con la IP pública, sin esta tarjeta, no se puede comunicarse con el exterior
  * Clave SSH: Es la clave pública almacenada en Azure que se usó para crear la máquina virtual. La clave privada es la que descargamos (.pem).
  * Disco: Es el disco duro virtual donde está instalado el sistema operativo y los archivos de tu VM.
    
3. ¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`? ¿Por qué debemos crear un *Inbound port rule* antes de acceder al servicio?
* Cuando cerramos la terminal o nos desconectamos del SSH), todos los procesos ligados al acceso Node.js se terminan. Debemos crear una regla de entrada, por que por defecto azure bloquea las peticiones entrantes a menos que se lo permitamos
4. Adjunte tabla de tiempos e interprete por qué la función tarda tando tiempo.
  
| Valor calculado (n) | Tiempo (s) | 
|---------------------|------------|
| 1,000,000           | 6.77       | 
| 1,010,000           | 6.90       | 
| 1,020,000           | 7.02       | 
| 1,030,000           | 7.12       | 
| 1,040,000           | 7.28       | 
| 1,050,000           | 7.38       | 
| 1,060,000           | 7.73       | 
| 1,070,000           | 12.48      | 
| 1,080,000           | 9.94       |
| 1,090,000           | 8.07       |

A pesar del incremento gradual en el tiempo de ejecución, el comportamiento general sugiere que la función utilizada tiene una complejidad lineal en estos valores de entrada. Sin embargo, el pico en 1,070,000 muestra que el rendimiento puede verse afectado por factores externos, como el uso de CPU o memoria de la máquina virtual, lo cual debe considerarse al desplegar aplicaciones de alto rendimiento.

5. Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.
* ![Screenshot 2025-04-10 085320](https://github.com/user-attachments/assets/d4ba6889-08bb-4dc2-a582-1b47599f0c7e)
  
6. Adjunte la imagen del resumen de la ejecución de Postman. Interprete:
    * Tiempos de ejecución de cada petición.
    * Si hubo fallos documentelos y explique.
## GUERRA
7. ¿Cuál es la diferencia entre los tamaños `B2ms` y `B1ls` (no solo busque especificaciones de infraestructura)?
* Respuesta: El tamaño de B1LS es ideal para tareas ligeras y financieras, como servidores de desarrollo o pruebas básicas, ya que tiene recursos limitados (1 VCPU, 0.5 GIB Memoria) y bajo costo ($ 3.80 por mes). Por otro lado, el tamaño B2MS ofrece una mayor capacidad (2 VCPU, 8 Memoria GIB) a un precio más alto ($ 58.40/mes), que es más adecuado para pequeñas aplicaciones web, bases de datoso que requieren más flexibilidad y rendimiento.
  
8. ¿Aumentar el tamaño de la VM es una buena solución en este escenario?, ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM?
* Respuesta: El análisis previo muestra que el escalamienton vertical es una solución efectiva: aunque el tamaño de B1LS casi alcanzó su capacidad máxima en varias aplicaciones, usando el tamaño de B2MS, el uso de CPU permaneció por debajo del 50%, alcanzando un porcentaje del 40%. Sin embargo, también es importante optimizar el algoritmo utilizado para mejorar la eficiencia general de la aplicación.

9. ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?
* Respuesta: El precio aumenta considerablemente, y se le delegan mas ram para los procesos
12. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?
  En los tiempos de respuesta, se alcanzo una reducción de aproximadamente un segundo, mientras que el consumo de CPU, aumento de una 30% a un 40 %
13. Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor?
  ## GUERRA

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

2. Realice las pruebas de carga con `newman` que se realizaron en la parte 1 y haga un informe comparativo donde contraste: tiempos de respuesta, cantidad de peticiones respondidas con éxito, costos de las 2 infraestrucruras, es decir, la que desarrollamos con balanceo de carga horizontal y la que se hizo con una maquina virtual escalada.

3. Agregue una 4 maquina virtual y realice las pruebas de newman, pero esta vez no lance 2 peticiones en paralelo, sino que incrementelo a 4. Haga un informe donde presente el comportamiento de la CPU de las 4 VM y explique porque la tasa de éxito de las peticiones aumento con este estilo de escalabilidad.

```
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
```

**Preguntas**

* ¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian?, ¿Qué es SKU, qué tipos hay y en qué se diferencian?, ¿Por qué el balanceador de carga necesita una IP pública?
* ¿Cuál es el propósito del *Backend Pool*?
* ¿Cuál es el propósito del *Health Probe*?
* ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?.
* ¿Qué es una *Virtual Network*? ¿Qué es una *Subnet*? ¿Para qué sirven los *address space* y *address range*?
* ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas?. ¿Qué significa que una IP sea *zone-redundant*?
* ¿Cuál es el propósito del *Network Security Group*?
* Informe de newman 1 (Punto 2)
* Presente el Diagrama de Despliegue de la solución.





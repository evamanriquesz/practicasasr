# PRACTICA 1 - MÁQUINAS VIRTUALES 

En esta práctica se van a llevar a cabo diferentes formas para conectar una máquina a Internet de manera segura. 

## SOLUCIÓN 1 - CREACIÓN DE MÁQUINA DE SALTO 

Para esta solución, se han de crear dos máquinas virtuales: el servidor web y el servidor de salto, que será a través del cual nos conectaremos al servidor web. Ambas contarán con IP privada y pública. Asimismo, será necesario crear una subred (vpc-prac2) a la cual se conectarán ambos servidores para estar relacionados entre sí. Por otro lado, también será necesario crear tres reglas de firewall para controlar el tráfico entre servidores y con internet. La primera regla permitirá a la máquina de salto conectarse con la otra máquina, y comunicarse a través del puerto TCP 22. La segunda regla hará referencia a la comunicación del servidor web por los puertos 80 y 443, así como a la otra máquina. Por último, la tercera máquina, deberá permitir el tráfico del servidor de salto al servidor web, y además, la comunicación por el puerto 22. 

Posteriormente, se ha de habilitar SSH tanto para la máquina de salto desde el ordenador, como para la máquina web desde la máquina de salto. Utilizando SSH, se debe instalar NGINX en el servidor web. 

#### Creación máquinas virtuales

![image](https://github.com/evamanriquesz/practicasasr/assets/91720934/06ec0c7f-b047-4944-9fbe-bcd283fad344)

#### Creación reglas firewall 

![image](https://github.com/evamanriquesz/practicasasr/assets/91720934/42e826c1-3a7e-42ae-9fb5-d18a45512fc7)

#### Creación claves SSH

 ![image](https://github.com/evamanriquesz/practicasasr/assets/91720934/dff71ce0-a285-46b7-8fa8-469420cfa2ea)

#### Prueba descarga NGINX y funcionamiento NGINX y SSH

![image](https://github.com/evamanriquesz/practicasasr/assets/91720934/eced59f5-fd52-4afa-ac53-a3137de3798e)

![image](https://github.com/evamanriquesz/practicasasr/assets/91720934/c6ed2656-6571-4101-94c7-bd6ed9deacbe)

## SOLUCIÓN 2 - INTRODUCCIÓN A LOS WAF - WEB APPLICATION FIREWALL (FIREWALL CAPA 7) 

En esta solución, se han de crear dos nuevas máquinas: de salto y servidor. La primera tendrá IP pública y privada, mientras que la segunda, únicamente tendrá IP pública. Así mismo, se deberá montar un balanceador de cargas de capa 7 con servicio de WAF haciendo HTTPS offloading para comunicar las máquinas con el exterior. Este balanceador recibirá peticiones por el puert o443 y las convertirá al puerto 80. Además, también se ha de restringir el tráfico únicamente a España, así como proteger nuestra máquina de SQL Injection y XSS. 

#### Creación máquinas virtuales 

![image](https://github.com/evamanriquesz/practicasasr/assets/91720934/484ac4e3-7e25-41fe-a454-b4bc4849828f)
![image](https://github.com/evamanriquesz/practicasasr/assets/91720934/7ff4e75a-72d5-4dd8-988a-544ceaa68bd9)

Como se ha mencionado previamente, el servidor web no cuenta con IP pública, pero, en cambio, pertenece a un grupo de instancias que le permitirá acceder a NGINX. Este es uno de los cambios necesarios respecto al anterior apartado. 

#### Creación reglas firewall 

- Regla 1: Para la comunicación entre el PC y el servidor de salto, mediante TCP puerto 22.
- Regla 3: Para la comunicación entre el servidor de salto y el servidor web, mediante TCP puerto 22.
- Regla 4: Para permitir el tráfico de Google en el servidor de salto, mediante TCP puerto 80.

![image](https://github.com/evamanriquesz/practicasasr/assets/91720934/e00f110d-3452-4f21-9fe3-39c21ed53698)
![image](https://github.com/evamanriquesz/practicasasr/assets/91720934/634d144a-6501-496e-b089-31108de154c0)

#### Configuración Load Balancer L7 

En primer lugar, para la configuración del balanceador de cargas de capa 7, es necesario crear un certificado mediante OpenSSL y firmarlo utilizando CRT. Puesto que la firma será realizada por mi, más adelante se considerará la conexión insegura. A continuación, se configura el front y el back end. Para el backend se creará un grupo de instancias sobre las que aplica. En este caso únicamente aplicará sobre el servidor web, pero esto es una buena práctica para añadir más máquinas en un futuro. Por último, se deberán implementar dos health checks, uno para asegurar que el servidor está operativo, haciendo una llamada TCP al puerto 80; y otro para asegurar la conexión del servidor de salto con el rango de IPs de Google. 

![image](https://github.com/evamanriquesz/practicasasr/assets/91720934/cfbfa9d7-69e9-44e7-b95c-ec0f4071fd2b)

![image](https://github.com/evamanriquesz/practicasasr/assets/91720934/548df74a-3405-4d89-8541-54a0daf8d4ed)

Todo este punto en sí difiere con lo realizado en el apartado anterior, donde la conexión se realizaba directamente a internet, sin pasar por este tipo de intermediarios. 

#### Configuración SSH y Cloud NAT

![image](https://github.com/evamanriquesz/practicasasr/assets/91720934/8ce88d54-aafc-403d-af51-55e8d40979c8)

Tras esta configuración, es posible acceder a las máquinas por ssh: 
![image](https://github.com/evamanriquesz/practicasasr/assets/91720934/66cd8400-b062-4038-9a3b-ec5c64297092)
![image](https://github.com/evamanriquesz/practicasasr/assets/91720934/4f3e9520-de89-47ec-9c75-a2e705264f12)

En cambio, esto no es suficiente para poder descargar NGINX, ya que la máquina server no tiene conexión a internet. Se habilita el Cloud NAT para poder hacerlo. Este es otro cambio introducido respecto al apartado 1, en el cual la máquina web tenia acceso a internet y no era necesario el paso intermedio de Cloud NAT. 

![image](https://github.com/evamanriquesz/practicasasr/assets/91720934/da8144c2-c6a6-4913-9761-67260188be3e)

#### Implantación WAF - Cloud Armor 

El último requerimiento del apartado consiste en que sólo se permita tráfico de la UE, y que se eviten ataques SQL Injection y XSS. Para ello, se crea una política de seguridad Cloud Armor. Cabe destacar que en esta política es importante tener en cuenta el orden de prioridad, siendo el número más bajo, el elemento más importante. 

![image](https://github.com/evamanriquesz/practicasasr/assets/91720934/4a978ab3-eb93-44f9-9207-7b8ff92d2333)

De nuevo, este paso también difiere de lo realizado en el apartado anterior. 

#### Resultado final  

Tras la implementación de los pasos previos, se realiza la conexión por SSH del PC a la máquina de salto, y de la máquina de salto a la máquina web, para instalar NGINX con el comando: sudo apt -y install nginx

Para verificar que se ha descargado se ejecuta el comando  curl localhost: 

![image](https://github.com/evamanriquesz/practicasasr/assets/91720934/d10288c4-ff77-41cc-be2e-b5aa9244a7c4)

Y para verificar que el apartado se ha realizado de manera correcta, se introduce la IP del balanceador de cargas en el navegador, lo cual nos permite comprobar que el apartado se ha realizado con éxito, ya que se muestra la página de bienvenida de NGINX, pudiendo así acceder al servidor web desde Internet, a través del balanceador de cargas. 

![image](https://github.com/evamanriquesz/practicasasr/assets/91720934/1b35efd8-b20a-4698-a7ac-35b6decab5ad)

#### ¿Qué ventajas e inconvenientes tiene hacer https offloading en el balanceador?

- Ventajas: Reduce la carga y aumenta la eficiencia del servidor web por múltiples razones. Por un lado, el balanceador realiza la tarea de cifrado y descifrado de SSL/TLS, permitiendo así al servidor web centrarse en la tarea de procesar peticiones y respuestas; ya que además, el balanceador de carga también puede realizar labores de enrutamiento. Todo esto permite gestionar un mayor número de conexiones de manera más eficiente.

- Inconvenientes: El uso de este tipo de balanceador supone una brecha de seguridad, ya que el tráfico interno entre los servidores y el balanceador no está cifrado, y puede ser susceptible a ataques. Además, puesto que el balanceador realiza múltiples tareas, se convierte en un punto de fallo potenicial, que puede desavastecer de comunicación a múltiples servidores.

#### ¿Qué pasos adicionales has tenido que hacer para que la máquina pueda salir a internet para poder instalar el servidor nginx?

En este segundo apartado, se ha modificado la máquina web para que no tenga IP Pública, se ha configurado Cloud NAT para poder acceder a Internet desde el servidor Web, se ha implementado una nueva regla de firewall referente a la comunicación con el rango de IPs de Google, se han implementado dos health checks para comprobar el correcto funcionamiento de la comunicación del servidor de salto con el PC y con las IPs de Google; y, por último, se ha implementado el balanceador de cargas, a través del cual pasa la información como intermediario entre el servidor web e Internet. 














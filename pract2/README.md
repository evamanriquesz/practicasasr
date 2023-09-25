# PRACTICA 1 - MÁQUINAS VIRTUALES 

En esta práctica se van a llevar a cabo diferentes formas para conectar una máquina a Internet de manera segura. 

## SOLUCIÓN 1 - CREACIÓN DE MÁQUINA DE SALTO 

Para esta solución, se han de crear dos máquinas virtuales: el servidor web y el servidor de salto, que será a través del cual nos conectaremos al servidor web. Ambas contarán con IP privada y pública. Asimismo, será necesario crear una subred (vpc-prac2) a la cual se conectarán ambos servidores para estar relacionados entre sí. Por otro lado, también será necesario crear tres reglas de firewall para controlar el tráfico entre servidores y con internet. La primera regla permitirá a la máquina de salto conectarse con la otra máquina, y comunicarse a través del puerto TCP 22. La segunda regla hará referencia a la comunicación del servidor web por los puertos 80 y 443, así como a la otra máquina. Por último, la tercera máquina, deberá permitir el tráfico del servidor de salto al servidor web, y además, la comunicación por el puerto 22. 

Posteriormente, se ha de habilitar SSH tanto para la máquina de salto desde el ordenador, como para la máquina web desde la máquina de salto. Utilizando SSH, se debe instalar NGINX en el servidor web. 

#### Creación máquinas virtuales




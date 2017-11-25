### Examen 3
**Universidad ICESI**  
**Curso:** Sistemas Operativos  
**Docente:** Daniel Barragán C.  
**Tema:** Descubrimiento de servicios, Microservicios  
**Correo:** daniel.barragan at correo.icesi.edu.co  
  
**Estudiante:** Ana Fernanda Valderrama V.  
**Código:** A00065868  


### Objetivos
* Implementar servicios web que puedan ser consumidos por usuarios o aplicaciones
* Conocer y emplear tecnologías de descubrimiento de servicio

### Prerrequisitos
* Virtualbox o WMWare
* Máquina virtual con sistema operativo CentOS7
* Framework consul, zookeper o etcd

### Descripción
El tercer parcial del curso sistemas operativos trata sobre la creación de servicios web y el uso de tecnologías para el descubrimiento de servicio

![][1]
**Figura 1.** Despliegue básico de microservicios

### Actividades
1. Incluir nombre, código (5%)
2. Ortografía y redacción cuando sea necesario (5%)
3. Despliegue un esquema como el mostrado en la **figura 1**. Empleen un servicio web de su preferencia (puede usar alguno de los ejemplos de clase). No es necesario incluir los componentes para monitoreo (Elasticsearch, Kibana, Logstash) (30%)
4. Adicione un microservicio igual al ya desplegado. Muestre a través de evidencias como las peticiones realizadas al balanceador son dirigidas a la replica del microservicio (30%)
5. Describa los cambios o adiciones necesarias en el diagrama de la **figura 1** para adicionar un microservicio diferente al ya desplegado en el ambiente, tenga en cuenta los siguientes conceptos en su descripción: API Gateway, paradigma reactivo, load balancer, protocolo publicador/suscriptor (interconexión de microservicios) (20%)
6. El informe debe ser entregado en formato pdf a través del moodle y el informe en formato README.md debe ser subido a un repositorio de github. El repositorio de github debe ser un fork de https://github.com/ICESI-Training/so-exam3 y para la entrega deberá hacer un Pull Request (PR) respetando la estructura definida. El código fuente y la url de github deben incluirse en el informe (10%)  
  
  
### Desarrollo  
**3.** Para el despliegue del esquema de la **figura 1** se emplearon varias máquinas virtuales en diferentes computadores de algunos compañeros. Primero, nos conectamos todos a la red por ethernet luego se le asignó a cada uno una tarea, teníamos un **balanceador de carga**, un **servidor de descubrimiento de servicio** y varios **servidores ejecutando microservicios**.  Este esquema permite que al un cliente hacer una solicitud en una aplicación web, el balanceador se encarga de reedirigir la petición que recibió al servidor adecuado para ejecutarla y que se encuentra disponible, los servidores los agrega el servidor de descubrimiento de servicio, cada vez que un servidor que ejecuta un microservicio es agregado mediante consul el balanceador actualiza automáticamente su lista de servidores.
Todos los servidores prestaban el mismo microservicio, mostraban mensajes diferentes para identificar que servidor estaba prestando el servicio.  
Primero se intalaron las dependencias necesarias,  
```
# yum install -y wget unzip
# wget https://bootstrap.pypa.io/get-pip.py -P /tmp
# python /tmp/get-pip.py
# wget https://releases.hashicorp.com/consul/1.0.0/consul_1.0.0_linux_amd64.zip -P /tmp
# unzip /tmp/consul_1.0.0_linux_amd64.zip -d /tmp
# mv /tmp/consul /usr/bin
# mkdir /etc/consul.d
# mkdir -p /etc/consul/data
```  
Se abren los puertos del firewall para el agente consul y los necesarios para acceder al microservicio:  
![][2]  
Empleando el usuario microservices que ya habíamos creado en clases anteriores creamos un ambiente con el nombre que desabamos para el microservicio y lo activabamos:  
![][3]  
Se instalaba la librería de flask, se creaba un script ***.py*** y se ejecutaba en un screen: 
![][4]  
Posteriormente se creaba un archivo de configuración con un healthcheck desde el usuario consul, que sirve para verificar la salud del servicio:  
![][5]  
Se iniciaba el agente consul en una sesión de screen:  
``` 
# su consul
$ consul agent -data-dir=/etc/consul/data -node=agent-one \
    -bind=192.168.130.152 -enable-script-checks=true -config-dir=/etc/consul.d
```  
Y por último, cuando ya estaba funcionando el **servidor de descubrimiento de servicio**, se unía el al ambiente de descubrimiento de servicio y se verificaban los usuarios que se encontraban conectados al ambiente:  
![][6]  
   
El **servidor de descurbimiento de servicio** es el que conoce a todos los servicios y puede localizarlos en la red.  
Primero, se instalan las dependecias necesarias:  
```
# yum install -y wget unzip
# wget https://releases.hashicorp.com/consul/1.0.0/consul_1.0.0_linux_amd64.zip -P /tmp
# unzip /tmp/consul_1.0.0_linux_amd64.zip -d /tmp
# mv /tmp/consul /usr/bin
# mkdir /etc/consul.d
# mkdir -p /etc/consul/data
```  
Se habilitan los puertos del firewall necesarios:  
```
# firewall-cmd --zone=public --add-port=8301/tcp --permanent
# firewall-cmd --zone=public --add-port=8300/tcp --permanent
# firewall-cmd --zone=public --add-port=8500/tcp --permanent
# firewall-cmd --reload
```  
Se ejecuta el agente consul:  
![][7]  
El servidor en ejecución se veía así, aquí ya se habían unido los diferentes servidores de microservicios:  
![][8]  
![][9]    

El **balanceador de carga** recibe múltiples peticiones de diferentes hosts y es el encargado de distribuir/redirigir estas peticiones al servidor apropiado para su atención. En este caso, se implementó HAProxy como balanceador de carga.   
Primero, se instalan las dependencias necesarias:  
```
# yum install -y wget haproxy unzip
# wget https://releases.hashicorp.com/consul-template/0.19.4/consul-template_0.19.4_linux_amd64.zip -P /tmp
# unzip /tmp/consul-template_0.19.4_linux_amd64.zip -d /tmp
# mv /tmp/consul-template /usr/bin
# mkdir /etc/consul-template
```   
Luego, se configura el HAProxy y se abren los puertos necesarios en el firewall:  
Configuración del HAProxy  
``` 
$ sudo mkdir -p /etc/haproxy
$ sudo mkdir -p /var/lib/haproxy 
$ sudo touch /var/lib/haproxy/stats
$ sudo cp ~/haproxy-1.7.8/examples/haproxy.init /etc/init.d/haproxy
$ sudo chmod 755 /etc/init.d/haproxy
$ sudo systemctl daemon-reload
$ sudo chkconfig haproxy on
$ sudo useradd -r haproxy
```  
Apertura de puertos del firewall  
```
$ sudo firewall-cmd --permanent --zone=public --add-service=http
$ sudo firewall-cmd --permanent --zone=public --add-port=8181/tcp
$ sudo firewall-cmd --reload
```  
Por último, se configuran las plantillas de consul-template  
```
# vi /etc/consul-template/haproxy.tpl
```  
Que como mencioné anteriormente son las que van a permitir que la lista de servidores se actualice automáticamente:  
![][10]  
Y se recarga el sistema:  
```  
$ sudo systemctl restart haproxy
```  
Se verifica que el balanceador esté funcionando:  
![][11]  
**4.** Se implementaron 4 clientes ejecutándose al mismo tiempo a los que se podía acceder a través de su dirección IP y su puerto, cuando ya eran parte de los consul members.  
| --- | --- |
|Microservicio 192.168.130.157|![][12]|  
|Microservicio 192.168.130.245|![][13]|  
|Microservicio 192.168.130.236|![][14]|  
|Microservicio 192.168.130.231|![][15]|  
Al hacer la petición al balanceador en el ***Browser*** el escoje a que servidor envía esa petición, esto lo hace utilizando el método de balanceo **roundrobin**.  
| --- | --- |
|Redirección desde 192.168.130.140 a 192.168.130.236|![][16]|  
  
**5.**  Cuando se incluyen diferentes microservicios en una aplicación y se necesitan al mismo tiempo es difícil acceder a ellos, además de que cada cliente necesita cceder a datos diferentes, para eso se emplea el balanceador, un **API Gateway** funciona como un balanceador, es el punto de entrada para todos los clientes, maneja a las peticiones del cliente de dos formas, las simples la enruta al servicio apropiado, y las complejas las maneja moviéndose entre los diferentes servicios que necesita, la diferencia entre un APIGateway y un load-balancer quizás es que el último provee Healthcheck y persistencia en la sesión. Como dirigir las peticiones del cliente adecuadamente se relaciona APIGateway, el paradigma reactivo y los balanceadores de carga. En nuestra implementación era posible implementar un APIGateway y el balanceador de carga y liberar al APIGateway del balnceo de cargas o se puede hacer que el ejecute todo el balanceo, pues también se ha demostrado que emplea algo como el método **round robin** en algunos casos. 
![][17]


### Referencias
https://github.com/ICESI/so-microservices-python    
http://microservices.io/patterns/microservices.html  
http://microservices.io/patterns/apigateway  
https://ronanquillevere.github.io/2017/09/28/gateway-lb-reverse-proxy.html#.WhjUZ1XibIU  
https://www.nginx.com/blog/building-microservices-using-an-api-gateway/  

[1]: images/Microservices_Deployment.png  
[2]: images/puertosFirewall.PNG  
[3]: images/ambienteElBuenMicroservicio.PNG  
[4]: images/scriptElBuenMicroservicio.PNG
[5]: images/Healthcheck.PNG
[6]: images/MiembrosAmbiente.PNG
[7]: images/consul_agent_server.PNG
[8]: images/consul_members.PNG
[9]: images/consul_logs.PNG
[10]: images/configuracionConsulTemplates.png
[11]: images/BalanceadoCorriendo.png
[12]: images/MiCliente.PNG
[13]: images/ClienteOperations.PNG
[14]: images/ClienteIguazo.PNG
[15]: images/ClienteElMicroservicio.PNG
[16]: images/RedireccionLB.PNG
[17]: images/Microservices.png





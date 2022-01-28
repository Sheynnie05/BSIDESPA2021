# BSIDESPA2021
Taller de detección de amenazas en red elaborado para el BSIDES Panamá 2021

Taller | Armando un laboratorio para detectar amenazas en la red






Dictado por: Sheyla Leacock
Objetivos del taller:

El objetivo de este taller es armar un laboratorio de detección de amenazas con herramientas OpenSource y simular ataques a nuestro entorno de red que nos permitan conocer cómo podemos mejorar nuestras defensas.
Se propone desplegar de forma automatizada el sistema de detección de intrusos Snort en un entorno Linux, así como realizar las configuraciones e implementar reglas que nos permitan detectar amenazas a nivel de red.
Se implementará también una arquitectura cliente servidor con Pytbull para generar tráfico malicioso en nuestra red a fin de que pueda ser detectado por el IDS.
Además desplegaremos herramientas que nos permitan monitorear las alertas generadas por el IDS y realizar correlación de eventos


Prerrequisitos:
Descargar e instalar VirtualBox: https://www.virtualbox.org/wiki/Downloads 

Descargar una máquina virtual con OpenSUSE 15.2 desde OSboxes: https://www.osboxes.org/opensuse/


Descargar una máquina virtual con Fedora 33 desde OSboxes: https://www.osboxes.org/fedora/ 


Nota: La ventaja de las máquinas de OSBoxes es que ya se encuentran “Listas para usar”. Utilizan el siguiente usuario y contraseña por defecto: Usuario: osboxes Contraseña: osboxes.org

Una vez descargadas las máquinas virtuales de OSboxes, descomprimir cada una en un subdirectorio:

Abrir virtualbox y crear 2 maquinas virtuales con las siguientes características: 
Mínimo 2048 mb de memoria



Asignar el archivo .vdi previamente extraído como disco duro existente de la máquina:



Dar click al botón crear para finalizar.

Una vez creada, desde virtualbox seleccione la opción herramientas -> Preferencias


Seleccionar la opción “Red” y dar click al botón con símbolo “+” para añadir una nueva red.

Nota: Si queremos ver los detalles como rango de IP con el cual se creó la red NAT podemos visualizarlo dandole click al botón con el símbolo de configuración (A pesar de que no es necesario editar ningún parámetro en este apartado se recomienda desmarcar el soporte para DHCP de forma que para pruebas posteriores no nos cambie la IP de las máquinas).

Una vez creada la red NAT, nos dirigimos a cada una de las máquinas creadas para enlazarlas a esta red.  Seleccione la máquina y de click al botón configuración->red y en la sección del primer adaptador, asigne la “red NAT” creada previamente:




Con estas configuraciones ya podemos iniciar las máquinas.

Notas: 
Ambas máquinas deben quedar bajo la misma red NAT para que puedan visualizarse entre sí.
Para la máquina FEDORA se debe habilitar en esta misma sección el modo promiscuo en: “permitir todo” para que pueda visualizar todo el tráfico de la red ya que en esta se instalará el IDS Snort.
Se recomienda actualizar las máquinas: 
Para actualizar Fedora, desde la terminal ingrese el comando: “sudo yum update”.
Para OpenSUSE, desde la terminal ingrese el comando “sudo zypper update”.

Parte 1 - Configuración del IDS Snort en la máquina Fedora.

Instalar las librerías necesarias desde la terminal:
Aplicaciones → buscador→terminal
Ingresamos lo siguiente para instalar las librerías:

sudo yum install pcre.* gcc git vim flex bison libpcap.* libdnet.* zlib.* libtirpc.* luajit autoconf openssl libtool perl lwp.* perl-libwww-perl perl-GD perl-Crypt-SSLeay perl-LWP-Protocol-https tcpdump zlib-devel libpcap-devel pcre-devel libdnet-devel libtirpc-devel


Nota: Los entornos linux son Case sensitive por lo que se deben respetar las mayúsculas y minúsculas.


Ahora, realizaremos el clon del repositorio github de Snorter en nuestro entorno local con el comando:  git clone https://github.com/joanbono/Snorter.git


Posicionarse en la carpeta Snorter/src y editar la línea 53 del archivo snorter.sh con el comando sudo vim Snorter.sh. Una vez en el editor agregaremos el parámetro –disable-open-appid como se muestra a continuación:

Para salir del editor presionamos la tecla escape ESC seguido de  :wq!
Luego, ejecutamos los siguientes comandos para asociar los comandos de instalación de Fedora en el script de snorter:
sed -i ‘s/apt-get/yum/g’ Snorter.sh
sed -i ‘s/--force-yes//g’ Snorter.sh


Una vez hecho esto ejecutamos snorter:
sudo bash Snorter.sh -o <‘oinkcode’> -i ‘interfaz de red’


Nota: 
El comando -o indica que ingresaremos el oinkcode asociado, en caso de no tenerlo, ejecutemos Snorter sin este comando.
Es recomendable ejecutar snorter con un oinkcode generado al registrarnos en Snort: https://snort.org/users/sign_up. El oinkcode es una llave asociada a nuestro registro que funciona como api para descargar las reglas de snort. Para el taller no es necesario el oinkcode, se utilizarán las reglas comunitarias.
Con el comando -i indicamos cuál será la interfaz de red sobre la cual queremos que Snort realice el monitoreo (Por ejemplo: eth1, enp0s3). Puedes ver tus interfaces de red desde la terminal ejecutando el comando ip address.
Una vez se realice la revisión y descarga de librerías se presentará la interfaz para configuración:

Primero, para la configuración de snort se deben modificar los valores de la red interna (la que indicamos al crear la red NAT de virtualbox) y la red externa (Todo aquello que no sea la red interna). 

Recuerda salir del editor con la tecla escape ESC seguido de  :wq!
Para la segunda pregunta de la configuración elegimos la opción 2 (cualquiera de las 3 primeras es válida).


Después de esto, se ejecutará Snort en modo de prueba para validar la configuración.
Anteriormente Snorter escribió una regla en el archivo de reglas locales que alertará cualquier tráfico ICMP generado, la que usará en el proceso de prueba de snort.

Nota: En este apartado, podemos probar snort lanzando un comando ping desde otra máquina que esté dentro de la red NAT hacia la máquina con snort. Presionamos enter y veremos las alertas generadas en tiempo real. Luego podemos salir de este modo presionando Ctrl+C para continuar con las configuraciones.





Ingresamos la opción n para no instalar Barnyard2 (fuera de alcance del taller ya que requiere otras configuraciones y dependencias).
Ingresamos la opción y para instalar Pulledpork.

Seleccionamos la opción n para no habilitar las reglas de Emerging Threats y la opción y para la creación del servicio snort y también para la opción de descargar nuevas reglas utilizando Pulledpork.








Instalamos websnort con la opción y

Habilitamos la descarga de las reglas comunitarias y luego reiniciamos la máquina virtual para aplicar todos los cambios con la opción y.

Una vez reiniciamos, nos dirigimos al archivo de configuración de snort para incluir la ruta al archivo de reglas de snort:
sudo vim /etc/snort/snort.conf

y debajo de la ruta de reglas locales (línea 547) agregamos la linea include $RULE_PATH/snort.rules y bajo la línea 690 agregamos la inclusión de las reglas comunitarias: include $RULE_PATH/community.rules




Ahora si, podemos ejecutar snort y dejarlo en modo escucha para ver las alertas en la consola con el comando: sudo snort -A console -i enp0s3 -u snort -g snort -c /etc/snort/snort.conf



Nota: los archivos de reglas de snort los encuentras en la ruta: /etc/snort/rules.

Parte 2 - Configuración del servidor Pytbull en la máquina Fedora.

Nuevamente desde la consola instalamos las librerías que necesitaremos ahora para el servidor Pytbull:
sudo yum install vsftpd httpd python2


Descargar pytbull con el comando: wget https://downloads.sourceforge.net/project/pytbull/pytbull-2.1.tar.bz2
Extraer los archivos a la carpeta var: tar -xvf pytbull-2.1.tar.bz2 -C /var/

Habilitar e iniciar los servicios para los servidores web (httpd), ftp (vsftpd) y ssh (sshd):
sudo systemctl enable vsftpd
sudo systemctl start vsftpd
sudo systemctl enable sshd
sudo systemctl start sshd
sudo systemctl enable httpd
sudo systemctl start httpd

Nota: podemos verificar el status de los servicios (deben quedar activos) por ejemplo: sudo systemctl status httpd 

Ahora, añadimos las reglas  al firewall para permitir la comunicación de los servicios:
sudo firewall-cmd --permanent --add-service=ftp
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-port=21/tcp
sudo firewall-cmd --permanent --add-port=22/tcp
sudo firewall-cmd --permanent --add-port=80/tcp
Luego reiniciamos el firewall:
sudo firewall-cmd --reload 

Nos dirigimos a /var/ftp/pub y creamos un archivo de alerta, el cual llamaremos desde el cliente pytbull:
cd /var/ftp/pub
sudo touch snort.alert
Nos dirigimos a la ruta /var/pytbull/server y ejecutamos el servidor de pytbull y lo dejamos en modo escucha:
cd /var/pytbull/server
sudo python2 pytbull-server.py


Parte 3 - Configuración del cliente Pytbull en la máquina OpenSUSE.



Instalamos las librerias necesarias:
sudo zypper install python2 python2-pip nmap hping tcpreplay apache2-utils python3 libdnet1 libssl45, libcrypto43 y libssh-devel python2-feedparser 


Ahora haremos un upgrade al comando pip para luego instalar las librería sscapy y cherrypy:
sudo pip install --upgrade pip
sudo pip install scapy
sudo pip install cherrypy

Agregamos el repositorio de seguridad que nos permitirá descargar nikto:
zypper addrepo https://download.opensuse.org/repositories/security/openSUSE_Leap_15.2/security.repo 
sudo zypper install nikto



Ahora agregaremos el repositorio packman para descargar ncrack:
zypper ar -cfp 90 'https://ftp.gwdg.de/pub/linux/misc/packman/suse/openSUSE_Leap_$releasever/' packman

Indicamos la opción y para continuar

Procedemos a instalar ncrack con: sudo zypper install ncrack

Descargar pytbull con el comando: wget https://downloads.sourceforge.net/project/pytbull/pytbull-2.1.tar.bz2
Extraemos la carpeta y la movemos al directorio /opt/ con: sudo tar -xvf pytbull-2.1.tar.bz2 -C /opt/
Desde el script de pytbull, agregaremos dos líneas para importar la librería datetime la cual es utilizada por varios de los tests a ejecutar:
sudo vim /opt/pytbull/pytbull

Ya dentro del archivo bajamos a la línea 368 y agregamos una nueva línea con lo siguiente: import datetime
Repetimos este paso en la línea 482 del archivo y guardamos los cambios con ESC :wq!

Ahora nos ubicamos en la siguiente ruta:
cd /opt/pytbull/conf y editamos el archivo de configuración config.cfg para ir ingresando los datos:
sudo vim config.cfg
En la estructura del archivo, la cual está dividida por secciones, ingresaremos la información correspondiente:
En la sección client, solo ingresaremos la ipaddr en la cual tenemos instalado nuestro cliente pytbull y la iface que indica la interfaz de salida que usará pytbull para enviar los payloads.
En la sección PATHS, en las variables alertfiles indicamos la dirección donde se almacenan nuestros archivos de alerta, en este caso indicaremos la siguiente ruta: /var/ftp/pub/snort.alert
Es importante mantener solo una ruta alertfiles activa, la otra se comenta con un #.
Nota: Para efectos del taller se creó anteriormente el archivo snort.alert para evitar el error de pytbull client al intentar acceder al archivo de alerta, ya que dicha configuración no está dentro del alcance del taller. su objetivo es mostrar en el reporte de pytbull las pruebas que el IDS detectó correctamente. Para ello se debe configurar el acceso FTP a la ruta de logs de snort con los permisos correspondientes.

En la sección FTP agregamos el protocolo FTP, puerto 21 y las credenciales de nuestra máquina osboxes para establecer la conexión:

Las secciones Timing y Server no requieren modificación.

En la sección ENV, modificamos las rutas por defecto y agregamos las siguientes para nikto (/usr/bin/nikto), niktoconf (/etc/nikto.conf) y  ncrack (/usr/bin/ncrack) como se muestra en la imagen:


El apartado de TESTS puede modificarse para habilitar (valor 1) o deshabilitar (valor 0) los tests que queramos realizar. Para este caso deshabilitamos los testRules, denialOfService y ipReputation.

Una vez realizadas las configuraciones, procedemos a ejecutar las pruebas desde pytbull client: sudo python2 ./pytbull -t 192.168.2.8












Luego de terminada la ejecución de los tests de pytbull, podemos regresar a la máquina snort a visualizar las alertas generadas utilizando websnort:
Asignamos los permisos correspondientes a la carpeta de logs de snort para que pueda utilizarlos websnort:
sudo chmod -R 775 /var/log/snort


Ejecutamos el comando websnort desde la consola:

Y nos dirigimos a la ruta indicada: http://0.0.0.0:8080 donde seleccionamos “choose file” para cargar un archivo de alerta:

Seleccionamos el archivo de log a visualizar:

Seleccionamos submit y luego podremos visualizar las alertas correspondientes:






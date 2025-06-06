# **Instalación de Nagios Core**

## **Requisitos del sistema**

La instalación de Nagios Core se realizará en un sistema con las siguientes características mínimas:

- **Sistema Operativo:** RHEL 9.4
- **Disco:** 20 GB de espacio
- **Memoria RAM:** 4 GB
- **CPU:** 4 vCPUs

## **Requisitos previos**

Antes de proceder con la instalación de Nagios Core, asegúrate de que el sistema esté:

1. **Licenciado con Red Hat:** El sistema debe contar con una licencia activa de Red Hat.
2. **Actualizado:** El sistema operativo debe estar actualizado con los últimos parches y actualizaciones disponibles.

## **Para empezar con la instalación de Nagios Core**
Asegúrate de que los siguientes paquetes estén instalados en tu sistema. Ejecuta los siguientes comandos para instalar las dependencias necesarias:


```bash
dnf install -y gcc glibc glibc-common perl httpd php wget gd gd-devel
dnf install -y openssl-devel
```

## **Descargar el código fuente**

Descarga el código fuente de Nagios Core desde el repositorio oficial:

```bash
cd /tmp
wget --output-document="nagioscore.tar.gz" $(wget -q -O - https://api.github.com/repos/NagiosEnterprises/nagioscore/releases/latest  | grep '"browser_download_url":' | grep -o 'https://[^"]*')
tar xzf nagioscore.tar.gz
```

## **Compilación**

Accede a la carpeta donde descargaste el código fuente:

```bash
cd /tmp/nagios-*
```

Configura y compila Nagios Core:

```bash
./configure
make all
```

## **Crear usuario y grupo**

Este paso crea el usuario y grupo `nagios`, y añade el usuario `apache` al grupo `nagios` para que Apache pueda interactuar con Nagios.

```bash
make install-groups-users
usermod -a -G nagios apache
```

## **Instalar los archivos binarios**

Instala los archivos binarios, los CGIs y los archivos HTML necesarios para Nagios Core:

```bash
make install
```

## **Configuración del servicio**

Instala los scripts de inicio del servicio para que Nagios se inicie automáticamente con el sistema.

```bash
make install-daemoninit
systemctl enable --now httpd.service
```

## **Instalar el modo de comandos**

Este paso instala y configura el archivo de comandos externos de Nagios.

```bash
make install-commandmode
```

## **Instalar los archivos de configuración**

Instala los archivos de configuración de ejemplo necesarios para que Nagios pueda arrancar correctamente:

```bash
make install-config
```

## **Instalar archivos de configuración de apache**

Este paso instala los archivos de configuración de Apache para integrarlos con Nagios:

```bash
make install-webconf
```

## **Configurar el firewall**

Abre el puerto 80 en el firewall para permitir el acceso a la interfaz web de Nagios:

```bash
firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --zone=public --reload
```

## **Crear la cuenta de usuario `nagiosadmin`**

Para poder acceder a la interfaz web de Nagios, crea la cuenta `nagiosadmin`:

```bash
htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
```

> [!NOTE]
> Cuando agregues usuarios adicionales en el futuro, no uses `-c` para evitar reemplazar la cuenta `nagiosadmin`.*

## **Iniciar el servicio de apache**

Inicia el servicio de Apache para servir la interfaz web de Nagios:

```bash
systemctl enable --now httpd.service
```

## **Iniciar el servicio de Nagios**

Inicia el servicio de Nagios para comenzar a monitorear:

```bash
systemctl enable --now nagios.service
```

## **Probar Nagios**

Una vez instalado y configurado Nagios Core, accede a la interfaz web utilizando la IP o FQDN de tu servidor Nagios Core:

```bash
http://IP_LOCAL/nagios
http://core-013.domain.local/nagios
```

Inicia sesión con el nombre de usuario `nagiosadmin` y la contraseña que configuraste previamente. Después deberías ver la interfaz de Nagios.

Ten en cuenta que solo se ha instalado el motor de Nagios Core y verás errores relacionados con los hosts y servicios, ya que no se ha configurado nada aún para monitorear.

# **Instalación de Nagios Plugins**

Nagios Core requiere de plugins para funcionar correctamente. Sigue estos pasos para instalar los **Nagios Plugins**.

Asegúrate de tener instalados los siguientes paquetes

```bash
dnf install -y gcc glibc glibc-common make gettext automake autoconf wget openssl-devel net-snmp net-snmp-utils
```

## **Descargar el código fuente de los plugins**

Descarga el código fuente de los plugins desde el repositorio oficial:

```bash
cd /tmp
wget --output-document="nagios-plugins.tar.gz" $(wget -q -O - https://api.github.com/repos/nagios-plugins/nagios-plugins/releases/latest  | grep '"browser_download_url":' | grep -o 'https://[^"]*')
tar zxf nagios-plugins.tar.gz
```

## **Compilación e instalación de los Plugins**

Compila e instala los plugins:

```bash
cd /tmp/nagios-plugins-*
./configure
make
make install
```

## **Probar los Plugins**

Accede a la interfaz web de Nagios y realiza una verificación en un objeto de host o servicio:

Apunta tu navegador a la dirección IP o FQDN de tu servidor Nagios Core:

   ```bash
   http://IP_LOCAL/nagios
   http://core-013.domain.local/nagios
   ```

Ve a un objeto de host o servicio y selecciona "Reprogramar la siguiente verificación" en el menú de Comandos. El error anterior debería desaparecer y ahora se mostrará la salida correcta.

> [!Note] 
> Puedes controlar el servicio de Nagios con los siguientes comandos:

```bash
systemctl start nagios.service   # Inicia el servicio de Nagios
systemctl stop nagios.service    # Detiene el servicio de Nagios
systemctl restart nagios.service # Reinicia el servicio de Nagios
systemctl status nagios.service  # Muestra el estado del servicio de Nagios
```

# **Instalación del primer cliente Nagios, utilizando el agente NCPA.**

La instalación del cliente Nagios se realizará en un sistema con las siguientes características:

- **Sistema Operativo:** Red Hat 8.10
- **Disco:** 5 GB de espacio 
- **Memoria RAM:** 2 GB
- **CPU:** 1 vCPUs

### **El primer paso es instalar el repositorio**

```
rpm -Uvh https://repo.nagios.com/nagios/8/nagios-repo-8-2.el8.noarch.rpm
```

### **Una vez instalado el repositorio, deberá ejecutar el siguiente comando para instalar NCPA:**
```
dnf install ncpa -y
```

### **Ejecute el siguiente comando para abrir el archivo de configuración del agente.**
```
sudo vi /usr/local/ncpa/etc/ncpa.cfg
```

Busque la siguiente línea:
```
community_string = mytoken
```
Reemplace mytoken con un token seguro de su elección. Por ejemplo:
```
community_string = Str0ngT0k3n
```
Ahora deberá reiniciar el servicio `ncpa` para que los cambios surtan efecto.

```
systemctl restart ncpa.service
```
### **Configurar el firewall**
Es necesario crear una regla de firewall en su equipo Linux para permitir el tráfico entrante a NCPA en el puerto TCP 5693
```
firewall-cmd --add-port=5693/tcp –-permanent
firewall-cmd --reload
```

### **Probar NCPA**
Para asegurarse de que la instalación se realizó correctamente y que NCPA está funcionando, acceda a su interfaz web. Para ello, necesitará:

La dirección IP del equipo donde instaló NCPA.

El token que configuró para la autenticación.

Abra un navegador web y acceda a la interfaz de NCPA con la siguiente URL:

```
https://<Dirección IP de NCPA>:5693/
```

Es posible que el navegador muestre una advertencia de seguridad. Esto ocurre porque NCPA utiliza un certificado autofirmado para cifrar la comunicación, y el navegador no lo reconoce como una entidad de confianza.

Para continuar, haga clic en "Avanzado" y luego en "Agregar excepción" o "Continuar a xxx", según el navegador que esté utilizando.


# **Iniciar monitoreo en Nagios Core**
Para monitorear equipos mediante comprobaciones activas, puede utilizar el Asistente de configuración de NCPA.
Esto descarga, descomprime y copia el plugin check_ncpa.py al directorio de plugins de Nagios, donde se utilizará para realizar comprobaciones activas con el agente NCPA.
```
cd /tmp
wget https://assets.nagios.com/downloads/ncpa/check_ncpa.tar.gz
tar -xvzf check_ncpa.tar.gz
cp check_ncpa.py /usr/local/nagios/libexec/
```

# **Crear la definición del comando check**
Cree la definición del comando check_ncpa en sus archivos de configuración de Nagios Core. Generalmente, estos archivos se encuentran en /usr/local/nagios/etc/objects/. Si tiene un archivo llamado commands.cfg, es posible que desee agregar la definición del comando allí.

```
define command {
    command_name    check_ncpa
    command_line    $USER1$/check_ncpa.py -H $HOSTADDRESS$ $ARG1$
}
```
### **Crear checks de Nagios**
Ahora puede crear las comprobaciones en un archivo de configuración dentro de /usr/local/nagios/etc/. En este ejemplo, crearemos un archivo de configuración llamado ncpa.cfg con las siguientes definiciones:
```
define host {
    host_name               cliente_1
    address                 192.168.1.10
    check_command           check_ncpa!-t 'mytoken' -P 5693 -M system/agent_version
    max_check_attempts      5
    check_interval          5
    retry_interval          1
    check_period            24x7
    contacts                nagiosadmin
    notification_interval   60
    notification_period     24x7
    notifications_enabled   1
    icon_image              ncpa.png
    statusmap_image         ncpa.png
    register                1
}

define service {
    host_name               cliente_1
    service_description     CPU Usage
    check_command           check_ncpa!-t 'mytoken' -P 5693 -M cpu/percent -w 20 -c 40 -q 'aggregate=avg'
    max_check_attempts      5
    check_interval          5
    retry_interval          1
    check_period            24x7
    notification_interval   60
    notification_period     24x7
    contacts                nagiosadmin
    register                1
}

define service {
    host_name               cliente_1
    service_description     Memory Usage
    check_command           check_ncpa!-t 'mytoken' -P 5693 -M memory/virtual -w 50 -c 80 -u G
    max_check_attempts      5
    check_interval          5
    retry_interval          1
    check_period            24x7
    notification_interval   60
    notification_period     24x7
    contacts                nagiosadmin
    register                1
}

define service {
    host_name               cliente_1
    service_description     Process Count
    check_command           check_ncpa!-t 'mytoken' -P 5693 -M processes -w 150 -c 200
    max_check_attempts      5
    check_interval          5
    retry_interval          1
    check_period            24x7
    notification_interval   60
    notification_period     24x7
    contacts                nagiosadmin
    register                1
}
```
> [!NOTE]
> Reemplace -t `'mytoken'` con su propio token y también cambie la dirección IP de `address` con la IP de su host monitoreado. Esto indicará a Nagios que realice comprobaciones activas y creará un host denominado `"cliente_1"` con las comprobaciones de Uso de CPU, Uso de Memoria y Recuento de Procesos.

### **Reiniciar Nagios y verificar los controles activos**
Reinicie el servicio de Nagios, y los hosts/servicios deberían aparecer como pendientes. Una vez que se realicen las comprobaciones iniciales, podrá ver los datos de NCPA en Nagios.


# Errores SELinux

El error tiene que ver con los permisos de `SELinux` y cómo está configurado el acceso a los archivos que necesita Nagios para funcionar correctamente, específicamente el archivo nagios.cmd que se encuentra en la ruta /usr/local/nagios/var/rw/nagios.cmd.

El error en el log de `SELinux` (avc: denied { getattr }) indica que el proceso de Apache (cmd.cgi) intentó acceder al archivo nagios.cmd, pero `SELinux` bloqueó ese acceso debido a que el archivo no tenía el contexto adecuado.

```
type=AVC msg=audit(1743887453.397:1378): avc:  denied  { getattr } for  pid=13389 comm="cmd.cgi" path="/usr/local/nagios/var/rw/nagios.cmd" dev="dm-0" ino=17804920 scontext=system_u:system_r:httpd_sys_script_t:s0 tcontext=system_u:object_r:usr_t:s0 tclass=fifo_file permissive=0
```

### Solución aplicada
Estas tres líneas aplican los contextos de SELinux adecuados a las carpetas de Nagios para que Apache pueda acceder a ellas. Utiliza el comando --reference para asignar el contexto de las carpetas de Apache (/var/www/html, /var/www/cgi-bin) a las carpetas de Nagios correspondientes.

```
chcon -R --reference=/var/www/html /usr/local/nagios/share # Asigna el contexto de /var/www/html (donde Apache normalmente sirve contenido web) a la carpeta /usr/local/nagios/share, donde se encuentran los archivos web de Nagios.

chcon -R --reference=/var/www/html /usr/local/nagios/var # Aplica el contexto de /var/www/html a la carpeta /usr/local/nagios/var, que almacena archivos de datos de Nagios.

chcon -R --reference=/var/www/cgi-bin /usr/local/nagios/sbin # Asigna el contexto de ejecución de CGI (usualmente en /var/www/cgi-bin) a la carpeta /usr/local/nagios/sbin, que contiene los archivos ejecutables de Nagios.
```

### Permitir acceso de escritura a la carpeta rw
Este comando asegura que Apache pueda acceder a la carpeta rw dentro de /usr/local/nagios/var/, dándole permisos de escritura. Esto es necesario porque el archivo nagios.cmd está en esta carpeta y se necesita acceso de escritura para que Nagios pueda enviarle comandos desde la interfaz web.

```
chcon -R -t httpd_sys_rw_content_t /usr/local/nagios/var/rw
```

Referencias
https://support.nagios.com/forum/viewtopic.php?t=5002

# Definir un `event_handler` en `Nagios Core`

El proceso de configurar un event handler en `Nagios Core`, el cual se ejecutará en el cliente para reiniciar automáticamente el servicio SSH (sshd) en caso de que falle. El objetivo es asegurar que, si el servicio SSH no está en ejecución, se reinicie de forma automática sin intervención manual.

### Definición del event_handler
El primer paso consiste en configurar el event handler en el servidor Nagios. Este event handler se ejecutará cuando Nagios detecte que el servicio SSH ha fallado en el cliente.

Para ello, se editó el archivo de configuración /usr/local/nagios/etc/servers/HOST.cfg del servicio en Nagios, añadiendo la siguiente definición:
```
define service {
    host_name               cliente-1
    service_description     Service SSH
    check_command           check_ncpa!-t 'token' -P 5693 -M services -q 'service=sshd,status=running'
    max_check_attempts      5
    check_interval          5
    retry_interval          1
    check_period            24x7
    notification_interval   60
    notification_period     24x7
    contacts                nagiosadmin
    register                1
    event_handler           check_ncpa! -t 'token' -p 5693 -M 'plugins/restart_services.sh'
}
```

### Configuración de permisos en el cliente
Para permitir que el script de reinicio del servicio SSH se ejecute sin pedir una contraseña, es necesario modificar el archivo /etc/sudoers en el cliente.

Esto otorga al usuario nagios permisos para ejecutar el comando `systemctl restart sshd` sin requerir una contraseña. Este paso es esencial para que el script pueda reiniciar el servicio SSH sin intervención manual.

```
nagios    ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart sshd
```

> [!CAUTION] 
> Es importante realizar esta modificación de forma segura usando el comando `visudo`, ya que este validará la sintaxis antes de guardar los cambios, evitando posibles errores de configuración.


### Creación del script para reiniciar el servicio
En el cliente, se creó un script llamado restart_services.sh que se encarga de reiniciar el servicio SSH utilizando el comando systemctl restart sshd.

El script se ubicó en el directorio `/usr/local/ncpa/plugins/` en el cliente y se configuró con permisos de ejecución adecuados para el usuario nagios.

```
#!/bin/bash
sudo systemctl restart sshd
exit 0
```

### Verificación del comando `NCPA` en Nagios Core
Para que el event handler funcione correctamente, es necesario verificar que el plugin check_ncpa esté correctamente configurado para comunicarse con el cliente. El comando utilizado en el archivo de servicio de Nagios es el siguiente:

```
check_ncpa! -t -H IP_ADDRESS 'token' -P 5693 -M 'services' -q 'service=sshd,status=running'
```



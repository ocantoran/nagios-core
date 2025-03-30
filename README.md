

# Instalación de Nagios Core

## Requisitos del Sistema

La instalación de Nagios Core se realizará en un sistema con las siguientes características mínimas:

- **Sistema Operativo:** RHEL 9.4
- **Disco:** 20 GB de espacio libre
- **Memoria RAM:** 4 GB
- **CPU:** 4 vCPUs

## Requisitos Previos

Antes de proceder con la instalación de Nagios Core, asegúrate de que el sistema esté:

1. **Licenciado con Red Hat:** El sistema debe contar con una licencia activa de Red Hat.
2. **Actualizado:** El sistema operativo debe estar actualizado con los últimos parches y actualizaciones disponibles.

## Para empezar con la instalación de Nagios Core
Asegúrate de que los siguientes paquetes estén instalados en tu sistema. Ejecuta los siguientes comandos para instalar las dependencias necesarias:


```bash
dnf install -y gcc glibc glibc-common perl httpd php wget gd gd-devel
dnf install -y openssl-devel
```

## Descargar el Código Fuente

Descarga el código fuente de Nagios Core desde el repositorio oficial:

```bash
cd /tmp
wget --output-document="nagioscore.tar.gz" $(wget -q -O - https://api.github.com/repos/NagiosEnterprises/nagioscore/releases/latest  | grep '"browser_download_url":' | grep -o 'https://[^"]*')
tar xzf nagioscore.tar.gz
```

## Compilación

Accede a la carpeta donde descargaste el código fuente:

```bash
cd /tmp/nagios-*
```

Configura y compila Nagios Core:

```bash
./configure
make all
```

## Crear Usuario y Grupo

Este paso crea el usuario y grupo `nagios`, y añade el usuario `apache` al grupo `nagios` para que Apache pueda interactuar con Nagios.

```bash
make install-groups-users
usermod -a -G nagios apache
```

## Instalar los Archivos Binarios

Instala los archivos binarios, los CGIs y los archivos HTML necesarios para Nagios Core:

```bash
make install
```

## Configuración del Servicio

Instala los scripts de inicio del servicio para que Nagios se inicie automáticamente con el sistema.

```bash
make install-daemoninit
systemctl enable --now httpd.service
```

## Instalar el Modo de Comandos

Este paso instala y configura el archivo de comandos externos de Nagios.

```bash
make install-commandmode
```

## Instalar los Archivos de Configuración

Instala los archivos de configuración de ejemplo necesarios para que Nagios pueda arrancar correctamente:

```bash
make install-config
```

## Instalar Archivos de Configuración de Apache

Este paso instala los archivos de configuración de Apache para integrarlos con Nagios:

```bash
make install-webconf
```

## Configurar el Firewall

Abre el puerto 80 en el firewall para permitir el acceso a la interfaz web de Nagios:

```bash
firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --zone=public --reload
```

## Crear la Cuenta de Usuario `nagiosadmin`

Para poder acceder a la interfaz web de Nagios, crea la cuenta `nagiosadmin`:

```bash
htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
```

> [!NOTE]
> Cuando agregues usuarios adicionales en el futuro, no uses `-c` para evitar reemplazar la cuenta `nagiosadmin`.*

## Iniciar el Servicio de Apache

Inicia el servicio de Apache para servir la interfaz web de Nagios:

```bash
systemctl enable --now httpd.service
```

## Iniciar el Servicio de Nagios

Inicia el servicio de Nagios para comenzar a monitorear:

```bash
systemctl enable --now nagios.service
```

## Probar Nagios

Una vez instalado y configurado Nagios Core, accede a la interfaz web utilizando la IP o FQDN de tu servidor Nagios Core:

```bash
http://IP_LOCAL/nagios
http://core-013.domain.local/nagios
```

Inicia sesión con el nombre de usuario `nagiosadmin` y la contraseña que configuraste previamente. Después de iniciar sesión, deberías ver la interfaz de Nagios.

Ten en cuenta que solo se ha instalado el motor de Nagios Core y verás errores relacionados con los hosts y servicios, ya que no se ha configurado nada aún para monitorear.

## Instalación de Nagios Plugins

Nagios Core requiere de plugins para funcionar correctamente. Sigue estos pasos para instalar los **Nagios Plugins**.

## **Prerequisitos:**

Asegúrate de tener instalados los siguientes paquetes

```bash
dnf install -y gcc glibc glibc-common make gettext automake autoconf wget openssl-devel net-snmp net-snmp-utils
```

## Descargar el Código Fuente de los Plugins

Descarga el código fuente de los plugins desde el repositorio oficial:

```bash
cd /tmp
wget --output-document="nagios-plugins.tar.gz" $(wget -q -O - https://api.github.com/repos/nagios-plugins/nagios-plugins/releases/latest  | grep '"browser_download_url":' | grep -o 'https://[^"]*')
tar zxf nagios-plugins.tar.gz
```

## Compilación e Instalación de los Plugins

Compila e instala los plugins:

```bash
cd /tmp/nagios-plugins-*
./configure
make
make install
```

## Probar los Plugins

Accede a la interfaz web de Nagios y realiza una verificación en un objeto de host o servicio:

Apunta tu navegador a la dirección IP o FQDN de tu servidor Nagios Core:

   ```bash
   http://IP_LOCAL/nagios
   http://core-013.domain.local/nagios
   ```

Ve a un objeto de host o servicio y selecciona "Reprogramar la siguiente verificación" en el menú de Comandos. El error anterior debería desaparecer y ahora se mostrará la salida correcta.

## Administrar el Servicio de Nagios

Puedes controlar el servicio de Nagios con los siguientes comandos:

```bash
systemctl start nagios.service   # Inicia el servicio de Nagios
systemctl stop nagios.service    # Detiene el servicio de Nagios
systemctl restart nagios.service # Reinicia el servicio de Nagios
systemctl status nagios.service  # Muestra el estado del servicio de Nagios
```




# Instalación de cliente Nagios con NCPA en Linux basado en RPM.
### El primer paso es instalar el repositorio

```
rpm -Uvh https://repo.nagios.com/nagios/8/nagios-repo-8-2.el8.noarch.rpm
```

### Una vez instalado el repositorio, deberá ejecutar el siguiente comando para instalar NCPA:
```
dnf install ncpa -y
```

### Ejecute el siguiente comando para abrir el archivo de configuración del agente.
```
sudo vi /usr/local/ncpa/etc/ncpa.cfg
```

Busque la siguiente línea:
```
community_string = mytoken
```
Cámbielo al token requerido, por ejemplo:
```
community_string = Str0ngT0k3n
```
Ahora deberá reiniciar el servicio `ncpa` para que los cambios surtan efecto.

```
systemctl restart ncpa.service
```
### Configurar el firewall 
Es necesario crear una regla de firewall en su equipo Linux para permitir el tráfico entrante a NCPA en el puerto TCP 5693
```
firewall-cmd --add-port=5693/tcp –-permanent
firewall-cmd --reload
```

### Probar NCPA
Para garantizar que la instalación se haya realizado correctamente y que NCPA esté escuchando, intente acceder a la interfaz web del agente. Para ello, necesitará saber:

- La dirección IP del equipo donde instaló NCPA
- El token o la cadena de comunidad que configuró para NCPA

Abra un navegador web y conéctese a la interfaz web de NCPA mediante la siguiente URL:
`https://<Dirección IP de NCPA>:5693/`

Aparecerá un mensaje de seguridad.
Esto es completamente normal y previsible. `NCPA` utiliza certificados autofirmados, ya que permiten cifrar la comunicación. Su navegador web le advierte porque desconoce el certificado.
Deberá hacer clic en "Avanzado" y luego en "Agregar excepción" o "Continuar a xxx" para poder usar la página de `NCPA`.

# Iniciar Monitoreo en Nagios Core
Una de las formas más fáciles de comenzar a monitorear usando comprobaciones activas es ejecutando el Asistente de configuración `NCPA`

### Descargando check_ncpa.py
Si está utilizando una instalación estándar de Nagios Core, comience descargando el check_en.py plugin en el directorio de plugins, normalmente `/usr/local/nagios/libexe`

# Crear la definición del comando check
Crear el check_ncpa comando en sus archivos de configuración para Nagios Core, normalmente se encuentran en `/usr/local/nagios/etc` usted puede tener un commands.cfg archivo en el que querrá poner este comando

```
definir comando {
    comando_nombre check_ncpa
    command_line $USER1$/check_ncpa.py -H $HOSTADDRESS$ $ARG1$
}
```
### Crear cheques de Nagios
Puede crear las comprobaciones en un archivo de configuración en /usr/local/nagios/etc. Para este ejemplo crearemos un archivo de configuración llamado ncpa.cfg con lo siguiente definido:
```
definir host {
    host_name NCPA 2 Anfitrión
    dirección 192.168.1.10
    check_command check_ncpa!-t 'mytoken' - P 5693 - Sistema/agente_versión
    max_check_intentos 5
    check_intervalo 5
    retry_intervalo 1
    check_período 24x7
    contactos nagiosadmin
    notificación_intervalo 60
    notificación_período 24x7
    notificaciones_habilitado 1
    icon_imagen ncpa.png
    statusmap_imagen ncpa.png
    registro 1
}

definir servicio {
    host_name NCPA 2 Anfitrión
    service_description Uso de CPU
    check_command check_ncpa!-t 'mytoken' -P 5693 -M cpu/porcentaje -w 20 -c 40 -q 'aggregate=avg'
    max_check_intentos 5
    check_intervalo 5
    retry_intervalo 1
    check_período 24x7
    notificación_intervalo 60
    notificación_período 24x7
    contactos nagiosadmin
    registro 1
}

definir servicio {
    host_name NCPA 2 Anfitrión
    service_description Uso de la memoria
    check_command check_ncpa!-t 'mytoken' - P 5693 -M memoria/virtual - w 50 - c 80 - u G
    max_check_intentos 5
    check_intervalo 5
    retry_intervalo 1
    check_período 24x7
    notificación_intervalo 60
    notificación_período 24x7
    contactos nagiosadmin
    registro 1
}

definir servicio {
    host_name NCPA 2 Anfitrión
    conteo de Procesos Service_Description
    check_command check_ncpa!-t 'mytoken' - P 5693 - Procesos M - w 150 - c 200
    max_check_intentos 5
    check_intervalo 5
    retry_intervalo 1
    check_período 24x7
    notificación_intervalo 60
    notificación_período 24x7
    contactos nagiosadmin
    registro 1
}
```
> [!NOTE]
> Reemplazar el -t 'mytoken' con tu propio token. Esto le dirá a Nagios que realice comprobaciones activas y creará un host llamado "NCPA 2 Host" con comprobaciones de Uso de CPU, Uso de Memoria y Recuento de Procesos.

### Reinicie Nagios y espere los controles activos
Reinicie el servicio de Nagios y debería ver aparecer `hosts/servicios` pendientes. Una vez que hagan sus comprobaciones iniciales, debería ver sus datos de NCPA en Nagios.

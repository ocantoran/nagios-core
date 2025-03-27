

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

> _[!NOTE]_
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


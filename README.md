## Border router thread

En este repositorio se describe el desarrollo de un BR formado por:

- ESP32 H2
- RPI 4

El ESP32 H2 funcionará como Radio Coprocessor (RCP) y la RPI como sistema host. 


## Configuración del entorno IDF
La configuración del entorno de desarrollo viene descrita en: 
https://docs.espressif.com/projects/esp-idf/en/latest/esp32h2/get-started/linux-macos-setup.html 



Para la instalación de python creo un entorno según las dependencias indicadas en el archivo ```esp-env.yml```.

Instalo las dependencias para MacOS con brew:

```
brew install cmake ninja dfu-util
```

Se obtiene el toolchain de IDF descargando los archivos de su git oficial:
```bash
mkdir -p ~/esp
cd ~/esp
git clone --single-branch --recursive https://github.com/espressif/esp-idf.git
```

Las herramientas de desarrollo se almacenarán en: ```~/esp/esp-idf```

Se configuran las herramientas de desarrollo:
```bash
cd ~/esp/esp-idf
./install.sh esp32h2
```

Se añade un alias a bash_profile para configurar las variables de entorno necesarias cada vez que se intente desarrollar con IDF:

```bash
alias get_idf='. $HOME/esp/esp-idf/export.sh'
```


Instalo la extensión de VS code de IDF. Configuro la extensión para que tome como python el python instalado en el entorno de anaconda:

```
cmd+shift+p --> ESP-IDF:Configure ESP-IDF Extension

Se selecciona el path de python: ...esp-env/python3
```

## Primer proyecto

Copio el ejemplo hello world. Dentro de ese proyecto configuro el target:

```bash
get_idf
idf.py set-target esp32h2
```

Se puede configurar el proyecto con:

```bash
idf.py menuconfig
```

Se compila 

```
idf.py build
```
Conecto el ordenador al puerto USB
Se flashea con:

```
idf.py -p 
```

Para obtener el resultado del programa por serie:

```bash
screen /dev/tty.usbmodem
```


## Configuración del firmware (ESP32)
Pruebo a compilar y cargar en el micro el ejemplo de espressif de RCP:
https://github.com/espressif/esp-idf/tree/master/examples/openthread/ot_rcp

Dentro del repositorio descargado configuramos el target y obtenemos idf:

```bash
get_idf
idf.py set-target esp32h2
```

Configuramos el proyecto:
```bash
idf.py menuconfig
```
Reviso la configuración y no cambio nada. Parece que todo es correcto.

Compilo el programa:
```bash
idf.py build
```
Se transfiere el programa:

```bash
idf.py -p /dev/tty.XXXXX  flash
```

UNA VEZ CARGADO SE CONECTA LA RPI AL ESP MEDIANTE SU PUERTO USB-C DENOMINADO UART (NO USB!)

## Configuración del host (RPI)
La configuración como host viene descrita en la página oficial de openthread:
https://openthread.io/guides/border-router/prepare?hl=es-419

Lo primero actualizo la RPI:
```
sudo apt-get update
sudo apt-get upgrade
```

El stack de thread se configurará en la raspberry como un contenedor docker. Para gestionar los contenedores se empleará Portainer.

```
docker volume create portainer_data
```


Se levanta el contenedor de portainer:
```
docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:lts
```
Una vez levantado el contenedor, los contenedores de la RPI se gestionarán desde:
```https://192.x.x.x:9443```




Descargo la imagen oficial de docker para el BR:

```bash
docker pull openthread/border-router:latest
```

Alternativa (esta incluye interfaz web para configurar los nodos):
```bash
docker pull openthread/otbr:latest
```

Creamos un archivo para las variables de entorno ```otbr-env.list```:
```
OT_RCP_DEVICE=spinel+hdlc+uart:///dev/ttyACM0?uart-baudrate=460800
OT_INFRA_IF=wlan0
OT_THREAD_IF=wpan0
OT_LOG_LEVEL=7
OTBR_WEB=1
OTBR_REST=1
FIREWALL=0
NAT64=0
DNS64=0
```

OJO: Hay que cambiar el baudrate a 460800, que es el baudrate que espressif pone por defecto en el ejemplo de RCP.

La configuración del baudrate del rcp se puede ver en el archivo: ```esp_ot_config.h```


Además, se crea el archivo otbr-web.conf:
```
[thread-web]
interface_name = wpan0
enabled = true
listen_address = "::"
listen_port = 80
```


Se ejcuta el contenedor:

```
docker run --name=otbr --detach --network=host --cap-add=NET_ADMIN --device=/dev/ttyACM0 --device=/dev/net/tun --volume=/var/lib/otbr:/data --volume=./otbr_config/:/etc/otbr --env-file=./otbr_config/otbr-env.list --restart=always openthread/border-router
```

Con la imagen openthread/otbr:

```
docker run --name=otbr --detach --network=host --cap-add=NET_ADMIN --device=/dev/ttyACM0 --device=/dev/net/tun --volume=/var/lib/otbr:/data --volume=./otbr_config/:/etc/otbr --env-file=./otbr_config/otbr-env.list --restart=always openthread/otbr
```


Si todo va bien deberíamos ver algo así en los logs:

```
00:00:00.085 [I] P-Netif-------: NAT64 CIDR updated to 192.168.255.0/24.
00:00:00.085 [I] P-Netif-------: Sent request#2 to delete route 192.168.255.0/24
00:00:00.085 [I] P-Netif-------: Deleting route for NAT64
00:00:00.085 [I] P-McastRtMgr--: Disable: OK
[DEBG]-BBA-----: BackboneAgent: HandleBackboneRouterState: state=1, mBackboneRouterState=0
00:00:00.085 [I] RouterTable---: Route table
00:00:00.085 [I] TrelDiscoverer: Registering service otTRELc26af120cab9423a._trel._udp
00:00:00.085 [I] TrelDiscoverer:     port:46709, ext-addr:c26af120cab9423a, ext-panid:dead00beef00cafe
00:00:00.086 [I] MulticastDns--: Adding host address 192.168.0.111
00:00:00.086 [I] MulticastDns--: Adding host address fe80:0:0:0:b662:f958:a91c:2a00
00:00:00.086 [I] P-Netif-------: Host netif is down
00:00:00.088 [I] P-Netif-------: Succeeded to process request#1
00:00:00.088 [W] P-Netif-------: Failed to process request#2: No such process
s6-rc: info: service otbr-agent successfully started
s6-rc: info: service legacy-services: starting
s6-rc: info: service legacy-services successfully started
00:00:00.922 [I] TrelDiscoverer: DNS-SD service registered successfully
00:00:00.923 [I] TrelPeerTable-: Added peer otTRELc26af120cab9423a, dnssd-state:resolving
00:00:00.923 [I] TrelDiscoverer: Peer otTRELc26af120cab9423a is this device itself
00:00:00.923 [I] TrelPeerTable-: Deleted peer otTRELc26af120cab9423a, dnssd-state:resolving
```


## Configuración del entorno gráfico web

Si queremos acceder a la web de configuración tenemos que abrir un tunel ssh y redireccionar el puerto 80 de la rpi a un puerto de nuestro localhost:

```
ssh -L 7586:127.0.0.1:80 pi@192.168.0.111
```

Después podemos acceder a la configuración desde el navegador con la ruta:

```
localhost:7586
```

## Referencias

Encuentro la siguiente información de interés

- Blog de Medium: https://medium.com/engineering-iot/building-your-own-thread-border-router-a-complete-diy-guide-8b308d27aa64 

- Repo del blog: https://github.com/psmgeelen/diythreadrouter?tab=readme-ov-file 
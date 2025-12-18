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


## Configuración del host (RPI)


## Referencias

Encuentro la siguiente información de interés

- Blog de Medium: https://medium.com/engineering-iot/building-your-own-thread-border-router-a-complete-diy-guide-8b308d27aa64 

- Repo del blog: https://github.com/psmgeelen/diythreadrouter?tab=readme-ov-file 
# Práctica 1.2 - DNS Linux
## Requisitos previos
### Instalación de VirtualBox Guest Additions
En el caso de querer utilizar carpetas compartidas o poder copiar-pegar texto entre la máquina virtual y el anfitrión, es necesario isntalar las Guest Additions.
En el caso de Debian, antes es necesario isntalar ciertos paquetes para que se puedan compilar los módulos del kernel necesarios.
Para ello, abrimos una terminal y ejecutamos:
```console
sudo apt install build-essential dkms
```
El siguiente paso es instalar las Guest Additions. Para ello, hacemos clic en "Dispositivos/Insertar imagen de CD  de los complementos de invitado" en el menú de la máquina virtual.
Abrimos la unidad de CD en la máquina virtual, y sobre el archivo `autorun.sh` hacemos click derecho, ejecutamos como un programa y seguimos las instrucciones.

## Instalación de servidor DNS

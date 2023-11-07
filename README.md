# Práctica 1.2 - DNS Linux
## Requisitos previos
### Instalación de VirtualBox Guest Additions
En el caso de querer utilizar carpetas compartidas o poder copiar-pegar texto entre la máquina virtual y el anfitrión, es necesario isntalar las Guest Additions.
En el caso de Debian, antes es necesario instalar ciertos paquetes para que se puedan compilar los módulos del kernel necesarios.
Para ello, abrimos una terminal y ejecutamos:
```console
sudo apt install build-essential dkms
```
El siguiente paso es instalar las Guest Additions. Para ello, hacemos clic en "Dispositivos/Insertar imagen de CD  de los complementos de invitado" en el menú de la máquina virtual.
Abrimos la unidad de CD en la máquina virtual, y sobre el archivo `autorun.sh` hacemos click derecho, ejecutamos como un programa y seguimos las instrucciones.

## Instalación de servidor DNS

Instalamos los paquetes `bind9` que contiene el servicio DNS, y `dnsutils` para poder utilizar la herramienta `dig`.
```console
sudo apt install bind9 dnsutils
```
Editamos los archivos de configuración para añadir las zonas al servidor DNS.
### /etc/bind/named.conf
```
include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
include "/etc/bind/named.conf.default-zones";
```
### /etc/bind/named.conf.local
```
zone "asircastelao.int" {
	type master;
	file "/var/lib/bind/db.asircastelao.int";
	allow-query {
		any;
		};
	};
```
### /etc/bind/named.conf.options
```
options {
	directory "/var/cache/bind";

	forwarders {
	 	8.8.8.8;
		1.1.1.1;
	 };
	 forward only;

	listen-on { any; };
	listen-on-v6 { any; };

	allow-query {
		any;
	};
};
```
### /var/lib/bind/db.asircastelao.int
```
$TTL 38400	; 10 hours 40 minutes
@		IN SOA	ns.asircastelao.int. some.email.address. (
				10000002   ; serial
				10800      ; refresh (3 hours)
				3600       ; retry (1 hour)
				604800     ; expire (1 week)
				38400      ; minimum (10 hours 40 minutes)
				)
@		IN NS	ns.asircastelao.int.
ns		IN A		172.28.5.1
test	IN A		172.28.5.4
web1    IN A        172.28.5.6
web2	IN A		172.28.5.7
intra	IN A		172.28.5.8
alias	IN CNAME	test
texto	IN TXT		mensaje
```
## Reiniciamos el servicio named
Para reiniciar el servicio DNS, ejecutamos:
```console
sudo systemctl restart bind9 
```
Podemos comprobar que está corriendo de forma satisfactoria (active - running) con el comando `systemctl status bind9`.
```console
ubuntu@ubuntu-VirtualBox:~$ sudo systemctl status bind9
● named.service - BIND Domain Name Server
     Loaded: loaded (/lib/systemd/system/named.service; enabled; preset: enable>
     Active: active (running) since Tue 2023-11-07 16:09:08 CET; 1h 5min ago
       Docs: man:named(8)
   Main PID: 745 (named)
     Status: "running"
      Tasks: 10 (limit: 4581)
     Memory: 13.3M
        CPU: 88ms
     CGroup: /system.slice/named.service
             └─745 /usr/sbin/named -f -u bind

nov 07 16:09:09 ubuntu-VirtualBox named[745]: none:99: 'max-cache-size 90%' - s>
nov 07 16:09:09 ubuntu-VirtualBox named[745]: obtaining root key for view _defa>
nov 07 16:09:09 ubuntu-VirtualBox named[745]: configuring command channel from >
nov 07 16:09:09 ubuntu-VirtualBox named[745]: configuring command channel from >
nov 07 16:09:09 ubuntu-VirtualBox named[745]: managed-keys-zone: Unable to fetc>
nov 07 16:09:09 ubuntu-VirtualBox named[745]: reloading configuration succeeded
nov 07 16:09:09 ubuntu-VirtualBox named[745]: scheduled loading new zones
nov 07 16:09:09 ubuntu-VirtualBox named[745]: any newly configured zones are no>
nov 07 16:09:09 ubuntu-VirtualBox named[745]: running
nov 07 16:09:09 ubuntu-VirtualBox named[745]: managed-keys-zone: Key 20326 for >
lines 1-22/22 (END)

```
## Comprobamos que el servicio funciona correctamente
Utilizamos la herramienta `dig` utilizando como servidor DNS la propia dirección de la máquina, y probamos a consultar la dirección 'test.asircastelao.int'.
Vemos que se nos devuelve la IP 172.28.5.4, por lo que el servicio DNS funciona correctamente.
```console
ubuntu@ubuntu-VirtualBox:~$ dig @10.0.2.15 test.asircastelao.int

; <<>> DiG 9.18.18-0ubuntu0.23.04.1-Ubuntu <<>> @10.0.2.15 test.asircastelao.int
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 24886
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 1986c1894bcd002901000000654a652663494a1cfaa256c4 (good)
;; QUESTION SECTION:
;test.asircastelao.int.		IN	A

;; ANSWER SECTION:
test.asircastelao.int.	38400	IN	A	172.28.5.4

;; Query time: 0 msec
;; SERVER: 10.0.2.15#53(10.0.2.15) (UDP)
;; WHEN: Tue Nov 07 17:26:14 CET 2023
;; MSG SIZE  rcvd: 94
```
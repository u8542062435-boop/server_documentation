# server_documentation
###En el primer dia de trabajo estoy instalando Ubuntu Server, para ello enumerare los comandos utilizados.
##CONFIGURACIÓN SERVIDOR

###Cambiar hostname
sudo hostnamectl set-hostname ls14

###Modificar fichero hosts
sudo nano /etc/hosts
172.30.20.55 ls14.lab14.lan ls14

###Verificar el FQDN
hostname -f

###Verificar si el FQDN es capaz de resolver la dirección Ip del Samba
ping -c2 ls14.lab14.lan

###Desactivar servicio systemd-resolved
sudo systemctl disable --now systemd-resolved

###Eliminar enlace simbólico al archivo /etc/resolv.conf
sudo unlink /etc/resolv.conf

###Creamos de nuevo el archivo /etc/resolv.conf
sudo nano /etc/resolv.conf

###Añadimos las siguientes líneas:
nameserver 172.30.20.55
nameserver 8.8.8.8
search lab14.lan

###Hacemos inmutable al archivo /etc/resolv.conf para que no pueda cambiar
sudo chattr +i /etc/resolv.conf

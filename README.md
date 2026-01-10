# server_documentation
### En el primer dia de trabajo estoy instalando Ubuntu Server, para ello enumerare los comandos utilizados.<br>
## CONFIGURACIÓN SERVIDOR

### Cambiar hostname<br>
` sudo hostnamectl set-hostname ls14
`
### Modificar fichero hosts<br>
sudo nano /etc/hosts<br>
172.30.20.55 ls14.lab14.lan ls14

### Verificar el FQDN<br>
hostname -f

### Verificar si el FQDN es capaz de resolver la dirección Ip del Samba<br>
ping -c2 ls14.lab14.lan

### Desactivar servicio systemd-resolved<br>
sudo systemctl disable --now systemd-resolved

### Eliminar enlace simbólico al archivo /etc/resolv.conf<br>
sudo unlink /etc/resolv.conf

### Creamos de nuevo el archivo /etc/resolv.conf<br>
sudo nano /etc/resolv.conf

### Añadimos las siguientes líneas:<br>
nameserver 172.30.20.55<br>
nameserver 8.8.8.8<br>
search lab14.lan<br>

### Hacemos inmutable al archivo /etc/resolv.conf para que no pueda cambiar<br>
sudo chattr +i /etc/resolv.conf<br>

<p align="center">
  <img src="Images/1.netplan.jpg" width="400">
</p>

<p align="center">
  <em>Netplan_Server</em>
</p>
<br>
<p align="center">
  <img src="Images/2.etc_hosts.jpg" width="500">
</p>

<p align="center">
  <em>etc_hosts</em>
</p>

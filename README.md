# server_documentation
### En el primer dia de trabajo estoy instalando Ubuntu Server, para ello enumerare los comandos utilizados.<br>
## CONFIGURACIÓN SERVIDOR

### Cambiar hostname<br>
`sudo hostnamectl set-hostname ls14
`
### Modificar fichero hosts <br>
`sudo nano /etc/hosts
`<br>172.30.20.55 ls14.lab14.lan ls14

### Verificar el FQDN<br>
`hostname -f`

### Verificar si el FQDN es capaz de resolver la dirección Ip del Samba<br>
`ping -c2 ls14.lab14.lan`

### Desactivar servicio systemd-resolved<br>
`sudo systemctl disable --now systemd-resolved`

### Eliminar enlace simbólico al archivo /etc/resolv.conf<br>
`sudo unlink /etc/resolv.conf`

### Creamos de nuevo el archivo /etc/resolv.conf<br>
`sudo nano /etc/resolv.conf`

### Añadimos las siguientes líneas:<br>
nameserver 172.30.20.55<br>
nameserver 8.8.8.8<br>
search lab14.lan<br>

### Hacemos inmutable al archivo /etc/resolv.conf para que no pueda cambiar<br>
`sudo chattr +i /etc/resolv.conf`<br>

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
## INSTALACIÓN SAMBA

Actualizar el índice de paquetes<br>
`sudo apt update`

Instalar samba con sus paquetes y dependencias<br>
`sudo apt install -y acl attr samba samba-dsdb-modules samba-vfs-modules smbclient winbind libpam-winbind libnss-winbind libpam-krb5 krb5-config krb5-user dnsutils chrony net-tools`

LAB14.LAN
ls14.lab14.lan
ls14.lab14.lan

Detener y deshabilitar los servicios que el servidor de Active Directory de Samba no requiere (smbd, nmbd y winbind)<br>
`sudo systemctl disable --now smbd nmbd winbind`

El servidor solo necesita samba-ac-dc para funcionar como Active Directory y controlador de dominio.<br>
`sudo systemctl unmask samba-ad-dc`<br>
`sudo systemctl enable samba-ad-dc`

## CONFIGURACIÓN SAMBA ACTIVE DIRECTORY

Crear una copia de seguridad del archivo /etc/samba/smb.conf<br>
`sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.orig`

Ejecutar el comando samba-tool para comenzar a aprovisionar Samba Active Directory.<br>
`sudo samba-tool domain provision`

Realm: CLOCKWORK.LOCAL<br>
Domain: CLOCKWORK<br>
Server Role: dc<br>
DNS backend: SAMBA_INTERNAL<br>
DNS forwarder IP address: 8.8.8.8<br>

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

LAB14.LAN<br>
ls14.lab14.lan<br>
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

Realm: LAB14.LAN<br>
Domain: LAB14<br>
Server Role: dc<br>
DNS backend: SAMBA_INTERNAL<br>
DNS forwarder IP address: 8.8.8.8<br>

Crear copia de seguridad de la configuración predeterminada de Kerberos.<br>
`sudo mv /etc/krb5.conf /etc/krb5.conf.orig`

Reemplazar con el archivo /var/lib/samba/private/krb5.conf.<br>
`sudo cp /var/lib/samba/private/krb5.conf /etc/krb5.conf`

Iniciar servicio Samba Active Directory samba-ad-dc<br>
`sudo systemctl start samba-ad-dc`

Comprobar servicio<br>
`sudo systemctl status samba-ad-dc`

CONFIGURAR SINCRONIZACIÓN DE TIEMPO<br>
Samba Active Directory depende del protocolo Kerberos, y el protocolo Kerberos requiere que los tiempos del servidor AD y de la estación de trabajo estén sincronizados. Para garantizar una sincronización de tiempo adecuada, también deberemos configurar un servidor de Protocolo de tiempo de red (NTP) en Samba.
Los beneficios de la sincronización de tiempo de AD incluyen la prevención de ataques de repetición y la resolución de conflictos de replicación de AD.

Cambiar el permiso y la propiedad predeterminados del directorio /var/lib/samba/ntp_signd/ntp_signed. El usuario/grupo chrony debe tener permiso de lectura en el directorio ntp_signed.<br>
`sudo chown root:_chrony /var/lib/samba/ntp_signd/`<br>
`sudo chmod 750 /var/lib/samba/ntp_signd/`

Modificar el archivo de configuración /etc/chrony/chrony.conf para habilitar el servidor NTP de chrony y apuntar a la ubicación del socket NTP a /var/lib/samba/ntp_signd.<br>
`sudo nano /etc/chrony/chrony.conf`

bindcmdaddress 172.30.20.55<br>
allow 172.30.20.0/24<br>
ntpsigndsocket /var/lib/samba/ntp_signd

Reiniciar y verificar el servicio chronyd en el servidor Samba AD.<br>
`sudo systemctl restart chronyd`
`sudo systemctl status chronyd`

VERIFICAR SAMBA ACTIVE DIRECTORY<br>

Verificar nombres de dominio<br>
`host -t A ls14.lab14.lan`<br>
`host -t A ls14.lab14.lan`

Verificar que los registros de servicio kerberos y ldap apunten al FQDN de su servidor Samba Active Directory.<br>
`host -t SRV _kerberos._udp.lab14.lan`<br>
`host -t SRV _ldap._tcp.lab14.lan`

Verificar los recursos predeterminados disponibles en Samba Active Directory.<br>
`smbclient -L lab14.lan -N`

Comprobar autenticación en el servidor de Kerberos mediante el administrador de usuarios<br>
`kinit administrator@LAB14.LAN`
`klist`

Iniciar sesión en el servidor a través de smb<br>
`sudo smbclient //localhost/netlogon -U 'administrator'`

Cambiar contraseña usuario administrator<br>
`sudo samba-tool user setpassword administrator`

Verificar la integridad del archivo de configuración de Samba.<br>
`testparm`

Verificar funcionamiento WINDOWS AD DC 2008<br>
`sudo samba-tool domain level show`

Crear usuario SAMBA AD<br>
`sudo samba-tool user create George`

Listar usuarios SAMBA AD<br>
`sudo samba-tool user list`

Eliminar un usuario<br>
`samba-tool user delete <nombre_del_usuario>`

Listar equipos SAMBA AD<br>
`sudo samba-tool computer list`

Eliminar equipo SAMBA AD<br>
`sudo samba-tool computer delete <nombre_del_equipo>`

Crear grupo<br>
`samba-tool group add <nombre_del_grupo>`

Listar grupos<br>
`samba-tool group list`

Listar miembros de un grupo<br>
`samba-tool group listmembers 'Domain Admins'`

Agregar un miembro a un grupo<br>
`samba-tool group addmembers <nombre_del_grupo> <nombre_del_usuario>`

Eliminar un miembro de un grupo<br>
`samba-tool group removemembers <nombre_del_grupo> <nombre_del_usuario>`

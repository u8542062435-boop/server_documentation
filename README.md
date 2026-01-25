# Guía de Configuración de Ubuntu Server y Unión de Clientes Ubuntu y Windows al Dominio

## SERVER CONFIGURATION

### Change hostname<br>
`sudo hostnamectl set-hostname ls14`

### Modify file hosts <br>

`sudo nano /etc/hosts`<br>
172.30.20.55 ls14.lab14.lan ls14

<p align="center">
  <img src="Images/2.etc_hosts.jpg" width="500">
</p>

<p align="center">
  <em>etc_hosts</em>
</p>

### Verify the FQDN<br>
`hostname -f`

### Verify if the FQDN is able to solv the Samba Ip address<br>
`ping -c2 ls14.lab14.lan`

### Disable systemd-resolved<br>
`sudo systemctl disable --now systemd-resolved`

### eliminate and unlink /etc/resolv.conf<br>
`sudo unlink /etc/resolv.conf`

### Create the new file /etc/resolv.conf<br>
`sudo nano /etc/resolv.conf`

### We add the next lines:<br>
nameserver 172.30.20.55<br>
nameserver 8.8.8.8<br>
search lab14.lan<br>

<p align="center">
  <img src="Images/6.resolv.jpg" width="400">
</p>

<p align="center">
  <em>Resolv_conf</em>
</p>

### We make the /etc/resolv.conf file immutable so it cannot be changed.<br>
`sudo chattr +i /etc/resolv.conf`<br>

<p align="center">
  <img src="Images/1.netplan.jpg" width="400">
</p>

<p align="center">
  <em>Netplan_Server</em>
</p>
<br>

## SAMBA INSTALL

### Update the package index<br>
`sudo apt update`

### Install Samba with its packages and dependencies<br>
`sudo apt install -y acl attr samba samba-dsdb-modules samba-vfs-modules smbclient winbind libpam-winbind libnss-winbind libpam-krb5 krb5-config krb5-user dnsutils chrony net-tools`

LAB14.LAN<br>
ls14.lab14.lan<br>
ls14.lab14.lan

### Stop and disable the services that the Samba Active Directory server does not require  (smbd, nmbd y winbind)<br>
`sudo systemctl disable --now smbd nmbd winbind`

### The server only needs samba-ac-dc to function as an Active Directory and controller domain.<br>
`sudo systemctl unmask samba-ad-dc`<br>
`sudo systemctl enable samba-ad-dc`

## SAMBA ACTIVE DIRECTORY CONFIGURATION

### Create a backup of /etc/samba/smb.conf<br>
`sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.orig`

### Run the samba-tool command to begin provisioning Samba Active Directory.<br>
`sudo samba-tool domain provision`

Realm: LAB14.LAN<br>
Domain: LAB14<br>
Server Role: dc<br>
DNS backend: SAMBA_INTERNAL<br>
DNS forwarder IP address: 8.8.8.8<br>

### Create a backup of the default Kerberos configuration.<br>
`sudo mv /etc/krb5.conf /etc/krb5.conf.orig`

### Replace with the file /var/lib/samba/private/krb5.conf.<br>
`sudo cp /var/lib/samba/private/krb5.conf /etc/krb5.conf`

### Start Samba Active Directory service samba-ad-dc<br>
`sudo systemctl start samba-ad-dc`

### Test service<br>
`sudo systemctl status samba-ad-dc`<br>

<p align="center">
  <img src="Images/8.samba_status.jpg" width="500">
</p>

<p align="center">
  <em>Samba_file</em>
</p>

## SETTING TIME SYNCHRONIZATION<br>

Samba Active Directory relies on the Kerberos protocol, and Kerberos requires
that the times of the AD server and the workstation be synchronized.

To ensure proper time synchronization, we must also configure a network Time
Protocol (NTP) server in Samba.

### Change the default permissions and ownership of the /var/lib/samba/ntp_signd/ntp_signed. The chrony user/group must have read permissions in the ntp_signed.<br>
`sudo chown root:_chrony /var/lib/samba/ntp_signd/`<br>
`sudo chmod 750 /var/lib/samba/ntp_signd/`

### Modify the /etc/chrony/chrony.conf configuration file to enable the chrony NTP server and point the NTP socket location to /var/lib/samba/ntp_signd.<br>
`sudo nano /etc/chrony/chrony.conf`

bindcmdaddress 172.30.20.55<br>
allow 172.30.20.0/24<br>
ntpsigndsocket /var/lib/samba/ntp_signd

<p align="center">
  <img src="Images/5.chrony.jpg" width="500">
</p>

<p align="center">
  <em>Chrony</em>
</p>


### Restart and verify the chronyd service on the Samba AD server.<br>
`sudo systemctl restart chronyd`
`sudo systemctl status chronyd`


## VERIFY SAMBA ACTIVE DIRECTORY<br>


### Verify domain names<br>
`host -t A ls14.lab14.lan`<br>
`host -t A ls14.lab14.lan`<br>


### Verify that the Kerberos and LDAP service records point to the FQDN of your Samba Active Directory server<br>
`host -t SRV _kerberos._udp.lab14.lan`<br>
`host -t SRV _ldap._tcp.lab14.lan`


### Verify the default resources available in Samba Active Directory.<br>
`smbclient -L lab14.lan -N`


### Verify authentication on the Kerberos server using the user manager<br>
`kinit administrator@LAB14.LAN`
`klist`


### Log in to the server via SMB<br>
`sudo smbclient //localhost/netlogon -U 'administrator'`


### Change the administrator user password<br>
`sudo samba-tool user setpassword administrator`


### Verify the integrity of the Samba configuration file.<br>
`testparm`


### Verify the operation of Windows Active Directory Domain Controller 2008<br>
`sudo samba-tool domain level show`


### Create user SAMBA AD<br>
`sudo samba-tool user create George`


### List SAMBA AD users<br>
`sudo samba-tool user list`


### Delete a user<br>
`samba-tool user delete <nombre_del_usuario>`


### List SAMBA AD computers<br>
`sudo samba-tool computer list`


### Delete SAMBA AD computer<br>
`sudo samba-tool computer delete <nombre_del_equipo>`


### Create a group<br>
`samba-tool group add <nombre_del_grupo>`


### List groups<br>
`samba-tool group list`


### List group members<br>
`samba-tool group listmembers 'Domain Admins'`


### Add a member to a group<br>
`samba-tool group addmembers <nombre_del_grupo> <nombre_del_usuario>`


### Remove a member from a group<br>
`samba-tool group removemembers <nombre_del_grupo> <nombre_del_usuario>`

[https://github.com/u8542062435-boop/Make-your-Ubuntu-Server-a-functional-router](https://github.com)


## UBUNTU CLIENT CONFIGURATION


### Change hostname<br>
`sudo hostnamectl set-hostname LSC14`
`hostname -f`


### Configure the /etc/hosts file<br>
`sudo nano /etc/hosts`

192.168.1.8     lab14.lan lab14
192.168.1.8     ls14.lab14.lan ls14


### Check connectivity<br>
`ping -c2 lab14.lan`


### Install NTPDATE<br>
`sudo apt-get install ntpdate`
`sudo ntpdate -q lab14.lan`
`sudo ntpdate lab14.lan`


### Install required packages<br>
`sudo apt-get install samba krb5-config krb5-user winbind libpam-winbind libnss-winbind`<br>

LAB14.LAN<br>
ls14.lab14.lan<br>
ls14.lab14.lan<br>


### Verify authentication on the Kerberos server using the user administrator<br>
`kinit administrator@LAB14.LAN`<br>
`klist`<br>


### Move smb.conf file and create a backup<br>
`mv /etc/samba/smb.conf /etc/samba/smb.conf.initial`


### Create an empty smb.conf file<br>
`nano /etc/samba/smb.conf`

[global]<br>
        workgroup = LAB14<br>
        realm = LAB14.LAN<br>
        netbios name = LSC14<br>
        security = ADS<br>
        dns forwarder = 172.30.20.55<br>

idmap config * : backend = tdb<br>
idmap config *:range = 50000-1000000<br>

   template homedir = /home/%D/%U<br>
   template shell = /bin/bash<br>
   winbind use default domain = true<br>
   winbind offline logon = false<br>
   winbind nss info = rfc2307<br>
   winbind enum users = yes<br>
   winbind enum groups = yes<br>

  vfs objects = acl_xattr<br>
  map acl inherit = Yes<br>
  store dos attributes = Yes<br>

<p align="center">
  <img src="Images/9.smb_conf.jpg" width="500">
</p>

<p align="center">
  <em>smb_conf</em>
</p>

### Restart all Samba daemons<br>
`sudo systemctl restart smbd nmbd`


### Stop unnecessary services<br>
`sudo systemctl stop samba-ad-dc`


### Enable Samba services<br>
`sudo systemctl enable smbd nmbd`


### Join Ubuntu Desktop to SAMBA AD DC<br>
`sudo net ads join -U administrator`


### List SAMBA AD computers<br>
`sudo samba-tool computer list`


## CONFIGURE AD ACCOUNT AUTHENTICATION<br>

### Edit the Name Service Switch (NSS) configuration file<br>
`sudo nano /etc/nsswitch.conf`

passwd:       compat winbind<br>
group:        compat winbind<br>
shadow:       compat winbind<br>
hosts:        files dns<br>

<p align="center">
  <img src="Images/10.nsswitch_conf.jpg" width="500">
</p>

<p align="center">
  <em>nsswitch_conf</em>
</p>

### Restart the Winbind service<br>
`sudo systemctl restart winbind`

### Check if Ubuntu Desktop was integrated into the domain<br>
`wbinfo -u`<br>
`wbinfo -g`<br>

### Verify the Winbind NSS module using the getent command<br>
`sudo getent passwd | grep administrator`<br>
`sudo getent group | grep 'domain admins'`<br>
`id administrator`<br>

### Configure pam-auth-update to authenticate with domain accounts and automatically create home directories<br>
`sudo pam-auth-update`

### Edit the /etc/pam.d/common-account file to automatically create home directories<br>
`nano /etc/pam.d/common-account`

### Add the following at the end of the file:<br>

session    required    pam_mkhomedir.so    skel=/etc/skel/    umask=0022

<p align="center">
  <img src="Images/4.etc_pam.jpg" width="500">
</p>

<p align="center">
  <em>common_account</em>
</p>

### Authenticate with a Samba4 AD account<br>
`su administrator`

<p align="center">
  <img src="Images/3.administrator.jpg" width="500">
</p>

<p align="center">
  <em>su_administrator</em>
</p>

### Add domain account with root privileges<br>
`sudo usermod -aG sudo administrator`

### Authenticate via GUI<br>
`administrator@lab14.lan`

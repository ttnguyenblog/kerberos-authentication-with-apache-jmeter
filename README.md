# Using Kerberos authentication with Apache JMeter

## 1. Prepare

- VM Ubuntu 20.04
- Domain Controller (domain.com.vn)
- Windows 10 (install JMeter)

## 2. Install

### 2.1 VM Ubuntu 20.04

- Configure Apache Server:
```bash
Configure SSH:

sudo apt update
sudo apt install openssh-server

sudo systemctl enable ssh
sudo systemctl start ssh

sudo ufw allow ssh
sudo ufw enable
sudo ufw status

sudo apt-get update
sudo apt-get install apache2


sudo systemctl start apache2
sudo systemctl status apache2

sudo ufw status
sudo ufw disable
sudo ufw status
```
- Configure Kerberos:
```bash
[libdefaults]
        default_realm = TECHCOMBANK.COM.VN

# The following krb5.conf variables are only for MIT Kerberos.
        kdc_timesync = 1
        ccache_type = 4
        forwardable = true
        proxiable = true

# The following encryption type specification will be used by MIT Kerberos
# if uncommented.  In general, the defaults in the MIT Kerberos code are
# correct and overriding these specifications only serves to disable new
# encryption types as they are added, creating interoperability problems.
#
# The only time when you might need to uncomment these lines and change
# the enctypes is if you have local software that will break on ticket
# caches containing ticket encryption types it doesn't know about (such as
# old versions of Sun Java).

#       default_tgs_enctypes = des3-hmac-sha1
#       default_tkt_enctypes = des3-hmac-sha1
#       permitted_enctypes = des3-hmac-sha1

# The following libdefaults parameters are only for Heimdal Kerberos.
        fcc-mit-ticketflags = true

[realms]
        TECHCOMBANK.COM.VN = {
                kdc = TCB-AD-01.techcombank.com.vn
                admin_server = TCB-AD-01.techcombank.com.vn
                default_domain = techcombank.com.vn
        }

[domain_realm]
        .TCB-AD-01.techcombank.com.vn = TECHCOMBANK.COM.VN
        TCB-AD-01.techcombank.com.vn = TECHCOMBANK.COM.VN

kinit kdc@techcombank.com.vn
klist
```

- Install libapache2-mod-auth-kerb
```bash
sudo apt-get install libapache2-mod-auth-kerb

vi /etc/apache2/sites-avaiable/000-default.conf

service apache2 restart
```
- Configure /etc/apache2/sites-avaiable/000-default.conf
```bash
<VirtualHost *:80>
	# The ServerName directive sets the request scheme, hostname and port that
	# the server uses to identify itself. This is used when creating
	# redirection URLs. In the context of virtual hosts, the ServerName
	# specifies what hostname must appear in the request's Host: header to
	# match this virtual host. For the default virtual host (this file) this
	# value is not decisive as it is used as a last resort host regardless.
	# However, you must set it for any further virtual host explicitly.
	#ServerName www.example.com

	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/html

	# Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
	# error, crit, alert, emerg.
	# It is also possible to configure the loglevel for particular
	# modules, e.g.
	#LogLevel info ssl:warn

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined

	# For most configuration files from conf-available/, which are
	# enabled or disabled at a global level, it is possible to
	# include a line for only one particular virtual host. For example the
	# following line enables the CGI configuration for this host only
	# after it has been globally disabled with "a2disconf".
	#Include conf-available/serve-cgi-bin.conf
 
 	<Location />
		AuthType Kerberos
		AuthName "Kerberos authenticated intranet with mod_auth_kerb"

		KrbAuthRealms TECHCOMBANK.COM.VN
		KrbServiceName HTTP/kerberos.techcombank.com.vn
		Krb5Keytab /etc/kerberos.keytab
		KrbMethodNegotiate On
		KrbMethodK5Passwd On
		
		require valid-user
	</Location>
 
</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```
## 2.2 Domain Controller
```bash
setspn -A HTTP/apacheserver.techcombank.com.vn kdc
setspn -L kdc
ktpass /princ HTTP/apacheserver.techcombank.com.vn@TECHCOMBANK.COM.VN /mapuser kdc@techcombank.com.vn /pass admin@123 /out c:kerberos.keytab KRBS_NT_PRINCIPAL /crypto ALL
```
## 2.3 Window 10 (JMeter)

https://ttnguyen.net/cai-dat-jmeter-tren-windows/

# Reference

- [Using Kerberos authentication with Apache JMeter](https://www.robin-gueldenpfennig.de/2022/05/using-kerberos-authentication-with-apache-jmeter/)
- [All About Kerberos “The Three Headed Dog” with respect to IIS and Sql](https://learn.microsoft.com/en-us/archive/blogs/chiranth/all-about-kerberos-the-three-headed-dog-with-respect-to-iis-and-sql)
- [HTTP Authorization Manager](https://jmeter.apache.org/usermanual/component_reference.html#HTTP_Authorization_Manager)
- [Installing mod_auth_kerb](https://modauthkerb.sourceforge.net/install.html)
- [Configure Apache to use Kerberos authentication](https://edzeame.wordpress.com/2022/11/02/configure-apache-to-use-kerberos-authentication/)
- Video: [Kerberos authentication with Apache Server](https://www.youtube.com/watch?v=i5efYHYZlyQ)




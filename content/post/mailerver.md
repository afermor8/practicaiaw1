---
title: "Práctica: Servidor de correos"
date: 2023-06-05T20:03:42+02:00
draft: true
tags: ["SRI+HLC"]
bigimg: [{src: "/img/triangle.jpg", desc: "Triangle"}, {src: "/img/sphere.jpg", desc: "Sphere"}, {src: "/img/hexagon.jpg", desc: "Hexagon"}]
---

**Instala y configura de manera adecuada el servidor de correos en tu VPS. El nombre del servidor de correo será mail.tudominio.es (este es el nombre que deberá aparecer en el registro MX).**

```bash
ssh arantxa@217.160.225.205
```

```bash
sudo apt update
sudo apt upgrade
sudo apt install postfix bsd-mailx
```

Seleccionar la opción Internet Site y poner como nombre de dominio afm-tars.es.

Vamos a la web de IONOS para añadir un registro CNAME con el nombre **mail** que apunte al nombre de mi máquina (**walle**). Y creo un registro MX con el nombre del servidor de correo.

![registros](/img/mailserver/1.png)

![registros](/img/mailserver/2.png)

![CNAME](/img/mailserver/3.png)

![MX](/img/mailserver/6.png)

Agrego un registro SPF para evitar el spam.

![SPF](/img/mailserver/4.png)

Agrego en las políticas de firewall el puerto 25 para que se permita el envío de correos.

![port25](/img/mailserver/7.png)

## Gestión de correos desde el servidor

**El envío y recepción se hará desde el servidor usando la herramienta mail.**

### **Tarea 1: Documenta una prueba de funcionamiento, donde envíes desde tu servidor local al exterior. Muestra el log donde se vea el envío. Muestra el correo que has recibido. Muestra el registro SPF.**

Envío un correo desde mi VPS y compruebo el log para ver que se ha enviado correctamente.

![log](/img/mailserver/8.png)

Compruebo si lo he recibido:

![correo](/img/mailserver/9.png)

Compruebo el mensaje original para ver si ha funcionado el registro SPF.

![original](/img/mailserver/10.png)

![original](/img/mailserver/11.png)

### **Tarea 2: Documenta una prueba de funcionamiento, donde envíes un correo desde el exterior (gmail, hotmail,…) a tu servidor local. Muestra el log donde se vea el envío. Muestra cómo has leído el correo. Muestra el registro MX de tu dominio.**

He respondido al mensaje que recibí en mi Gmail.

![respuesta](/img/mailserver/12.png)

Miro el log para comprobar que he recibido el correo.

![log](/img/mailserver/17.png)

Leo el correo recibido con `mail` desde mi usuario:

![mail](/img/mailserver/13.png)

![mail](/img/mailserver/14.png)

Ya expliqué anteriormente cómo creé el registro. Para ver el registro MX de mi dominio desde mi servidor he usado `dig MX afm-tars.es`.

![registro-MX](/img/mailserver/16.png)

## Uso de alias y redirecciones

### **Tarea 3: Uso de alias y redirecciones.**

**Vamos a comprobar como los procesos del servidor pueden mandar correos para informar sobre su estado. Por ejemplo cada vez que se ejecuta una tarea cron podemos enviar un correo informando del resultado. Normalmente estos correos se mandan al usuario root del servidor, para ello:**

```bash
crontab -e
```

**E indico dónde se envía el correo:**

```txt
MAILTO = root
```

**Puedes poner alguna tarea en el cron para ver como se mandan correo.**
**Posteriormente usando alias y redirecciones podemos hacer llegar esos correos a nuestro correo personal.**
**Configura el cron para enviar correo al usuario root. Comprueba que están llegando esos correos al root. Crea un nuevo alias para que se manden a un usuario sin privilegios. Comprueba que llegan a ese usuario. Por último crea una redirección para enviar esos correo a tu correo personal (gmail,hotmail,…).**

Agrego a crontab lo siguiente:

```bash
arantxa@walle:~$ crontab -e
```

```txt
MAILTO = root
* * * * * echo Prueba de envío con crontab
```

La tarea mandará un correo a root cada vez que se ejecute el echo, en este caso, cada minuto.

Entramos con el usuario root a mail para ver los mensajes enviados.

```bash
arantxa@walle:~$ sudo su
root@walle:/home/arantxa# mail
Mail version 8.1.2 01/15/2001.  Type ? for help.
"/var/mail/root": 3 messages 2 new 3 unread
 U  1 root@afm-tars.es   Tue Jun  6 11:14   23/755   Cron <arantxa@walle> echo Prueba de envío con crontab
>N  2 root@afm-tars.es   Tue Jun  6 11:15   22/745   Cron <arantxa@walle> echo Prueba de envío con crontab
 N  3 root@afm-tars.es   Tue Jun  6 11:16   22/745   Cron <arantxa@walle> echo Prueba de envío con crontab
& 
```

![](/img/mailserver/18.png)

Usando un alias, voy a hacer que los correo también le lleguen a mi usuario **arantxa**. Para ello modifico /etc/aliases con el siguiente contenido.

```bash
sudo nano /etc/aliases
```

```conf
# See man 5 aliases for format
postmaster: root
root: arantxa
```

Ejecuto este comando para actualizar los cambios.

```bash
sudo newaliases
```

Compruebo si ya llegan lo correos a mi usuario **arantxa**.

```bash
arantxa@walle:~$ mail
Mail version 8.1.2 01/15/2001.  Type ? for help.
"/var/mail/arantxa": 2 messages 1 new 2 unread
 U  1 root@afm-tars.es   Tue Jun  6 11:22   23/755   Cron <arantxa@walle> echo Prueba de envío con crontab
>N  2 root@afm-tars.es   Tue Jun  6 11:23   22/745   Cron <arantxa@walle> echo Prueba de envío con crontab
& 
```

Al abrir el mensaje vemos que este va dirigido a root (To: root@afm-tars.es), pero le ha llegado a mi usuario también, lo que indica que el alias ha funcionado.

![correo](/img/mailserver/19.png)

Creo en el home de mi usuario el fichero *.forward*, cuyo contenido será mi correo personal de Gmail.

```bash
sudo nano /home/arantxa/.forward
```

```bash
ara.fer.mor@gmail.com
```

Compruebo que me llegan los correos a Gmail.

![correos](/img/mailserver/20.png)


## Para asegurar el envío

### **Tarea 4 (No obligatoria): Configura de manera adecuada DKIM en tu sistema de correos. Comprueba el registro DKIM en la página https://mxtoolbox.com/dkim.aspx. Configura postfix para que firme los correos que envía. Manda un correo y comprueba la verificación de las firmas en ellos.**

Instalo los paquetes necesarios.

```bash
sudo apt install opendkim opendkim-tools
```

Agrego el usuario **postfix** al grupo **opendkim**.

```bash
sudo gpasswd -a postfix opendkim
```

Configuro DKIM en el fichero /etc/opendkim.conf

```bash
sudo nano /etc/opendkim.conf
```

Sin los comentario se quedaría de la siguiente forma:

```conf
Syslog              yes
SyslogSuccess       yes

Canonicalization    relaxed/simple
Mode                sv
OversignHeaders     From

UserID              opendkim
UMask               007

Socket              local:/var/spool/postfix/opendkim/opendkim.sock

PidFile             /run/opendkim/opendkim.pid

TrustAnchorFile     /usr/share/dns/root.key

ExternalIgnoreList      refile:/etc/opendkim/TrustedHosts
InternalHosts           refile:/etc/opendkim/TrustedHosts
KeyTable                refile:/etc/opendkim/KeyTable
SigningTable            refile:/etc/opendkim/SigningTable
```

Creo el directrio opendkmi, edito el fichero /etc/opendkim/TrustedHosts e incluyo una lista de todos los nombres e IPs en los que se confía directamente.

```bash
sudo mkdir /etc/opendkim
sudo nano /etc/opendkim/TrustedHosts
```

```txt
127.0.0.1
127.0.1.1
217.160.225.205
::1
localhost
walle
*.afm-tars.es
```

Creo el directorio donde irán guardadas las claves y lo protejo.

```bash
sudo mkdir -p /etc/opendkim/keys/afm-tars.es/
sudo chown opendkim. /etc/opendkim/keys/afm-tars.es/
sudo chmod 0710 /etc/opendkim/keys/afm-tars.es/
```

Utilizo opendkim-genkey para crear la clave privada y un fichero que contiene el regitro que debemos agregar a nuestro DNS con la clave pública.

```bash
sudo opendkim-genkey -D /etc/opendkim/keys/afm-tars.es/ -d afm-tars.es -s dkim-pass
```

Esto genera dkim-pass.private y dkim-pass.txt (el primero la clave privada y el segundo el txt de la clave pública a añadir al DNS).

```bash
arantxa@walle:~$ sudo ls -l /etc/opendkim/keys/afm-tars.es/
total 8
-rw------- 1 root root 1675 Jun  6 12:56 dkim-pass.private
-rw------- 1 root root  511 Jun  6 12:56 dkim-pass.txt
```

Cambio el propietario de la clave privada a **opendkim**.

```bash
sudo chown opendkim:opendkim /etc/opendkim/keys/afm-tars.es/dkim-pass.private
```

Modifico los ficheros **SigningTable** y **KeyTable**.

```bash
sudo nano /etc/opendkim/SigningTable
```

```txt
*@afm-tars.es dkim-pass._domainkey.afm-tars.es
```

```bash
sudo nano /etc/opendkim/KeyTable
```

```txt
dkim-pass._domainkey.afm-tars.es afm-tars.es:dkim-pass:/etc/opendkim/keys/afm-tars.es/dkim-pass.private
```

Agrego la clave pública en el DNS. Para ello voy a la web de IONOS > Dominios y SSL > DNS y creo un registro TXT con nombre **dkim-pass._domainkey** y el valor que teníamos en el fichero, que es el siguiente:

```txt
v=DKIM1; h=sha256; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA5fcqeqP5oOPPBFq8bHsQeeQoo9SDlXGC09m4LJCaDMmgAsVFhGv1AecZWw25usaRRNXgrYqaYvsbWiBpWmBI4p9gl4RkAhOGwzERWzIlmukCTcf6vqF22wnGKrYMhkmJhqtJYZEKaWTFJvOUFilxTXUkG5KHR+YwO3FM+ChwuGveJs0o92OP9Jse87VpMYuca+Wk2RnhEnbLqnL6EOyJRcT5ecRIKE1lF76k7CCO3Xv9QP1TZ2Ai7cs90XTSerFiFPJJYZCFVUIiyFln1B5XC2e3bD7z6RG15EwwvOITYNYKBhYNYuKU/iQ3zEnNarqinAJ+EoTmdhDJvpx20SPc+wIDAQAB
```

![registro-dkim](/img/mailserver/21.png)

Entro en la web https://mxtoolbox.com/dkim.aspx para comprobar el registro DKIM.

![mxtoolbox](/img/mailserver/22.png)

![mxtoolbox](/img/mailserver/23.png)

También se puede comprobar de la siguiente forma:

```bash
arantxa@walle:~$ sudo opendkim-testkey -d afm-tars.es -s dkim-pass -vvv
opendkim-testkey: using default configfile /etc/opendkim.conf
opendkim-testkey: checking key 'dkim-pass._domainkey.afm-tars.es'
opendkim-testkey: key not secure
opendkim-testkey: key OK
```

Por último añadimos a la configuración de Postfix lo siguiente para que firme los correos al enviarlos.

```bash
sudo nano /etc/postfix/main.cf
```

```conf
#DKIM
milter_default_action = accept
milter_protocol = 6
smtpd_milters = local:/opendkim/opendkim.sock
non_smtpd_milters = $smtpd_milters
```

Reiniciar ambos servicios.

```bash
sudo systemctl restart opendkim postfix
```

Para que no de fallo al enviar un correo de prueba primero hay crear el directorio donde se creará el socket de OpenDKIM y modificar el propietario:

```bash
sudo mkdir /var/spool/postfix/opendkim
sudo chown opendkim:postfix /var/spool/postfix/opendkim
```

Ahora sí envío un correo a mi correo personal de Gmail desde mi VPS.

```bash
arantxa@walle:~$ mail ara.fer.mor@gmail.com
Subject: Prueba Dkim
Esto es una prueba de envío usando Dkim
Cc: 
```

Compruebo en el log que se ha enviado.

```bash
arantxa@walle:~$ sudo tail /var/log/mail.log
Jun  6 17:30:40 walle opendkim[18320]: OpenDKIM Filter v2.11.0 starting
Jun  6 17:31:13 walle postfix/pickup[18290]: BDFC747407: uid=1000 from=<arantxa>
Jun  6 17:31:13 walle postfix/cleanup[18329]: BDFC747407: message-id=<20230606173113.BDFC747407@walle.afm-tars.es>
Jun  6 17:31:13 walle opendkim[18320]: BDFC747407: DKIM-Signature field added (s=dkim-pass, d=afm-tars.es)
Jun  6 17:31:13 walle postfix/qmgr[18291]: BDFC747407: from=<arantxa@afm-tars.es>, size=468, nrcpt=1 (queue active)
Jun  6 17:31:13 walle postfix/smtp[18331]: connect to gmail-smtp-in.l.google.com[2a00:1450:4013:c1a::1a]:25: No route to host
Jun  6 17:31:14 walle postfix/smtp[18331]: BDFC747407: to=<ara.fer.mor@gmail.com>, relay=gmail-smtp-in.l.google.com[142.251.31.27]:25, delay=0.97, delays=0.03/0.01/0.41/0.51, dsn=2.0.0, status=sent (250 2.0.0 OK  1686072674 x8-20020a170906148800b0094edce87118si6756279ejc.1042 - gsmtp)
Jun  6 17:31:14 walle postfix/qmgr[18291]: BDFC747407: removed
```

Y compruebo el correo que me ha llegado a Gmail.

![gmail](/img/mailserver/24.png)

![original](/img/mailserver/25.png)

![original](/img/mailserver/26.png)

## Para luchar contra el SPAM

### **Tarea 5 (No obligatorio): Configura de manera adecuada Postfix para que tenga en cuenta el registro SPF de los correos que recibe. Muestra el log del correo para comprobar que se está haciendo el testeo del registro SPF.**

Para que Postfix tenga en cuenta el registro SPF de lo correos que recibe tenemos que instalar el siguiente paquete:

```bash
sudo apt install postfix-policyd-spf-python
```

Añadimos esta línea a /etc/postfix/master.cf.

```bash
sudo nano /etc/postfix/master.cf
```

```conf
#SPF
policyd-spf  unix  -       n       n       -       0       spawn     user=policyd-spf argv=/usr/bin/policyd-spf
```

Se ejecutará un proceso en un socket UNIX para realizar el análisis de SPF. Agregamos lo siguiente a Postfix para que los correos que se validen con SPF los acepte.

```bash
sudo nano /etc/postfix/main.cf
```

```conf
#SPF
policyd-spf_time_limit = 3600
smtpd_recipient_restrictions =
    check_policy_service unix:private/policyd-spf
```

Reinicio el servicio.

```bash
sudo systemctl restart postfix
```

Veo el log.

```bash
arantxa@walle:~$ sudo tail /var/log/mail.log
Jun  6 18:07:31 walle postfix/postfix-script[19275]: starting the Postfix mail system
Jun  6 18:07:31 walle postfix/master[19277]: daemon started -- version 3.5.18, configuration /etc/postfix
```

Me envío un correo desde Delta y vuelvo a comprobar el log.

![log](/img/mailserver/28.png)

Vemos que el análisis SPF se realiza correctamente pero el correo no me llega ya que al tener activado la redirección a Gmail me falla la comprobación. Si comento la línea del fichero **.forward** debería funcionar correctamente. Vuelvo a probar el envío desde Delta y compruebo que ahora sí he recibido el correo.

![log](/img/mailserver/29.png)

![mail](/img/mailserver/30.png)

### **Tarea 6 (No obligatoria): Configura un sistema antispam. Realiza comprobaciones para comprobarlo.**

Instalo y activo el paquete spamassassin.

```bash
sudo apt install spamassassin spamc
sudo systemctl enable --now spamassassin
```

Modifico /etc/default/spamassassin y pongo en 1 la línea CRON.

```bash
sudo nano /etc/default/spamassassin
```

```txt
CRON=1
```

Añado a /etc/postfix/master.cf lo siguiente:

```bash
sudo nano /etc/postfix/master.cf
```

```conf
#Spamassassin
smtp      inet  n       -       y       -       -       smtpd
  -o content_filter=spamassassin
submission inet n       -       y       -       -       smtpd
  -o content_filter=spamassassin
spamassassin unix -     n       n       -       -       pipe
  user=debian-spamd argv=/usr/bin/spamc -f -e /usr/sbin/sendmail -oi -f ${sender} ${recipient}
```

Ponemos una etiqueta de spam a los correos que se detecten como tal. Para ello descomentamos la siguiente línea en el fichero local.cf.

```bash
sudo nano /etc/spamassassin/local.cf
```

```conf
rewrite_header Subject *****SPAM*****
```

Reiniciamos el servicio:

```bash
sudo systemctl restart postfix spamassassin
```

Activo el log para ver la información mientras se envía.

```bash
sudo tail -f /var/log/mail.log
```

Uso la web [gtube](https://spamassassin.apache.org/gtube/) para enviarme un correo a mi VPS. Desde Delta hago lo siguiente:

```bash
wget http://spamassassin.apache.org/gtube/gtube.txt
mail arantxa@afm-tars.es < gtube.txt 
```

Compruebo el log y se puede observar que lo ha detectado como spam.

![log](/img/mailserver/31.png)

Miro el correo recibido para ver si tiene la etiqueta de Spam.

![mail](/img/mailserver/32.png)

![mail](/img/mailserver/33.png)

![mail](/img/mailserver/34.png)


### **Tarea 7 (No obligatoria): Configura un sistema antivirus. Realiza comprobaciones para comprobarlo.**

*(!!!!Como explico al final del ejercicio, no he podido finalizar esta tarea debido a un error por falta de memoria RAM. ClamAV requiere al menos 3GB de RAM. Aún así he dejado los pasos que he seguido y que deberían funcionar en una máquina con mayor RAM)*

Instalar los paquetes necesarios y activar el servicio **clamav**.

```bash
sudo apt install clamav clamav-daemon clamsmtp clamav-freshclam
sudo systemctl enable --now clamav-daemon
```

Editamos /etc/clamsmtpd.conf y cambiamos estas líneas:

```bash
OutAddress: 10025
Listen: 127.0.0.1:10026
```

Por estas:

```bash
OutAddress: 10026
Listen: 127.0.0.1:10025
```

Añadimos a la configuración de master.cf de Postfix lo siguiente:

```bash
sudo nano /etc/postfix/master.cf
```

```conf
#Clamav-antivirus
##AV scan filter (used by content_filter)
scan unix -       -       n       -       16       smtp
  -o smtp_send_xforward_command=yes
##For injecting mail back into postfix from the filter
127.0.0.1:10026 inet n       -       n       -       16       smtpd
  -o content_filter=
  -o receive_override_options=no_unknown_recipient_checks,no_header_body_checks
  -o smtpd_helo_restrictions=
  -o smtpd_client_restrictions=
  -o smtpd_sender_restrictions=
  -o smtpd_recipient_restrictions=permit_mynetworks,reject
  -o mynetworks_style=host
  -o smtpd_authorized_xforward_hosts=127.0.0.0/8
```

Indicamos en main.cf que se usará el servicio clamav.

```bash
sudo nano /etc/postfix/main.cf
```

```conf
#Clamav-antivirus
content_filter = scan:127.0.0.1:10025
receive_override_options = no_address_mappings
```

Para que utilice los parámetros que acabamos de configurar tenemos que reconfigurar el demonio clamav. Al reconfigurarlo nos hará algunas preguntas. Se puede ver el reultado de estas preguntas en /etc/clamav/clamd.conf.

```bash
sudo systemctl restart postfix clamav-daemon clamsmtp
sudo dpkg-reconfigure clamav-daemon
```

```bash
arantxa@walle:~$ sudo cat /etc/clamav/clamd.conf 
#Automatically Generated by clamav-daemon postinst
#To reconfigure clamd run #dpkg-reconfigure clamav-daemon
#Please read /usr/share/doc/clamav-daemon/README.Debian.gz for details
LocalSocket /var/run/clamav/clamd.ctl
FixStaleSocket true
LocalSocketGroup clamav
LocalSocketMode 777
# TemporaryDirectory is not set to its default /tmp here to make overriding
# the default with environment variables TMPDIR/TMP/TEMP possible
User clamav
ScanMail true
ScanArchive true
ArchiveBlockEncrypted false
MaxDirectoryRecursion 15
FollowDirectorySymlinks false
FollowFileSymlinks true
ReadTimeout 180
MaxThreads 12
MaxConnectionQueueLength 15
LogSyslog true
LogRotate true
LogFacility LOG_LOCAL6
LogClean false
LogVerbose false
PreludeEnable no
PreludeAnalyzerName ClamAV
DatabaseDirectory /var/lib/clamav
OfficialDatabaseOnly false
SelfCheck 3600
Foreground false
Debug false
ScanPE true
MaxEmbeddedPE 10M
ScanOLE2 true
ScanPDF true
ScanHTML true
MaxHTMLNormalize 10M
MaxHTMLNoTags 2M
MaxScriptNormalize 5M
MaxZipTypeRcg 1M
ScanSWF true
ExitOnOOM false
LeaveTemporaryFiles false
AlgorithmicDetection true
ScanELF true
IdleTimeout 30
CrossFilesystems true
PhishingSignatures true
PhishingScanURLs true
PhishingAlwaysBlockSSLMismatch false
PhishingAlwaysBlockCloak false
PartitionIntersection false
DetectPUA false
ScanPartialMessages false
HeuristicScanPrecedence false
StructuredDataDetection false
CommandReadTimeout 30
SendBufTimeout 200
MaxQueue 100
ExtendedDetectionInfo true
OLE2BlockMacros false
AllowAllMatchScan true
ForceToDisk false
DisableCertCheck false
DisableCache false
MaxScanTime 120000
MaxScanSize 100M
MaxFileSize 25M
MaxRecursion 16
MaxFiles 10000
MaxPartitions 50
MaxIconsPE 100
PCREMatchLimit 10000
PCRERecMatchLimit 5000
PCREMaxFileSize 25M
ScanXMLDOCS true
ScanHWP3 true
MaxRecHWP3 16
StreamMaxLength 25M
LogFile /var/log/clamav/clamav.log
LogTime true
LogFileUnlock false
LogFileMaxSize 0
Bytecode true
BytecodeSecurity TrustSigned
BytecodeTimeout 60000
OnAccessMaxFileSize 5M
```

Para comprobarlo activo el log en mi VPS y envío un correo desde Delta con el siguiente contenido:

```bash
sudo tail -f /var/log/mail.log
```

```bash
arantxa@delta:~$ mail arantxa@afm-tars.es
Subject: virus
X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*
Cc: 
```

**¡¡ERROR!!**

Al recibir el correo me aparece en el log un error que no he conseguido solucionar.

```txt
Jun  7 12:29:38 walle clamsmtpd: 100000: CLAMAV: couldn't connect to: /var/run/clamav/clamd.ctl: No such file or directory
```

Ese fichero no se crea por defecto, y si lo intento crear por mi misma o crear uno nuevo modificando la configuración de clamav me dice "Permission denied".

He intentado hacerlo con Amavis siguiendo [este post](https://www.linuxbabe.com/mail-server/postfix-amavis-spamassassin-clamav-debian), el cual me ha parecido muy interesante. Pero Amavis utiliza también el demonio clamav, y, por lo que pone en el post y en las recomendaciones que pone ClamAV en su web sobre requerimientos del sistema, me he dado cuenta de que mi máquina no tiene suficiente memoria RAM para usar ClamAV, por esto me da error.

![clamav](/img/mailserver/52.png)

![clamav](/img/mailserver/53.png)


He eliminado todo lo instalado para que no de error en los siguientes ejercicios, y he comentado la configuracíon que había añadido a Postfix, en los ficheros main.cf y master.cf.

## Gestión de correos desde un cliente

### **Tarea 8: Configura el buzón de los usuarios de tipo Maildir. Envía un correo a tu usuario y comprueba que el correo se ha guardado en el buzón Maildir del usuario del sistema correspondiente. Recuerda que ese tipo de buzón no se puede leer con la utilidad mail.**

Modificamos la configuración principal de Postfix para que utilice Maildir en lugar de mbox, que es el buzón por defecto de Postfix.

```bash
sudo nano /etc/postfix/main.cf
```

```conf
home_mailbox = Maildir/
```

Reiniciamos el servicio:

```bash
sudo systemctl restart postfix
```

Me envío un correo desde Delta.

```bash
arantxa@delta:~$ mail arantxa@afm-tars.es
Subject: prueba Maildir           
Esto es una prueba para usar el buzon Maildir
Cc: 
```

Comprobamos en el log que el correo ha llegado a maildir.

```bash
sudo tail -f /var/log/mail.log
```

```txt
Jun  7 17:14:21 walle postfix/smtpd[58789]: connect from 152.red-80-59-1.staticip.rima-tde.net[80.59.1.152]
Jun  7 17:14:41 walle policyd-spf[58795]: prepend Received-SPF: None (mailfrom) identity=mailfrom; client-ip=80.59.1.152; helo=babuino-smtp.gonzalonazareno.org; envelope-from=arantxa@arantxa.gonzalonazareno.org; receiver=<UNKNOWN> 
Jun  7 17:14:41 walle postfix/smtpd[58789]: 8C7FC474CA: client=152.red-80-59-1.staticip.rima-tde.net[80.59.1.152]
Jun  7 17:14:41 walle postfix/cleanup[58796]: 8C7FC474CA: message-id=<20230607171420.9A2D863400@delta.arantxa.gonzalonazareno.org>
Jun  7 17:14:41 walle postfix/qmgr[58782]: 8C7FC474CA: from=<arantxa@arantxa.gonzalonazareno.org>, size=1151, nrcpt=1 (queue active)
Jun  7 17:14:41 walle postfix/smtpd[58789]: disconnect from 152.red-80-59-1.staticip.rima-tde.net[80.59.1.152] ehlo=2 starttls=1 mail=1 rcpt=1 data=1 quit=1 commands=7
Jun  7 17:14:42 walle spamd[29351]: spamd: connection from 127.0.0.1 [127.0.0.1]:58398 to port 783, fd 6
Jun  7 17:14:42 walle spamd[29351]: spamd: setuid to debian-spamd succeeded
Jun  7 17:14:43 walle spamd[29351]: spamd: processing message <20230607171420.9A2D863400@delta.arantxa.gonzalonazareno.org> for debian-spamd:110
Jun  7 17:14:57 walle spamd[29351]: spamd: clean message (0.0/5.0) for debian-spamd:110 in 15.2 seconds, 1097 bytes.
Jun  7 17:14:57 walle spamd[29351]: spamd: result: . 0 - KHOP_HELO_FCRDNS,SPF_NONE,T_SCC_BODY_TEXT_LINE,T_SPF_HELO_TEMPERROR scantime=15.2,size=1097,user=debian-spamd,uid=110,required_score=5.0,rhost=127.0.0.1,raddr=127.0.0.1,rport=58398,mid=<20230607171420.9A2D863400@delta.arantxa.gonzalonazareno.org>,autolearn=ham autolearn_force=no
Jun  7 17:14:57 walle postfix/pickup[58781]: 91E04474D0: uid=110 from=<arantxa@arantxa.gonzalonazareno.org>
Jun  7 17:14:57 walle postfix/pipe[58797]: 8C7FC474CA: to=<arantxa@afm-tars.es>, relay=spamassassin, delay=36, delays=20/0.01/0/16, dsn=2.0.0, status=sent (delivered via spamassassin service)
Jun  7 17:14:57 walle postfix/qmgr[58782]: 8C7FC474CA: removed
Jun  7 17:14:57 walle postfix/cleanup[58796]: 91E04474D0: message-id=<20230607171420.9A2D863400@delta.arantxa.gonzalonazareno.org>
Jun  7 17:14:57 walle postfix/qmgr[58782]: 91E04474D0: from=<arantxa@arantxa.gonzalonazareno.org>, size=1531, nrcpt=1 (queue active)
Jun  7 17:14:57 walle postfix/local[58804]: 91E04474D0: to=<arantxa@afm-tars.es>, relay=local, delay=0.04, delays=0.02/0.02/0/0, dsn=2.0.0, status=sent (delivered to maildir)
Jun  7 17:14:57 walle postfix/qmgr[58782]: 91E04474D0: removed
```

En la penúltima línea vemos que, efectivamente, ha llegado a maildir.

Aunque hemos recibido el correo si entramos en mail para leerlo no nos aparece nada. Esto se debe a que mail está diseñado para trabajar con el formato del buzón mbox. Podemos instalar otro cliente de correo como **mutt**.

```bash
sudo apt install mutt
```

Una vez instalado tenemos que configurar mutt para que busque los nuevos correos en Maildir. Para ello creamos el siguiente fichero con este contenido.

```bash
nano ~/.muttrc
```

```bash
set mbox_type=Maildir
set folder="~/Maildir"
set mask="!^\\.[^.]"
set mbox="~/Maildir"
set record="+.Sent"
set postponed="+.Drafts"
set spoolfile="~/Maildir"
```

Y ahora abrimos el cliente de correo mutt.

```bash
arantxa@walle:~$ mutt
```

![mail](/img/mailserver/35.png)

![mail](/img/mailserver/36.png)


### **Tarea 9: Instala configura dovecot para ofrecer el protocolo IMAP. Configura dovecot de manera adecuada para ofrecer autentificación y cifrado.**

**Para realizar el cifrado de la comunicación crea un certificado en LetsEncrypt para tu dominio mail.tudominio.es. Recuerda que para el ofrecer el cifrado tiene varias soluciones:**

- **IMAP con STARTTLS: STARTTLS transforma una conexión insegura en una segura mediante el uso de SSL/TLS. Por lo tanto usando el mismo puerto 143/tcp tenemos cifrada la comunicación.**

- **IMAPS: Versión segura del protocolo IMAP que usa el puerto 993/tcp.**
  
- **Ofrecer las dos posibilidades.**

**Elige una de las opciones anterior para realizar el cifrado. Y muestra la configuración de un cliente de correo (evolution, thunderbird, …) y muestra como puedes leer los correos enviado a tu usuario.**

Instalo dovecot:

```bash
sudo apt install dovecot-imapd
```

Podemos ver los puertos que utiliza dovecot y que están escuchando.

```bash
arantxa@walle:~$ sudo netstat -putanl | egrep dovecot
tcp        0      0 0.0.0.0:143             0.0.0.0:*               LISTEN      62524/dovecot       
tcp        0      0 0.0.0.0:993             0.0.0.0:*               LISTEN      62524/dovecot       
tcp6       0      0 :::143                  :::*                    LISTEN      62524/dovecot       
tcp6       0      0 :::993                  :::*                    LISTEN      62524/dovecot 
```

En las políticas de firewall de mi VPS he abierto el tráfico en el puerto 993.

![mail](/img/mailserver/37.png)

Modifico el fichero /etc/dovecot/conf.d/10-mail.conf para que busque el correo en Maildir.

```bash
sudo nano /etc/dovecot/conf.d/10-mail.conf
```

```conf
mail_location = maildir:~/Maildir
```

Paro el servicio nginx para que no haya problemas al crear el certificado.

```bash
sudo systemctl stop nginx
sudo certbot certonly --standalone -d mail.afm-tars.es
```

Modifico el fichero de configuración 10-ssl.conf de dovecot para añadir las rutas de las claves.

```bash
sudo nano /etc/dovecot/conf.d/10-ssl.conf
```

```conf
ssl_cert = </etc/letsencrypt/live/mail.afm-tars.es/fullchain.pem
ssl_key = </etc/letsencrypt/live/mail.afm-tars.es/privkey.pem
```

Reinicio el servicio:

```bash
sudo systemctl restart dovecot
```

*(!!!!!La configuración en evolution y el mostrar que se pueden leer los correos enviados a mi usuario las realizaré en conjunto con la Tarea 11 (que la hago a continuación) para no repetir pasos)*


### **Tarea 11: Configura de manera adecuada postfix para que podamos mandar un correo desde un cliente remoto. La conexión entre cliente y servidor debe estar autentificada con SASL usando dovecor y además debe estar cifrada. Para cifrar esta comunicación puedes usar dos opciones:**

- **ESMTP + STARTTLS: Usando el puerto 567/tcp enviamos de forma segura el correo al servidor.**

- **SMTPS: Utiliza un puerto no estándar (465) para SMTPS (Simple Mail Transfer Protocol Secure). No es una extensión de smtp. Es muy parecido a HTTPS.**
  
**Elige una de las opciones anterior para realizar el cifrado. Y muestra la configuración de un cliente de correo (evolution, thunderbird, …) y muestra como puedes enviar los correos.**

Añado lo siguiente a la configuración master.cf de Postfix.

```bash
sudo nano /etc/postfix/master.cf
```

```conf
#SMTPS-envío correos
submission inet n       -       y       -       -       smtpd
  -o content_filter=spamassassin
  -o syslog_name=postfix/submission
  -o smtpd_tls_security_level=encrypt
  -o smtpd_sasl_auth_enable=yes
  -o smtpd_tls_auth_only=yes
  -o smtpd_reject_unlisted_recipient=no
  -o smtpd_client_restrictions=$mua_client_restrictions
  -o smtpd_helo_restrictions=$mua_helo_restrictions
  -o smtpd_sender_restrictions=$mua_sender_restrictions
  -o smtpd_recipient_restrictions=
  -o smtpd_relay_restrictions=permit_sasl_authenticated,reject
  -o milter_macro_daemon_name=ORIGINATING
smtps     inet  n       -       y       -       -       smtpd
  -o syslog_name=postfix/smtps
  -o smtpd_tls_wrappermode=yes
  -o smtpd_sasl_auth_enable=yes
  -o smtpd_reject_unlisted_recipient=no
  -o smtpd_client_restrictions=$mua_client_restrictions
  -o smtpd_helo_restrictions=$mua_helo_restrictions
  -o smtpd_sender_restrictions=$mua_sender_restrictions
  -o smtpd_recipient_restrictions=
  -o smtpd_relay_restrictions=permit_sasl_authenticated,reject
  -o milter_macro_daemon_name=ORIGINATING
```

Reiniciar el servicio:

```bash
sudo systemctl restart postfix
```

Ver los puertos a la escucha.

```bash
arantxa@walle:~$ sudo netstat -putanl | egrep master
tcp        0      0 0.0.0.0:465             0.0.0.0:*               LISTEN      63851/master        
tcp        0      0 0.0.0.0:25              0.0.0.0:*               LISTEN      63851/master        
tcp        0      0 0.0.0.0:587             0.0.0.0:*               LISTEN      63851/master        
tcp6       0      0 :::465                  :::*                    LISTEN      63851/master        
tcp6       0      0 :::25                   :::*                    LISTEN      63851/master        
tcp6       0      0 :::587                  :::*                    LISTEN      63851/master  
```

Permitimos el tráfico en las políticas de firewall del VPS para el puerto TCP 465.

![firewall](/img/mailserver/45.png)

Modificamos la configuración de Postfix.

```bash
sudo nano /etc/postfix/main.cf
```

He comentado las líneas:

```conf
smtpd_tls_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
smtpd_tls_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
```

Y he añadido lo siguiente:

```conf
#SMTPS-envio correos
smtpd_tls_cert_file=/etc/letsencrypt/live/mail.afm-tars.es/fullchain.pem
smtpd_tls_key_file=/etc/letsencrypt/live/mail.afm-tars.es/privkey.pem
##Authentication
smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth
smtpd_sasl_auth_enable = yes
smtpd_sasl_authenticated_header = yes
broken_sasl_auth_clients = yes
```

Reiniciamos Postfix.

```bash
sudo systemctl restart postfix
```

En dovecot descomento lo siguiente:

```bash
sudo nano /etc/dovecot/conf.d/10-master.conf
```

```conf
  # Postfix smtp-auth
  unix_listener /var/spool/postfix/private/auth {
    mode = 0666
  }
```

Reiniciamos dovecot.

```bash
sudo systemctl restart dovecot
```

Instalo Evolution en una máquina cliente y lo inicio.

```bash
usuario@cliente:~$ sudo apt install evolution
usuario@cliente:~$ evolution
```

**Configuración de Evolution:**

![evolution](/img/mailserver/38.png)

![evolution](/img/mailserver/39.png)

![evolution](/img/mailserver/40.png)

![evolution](/img/mailserver/41.png)

![evolution](/img/mailserver/42.png)

![evolution](/img/mailserver/43.png)

**Comprobación de que puedo leer los correos recibidos en el usuario de mi VPS a través de Evolution:**

![evolution](/img/mailserver/44.png)

![evolution](/img/mailserver/46.png)

**Comprobación de que puedo realizar envíos desde Evolution:**

![evolution](/img/mailserver/47.png)

![evolution](/img/mailserver/48.png)

![evolution](/img/mailserver/49.png)

![evolution](/img/mailserver/50.png)

![evolution](/img/mailserver/51.png)

### **Tarea 10 (No obligatoria): Instala un webmail (roundcube, horde, rainloop) para gestionar el correo del equipo mediante una interfaz web. Muestra la configuración necesaria y cómo eres capaz de leer los correos que recibe tu usuario.**

*(Las tareas 10 y 12 se realizan juntas)*

### **Tarea 12 (No obligatoria): Configura el cliente webmail para el envío de correo. Realiza una prueba de envío con el webmail.**

He elegido el webmail Roundcube.

Antes de instalar el contenedor de docker tenemos que crear el siguiente fichero con este contenido para mandarselo como parámetro a la configuración del contenedor docker. Si no hacemos esto podremos acceder a Roundcube pero no podremos enviar mensajes.

```bash
sudo mkdir /root/config-roundcube
sudo nano /root/config-roundcube/custom.inc.php
```

```php
<?php
$config['mail_domain'] = array(
    'mail.afm-tars.es' => 'afm-tars.es'
);
?>
```

Ahora instalo un contenedor Docker de Roundcube enviandole la configuración anterior.

```bash
sudo docker run -v /root/config-roundcube/:/var/roundcube/config/ -e ROUNDCUBEMAIL_DEFAULT_HOST=ssl://mail.afm-tars.es -e ROUNDCUBEMAIL_SMTP_SERVER=ssl://mail.afm-tars.es -e ROUNDCUBEMAIL_SMTP_PORT=465 -e ROUNDCUBEMAIL_DEFAULT_PORT=993 -p 8001:80 -d roundcube/roundcubemail```
```

Crear un registro CNAME en el DNS llamado roundcube.afm-tars.es que apunte a walle.afm-tars.es

![cname](/img/mailserver/54.png)

Creo el virtualhost en nginx.

```bash
sudo nano /etc/nginx/sites-available/roundcube
```

```conf
server {
        listen 80;
        listen [::]:80;

        server_name roundcube.afm-tars.es;

        return 301 https://$host$request_uri;
}

server {
        listen 443 ssl http2;
        listen [::]:443 ssl http2;

        index index.html index.php index.htm index.nginx-debian.html;

        server_name roundcube.afm-tars.es;

        location / {
                proxy_pass http://localhost:8001;
                include proxy_params;
        }
}
```

Creo el enlace simbólico:

```bash
sudo ln -s /etc/nginx/sites-available/roundcube /etc/nginx/sites-enabled/roundcube
sudo systemctl restart nginx
```

Creo el certificado Let'sEncrypt:

```bash
sudo certbot certonly --standalone -d roundcube.afm-tars.es
```

Añado al virtualhost lo siguiente:

```bash
sudo nano /etc/nginx/sites-available/roundcube
```

```conf
        ssl    on;
        ssl_certificate /etc/letsencrypt/live/roundcube.afm-tars.es/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/roundcube.afm-tars.es/privkey.pem;
```

Y reinicio el servicio.

```bash
sudo systemctl restart nginx
```

Ingresamos a roundcube.afm-tars.es.

![roundcube](/img/mailserver/55.png)

Ingreso con mi usuario y contraseña de mi usuario local **arantxa**.

![roundcube](/img/mailserver/56.png)

Como vemos en la captura, podemos ver sin problema los correos de nuestro usuario.

Probamos a enviar un correo a Gmail y que se recibe.

![roundcube](/img/mailserver/57.png)

![roundcube](/img/mailserver/58.png)

![roundcube](/img/mailserver/59.png)

![roundcube](/img/mailserver/60.png)

![roundcube](/img/mailserver/61.png)

Por último respondemos al correo recibido en Gmail y comprobamos que lo recibimos y podemos leer en Roundcube.

![roundcube](/img/mailserver/62.png)

![roundcube](/img/mailserver/63.png)

![roundcube](/img/mailserver/64.png)

## Comprobación final

### **Tarea 13: Prueba de envío de correo. En [esta página](https://www.mail-tester.com/) tenemos una herramienta completa y fácil de usar a la que podemos enviar un correo para que verifique y puntúe el correo que enviamos. Captura la pantalla y muestra la puntuación que has sacado.**

La primera vez que he enviado el correo a test-radvng5qk@srv1.mail-tester.com me ha dado una puntuación de 6'8. Por lo que he mejorado algunas cosas para obtener una puntuación mayor.

Añado un registro TXT para el dominio _dmarc.afm-tars.es con el valor **v=DMARC1; p=none**.

![dmarc](/img/mailserver/66.png)

Vuelvo a entrar en la web. Esta vez me da un correo diferente.

![testmail](/img/mailserver/67.png)

Envío un correo con contenido.

![testmail](/img/mailserver/68.png)

Vuelvo a la página y compruebo la puntuación.

![testmail](/img/mailserver/69.png)

Mi puntuación ahora es de 8'9. 

Algunas cosas que me dice que podría mejorar son:

![testmail](/img/mailserver/70.png)

![testmail](/img/mailserver/71.png)

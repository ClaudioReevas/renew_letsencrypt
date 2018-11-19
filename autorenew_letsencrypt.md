#Apache certificado LetsEncrypt autorenovación
Autor: Claudio Rivas

Fecha: 18 de noviembre de 2018

## Aviso de exclusión de garantías y responsabilidad


*La información provista en este documento no constituye una responsabilidad para el autor ni para los entes mencionados, por lo cual deberá ser utilizado a discreción del lector bajo su propio riesgo.

La utilización de plataformas en la nube generalmente implica un costo, por lo cual es necesario revisar dichos costos con los proveedores de servicios, los cuales por ninguna razón son transferibles al autor.

DigitalOcean no patrocina al autor de este documento.*


## Sugerencia
Sigue las actividades de la tabla de contenido, están puestas en orden para realizar toda la configuración de manera ordenada.

## Contenido
[Supuestos](supuestos)
[Instalar Certbot](instalar-certbot)
[Ejecutar Certbot](ejecutar-certbot)
[Validar configuración del archivo httpd-le-ssl.conf](validar-configuracion-del-archivo-httpd-le-ssl.conf)
[Configurar Certbot para autorrenovación](configurar-certbot-para-autorenovacion)


## Supuestos
Para obtener los beneficios de esta guía se supone que el usuario cuenta con los siguientes requerimientos instalados:

- CentOS 7
- Apache


## Instalar Certbot
Para instalar certbot ejecuta los siguientes comandos en el terminal:

```
sudo yum install epel-release
sudo yum install  mod_ssl python-certbot-apache
sudo yum install certbot
```

## Ejecutar Certbot
En la terminal ejecutar:
```
certbot
```

Selecciona el dominio que deseas configurar, este valor es tomado si ya cuentas con un sitio configurado:

```
Which names would you like to activate HTTPS for?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: www.yourdomain.com
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter 'c' to cancel): 1
```

Selecciona reinstalar o renovar certificado:

```
What would you like to do?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: Attempt to reinstall this existing certificate
2: Renew & replace the cert (limit ~5 per 7 days)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate number [1-2] then [enter] (press 'c' to cancel): 1
```
Selecciona la opción de redirección conforme tus preferencias, se sugiere seleccionar **2: Redirect**.

```
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate number [1-2] then [enter] (press 'c' to cancel): 1
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
-
```

Si realizaste los pasos correctamente deberás ver un resultado como este:

```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Your existing certificate has been successfully renewed, and the new certificate
has been installed.
The new certificate covers the following domains: https://www.yourdomain.com
You should test your configuration at:
https://www.ssllabs.com/ssltest/analyze.html?d=www.yourdomain.com
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/www.yourdomain.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/www.yourdomain.com/privkey.pem
   Your cert will expire on 2019-02-12. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot again
   with the "certonly" option. To non-interactively renew *all* of
   your certificates, run "certbot renew"
 - If you like Certbot, please consider supporting our work by:
   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```
   
   
## Validar configuración del archivo httpd-le-ssl.conf

El archivo **httpd-le-ssl.conf** se encuentra en la ruta **/etc/httpd/conf**, se aconseja tener las siguientes prácticas:

1. Asegurate de su nombre de servidor en toda la configuración.

2. Corrobora que no tienes un redireccionamiento en su configuración de **https**  (VirtualHost *: 443) en el ejemplo en el que se documenta la cuarta línea; comúnmente certbot agrega esta línea, así que asegúrese de comentarla con **\#**.

3. Para reedirigir el tráfico no seguro a una página segura, verifica de tener un redireccionamiento en su configuración de **http** (VirtualHost *: 80).

4. Verifica de que su configuración **DocumentRoot** esté apuntando a la carpeta correcta donde vive su sitio web o aplicación.
5. Siempre que modifique el archivo **httpd-le-ssl.conf** deberá reiniciar el servicio de apache, para el caso de CentOS 7 se realiza con el comando ```systemctl restart httpd```

```
<IfModule mod_ssl.c>
<VirtualHost *:443>
        ServerName www.yourdomain.com
#       Redirect / https://www.yourdomain.com/
Include /etc/letsencrypt/options-ssl-apache.conf
SSLCertificateFile /etc/letsencrypt/live/www.yourdomain.com/cert.pem
SSLCertificateKeyFile /etc/letsencrypt/live/www.yourdomain.com/privkey.pem
SSLCertificateChainFile /etc/letsencrypt/live/www.yourdomain.com/chain.pem
DocumentRoot /var/www/html
</VirtualHost>
</IfModule>
<IfModule mod_ssl.c>
<VirtualHost *:80>
        ServerName www.yourdomain.com
        Redirect / https://www.yourdomain.com/
        SSLCertificateFile /etc/letsencrypt/live/www.yourdomain.com/cert.pem
        SSLCertificateKeyFile /etc/letsencrypt/live/www.yourdomain.com/privkey.pem
        Include /etc/letsencrypt/options-ssl-apache.conf
        SSLCertificateChainFile /etc/letsencrypt/live/www.yourdomain.com/chain.pem
# RewriteEngine on
# Some rewrite rules in this file were disabled on your HTTPS site,
# because they have the potential to create redirection loops.
# RewriteCond %{SERVER_NAME} =www.yourdomain.com
# RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]
</VirtualHost>
</IfModule>
```


## Configurar Certbot para autorrenovación
La configuración de la autorrenovación es realizada a través del propio certbot y mediante la configuración de una tarea programada o cron job. Para ello:
Abre el archivo **crontab**

```
sudo cat /etc/crontab
```

y agrega la línea:

```
10 16 1 * * certbot renew && systemctl restart httpd
```

Para verificar la ejecución puedes verificar el log mediante el comando:

```
systemctl status crond
```
ó

```
tail -f /var/log/cron
```
Al identificar una línea como esta, puedes corroborar la ejecución de dicho comando:

```
Nov 1 16:14:01 root CROND[2294]: (root) CMD (certbot renew && systemctl restart httpd)
```
También en tu sitio puedes verificar la nueva vigencia del certificado.


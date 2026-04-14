# EC2 - Elastic Compute Cloud

#### Para un microservicio pequeño de Spring Boot WebFlux
Una instancia t3.micro o t2.micro suele bastar para desarrollo.

#### Qué se usa en el trabajo: 
Amazon Linux 2023 o Ubuntu Server. Se usan "Security Groups" como firewall para cerrar todos los puertos excepto el 22 (SSH) y el 8080 (Spring Boot).

#### Error común a evitar: No guardar el archivo .pem (la llave privada). 
Si lo pierdes, pierdes el acceso al servidor para siempre. No hay "recuperar contraseña".

#### Par de claves (inicio de sesión) 
Click en "Create new key pair". Nombre: mi-llave-aws. Formato: .pem. Descárgalo y guárdalo en una carpeta segura. Para conectarnos por OpenSSH, el formato .pem es el adecuado. Para PuTTY, se necesita convertirlo a .ppk.

#### Network Settings (Security Group):

Permitir SSH (puerto 22) desde "Anywhere" (o mejor "My IP").

Click en "Add security group rule": Custom TCP, puerto 8080, desde "Anywhere" (0.0.0.0/0). Aquí es donde escuchará tu API.

#### Protege la llave (Requisito de Linux)
###### chmod 400 llave-aws.pem
Si no haces esto, SSH rechazará la conexión por ser "demasiado abierta".

```
nick@fedora:~/Documentos$ ssh -i dev-nick-key.pem ec2-user@3.19.32.42
The authenticity of host '3.19.32.42 (3.19.32.42)' can't be established.
ED25519 key fingerprint is SHA256:rzIxdsUzJn3NsiSNSCrIgvt6ul6yWXKCfu0F2UColJY.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '3.19.32.42' (ED25519) to the list of known hosts.
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0644 for 'dev-nick-key.pem' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "dev-nick-key.pem": bad permissions
ec2-user@3.19.32.42: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
```
##### Conexion ejemplo
`ssh -i dev-nick-key.pem ec2-user@3.19.32.42`

Amazon Linux 2023 está basado en Fedora (específicamente en partes de Fedora 34/35/36).
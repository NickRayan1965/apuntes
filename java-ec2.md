##### Configuracion para un proyecto java en una instancia EC2 de AWS de tipo Amazon Linux 2023 basada en Fedora

```bash
# Actualiza el gestor de paquetes (igual que en Fedora)
sudo dnf update -y

# Instala Java (17 o 21 según tu proyecto)
sudo dnf install java-17-amazon-corretto-devel -y

# Verifica Java
java -version
```

##### Cómo subir tu API (El flujo real)
 * ###### Opción A: El "Camino rápido" (Desde tu terminal Fedora local)
  ```bash scp -i "llave-aws.pem" target/tu-proyecto-0.0.1.jar ec2-user@TU_IP_PUBLICA:/home/ec2-user/```
  * ###### Opción B: El "Camino Pro" (Git)

##### Configurar Git y SSH en la EC2
```bash
sudo dnf install git -y
```
##### Generar la llave SSH en la EC2
```bash
ssh-keygen -t ed25519 -C "tu-email@ejemplo.com"
```
##### Obtener la llave pública para GitHub
Ejecuta esto y copia todo el texto que empieza con ssh-ed25519...:
```bash
cat ~/.ssh/id_ed25519.pub
```

#### Agregarla a GitHub
* Ve a tu GitHub -> Settings (clic en tu foto).

* Menú lateral: SSH and GPG keys.

* Botón: New SSH key.

* Título: AWS-EC2-Backend.

* Pega el texto que copiaste y guarda

##### Clonar tu repositorio en la EC2
```bash
git clone git@github.com:NickRayan1965/localizaciones.git
```

#### OJO ./mvnw clean package
Como usas Spring Boot, podrías intentar compilar con ./mvnw clean package, pero cuidado: las instancias t3.micro tienen solo 1GB de RAM. Maven/Gradle al compilar suelen consumir más que eso y la instancia se va a congelar (se "muere").

Qué se usa en el trabajo:

CI/CD: GitHub Actions compila el JAR y lo envía a AWS (lo veremos después).

Swap File: Crear un "archivo de intercambio" en el disco para que actúe como RAM extra.


##### Entra donde está el archivo .jar
cd target 

##### Ejecuta el JAR
java -jar tu-archivo-0.0.1-SNAPSHOT.jar


### Persistencia: Systemd Services
Que tu API de Spring Boot se inicie sola al encender el servidor y se reinicie si ocurre un error (Crash).

* ###### Crea el archivo de configuración (necesitas permisos de sudo):
    ```bash
    sudo nano /etc/systemd/system/backend.service
    
    [Unit]
    Description=Spring Boot WebFlux API
    After=syslog.target network.target

    [Service]
    User=ec2-user
    # Ajusta la ruta a donde está tu JAR realmente
    ExecStart=/usr/bin/java -jar /home/ec2-user/tu-repo/target/tu-app-0.0.1.jar
    SuccessExitStatus=143
    Restart=always
    RestartSec=10

    [Install]
    WantedBy=multi-user.target
* ###### Guardar, recargar y activar:
    ```bash
    sudo systemctl daemon-reload
    sudo systemctl enable backend.service
    sudo systemctl start backend.service
* ###### Verificar que esté corriendo:
    ```bash
    sudo systemctl status backend.service

#### Profundización
  * ##### Análisis del backend.service
    1 Description y After
      * 'Description': Simplemente un nombre legible. Es lo que verás cuando ejecutes systemctl status
      * 'After': Define el orden de arranque. Aquí le decimos: "No intentes arrancar la API si la red (network.target) aún no está lista". Si no pones esto, la app podría intentar subir antes de que el servidor tenga IP y fallará.
        ```bash
        [Unit]
        Description=Spring Boot WebFlux API
        After=syslog.target network.target
    2 [Service]
      * 'User' => CRÍTICO por seguridad. Ante una vulverabilidad limita los privilegios
      * 'ExecStart' => La ruta absoluta al ejecutable y al archivo. En Systemd siempre usa rutas completas (ej. /usr/bin/java en lugar de solo java).
      * 'SuccessExitStatus=143' => Cuando detienes Spring Boot (SIGTERM), el código de salida suele ser 143. Sin esta línea, Systemd podría pensar que la app "falló" al cerrarse y marcar un error innecesario.
      * 'Restart=always' => Si la app se cae por cualquier motivo, Systemd intentará reiniciarla automáticamente.
      * 'RestartSec=10' => El tiempo de espera antes de reintentar el arranque. Útil para no saturar el CPU si la app está en un bucle de fallos.
        ```bash
        [Service]
        User=ec2-user
        ExecStart=/usr/bin/java -jar /home/ec2-user/tu-repo/target/tu-app-0.0.1.jar
        SuccessExitStatus=143
        Restart=always
        RestartSec=10
    3 [Install]
      * 'WantedBy=multi-user.target' => Define en qué nivel de ejecución debe activarse. multi-user.target es el estándar para servidores; básicamente significa: "Arranca este servicio cuando el sistema esté listo para recibir usuarios"
        ```bash
        [Install]
        WantedBy=multi-user.target
      
    4 VARIABLES DE ENTORNO
      * 4.1
        ```bash
          Environment=SPRING_PROFILES_ACTIVE=prod
          Environment=DB_PASSWORD=mi-clave-segura
          EnvironmentFile=/home/ec2-user/backend.env
    5 Limites de memoria
      * -
        ```bash
        ExecStart=/usr/bin/java -Xms256m -Xmx512m -jar /ruta/al/jar
    6 Working Directory
      * Si tu app necesita leer archivos locales (como logs o configs externas) que están en una carpeta específica:
        ```bash
          WorkingDirectory=/home/ec2-user/tu-repo/
  * ##### Monitorear Servicio
    * Ver logs en tiempo real:
      ```bash
      sudo journalctl -u backend.service -f
    * Ver solo errores desde el último reinicio:
      ```bash
      journalctl -u backend.service -b -p err
    * Ver el último reinicio completo:
      ```bash
      journalctl -u backend.service -b
##### Extras
  * Usa After para dependencias técnicas (Red, Base de Datos local, Sistema de archivos).

  * Usa WantedBy para decirle a Linux: "Oye, cada vez que el servidor se prenda de forma normal (modo multi-usuario), mi API debe estar prendida". Usa Targets que son agrupaciones de servicios. multi-user.target es el más común para servidores sin interfaz gráfica. Incluye Sistema con red, múltiples usuarios, pero sin interfaz gráfica.
  graphical.target incluye lo mismo que multi-user.target pero también arranca el entorno gráfico (X11, GDM, etc). 
###### AWS CLI

##### Intalacion fedora
```bash
sudo dnf install awscli
```
##### Instalacion Windows
1. Metodo 1: Descargar el instalador desde la página oficial de AWS
2. Metodo 2: desde la terminal de Windows (PowerShell) usando el siguiente comando con winget:
```bash
winget install Amazon.AwsCLI
```


#### Generar Credenciales
  1. IAM
  2. Seleccionar el usuario
  3. Pestaña "Security credentials"
  4. Access keys -> Create access key
  5. Seleccionamos el tipo Command line interface (CLI) 
  6. Copiamos el Access key ID, el Secret access key y el .csv 
#### Configurar AWS CLI
```bash
aws configure
```
Luego se nos pedirá ingresar el Access key ID, el Secret access key, la región por defecto (ejemplo: us-east-1) y el formato de salida (ejemplo: json).

#### Comandos básicos
 - Validar conexion
```bash
aws sts get-caller-identity
```
 - Listar instancias EC2
```bash
aws ec2 describe-instances
```
 - Listar instancias EC2 con formato de tabla
```bash
aws ec2 describe-instances --query "Reservations[*].Instances[*].{ID:InstanceId,State:State.Name,PUBLIC_IP:PublicIpAddress}" --output table
```
 - Listar buckets S3
```bash
aws s3 ls
```


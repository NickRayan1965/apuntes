# SonarQube
Plataforma analis de codigo, puede detectar bugs, vulnerabilidades, malas practicas, codigo sucio y cobertura de pruebas

## Composición
  ### 1. Servidor SonarQube
  ### 2. Scanner SonarQube

## Instalacion
  ### 1. Extender el limite de memorias mapeadas para Elasticsearch en Fedora (tambien se puede en windows con wsl 2)
  ```bash
  sudo sysctl -w vm.max_map_count=262144
  ```
  Pero para que sea permanente
  ```bash
  echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.d/99-sonarqube.conf
  ```

## Conceptos clave

### 1. Quality Gate (La compuerta de calidad)

### 2. Deuda Técnica

### 3. Clean as You You Go (Enfoque en Código Nuevo)
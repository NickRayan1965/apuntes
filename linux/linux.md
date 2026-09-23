# Linux
### INODO
  En linux los datos reales no se guardan con los nombres que vemos sino en una estructura llamada inodo a la que despues podemos apuntar con un nombre de archivo.
### Enlace duro y enlace simbolico
  - Enlace duro: es otra forma de llamar a un archivo, apunta a un inodo, normalmente si solo hay un enlace duro a un archivo, cuando borramos el archivo, el inodo se borra y con ello los datos. Si hay varios enlaces duros apuntando al mismo inodo, cuando borramos uno de ellos, los otros siguen apuntando al mismo inodo y por lo tanto los datos no se pierden.
    ```bash
      touch archivo.txt # crea un archivo vacio
      ln archivo.txt enlace_duro.txt # crea un enlace duro a archivo.txt
      # si ahora hacemo ls -l archivo.txt 

  - Enlace simbolico: es un archivo especial que apunta a otro archivo, si borramos el archivo al que apunta el enlace simbolico, el enlace simbolico queda roto y no podemos acceder a los datos.
### Comando "ls"
  ```bash 
  ls
  # Parametros
  - `-l`: Muestra la información detallada de los archivos y directorios.
  ```
  Respuesta 
  ```bash
  ls -l /var/run/docker.sock
  srw-rw---- 1 root docker 0 set  6 16:19 /var/run/docker.sock
  ```
  - "srw-rw----":
    - `s`: es un socket UNIX, sirve para comunicacion entre procesos (d => directorio, l => enlace simbolico, - => archivo normal)
    - `rw-`: permisos del propietario (root) de lectura y escritura (r=>read, w=>write, x=>execute, - => no permiso)
    - 2do `rw-`: permisos del grupo (docker) de lectura y escritura(r=>read, w=>write, x=>execute, - => no permiso)
    - 3er `---`: permisos de otros usuarios (ninguno)
  
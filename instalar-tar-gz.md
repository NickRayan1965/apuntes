##### Como instalar un tar.gz en Linux (Fedora)
```bash
# 1. Descomprimir
tar -xvf anydesk*.tar.gz

# 2. Entrar a la carpeta
cd anydesk*

# 3. Dar permisos (si aplica)
chmod +x anydesk

# 4. Ejecutar
./anydesk
```

##### Crear un enlace simbólico al Escritorio (opcional)
```bash
nano ~/.local/share/applications/anydesk.desktop
#contenido
[Desktop Entry]
Name=AnyDesk Portable
Exec=/home/nick/Descargas/anydesk-8.0.2/anydesk
Icon=/home/nick/Descargas/anydesk-8.0.2/icons/hicolor/32x32/apps/anydesk.pnp
Type=Application
StartupWMClass=jetbrains-studio # Es para que el icono del menu al ser abierto coincida con el del escritorio, obtenemos este nombre ejecutando `xprop WM_CLASS` y haciendo click en la ventana del app
Terminal=false
```
Permisos y refrescar
```bash
chmod +x ~/.local/share/applications/anydesk.desktop
update-desktop-database ~/.local/share/applications
```



##### Notas
1. tar -xvf anydesk*.tar.gz
tar = herramienta para manejar archivos .tar (archivador).
Flags:
-x → extract (extraer)
-v → verbose (muestra archivos mientras extrae)
-f → file (indica que el siguiente argumento es el archivo)
anydesk*.tar.gz:
* = glob (expansión del shell), coincide con cualquier texto.
.tar.gz:
.tar = contenedor
.gz = compresión con gzip
👉 Internamente: tar descomprime (gzip) y luego extrae el tar.

2. chmod +x anydesk
chmod = change mode (cambia permisos de archivo).
* +x:
  + → agrega permiso
  x → permiso de ejecución (execute)
👉 Afecta bits de permisos en inode:

Permisos básicos:
  * r (4) → read
  * w (2) → write
  * x (1) → execute
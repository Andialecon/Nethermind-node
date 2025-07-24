
******************
COMANDOS DE DOCKER
******************

docker-compose up -d
docker exec -it geth-mempool geth attach
eth.syncing
eth.blockNumber
docker restart geth-mempool
docker restart frpc
docker-compose down && docker-compose up -d

#ver la cantidad de disco que ocupa docker
docker system df
docker stats

#Eliminá imágenes y contenedores que no usás:
docker system prune -a --volumes
docker system prune --all --volumes -f

docker logs frpc
docker exec -it frpc sh
docker network ls
docker network inspect ethereumnode_default
docker ps -a
docker-compose restart
docker exec -it lighthouse bash
docker logs nethermind --tail 50
docker logs lighthouse --tail 50


****************
Comandos de WLS
****************
#listar las distribuciones de Linus instaladas y su estado
wsl -l -v

# Lista todas las distribuciones WSL
wsl --list --all

# Desinstala Ubuntu (ambas versiones)
wsl --unregister Ubuntu
wsl --unregister Ubuntu-20.04

#obtener la ip de WLS
wsl hostname -I

#Listar todos los discos virtuales de wsl
Get-ChildItem "$env:USERPROFILE\AppData\Local\Packages" -Recurse -Filter ext4.vhdx -ErrorAction SilentlyContinue

#Apagar WLS
wsl --shutdown
wsl  # Reinicia WSL

******************
Borrar docker por completo
******************
wsl --shutdown
Remove-Item -Recurse -Force "$env:LOCALAPPDATA\Docker"
Remove-Item -Recurse -Force "$env:USERPROFILE\.docker"
Remove-Item -Recurse -Force "$env:APPDATA\Docker"




#Monitorear logs específicos
docker logs nethermind --tail 100 -f | grep -E 'Old Headers|Snap|Peers'





CREAR TUNEL INVERSO EN UNA MAQUINA DE EC2 

1 Conectate a tu EC2 por SSH

    ssh -i "tunel.pem" ubuntu@18.188.178.0

2. Instalar Docker  
    sudo apt update && sudo apt upgrade -y
    sudo apt install -y docker.io
    sudo systemctl start docker
    sudo systemctl enable docker
    sudo usermod -aG docker ubuntu

    IMPORTANTE: después de usar usermod -aG docker, cerrá y volvé a abrir tu sesión SSH para que surta efecto.

3. Verificar que Docker funciona
    docker version
    docker run hello-world

4. Instalar Docker Compose (opcional pero útil)
    sudo curl -L "https://github.com/docker/compose/releases/download/v2.24.6/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose
    docker-compose version

CONFIGURAR EL CONTENEDOR DE FRPS
1. Crea una carpeta para los archivos
    mkdir -p ~/frp-server
    cd ~/frp-server
2. Crea el archivo frps.ini
    nano frps.ini
3. pegar esto:
    [common]
    bind_port = 7000
    dashboard_port = 7500
    dashboard_user = admin
    dashboard_pwd = admin
4. Presioná Ctrl + O, luego Enter para guardar y Ctrl + X para salir.
5. Crea el docker-compose.yml
    nano docker-compose.yml
6. pega esto: 
version: "3.8"

services:
  frps:
    image: snowdreamtech/frps
    container_name: frps
    restart: always
    ports:
      - "7000:7000"
      - "7500:7500"  # dashboard
      - "8545:8545"  # RPC
      - "8546:8546"  # WebSocket
    volumes:
      - ./frps.ini:/etc/frp/frps.ini
    command: ["-c", "/etc/frp/frps.ini"]
7. Presioná Ctrl + O, luego Enter para guardar y Ctrl + X para salir.
8. Levanta el contenedor
    docker-compose up -d
9.Verifica que está corriendo:
    docker ps



VALIDAR PUERTOS ABIERTOS EN ubuntu
sudo ss -tulnp | grep -E '7000|8545|18545'


opcion 2. CONFIGURAR WireGuard en el servidor para hacer un tunel con VPN 
(WireGuard es una solución VPN moderna, rápida y segura que soluciona tu problema de CGNAT creando un túnel cifrado entre tu nodo local y tu instancia EC2)

#instalar en ubuntu
sudo apt install wireguard
wg genkey | sudo tee /etc/wireguard/privatekey | wg pubkey | sudo tee /etc/wireguard/publickey

# Para ver tu clave privada (EC2):
sudo cat /etc/wireguard/privatekey
cat ~/client_private.key

# Para ver tu clave pública (EC2):
sudo cat /etc/wireguard/publickey
cat ~/client_public.key


# editar archivo de configuracion: 
sudo nano /etc/wireguard/wg0.conf

#verificar el estado de la conexion 
sudo systemctl status wg-quick@wg0.service
sudo wg show

VERIFICAR SI GETH ESTÁ ESCUCHANDO POR LOS PUERTOS DE MI MAQUINA
netstat -ano | findstr 8545

PROBAR CONECTIVIDAD AL PUERTO DE UNA IP 
Test-NetConnection 18.188.178.0 -Port 18545

#Para reiniciar el tunel
sudo wg-quick down wg0
sudo wg-quick up wg0

**********
nethermind
**********
# Detener de forma correcta el nodo de Nethermaind
docker stop nethermind

docker-compose restart nethermind


#CONSULTAR SI EL NODO ESTÁ CORRIENDO consultando el numero del bloque
curl -X POST http://localhost:8545 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"eth_blockNumber","params":[]}'


#Cosultar el estado de la sincronizacion
curl -X POST http://localhost:8545 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"eth_syncing","params":[]}'

#Consultar peer conectados
Mauro@DESKTOP-0JKN22F MINGW64 ~/Documents/Ethereum node
curl -X POST http://localhost:8545 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"net_peerCount","params":[]}'
{"jsonrpc":"2.0","result":"0x29","id":1}


#ver si ya se puede consultar la menpool desde powershell
curl -Uri "http://localhost:8545" -Method POST -Body '{"jsonrpc":"2.0","method":"eth_pendingTransactions","params":[],"id":1}' -ContentType "application/json"




#abrir los puertos en windows (ejecutar en ´PowerShell como administrador)
New-NetFirewallRule -DisplayName "Allow Nethermind Ports" -Direction Inbound -LocalPort 8545,8546,8551,5052 -Protocol TCP -Action Allow



*******************************
configurar nethermind en ubuntu
*******************************
1.Instalar docker.
2.Crear un directorio para el proyecto
mkdir -p ~/nethermind-docker

3.Crear el archivo de docker-compose dentro de la nueva carpeta
nano docker-compose.yml (pegar la configuracion en el editor de nano)

4. Crear otros archivos como claves ect (ejemplo con clave para ethermind)
openssl rand -hex 32 > engine.jwt

5. Verifica que todos los archivos estén creados en el directorio
ls -l ~/nethermind-docker

6. iniciar docker
cd ~/nethermind-docker
docker-compose up -d



*********************************************
Aplicar prune a docker para recuperar espacio
*********************************************

wsl --shutdown
Stop-Process -Name "Docker Desktop" -Force

#Te dirá cuanto espacio se liberó de docker, sino se ve reflejado en el disco, comprime los discos que usa WSL
docker system prune -a --volumes


#Buscar en que ruta están los discos vhdx de WSL
Get-ChildItem -Path "C:\" -Recurse -Filter *.vhdx -ErrorAction SilentlyContinue | Select-Object FullName

# 1. Docker Data (donde está tu nodo)
Optimize-VHD -Path "C:\Users\Mauro\AppData\Local\Docker\wsl\disk\docker_data.vhdx" -Mode Full

# 2. Docker Main
Optimize-VHD -Path "C:\Users\Mauro\AppData\Local\Docker\wsl\main\ext4.vhdx" -Mode Full

# 3. Ubuntu (opcional)
Optimize-VHD -Path "C:\Users\Mauro\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu20.04LTS_79rhkp1fndgsc\LocalState\ext4.vhdx" -Mode Full
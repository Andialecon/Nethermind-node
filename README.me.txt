
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
docker logs frpc
docker exec -it frpc sh
docker network ls
docker network inspect ethereumnode_default
docker ps -a
docker exec -it lighthouse bash
docker logs nethermind --tail 50
docker logs lighthouse --tail 50





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

#CONSULTAR SI EL NODO ESTÁ CORRIENDO consultando el numero del bloque
curl -X POST http://localhost:8545 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"eth_blockNumber","params":[]}'


#Cosultar el estado de la sincronizacion
curl -X POST http://localhost:8545 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"eth_syncing","params":[]}'


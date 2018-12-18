
# Conectarse a darkmesh (usando contenedores docker)

## Llevar la configuración inicial a tu servidor
```bash
mkdir ~/workspace/
cd ~/workspace/
git clone https://git.darkme.sh/darkmesh/darkmesh.git
```
## Copiar la configuración inicial a un directorio del servidor
```bash
cp -a darkmesh /opt/darkmesh
```

## Construir la imagen docker para tinc (no necesario si usas AMD64)
```bash
cd ~/workspace/
git clone https://git.darkme.sh/darkmesh/docker-tinc.git
cd docker-tinc
docker build -t tinc:0.0.1 .
```

## Generar el fichero de configuración de tu nodo
```bash
cd /opt/darkmesh/
docker run --rm -ti --name tinc --net=host --device=/dev/net/tun --cap-add NET_ADMIN -v /opt/darkmesh/conf/:/etc/tinc/darkmesh --entrypoint tincd tinc:0.0.1 -n darkmesh -K4096


vim hosts/MINODO

Address=DIRECCIONPUBLICA (cosa.midominio.net)
Cipher=blowfish
Compression=0
Digest=sha1
IndirectData=no
Port=655
Subnet=IPENDARKMESH/32
TCPonly=yes
-----BEGIN RSA PUBLIC KEY-----
LA CLAVE PUBLICA GENERADA ANTES
estará en /opt/darkmesh/conf/rsa_key.pub
Copia el contenido aqui
-----END RSA PUBLIC KEY-----
```

### Comparte el fichero de tu host con el rersto de la red a través de git
```bash
cp hosts/MINODO ~/workspace/darkmesh/hosts/MINODO
cd ~/workspace/darkmesh/
git add hosts/MINODO
git commit "Here is Me trying to join to the darkmesh"
git push
```

### Configura tu servidor 
```bash
cd /opt/darkmesh/
vim tinc.conf

Name=MINODO
Hostnames=no
Port=655
Mode=router
AddressFamily=ipv4
PingTimeout=60
Mode=router
TCPOnly = yes
StrictSubnets=no
ConnectTo=OTRONODOCONOCIDODELARED

vim tinc-up
#!/bin/sh
ifconfig $INTERFACE IPENDARKMESH netmask MASCARADELMESH


docker run --rm -d --name tinc --net=host --device=/dev/net/tun --cap-add NET_ADMIN -v /opt/darkmesh/conf/:/etc/tinc/darkmesh tinc:0.0.1  start -D -d 5 -n darkmesh
``` 


# Conectarse a darkmesh (usando contenedores docker)

## Llevar la configuración inicial a tu servidor
```bash
mkdir ~/workspace/
cd ~/workspace/
git clone ssh://git.darkme.sh:2222/millaguie/darkmesh.git
```
## Copiar la configuración inicial a un directorio del servidor
```bash
cp -a darkmesh /opt/darkmesh
```

## Construir la imagen docker para tinc (no necesario si usas AMD64)
```bash
cd ~/workspace/
git clone https://github.com/millaguie/docker-tinc.git
cd docker-tinc
docker build -t tinc:0.0.1 .
```

## Generar el fichero de configuración de tu servidor
```bash
cd /opt/darkmesh/
docker run --rm -ti --name tinc --net=host --device=/dev/net/tun --cap-add NET_ADMIN -v /opt/darkmesh/conf/:/etc/tinc/darkmesh --entrypoint tincd tinc:0.0.1 -n darkmesh -K4096


vim hosts/MIHOST

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

vim /root/tinc/etc/darkmesh/tinc-up
#!/bin/sh
ifconfig $INTERFACE IPENDARKMESH netmask MASCARADELMESH


docker run --rm -d --name tinc --net=host --device=/dev/net/tun --cap-add NET_ADMIN -v /root/tinc/etc/:/etc/tinc/ -v /root/etc/hosts:/etc/tinc/hosts  tinc:0.0.1  start -D -d 5 -n darkmesh
``` 

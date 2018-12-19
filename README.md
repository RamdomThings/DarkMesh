
# Conectarse a darkmesh (usando contenedores docker)
## Prerequisitos
* Tener docker instalado en la raspberry, siguiendo por ejemplo esta documentación: https://docs.docker.com/install/linux/docker-ce/debian/ (recuerda que raspberry es armhf)
* Estar dado de alta en https://git.darme.sh.
* Subir tu llave ssh pública a https://git.darme.sh para poder usar git por ssh.
* Haber pedido al grupo el direccionamiento a usar.
* Contactar con algún otro nodo para que actualice sus hosts cuando subas el tuyo y así poderte conectar.

Nota, las palabras en mayúsculas como MINODO hacen referencia a "variables" han de ser cambiadas.

## Llevar la configuración inicial a tu servidor
```bash
mkdir ~/workspace/
cd ~/workspace/
git clone ssh://git@git.darkme.sh:2222/darkmesh/darkmesh.git
```
## Copiar la configuración inicial a un directorio del servidor
```bash
cp -a darkmesh /opt/darkmesh
```

## Construir la imagen docker para tinc (no necesario si usas AMD64)
```bash
cd ~/workspace/
git clone ssh://git@git.darkme.sh:2222/darkmesh/docker-tinc.git
cd docker-tinc
# El paso siguiente va a tardar un buen rato, tomate un algo
docker build -t tinc:0.0.1 . 
```

## Configura tu servidor 
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

## Generar el fichero de configuración de tu nodo
```bash
cd /opt/darkmesh/
# cuando pregunte en el siguiente paso dile que si a las opciones por defecto
docker run --rm -ti --name tinc --net=host --device=/dev/net/tun --cap-add NET_ADMIN -v /opt/darkmesh:/etc/tinc/darkmesh --entrypoint tincd tinc:0.0.1 -n darkmesh -K4096


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
Probablemente la key publica se haya añadido automáticamente
y solamente tengas que borrar estas líneas
-----END RSA PUBLIC KEY-----
```

## Comparte el fichero de tu host con el rersto de la red a través de git
```bash
cp hosts/MINODO ~/workspace/darkmesh/hosts/MINODO
cd ~/workspace/darkmesh/
git add hosts/MINODO
git commit "Here is Me trying to join to the darkmesh"
git push
```

## Arranca el mesh

docker run --rm -d --name darkmes --net=host --device=/dev/net/tun --cap-add NET_ADMIN -v /opt/darkmesh:/etc/tinc/darkmesh tinc:0.0.1  start -D -d 5 -n darkmesh
``` 

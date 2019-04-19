
# Conectarse a darkmesh (usando contenedores docker)
## Prerequisitos
* Tener docker instalado en la raspberry, siguiendo por ejemplo esta documentación: https://docs.docker.com/install/linux/docker-ce/debian/ (recuerda que raspberry es armhf)
* Estar dado de alta en https://git.darme.sh.
* Subir tu llave ssh pública a https://git.darme.sh para poder usar git por ssh. ( -t dsa da problemas)
* Pedir permiso para escribir en el repo
* Haber pedido al grupo el direccionamiento IP a usar.
* Contactar con algún otro nodo para que actualice sus hosts cuando subas el tuyo y así poderte conectar.

Nota, las palabras en mayúsculas como MINODO hacen referencia a "variables" han de ser cambiadas.

## Configurar globals si no lo has hecho ya
```
git config --global user.email "tu@email.com"
git config --global user.name "Tu nombre"
```

## Llevar la configuración inicial a tu servidor
```bash
mkdir ~/workspace/
cd ~/workspace/
git clone ssh://git@git.darkme.sh:2222/darkmesh/darkmesh.git
```
## Copiar la configuración inicial a un directorio del servidor (con poderes de root)
```bash
mkdir /opt
cp -a darkmesh /opt/darkmesh
chown TUUSUARIO:TUGRUPO /opt/darkmesh -R
```

## Construir la imagen docker para tinc (no necesario si usas AMD64)
```bash
cd ~/workspace/
git clone ssh://git@git.darkme.sh:2222/darkmesh/docker-tinc.git
cd docker-tinc
# El paso siguiente va a tardar un buen rato, tomate un algo (con poderes de root)
docker build . -t tinc:1.1pre
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
ConnectTo=darkening
ConnectTo=OTRONODOCONOCIDODELARED

vim tinc-up
#!/bin/sh
ifconfig $INTERFACE IPENDARKMESH netmask 255.255.0.0

## Generar el fichero de configuración de tu nodo
```bash
cd /opt/darkmesh/
# cuando pregunte en el siguiente paso dile que si a las opciones por defecto (con poderes de root)
sudo docker run --rm -ti --name tinc --net=host --device=/dev/net/tun --cap-add NET_ADMIN -v /opt/darkmesh:/etc/tinc/darkmesh --entrypoint tinc tinc:1.1pre -n darkmesh generate-ed25519-keys

vim hosts/MINODO

Address=DIRECCIONPUBLICA (cosa.midominio.net)
Cipher=blowfish
Compression=0
Digest=sha1
IndirectData=no
Port=655
Subnet=IPENDARKMESH/32
TCPonly=yes
Ed25519PublicKey = LaKeyApareceráAutomáticamenteAquíAlGenerarlaNoLacambies
```

## Comparte el fichero de tu host con el resto de nodos de la red a través de git
```bash
cp hosts/MINODO ~/workspace/darkmesh/hosts/MINODO
cd ~/workspace/darkmesh/
# Cada vez que alguien sube a git una rama que se llama MIRAMAPARAUNIRMEALMESH dios mata a un gatito
git checkout -b MIRAMAPARAUNIRMEALMESH
git add hosts/MINODO
git commit -m "Nuevo nodo MINODO con IP MIIP"
git push --set-upstream origin MIRAMAPARAUNIRMEALMESH
```

Ahora puedes ir a https://git.darkme.sh/darkmesh/darkmesh/ y pedir pull :) es el momento de avisar a alguien con poderes de administración que compruebe la petición y si todo está bien, acepte el pull. Automáticamente el nodo darkening se actualizará con el host nuevo.

## Arranca el mesh (con poderes de root)
```bash
docker run --rm -d --name darkmes --net=host --device=/dev/net/tun --cap-add NET_ADMIN -v /opt/darkmesh:/etc/tinc/darkmesh -e MESH=darkmesh -e GIT_URL="git.darkme.sh/darkmesh/darkmesh" tinc:1.1pre
``` 
Cada vez que reinicies el contenedor se conectará al servidor git para actualizar el listado de nodos.

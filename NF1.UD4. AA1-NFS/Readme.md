# Compartició d'arxius i carpetes

Als sistemes *nix*, com Linux, la compartició d'arxius i carpetes es pot realitzar mitjançant diversos protocols, com ara `NFS` (Network File System) o `Samba` (SMB/CIFS). Mentre que el primer és natiu del món Unix, el segon va néixer per oferir compatibilitat amb sistemes Windows.

En aquesta unitat estudiarem el protocol `NFS`, deixant Samba per més endanvant, quan veiem la integració de Linux amb Windows.

## Què és NFS?

NFS (Network File System) és un protocol de compartició de fitxers que permet als usuaris accedir a fitxers i directoris en un servidor remot com si fossin locals. Malgrat el seu nom, no és un sistema de fitxers en si mateix, sinó un protocol que permet muntar sistemes de fitxers remots a la màquina local.

Aquest protocol va ser desenvolupat per Sun Microsystems a principis dels anys 80 per a permetre la compartició de fitxers entre sistemes Unix, sent adoptat àmpliament en entorns Linux i altres sistemes operatius com els derivats de BSD i macOS. Fins i tot Windows ofereix suport per a NFS tant a nivell de servidor com de client, encara que no és tan comú com el protocol SMB. Es troba definit a la RFC 1094.

El seu funcionament es basa a les següents característiques:

- **Model client-servidor**: NFS segueix un model client-servidor, on el servidor NFS comparteix directoris i els clients NFS poden muntar aquests recursos compartits com si fossin part del seu sistema de fitxers local.

- **Transparència**: Els usuaris poden accedir als fitxers remots com si estiguessin en el seu propi sistema, sense necessitat de conèixer la ubicació física dels fitxers.

- **Crides RPC (Remote Procedure Call)**: NFS utilitza crides RPC per a la comunicació entre el client i el servidor, permetent que les operacions de fitxers es realitzin de manera remota.

Pel que fa les versions, les dues que s'usen a l'actualitat són:

- **NFSv3**: És la versió més utilitzada i estable, que ofereix suport per a operacions de lectura i escriptura, així com per a la gestió de permisos i propietats dels fitxers. No disposa però de mecanismes de seguretat com el xifrat.

- **NFSv4**: És la versió més recent i completa, que ofereix millors funcionalitats de seguretat, com ara autenticació, les llistes de control d'accés (ACLs) i xifrat.

Les opcions d'exportació de NFS es configuren al fitxer /etc/exports per definir els permisos i les restriccions dels directoris compartits. Cada directori es defineix amb la sintaxi: `directori client(opcions)`

Les opcions més utilitzades es classifiquen segons la seva funció:

1. **Permisos de Lectura/Escriptura**

    - rw (Read/Write): Permet operacions tant de lectura com d'escriptura en el volum.
    - ro (Read-Only): Atorga accés de només lectura (és l'opció predeterminada).

2. **Rendiment vs. Seguretat de dades**
    - sync: Força el servidor a esperar que els canvis s'escriguin al disc físic abans de respondre al client. Garanteix la integritat de les dades en cas de caigudes sobtades del servidor.
    - async: Permet al servidor respondre a les sol·licituds abans que els canvis s'escriguin a l'emmagatzematge estable. Millora el rendiment, però augmenta el risc de pèrdua de dades si hi ha fallades inesperades.

3. **Mapeig d'Usuaris (User Squashing)**

    - root_squash: És l'opció predeterminada. Si un usuari root accedeix al recurs compartit des d'un client, el servidor el degrada i li assigna l'ID d'un usuari anònim (ex. nfsnobody). Això evita que el root remot tingui accés total als fitxers.
    - no_root_squash: Desactiva la degradació, permetent que l'usuari root del client mantingui privilegis de superusuari al servidor. Fes-la servir amb precaució, ja que presenta un risc de seguretat elevat.
    - all_squash: Mapeja tots els usuaris (independentment del seu UID) a l'usuari anònim. És ideal per a directorios compartits públics.

4. **Accessos i Xarxes**
    Permet definir hosts específics, subxarxes o dominis. Exemples:
    - 192.168.1.10(rw,sync): Dóna accés només a una IP específica.
    - 192.168.1.0/24(rw,sync): Atorga accés a tota una xarxa.
    - \*(rw,sync): L'asterisc \* permet que qualsevol client accedeixi al recurs.

## Recordatori sistema permisos UGO

Als sistemes Unix, els permisos d'accés als fitxers i directoris es gestionen mitjançant un model de permisos conegut com a UGO (User, Group, Others). Aquest model defineix tres tipus d'usuaris:

- **User (U)**: L'usuari propietari del fitxer o directori.
- **Group (G)**: El grup propietari del fitxer o directori.
- **Others (O)**: Tots els altres usuaris que no són ni el propietari ni pertanyen al grup del fitxer o directori.

És a dir, que cada fitxer o carpeta té associats un **usuari propietari**, un **grup propietari**.

A Sistemes Operatius Monolloc, ja vam veure com es poden canviar la propietat d'un fitxer o carpeta amb la comanda `chown` que permet canviar l'usuari i el grup o la comanda `chgrp` que permet canviar només el grup.

```bash
# Canviar propietari i grup d'un fitxer
chown usuari:grup fitxer
# Canviar només el propietari d'un fitxer
chown usuari fitxer
# Canviar només el grup d'un fitxer
chgrp grup fitxer
chown :grup fitxer

# Canviar propietari i grup d'una carpeta i tot el seu contingut
chown -R usuari:grup carpeta

# Comprovar propietari i grup d'un fitxer o carpeta
ls -l fitxer
ls -l carpeta
```

Els permisos d'accés es representen amb tres bits per a cada tipus d'usuari, que indiquen si tenen permís de lectura (r), escriptura (w) i execució (x). D'aquesta manera, els permisos es poden expressar de la següent manera:

| Permís  | Significat                                                                                                       |
|---------|------------                                                                                                      |
| r       | Permís de lectura: l'usuari pot llegir el contingut del fitxer o llistar el contingut del directori.             |
| w       | Permís d'escriptura: l'usuari pot modificar el contingut del fitxer o afegir/eliminar fitxers dins del directori.|
| x       | Permís d'execució: l'usuari pot executar el fitxer o accedir al directori.                                       |

El permís d'accés per cada tipus d'usuari es pot representar amb una combinació de tres bits, on cada bit correspon a un permís específic. Per exemple, els permisos `rwx` es representen com `111` en binari, mentre que els permisos `rw-` es representen com `110`. Aquests permisos es poden expressar en forma octal, on cada combinació de tres bits es converteix en un dígit octal (0-7).

Per exemple:

| Permisos | Binari | Octal |
|----------|--------|-------|
| rwx      | 111    | 7     |
| rw-      | 110    | 6     |
| r-x      | 101    | 5     |
| r--      | 100    | 4     |

Per modificar o establir els permisos d'accés a fitxers i directoris, es pot utilitzar la comanda `chmod`. Aquesta comanda permet canviar els permisos d'accés de manera simbòlica o numèrica.

```bash
# Canviar permisos d'accés de manera simbòlica
# Afegir permís d'escriptura per al grup
chmod g+w fitxer
# Treure permís d'execució per a altres usuaris
chmod o-x fitxer

# Establir permisos d'accés de manera numèrica
chmod 755 fitxer

## Aplicar permisos d'accés a una carpeta i tot el seu contingut
chmod -R 775 carpeta
```

## Permisos d'accés a recursos compartits NFS

Per defecte, NFS no valida els usuaris, sinó que confia en el sistema de permisos del sistema de fitxers local. Això significa que els permisos d'accés als fitxers i directoris compartits a través de NFS es basen en els permisos establerts al sistema de fitxers del servidor NFS usant el UID i el GID definits a la propietat del fitxer o directori. Per tant, és important assegurar-se que els usuaris i grups tinguin els mateixos UID i GID tant al servidor com als clients NFS per garantir un accés correcte.

Posem el següent exemple per il·lustrar millor:

Al servidor NFS tenim definits els següents usuaris (UID i GID):

| Usuari | UID  | GID          |
|--------|----- |-----         |
| joan   | 1001 | 1001 (joan)  |
| bob    | 1002 | 1310 (users) |
| pep    | 1004 | 1310 (pep)   |

A una màquina client els usuaris definits són:

| Usuari | UID  | GID          |
|--------|----- |------------  |
| pep    | 1001 | 1001 (pep)   |
| joan   | 1002 | 1310 (users) |

Suposem una carpeta anomenada `recurs` amb propietat de joan i grup 1310 amb permisos 750. Quin tipus d’accés tindran els usuaris d’un host al que s’ha compartit el recurs com rw?

Com el que importa és el UID i GID, i no el nom d'usuari, els permisos d'accés es determinaran en funció dels UID i GID associats a cada usuari. En aquest cas:

|Accés  | Servidor NFS    | Client NFS       |
|-------|--------------   |------------      |
| RW    | 1001 (joan)     | 1001 (pep)       |
| RO    | 1002 (bob)      | 1310 (joan)      |

És per aquest motiu que NFS en entorns corporatius requereix un sistema d'usuaris centralitzat, com ara **LDAP** o **Active Directory**, per garantir que els UID i GID siguin coherents a través de tots els sistemes que accedeixen als recursos compartits. És l'acció necessària si volem definir perfils mòbils d'usuari amb clients Linux.

Per entorns més petits, pot ser suficient assegurar-se que els UID i GID coincideixin manualment entre el servidor i els clients NFS, per exemple usant scripts per automatitzar la creació d'usuaris i grups amb els mateixos UID i GID a tots els sistemes.

En qualsevol cas, fixeu-vos que si necessiteu donar permisos diferents a diferents grups, el sistema UGO no és suficient, i caldrà usar ACLs (Access Control Lists) per establir permisos més detallats, definint permisos específics per a usuaris i grups concrets, independentment de la seva relació amb el propietari del fitxer o directori.

> **Nota**: Com a curiositat, Windows Server permet crear un servidor NFS basat en els usuaris del directori actiu, de manera que els UID i GID es poden mapear amb els usuaris i grups de Windows. Això permet una integració més fluida entre entorns Linux i Windows quan es comparteixen recursos mitjançant NFS. De la mateixa manera, els clients Windows incorporen dins les característique opcionals un client NFS.

## Instal·lació i configuració d'un servidor NFS

### Preparació de l'entorn

Per aquest laboratori, necessitarem un servidor NFS (Ubuntu Server) i un client NFS (Zorin OS), totes es dues es veuran per la interfície comuna (xarxa NAT o adaptador pont segons sigui el cas).

Abans de començar amb els escenaris, cal instal·lar els paquets necessaris i assegurar-nos que les màquines es comuniquen.

### Instal·lació servei NFS

```bash
# Instal·lar el servidor NFS
sudo apt update
sudo apt install nfs-kernel-server -y
```

### Instal·lació client NFS

```bash
# Instal·lar el client NFS
sudo apt update
sudo apt install nfs-common -y
```

### Exemple senzill: Compartició bàsica d'una carpeta

Compartim una carpeta per un usuari i grup concret.

#### Configuració al servidor

Creació de l'usuari, el grup i la carpeta compartida amb els permisos adequats:

```bash
#Creem el grup i l'usuari que serà propietari del recurs:
sudo groupadd -g 1100 grup_smx
sudo useradd -u 1100 -m -g grup_smx usuari_smx

# Crear una carpeta per compartir
sudo mkdir -p /srv/nfs/compartit
# Canviar propietari i grup de la carpeta
sudo chown joan:users /srv/nfs/compartit
# Establir permisos d'accés a la carpeta
sudo chmod 750 /srv/nfs/compartit
```

Editem l'arxiu `/etc/exports` per definir els recursos compartits i els permisos d'accés:

```plain
/srv/nfs/compartit *(rw,sync,no_subtree_check)
```

Apliquem els canvis i reiniciem el servei NFS:

```bash
sudo exportfs -a
sudo systemctl restart nfs-kernel-server
```

#### Configuració al client

Cal crear l'usuari i el grup amb els mateixos UID i GID que al servidor per assegurar la coherència dels permisos:

```bash
#Creem el grup i l'usuari que serà propietari del recurs:
sudo groupadd -g 1100 grup_smx
sudo useradd -u 1100 -m -g grup_smx usuari_smx -s /bin/bash
```

Cal crea el punt de muntatge i muntar la carpeta compartida:

```bash
# Crear el punt de muntatge
sudo mkdir -p /mnt/compartit
# Muntar la carpeta compartida
sudo mount -t nfs servidor_nfs:/srv/nfs/compartit /mnt/compartit
```

Comprovem que l'usuari creat pot accedir a la carpeta compartida i crear fitxers dins d'ella:

```bash
# Canviem a l'usuari creat
su - usuari_smx
# Comprovar accés a la carpeta compartida
cd /mnt/compartit
# Crear un fitxer de prova
touch fitxer_prova.txt
# Comprovar que el fitxer s'ha creat correctament
ls -l fitxer_prova.txt
```

Un cop accedit, desmuntem la carpeta compartida:

```bash
# Desmuntar la carpeta compartida
sudo umount /mnt/compartit
```

Aquí hem vist un muntatge temporal de la carpeta compartida (si tanqueu sessió es perd el muntatge). Si volem que el muntatge sigui permanent i automàtic, cal afegir una entrada a l'arxiu `/etc/fstab` del client:

```plain
IP_servidor_nfs:/srv/nfs/compartit /mnt/compartit nfs defaults 0 0
```

> **Explicació de l'entrada:**
> Ruta del recurs compartit al servidor inclou la IP i la ruta absoluta.
> Ruta del punt de muntatge al client.
> Tipus de sistema de fitxers (nfs).
> Opcions de muntatge (defaults).
> 0 per a la verificació del sistema de fitxers (no es comprovarà).
> 0 per a l'ordre de muntatge (no es muntarà en ordre específic).

Un cop editat l'arxiu, podem muntar totes les entrades de `/etc/fstab` amb la comanda:

```bash
sudo mount -a
```

Si ara iniceu sessió amb l'usuari creat a Zorin OS, podreu accedir a la carpeta compartida de manera automàtica.

## Enllaços d'interès

- [RFC 1094: NFS: Network File System Protocol specification](https://www.rfc-editor.org/info/rfc1094/)

- [LinuxConfig.org: How to Setup NFS Server and Client on Ubuntu 26.04](https://linuxconfig.org/how-to-setup-nfs-server-and-client-on-ubuntu-26-04)

- [Ubuntu Server Documentation: Network File System (NFS)](https://ubuntu.com/server/docs/how-to/networking/install-nfs/)

- [Stack Harbor. *NFSv4 ACLs and permissions: making fine-grained access work end to end*](https://stackharbor.com/en/knowledge-base/nfs-v4-acls-permissions/)
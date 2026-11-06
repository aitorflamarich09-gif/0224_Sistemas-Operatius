### Fase 1. Preparació de l'entorn

Per al client utilitzarem Zorin OS 18 amb configuració de "xarxa NAT". El servidor tindrà una interfície amb "xarxa només-amb-amfitrió" i l'altra amb "xarxa NAT". Això permetrà que el client tingui accés a Internet i que el servidor pugui comunicar-se amb ell.

![Configuració de la xarxa NAT](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20184253.png)

![Configuració de la màquina virtual](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20183849.png)

Ens assegurarem que ambdues màquines tinguin accés a Internet i que es puguin comunicar entre elles a través de la xarxa només-amb-amfitrió. També actualitzarem els sistemes amb les últimes actualitzacions disponibles.

### Fase 2: Preparació del servidor

Abans de compartir recursos, hem de preparar els usuaris i els directoris al servidor.

1. Creació de grups:
   Crear dos grups per al client: `devs` i `admins`.

![Creació de grups al servidor](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20184343.png)

![Creació de grups al client](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20184435.png)

2. Creació d'usuaris:
   Crear un usuari `dev01` (membre del grup `devs`).

![Creació d'usuaris del servidor](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20184453.png)

![Afegir usuaris als grups del client](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20184528.png)

3. Crear un usuari `admin01` (membre del grup `admins`).

![Usuari admin01 a Zorin](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20184711.png)

![Usuari admin01 a Ubuntu](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20184726.png)

4. Creació de directoris (al servidor):
   - Crear el directori per als projectes de desenvolupament: `/srv/nfs/dev_projects`
   - Crear el directori per a les eines d'administració: `/srv/nfs/admin_tools`

![Creació de directoris](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20184739.png)

![Directori admin_tools i dev_projects](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20185218.png)

5. Permisos del servidor:
   - Els developers tindran control sobre els seus projectes.
   - Els administradors tindran control sobre les seves eines.
   - En ambdós casos, l'usuari propietari serà `root`.

![Permisos de devs](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20185229.png)

![Permisos d'admins](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20185239.png)

6. Instal·lació del servei NFS.

![Instal·lació del servei NFS](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20185324.png)

> Nota: Perquè aquesta pràctica funcioni correctament, cal replicar aquests usuaris i grups al client i assegurar-se que UID i GID coincideixen a les dues màquines.

### Fase 3: L'exportació d'administració (el dilema de `root_squash`)

![Exportació del directori admin_tools](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20185403.png)

El client necessita que `/srv/nfs/admin_tools` sigui accessible per l'equip d'administradors. A vegades, l'usuari `root` del client necessitarà escriure en aquest directori per instal·lar eines. Aquí mostrarem un error típic i la seva solució.

#### Prova 1 (l'error comú)

1. Exportar el directori `/srv/nfs/admin_tools` amb les opcions `rw,sync`.

![Configuració de l'exportació a Zorin](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20185428.png)

![Configuració de l'exportació al client](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20185448.png)

2. Des del client, muntar aquest recurs compartit a `/mnt/admin_tools`. Com a root del client, intentar crear un fitxer dins d'aquest directori muntat.

![Muntatge de /mnt/admin_tools](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20185502.png)

3. Verificar quin és el propietari del fitxer creat. Per què? Justificar la resposta amb l'explicació tècnica de `root_squash`.

![Resultat de root_squash](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20185533.png)

#### Prova 2 (la solució)

1. Modificar l'exportació del directori `/srv/nfs/admin_tools` per incloure l'opció `no_root_squash`.

![Ajust de no_root_squash](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20185608.png)

2. Al client, desmuntar i tornar a muntar el recurs compartit.

![Desmuntatge i muntatge](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20184528.png)

3. Com a root del client, intentar crear un fitxer dins d'aquest directori muntat. Observeu quin és el propietari del fitxer creat aquesta vegada. Ha canviat alguna cosa? Justificar la resposta amb l'explicació tècnica de `no_root_squash`.

![Solució amb no_root_squash](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20185239.png)

### Fase 4: L'exportació de desenvolupament (permisos `rw` vs `ro`)

![Configuració de permisos rw/ro](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20185448.png)

1. Editar `/etc/exports` per afegir dues exportacions per al mateix directori. El client vol que la xarxa d'administració pugui escriure-hi, però que la xarxa de consultors només la pugui llegir.

![IP de xarxa i configuració](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20185448.png)

2. Des del client, muntar el recurs compartit a `/mnt/dev_projects` i provar d'escriure-hi com a usuari `dev01`.

![Muntatge de /mnt/dev_projects](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20185502.png)

3. Desmuntar el recurs i canviar la IP del client a 192.168.56.150. Tornar a provar d'escriure al directori muntat com a usuari `dev01` i comprovar que només funciona la lectura.

![Comprovació de lectura només](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20185533.png)

4. Canvieu d'usuari al client a `admin01` i torneu a provar d'escriure al directori muntat. Comproveu que no es pot escriure, ja que `admin01` no és membre del grup `devs` i els permisos locals del sistema de fitxers no ho permeten.

![No pot escriure admin01](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20185608.png)

### Fase 5: Muntatge automàtic amb `/etc/fstab`

És evident que els usuaris no poden estar muntant manualment els recursos compartits cada vegada que reinicien el sistema. Per això, es configura el muntatge automàtic mitjançant `/etc/fstab` al client.

![Configuració de /etc/fstab](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20190210.png)

1. Editar el fitxer `/etc/fstab` al client per afegir les entrades necessàries per muntar automàticament els recursos compartits NFS durant l'inici del sistema.

![Edició del fitxer /etc/fstab](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20191417.png)

2. Executar la comanda `mount -a` per provar les entrades sense reiniciar.

![Prova amb mount -a](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20192820.png)

3. Reiniciar el client i verificar que els recursos compartits s'han muntat correctament.

![Verificació després del reinici](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20192900.png)

### Resum

Aquest exercici demostra com configurar un servidor NFS amb usuaris, grups, permisos i exportacions, i com controlar l'accés amb `root_squash`, `no_root_squash`, i permisos `rw`/`ro`. La clau és que el sistema de fitxers local del client també aplica les seves pròpies regles de permisos i que NFS només determina qui pot accedir a l'estructura compartida des de la xarxa.

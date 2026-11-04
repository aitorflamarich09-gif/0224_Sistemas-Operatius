### Fase 1. Preparació de l'entorn

Per al client utilitzarem Zorin OS 18 amb configuració de "xarxa NAT". El servidor tindrà una interfície amb "xarxa només-amb-amfitrió" i l'altra amb "xarxa NAT". Això permetrà que el client tingui accés a Internet i que el servidor pugui comunicar-se amb ell.

Ens assegurarem que ambdues màquines tinguin accés a Internet i que es puguin comunicar entre elles a través de la xarxa només-amb-amfitrió. També actualitzarem els sistemes amb les últimes actualitzacions disponibles.

### Fase 2: Preparació del servidor

Abans de compartir recursos, hem de preparar els usuaris i els directoris al servidor.

1. Creació de grups:
   Crear dos grups per al client: `devs` i `admins`.

2. Creació d'usuaris:
   Crear un usuari `dev01` (membre del grup `devs`).

3. Crear un usuari `admin01` (membre del grup `admins`).

4. Creació de directoris (al servidor):
   - Crear el directori per als projectes de desenvolupament: `/srv/nfs/dev_projects`
   - Crear el directori per a les eines d'administració: `/srv/nfs/admin_tools`

5. Permisos del servidor:
   - Els developers tindran control sobre els seus projectes.
   - Els administradors tindran control sobre les seves eines.
   - En ambdós casos, l'usuari propietari serà `root`.

6. Instal·lació del servei NFS.

> Nota: Perquè aquesta pràctica funcioni correctament, cal replicar aquests usuaris i grups al client i assegurar-se que UID i GID coincideixen a les dues màquines.

### Fase 3: L'exportació d'administració (el dilema de `root_squash`)

El client necessita que `/srv/nfs/admin_tools` sigui accessible per l'equip d'administradors. A vegades, l'usuari `root` del client necessitarà escriure en aquest directori per instal·lar eines. Aquí mostrarem un error típic i la seva solució.

#### Prova 1 (l'error comú)

1. Exportar el directori `/srv/nfs/admin_tools` amb les opcions `rw,sync`.

2. Des del client, muntar aquest recurs compartit a `/mnt/admin_tools`. Com a root del client, intentar crear un fitxer dins d'aquest directori muntat.

3. Verificar quin és el propietari del fitxer creat. Per què? Justificar la resposta amb l'explicació tècnica de `root_squash`.

#### Prova 2 (la solució)

1. Modificar l'exportació del directori `/srv/nfs/admin_tools` per incloure l'opció `no_root_squash`.

2. Al client, desmuntar i tornar a muntar el recurs compartit.

3. Com a root del client, intentar crear un fitxer dins d'aquest directori muntat. Observeu quin és el propietari del fitxer creat aquesta vegada. Ha canviat alguna cosa? Justificar la resposta amb l'explicació tècnica de `no_root_squash`.

### Fase 4: L'exportació de desenvolupament (permisos `rw` vs `ro`)

1. Editar `/etc/exports` per afegir dues exportacions per al mateix directori. El client vol que la xarxa d'administració pugui escriure-hi, però que la xarxa de consultors només la pugui llegir.

2. Des del client, muntar el recurs compartit a `/mnt/dev_projects` i provar d'escriure-hi com a usuari `dev01`.

3. Desmuntar el recurs i canviar la IP del client a 192.168.56.150. Tornar a provar d'escriure al directori muntat com a usuari `dev01` i comprovar que només funciona la lectura.

4. Canvieu d'usuari al client a `admin01` i torneu a provar d'escriure al directori muntat. Comproveu que no es pot escriure, ja que `admin01` no és membre del grup `devs` i els permisos locals del sistema de fitxers no ho permeten.

### Fase 5: Muntatge automàtic amb `/etc/fstab`

És evident que els usuaris no poden estar muntant manualment els recursos compartits cada vegada que reinicien el sistema. Per això, es configura el muntatge automàtic mitjançant `/etc/fstab` al client.

1. Editar el fitxer `/etc/fstab` al client per afegir les entrades necessàries per muntar automàticament els recursos compartits NFS durant l'inici del sistema.

2. Executar la comanda `mount -a` per provar les entrades sense reiniciar.

3. Reiniciar el client i verificar que els recursos compartits s'han muntat correctament.

### Resum

Aquest exercici demostra com configurar un servidor NFS amb usuaris, grups, permisos i exportacions, i com controlar l'accés amb `root_squash`, `no_root_squash`, i permisos `rw`/`ro`. La clau és que el sistema de fitxers local del client també aplica les seves pròpies regles de permisos i que NFS només determina qui pot accedir a l'estructura compartida des de la xarxa.

### Fase 1. Preparació de l'entorn

Per al client utilitzarem Zorin OS 18 amb configuració de xarxa NAT. El servidor tindrà una interfície amb xarxa només amb amfitrió i l'altra amb xarxa NAT. Això permetrà que el client tingui accés a Internet i que el servidor pugui comunicar-se amb ell.

<<<<<<< HEAD
![Configuració de la xarxa NAT](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20184253.png)

![Configuració de la màquina virtual](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20183849.png)

Ens assegurarem que ambdues màquines tinguin accés a Internet i que es puguin comunicar entre elles a través de la xarxa només-amb-amfitrió. També actualitzarem els sistemes amb les últimes actualitzacions disponibles.
=======
Ens assegurarem que ambdues màquines tinguin accés a Internet i es puguin comunicar a través de la xarxa només amb amfitrió. També actualitzarem els sistemes.

![Xarxa NAT de Zorin a VirtualBox](fotos/Xarxa_nat_Zorin.png)

Sense comanda visible (captura de configuració gràfica).

![Xarxa NAT d’Ubuntu a VirtualBox](fotos/Xarxa_nat_ubuntu.png)

Sense comanda visible (captura de configuració gràfica).
>>>>>>> ea0342a3b8df5229d627b5e086a4e8092a3cc0e6

### Fase 2: Preparació del servidor

Abans de compartir recursos, preparem els usuaris, els grups i els directoris al servidor i al client.

1. Creació de grups: `devs` i `admins`.

<<<<<<< HEAD
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
=======
   **Servidor:**
   ![Creació dels grups al servidor](fotos/01-grupos-servidor.png)

   ```bash
   sudo addgroup devs && admins
   sudo addgroup admins
   ```

   **Client:**
   ![Creació dels grups al client](fotos/02-grupos-cliente.png)

   ```bash
   sudo addgroup devs
   sudo addgroup admins
   ```

2. Creació d'usuaris `dev01` i `admin01`.

   **Client — assignació als grups:**
   ![Assignació de dev01 a devs i admin01 a admins al client](fotos/03-usuarios-grupos-cliente.png)

   ```bash
   sudo usermod -aG devs dev01
   sudo usermod -aG admins admin01
   ```

   **Servidor — creació d'usuaris:**
   ![Creació de dev01 i admin01 al servidor](fotos/04-usuarios-servidor.png)

   ```bash
   sudo adduser dev01
   sudo adduser admin01
   ```

3. Creació de directoris al servidor:

   ![Creació dels directoris al servidor](fotos/05-directorios-servidor.png)

   ```bash
   sudo mkdir -p /srv/nfs/devs_01
   sudo mkdir -p /srv/nfs/admin_01
   ```

4. Comprovació de propietaris i permisos:

   ![Comprovació de permisos dels directoris](fotos/06-permisos-directorios-servidor.png)

   ```bash
   ls -ld /srv/admins_projects
   ls -ld /srv/dev_projects
   ```

5. Instal·lació del servei NFS al servidor i del client NFS al client.

   **Servidor:**
   ![Instal·lació del servidor NFS](fotos/07-instalacion-nfs-servidor.png)

   ```bash
   sudo apt install nfs-kernel-server
   ```

   **Client:**
   ![Instal·lació del client NFS](fotos/08-instalacion-nfs-cliente.png)

   ```bash
   sudo apt install nfs-common
   ```

> Nota: Perquè aquesta pràctica funcioni correctament, cal replicar els usuaris i grups al client i assegurar que UID i GID coincideixen a les dues màquines.

### Fase 3: L'exportació d'administració (el dilema de `root_squash`)

El client necessita accedir a `/srv/nfs/admin_tools`. Aquesta fase mostra la configuració de les exportacions i les proves des del client.
>>>>>>> ea0342a3b8df5229d627b5e086a4e8092a3cc0e6

1. Al servidor, configurar i recarregar les exportacions NFS.

   ![Configuració i recàrrega de les exportacions NFS](fotos/09-exportaciones-servidor.png)

<<<<<<< HEAD
![Configuració de l'exportació a Zorin](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20185428.png)

![Configuració de l'exportació al client](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20185448.png)

2. Des del client, muntar aquest recurs compartit a `/mnt/admin_tools`. Com a root del client, intentar crear un fitxer dins d'aquest directori muntat.

![Muntatge de /mnt/admin_tools](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20185502.png)

3. Verificar quin és el propietari del fitxer creat. Per què? Justificar la resposta amb l'explicació tècnica de `root_squash`.

![Resultat de root_squash](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20185533.png)

#### Prova 2 (la solució)
=======
   ```bash
   sudo exportfs -ra
   sudo exportfs -v
   ```

2. Al client, provar l'escriptura indicada a la captura.

   ![Prova d'escriptura denegada al client](fotos/10-prueba-root-squash-cliente.png)
>>>>>>> ea0342a3b8df5229d627b5e086a4e8092a3cc0e6

   ```bash
   touch /mnt/dev_projects/prova_dev01.txt
   ```

<<<<<<< HEAD
![Ajust de no_root_squash](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20185608.png)

2. Al client, desmuntar i tornar a muntar el recurs compartit.

![Desmuntatge i muntatge](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20184528.png)

3. Com a root del client, intentar crear un fitxer dins d'aquest directori muntat. Observeu quin és el propietari del fitxer creat aquesta vegada. Ha canviat alguna cosa? Justificar la resposta amb l'explicació tècnica de `no_root_squash`.
=======
3. Al client, comprovar els recursos muntats.

   ![Comprovació dels muntatges NFS al client](fotos/11-prueba-no-root-squash-cliente.png)

   ```bash
   findmnt /mnt/admin_tools
   findmnt /mnt/dev_projects
   ```
>>>>>>> ea0342a3b8df5229d627b5e086a4e8092a3cc0e6

![Solució amb no_root_squash](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20185239.png)

### Fase 4: L'exportació de desenvolupament (permisos `rw` vs `ro`)

<<<<<<< HEAD
![Configuració de permisos rw/ro](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20185448.png)

1. Editar `/etc/exports` per afegir dues exportacions per al mateix directori. El client vol que la xarxa d'administració pugui escriure-hi, però que la xarxa de consultors només la pugui llegir.

![IP de xarxa i configuració](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20185448.png)

2. Des del client, muntar el recurs compartit a `/mnt/dev_projects` i provar d'escriure-hi com a usuari `dev01`.

![Muntatge de /mnt/dev_projects](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20185502.png)

3. Desmuntar el recurs i canviar la IP del client a 192.168.56.150. Tornar a provar d'escriure al directori muntat com a usuari `dev01` i comprovar que només funciona la lectura.

![Comprovació de lectura només](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20185533.png)

4. Canvieu d'usuari al client a `admin01` i torneu a provar d'escriure al directori muntat. Comproveu que no es pot escriure, ja que `admin01` no és membre del grup `devs` i els permisos locals del sistema de fitxers no ho permeten.
=======
1. Al client, provar de muntar el recurs NFS de desenvolupament.

   ![Intent de muntatge del recurs NFS al client](fotos/12-exportacion-ro-servidor.png)

   ```bash
   sudo mount 10.0.2.4:/srv/dev_projects /mnt/dev_projects
   ```

2. Com a `dev01`, provar d'escriure al directori compartit.

   ![Escriptura de dev01 al directori compartit](fotos/13-prueba-dev01-cliente.png)

   ```bash
   touch /mnt/dev_projects/prova_dev01.txt
   ```

3. Com a administrador al client, comprovar la denegació d'escriptura.

   ![Escriptura denegada al client](fotos/14-prueba-solo-lectura-cliente.png)

   ```bash
   sudo mkdir -p /mnt/dev_projects
   touch /mnt/dev_projects/prova_admin.txt
   ```

4. Es pot desmuntar el recurs amb:

   ![Desmuntatge del recurs NFS al client](fotos/19-desmuntar-recurs.png)

   ```bash
   sudo umount /mnt/dev_projects
   ```

5. Al servidor, assignar els grups propietaris com es mostra a la captura.

   ![Assignació dels grups propietaris al servidor](fotos/17-comandos-chown-servidor.png)

   ```bash
   sudo chown root:devs /srv/nfs/dev_tools
   sudo chown root:admins /srv/nfs/admin_tools
   ```
>>>>>>> ea0342a3b8df5229d627b5e086a4e8092a3cc0e6

![No pot escriure admin01](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20185608.png)

### Fase 5: Muntatge automàtic amb `/etc/fstab`

1. Configurar les entrades NFS al fitxer `/etc/fstab` del client.

<<<<<<< HEAD
![Configuració de /etc/fstab](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20190210.png)

1. Editar el fitxer `/etc/fstab` al client per afegir les entrades necessàries per muntar automàticament els recursos compartits NFS durant l'inici del sistema.

![Edició del fitxer /etc/fstab](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20191417.png)

2. Executar la comanda `mount -a` per provar les entrades sense reiniciar.

![Prova amb mount -a](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20192820.png)

3. Reiniciar el client i verificar que els recursos compartits s'han muntat correctament.
=======
   ![Configuració de fstab al client](fotos/18-configuracion-fstab-cliente.png)

   ```text
   192.168.56.101:/srv/admin_tools /mnt/admin_tools nfs defaults,_netdev 0 0
   192.168.56.101:/srv/dev_projects /mnt/dev_projects nfs defaults,_netdev 0 0
   ```

2. Executar `mount -a` per provar les entrades sense reiniciar.

   ![Prova de mount -a al client](fotos/16-mount-a-y-reinicio-cliente.png)

   ```bash
   sudo mount -a
   ```
>>>>>>> ea0342a3b8df5229d627b5e086a4e8092a3cc0e6

![Verificació després del reinici](Crea%20una%20carpeta/Captura%20de%20pantalla%202026-11-04%20192900.png)

### Resum

L'exercici mostra la configuració bàsica d'un servidor NFS, els usuaris i grups, els permisos, les exportacions i el muntatge al client. Les captures i els comandos transcrits s'han conservat tal com apareixen a les evidències.

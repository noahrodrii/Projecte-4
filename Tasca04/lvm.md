# Guia Tècnica d'Usuari: Gestió d'Emmagatzematge amb LVM a Zorin OS

Aquesta guia detalla el procediment complet per configurar, administrar, protegir i escalar volums d'emmagatzematge mitjançant **Logical Volume Manager (LVM)** en entorns Linux basats en Ubuntu/Debian (com **Zorin OS**).

---

## Preparació de l'Entorn i Requisits

Per dur a terme aquesta implementació sobre **Zorin OS**, cal disposar d'una màquina virtual amb accés a terminal (`sudo`) i incorporar els discos virtuals necessaris des del programari de virtualització (VirtualBox o VMware).

![Discos en virtualbox](/Tasca04/img/crea_disc.png)

![Discos totals](/Tasca04/img/discos_totals.png)


1. **Instal·lació de les utilitats LVM:**
   Obre el terminal i instal·la el paquet d'administració `lvm2`:

   ```bash
   sudo apt update && sudo apt install lvm2 -y
   ```
---
   > 💡 **Resolució d'incidències de xarxa:** Si la descàrrega falla per un error de resolució DNS (`Fallo temporal al resolver...`), assegura't que l'adaptador de xarxa de la màquina virtual estigui configurat en mode **NAT**.

---

2. **Identificació dels dispositius de bloc:**
   Abans de manipular discos, verifica quins noms els ha assignat el sistema per evitar tocar el disc principal del sistema operatiu (`/dev/sda`):

   ```bash
   lsblk
   ```

   ![lsblk](/Tasca04/img/lsblk.png)

---

## Fase 1: Configuració Inicial i Muntatge Persistent

En aquesta primera etapa es connecten **dos discos durs de 10 GB** (`/dev/sdb` i `/dev/sdc`) per construir la base de l'emmagatzematge i automatitzar-ne la disponibilitat en arrencar l'equip.

### 1.1. Inicialització dels Volums Físics i creació del Grup de Volums

1. Prepara els dos discos de 10 GB perquè formin part de l'ecosistema LVM:

   ```bash
   sudo pvcreate /dev/sdb /dev/sdc
   ```

   ![crear devs](/Tasca04/img/crea_devs.png)
   
2. Crea un grup de volums anomenat `vg_practica` agrupant tots dos discos (generant un espai total aproximat de 20 GB):

   ```bash
   sudo vgcreate vg_practica /dev/sdb /dev/sdc
   ```

   <br>

   > 🔔 **Comprovació:** Executa `sudo vgs` per veure un resum de l'espai total i lliure del grup, o `sudo vgdisplay` si necessites inspeccionar els detalls tècnics interns (com la mida dels *Physical Extents*).

### 1.2. Creació, Formatació i Muntatge Automàtic del Volum Lògic

1. Crea un primer volum lògic de **6 GB** anomenat `lv_inicial` i dota'l d'un sistema de fitxers `ext4`:

   ```bash
   sudo lvcreate -L 6G -n lv_inicial vg_practica
   sudo mkfs.ext4 /dev/vg_practica/lv_inicial
   ```

   ![crear volum](/Tasca04/img/lvcreate.png)

   ![sistema ext4](/Tasca04/img/mkfs.png)

2. Crea el directori destí i consulta l'identificador únic (**UUID**) del nou volum:

   ```bash
   sudo mkdir -p /mnt/inicial
   sudo blkid /dev/vg_practica/lv_inicial
   ```

   ![crear mount](/Tasca04/img/crear_mnt.png)

3. Per evitar haver de muntar el volum manualment després de cada reinici, edita l'arxiu `/etc/fstab` (`sudo nano /etc/fstab`) i afegeix-hi la línia següent al final (substituint el valor del teu UUID sense cometes):

   ```text
   UUID=el-teu-codi-uuid-aqui  /mnt/inicial  ext4  defaults  0  2
   ```

4. Aplica la configuració de muntatge immediatament i verifica que el volum està operatiu:

   ```bash
   sudo mount -av
   df -h /mnt/inicial
   ```

   ![mount](/Tasca04/img/mount.png)

---

## Fase 2: Alta Disponibilitat mitjançant Volum Mirall (`lvm_mirror`)

Per protegir informació crítica davant la fallada física d'un disc dur, LVM permet configurar volums en mirall (equivalent a un RAID 1 per programari). Cada bloc d'escriptura es duplica simultàniament en dos dispositius físics separats.

1. **Desplegament del volum mirall:**
   Aprofitant l'espai lliure restant a `/dev/sdb` i `/dev/sdc`, crea un volum de **3.9 GB** anomenat `lvm_mirror`. El modificador `-m 1` indica al sistema que mantingui una còpia rèplica addicional:

   > (*no son 4 GB perque lvm reserva una petita part per gestió interna*)

   ```bash
   sudo lvcreate -L 3.9G -m 1 -n lvm_mirror vg_practica
   ```

   ![lvm mirror](/Tasca04/img/crear_lvm_mirror.png)

2. **Formatació, muntatge i verificació d'escriptura:**

   ```bash
   sudo mkfs.ext4 /dev/vg_practica/lvm_mirror
   sudo mkdir -p /mnt/mirror
   sudo mount /dev/vg_practica/lvm_mirror /mnt/mirror
   echo "Informacio redundant protegida contra fallades de maquinari" | sudo tee /mnt/mirror/seguretat.txt
   ```

   ![lvm mirror](/Tasca04/img/mkfs_lvm_mirror.png)

   ![formatació etc](/Tasca04/img/formatació.png)

---
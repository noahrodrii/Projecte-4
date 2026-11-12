# Guia Tècnica d'Usuari: Gestió d'Emmagatzematge amb LVM a Zorin OS

Aquesta guia detalla el procediment complet per configurar, administrar, protegir i escalar volums d'emmagatzematge mitjançant **Logical Volume Manager (LVM)** en entorns Linux basats en Ubuntu/Debian (com **Zorin OS**).

---

## Preparació de l'Entorn i Requisits

Per dur a terme aquesta implementació sobre **Zorin OS**, cal disposar d'una màquina virtual amb accés a terminal (`sudo`) i incorporar els discos virtuals necessaris des del programari de virtualització (VirtualBox o VMware).

![Discos en virtualbox](/Projecte-4/Tasca04/img/crea_disc.png)

![Discos en virtualbox](/Projecte-4/Tasca04/img/discos_totals.png)


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
![check discos](/Projecte-4/Tasca04/img/lsblk.png)

---
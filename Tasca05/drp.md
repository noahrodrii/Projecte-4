# DRP: Pla de còpies

---

## Resum de l'Escenari i Requisits (Anàlisi Inicial)

- **Volum total al servidor:** 420 GB (300 GB Projectes + 20 GB BBDD + 100 GB Carpetes Personals).
- **Equips clients:** 10 PCs amb Windows 11 (amb documents temporals locals).
- **Connexió:** Fibra òptica simètrica de 600 Mbps.
- **RTO (Temps màxim de recuperació):** < 4 hores per a Comptabilitat/Clients.
- **RPO (Pèrdua màxima de dades admesa):** 4 hores (BBDD) / 24 hores (resta de dades).
- **Retenció mínima:** 1 mes (30 dies).

---

## Fase 1: Treball Individual

### 1. Què copiar? (Priorització)

#### Classificació de les dades del servidor

| Dades | Volum | Criticitat | RPO Màxim | Justificació |
| --- | --- | --- | --- | --- |
| **Bases de Dades (Comptabilitat/Clients)** | 20 GB | Alta / Crítica | 4 hores | Són el nucli operatiu de l'empresa. Tenen els requisits més estrictes (RPO i RTO inferiors a 4 hores) i pateixen canvis constants; sense elles, l'empresa quedaria bloquejada a nivell de facturació, comandes i gestió de clients. |
| **Documents de Projectes** | 300 GB | Mitjana-Alta | 24 hores | Són el capital intel·lectual i tècnic de la companyia (plànols i especificacions). Tenen un RPO de 24 hores, però perdre'ls suposaria aturar les instal·lacions i manteniments. |
| **Carpetes Personals dels Usuaris** | 100 GB | Mitjana-Baixa | 24 hores | Tenen menor criticitat immediata (RPO de 24 hores), tot i que contenen feina diària i documentació dels tècnics que cal protegir. |

#### Gestió dels 10 equips clients (Windows 11)

**Cal fer còpia completa dels equips?** No s'ha de fer una imatge completa de tot el sistema operatiu de cada ordinador, ja que seria ineficient, costós en emmagatzematge i redundant (el programari base es pot reinstal·lar ràpidament).

**Solució per als informes temporals de la carpeta `Documents`:** Configurar la redirecció de carpetes cap al servidor Ubuntu (o sincronització automàtica en temps real amb Nextcloud/SMB) perquè la carpeta `Documents` dels 10 clients es bolqui directament a les **Carpetes Personals** del servidor. D'aquesta manera, s'integren dins la política centralitzada de còpies del servidor sense necessitat de gestionar tasques individuals a cada PC.

---

### 2. Periodicitat i Tipus de Còpia

| Dades | Freqüència | Tipus de còpia | Horari | Retenció mínima |
| --- | --- | --- | --- | --- |
| **BBDD Comptabilitat/Clients** | Diària + Cada 4 h | Completa (nit) + Diferencial / *Dump* (dia) | 22:00 h / 10h, 14h, 18h | 30 dies (i mensual 6 mesos) |
| **Projectes + Carpetes Usuaris (inclou `Documents` clients)** | Setmanal | Completa (*Full*) | Dissabte 01:00 h | 4 setmanes (i mensual 6 mesos) |
| **Projectes + Carpetes Usuaris (inclou `Documents` clients)** | Diària (Dl-Dv) | Incremental | 23:00 h | 30 dies |

> Per complir amb un **RPO de 4 hores** a les bases de dades i garantir un **RTO inferior a 4 hores**, l'estratègia s'adapta a la naturalesa de cada dada. Les **Bases de Dades (20 GB)** realitzen una còpia completa diària (22:00 h) i còpies **diferencials mitjançant volcat lògic consistent** cada 4 hores (10h, 14h, 18h); això evita cadenes llargues de dependències, assegura la integritat de les dades en calent i permet una restauració molt més ràpida.

> Els **Projectes i les Carpetes d'Usuaris (400 GB, incloent-hi els documents redirigits dels clients)** fan una còpia completa setmanal (dissabte 01:00 h) per no saturar la xarxa en horari laboral, i incrementals diàries entre setmana (23:00 h). Seguint l'esquema **Avi-Pare-Fill (GFS)**, totes les còpies diàries i setmanals es retenen un mínim de **30 dies (4 setmanes)**, i l'última còpia completa de cada mes es conserva durant **6 mesos** per recuperar arxius esborrats per error a llarg termini.

---

## 3. Mitjans i Ubicació (Regla 3-2-1-1-0)

### Tipus de mitjans escollits

- **Servidor NAS (en RAID 1 o RAID 5) a la xarxa local:** És la millor opció per a una empresa de 10 treballadors. Permet automatitzar les còpies sense intervenció humana i restaurar els 20 GB de bases de dades per xarxa local Gigabit en pocs minuts.
- **Emmagatzematge al Núvol (*Cloud Backup* xifrat):** Com que l'empresa disposa de fibra òptica simètrica de 600 Mbps (~75 MB/s teòrics), té una velocitat de pujada molt alta que permet enviar les còpies incrementals diàries en minuts i la còpia completa de 420 GB en menys de dues hores (aprox. 1 h 35 min).

### Aplicació de la Regla 3-2-1 (amb immutabilitat)

- **3 còpies de les dades:** Les dades originals al Servidor Ubuntu, la còpia local al NAS i la còpia externa al Cloud.
- **2 suports diferents:** Els discs del servidor de fitxers principal i el sistema NAS dedicat (més el núvol com a tercer entorn aïllat).
- **1 còpia fora de les instal·lacions (*Off-site*):** La còpia allotjada al Cloud en un centre de dades extern, que protegeix l'empresa davant d'incendis, robatoris o inundacions a l'oficina.
- **Protecció anti-ransomware (Immutabilitat):** Tant el NAS com el repositori al Núvol s'han de configurar amb **instantànies immutables (WORM / *Object Lock*)** i credencials aïllades (model *pull* des del NAS) perquè, en cas d'infecció per *ransomware* al servidor Ubuntu, el programari maliciós no pugui xifrar ni esborrar les còpies de seguretat.

### On s'ha de guardar físicament la còpia més recent?

La còpia més recent s'ha de mantenir al **NAS local** (ubicat dins un armari *rack* tancat amb clau, connectat a un SAI i físicament separat del servidor principal) per garantir la restauració immediata per xarxa local en menys de 4 hores (**RTO**).

De manera paral·lela, les còpies més recents (especialment els volcats diferencials de cada 4 hores de comptabilitat i clients) s'han de replicar automàticament al **Núvol (*Cloud*)** per assegurar que la versió més actualitzada estigui sempre fora de l'edifici en cas d'un desastre físic greu a les instal·lacions.
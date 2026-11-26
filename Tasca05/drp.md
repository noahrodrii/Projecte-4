# DRP: Pla de còpies de seguretat — Muntatges i Serveis Tècnics SL

## Resum de l'Escenari i Requisits (Anàlisi Inicial)

* **Volum total al servidor:** 420 GB (300 GB Projectes + 20 GB BBDD + 100 GB Carpetes Personals).
* **Equips clients:** 10 PCs amb Windows 11 (amb documents temporals locals).
* **Connexió a Internet:** Fibra òptica simètrica de 600 Mbps.
* **RTO (Temps màxim de recuperació):** < 4 hores per a Comptabilitat/Clients.
* **RPO (Pèrdua màxima de dades admesa):** 4 hores (BBDD) / 24 hores (resta de dades).
* **Retenció mínima:** 1 mes (30 dies).

---

## Fase 1: Treball Individual

### 1. Què copiar? (Priorització)

#### Classificació de les dades del servidor

| Dades | Volum | Criticitat | RPO Màxim | Justificació |
| :--- | :--- | :--- | :--- | :--- |
| **Bases de Dades (Comptabilitat/Clients)** | 20 GB | Alta / Crítica | 4 hores | Són el nucli operatiu de l'empresa. Tenen els requisits més estrictes (RPO i RTO inferiors a 4 hores) i pateixen canvis constants; sense elles, l'empresa quedaria bloquejada a nivell de facturació, comandes i gestió de clients. |
| **Documents de Projectes** | 300 GB | Mitjana-Alta | 24 hores | Són el capital intel·lectual i tècnic de la companyia (plànols i especificacions). Tenen un RPO de 24 hores, però perdre'ls suposaria aturar les instal·lacions i manteniments. |
| **Carpetes Personals dels Usuaris** | 100 GB | Mitjana-Baixa | 24 hores | Tenen menor criticitat immediata (RPO de 24 hores), tot i que contenen feina diària i documentació dels tècnics que cal protegir. |

#### Gestió dels 10 equips clients (Windows 11)

* **Cal fer còpia de seguretat periòdica dels equips?** No. Fer còpies contínues dels 10 sistemes operatius seria ineficient i redundant. En el seu lloc, es crearà **una única imatge del sistema** (amb eines com *Clonezilla* o *FOG*) del sistema operatiu base amb el programari corporatiu instal·lat per poder restaurar qualsevol equip en pocs minuts en cas d'avaria de disc.
* **Solució per als informes temporals de la carpeta *Documents*:** Configurar la redirecció de carpetes cap al servidor Ubuntu (mitjançant SMB o sincronització automàtica) perquè la carpeta *Documents* dels 10 clients es bolqui directament a les Carpetes Personals del servidor. D'aquesta manera, totes les dades crítiques dels tècnics queden centralitzades i protegides dins la política de còpies del servidor.

---

### 2. Periodicitat i Tipus de Còpia (Esquema GFS)

| Dades | Freqüència | Tipus de còpia | Horari | Rotació / Retenció (GFS) |
| :--- | :--- | :--- | :--- | :--- |
| **BBDD Comptabilitat/Clients** | Diària + Cada 4 h | Completa (nit) + Diferencial / Dump (dia) | 22:00 h / 10h, 14h, 18h | **Fill (Diària):** 7 dies / **Pare:** 4 setmanes / **Avi:** 6-12 mesos |
| **Projectes + Carpetes Usuaris (inclou Documents clients)** | Setmanal | Completa (Full) | Diumenge 01:00 h | **Pare (Setmanal):** 4-5 setmanes (cintes S1-S5) / **Avi:** Mensual |
| **Projectes + Carpetes Usuaris (inclou Documents clients)** | Diària (Dl-Ds) | Incremental *(o Diferencial si < 4 GB)* | 23:00 h | **Fill (Diària):** 7 dies (es sobreescriuen setmanalment) |

#### Justificació tècnica de l'estratègia

* **Bases de Dades (20 GB):** Com que el volum és inferior a 50 GB, es realitza una còpia completa diària a les 22:00 h i volcats (*dumps*) diferencials cada 4 hores durant la jornada laboral (10:00 h, 14:00 h i 18:00 h). Això garanteix el compliment estricte del **RPO de 4 hores**, manté la consistència de les dades en calent i permet restaurar en pocs minuts per complir el **RTO inferior a 4 hores**.
* **Projectes i Carpetes d'Usuaris (400 GB):** En superar els 50 GB totals, s'aplica una còpia completa setmanal (diumenge a la matinada per no saturar la xarxa) combinada amb còpies incrementals diàries (de dilluns a dissabte) assumint un volum de canvis diaris superior a 4 GB *(si en la pràctica les modificacions diàries fossin inferiors a 4 GB, s'aplicaria còpia diferencial per agilitzar la restauració)*.
* **Esquema de rotació *Grandfather-Father-Son* (Avi-Pare-Fill):**
  * **Fill (*Son* - Diària):** Còpies diàries de dilluns a dissabte que es van reutilitzant cada setmana.
  * **Pare (*Father* - Setmanal):** Cada diumenge es fa una còpia completa en una cinta dedicada per setmana (S1, S2, S3, S4 i S5), garantint l'historial mínim d'un mes exigit pel cas.
  * **Avi (*Grandfather* - Mensual/Anual):** L'última còpia completa de cada mes s'extreu de l'esquema de rotació setmanal per conservar-se a llarg termini (*archiving*), permetent recuperar fitxers modificats o esborrats accidentalment mesos enrere.

---

### 3. Mitjans i Ubicació (Regla 3-2-1)

#### Tipus de mitjans escollits

1. **Servidor NAS local (en RAID 1 o RAID 5):** Automatitza les còpies ràpides (incloent-hi els volcats cada 4 hores de les BBDD) i permet restaurar per xarxa local Gigabit en qüestió de minuts per garantir el RTO < 4 hores.
2. **Unitat de Cintes magnètiques (LTO):** S'utilitza per implementar l'esquema de rotació GFS (especialment les còpies setmanals *Pare* i mensuals *Avi*). Les cintes ofereixen una gran durabilitat (fins a 50 anys en condicions òptimes), són econòmiques per GB i proporcionen aïllament físic real (*air-gap*) un cop s'extreuen de la unitat.
3. **Emmagatzematge al Núvol (Cloud Backup xifrat):** Aprofitant la fibra òptica simètrica de 600 Mbps (~75 MB/s teòrics), les còpies diàries es repliquen en minuts i una còpia completa de 420 GB es pot transferir en menys de dues hores (~1 h 35 min). El proveïdor estarà ubicat dins la **Unió Europea** per garantir el compliment estricte del **RGPD** en les dades de clients i comptabilitat.

#### Aplicació de la Regla 3-2-1

* **3 còpies de les dades:** Les dades originals en producció al Servidor Ubuntu, la còpia local automatitzada al NAS/Cintes i la còpia externa replicada al Cloud.
* **2 suports físics diferents:** Discs durs magnètics (Servidor i NAS) combinats amb **suport de cinta magnètica (LTO)** per a la rotació GFS i l'arxivament a llarg termini, eliminant el risc de fallada simultània d'una mateixa tecnologia d'emmagatzematge.
* **1 còpia fora de les instal·lacions (*Off-site*):** Doble protecció externa mitjançant el repositori al Cloud xifrat i l'extracció física de la cinta mensual (*Avi*) fora de l'edifici de l'empresa.
* **Seguretat i protecció anti-ransomware:** Totes les còpies es xifren abans de sortir del servidor. A més, l'ús de cintes extraïbles garanteix que el *ransomware* no pugui accedir físicament a les còpies històriques, i els suports descartats al final de la seva vida útil se sotmetran a destrucció segura (desmagnetització o trituració).

#### On s'ha de guardar físicament la còpia més recent?

* **A nivell local (immediatesa):** La còpia més recent (especialment els volcats cada 4 hores de comptabilitat) es manté al NAS i a la cinta diària en curs dins un **armari rack ignífug tancat amb clau**, connectat a un SAI i en una sala separada del servidor principal per assegurar una restauració immediata (< 4 hores).
* **A nivell extern (supervivència davant desastres):** Aquesta mateixa còpia recent es replica automàticament al Cloud i, al final de cada setmana/mes, les cintes corresponents de l'esquema GFS es traslladen físicament a una **ubicació externa segura** (com una caixa forta en una altra seu o un servei de custòdia externa amb control ambiental) per protegir l'empresa davant d'incendis, inundacions o robatoris.
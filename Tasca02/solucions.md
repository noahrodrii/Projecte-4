# Tasca 02 - Historial de versions i recuperació de canvis amb Git

## Descripció de l'activitat

En aquesta pràctica es treballa la gestió de l'historial de versions utilitzant **Git** i **Visual Studio Code** sobre el repositori `projecte4-tasca02`. L'objectiu principal és aprendre a documentar correctament els canvis mitjançant commits estructurats i dominar les diferents tècniques per viatjar en el temps dins d'un repositori i recuperar estats anteriors.

---

## Requeriments previs

Abans de començar l'activitat, s'ha creat el repositori públic `projecte4-tasca02` a GitHub, s'ha clonat en local i s'ha verificat la configuració de la identitat de Git:

```bash
git config user.name "Nom Cognoms"
git config user.email "el_teu_correu@mataro.epiaedu.cat"
```

---

## Part 1: Bones pràctiques en la documentació de commits

### 1. Creació del README inicial i primer commit

S'ha creat el fitxer `README.md` a l'arrel del repositori `projecte4-tasca02` descrivint breument la funció del repositori de proves. Tot seguit, s'ha afegit a l'àrea de preparació i s'ha realitzat el primer commit seguint la convenció `docs:`:

```bash
git add README.md
git commit -m "docs: inicialitza la repositori de prova per tasca 2"
```

![Captura dels dos primers commits](/Projecte-4/Tasca02/img/docs.png)


### 2. Configuració inicial de xarxa i segon commit

S'ha creat el fitxer `00-installer-config.yaml` amb la configuració DHCP per a la interfície `enps03`:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enps03:
      dhcp4: true
```
![commit feat](/Projecte-4/Tasca02/img/config_yaml_1.png)

Com que s'està afegint una nova funcionalitat/configuració al projecte, s'ha utilitzat el prefix `feat:` per al missatge del commit:

```bash
git add 00-installer-config.yaml
git commit -m "feat: afegeix configuracio inicial de xarxa amb dhcp a enps03"
```

![commit feat](/Projecte-4/Tasca02/img/feat.png)

---

## Part 2: L'historial del repositori

### 1. Historial detallat vs. historial resumit

S'han executat les dues variants de `git log` per comparar la informació que ofereixen:

* **`git log`**: Mostra el hash complet (SHA-1) de cada commit, on apunta `HEAD`, l'autor, la data exacta i el missatge complet.
* **`git log --oneline`**: Mostra una versió compacta d'una sola línia per commit amb els primers 7 caràcters del hash i el títol del missatge, ideal per a una lectura ràpida.

![git log](/Projecte-4/Tasca02/img/git%20log_all.png)
![git log --oneline](/Projecte-4/Tasca02/img/git_log-oneline.png)

### 2. Inspecció visual i detallada de `HEAD`

Mitjançant l'eina gràfica de VS Code (*Git Graph*) s'ha comprovat la mateixa línia temporal. Per veure els canvis exactes introduïts en l'últim commit (`HEAD`), s'ha executat:

```bash
git show HEAD
```

Aquesta comanda mostra les metadades del darrer commit juntament amb el *diff* (les línies afegides en verd amb un `+`).

![Inspecció de HEAD a la terminal](/Projecte-4/Tasca02/img/git_showHEAD.png)
![Inspecció de HEAD eina grafica](/Projecte-4/Tasca02/img/feat.png)
---

## Part 3: Recuperar versions anteriors del working directory

### 1. Simulació d'un error humà i comprovació d'estat

S'ha editat `00-installer-config.yaml` canviant erròniament el nom de la interfície a `eth0`:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: true
```

A VS Code el fitxer apareix marcat amb una **`M`** (*Modified*), i executant `git status` es confirma que els canvis estan al *Working Directory* sense preparar:

```bash
git status
```

![Estat del repositori amb el fitxer modificat](/Projecte-4/Tasca02/img/modific.png)

### 2. Descartar els canvis (`git revert`)

Per revertir l'error i tornar el fitxer a l'estat exacte del darrer commit, s'ha utilitzat l'opció *Discard Changes* de VS Code (fent clic dret sobre el fitxer a la pestanya de control de codi font) o la comanda equivalent a la terminal:

```bash
git restore 00-installer-config.yaml
git status
```

Un cop executat, el fitxer ha recuperat la interfície `enps03` i `git status` indica *"nothing to commit, working tree clean"*, confirmant que el *Working Directory*, l'*Staging Area* i el repositori estan sincronitzats.

![Estat net després d'executar git reset](/Projecte-4/Tasca02/img/git_status.png)

---
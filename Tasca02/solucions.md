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
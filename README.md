# Nest Hub Solaire

Tableau de bord **Home Assistant** plein écran pour **Google Nest Hub** : production solaire, consommation, eau chaude, douches restantes, couleurs **EDF Tempo** et **météo en temps réel**, sur une image de fond illustrée animée en HTML/SVG.

![Aperçu du tableau de bord](docs/apercu.png)

*Aperçu avec des valeurs d'exemple.*

## Ce qu'il affiche

| Zone | Valeurs | Animation |
|---|---|---|
| Production | Puissance instantanée, énergie du jour | Reflet de lumière et étincelles sur les panneaux (vitesse selon la production), voile sombre la nuit, voyant de l'onduleur qui pulse, flux d'énergie vers la maison |
| Consommation | Puissance, « 100 % solaire · surplus X W » ou « Réseau : X W » | Arcs lumineux qui tournent autour du logo (vitesse selon la consommation), verts en autoconsommation, orange quand on tire sur le réseau |
| Eau chaude | Litres à 38 °C disponibles, température du ballon | Niveau d'eau qui ondule, couleur selon la température, bulles, halo du thermomètre |
| Douches restantes | Nombre de douches de 40 L | Une goutte qui tombe par douche restante |
| Tempo aujourd'hui / demain | Couleur et tarif | L'icône calendrier prend la couleur du jour (bleu, gris, rouge) avec un halo |
| Pic de production | Valeur et heure du pic du jour | Soleil qui rayonne, éclair qui scintille, onde sur le sommet de la courbe |
| Météo | Temps actuel et température | Icône animée : soleil, lune et étoiles, nuages qui dérivent, pluie, averse, neige, orage, brouillard, vent |
| Jours Tempo restants | Bleu / Blanc / Rouge | — |

L'affichage reste **permanent** sur le Nest Hub : une automatisation le relance dès que l'écran revient au repos, sans jamais interrompre une vidéo ou une autre application.

## Prérequis

- Home Assistant 2024.8 ou plus récent.
- [HACS](https://hacs.xyz/) avec :
  - [button-card](https://github.com/custom-cards/button-card) (calques animés) ;
  - [card-mod](https://github.com/thomasloven/lovelace-card-mod) (suppression des bords de la carte).
- Un **Google Nest Hub** (ou tout écran Chromecast) ajouté à Home Assistant via l'intégration **Google Cast**.
- Home Assistant accessible en **HTTPS avec un certificat valide** : le Nest Hub refuse d'afficher une page en HTTP. Au choix :
  - Home Assistant Cloud (Nabu Casa) ;
  - l'add-on **Duck DNS** avec Let's Encrypt (gratuit), puis renseigner l'URL `https://…` dans *Paramètres → Système → Réseau* (URL interne **et** externe).
- Pour Tempo : l'intégration [RTE Tempo](https://github.com/hekmon/rtetempo) (HACS).
- Une entité météo (par exemple `weather.forecast_home`, créée par défaut par Met.no).

## Installation

### 1. Image de fond

Copiez [`www/nesthub_fond.png`](www/nesthub_fond.png) dans le dossier `config/www/` de Home Assistant (créez-le s'il n'existe pas). Elle sera servie à l'adresse `/local/nesthub_fond.png`.

> Si le dossier `www` vient d'être créé, redémarrez Home Assistant une fois.

### 2. Capteurs calculés

1. Activez les *packages* dans `configuration.yaml` (s'ils ne le sont pas déjà) :

   ```yaml
   homeassistant:
     packages: !include_dir_named packages
   ```

2. Copiez [`packages/nesthub_solaire.yaml`](packages/nesthub_solaire.yaml) dans `config/packages/`.
3. Dans ce fichier, remplacez les entités sources par les vôtres (voir le tableau ci-dessous) et, si besoin, la capacité du ballon et la température de l'eau froide.
4. *Outils de développement → YAML → Vérifier la configuration*, puis redémarrez.

Ce package crée :

| Entité | Rôle |
|---|---|
| `sensor.eau_chaude_disponible` | Litres d'eau à 38 °C disponibles (mélangée avec l'eau froide) |
| `sensor.douches_restantes` | Douches de 40 L |
| `sensor.tempo_texte_du_jour`, `sensor.tempo_texte_demain` | « Bleu · tarif réduit », etc. |
| `sensor.consommation_sous_titre` | Surplus solaire ou puissance tirée du réseau |
| `sensor.pic_de_production_du_jour` | Maximum du jour (attribut `heure`), remis à zéro à minuit |

### 3. Thème

Copiez [`themes/nesthub.yaml`](themes/nesthub.yaml) dans `config/themes/` et vérifiez que `configuration.yaml` contient :

```yaml
frontend:
  themes: !include_dir_merge_named themes
```

Puis *Outils de développement → Actions → `frontend.reload_themes`*.

### 4. Vue « Solaire »

1. Ouvrez votre tableau de bord, puis **⋮ → Modifier le tableau de bord → ⋮ → Éditeur de configuration**.
2. Dans la liste `views:`, collez le contenu de [`dashboard/vue-solaire.yaml`](dashboard/vue-solaire.yaml) comme une nouvelle vue (en ajoutant `- ` devant `title:` et en indentant le reste).
3. Remplacez les entités par les vôtres (voir ci-dessous), puis enregistrez.

La vue est de type `panel` : la carte remplit tout l'écran 1024 × 600 du Nest Hub.

### 5. Affichage permanent sur le Nest Hub

1. Créez une automatisation, passez en mode YAML et collez [`automations/nesthub_affichage_permanent.yaml`](automations/nesthub_affichage_permanent.yaml).
2. Remplacez `media_player.nest_hub` par l'entité de votre écran.
3. Si votre tableau de bord n'est pas celui par défaut, changez `dashboard_path` (par exemple `dashboard-maison`).

Pour tester tout de suite : *Outils de développement → Actions → `cast.show_lovelace_view`* avec votre écran, `dashboard_path: lovelace` et `view_path: solaire`.

## Adapter les capteurs

Remplacez ces noms génériques par vos propres entités, dans le package **et** dans la vue (un *Rechercher / Remplacer* suffit) :

| Nom générique | À remplacer par |
|---|---|
| `sensor.solar_power` | Puissance produite par les panneaux (W) |
| `sensor.solar_energy_today` | Énergie produite aujourd'hui (kWh) |
| `sensor.house_power` | Puissance consommée par la maison (W) |
| `sensor.grid_import_power` | Puissance tirée du réseau (W, 0 quand vous n'importez pas) |
| `sensor.water_heater_temperature` | Température du ballon d'eau chaude (°C) |
| `weather.forecast_home` | Votre entité météo |
| `media_player.nest_hub` | Votre Nest Hub (automatisation uniquement) |

Les entités `sensor.rte_tempo_*` sont celles créées par l'intégration RTE Tempo ; vous n'avez normalement rien à changer.

Les valeurs en kW sont converties automatiquement en W dans les animations.

## Personnaliser

- **Positions** : chaque élément a un `style` avec `left` / `top` en pourcentage de l'image. Les calques animés sont centrés sur leur point.
- **Animations** : chaque calque `custom:button-card` contient un champ `custom_fields.a` en JavaScript qui renvoie du SVG selon l'état des capteurs, et un `extra_styles` avec les `@keyframes` CSS. Les vitesses, couleurs et seuils se règlent directement dans ce code.
- **Couleurs de la météo** : les dégradés `gn`, `gp`, `go` (nuages) et `gs` (soleil) sont définis en tête du SVG du calque `anime-meteo`.

## Bon à savoir

- Le Nest Hub n'affiche pas les éléments `conditional` d'une carte picture-elements en mode Cast : c'est pour cela que les textes Tempo et le sous-titre de consommation passent par des capteurs template.
- Le Nest Hub ne transmet pas le toucher à l'affichage Home Assistant : la vue est faite pour être regardée, pas pour être utilisée au doigt.
- Le Nest Hub coupe l'affichage Cast au bout d'une dizaine de minutes ; l'automatisation le relance dans la minute suivante.
- Pour accéder à Home Assistant depuis l'extérieur sans ouvrir de port, l'add-on **Tailscale** fonctionne très bien en complément.

## Licence

[MIT](LICENSE)

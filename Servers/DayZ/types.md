## Guide complet de configuration types.xml (DayZ)

Ce document explique en détail le fichier `types.xml` de DayZ, c’est-à-dire la configuration principale de l’économie de loot (Central Economy). Il est librement inspiré et paraphrasé de la documentation publique de DZconfig, avec attribution ci‑dessous.

- Référence utile: https://dzconfig.com/wiki/types


### À quoi sert `types.xml` ?

Le fichier `types.xml` détermine:
- Quels objets peuvent apparaître sur le serveur
- Combien d’exemplaires (objectif/mini) existent simultanément
- Où ils peuvent apparaître (catégories, usages/contexte, tags)
- Leur persistance (lifetime) et la cadence de réapparition (restock)
- La répartition régionale ou par “tiers” (valeurs `Tier1` → `Tier4`)


### Où placer `types.xml` ?

Selon la mission, typiquement:
- `mpmissions/dayzOffline.chernarusplus/db/types.xml` (Chernarus)
- `mpmissions/dayzOffline.enoch/db/types.xml` (Livonia)
- Pour une carte modée, utilisez le dossier `mpmissions/<mission>/db/`

Remarque: ce dépôt contient des scripts d’installation/gestion (`install_dayz_server.py`, `serversManager.py`) et un serveur packagé sous `lasted_server/`. Adaptez le chemin de `types.xml` à votre mission active.


## Structure d’un type

Chaque entrée est un bloc `<type>` pour une classe d’objet DayZ (nom de classe côté jeu, p.ex. `PeachesCan`, `M4A1`, `Ammo_556x45`):

```
<type name="PeachesCan">
	<nominal>15</nominal>
	<lifetime>14400</lifetime>
	<restock>0</restock>
	<min>12</min>
	<quantmin>-1</quantmin>
	<quantmax>-1</quantmax>
	<cost>100</cost>
	<flags count_in_cargo="0" count_in_hoarder="0" count_in_map="1" count_in_player="0" crafted="0" deloot="0"/>
	<category name="food"/>
	<tag name="shelves"/>
	<usage name="Town"/>
	<usage name="Village"/>
	<value name="Tier1"/>
	<value name="Tier2"/>
</type>
```


## Paramètres essentiels

- nominal (entier): volume “cible” pour cet objet. Si la quantité réelle passe en dessous, l’objet redevient éligible au spawn.
- min (entier): plancher garanti. Le CE tente de maintenir au moins ce nombre.
- lifetime (secondes): durée de vie d’un objet posé dans le monde avant nettoyage s’il n’est pas manipulé/interagi.
- restock (secondes): délai minimum après retrait avant qu’un nouvel exemplaire puisse être rééligible.
- cost (entier): poids relatif de probabilité au moment du choix de spawn. Plus haut = plus de chances, à rareté égale.


## Paramètres de quantité (stackables)

- quantmin (entier): quantité minimale à la création (munitions dans un chargeur, eau dans une gourde, etc.). `-1` pour objets sans quantité.
- quantmax (entier): quantité maximale initiale. `-1` si non applicable.

Règles rapides:
- Utilisez `-1/-1` pour les objets non empilables.
- `quantmin` doit être ≤ `quantmax` (sinon aucun spawn ou erreurs CE).


## Drapeaux (flags)

Les drapeaux définissent où l’on “compte” les objets pour les seuils `nominal`/`min` et quelques comportements spéciaux:

- count_in_cargo (0/1): compter les objets dans le cargo (sacs, coffres, véhicules).
- count_in_hoarder (0/1): compter les objets dans les conteneurs de stockage/planques (tentes, barils, stash).
- count_in_map (0/1): compter ceux posés dans le monde/bâtiments.
- count_in_player (0/1): compter ceux dans l’inventaire des joueurs.
- crafted (0/1): si 1, l’objet est prévu pour être fabriqué par les joueurs (non spawné par l’économie).
- deloot (0/1): loot dynamique d’événements (crash hélico, zones contaminées, etc.).

Bonnes valeurs par défaut (indicatives):
- Loot de carte “classique”: `count_in_map=1`, le reste `0`, pour éviter que l’accaparement en bases/joueurs bloque les respawns.
- Pour limiter le dupe via hoarding: mettez `count_in_hoarder=1` sur les objets rares.


## Catégorie, usage et valeur (tiers)

- category name (string): groupe logique (exemples courants: `weapons`, `food`, `tools`, `clothes`, `ammo`, `magazine`, `medical`, `containers`, `misc`). Sert au filtrage/organisation.
- usage name (string): contexte/zones d’apparition. Exemples: `Village`, `Town`, `City`, `Industrial`, `Military`, `Police`, `Firefighter`, `Hunting`, `Farm`, `School`, `Coast`. Jusqu’à 4 par type.
- value name (string): groupes de valeur/régions (“tiers”), p.ex. `Tier1` (bord de carte), `Tier2`, `Tier3`, `Tier4` (militaire haut niveau). Utilisé pour l’équilibrage régional.

Note: `tag name` (p.ex. `shelves`, `workbench`) cible certains emplacements/proxies d’intérieur; utile pour du loot “sur étagère”.


## Exemples pratiques

### Arme militaire (rare, hoarding comptabilisé)
```
<type name="M4A1">
	<nominal>8</nominal>
	<min>3</min>
	<lifetime>14400</lifetime>
	<restock>1800</restock>
	<cost>50</cost>
	<flags count_in_cargo="0" count_in_hoarder="1" count_in_map="1" count_in_player="0" crafted="0" deloot="0"/>
	<category name="weapons"/>
	<usage name="Military"/>
	<value name="Tier3"/>
	<value name="Tier4"/>
</type>
```

### Nourriture (abondante, sans quantité)
```
<type name="TacticalBaconCan">
	<nominal>40</nominal>
	<min>25</min>
	<lifetime>7200</lifetime>
	<restock>1800</restock>
	<quantmin>-1</quantmin>
	<quantmax>-1</quantmax>
	<cost>100</cost>
	<flags count_in_cargo="0" count_in_hoarder="0" count_in_map="1" count_in_player="0" crafted="0" deloot="0"/>
	<category name="food"/>
	<usage name="Town"/>
	<usage name="Village"/>
	<value name="Tier1"/>
	<value name="Tier2"/>
</type>
```

### Munitions (stackables)
```
<type name="Ammo_556x45">
	<nominal>120</nominal>
	<min>60</min>
	<lifetime>10800</lifetime>
	<restock>1200</restock>
	<quantmin>10</quantmin>
	<quantmax>30</quantmax>
	<cost>80</cost>
	<flags count_in_cargo="0" count_in_hoarder="0" count_in_map="1" count_in_player="0" crafted="0" deloot="0"/>
	<category name="ammo"/>
	<usage name="Military"/>
	<value name="Tier2"/>
	<value name="Tier3"/>
	<value name="Tier4"/>
</type>
```


## Bonnes pratiques (Do/Don’t)

À faire:
- Ajuster `nominal` selon la rareté voulue et le nombre moyen de joueurs.
- Calibrer `lifetime` en fonction de la catégorie (consommables plus court, armes plus long).
- Utiliser `usage`, `category`, `value` de manière cohérente pour maîtriser la distribution.
- Valider l’XML après chaque série de modifications et conserver des sauvegardes versionnées.

À éviter:
- Des `nominal`/`min` extravagants (risque de surcharge ou “loot explosion”).
- Mélanger des usages sans lien (ex. `Military` et `School` pour une arme lourde).
- Oublier de compter le hoarding (`count_in_hoarder=0`) pour les objets rares que vous souhaitez réellement limiter.


## Méthode de tuning rapide

1) Rareté cible: définissez des fourchettes par catégorie (ex: rifle T3‑T4: `nominal 5–12`, `min 2–4`).
2) Joueurs et carte: augmentez `nominal` de ~10–20% par tranche de +10 joueurs simultanés; réduisez si petite carte.
3) Lifetime: objets “consommés” (nourriture, bandages) 2–6 h; armes/outils 6–16 h.
4) Restock: 15–45 min pour consommables; 30–60+ min pour armes/magasins.
5) Flags: activez `count_in_hoarder` pour les objets que vous refusez de voir “retirés” de l’économie par stockage.


## Pièges fréquents

- `min` > `nominal`: force le CE à sur‑spawner pour atteindre le plancher; incohérent sur la durée.
- `quantmin` > `quantmax`: aucun spawn ou erreurs CE.
- Usages/catégories/valeurs inconnus: l’objet ne spawnera tout simplement pas.
- `count_in_player=0` sur objets rares: farming possible par transfert vers inventaires sans impacter la cible `nominal`.
- Mélange tiers/cartes: des `value TierX` non utilisés par votre carte/mission ne produisent aucun effet.


## Validation et tests

- Valider la syntaxe XML (ex. sous Linux avec `xmllint`) et surveiller les logs du serveur DayZ pour les erreurs CE.
- Tester sur instance de staging avant la prod (risque de loot vide ou saturé).
- Outils: DZconfig (édition/validation), DayZ Tools.

Commandes optionnelles (Linux):

```bash
# Vérifier la syntaxe XML
xmllint --noout mpmissions/<mission>/db/types.xml

# Sauvegarder puis comparer vos changements
cp mpmissions/<mission>/db/types.xml backups/types_$(date +%F).xml
diff -u backups/types_*.xml | less
```


## Intégration dans ce dépôt

- Déployez/éditez `types.xml` dans le dossier `mpmissions/<votre_mission>/db/` utilisé par votre instance.


## Aide‑mémoire (cheatsheet)

- nominal: objectif de population
- min: plancher garanti
- lifetime: durée avant nettoyage (s)
- restock: délai mini avant rééligibilité (s)
- quantmin/quantmax: quantité initiale min/max (stackables)
- cost: poids relatif de spawn
- flags: compte en cargo/hoarder/map/player, crafted, deloot
- category: groupe logique (weapons, food, …)
- usage: contexte (Military, Town, …)
- value: tiers régionaux (Tier1 → Tier4)
- tag: cibles d’emplacements (shelves, …)


## Attribution et ressources

- Documentation paraphrasée depuis DZconfig: https://dzconfig.com/wiki/types
- Index CE: https://dzconfig.com/wiki
- Économie: https://dzconfig.com/wiki/economy
- Événements: https://dzconfig.com/wiki/events


---

Dernière mise à jour: 2025‑10‑27.

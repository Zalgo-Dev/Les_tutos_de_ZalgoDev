## Variables dans les scripts Batch (Windows 10) — Tutoriel

Ce document explique les variables dans les scripts batch Windows (fichiers .bat/.cmd). Il couvre : déclaration, lecture, portée, expansion retardée (delayed expansion), variables d'environnement persistantes, arguments de ligne de commande et exemples pratiques.

### 1) Notions de base

- Définir une variable :

  SET NOM=Valeur

  Exemple :

  SET GREETING=Bonjour

- Lire une variable : entourez le nom avec des pourcentages `%NOM%` :

  ECHO %GREETING%

- Attention aux espaces : `SET X=1` crée `X` avec valeur `1`. `SET X =1` crée une variable `X ` (avec un espace dans le nom). Évitez les espaces autour du `=`.

### 2) Variables temporaires et `setlocal`

- `setlocal` limite les modifications à l'environnement au script courant (et `endlocal` restaure). Utile pour éviter de polluer l'environnement global.

  Exemple :

  @ECHO off
  setlocal
  SET TEMPVAR=abc
  ECHO TEMPVAR=%TEMPVAR%
  endlocal
  REM ici %TEMPVAR% n'existe plus

### 3) Variables persistantes : `setx`

- `setx` écrit une variable d'environnement persistante (user ou machine) dans le registre. Elle est disponible dans les futures sessions mais pas automatiquement dans la session actuelle.

  Ex : setx MONVAR "ma valeur"

  Pour la voir dans la session en cours, relancez l'invite de commande ou récupérez-la depuis le registre.

### 4) Arguments du script

- `%0` : le nom du script
- `%1`, `%2`, ... : premier, deuxième argument, etc.
- `%*` : tous les arguments

  Exemple : script `test.bat` lancé `test.bat Alice 25` : `%1` vaut `Alice`, `%2` vaut `25`.

### 5) Expansion retardée (Delayed Expansion)

- Problème : l'expansion `%VAR%` est faite quand la ligne est lue. Dans les blocs `IF (...) DO (...)` ou `FOR (...) DO (...)`, cela peut conduire à des valeurs obsolètes.

- Solution : activer l'expansion retardée et utiliser `!VAR!` pour obtenir la valeur à l'exécution.

  Exemple :

  @ECHO off
  setlocal enabledelayedexpansion
  SET COUNT=0
  FOR /L %%i IN (1,1,3) DO (
    SET /A COUNT+=1
    ECHO boucle %%i -- COUNT=!COUNT!
  )
  endlocal

### 6) Opérations arithmétiques

- `SET /A` permet les calculs entiers :

  SET /A X=5+3
  ECHO %X%

  `SET /A` accepte les variables sans `%` : `SET /A X=X+1`.

### 7) Tests et conditions avec variables

- Comparaison de chaînes (case-insensitive avec /I) :

  IF /I "%VAR%"=="oui" ECHO OK

- Tester l'existence d'une variable :

  IF DEFINED VAR ( ECHO VAR existe ) ELSE ( ECHO VAR manquante )

### 8) Bonnes pratiques pour les accents et l'encodage

- Windows `cmd.exe` utilise par défaut une page de code (code page) différente selon la locale. Pour afficher correctement les accents, deux approches courantes :

  1) Sauvegarder le fichier en ANSI (CP1252) et utiliser la page de code 1252 :

     chcp 1252 >nul

  2) Sauvegarder le fichier en UTF-8 sans BOM et utiliser la page de code UTF-8 (65001) :

     chcp 65001 >nul

- Note : `chcp 65002` est invalide (erreur fréquente). Utilisez `65001` pour UTF-8.
- PowerShell gère mieux UTF-8; dans `cmd.exe` l'affichage peut encore poser problème selon la police et la configuration. Si vous rencontrez des caractères bizarres, essayez la police `Consolas` ou `Lucida Console` dans la fenêtre de l'invite.

### 9) Exemples complets

- Exemple 1 : script interactif simple

  @ECHO off
  chcp 65001 >nul 2>nul || chcp 1252 >nul
  setlocal enabledelayedexpansion

  ECHO Quel est votre prénom ?
  SET /p NAME=
  ECHO Bonjour %NAME% !

  endlocal

- Exemple 2 : utilisation d'arguments et enregistrement

  @ECHO off
  chcp 65001 >nul 2>nul || chcp 1252 >nul
  IF "%1"=="" (
    ECHO Usage: %0 Nom
    GOTO :EOF
  )
  SET NOM=%1
  ECHO Nom = %NOM%
  ECHO %NOM% >> sortie.txt

### 10) Exercices rapides

1) Écrire un script `age.bat` qui prend en argument l'année de naissance et affiche l'âge.

2) Écrire un script qui demande le prénom et le nom, puis crée un fichier `prenom_nom.txt` contenant les infos.

Solutions (brèves) :

1) age.bat

  @ECHO off
  setlocal
  IF "%1"=="" (
    ECHO Usage: %0 <annee_naissance>
    GOTO :EOF
  )
  REM Extraction de l'année depuis %DATE% peut dépendre du format local
  FOR /F "tokens=4 delims=/.- " %%a IN ("%DATE%") DO SET ANNEE=%%a
  SET /A AGE=ANNEE - %1
  ECHO Vous avez %AGE% ans.

2) prenom_nom.bat

  @ECHO off
  chcp 65001 >nul 2>nul || chcp 1252 >nul
  SET /p PRENOM=Prénom: 
  SET /p NOM=Nom: 
  ECHO %PRENOM% %NOM% > %PRENOM%_%NOM%.txt

### 11) Commandes de test et recommandations

Dans Windows PowerShell (si vous utilisez PowerShell pour lancer le .bat) :

  cmd /c .\votre_script.bat

Pour exécuter depuis l'invite de commandes `cmd.exe` :

  .\votre_script.bat

Si vous avez des problèmes d'accents :

- Ouvrez le fichier dans un éditeur (Notepad++) et enregistrez en "UTF-8 sans BOM" ou en "ANSI" selon l'approche choisie.
- Ajoutez en tête du script :

  chcp 65001 >nul 2>nul || chcp 1252 >nul

  Cela force la console à utiliser UTF-8 si possible, sinon CP1252.

### 12) Ressources et lecture complémentaire

- `help set` et `help setx` dans `cmd.exe`.
- Documentation Microsoft sur les variables d'environnement et les pages de code.

# RL Settings — profils de réglages pour Rocket League

> Enregistre tes réglages Rocket League (caméra, commandes, interface, audio, vidéo, touches) dans des profils, puis laisse une macro les appliquer dans le jeu à ta place, et vérifier qu'ils sont bien pris en compte.

Application Windows (Python / Tkinter, aussi fournie en `.exe`). Elle ne touche ni à la mémoire du jeu ni à ses fichiers de configuration : elle pilote le menu **PARAMÈTRES** comme le ferait un joueur (clics, flèches, touches), et **lit à l'écran** ce que le jeu affiche pour confirmer chaque valeur.

## Ce que fait l'application

### 1. Éditeur de profils
- Profils personnalisés illimités : **Nouveau**, **Dupliquer**, **Enregistrer**, **Enregistrer sous…**, **Importer / Exporter** (JSON).
- Un profil de départ « Valeurs des captures » (en lecture seule) sert de modèle.
- Couvre **les menus Caméra, Commandes, Interface, Audio, Vidéo** (≈ 75 réglages) et **Touches** (32 actions, affectations clavier/souris **et** manette).
- Pour chaque réglage, une case **Inclure** : seuls les réglages cochés sont traités. Boutons *Tout inclure*, *Aucun*, et dans l'onglet Touches *Clavier/souris seulement* / *Manette seulement*.
- Validation des valeurs (nombres, listes, noms de touches français ou anglais, codes `scXXX`), `F8` réservée à l'arrêt.
- Chaque compte Windows a ses propres profils (`%LOCALAPPDATA%\RLSettings`).

### 2. Application automatique dans le jeu
- **Appliquer la sélection** : la macro met Rocket League au premier plan, ouvre chaque menu et règle les valeurs.
- **Vérifier seulement** : lit les valeurs actuelles et indique ce qui diffère, sans rien modifier.
- **Curseurs** : la macro mesure la plage et le pas réels du jeu avec les flèches du clavier, vise la valeur demandée et la relit. Les plages mesurées sont mémorisées pour les passages suivants. Les valeurs négatives (ex. angle de caméra) sont gérées.
- **Cases à cocher** : état lu sur les pixels de la case.
- **Listes déroulantes** : option choisie par lecture OCR du libellé exact (ou du libellé étendu du jeu, ex. « 165 FPS (Recommended) »), y compris pour les longues listes.
- **Échelles d'interface** : traitées en dernier, avec lecture qui suit le redimensionnement du menu.
- **Vidéo** : « Détails du rendu » est traité comme un résumé déduit des réglages avancés ; CONFIRMER n'est cliqué que si la résolution ou le mode d'affichage change.
- **Options absentes** (ex. 240 FPS sur un écran 165 Hz) : signalées *indisponible* sans interrompre le reste du passage.

### 3. Touches : clavier, souris et manette
- **Clavier / souris** : touches envoyées à la position physique QWERTY (même sous Windows AZERTY), boutons de souris, aliases français.
- **Manette** : la macro branche une **manette Xbox 360 virtuelle** (pilote [ViGEmBus](https://github.com/nefarius/ViGEmBus), client écrit en `ctypes`, aucune bibliothèque tierce), la fait « rejoindre » le jeu avec Start, puis appuie sur le bouton demandé quand le jeu attend une entrée. **Aucune action requise de ta part, même avec ta vraie manette branchée.**
- Elle n'appuie **que** lorsque l'invite *« Appuyez sur n'importe quelle touche »* est visible (hors invite, RB/LB/B navigueraient dans les menus).
- **Retirer une touche** (« Non affecté ») : clic sur la croix du jeu pour clavier/souris ; « Touche par défaut » pour une touche manette.
- Les icônes de boutons apprises sont mémorisées par action pour ne pas refaire inutilement une affectation déjà bonne.
- Les axes de souris (MouseX / MouseY) restent guidés.

### 4. Sécurité et traçabilité
- **F8** arrête immédiatement ; perdre le premier plan ou changer de fenêtre arrête aussi la macro.
- Avant d'appliquer, copie de sauvegarde des dossiers locaux `Config` et `SaveData` de Rocket League (jamais restaurés automatiquement).
- Chaque passage écrit un **rapport** (`rapport.json`) et des **captures avant / après** dans `%LOCALAPPDATA%\RLSettings\runs\`.
- Fenêtre cible vérifiée (une seule fenêtre `RocketLeague.exe`, 16:9, 2560 × 1440).

## Comment ça marche
| Étape | Technique |
|---|---|
| Lire l'écran | capture de la fenêtre du jeu, image de référence 2560 × 1440 |
| Lire les valeurs | **OCR Windows local** (WinRT via PowerShell, langues installées), plusieurs lectures concordantes exigées |
| Agir | `SendInput` Win32 (souris, clavier, molette), délais adaptés à la latence du menu |
| Manette | ViGEmBus (X360 virtuelle), lecture XInput pour les tests |
| Comparer | masques d'encre / gabarits (NumPy, Pillow) pour icônes et cases |

## Prérequis
- Windows 10 / 11.
- Rocket League en **français**, **2560 × 1440**, mode sans bords, **échelle d'interface à 100 %** au début du passage.
- Pour l'automatisation des touches manette : pilote **ViGEmBus** installé (sinon retour au mode manuel : tu maintiens le bouton demandé).
- Pour lancer depuis les sources : Python 3.10+ avec Tkinter, Pillow et NumPy (`pip install -r requirements.txt`).

## Utilisation
1. Lance `RL Settings.exe` (ou double-clique sur `Demarrer.cmd`).
2. Crée ou choisis un profil, règle les valeurs, coche les réglages à traiter.
3. Ouvre **PARAMÈTRES** dans Rocket League.
4. Clique sur **Vérifier seulement** ou **Appliquer la sélection**. Un délai de 2 secondes laisse la macro prendre la main ; **F8** pour arrêter.

Un passage complet de plus de 100 réglages dure environ une minute quand le jeu est déjà presque à jour, et quelques minutes pour tout changer (la première mesure des curseurs est la plus longue).

## Construire l'exécutable
```bat
Construire l'exe.cmd
```
(PyInstaller, Python 3.11 + Pillow + NumPy.) Le résultat est `dist\RL Settings.exe`, sans dépendance à installer, sauf ViGEmBus pour la manette.

## Tests
```bat
python -m unittest discover -q
```
Plus de 160 tests hors jeu : profils, import/export, lecture de nombres (dont signe), cases, listes, curseurs, échelles, touches, manette virtuelle simulée, arrêt F8. Ils travaillent sur des images enregistrées et des simulations, jamais sur le jeu réel.

## Organisation du dépôt
| Fichier | Rôle |
|---|---|
| `app.py` | éditeur de profils et lancement |
| `macro_engine.py` / `custom_engine.py` | navigation dans les menus, application et rapports |
| `profile_store.py`, `settings_catalog.json`, `profile.json` | profils, catalogue des réglages, repères d'écran |
| `ocr_windows.py`, `ocr_worker.ps1` | OCR local Windows |
| `scale_controls.py`, `slider_geometry.py` | curseurs et échelles d'interface |
| `native_windows.py` | entrées Win32, arrêt F8, sauvegardes |
| `virtual_pad.py` | manette Xbox 360 virtuelle (ViGEmBus) |
| `vision.py` | comparaison d'images |
| `references/` | captures de référence du menu |

## Limites
- Présentation de départ supportée : français, 2560 × 1440, sans bords. Les autres résolutions ne sont pas annoncées compatibles.
- Les onglets Jeu, Adapté aux streams, Score, Chat et Extras ne sont pas couverts.
- Le menu du jeu réagit avec un léger retard : les délais sont réglés pour cette latence, une machine très chargée peut demander une relance.
- Les libellés non vus à l'écran (autres langues ou nouvelles options) peuvent demander de nouveaux repères.

## Avertissement
Projet indépendant, **non affilié à Psyonix / Epic Games**. L'application simule des entrées comme un joueur et ne modifie ni la mémoire ni les fichiers du jeu ; l'utilisation reste sous ta responsabilité, notamment au regard des conditions d'utilisation du jeu.

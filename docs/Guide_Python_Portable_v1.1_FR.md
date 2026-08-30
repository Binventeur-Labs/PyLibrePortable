# Guide : Créer une application Python portable sous Windows (sans installation)

## Version 1.1

## Sommaire

[1. Introduction : philosophie du Python portable](#section-1)  
[2. Pourquoi utiliser le Windows Embeddable Package](#section-2)  
[3. Installation minimale de Python portable](#section-3)  
[4. Activation du système de modules externes](#section-4)  
[5. Installation de Pip](#section-5)  
[6. Installation des bibliothèques Python externes](#section-6)  
[7. Ajout manuel de Tkinter](#section-7)  
[8. Utilisation de Visual Studio Code avec le Python portable](#section-8)  
[9. Scripts Batch de lancement et de maintenance](#section-9)  
[10. Création d’un lanceur `.exe` natif Windows (avec support de l'image de démarrage)](#section-10)  
[11. Structure finale du projet](#section-11)  
[12. Résultat et philosophie générale](#section-12)  

---

<div id="section-1"></div>

## 1. Introduction : philosophie du Python portable

### 1.1 Objectif

Ce guide explique comment créer une **application Python entièrement portable sous Windows**, c’est‑à‑dire :

- sans installation de Python sur la machine ;
- sans modification du système (registre, variables d’environnement, etc.) ;
- pouvant être copiée sur une clé USB, un disque externe ou dans une archive ZIP ;
- avec tous les fichiers de l’application facilement accessibles et modifiables.

L’idée est d’obtenir une application :

- **souple** (modifications directes des fichiers) ;
- **simple à maintenir** (pas de recompilation) ;
- **légère** (seulement les composants nécessaires) ;
- **proche d’une vraie application Windows** (icône, scripts de lancement, image de démarrage, etc.).

### 1.2 Application compilée vs application Python portable

Une application compilée fonctionne généralement ainsi :

```text
Code source
      ↓
Compilation
      ↓
Exécutable (.exe)
      ↓
Distribution
```

Chaque modification importante implique souvent :

```text
Modification
      ↓
Nouvelle compilation
      ↓
Nouvelle version
```

Une application Python portable fonctionne différemment :

```text
Fichiers Python
      ↓
Modification directe
      ↓
Lancement immédiat
```

Les fichiers restent accessibles :

- `.py` (code source) ;
- fichiers de configuration ;
- images et ressources ;
- modèles ;
- bases de données locales, etc.

On modifie, on relance — sans recompilation.

### 1.3 Philosophie générale

Le but n’est pas de créer une usine à gaz avec tous les outils possibles, mais :

> **Ajouter uniquement les composants nécessaires au projet.**

Exemples :

- Un petit script en ligne de commande peut fonctionner avec :
  ```text
  Python portable
  ```
- Une application graphique complète peut nécessiter :
  ```text
  Python portable
  + Activation des modules externes
  + Pip
  + Bibliothèques externes
  + Tkinter
  + VS Code (pour le développement)
  + Scripts de lancement
  + Icône Windows
  ```

### 1.4 Ce qui est obligatoire et ce qui est optionnel

La seule étape **obligatoire** est :

```text
Installer Python portable
```

Tout le reste est **optionnel**, selon les besoins du projet.

| Élément                                   | Obligatoire | Prérequis                        | Rôle                                                 |
| ----------------------------------------- | ----------- | -------------------------------- | ---------------------------------------------------- |
| Python portable                           | Oui         | Aucun                            | Exécuter du code Python                              |
| Activation du système de modules externes | Non         | Python portable                  | Autoriser les packages externes                      |
| Pip                                       | Non         | Modules externes activés         | Installer des bibliothèques                          |
| Bibliothèques externes                    | Non         | Modules externes activés (+ Pip) | Ajouter des fonctionnalités                          |
| Tkinter                                   | Non         | Modules externes activés         | Interface graphique native                           |
| VS Code                                   | Non         | Python portable                  | Développement confortable                            |
| Fichiers `.bat`                           | Non         | Aucun                            | Simplifier l’utilisation                             |
| Icône Windows                             | Non         | Application graphique terminée   | Apparence professionnelle                            |
| Image de démarrage (Splash Screen)        | Non         | Lanceur `.exe` natif             | Améliorer l'expérience utilisateur au lancement      |

---

<div id="section-2"></div>

## 2. Pourquoi utiliser le Windows Embeddable Package

### 2.1 Les différentes formes de Python

Python existe sous plusieurs formes :

- installateur classique ;
- Microsoft Store ;
- environnements virtuels ;
- version portable (Windows Embeddable Package).

Pour ce projet, on utilise :

> **Windows Embeddable Package (64‑bit)**

Cette version est conçue pour être intégrée dans une application et fonctionne simplement par **extraction d’une archive ZIP**.

### 2.2 Avantages

- **Aucun installateur** : on copie un dossier, c’est tout.
- **Aucun impact sur Windows** :
  - pas de modification du registre ;
  - pas de variables d’environnement ;
  - pas de conflit avec une installation Python existante.
- **Plusieurs versions peuvent cohabiter** :

  ```text
  Projet_A\
  └── python_env\ (Python 3.11)

  Projet_B\
  └── python_env\ (Python 3.12)
  ```

- **Parfait pour la distribution** : l’utilisateur reçoit un dossier contenant :
  ```text
  Application complète
  + Python intégré
  ```
  Il n’a rien à installer.

### 2.3 Limites

La version Embeddable est volontairement minimale :

- pas de Pip ;
- pas de Tkinter ;
- pas de bibliothèques tierces (Pillow, Requests, NumPy, CustomTkinter, etc.) ;
- pas d’outils de développement.

On ajoute uniquement ce qui est nécessaire au projet.

<div id="section-24"></div>

### 2.4 Version recommandée et compatibilité système

Dans ce guide, nous utiliserons **Python 3.11.0 (64 bits)**.  
Cette version est stable, largement compatible avec les bibliothèques courantes, et c’est celle utilisée dans l’application présentée en exemple.

* **Historique et compatibilité du format portable :**
* Le pack portable (*Windows Embeddable Package*) est disponible nativement à partir de la version **3.5** de Python.
* Concernant les systèmes d'exploitation, ce guide peut être suivi dès la version **3.5** si vous ciblez encore **Windows 7** (le support officiel de Windows 7 s'arrêtant avec Python 3.8).
* **Attention :** À partir de **Python 3.9**, Windows 7 n'est plus pris en charge par l'interpréteur. Pour utiliser **Python 3.11** (recommandé dans ce guide), la version minimale requise est **Windows 8** (ou Windows 8.1, 10, 11).

Mais vous pouvez prendre d'autres versions répondant à vos besoins.

### 2.5 Compatibilité des composants

Les éléments ajoutés doivent être **compatibles** avec la version de Python utilisée.

Exemple correct :

```text
Python portable : 3.11.9 (64 bits)

Compatible :
- Pip pour Python 3.11
- Tkinter provenant d’une installation Python 3.11 64 bits
- Bibliothèques compilées pour Python 3.11
```

À éviter :

```text
Python portable : 3.11
Bibliothèque compilée : Python 3.12

ou

Python portable : 64 bits
Bibliothèques compilées : 32 bits
```

Cela peut provoquer :

- erreurs d’import ;
- incompatibilités de DLL ;
- problèmes graphiques.

---

<div id="section-3"></div>

## 3. Installation minimale de Python portable

Cette partie explique comment installer Python portable dans votre projet, en utilisant la version recommandée : **Python 3.11.0 (64 bits)**(compatible à partir de Windows 8)
Pour tout choix d'une autre version selon votre système d'exploitation, référez-vous à la [section 2.4](#section-24)

### 3.1 Téléchargement de Python portable

Télécharger Python depuis le site officiel :

👉 **https://www.python.org/downloads/windows/**

Dans la liste des fichiers disponibles, choisir :

```
Python 3.11.0 - Oct. 24, 2022
- Download Windows embeddable package (64-bit)
```

Exemple de fichier :

```
python-3.11.0-embed-amd64.zip
```

Ce fichier ZIP contient la version portable de Python.

### 3.2 Création du projet

Créer un dossier principal :

```text
MonProjet\
```

Puis :

```text
MonProjet\
└── python_env\
```

### 3.3 Extraction de Python

Extraire tout le contenu de l’archive ZIP dans :

```text
MonProjet\python_env\
```

Vous pouvez ensuite supprimer le fichier ZIP.

Structure obtenue :

```text
MonProjet\
└── python_env\
    ├── python.exe
    ├── pythonw.exe
    ├── python311.dll
    ├── python311.zip
    ├── python311._pth
    └── ...
```

### 3.4 Premier test

Dans un terminal ouvert dans `MonProjet` :

```bat
python_env\python.exe --version
```

Résultat attendu :

```text
Python 3.11.0
```

### 3.5 Test d’exécution

Créer un fichier :

```text
MonProjet\test.py
```

Contenu :

```python
print("Python portable fonctionne correctement")
```

Lancer :

```bat
python_env\python.exe test.py
```

Résultat :

```text
Python portable fonctionne correctement
```

### 3.6 Résultat de l’installation minimale

Vous possédez maintenant :

✅ Python autonome  
✅ une installation indépendante de Windows  
✅ un environnement transportable  
✅ la bibliothèque standard Python  
✅ la possibilité d’exécuter des scripts.

Vous **ne possédez pas encore** :

❌ bibliothèques tierces  
❌ Pip  
❌ Tkinter  
❌ configuration VS Code  
❌ scripts de lancement  
❌ icône Windows.

Si votre projet n’a besoin que de Python standard, vous pouvez vous arrêter ici.

---

<div id="section-4"></div>

## 4. Activation du système de modules externes

Pour pouvoir utiliser Pip et des bibliothèques externes, il faut configurer le fichier :

```text
python_env\python311._pth
```

(Le nom varie selon la version.)

Ouvrez ce fichier avec un éditeur de texte et remplacez son contenu par :

```text
python311.zip
.
Scripts
Lib\site-packages

# Uncomment to run site.main() automatically
import site
```

### Explications

- `python311.zip`  
  Charge la bibliothèque standard de Python.

- `.`  
  Autorise le chargement des modules présents dans le dossier courant.

- `Scripts`  
  Permet d’utiliser les exécutables installés par Pip.

- `Lib\site-packages`  
  Emplacement où seront installées les bibliothèques Python.

- `import site`  
  **Essentiel** : sans cette ligne, Python reste en mode isolé, Pip ne fonctionne pas correctement et les packages ne sont pas détectés.

---

<div id="section-5"></div>

## 5. Installation de Pip

La version Embeddable ne contient pas Pip. Il faut l’ajouter manuellement.

### 5.1 Télécharger `get-pip.py`

Télécharger le script officiel :

👉 **https://bootstrap.pypa.io/get-pip.py**

Enregistrer le fichier `get-pip.py` **à la racine du projet** :

```text
MonProjet\
├── get-pip.py
└── python_env\
```

### 5.2 Installation

Dans un terminal ouvert dans `MonProjet` :

```bat
python_env\python.exe get-pip.py
```

Une fois l'installation terminée, vous pouvez supprimer `get-pip.py`.

### 5.3 Résultat

Python crée automatiquement :

```text
python_env\
├── Scripts\
└── Lib\
    └── site-packages\
```

Votre environnement peut désormais installer des bibliothèques Python.

---

<div id="section-6"></div>

## 6. Installation des bibliothèques Python externes

Toutes les bibliothèques doivent être installées avec :

```bat
python_env\python.exe -m pip install NomDuPackage
```

Exemples :

```bat
python_env\python.exe -m pip install pillow
python_env\python.exe -m pip install requests
python_env\python.exe -m pip install customtkinter
```

Les packages sont installés dans :

```text
python_env\Lib\site-packages\
```

> 💡 Vous pouvez également utiliser le fichier Batch présenté dans la **section 9.4 du guide**.  
> Ce script (`Installer_Libs.bat`) se charge de rappeler automatiquement la commande Pip pour vous et affiche une petite interface dans le terminal où il suffit simplement de **taper le nom de la bibliothèque**.

### Recommandation

Installez uniquement les bibliothèques **nécessaires** à votre application afin de conserver un environnement portable **léger**, **rapide** et **facile à distribuer**.

---

<div id="section-7"></div>

## 7. Ajouter Tkinter

La version portable de Python ne contient **pas** Tkinter.  
C’est la seule différence majeure avec une installation classique.

👉 Tkinter n’est **pas installable via pip**  
👉 Tkinter n’est **pas disponible en téléchargement indépendant**  
👉 Tkinter doit obligatoirement être récupéré depuis **une installation complète de Python**

Pourquoi ?  
Parce que Tkinter dépend de fichiers compilés et fournis uniquement dans une installation Python classique :

- `_tkinter.pyd`
- `tcl86t.dll`
- `tk86t.dll`
- le dossier `tcl\` **complet**
- le dossier `Lib\tkinter\` **complet**

Ces fichiers **n’existent pas** dans Python portable et **ne peuvent pas être installés autrement**.

la méthode recommandée est une **installation temporaire** de Python, dans la **même version** que la version portable.

### 7.1 Télécharger de Python

Rendez-vous sur le site officiel :

👉 https://www.python.org/downloads/windows/

Téléchargez **la même version** que votre Python portable.  
Exemple : si votre version portable est **Python 3.11.0**  
(`python-3.11.0-embed-amd64.zip` — *Windows embeddable package (64‑bit)*),  
téléchargez **Python 3.11.0**  
(`python-3.11.0-amd64.exe` — *Windows installer (64‑bit)*).

Cela garantit que les DLL et les fichiers Tcl/Tk seront **100 % compatibles**.

### 7.2 Installation temporaire (propre et supprimable)

Lors de l’installation :

1. Cliquez sur **Customize installation**

2. Dans **Optional Features** :

👉 **Cocher uniquement : `tcl\tk and IDLE`**

Les autres options ne sont pas nécessaires pour Tkinter.

3. Dans **Advanced Options** :

👉 **Tout décocher**

Ces options peuvent modifier le système (PATH, associations de fichiers, installation globale, précompilation, etc.).  
Les laisser activées pourrait perturber Windows ou créer des conflits avec d’autres versions de Python.  
Pour une installation temporaire, elles doivent donc rester **toutes désactivées**.

4. En bas de *Advanced Options* :

👉 **Sélectionner un dossier d’installation temporaire**

Choisissez un dossier dédié, par exemple :

```
C:\PythonTemp\
```

Puis terminez l’installation.

Une fois l'installation terminée tu peux supprimer l'installateur.

> 💡 Cette installation ne modifie pas Windows.  
> Pour la supprimer, il suffit d’effacer le dossier `C:\PythonTemp\`.

### 7.3 Récupération des fichiers Tkinter

Une fois Python installé temporairement, vous devez récupérer **exactement les fichiers suivants** :

#### 📁 1. Le dossier `tkinter`
```
C:\PythonTemp\Lib\tkinter\
```

Copier vers :

```
python_env\Lib\site-packages\tkinter\
```

#### 📁 2. Le dossier `tcl`
```
C:\PythonTemp\tcl\
```

Vers :

```
python_env\tcl\
```

#### 📄 3. Les DLL Tcl/Tk + `_tkinter.pyd`

Dans une installation python.org, les DLL se trouvent dans :

```
C:\PythonTemp\DLLs\
```

Vous devez copier :

```
_tkinter.pyd
tcl86t.dll
tk86t.dll
```

Vers :

```
python_env\
```

### 7.4 Vérification

Lancez :

```bat
python_env\python.exe
```

Puis :

```python
import tkinter as tk
root = tk.Tk()
root.mainloop()
```

Si une fenêtre apparaît, Tkinter est installé correctement.

### 7.5 Suppression de l’installation temporaire

Une fois les fichiers copiés :

👉 Supprimez simplement le dossier `C:\PythonTemp\`  
👉 Tkinter fonctionnera désormais uniquement dans votre version portable

### 7.6 Important : Tkinter n’est pas installable via pip

Tkinter dépend de fichiers compilés et de DLL spécifiques à Python.  
Pip ne peut pas installer :

- `_tkinter.pyd`
- `tcl86t.dll`
- `tk86t.dll`
- le dossier `tcl\`

Toute bibliothèque dépendante (ex : `tkinterdnd2`)  
nécessite Tkinter **déjà présent** dans l’environnement portable.

---

<div id="section-8"></div>

## 8. Utilisation de Visual Studio Code avec le Python portable

VS Code peut utiliser le Python portable du projet pour exécuter les scripts Python (Run, Debug, F5, bouton Play).  
Les versions récentes de VS Code **n’appliquent plus automatiquement cet interpréteur au terminal intégré**.  
C’est normal.

### 8.1 Ouvrir le projet à la racine

VS Code applique correctement l’interpréteur **uniquement** si vous ouvrez le dossier du projet **à sa racine** :

✔ Correct :  
```
Ouvrir : C:\MonProjet\
```

❌ Incorrect :  
```
Ouvrir : C:\MonProjet\src\
Ouvrir : C:\MonProjet\python_env\
Ouvrir : un fichier seul
```

### 8.2 Sélectionner l’interpréteur pour exécuter les scripts

Dans VS Code :

1. **Ctrl + Shift + P**  
2. Tapez :  
   **Python: Select Interpreter**  
3. Choisissez :  
   **python_env\python.exe**

VS Code utilisera cet interpréteur pour :

- **Run Python File**
- **Debug Python File**
- **F5**
- **Bouton Play**

> 🛈 VS Code crée automatiquement un dossier **`.vscode\`** dans le projet.  
> Ce dossier contient des réglages internes, **mais il ne stocke pas le choix de l’interpréteur**.

> ⚠️ Le choix de l’interpréteur est enregistré **uniquement dans le cache local de VS Code**.  
> Sur une autre machine, VS Code **oubliera ce réglage** :  
> il faudra **re‑sélectionner l’interpréteur** via Ctrl + Shift + P.

### 8.3 Utiliser Python dans le terminal intégré

Le terminal intégré **n’utilise pas automatiquement** l’interpréteur sélectionné.  
C’est le comportement normal des versions récentes de VS Code.

Pour utiliser le Python portable dans le terminal, **veuillez pointer vers un code Python qui existe** (si vous n'avez pas de code à des fins de test, vous pouvez utiliser le code donné en partie 10.5) :

```powershell
.\python_env\python.exe
```

Pour exécuter un script :

```powershell
.\python_env\python.exe .\src\main.py
```

---

<div id="section-9"></div>

## 9. Scripts Batch de lancement et de maintenance

Les scripts `.bat` simplifient l’utilisation du projet.

### 9.1 Ligne indispensable

Au début de chaque script :

```bat
cd /d "%~dp0"
```

Cette ligne place le terminal dans le dossier du script, ce qui garantit que tous les chemins relatifs fonctionnent, même si le script est lancé depuis un autre emplacement.

### 9.2 `Lancer_App.bat` (mode normal, sans console)

Pour exécuter du code Python, assurez-vous de pointer vers un fichier existant (**si vous n'avez pas de code à des fins de test, vous pouvez utiliser le code donné en  partie 10.5**) :

```bat
@echo off

cd /d "%~dp0"

start "" "python_env\pythonw.exe" "src\main.py"
```

- Utilise `pythonw.exe` pour ne pas afficher de console.
- Lance `src\main.py`.

> 💡 L'exécution d'un fichier .bat provoque toujours un léger clignotement de terminal à l'écran, même s'il lance une application sans console. Ce comportement est tout à fait normal.

### 9.3 `Debug.bat` (mode debug, avec console)

Pour tester ou déboguer, veillez à pointer vers un script valide (**si vous n'avez pas de code à des fins de test, vous pouvez utiliser le code donné en partie 10.5**) :

```bat
@echo off

cd /d "%~dp0"

echo ===================================================
echo        Lancement en mode Debug
echo ===================================================
echo.

"python_env\python.exe" "src\main.py"

echo.
echo ===================================================
echo Application terminee.
pause
```

- Utilise `python.exe` pour afficher la console.
- Permet de voir les erreurs et les `print()`.

### 9.4 `Installer_Libs.bat` (installation de bibliothèques)

```bat
@echo off

cd /d "%~dp0"

echo ===================================================
echo       Installateur de bibliotheque python pip
echo ===================================================
echo.

set /p LIB=" >> python_env\python.exe -m pip install "

echo.

python_env\python.exe -m pip install %LIB%

echo.
echo Installation terminee.
pause
```

- Demande le nom d’une bibliothèque ;
- l’installe dans l’environnement portable via Pip.

---

<div id="section-10"></div>

## 10. Création d’un lanceur `.exe` natif Windows (avec support de l'image de démarrage)

Pour transformer vos scripts en une véritable application Windows professionnelle et portable, nous allons créer un **lanceur exécutable natif en C#**.

Cette méthode utilise directement le compilateur natif inclus par défaut dans Windows, ce qui signifie qu'**aucune installation de logiciel tiers ou d'outil lourd n'est nécessaire**.

#### Ce que permet ce lanceur exécutable :

* D’avoir une **icône propre et un nom d'application** dans la barre des tâches.
* De **regrouper et fusionner les fenêtres** d'une même application sous une seule et même icône grâce à l'attribution dynamique d'un identifiant unique (`AppUserModelID`).
* De **prendre en charge nativement une image de démarrage (Splash Screen / `splash.png`)** s'affichant de manière fluide au démarrage pendant que Python s'initialise, avec une fermeture synchronisée pilotée par l'application.
* De **transmettre des informations dynamiques** à Python au démarrage via des variables d'environnement (comme le rôle du processus — maître ou esclave —, l'état de la console, et le nom du mutex du splash screen).
* De **transmettre les arguments de ligne de commande** saisis par l'utilisateur (ex: fichiers glissés-déposés ou paramètres lancés en CLI) directement au script Python (`sys.argv`).
* De **prendre en charge nativement le module `multiprocessing` de Python**, en évitant que les processus de calcul (workers) ne relancent en boucle le lanceur C# ou n'ouvrent des fenêtres graphiques de manière intempestive.
* De **gérer automatiquement les logs de lancement et de crash**, en enregistrant dans un dossier dédié (`logs/`) les éventuelles erreurs d'exécution ou rapports de plantage de Python (ce dossier de logs étant automatiquement nettoyé avec une limite de taille d'historique pour éviter qu'il ne s'alourdisse inutilement avec le temps).
* D’éviter l’apparition d’une **console noire** lors du lancement (si configuré en mode GUI pur), tout en offrant la flexibilité de gérer le mode **terminal** ou **GUI** selon la configuration et de l'afficher si des logs doivent y être écrits.
* De rendre l’application plus **fiable**, **propre** et **intégrée** à l'écosystème Windows.
* D'utiliser des **chemins relatifs** garantissant que le dossier du projet reste totalement portable, déplaçable et indépendant de l'emplacement cible sans créer d'erreurs.
* De lancer le script Python dans un environnement portable sans nécessiter la moindre installation ni modification du système sur la machine cible.

Pour cela, nous allons :

1. Préparer l'icône de l'application et l'image de démarrage *(Optionnels)*.
2. Écrire le programme C# (`Launcher.cs`) qui sert de point d’entrée à l’application.
3. Écrire le script Bash (`Build_launcher.bat`) permettant d'automatiser la compilation.
4. Réaliser la compilation.
5. Apprendre à exploiter les informations transmises au démarrage par l'exécutable vers le script Python.

### 10.1 — Préparer l’icône et l’image de démarrage *(Optionnels)*

Si vous souhaitez personnaliser l'apparence de votre application, placez vos fichiers dans les dossiers correspondants :

```text
MonProjet\assets\app.ico
MonProjet\assets\splash.png
```

> **Format recommandé pour l'icône :**  
> `.ico` multi‑résolution (16×16, 20×20, 24×24, 32×32, 40×40, 48×48, 64×64, 128×128, 256×256) en **profondeur de couleur 32 bits (True Color + Transparence Alpha / RGBA)** pour toutes les tailles.
> Pour générer votre fichier `.ico` à partir d'un fichier PNG :  
> * **Option simple (en ligne) :** Téléversez une image PNG carrée de haute qualité avec fond transparent si vous le voulez (ex. 512×512 px) sur le site **[ConvertICO](https://www.convertico.com/)**, puis sélectionnez l'ensemble des résolutions souhaitées avant de lancer la conversion et de télécharger votre fichier.  
> * **Option avancée (logiciel local) :** Pour une plus grande liberté, vous pouvez utiliser **[Greenfish Icon Editor Pro (GFIE)](https://greenfishsoftware.org/gfie.php)** (disponible en version portable ou installable). C'est un peu le "Paint" dédié aux icônes : il permet de retoucher le visuel résolution par résolution, avec une édition fine pixel par pixel.  

> **Format recommandé pour l'image de démarrage :**
> Un fichier `.png` avec transparence (fond transparent ou coins arrondis), qui sera automatiquement centré et redimensionné proportionnellement sur l'écran actif par le lanceur.

### 10.2 — Code du lanceur C# (`Launcher.cs`)

Créer le fichier à la racine de votre projet :

```
MonProjet\Launcher.cs
```

> 💡 **Conseil de configuration :** Pensez à bien vérifier et adapter les paramètres situés tout en haut du code ci-dessous (nom de l'application, affichage de la console, instance unique, chemins relatifs, métadonnées des propriétés de l'exécutable, ...). Le code est **densément commenté** : n'hésitez pas à le lire en détail si vous souhaitez comprendre chaque mécanisme technique sous-jacent.

Contenu :

```cpp
// Launcher.cs
using System;
using System.IO;
using System.Text;
using System.Runtime.InteropServices;
using System.Reflection;
using System.Resources;
using System.Windows.Forms;
using System.Collections.Generic;
using System.Threading;
using System.Linq;
using System.Diagnostics;
using System.Text.RegularExpressions;
using System.Drawing;

// ============================================================================
// MÉTADONNÉES DE LA COMPILATION ET DE LA VERSION DE L'EXÉCUTABLE.
// ============================================================================
// CONFIGURATION DES PARAMÈTRES PRINCIPAUX DE L'APPLICATION.
[assembly: AssemblyTitle("Mon Application Superbe")]                   // -> Description du fichier et titre principal : indique le nom de l'application.
[assembly: AssemblyDescription("Lanceur portable Python .NET Native")] // -> Commentaires et description secondaire : indique le rôle du programme.
[assembly: AssemblyConfiguration("Release")]                           // -> Configuration interne : règle stricte, mettre "Release" ou "Debug".
[assembly: AssemblyCompany("Mon Éditeur")]                             // -> Société ou éditeur : indique l'auteur ou l'entreprise.
[assembly: AssemblyProduct("Mon Application Superbe")]                 // -> Nom du produit : indique le nom global du logiciel.
[assembly: AssemblyCopyright("Copyright © 2026 Mon Éditeur.")]         // -> Droit d'auteur : indique la mention de copyright légale.
[assembly: AssemblyTrademark("Mon Éditeur")]                           // -> Marques légales : indique la marque ou l'éditeur.
[assembly: AssemblyCulture("")]                                        // -> Culture : règle stricte, doit impérativement rester vide.
[assembly: NeutralResourcesLanguage("fr-FR")]                          // -> Langue neutre : indique le code langue (ex: fr-FR).
[assembly: AssemblyVersion("1.0.0.0")]                                 // -> Version d'assemblage : règle stricte, format numérique X.X.X.X.
[assembly: AssemblyFileVersion("1.0.0.0")]                             // -> Version du fichier : règle stricte, format numérique X.X.X.X.
[assembly: AssemblyInformationalVersion("Version 1.0.0.0")]            // -> Version du produit : indique la version publique visible.

// Désactive les warnings de code mort ou inaccessible pour les constantes de configuration.
#pragma warning disable 0162, 0414, 0429

class Program {
    // Paramètres de configuration du lanceur.
    private const bool SHOW_CONSOLE = true;                                    // Affichage de la console : true pour afficher le terminal, false pour masquer.
    private const bool ENABLE_SINGLE_INSTANCE = false;                         // Sécurité instance unique : true pour empêcher les doubles lancements, false pour autoriser plusieurs instances (les instances multiprocessing restent toujours autorisées).
    private static readonly string CONSOLE_TITLE = "Mon Application Terminal"; // Titre de la fenêtre de console si elle est active.
    private static readonly string APP_USER_MODEL_ID = "MonEditeur.App.1.0";   // ID Windows pour regrouper les fenêtres dans la barre des tâches.
    private static readonly string PYTHON_HOME_PATH = @"python_env";           // Dossier Python : chemin relatif vers l'environnement portable.
    private static readonly string SCRIPT_PATH = @"src\main.py";               // Script principal : chemin relatif vers le fichier Python à lancer (Ex : @"src\main.py").
    private static readonly string SPLASH_IMAGE_PATH = @"assets\splash.png";   // Image de démarrage : chemin relatif vers l'image PNG du splash screen ou chaîne vide pour désactiver (Ex : @"assets\splash.png" ou @"").
    // FIN DE LA CONFIGURATION DES PARAMÈTRES.

    // DÉCLARATIONS SYSTÈME : Fonctions Windows (Win32) et API Python (Cdecl).
    [DllImport("shell32.dll", SetLastError = true)] private static extern void SetCurrentProcessExplicitAppUserModelID([MarshalAs(UnmanagedType.LPWStr)] string AppID);
    [DllImport("kernel32.dll", SetLastError = true)] private static extern bool SetDllDirectory(string lpPathName);
    [DllImport("kernel32.dll", SetLastError = true)] private static extern IntPtr LoadLibrary(string lpFileName);
    [DllImport("kernel32.dll", SetLastError = true)] private static extern IntPtr GetProcAddress(IntPtr hModule, string procName);
    [DllImport("kernel32.dll", SetLastError = true)] private static extern bool FreeLibrary(IntPtr hModule);
    [DllImport("kernel32.dll", SetLastError = true)] private static extern bool AllocConsole();
    [DllImport("kernel32.dll", SetLastError = true)] private static extern bool FreeConsole();
    [DllImport("kernel32.dll", SetLastError = true)] private static extern bool SetConsoleTitle(string lpConsoleTitle);
    [UnmanagedFunctionPointer(CallingConvention.Cdecl)] private delegate void Py_SetPythonHomeDelegate([MarshalAs(UnmanagedType.LPWStr)] string home);
    [UnmanagedFunctionPointer(CallingConvention.Cdecl)] private delegate void Py_InitializeExDelegate(int initsigs);
    [UnmanagedFunctionPointer(CallingConvention.Cdecl)] private delegate void Py_FinalizeDelegate();
    [UnmanagedFunctionPointer(CallingConvention.Cdecl)] private delegate int PyRun_SimpleStringFlagsDelegate(IntPtr utf8Code, IntPtr flags);
    [UnmanagedFunctionPointer(CallingConvention.Cdecl)] private delegate void PySys_SetArgvDelegate(int argc, [MarshalAs(UnmanagedType.LPArray, ArraySubType = UnmanagedType.LPWStr)] string[] argv);
    
    // FONCTIONS UTILITAIRES : Résolution de chemins et gestion des erreurs (Format traditionnel .NET 4.0).
    private static string GetAbsolutePath(string relPath) {
        // Combine le répertoire de base de l'application avec un chemin relatif pour obtenir un chemin absolu sécurisé.
        return Path.Combine(AppDomain.CurrentDomain.BaseDirectory, relPath);
    }
    private static string FindPythonDLL(string pyHomeFolder) {
        // Vérifie si le dossier de l'environnement Python existe, retourne null si introuvable.
        if (!Directory.Exists(pyHomeFolder)) return null;
        try {
            // Parcourt les fichiers du dossier pour trouver la DLL principale de Python (ex: python310.dll).
            foreach (string file in Directory.GetFiles(pyHomeFolder, "python3*.dll")) {
                string fname = Path.GetFileName(file).ToLower();
                // Ignore la DLL générique "python3.dll" et les versions de débogage "_d.dll".
                if (fname != "python3.dll" && !fname.EndsWith("_d.dll")) return file;
            }
        } catch {} return null;
    }
    private static void LogDebugInfo(int pid, string suffix, string status, string[] args, string logDir) {
        // Journal de débogage pour analyser les lancements.
        try {
            string formattedArgs = (args != null && args.Length > 0) 
                ? "Arguments: [" + string.Join(", ", args) + "]" 
                : "Arguments: []";
                
            DateTime now = DateTime.Now; // Capture l'heure une seule fois pour éviter tout décalage.
            string tsFile = now.ToString("yyyyMMdd-HHmmss"); // Format pour le nom du fichier.
            string tsLog = now.ToString("yyyy-MM-dd HH:mm:ss"); // Format pour l'intérieur du fichier.
            
            string logContent = "[" + tsLog + "] STATUT: " + status + "\n" +
                                "PID: " + pid + "\n" +
                                formattedArgs + "\n" +
                                "---------------------------------------------------\n";
            string currentLogPath = Path.Combine(logDir, tsFile + "_" + pid + "_" + suffix + ".log");
            File.WriteAllText(currentLogPath, logContent, Encoding.UTF8);
        } catch { }
    }
    private static void CleanupOldMasterLogs(string logDir) {
        // Nettoie les anciens fichiers de logs pour ne conserver que les 3 dernières sessions "Maître".
        // Filtre pour identifier uniquement les fichiers de l'application (qui commencent par une date AAAAMMJJ).
        Regex datePrefixRegex = new Regex(@"^\d{8}");
        // Récupère tous les fichiers valides du dossier et les trie du plus récent au plus ancien.
        var datedFiles = Directory.GetFiles(logDir)
                                .Select(f => new FileInfo(f))
                                .Where(f => datePrefixRegex.IsMatch(f.Name))
                                .OrderByDescending(f => f.CreationTime)
                                .ToList();
        int masterLogCount = 0;
        // Parcourt la liste du plus récent au plus vieux pour repérer les maîtres et supprimer les excédents.
        foreach (var file in datedFiles) {
            // Si on a déjà dépassé les 5 maîtres, tout le reste est trop ancien : on supprime.
            if (masterLogCount >= 5) {
                try { file.Delete(); } catch { } // Ignore l'erreur si le fichier est ouvert (ex: Bloc-notes).
            }
            // Sinon, on compte ce fichier comme une log de maître ("-LM.log") en plus.
            else if (file.Name.EndsWith("_LM.log", StringComparison.OrdinalIgnoreCase)) {
                masterLogCount++;
            }
        }
    }
    [STAThread]
    static void Main(string[] args) {
        // ====================================================================
        // ÉTAPE 1 : VÉRIFICATION IMMÉDIATE DE L'INSTANCE UNIQUE ET DÉMARRAGE DU SPLASH SCREEN.
        // ====================================================================
        // Détecte si l'exécutable est appelé par le module multiprocessing de Python (mode spawn).
        bool isMultiprocessingChild = args.Any(arg => arg != null && (arg.Contains("multiprocessing") || arg.Contains("--multiprocessing")));
        // Préparation du dossier de logs.
        string logDir = GetAbsolutePath("logs");
        if (!Directory.Exists(logDir)) Directory.CreateDirectory(logDir);
        // Récupère le PID actuel.
        int CurrentPID = System.Diagnostics.Process.GetCurrentProcess().Id;
        bool isNewInstance = true;
        Mutex mutex = null;
        // Cas 1 : C'est un sous-processus de calcul (Multiprocessing).
        if (isMultiprocessingChild) {
            LogDebugInfo(CurrentPID, "LP", "Launch Multiprocessing", args, logDir);
        }
        // Cas 2 : C'est un lancement manuel effectué par l'utilisateur.
        else {
            string MUTEX_NAME = "Global\\" + APP_USER_MODEL_ID + "_Mutex"; // Nom unique du Mutex système global : utilise l'ID de l'application pour garantir un verrou matériel propre à ce programme.
            mutex = new Mutex(true, MUTEX_NAME, out isNewInstance);
            // Si le Mutex appartient déjà à une autre instance (ce n'est pas la première).
            if (!isNewInstance) {
                // Cas 2A : L'instance unique est activée (true) -> Le lancement est rejeté et bloqué.
                if (ENABLE_SINGLE_INSTANCE) {
                    LogDebugInfo(CurrentPID, "LSr", "Launch Secondary Refused", args, logDir);
                    // On affiche une alerte graphique pour prévenir l'utilisateur.
                    MessageBox.Show(
                        "Une instance de l'application est déjà en cours d'exécution.", 
                        "Application déjà ouverte", 
                        MessageBoxButtons.OK, 
                        MessageBoxIcon.Warning
                    );
                    return; // Fin du processus secondaire : il s'arrête proprement ici.
                }
                // Cas 2B : L'instance unique est désactivée (false) -> Les lancements multiples sont autorisés.
                else {
                    SplashScreenManager.Start(SPLASH_IMAGE_PATH, APP_USER_MODEL_ID, CurrentPID, args); // Démarre l'affichage de l'image de démarrage dans un thread d'arrière-plan sécurisé si toutes les conditions sont réunies.
                    LogDebugInfo(CurrentPID, "LSv", "Launch Secondary Validated / Valid", args, logDir);
                }
            } 
            // Cas 2C : C'est la toute première instance lancée -> C'est le processus Maître (LM).
            else  {
                SplashScreenManager.Start(SPLASH_IMAGE_PATH, APP_USER_MODEL_ID, CurrentPID, args); // Démarre l'affichage de l'image
                LogDebugInfo(CurrentPID, "LM", "Launch Master / Maître", args, logDir); 
                CleanupOldMasterLogs(logDir); // Lance le nettoyage des anciens logs du maître.
            }
        }
        // ====================================================================
        // ÉTAPE 2 : GESTION DES CAS (MULTIPROCESSING ENFANT VS SCRIPT PRINCIPAL).
        // ====================================================================
        try {
            // Initialise les styles visuels Windows si on est sur le processus principal.
            if (!isMultiprocessingChild) {
                Application.EnableVisualStyles();
                Application.SetCompatibleTextRenderingDefault(false);
            }
            // Force le répertoire de travail actuel à correspondre au dossier où se trouve l'exécutable.
            Directory.SetCurrentDirectory(AppDomain.CurrentDomain.BaseDirectory);
            // Applique l'ID unique de l'application pour regrouper proprement les fenêtres dans la barre des tâches Windows.
            try { SetCurrentProcessExplicitAppUserModelID(APP_USER_MODEL_ID); } catch {}
            // Transmission des variables globales vers Python.
            Environment.SetEnvironmentVariable("APP_IS_MASTER", isNewInstance ? (ENABLE_SINGLE_INSTANCE ? "1" : "2") : "0"); // 2 : Maître avec instances multiples | 1 : Maître en mode instance unique strict | 0 : Instance secondaire.
            Environment.SetEnvironmentVariable("APP_USER_MODEL_ID", APP_USER_MODEL_ID);                                      // Transmet l'ID unique Windows pour regrouper les fenêtres dans la barre des tâches.
            Environment.SetEnvironmentVariable("APP_HAS_CONSOLE", SHOW_CONSOLE ? "1" : "0");                                 // 1 : Le terminal est actif et visible | 0 : Le terminal est masqué.
            // Note : La variable d'environnement "APP_SPLASH_MUTEX_NAME" est également créée en présence d'une image de démarrage afin que le script Python puisse indiquer à quel moment la fermer.

            // Si l'affichage de la console est activé, alloue un terminal Windows et définit son titre.
            if (SHOW_CONSOLE && AllocConsole()) SetConsoleTitle(CONSOLE_TITLE);
            // Résout le chemin absolu vers le dossier Python portable.
            string pyHome = GetAbsolutePath(PYTHON_HOME_PATH);
            if (!Directory.Exists(pyHome)) throw new DirectoryNotFoundException("Dossier Python introuvable :\n" + pyHome);
            // Configure les chemins des bibliothèques TCL/TK nécessaires si Tkinter est utilisé dans l'application Python.
            string tclPath = Path.Combine(pyHome, @"tcl\tcl8.6"), tkPath = Path.Combine(pyHome, @"tcl\tk8.6");
            if (Directory.Exists(tclPath)) Environment.SetEnvironmentVariable("TCL_LIBRARY", tclPath);
            if (Directory.Exists(tkPath)) Environment.SetEnvironmentVariable("TK_LIBRARY", tkPath);
            // Force Python à désactiver le buffering de sortie et à utiliser l'encodage UTF-8 standard.
            Environment.SetEnvironmentVariable("PYTHONUNBUFFERED", "1");
            Environment.SetEnvironmentVariable("PYTHONIOENCODING", "utf-8");
            // Recherche la DLL native de Python à l'intérieur du dossier portable.
            string dllPath = FindPythonDLL(pyHome);
            if (dllPath == null) throw new FileNotFoundException("Aucune DLL Python valide trouvée dans :\n" + pyHome);
            // Ajoute le répertoire de la DLL au PATH système de recherche Windows et charge dynamiquement la DLL en mémoire.
            SetDllDirectory(pyHome);
            IntPtr hPyDLL = LoadLibrary(dllPath);
            if (hPyDLL == IntPtr.Zero) throw new DllNotFoundException("Échec du chargement de la DLL Python. Code erreur Win32 : " + Marshal.GetLastWin32Error());
            // Récupère les pointeurs vers les fonctions natives de l'API C de Python (initialisation, nettoyage, exécution).
            var Py_SetPythonHome = (Py_SetPythonHomeDelegate)Marshal.GetDelegateForFunctionPointer(GetProcAddress(hPyDLL, "Py_SetPythonHome"), typeof(Py_SetPythonHomeDelegate));
            var Py_InitializeEx = (Py_InitializeExDelegate)Marshal.GetDelegateForFunctionPointer(GetProcAddress(hPyDLL, "Py_InitializeEx"), typeof(Py_InitializeExDelegate));
            var Py_Finalize = (Py_FinalizeDelegate)Marshal.GetDelegateForFunctionPointer(GetProcAddress(hPyDLL, "Py_Finalize"), typeof(Py_FinalizeDelegate));
            var PyRun_SimpleString = (PyRun_SimpleStringFlagsDelegate)Marshal.GetDelegateForFunctionPointer(GetProcAddress(hPyDLL, "PyRun_SimpleStringFlags"), typeof(PyRun_SimpleStringFlagsDelegate));
            IntPtr pSetArgv = GetProcAddress(hPyDLL, "PySys_SetArgv");
            // Configure le dossier racine de Python et initialise l'interpréteur.
            Py_SetPythonHome(pyHome);
            Py_InitializeEx(0);

            // ====================================================================
            // ÉTAPE 3 : CONSTRUCTION DE SYS.ARGV ET OUTIL D'EXÉCUTION PYTHON.
            // ====================================================================
            // Initialise la liste qui servira de base pour le "sys.argv" de l'interpréteur Python.
            var fullArgv = new List<string>();
            string targetScriptPath = "";
            // --- SOUS-ÉTAPE 3A : DÉTERMINATION DU CONTENU DE SYS.ARGV ---
            if (isMultiprocessingChild) {
                // Si c'est un sous-processus multiprocessing, on récupère uniquement les arguments bruts transmis par Python.
                if (args != null) fullArgv.AddRange(args);
            } else {
                // Pour un lancement normal, on cherche le chemin absolu du script.
                targetScriptPath = GetAbsolutePath(SCRIPT_PATH);
                // Si le script est introuvable sur le disque, on lève une exception capturée plus bas.
                if (!File.Exists(targetScriptPath)) throw new FileNotFoundException("Script Python principal introuvable :\n" + targetScriptPath);
                // On place obligatoirement le chemin du script en première position (sys.argv[0]).
                fullArgv.Add(targetScriptPath);
                // On ajoute ensuite à la suite les éventuels arguments saisis par l'utilisateur.
                if (args != null) fullArgv.AddRange(args);
            }
            // --- SOUS-ÉTAPE 3B : TRANSMISSION OFFICIELLE DU TABLEAU À PYTHON ---
            // Transmet le tableau d'arguments final à l'interpréteur pour configurer son "sys.argv" interne.
            if (pSetArgv != IntPtr.Zero) {
                var PySys_SetArgv = (PySys_SetArgvDelegate)Marshal.GetDelegateForFunctionPointer(pSetArgv, typeof(PySys_SetArgvDelegate));
                PySys_SetArgv(fullArgv.Count, fullArgv.ToArray());
            }
            // --- SOUS-ÉTAPE 3C : CRÉATION DE LA FONCTION UTILITAIRE POUR EXÉCUTER DU CODE ---
            // Fonction utilitaire pour exécuter du code Python et intercepter les erreurs de syntaxe/compilation.
            Action<string> ExecPy = (code) => {
                // Convertit le code texte en octets compatibles C (terminés par '\0').
                byte[] bytes = Encoding.UTF8.GetBytes(code + "\0");
                // Alloue la mémoire nécessaire pour transmettre les octets à la DLL de Python.
                IntPtr ptr = Marshal.AllocHGlobal(bytes.Length);
                try { 
                    Marshal.Copy(bytes, 0, ptr, bytes.Length); 
                    // Exécute le code via l'API C de Python et récupère le code de retour (0 = succès, -1 = échec).
                    int result = PyRun_SimpleString(ptr, IntPtr.Zero); 
                    // Si Python rejette le code (ex: erreur de syntaxe ou d'indentation), on lève une erreur explicite.
                    if (result != 0) {
                        throw new Exception(
                            "Erreur de syntaxe ou de compilation dans le code Python embarqué.\n" +
                            "L'interpréteur a rejeté le bloc (Code de retour : " + result + "). Vérifiez l'indentation et les caractères spéciaux."
                        );
                    }
                }
                // Nettoie et libère la mémoire allouée dans tous les cas pour éviter les fuites.
                finally { Marshal.FreeHGlobal(ptr); }
            };
            // Redirige les flux standard (stdout, stderr, stdin) vers le terminal actif si l'affichage console est activé.
            if (SHOW_CONSOLE) ExecPy("import sys\nsys.stdout = open('CONOUT$', 'w', encoding='utf-8')\nsys.stderr = open('CONOUT$', 'w', encoding='utf-8')\nsys.stdin = open('CONIN$', 'r', encoding='utf-8')");
            // Échappe les antislashs du dossier Python pour les injecter en toute sécurité dans l'interpréteur.
            string pyHomeEsc = pyHome.Replace("\\", "\\\\");
            // Ajoute le dossier racine de l'environnement Python au "sys.path" pour qu'il trouve ses modules de base.
            ExecPy("import sys\nsys.path.insert(0, r'" + pyHomeEsc + "')");
            // ====================================================================
            // ÉTAPE 4 : EXÉCUTION DU CODE SELON LE TYPE DE PROCESSUS.
            // ====================================================================
            // Bloc commun de gestion d'erreur Python (avec horodatage unique et commentaires inline).
            string pythonErrorHandler = 
                "except Exception as _outer_err:\n" + // Attrape l'exception globale du script.
                "    try:\n" + // Sécurité globale du gestionnaire d'erreur.
                "        import traceback, ctypes, os, datetime\n" + // Importe les modules nécessaires.
                "        tb_str = traceback.format_exc().replace('\"<string>\"', '\"lanceur_application\"')\n" + // Formatte la pile d'exécution.
                "        now = datetime.datetime.now()\n" + // Récupère l'heure une seule fois pour éviter tout décalage.
                "        err_msg = f\"[{now.strftime('%Y-%m-%d %H:%M:%S')}] PID [" + CurrentPID + "] ERREUR FATALE PYTHON:\\n\" + tb_str\n" + // Construit le message complet avec l'heure formatée.
                "        log_dir = r\"" + logDir + "\"\n" + // Récupère le dossier de logs injecté par le C#.
                "        has_console = " + (SHOW_CONSOLE ? "True" : "False") + "\n" + // Injecte directement le booléen C# (True/False).
                "        log_saved, log_path = True, \"\"\n" + // Initialise l'état et le chemin du fichier log.
                "        try:\n" + // Tentative d'écriture du fichier de log.
                "            log_path = os.path.join(log_dir, now.strftime('%Y%m%d-%H%M%S') + '_" + CurrentPID + "_CP.log')\n" + // Génère le nom unique du log.
                "            with open(log_path, 'w', encoding='utf-8') as f: f.write(err_msg)\n" + // Écrit le rapport sur le disque.
                "        except: log_saved = False\n" + // Désactive le flag si l'écriture échoue.
                "        log_msg = f\"\\n\\nUn rapport détaillé a été enregistré dans :\\n{log_path}\" if log_saved else \"\\n\\n(Impossible d'enregistrer le fichier log sur le disque)\"\n" + // Message de statut du log.
                "        title = 'Erreur Fatale Python' if log_saved else 'Erreur Fatale Python (Échec fichier Log)'\n" + // Définit le titre de la fenêtre d'alerte.
                "        if has_console:\n" + // Utilise directement la variable booléenne injectée.
                "            try:\n" + // Tente d'afficher dans la console.
                "                print(\"\\n\" + err_msg + log_msg)\n" + // Imprime l'erreur dans la console.
                "                input('Appuyez sur une touche pour quitter...')\n" + // Met en pause la console.
                "            except: ctypes.windll.user32.MessageBoxW(0, err_msg + log_msg, title, 0x10)\n" + // Secours MessageBox si la console plante.
                "        else:\n" + // Si pas de console disponible.
                "            try: ctypes.windll.user32.MessageBoxW(0, err_msg + log_msg, title, 0x10)\n" + // Affiche directement en pop-up Windows.
                "            except: pass\n" + // Ignore si le pop-up échoue.
                "    except Exception as _inner_err:\n" + // Dernier recours si le gestionnaire lui-même plante.
                "        import ctypes\n" + // Importe ctypes pour l'alerte critique.
                "        ctypes.windll.user32.MessageBoxW(0, f'CRASH DU GESTIONNAIRE D\\'ERREUR:\\n{str(_inner_err)}', 'Erreur Interne', 0x10)\n"; // Pop-up d'échec critique ultime.
            if (isMultiprocessingChild) {
                // On ajoute le dossier du script au sys.path pour que le worker trouve les modules.
                string scriptDirEsc = Path.GetDirectoryName(GetAbsolutePath(SCRIPT_PATH)).Replace("\\", "\\\\");
                ExecPy("import sys\nsys.path.insert(0, r'" + scriptDirEsc + "')");
                // Récupère l'argument de commande passé par Python (qui suit le "-c").
                string inlinePythonCode = "";
                for (int i = 0; i < args.Length; i++) {
                    if (args[i] == "-c" && i + 1 < args.Length) {
                        inlinePythonCode = args[i + 1];
                        break;
                    }
                }
                // Exécute la commande de spawn de Python avec une capture d'erreur horodatée et une pause/popup en cas de crash.
                string mpRunner = 
                    "try:\n" +
                    "    import multiprocessing.spawn\n" +
                    (string.IsNullOrEmpty(inlinePythonCode) ? "    multiprocessing.spawn.freeze_support()\n" : "    " + inlinePythonCode + "\n") +
                    pythonErrorHandler;
                ExecPy(mpRunner);
            } else {
                // Ajoute le dossier du script principal au sys.path.
                string scriptDirEsc = Path.GetDirectoryName(targetScriptPath).Replace("\\", "\\\\");
                ExecPy("import sys\nsys.path.insert(0, r'" + scriptDirEsc + "')");
                // Exécution sécurisée du script principal avec capture des crashs, horodatage et journalisation.
                string escPath = targetScriptPath.Replace("\\", "\\\\");
                string safePythonRunner = 
                    "try:\n" +
                    "    with open(r'" + escPath + "', 'r', encoding='utf-8') as f: _c = f.read()\n" +
                    "    import __main__\n" +
                    "    __main__.__dict__['__file__'] = r'" + escPath + "'\n" +
                    "    __main__.__dict__['__name__'] = '__main__'\n" +
                    "    exec(compile(_c, r'" + escPath + "', 'exec'), __main__.__dict__)\n" +
                    pythonErrorHandler;
                ExecPy(safePythonRunner);
            }
            // Ferme proprement l'interpréteur Python et libère la DLL chargée en mémoire.
            Py_Finalize();
            FreeLibrary(hPyDLL);
            SetDllDirectory(null);
        } catch (Exception ex) { 
            // Sécurité : Fermeture du Splash Screen si une exception C# survient.
            SplashScreenManager.Stop();

            // Prépare le contenu du message d'erreur avec l'horodatage et les détails de l'exception.
            DateTime now = DateTime.Now; // Capture l'heure une seule fois pour éviter tout décalage.
            string tsFile = now.ToString("yyyyMMdd-HHmmss"); // Format pour le nom du fichier.
            string tsLog = now.ToString("yyyy-MM-dd HH:mm:ss"); // Format pour l'intérieur du fichier.
            string logPath = Path.Combine(logDir, tsFile + "_" + CurrentPID + "_CC.log");
            string errTxt = "[" + tsLog + "] ERREUR CRITIQUE C#:\n" + ex.ToString();
            // Tente d'écrire le rapport de crash sur le disque et mémorise si l'opération a réussi ou échoué.
            bool logSaved = true; try { File.WriteAllText(logPath, errTxt, Encoding.UTF8); } catch { logSaved = false; }
            // Vérifie si la console est demandée et réellement fonctionnelle pour y afficher l'erreur.
            bool consoleOk = false; if (SHOW_CONSOLE) { try { consoleOk = (Console.OpenStandardOutput() != Stream.Null); } catch { } }
            // Prépare le message additionnel informant l'utilisateur de la présence ou non du fichier log.
            string logMsg = logSaved ? "\n\nUn rapport détaillé a été enregistré dans :\n" + logPath : "\n\n(Impossible d'enregistrer le fichier log sur le disque)";
            // Si la console est active, écrit l'erreur, prévient pour le log, met en pause, sinon affiche un pop-up adapté.
            if (consoleOk) { 
                try { Console.WriteLine("\n" + errTxt + logMsg); Console.WriteLine("Appuyez sur une touche pour quitter..."); Console.ReadKey(); } 
                catch { try { MessageBox.Show(errTxt + logMsg, "Erreur Critique C#", MessageBoxButtons.OK, MessageBoxIcon.Error); } catch { } } 
            }
            // Si pas de console, affiche un pop-up de secours (différencié selon que le log a pu être créé ou non).
            else { 
                try { 
                    string title = logSaved ? "Erreur Critique C#" : "Erreur Critique C# (Échec fichier Log)";
                    MessageBox.Show(errTxt + logMsg, title, MessageBoxButtons.OK, MessageBoxIcon.Error); 
                } catch { } 
            }
        } finally { 
            // Exécute le nettoyage final de la console si le processus principal est actif.
            // Remarque : Si vous souhaitez une pause à la fin du script (pour garder le terminal ouvert), 
            // Vous devez placer à la fin de votre script python "input('Fin du programme, appuyez sur une touche pour fermer')".
            if (SHOW_CONSOLE) FreeConsole();
            
            // Arrêt ultime et centralisé du Splash Screen à la fin de l'application.
            SplashScreenManager.Stop();

            // Libère et nettoie proprement le Mutex système si l'instance actuelle le possède.
            if (mutex != null) {
                if (isNewInstance) { try { mutex.ReleaseMutex(); } catch {} mutex.Dispose(); }
            }
        }
    }
}

// ============================================================================
// GESTIONNAIRE CENTRALISÉ DE L'IMAGE DE DÉMARRAGE (SPLASH SCREEN).
// ============================================================================
public static class SplashScreenManager {
    private static Thread splashThread = null; // Thread dédié pour faire tourner l'interface graphique du splash screen en arrière-plan.
    private static Mutex splashCloseMutex = null; // Mutex permettant de synchroniser la fermeture avec le processus principal ou Python.
    private static volatile bool isShuttingDown = false; // Indicateur atomique de demande de fermeture.
    
    // Démarre l'affichage de l'image de chargement dans un thread STA dédié.
    public static void Start(string splashRelPath, string appUserModelId, int currentPid, string[] args) {
        // Résout le chemin absolu de l'image.
        string splashPath = !string.IsNullOrEmpty(splashRelPath) ? Path.Combine(AppDomain.CurrentDomain.BaseDirectory, splashRelPath) : "";
        // Vérifie si l'on doit ignorer l'affichage (via CLI, headless ou no-splash).
        bool forceCli = args != null && args.Any(arg => arg != null && (arg.Equals("--cli", StringComparison.OrdinalIgnoreCase) || arg.Equals("--headless", StringComparison.OrdinalIgnoreCase)));
        bool forceNoSplash = args != null && args.Any(arg => arg != null && (arg.Equals("--no-splash", StringComparison.OrdinalIgnoreCase) || arg.Equals("--nosplash", StringComparison.OrdinalIgnoreCase)));
        // Si l'une des conditions de masquage est remplie ou que l'image n'existe pas, on abandonne.
        if (forceCli || forceNoSplash || string.IsNullOrEmpty(splashPath) || !File.Exists(splashPath)) {
            return;
        }
        
        string SPLASH_MUTEX_NAME = "Global\\" + appUserModelId + "_SplashClose_Mutex_" + currentPid + "_" + DateTime.Now.ToString("yyyyMMdd-HHmmssfff"); // Nom du Mutex pour synchroniser la fermeture propre du Splash Screen avec un horodatage précis permettant de gérer un temps d'affichage minimal facilement côté python.
        Environment.SetEnvironmentVariable("APP_SPLASH_MUTEX_NAME", SPLASH_MUTEX_NAME); // Transmet le nom du Mutex du splash screen à Python pour lui permettre de signaler quand fermer l'image.
        
        bool createdNew;
        splashCloseMutex = new Mutex(true, SPLASH_MUTEX_NAME, out createdNew);
        
        // Crée et lance un thread séparé pour éviter de bloquer le thread principal de chargement.
        splashThread = new Thread(() => {
            Application.EnableVisualStyles();
            Application.SetCompatibleTextRenderingDefault(false);
            
            var form = new SplashForm(splashPath);
            System.Windows.Forms.Timer closeTimer = new System.Windows.Forms.Timer();
            closeTimer.Interval = 100; // Vérifie toutes les 100ms si le splash screen doit se fermer.
            closeTimer.Tick += (sender, e) => {
                try {
                    // Ferme le formulaire si une extinction globale est demandée ou si le Mutex de fermeture est libéré.
                    if (isShuttingDown || (splashCloseMutex != null && splashCloseMutex.WaitOne(0))) {
                        closeTimer.Stop();
                        form.Close();
                    }
                } 
                // Intercepte l'abandon brutal du Mutex (ex: crash ou fermeture forcée de Python/Terminal) pour tuer l'image proprement.
                catch (AbandonedMutexException) {
                    closeTimer.Stop();
                    form.Close();
                } 
                catch {}
            };
            closeTimer.Start();
            Application.Run(form); // Lance la boucle de messages Windows du formulaire.
        });
        
        splashThread.SetApartmentState(ApartmentState.STA); // Obligatoire pour les composants graphiques Windows Forms.
        splashThread.IsBackground = true; // Empêche ce thread de bloquer l'arrêt global de l'application en cas de force majeure.
        splashThread.Start();
    }

    // Demande l'arrêt propre et libère le Mutex associé au splash screen.
    public static void Stop() {
        isShuttingDown = true;
        if (splashCloseMutex != null) {
            try { splashCloseMutex.ReleaseMutex(); } catch {}
            try { splashCloseMutex.Dispose(); } catch {}
            splashCloseMutex = null;
        }
    }
}

// ============================================================================
// FORMULAIRE DE L'IMAGE DE DÉMARRAGE (TRANSPARENCE PNG & GESTIONNAIRE DES TÂCHES).
// ============================================================================
public class SplashForm : Form {
    private Bitmap splashBitmap; // Stocke l'image bitmap redimensionnée.
    private int windowLeft = 0;  // Position X calculée pour le centrage exact.
    private int windowTop = 0;   // Position Y calculée pour le centrage exact.

    // Fonctions API Win32 indispensables pour gérer une fenêtre transparente par pixel et trouver la fenêtre active.
    [DllImport("user32.dll")] private static extern IntPtr GetForegroundWindow();
    [DllImport("gdi32.dll", ExactSpelling = true)] private static extern IntPtr SelectObject(IntPtr hdc, IntPtr hgdiobj);
    [DllImport("gdi32.dll", ExactSpelling = true)] private static extern bool DeleteObject(IntPtr hObject);
    [DllImport("user32.dll", ExactSpelling = true)] private static extern IntPtr GetDC(IntPtr hWnd);
    [DllImport("user32.dll", ExactSpelling = true)] private static extern int ReleaseDC(IntPtr hWnd, IntPtr hDC);
    [DllImport("gdi32.dll", ExactSpelling = true)] private static extern IntPtr CreateCompatibleDC(IntPtr hdc);
    [DllImport("gdi32.dll", ExactSpelling = true)] private static extern bool DeleteDC(IntPtr hdc);
    [DllImport("user32.dll", ExactSpelling = true)] private static extern bool UpdateLayeredWindow(IntPtr hwnd, IntPtr hdcDst, ref POINT pptDst, ref SIZE psize, IntPtr hdcSrc, ref POINT pptSrc, uint crKey, ref BLENDFUNCTION pblend, uint dwFlags);

    [StructLayout(LayoutKind.Sequential)] private struct POINT { public int x; public int y; }
    [StructLayout(LayoutKind.Sequential)] private struct SIZE { public int cx; public int cy; }
    [StructLayout(LayoutKind.Sequential)] private struct BLENDFUNCTION { public byte BlendOp; public byte BlendFlags; public byte SourceConstantAlpha; public byte AlphaFormat; }

    private const byte AC_SRC_OVER = 0x00;
    private const byte AC_SRC_ALPHA = 0x01;
    private const uint ULW_ALPHA = 0x00000002;

    // Surcharge les paramètres de création pour forcer le style de fenêtre superposée (Layered) gérant la transparence alpha.
    protected override CreateParams CreateParams {
        get {
            CreateParams cp = base.CreateParams;
            cp.ExStyle |= 0x00080000; // WS_EX_LAYERED.
            return cp;
        }
    }

    public SplashForm(string imagePath) {
        this.FormBorderStyle = FormBorderStyle.None; // Supprime les bordures et la barre de titre standard.
        this.ShowInTaskbar = false; // Ne s'affiche pas dans la barre des tâches principale de Windows.
        
        // Définit explicitement un nom de fenêtre pour qu'il apparaisse proprement dans le Gestionnaire des Tâches Windows.
        this.Text = "Image de démarrage - " + Application.ProductName; 

        // Charge l'image source et calcule un redimensionnement proportionnel par rapport à l'écran actif.
        using (Image img = Image.FromFile(imagePath)) {
            // Récupère l'écran de la fenêtre active du système au moment du lancement (ex: l'explorateur ou le menu démarrer d'où on a cliqué).
            IntPtr hwndForeground = GetForegroundWindow();
            Screen activeScreen = hwndForeground != IntPtr.Zero ? Screen.FromHandle(hwndForeground) : Screen.PrimaryScreen;
            Rectangle workingArea = activeScreen.WorkingArea;

            int targetWidth = workingArea.Width / 2;
            int targetHeight = workingArea.Height / 2;

            float ratioX = (float)targetWidth / img.Width;
            float ratioY = (float)targetHeight / img.Height;
            float ratio = Math.Min(ratioX, ratioY);

            int finalWidth = (int)(img.Width * ratio);
            int finalHeight = (int)(img.Height * ratio);

            // Calcule et stocke la position exacte pour centrer la fenêtre sur cet écran.
            this.Width = finalWidth;
            this.Height = finalHeight;
            windowLeft = workingArea.Left + (workingArea.Width - finalWidth) / 2;
            windowTop = workingArea.Top + (workingArea.Height - finalHeight) / 2;
            
            this.Left = windowLeft;
            this.Top = windowTop;

            splashBitmap = new Bitmap(finalWidth, finalHeight);
            using (Graphics g = Graphics.FromImage(splashBitmap)) {
                g.InterpolationMode = System.Drawing.Drawing2D.InterpolationMode.HighQualityBicubic;
                g.DrawImage(img, 0, 0, finalWidth, finalHeight);
            }
        }
    }

    protected override void OnLoad(EventArgs e) {
        base.OnLoad(e);
        ShowBitmap(); // Applique le rendu transparent au chargement de la fenêtre.
    }

    // Méthode utilisant l'API Win32 pour peindre l'image PNG avec respect total du canal alpha (transparence).
    private void ShowBitmap() {
        if (splashBitmap == null) return;
        IntPtr screenDc = GetDC(IntPtr.Zero);
        IntPtr memDc = CreateCompatibleDC(screenDc);
        IntPtr hBitmap = IntPtr.Zero;
        IntPtr oldBitmap = IntPtr.Zero;

        try {
            hBitmap = splashBitmap.GetHbitmap(Color.FromArgb(0));
            oldBitmap = SelectObject(memDc, hBitmap);
            
            // Utilise les coordonnées calculées du centre pour l'API Windows.
            POINT topLoc = new POINT { x = windowLeft, y = windowTop };
            SIZE size = new SIZE { cx = splashBitmap.Width, cy = splashBitmap.Height };
            POINT srcLoc = new POINT { x = 0, y = 0 };
            BLENDFUNCTION blend = new BLENDFUNCTION { BlendOp = AC_SRC_OVER, BlendFlags = 0, SourceConstantAlpha = 255, AlphaFormat = AC_SRC_ALPHA };
            UpdateLayeredWindow(this.Handle, screenDc, ref topLoc, ref size, memDc, ref srcLoc, 0, ref blend, ULW_ALPHA);
        }
        finally {
            if (hBitmap != IntPtr.Zero) { SelectObject(memDc, oldBitmap); DeleteObject(hBitmap); }
            DeleteDC(memDc);
            ReleaseDC(IntPtr.Zero, screenDc);
        }
    }

    protected override void Dispose(bool disposing) {
        if (disposing && splashBitmap != null) { splashBitmap.Dispose(); }
        base.Dispose(disposing);
    }
}
```

### 10.3 — Script de compilation (`build_launcher.bat`)

Ce script automatise la compilation du lanceur C#. Il détecte automatiquement l'architecture de votre environnement Python (32 ou 64 bits) et génère l'exécutable `.exe` correspondant.

Créer le fichier à la racine de votre projet :

```text
MonProjet\build_launcher.bat
```

> 💡 **Conseil de configuration :** Prenez le temps de vérifier les paramètres situés au début du script ci-dessous pour vous assurer qu'ils correspondent bien aux noms et chemins de votre projet. Le fichier est **densément commenté** : n'hésitez pas à le lire si vous souhaitez comprendre en détail comment le script interroge l'environnement et pilote le compilateur natif de Windows.

Contenu :

```bat
:: build_launcher.bat
@echo off
chcp 65001 >nul
setlocal enabledelayedexpansion

echo =====================================================
echo   Compilation Automatique du Lanceur (.NET Native)
echo =====================================================
echo.

:: CONFIGURATION DES PARAMÈTRES
:: 1. SRC_FILE : Nom ou chemin (relatif / absolu) du fichier source C# à compiler. (Ex : "Launcher.cs", "src\Launcher.cs", "C:\MonProjet\Launcher.cs")
set "SRC_FILE=Launcher.cs"
:: 2. PY_FOLDER : Chemin du dossier contenant python3*.dll de l'environnement Python portable. (Ex : "python_env", "env\python", "C:\MonProjet\python_env")
set "PY_FOLDER=python_env"
:: 3. ICON_FILE : Chemin du fichier d'icône Windows (.ico) à intégrer au .exe.
::    - Exemples avec icône : "assets\app.ico", "logo.ico", "C:\Images\app.ico"
::    - Exemple sans icône  : set "ICON_FILE="
set "ICON_FILE=assets\app.ico"
:: 4. OUT_EXE : Nom de l'exécutable final généré. (Ex : "MonApplication.exe", "Lanceur.exe") il sera créé à l'emplacement du .bat
set "OUT_EXE=MonApplication.exe"
:: FIN CONFIGURATION DES PARAMÈTRES

:: VÉRIFICATION DE LA PRÉSENCE DU CODE SOURCE C#
if not exist "%SRC_FILE%" (
    echo [ERREUR] Le fichier source '%SRC_FILE%' est introuvable.
    echo Veuillez vérifier le chemin dans le paramètre SRC_FILE du script.
    echo.
    pause
    exit /b 1
)

echo [INFO] Fichier source C# détecté : %SRC_FILE%

:: RECHERCHE ET DÉTECTION DE LA DLL PYTHON PORTABLE
set "DLL_PATH="
if exist "%PY_FOLDER%\python3*.dll" (
    for %%F in ("%PY_FOLDER%\python3*.dll") do (
        set "FILENAME=%%~nxF"
        :: On exclut 'python3.dll' qui est le stub d'interface pour ne garder que la DLL majeure (ex: python310.dll)
        if not "!FILENAME!"=="python3.dll" (
            set "DLL_PATH=%%F"
        )
    )
)

if "%DLL_PATH%"=="" (
    echo [ERREUR] Impossible de trouver une DLL Python dans le dossier '%PY_FOLDER%'.
    echo Assurez-vous que le dossier '%PY_FOLDER%' existe et contient votre environnement Python.
    echo.
    pause
    exit /b 1
)

echo [INFO] DLL Python détectée       : %DLL_PATH%

:: INSPECTION DE L'ARCHITECTURE MÉMOIRE DE LA DLL (32 BITS VS 64 BITS)
set "ARCH=64"
:: Par défaut, on demande au compilateur de forcer le mode 64 bits natif
set "PLATFORM_PARAM=/platform:x64"

:: PowerShell lit les octets d'en-tête PE (Machine Header) de la DLL pour vérifier l'architecture
powershell -NoProfile -ExecutionPolicy Bypass -Command "$bytes = [System.IO.File]::ReadAllBytes('%DLL_PATH%'); $peOffset = [System.BitConverter]::ToInt32($bytes, 0x3C); $machine = [System.BitConverter]::ToUInt16($bytes, $peOffset + 4); if ($machine -eq 0x014c) { exit 32 } else { exit 64 }"

if %errorlevel% equ 32 (
    set "ARCH=32"
    :: Si la DLL est en 32 bits, on force le .exe à tourner en mode 32 bits (WOW64 sous Windows 64 bits)
    set "PLATFORM_PARAM=/platform:x86"
)

echo [INFO] Architecture Python       : %ARCH% bits

:: SÉLECTION DU COMPILATEUR C# WINDOWS NATIF CORRESPONDANT (csc.exe)
if "%ARCH%"=="64" (
    set "CSC=%SystemRoot%\Microsoft.NET\Framework64\v4.0.30319\csc.exe"
) else (
    set "CSC=%SystemRoot%\Microsoft.NET\Framework\v4.0.30319\csc.exe"
)

if not exist "%CSC%" (
    echo [ERREUR] Compilateur C# Windows %ARCH% bits introuvable à l'emplacement :
    echo %CSC%
    echo.
    pause
    exit /b 1
)

echo [INFO] Compilateur sélectionné   : %CSC%

:: GESTION INTUITIVE ET SÉCURISÉE DE L'ICÔNE D'APPLICATION (.ICO)
set "ICON_PARAM="

:: Cas 1 : L'utilisateur n'a pas renseigné le paramètre ICON_FILE (variable vide)
if "%ICON_FILE%"=="" (
    echo [INFO] Aucun chemin d'icône fourni, donc l'exe n'aura pas d'icône.
) else (
    :: Cas 2 : L'utilisateur a fourni un chemin, on vérifie si le fichier existe
    if exist "%ICON_FILE%" (
        echo [INFO] Icône personnalisée       : '%ICON_FILE%' trouvée et intégrée au .exe.
        set ICON_PARAM=/win32icon:"%ICON_FILE%"
    ) else (
        :: Cas 3 : Un chemin a été spécifié mais le fichier n'a pas été trouvé à cet endroit
        echo [AVERTISSEMENT] Icône spécifiée '%ICON_FILE%' introuvable à cet emplacement.
        echo                 L'exécutable généré sera compilé sans icône personnalisée.
    )
)

echo.
echo Compilation de l'exécutable en cours...

:: ============================================================================
:: LANCEMENT DE LA COMPILATION DU BINAIRE C# AVEC CSC.EXE
:: ============================================================================
:: Explication détaillée de chaque option passée au compilateur :
::  - "%CSC%"          : Chemin d'accès du compilateur c# windows natif correspondant (csc.exe)
::  - /nologo          : Supprime la bannière d'en-tête Microsoft au lancement de csc.exe.
::  - /target:winexe   : Génère un exécutable Windows GUI (évite l'ouverture du terminal noir au démarrage).
::  - /optimize+       : Enlève tous les chemins morts (les 'if false', que la condition soit 'false' directement ou qu'une variable soit constamment 'false').
::  - /debug-          : Enlève tous les commentaires, métadonnées et fichiers de débogage dans le .exe pour garder un binaire propre.
::  - %PLATFORM_PARAM% : Force le mode d'exécution (/platform:x86 ou /platform:x64) pour correspondre exactement à l'architecture de la DLL Python.
::  - /out:"%OUT_EXE%" : Définit le nom et l'emplacement du fichier exécutable généré (.exe).
::  - %ICON_PARAM%     : Injecte le fichier .ico dans la table des ressources du binaire (uniquement si renseigné et trouvé).
::  - "%SRC_FILE%"     : Fichier source C# contenant le code du lanceur.

"%CSC%" /nologo /target:winexe /optimize+ /debug- %PLATFORM_PARAM% /out:"%OUT_EXE%" %ICON_PARAM% "%SRC_FILE%"

if %errorlevel% neq 0 (
    echo.
    echo =====================================================
    echo [ERREUR] La compilation a échoué.
    echo Vérifiez le code source C# dans '%SRC_FILE%'.
    echo =====================================================
    echo.
    pause
    exit /b 1
)

echo.
echo =====================================================
echo   SUCCÈS : Fichier '%OUT_EXE%' généré (%ARCH% bits) !
echo =====================================================
echo.
pause
```

Voici la version mise en forme et prête à être intégrée pour la section 10.4 :

### 10.4 — Exécution et résultat de la compilation

#### 1. Arborescence minimale requise

Puisque le script utilise des chemins relatifs, il doit être placé à la racine de votre projet. Avant de lancer la compilation, vérifiez que vous disposez au minimum de la structure suivante :

```text
MonProjet\
├── assets\
│   ├── app.ico             (Optionnel : votre fichier d'icône)
│   └── splash.png          (Optionnel : image de démarrage / Splash Screen)
├── python_env\             (Dossier de l'environnement Python portable)
│   └── python310.dll       (Ou toute autre version de la DLL Python)
├── src\
│   └── main.py             (Le script Python principal de l'application)
├── build_launcher.bat      (Le script de compilation à la racine)
└── Launcher.cs             (Le code source C# du lanceur)

```

#### 2. Lancement

Double-cliquez simplement sur le fichier `build_launcher.bat`. Le script s'exécute dans le terminal, détecte automatiquement l'architecture de la DLL Python (32 ou 64 bits), puis génère l'exécutable.

#### 3. Résultat et nettoyage

Une fois le message `[SUCCÈS]` affiché :

* **Emplacement du `.exe` :** Votre exécutable (ex. `MonApplication.exe`) est généré à la racine de votre projet, exactement dans le même dossier que le script `.bat`.
* **Taille du binaire :** L'exécutable généré est très léger : son poids équivaut environ à celui du code C# compilé ($~30\text{ ko}$) augmenté de la taille de votre icône.
* **Gestion des fichiers de build (Optionnel) :** Vous **pouvez supprimer** `Launcher.cs` et `build_launcher.bat` pour distribuer un dossier final plus épuré, ou **les conserver** si vous prévoyez d'apporter des modifications à l'enveloppe du `.exe` (par exemple pour modifier l'icône, changer le chemin du script pointeur ou afficher/masquer la console). Leur présence n'impacte en rien les performances.

> **Rappel :** Ce `.exe` n'est qu'un simple lanceur sans logique métier. Toutes les modifications futures de votre application (code Python, bibliothèques, scripts) s'effectueront directement dans vos fichiers Python **sans jamais avoir besoin de recompiler l'exécutable**.

> ### 💡 Astuce : Actualisation de l'icône dans l'Explorateur Windows
> Lorsque vous recompilez votre exécutable avec une nouvelle icône, l'Explorateur Windows peut continuer d'afficher l'ancienne. C'est un comportement normal : Windows met en cache les icônes des fichiers pour optimiser ses performances et ne détecte pas qu'un exécutable existant a changé de visuel.
> Pour forcer l'affichage de la nouvelle icône, l'une de ces méthodes fonctionnera :
> * **Renommer l'exécutable** avec un nouveau nom.
> * **Redémarrer l'Explorateur Windows** via le Gestionnaire des tâches (`Ctrl + Maj + Échap`).
> * **Redémarrer l'ordinateur**.

### 10.5 — Exploitation des informations transmises et intégration dans Python

Dans cette section, nous faisons la distinction entre deux types d'éléments :

1. **Ce que l'exécutable transmet** (via les variables d'environnement, les arguments et la gestion du splash screen) : le rôle du processus, l'état de la console, le nom du mutex pour fermer l'image de démarrage et les paramètres de ligne de commande.
2. **Ce que l'on applique de notre côté** dans le code Python : la fermeture synchronisée de l'image de démarrage (`splash.png`), le regroupement des fenêtres dans la barre des tâches (qui utilise l'identifiant transmis) et l'application de l'icône de l'application (qui repose sur un fichier `.ico` totalement indépendant du `.exe`).

Voici un exemple complet de script Python intégrant ces aspects, avec des valeurs par défaut pour que le code fonctionne aussi bien à l'intérieur du `.exe` qu'en phase de développement (par exemple sous VS Code).

#### Code source complet (`main.py`)

Créer le fichier dans le dossier de votre projet :

```text
MonProjet\src\main.py
```

Contenu :

```python
# main.py

print("Hello World - Terminal")

import os
import sys
import ctypes
from datetime import datetime

# 1. RÉCUPÉRATION DE TOUTES LES INFORMATIONS ET VARIABLES D'ENVIRONNEMENT
app_role = os.environ.get("APP_IS_MASTER", "Inconnu (Hors lanceur)") # on définit un cas par défaut dans le cas où ce n'est pas exécuté par le exe exemple VS code
app_id = os.environ.get("APP_USER_MODEL_ID", "Non défini")
has_console = os.environ.get("APP_HAS_CONSOLE", "1") == "1"
splash_mutex_name = os.environ.get("APP_SPLASH_MUTEX_NAME", "Aucun (Pas de splash)")
args = sys.argv[1:]  # Récupère tous les arguments de la ligne de commande

# Traduction lisible du rôle pour le rapport
role_desc = {
    "2": "Maître (Multi-instances autorisées)",
    "1": "Maître (Instance unique stricte)",
    "0": "Processus Esclave (Multiprocessing / Exécution multi)"
}.get(app_role, f"Autre / Inconnu ({app_role})")

console_desc = "Activé (Terminal visible)" if has_console else "Désactivé (Mode pur GUI)"

# Construction du rapport textuel complet
env_report = (
    "=== RAPPORT DES VARIABLES D'ENVIRONNEMENT ===\n\n"
    f"• Rôle de l'application : {role_desc}\n"
    f"• ID Modèle Utilisateur : {app_id}\n"
    f"• Console active : {console_desc}\n"
    f"• Mutex du Splash Screen : {splash_mutex_name}\n"
    f"• Arguments (sys.argv) : {args if args else 'Aucun'}"
)

def get_splash_wait_time(min_duration_sec=1.0):
    """
    Calcule le temps d'attente restant en millisecondes pour respecter 
    la durée minimale d'affichage du splash screen à partir de son horodatage. 
    Garantit une valeur minimale d'une milliseconde pour la file d'attente GUI.
    """
    if not splash_mutex_name or splash_mutex_name == "Aucun (Pas de splash)":
        return 1

    try:
        # Récupère l'horodatage situé à la fin du nom du mutex
        timestamp_str = splash_mutex_name.split("_")[-1]
        launch_time = datetime.strptime(timestamp_str, "%Y%m%d-%H%M%S%f")
        
        # Calcule le temps écoulé et déduit le délai restant en millisecondes
        elapsed = (datetime.now() - launch_time).total_seconds()
        remaining_ms = int((min_duration_sec - elapsed) * 1000)
        print(f"{elapsed * 1000} ms\n{remaining_ms} ms\n{elapsed * 1000 + max(1, remaining_ms)} ms")

        # Force un minimum d'une milliseconde pour un traitement différé propre
        return max(1, remaining_ms)
    except Exception as e:
        print(f"Erreur lors du calcul du délai du splash screen : {e}")
        return 1

def close_splash_screen():
    """
    FONCTION D'ARRÊT DU SPLASH SCREEN :
    Ouvre le Mutex global unique créé par le lanceur C# et le libère (ReleaseMutex) 
    pour ordonner la fermeture immédiate de l'image de démarrage.
    """
    if splash_mutex_name and splash_mutex_name != "Aucun (Pas de splash)":
        try:
            h_mutex = ctypes.windll.kernel32.OpenMutexW(0x00100000, False, splash_mutex_name)
            if h_mutex:
                ctypes.windll.kernel32.ReleaseMutex(h_mutex)
                ctypes.windll.kernel32.CloseHandle(h_mutex)
        except Exception as e:
            print(f"Erreur lors de la fermeture du splash screen : {e}")

def apply_window_focus(root):
    """
    GESTION DU FOCUS DE LA FENÊTRE :
    Force la fenêtre Tkinter à s'afficher au premier plan au moment précis 
    où le splash screen disparaît.
    """
    if root:
        try:
            root.attributes('-topmost', True)
            root.lift()
            root.focus_force()
            root.after(100, lambda: root.attributes('-topmost', False)) # Repasse en mode normal après 100ms
        except:
            pass

def main():
    try:
        # Tentative d'importation et d'initialisation de Tkinter
        import tkinter as tk

        root = tk.Tk()
        root.title("Test Global - Lanceur .NET / Python")
        
        # Masque la fenêtre pendant qu'on la configure pour éviter les sauts visuels
        root.withdraw()
        
        # Dimensions de la fenêtre
        window_width = 520
        window_height = 320

        # Récupère l'écran de la fenêtre active actuelle pour un centrage multi-moniteurs précis
        try:
            user32 = ctypes.windll.user32
            hwnd_fore = user32.GetForegroundWindow()
            h_monitor = user32.MonitorFromWindow(hwnd_fore, 0x00000002) # MONITOR_DEFAULTTONEAREST
            
            class MONITORINFO(ctypes.Structure):
                _fields_ = [
                    ("cbSize", ctypes.c_ulong),
                    ("rcMonitor", ctypes.c_long * 4),
                    ("rcWork", ctypes.c_long * 4),
                    ("dwFlags", ctypes.c_ulong)
                ]
            mi = MONITORINFO()
            mi.cbSize = ctypes.sizeof(MONITORINFO)
            user32.GetMonitorInfoW(h_monitor, ctypes.byref(mi))
            
            work_left, work_top, work_right, work_bottom = mi.rcWork
            screen_w = work_right - work_left
            screen_h = work_bottom - work_top
            
            center_x = work_left + int((screen_w - window_width) / 2)
            center_y = work_top + int((screen_h - window_height) / 2)
        except:
            # Repli sur l'écran principal si l'API Win32 échoue
            screen_width = root.winfo_screenwidth()
            screen_height = root.winfo_screenheight()
            center_x = int((screen_width - window_width) / 2)
            center_y = int((screen_height - window_height) / 2)

        # Applique la taille et la position centrée sur le bon écran
        root.geometry(f"{window_width}x{window_height}+{center_x}+{center_y}")
        
        # Applique l'ID Windows pour fusionner les fenêtres dans la barre des tâches
        try:
            ctypes.windll.shell32.SetCurrentProcessExplicitAppUserModelID(app_id)
        except:
            pass
        
        # Applique l'icône si elle existe
        icon_path = os.path.join("assets", "app.ico")
        if os.path.exists(icon_path):
            root.iconbitmap(icon_path)

        # Utilisation d'une zone de texte unique pour afficher tout le rapport proprement
        text_area = tk.Text(root, wrap="word", font=("Consolas", 10), padx=10, pady=10)
        text_area.pack(expand=True, fill="both", padx=10, pady=10)
        
        text_area.insert("1.0", env_report)
        text_area.config(state="disabled")

        tk.Button(root, text="Fermer", command=root.destroy, width=15).pack(pady=10)

        # Calcule le délai pour respecter le temps d'affichage minimal du splash screen
        wait_ms = get_splash_wait_time(min_duration_sec=1.0)
        
        def handle_splash_closing():
            """Révèle la fenêtre principale, lui donne le focus, puis ferme le splash screen."""
            root.deiconify()
            apply_window_focus(root)
            close_splash_screen()

        # Programme l'action synchronisée dans la file d'attente Tkinter
        root.after(wait_ms, handle_splash_closing)

        root.mainloop()

    except Exception as gui_err:
        # ====================================================================
        # SYSTÈME DE REPLI (FALLBACK) EN CAS D'ÉCHEC DE TKINTER
        # ====================================================================
        close_splash_screen()

        print("[AVERTISSEMENT] Tkinter n'a pas pu s'initialiser ou s'exécuter.")
        print(env_report)
        print(f"Détail de l'erreur GUI : {gui_err}")

        # 1. Si le terminal est disponible, on y affiche le rapport
        if has_console or sys.stdout:
            try:
                input("\nAppuyez sur Entrée pour quitter...")
                return
            except:
                pass

        # 2. Sinon, on bascule sur un pop-up Windows natif
        try:
            error_title = "Rapport Environnement - Mode Secours"
            ctypes.windll.user32.MessageBoxW(0, env_report + f"\n\nErreur GUI : {str(gui_err)}", error_title, 0x40)
        except:
            pass

if __name__ == "__main__":
    main()
```

---

<div id="section-11"></div>

## 11. Organisation recommandée de l’application portable

Une structure claire facilite la maintenance et la distribution.

Après toutes les étapes, la structure complète peut ressembler à :

```text
MonProjet\
│
├── .vscode\
│   └── settings.json
│
├── python_env\
│   ├── python.exe
│   ├── pythonw.exe
│   ├── python311._pth
│   ├── python311.dll
│   ├── python311.zip
│   ├── _tkinter.pyd
│   ├── tcl86t.dll
│   ├── tk86t.dll
│   ├── tcl\
│   │   ├── tcl8.6\
│   │   └── tk8.6\
│   ├── Scripts\
│   └── Lib\
│       └── site-packages\
│           └── tkinter\
│
├── src\
│   └── main.py
│
├── assets\
│   ├── app.ico
│   └── splash.png
│
├── Lancer_App.(bat ou exe)
├── Debug.(bat ou exe)
└── Installer_Libs.bat
```

Séparer :

- **moteur Python** (`python_env\`) ;
- **code source** (`src\`) ;
- **ressources** (images, configs, etc.) ;
- **scripts de lancement** (`.(bat ou exe)`) ;
- **outils de développement** (`.vscode\`).

---

<div id="section-12"></div>

### 12. Résultat et philosophie générale

Votre environnement Python est désormais :

- **entièrement portable** ;
- **indépendant de Windows** ;
- **compatible avec VS Code** ;
- **capable d’installer des bibliothèques via Pip** ;
- **compatible avec Tkinter** ;
- **facilement distribuable** (dossier ou archive ZIP).

#### Préparation de la distribution (Optionnel mais recommandé)
Avant de compresser votre dossier `MonProjet\` pour l'envoyer à un utilisateur final, il est conseillé d'effectuer un petit "nettoyage de staging" en supprimant les éléments superflus qui n'ont pas à être transmis :
- Le dossier des logs (`logs\`), qui ne sert qu'au diagnostic et au débogage.
- Le dossier de configuration de développement (`.vscode\`).
- Le code source du lanceur (`Launcher.cs`) et le script de compilation (`build_launcher.bat`) si vous ne souhaitez distribuer que l'exécutable final.

Pour utiliser l’application sur un autre PC Windows compatible :

1. Copier le dossier `MonProjet\` (nettoyé) ou une archive ZIP décompressée.
2. Lancer `Lancer_App.(bat ou exe)` ou `Debug.(bat ou exe)`.
3. Aucune installation de Python n’est nécessaire.

### Philosophie finale

> Créer des applications Python portables, facilement modifiables et distribuables, où les fichiers restent accessibles (code, configurations, ressources), contrairement à une application compilée qui nécessite souvent une recompilation après chaque modification.

Tu obtiens ainsi une solution intermédiaire :

- plus flexible ;
- plus simple à maintenir ;
- plus légère ;
- plus facile à personnaliser ;
- tout en gardant un fonctionnement proche d’une application Windows classique.

---

## 📜 Licence et Droits d'utilisation

Ce guide (ainsi que les codes fournis à l'intérieur) fait partie du projet **PyLibrePortable**.  
Il est distribué sous termes de la **GNU General Public License v3.0 (GPLv3)**.

* **Vous êtes libres de :** Partager, utiliser et modifier ce guide et ses codes.
* **À condition de :** Maintenir la même ouverture (open source) et de créditer l'auteur original.

*© PyLibrePortable - Tous droits réservés sous licence GPLv3.*

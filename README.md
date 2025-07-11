# Compilation et Installation de GNU nano 8.5 sur macOS Tahoe bêta 2 (Intel)

Ce guide explique en détail comment installer ou compiler l’éditeur de texte GNU
nano 8.5 sur un Mac Intel équipé de macOS Tahoe (bêta 2). Il couvre à la fois
l’installation via Homebrew (recommandée) et la compilation manuelle, en tenant
compte des spécificités et précautions liées à l’utilisation d’une version bêta
de macOS.

---

## Sommaire

- [Contexte et avertissements](#contexte-et-avertissements)
- [Méthode 1 : Installation via Homebrew](#méthode-1--installation-via-homebrew)
- [Méthode 2 : Compilation manuelle depuis le code source](#méthode-2--compilation-manuelle-depuis-le-code-source)
- [Configuration de la coloration syntaxique](#configuration-de-la-coloration-syntaxique)
- [Dépannage et conseils spécifiques à macOS Tahoe (bêta 2)](#dépannage-et-conseils-spécifiques-à-macos-tahoe-bêta-2)
- [Résumé des commandes principales](#résumé-des-commandes-principales)
- [Note sur la détection automatique des bibliothèques Homebrew](#note-sur-la-détection-automatique-des-bibliothèques-homebrew)
- [Ressources utiles](#ressources-utiles)

---

## Contexte et avertissements

- **macOS Tahoe (bêta 2)** est une version très récente et potentiellement
  instable du système. Certains outils ou dépendances peuvent ne pas être
  totalement compatibles.
- **Sauvegardez vos données** avant toute manipulation sur une version bêta.
- **Utilisez la version bêta des outils de développement Xcode** (Command Line
  Tools) adaptée à Tahoe, téléchargeable sur le portail développeur Apple.
- **Les chemins de bibliothèques ou la détection automatique peuvent différer**
  par rapport à des versions stables de macOS.

---

## Méthode 1 : Installation via Homebrew

**Homebrew** est le gestionnaire de paquets le plus populaire sur macOS. Il
simplifie l’installation de logiciels et la gestion des dépendances.

### Étapes pour l'installation via Homebrew

1. **Installer ou mettre à jour les outils de développement Xcode**

   ```sh
   xcode-select --install
   ```

   > Vérifiez que la version installée correspond bien à macOS Tahoe (bêta 2).

2. **Installer ou mettre à jour Homebrew**

   ```sh
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```

3. **Diagnostiquer la compatibilité Homebrew**

   ```sh
   brew doctor
   ```

   > Corrigez les éventuels avertissements signalés.

4. **Installer nano**

   ```sh
   brew update
   brew install nano
   ```

5. **Vérifier l’installation**

   ```sh
   which nano
   nano --version
   ```

   > Le chemin doit pointer vers `/usr/local/bin/nano` (pour Mac Intel) et la
   > version affichée doit être 8.5 ou ultérieure.

---

## Méthode 2 : Compilation manuelle depuis le code source

À utiliser si Homebrew échoue ou si vous souhaitez contrôler précisément les
options de compilation.

### Étapes pour la compilation manuelle

1. **Installer les outils de développement Xcode**
   (voir Méthode 1, étape 1)

2. **Installer les dépendances nécessaires**

   ```sh
   brew install ncurses gettext libmagic
   ```

   - `ncurses` : gestion de l’interface terminal
   - `gettext` : internationalisation
   - `libmagic` : détection du type de fichier (optionnelle mais recommandée
     pour la coloration syntaxique avancée)

3. **Télécharger et extraire le code source**

   ```sh
   cd ~/Downloads
   curl -O https://www.nano-editor.org/dist/v8/nano-8.5.tar.xz
   tar -xf nano-8.5.tar.xz
   cd nano-8.5
   ```

4. **Configurer la compilation**

   Avant de lancer la configuration, exportez ces variables d’environnement pour
   garantir le support UTF-8 et la bonne détection des dépendances :

- **Tentative initiale avec exports préliminaires :**

     ```sh
     export CPPFLAGS="-I/usr/local/opt/ncurses/include"
     export LDFLAGS="-L/usr/local/opt/ncurses/lib"
     export PKG_CONFIG_PATH="/usr/local/opt/ncurses/lib/pkgconfig"
     ./configure --enable-utf8 --enable-libmagic
     ```

     > Astuce : Ces variables forcent l’utilisation de la version Homebrew de
     > ncurses, indispensable pour le support Unicode sur macOS.

- **Si une dépendance n’est pas trouvée (ex : "ncurses not found") :**

    ```sh
    export LDFLAGS="-L/usr/local/opt/ncurses/lib -L/usr/local/opt/gettext/lib -L/usr/local/opt/libmagic/lib"
    export CPPFLAGS="-I/usr/local/opt/ncurses/include -I/usr/local/opt/gettext/include -I/usr/local/opt/libmagic/include"
    ./configure --enable-utf8 --enable-libmagic
    ```

     > Adaptez les chemins si Homebrew est installé ailleurs.
<!-- markdownlint-disable-next-line MD029 -->
5. **Compiler et installer**

   ```sh
   make
   make check
   sudo make install
   ```
<!-- markdownlint-disable-next-line MD029 -->
6. **Vérifier l’installation**
   (voir Méthode 1, étape 5)

> **Note sur l’option `--with-magic` et la bibliothèque `libmagic` :**
>
> - **Le nom correct de l'option** : L'option pour le script `configure` de nano
>   n'est pas `--with-magic`, mais bien `--enable-libmagic`.
> - **À quoi sert cette option ?** Cette option permet à nano d'utiliser la
>   bibliothèque *libmagic*. Il s'agit de la bibliothèque qui se cache derrière
>   la commande `file` de macOS et Linux, capable d'identifier le type d'un
>   fichier (par exemple, un script Python, un document XML, un fichier C++) en
>   analysant son contenu, et non pas seulement son extension.  
>   Pour nano, l'avantage est considérable : si vous ouvrez un fichier sans
>   extension ou avec une extension incorrecte, nano peut quand même détecter
>   automatiquement son type et appliquer la coloration syntaxique appropriée.
> - **Le nom du paquet à installer** : Le paquet correspondant à cette
>   bibliothèque s'appelle bien `libmagic`. Vous devez donc l'installer avant de
>   lancer la compilation.

---

## Configuration de la coloration syntaxique

Pour bénéficier de la coloration syntaxique avancée (notamment grâce à
libmagic), ajoutez à votre fichier `~/.nanorc` :

```text
include "/usr/local/share/nano/*.nanorc"
```

> Si nano a été installé ailleurs, adaptez le chemin en conséquence.

---

## Dépannage et conseils spécifiques à macOS Tahoe (bêta 2)

- **En cas d’échec de compilation ou d’installation d’une dépendance**,
  consultez attentivement les messages d’erreur : ils vous indiqueront si un
  chemin doit être précisé ou si une bibliothèque manque.
- **Pour libmagic**, si nano ne détecte pas la bibliothèque, vérifiez que le
  fichier `libmagic.dylib` est bien accessible dans `/usr/local/lib` (Intel). Si
  besoin, créez un lien symbolique :

  ```sh
  sudo ln -s /usr/local/opt/libmagic/lib/libmagic.dylib /usr/local/lib/libmagic.dylib
  ```

- **Gardez à l’esprit que la stabilité des outils Homebrew ou Xcode peut varier
  sur une bêta.** Consultez régulièrement les issues sur GitHub ou les forums
  pour des solutions communautaires récentes.

---

## Résumé des commandes principales

| Étape                           | Commande principale                                                                 |
|---------------------------------|-------------------------------------------------------------------------------------|
| Installer Xcode CLT             | `xcode-select --install`                                                            |
| Installer Homebrew              | `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"` |
| Diagnostiquer Homebrew          | `brew doctor`                                                                       |
| Installer nano (Homebrew)       | `brew install nano`                                                                 |
| Installer dépendances (manuel)  | `brew install ncurses gettext libmagic`                                             |
| Télécharger nano                | `curl -O https://www.nano-editor.org/dist/v8/nano-8.5.tar.xz`                       |
| Configurer (manuel)             | `./configure --enable-utf8 --enable-libmagic`                                            |
| Compiler et installer           | `make && sudo make install`                                                         |

---

## Note sur la détection automatique des bibliothèques Homebrew

> **Note importante :**
>
> Sur macOS avec Homebrew, il n’est pas toujours indispensable de spécifier
> manuellement les variables d’environnement `LDFLAGS` et `CPPFLAGS` lors de la
> compilation d’un programme utilisant une bibliothèque comme libmagic. Homebrew
> configure généralement les chemins nécessaires automatiquement.
>
> Cependant, il arrive que le script de configuration (`./configure`) ne détecte
> pas correctement les bibliothèques installées par Homebrew, notamment si le
> paquet est "keg-only" ou si le script ne recherche pas dans les chemins
> Homebrew par défaut. Définir explicitement :
>
> ```sh
> export LDFLAGS="-L/usr/local/opt/libmagic/lib"
> export CPPFLAGS="-I/usr/local/opt/libmagic/include"
> ```
>
> avant d’exécuter `./configure` garantit que la compilation prendra bien en
> compte les bibliothèques Homebrew, en particulier dans des environnements non
> standards ou sur des versions récentes/bêtas de macOS.

---

## Ressources utiles

- [GNU nano – site officiel](https://www.nano-editor.org/)
- [Formule Homebrew nano](https://formulae.brew.sh/formula/nano)
- [Homebrew – site officiel](https://brew.sh/)
- [Apple Developer – Xcode](https://developer.apple.com/xcode/)

---

**Bonne installation et bonne édition avec nano sur macOS Tahoe !**

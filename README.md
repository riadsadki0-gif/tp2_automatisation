# **Automatisation des tâches d’administration**
    
   ## *TP No 2 - Shell Bash*
### EXERCICE 1 - Itération, chaînes de caractères, expressions

## EXERCICE 1 — Itération, chaînes de caractères, expressions

### Objectif

Renommer automatiquement des fichiers dont le nom est :

- dcp_1234.jpg
- DCP_1234.JPG
- etc.

Pour obtenir :

photo_1234.jpg 
photo_5678.jpg
ect


### Script utilisé

```bash
#!/bin/bash

for file in *.jpg *.JPG
do
    [ -e "$file" ] || continue

    number=$(echo "$file" | sed -E 's/.*_([0-9]+)\..*/\1/')
    newname="photo_${number}.jpg"

    mv "$file" "$newname"
done
```


## EXERCICE 2 — Lister les utilisateurs

### Objectif
Afficher la liste des logins définis dans `/etc/passwd` ayant un UID strictement supérieur à 100.

---

### Première tentative avec `cat`

```bash
for ligne in $(cat /etc/passwd)
do
  echo "$ligne"
done

**Problème** :
La substitution $(cat /etc/passwd) découpe le contenu selon les espaces et retours à la ligne.
On ne traite donc pas correctement les champs du fichier.

### Solution retenue avec `cut`

Le fichier `/etc/passwd` est structuré ainsi :

login:x:UID:GID:commentaire:home:shell

Les champs sont séparés par `:`.  
Le login correspond au champ 1 et l’UID au champ 3.
```
---

### Script final

```bash
#!/bin/bash

cut -d: -f1,3 /etc/passwd | while IFS=: read login uid
do
  [ "$uid" -gt 100 ] && echo "$login"
done
```
### Conclusion

L’utilisation de cat n’est pas adaptée pour traiter correctement un fichier structuré.
La combinaison de cut et while read permet une lecture fiable et un filtrage précis des utilisateurs.

## EXERCICE 3 — Lecture au clavier

### 1 — Que fait la commande `read` ?

La commande `read` permet de lire une chaîne de caractères saisie au clavier et de l’affecter à une variable.

#### Test réalisé

```bash
echo -n "Entrer votre nom: "
read nom
echo "Votre nom est $nom"
```

Lorsque l’utilisateur saisit une valeur puis appuie sur Entrée, celle-ci est stockée dans la variable nom.
La commande suivante permet ensuite d’afficher le contenu de cette variable.

### Conclusion :
read lit une entrée standard (clavier) et stocke la valeur dans une variable pour un traitement ultérieur.

### 2 — Que fait la commande `file` ?

La commande `file` permet d’identifier le type d’un fichier en analysant son contenu réel, et non uniquement son extension.

#### Tests réalisés

```bash
file /etc/passwd
file liste_users.sh
file /bin/ls
```

#### Résultats observés

/etc/passwd: fichier texte (ASCII text ou UTF-8 text)

liste_users.sh: script shell (shell script)

/bin/ls: fichier binaire exécutable (ELF 64-bit executable)

### 3 — Que fait la commande `less` ?

La commande `less` permet d’afficher le contenu d’un fichier texte page par page dans le terminal.  
Elle offre une navigation interactive à l’intérieur du fichier.

#### Test réalisé

```bash
less /etc/passwd
```

#### Commandes principales

- Quitter `less` :  
  `q`

- Avancer d’une ligne :  
  `Enter` ou flèche ↓

- Avancer d’une page :  
  `Space` ou `PageDown`

- Remonter d’une page :  
  `b` ou `PageUp`

- Rechercher une chaîne de caractères :  
  `/mot` puis `Enter`

- Passer à l’occurrence suivante :  
  `n`  
  (retour à l’occurrence précédente : `N`)

#### Conclusion

La commande `less` permet de consulter efficacement un fichier texte volumineux grâce à une navigation interactive et à des fonctions de recherche intégrées.

### 4 — Qu’appelle-t-on un fichier texte ? Comment les repérer avec `file` ?

### 4 — Qu’appelle-t-on un fichier texte ? Comment les repérer avec `file` ?

Un fichier texte est un fichier composé uniquement de caractères lisibles (lettres, chiffres, symboles), encodés en ASCII ou en UTF-8.  
Il peut être affiché directement dans le terminal avec des commandes comme `cat`, `more` ou `less`.

Un fichier binaire, en revanche, contient des données non lisibles directement (exécutables, images, archives, etc.).

---

### Tests réalisés

```bash
file /etc/passwd
```

Résultat observé :

```
/etc/passwd: ASCII text
```
ou
```
/etc/passwd: UTF-8 Unicode text
```

---

```bash
file liste_users.sh
```

Résultat observé :

```
liste_users.sh: Bourne-Again shell script, ASCII text executable
```

---

```bash
file /bin/ls
```

Résultat observé :

```
/bin/ls: ELF 64-bit LSB executable
```

---

### Analyse

Les fichiers `/etc/passwd` et `liste_users.sh` sont identifiés comme des fichiers texte car la sortie contient :

- `ASCII text`
- `UTF-8 Unicode text`
- `text`

En revanche, `/bin/ls` est identifié comme un exécutable ELF, donc un fichier binaire.

---

### Conclusion

La commande `file` permet de repérer les fichiers texte en recherchant dans sa sortie les mentions liées au texte (`ASCII text`, `UTF-8 Unicode text`, etc.).  
Si ces indications apparaissent, il s’agit d’un fichier texte ; sinon, il s’agit d’un fichier binaire.

### 5 — Script affichant uniquement les fichiers texte d’un répertoire

L’objectif est d’écrire un script qui affiche les noms de tous les fichiers texte (et uniquement ceux-ci) d’un répertoire passé en argument.

Un fichier texte peut être identifié à l’aide de la commande `file`, qui retourne des mentions comme :

- `ASCII text`
- `UTF-8 Unicode text`
- `text executable`

---

### Script réalisé

```bash
#!/bin/bash

# Vérification de l’argument
if [ -z "$1" ] || [ ! -d "$1" ]; then
    echo "Usage: $0 <repertoire>"
    exit 1
fi

# Parcours des fichiers du répertoire
for fichier in "$1"/*
do
    if [ -f "$fichier" ]; then
        if file "$fichier" | grep -qi "text"; then
            echo "$(basename "$fichier")"
        fi
    fi
done
```

---

### Test réalisé

```bash
./liste_textes.sh .
```

#### Résultat observé

```
liste_textes.sh
liste_users.sh
rename.sh
```

---

### Vérification complémentaire

```bash
file *
```

Résultat observé :

- `liste_textes.sh`: Bourne-Again shell script, UTF-8 Unicode text executable  
- `liste_users.sh`: Bourne-Again shell script, ASCII text executable  
- `rename.sh` : Bourne-Again shell script, UTF-8 Unicode text executable  
- `photo_*.jpg`: empty  

Les fichiers `.sh` sont bien identifiés comme fichiers texte, tandis que les fichiers `.jpg` ne le sont pas.

---

### Conclusion

Le script :

- prend un répertoire en argument,
- parcourt tous les fichiers,
- utilise `file` pour déterminer leur type,
- affiche uniquement ceux contenant la mention `text`.

Il permet donc de filtrer correctement les fichiers texte d’un répertoire donné.


### 6 — Script proposant à l’utilisateur de visualiser page par page chaque fichier texte

L’objectif de cet exercice est d’écrire un script qui propose à l’utilisateur de visualiser page par page chaque fichier texte d’un répertoire donné en argument. Le script affichera pour chaque fichier la question : “Voulez-vous visualiser le fichier machintruc ?”. En cas de réponse positive, il lancera `less`, avant de passer à l'examen du fichier suivant.

---

### Script réalisé

```bash
#!/bin/bash

# Vérification de l'argument
if [ -z "$1" ] || [ ! -d "$1" ]; then
    echo "Usage: $0 <repertoire>"
    exit 1
fi

# Parcours des fichiers du répertoire
for fichier in "$1"/*
do
    if [ -f "$fichier" ]; then
        # Vérification si le fichier est un fichier texte
        if file "$fichier" | grep -qi "text"; then
            echo -n "Voulez-vous visualiser le fichier $(basename "$fichier") ? (o/n) : "
            read rep

            if [ "$rep" = "o" ] || [ "$rep" = "O" ]; then
                less "$fichier"
            fi
        fi
    fi
done
```

---

### Explication

1. Le script prend un répertoire en argument.
2. Il vérifie que cet argument est valide (un répertoire existant).
3. Il parcourt chaque fichier du répertoire.
4. Pour chaque fichier, il utilise la commande `file` pour vérifier s'il s'agit d'un fichier texte.
5. Si le fichier est un fichier texte, le script demande à l'utilisateur s'il souhaite le visualiser.
6. Si la réponse est positive (`o` ou `O`), le script utilise `less` pour afficher le fichier page par page.

---

### Rendre le script exécutable

```bash
chmod +x visualise_textes.sh
```

---

### Exécution

```bash
./visualise_textes.sh 
```

---

### Comportement observé

Si le répertoire contient des fichiers comme `liste_textes.sh` et `liste_users.sh`, le script affichera :

```
Voulez-vous visualiser le fichier liste_textes.sh ? (o/n) : o
```

Puis il ouvrira `liste_textes.sh` avec `less`.  
Ensuite, il passera à `liste_users.sh` et répétera la même question.

---

### Conclusion

Le script permet de visualiser page par page chaque fichier texte dans un répertoire donné, en sollicitant l'avis de l'utilisateur avant d'afficher chaque fichier avec `less`.

## EXERCICE 4 — Créer un script `test-fichier`

### Objectif

Écrire un script `test-fichier` qui affiche :

1. La **nature** du fichier passé en paramètre (fichier ordinaire ou répertoire).
2. Les **permissions d’accès** pour l’utilisateur qui lance le script.

Le script doit utiliser des conditions (`if/then/else/elif`), des opérateurs de test du shell (`-d`, `-f`, etc.) et les commandes de base (`ls`, `cut`, ...).

---

### Script réalisé

```bash
#!/bin/bash

# Vérification de l'argument
if [ -z "$1" ]; then
    echo "Usage: $0 <fichier>"
    exit 1
fi

echo "Fichier passé en argument : $1"

# Vérification si c'est un répertoire
if [ -d "$1" ]; then
    echo "Le fichier $1 est un répertoire"
    permissions=$(ls -ld "$1" | cut -d ' ' -f1)
else
    # Vérification si c'est un fichier ordinaire
    if [ -f "$1" ]; then
        echo "Le fichier $1 est un fichier ordinaire"
        permissions=$(ls -l "$1" | cut -d ' ' -f1)
        if [ ! -s "$1" ]; then
            echo "Le fichier est vide"
        else
            echo "Le fichier n'est pas vide"
        fi
    else
        echo "$1 n'est ni un fichier ordinaire ni un répertoire"
        exit 1
    fi
fi

# Affichage des permissions pour l'utilisateur actuel
echo "\"$1\" est accessible par $(whoami) avec les permissions : $permissions"
```

---

### Tests réalisés

#### Exemple 1 : Script lancé pour un répertoire

```bash
$ ./test-fichier /etc
Le fichier /etc est un répertoire
"/etc" est accessible par riad avec les permissions : drwxr-xr-x
```
**Résultat affiché** : Le fichier `/etc` est un répertoire, accessible par l'utilisateur `riad` avec les permissions `drwxr-xr-x`.

---
#### Exemple 2 : Script lancé pour un fichier ordinaire

```bash
$ ./test-fichier /etc/smb.conf
/etc/smb.conf n'est ni un fichier ordinaire ni un répertoire
```

**Résultat affiché** : Le fichier `/etc/smb.conf` est un fichier ordinaire, il n'est pas vide et est accessible par l'utilisateur `riad` avec les permissions `-rw-r--r--`.

---

### Conclusion

Le script permet d'afficher la nature d’un fichier (répertoire ou fichier ordinaire) et ses permissions d’accès pour l’utilisateur qui lance le script. Il utilise des conditions et des opérateurs de test pour déterminer si le fichier est un répertoire, un fichier ordinaire, ou autre, et affiche ensuite les permissions d’accès.


## EXERCICE 6 — Afficher le contenu d’un répertoire

### Objectif

Écrire un script `listdir.sh` permettant d’afficher le contenu d’un répertoire passé en argument. Ensuite, améliorer ce script pour séparer l’affichage des fichiers et des répertoires.

---

### Script réalisé

```bash
#!/bin/bash

# Vérification de l'argument
if [ -z "$1" ]; then
    echo "Usage: $0 <repertoire>"
    exit 1
fi

# Affichage du contenu du répertoire
echo "-------------- Contenu de $1 --------------------"
echo ""

# Affichage des fichiers
echo "-------------- Fichiers dans $1 --------------------"
find "$1" -maxdepth 1 -type f | cut -d/ -f5- | sort

# Affichage des répertoires
echo ""
echo "-------------- Repertoires dans $1 --------------------"
find "$1" -maxdepth 1 -type d | cut -d/ -f5- | sort
```

---

### Explication du script

1. **Vérification de l'argument** :  
   Le script prend un répertoire en argument. Si aucun argument n'est fourni, il affiche un message d'usage et termine l'exécution.

2. **Affichage du contenu du répertoire** :  
   Le script utilise `ls "$1"` pour afficher le contenu du répertoire, mais pour séparer les fichiers et les répertoires, nous utilisons la commande `find` pour trouver respectivement les fichiers et répertoires.

3. **Séparation des fichiers et répertoires** :  
   La commande `find "$1" -maxdepth 1 -type f` affiche uniquement les fichiers dans le répertoire.
   La commande `find "$1" -maxdepth 1 -type d` affiche uniquement les répertoires dans le répertoire.

---

### Exemple d'exécution

#### Exécution avec `/etc/`

```bash
$ ./listdir.sh /etc
-------------- Contenu de /etc --------------------



-------------- Fichiers dans /etc --------------------
alternatives
apache2
apt
bash_completion.d
...

-------------- Repertoires dans /etc --------------------
init.d
rc0.d
rc1.d
rc2.d
...
```

---

### Conclusion

Le script `listdir.sh` permet d'afficher le contenu d'un répertoire passé en argument, en séparant les fichiers et répertoires. Il trie les résultats et les affiche de manière propre pour faciliter la consultation du contenu d’un répertoire.

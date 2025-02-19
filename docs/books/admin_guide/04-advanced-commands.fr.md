---
title: Commandes avancées Linux
---

# Commandes avancées pour utilisateurs Linux

Dans ce chapitre, vous apprendrez quelques commandes avancées pour Linux.

****

**Objectifs** : Dans ce chapitre, les futurs administrateurs Linux vont apprendre :

:heavy_check_mark: quelques commandes utiles non couvertes dans le chapitre précédent ;   
:heavy_check_mark: quelques commandes avancées.

:checkered_flag: **commandes utilisateurs**, **Linux**

**Connaissances : ** :star:   
**Complexité : ** :star: :star: :star:

**Temps de lecture : **20 minutes

****

## La commande `uniqu`

La commande `uniq` est une commande, utilisée avec la commande `sort`, très puissante, notamment pour l’analyse de fichiers de logs. Elle permet de trier et d’afficher des entrées en supprimant les doublons.

Pour illustre le fonctionnement de la commande `uniq`, utilisons un fichier `firstnames.txt` contenant une liste de prénoms :

```
antoine
xavier
steven
patrick
xavier
antoine
antoine
steven
```

!!! Note

    `uniq` nécessite que le fichier d'entrée soit trié car il ne compare que les lignes consécutives.

Sans argument, la commande `uniq` ne va pas afficher les lignes identiques qui se suivent du fichier `firstnames.txt` :

```
$ sort firstnames.txt | uniq
antoine
patrick
steven
xavier
```

Pour n’afficher que les lignes n’apparaissant qu’une seule fois, il faut utiliser l’option `-u` :

```
$ sort firstnames.txt | uniq -u
patrick
```

À l’inverse, pour n’afficher que les lignes apparaissant au moins deux fois dans le fichier, il faut utiliser l’option `-d` :

```
$ sort firstnames.txt | uniq -d
antoine
steven
xavier
```

Pour simplement supprimer les lignes qui n’apparaissent qu’une seule fois, il faut utiliser l’option `-D` :

```
$ sort firstnames.txt | uniq -D
antoine
antoine
antoine
steven
steven
xavier
xavier
```

Enfin, pour compter le nombre d’occurrences de chaque ligne, il faut utiliser l’option `-c` :

```
$ sort firstnames.txt | uniq -c
      3 antoine
      1 patrick
      2 steven
      2 xavier
```

```
$ sort firstnames.txt | uniq -cd
      3 antoine
      2 steven
      2 xavier
```

## La commande `xargs`

La commande `xargs` permet la construction et l’exécution de lignes de commandes à partir de l’entrée standard.

La commande `xargs` lit des arguments délimités par des blancs ou par des sauts de ligne depuis l’entrée standard, et exécute une ou plusieurs fois la commande (`/bin/echo` par défaut) en utilisant les arguments initiaux suivis des arguments lus depuis l’entrée standard.

Un premier exemple le plus simple possible serait le suivant :

```
$ xargs
use
of
xargs
<CTRL+D>
use of xargs
```

La commande `xargs` attend une saisie depuis l’entrée standard **stdin**. Trois lignes sont saisies. La fin de la saisie utilisateur est spécifiée à `xargs` par la séquence de touches <kbd>CTRL</kbd> + <kbd>D</kbd>. `xargs` exécute alors la commande par défaut `echo` suivi des trois arguments correspondants à la saisie utilisateur, soit :

```
$ echo "use" "of" "xargs"
use of xargs
```

Il est possible de spécifier une commande à lancer par `xargs`.

Dans l’exemple qui suit, `xargs` va exécuter la commande `ls -ld` sur l’ensemble des dossiers qui seront spécifiés depuis l’entrée standard :

```
$ xargs ls -ld
/home
/tmp
/root
<CTRL+D>
drwxr-xr-x. 9 root root 4096  5 avril 11:10 /home
dr-xr-x---. 2 root root 4096  5 avril 15:52 /root
drwxrwxrwt. 3 root root 4096  6 avril 10:25 /tmp
```

En pratique, la commande `xargs` a exécuté la commande `ls -ld /home /tmp /root`.

Que se passe-t-il si la commande à exécuter n’accepte pas plusieurs arguments comme c’est le cas pour la commande `find` ?

```
$ xargs find /var/log -name
*.old
*.log
find: paths must precede expression: *.log
```

La commande `xargs` a tenté d’exécuter la commande `find` avec plusieurs arguments derrière l’option `-name`, ce qui fait généré par `find` une erreur :

```
$ find /var/log -name "*.old" "*.log"
find: paths must precede expression: *.log
```

Dans ce cas, il faut forcer la commande `xargs` à exécuter plusieurs fois (une fois par ligne saisie en entrée standard) la commande `find`. L’option `-L `suivie d’un nombre **entier** permet de spécifier le nombre maximal d’entrées à traiter avec la commande en une seule fois :

```
$ xargs -L 1 find /var/log -name
*.old
/var/log/dmesg.old
*.log
/var/log/boot.log
/var/log/anaconda.yum.log
/var/log/anaconda.storage.log
/var/log/anaconda.log
/var/log/yum.log
/var/log/audit/audit.log
/var/log/anaconda.ifcfg.log
/var/log/dracut.log
/var/log/anaconda.program.log
<CTRL+D>
```

Si nous avions voulu pouvoir spécifier sur la même ligne les deux arguments, il aurait fallut utiliser l’option `-n 1` :

```
$ xargs -n 1 find /var/log -name
*.old *.log
/var/log/dmesg.old
/var/log/boot.log
/var/log/anaconda.yum.log
/var/log/anaconda.storage.log
/var/log/anaconda.log
/var/log/yum.log
/var/log/audit/audit.log
/var/log/anaconda.ifcfg.log
/var/log/dracut.log
/var/log/anaconda.program.log
<CTRL+D>
```

Cas concret d’une sauvegarde avec un `tar` en fonction d’une recherche :

```
$ find /var/log/ -name "*.log" -mtime -1 | xargs tar cvfP /root/log.tar
$ tar tvfP /root/log.tar
-rw-r--r-- root/root      1720 2017-04-05 15:43 /var/log/boot.log
-rw-r--r-- root/root    499270 2017-04-06 11:01 /var/log/audit/audit.log
```

La particularité de la commande `xargs` est quelle place l’argument en entrée à la fin de la commande appelée. Ceci fonctionne très bien avec l’exemple ci-dessus puisque les fichiers passés en entrée vont constituer la liste des fichiers à ajouter à l’archive.

Maintenant, si nous prenons l’exemple de la commande `cp` en voulant copier une liste de fichiers dans un répertoire, cette liste de fichiers sera ajoutée en fin de commande… or ce qui est attendu par la commande `cp` en fin de commande est plutôt la destination. Pour ce faire, nous utilisons l’option `-I` afin de placer les arguments en entrée ailleurs qu’en fin de ligne.

```
$ find /var/log -type f -name "*.log" | xargs -I % cp % /root/backup
```

L’option `-I` permet de spécifier un caractère (dans notre exemple le caractère `%`) où seront placés les fichiers en entrée de `xargs`.

## Le paquet `yum-utils`

Le paquet `yum-utils` est une collection d’utilitaires de différents auteurs pour `yum`, qui le rendent plus simple et plus puissant à utiliser.

!!! Note

    Bien que `yum` ait été remplacé par `dnf` dans Rocky Linux 8, le nom du paquet est resté `yum-utils` mais il peut tout aussi bien être installé avec le paquet `dnf-utils`. Ce sont des utilitaires YUM classiques implémentés comme des CLI au-dessus de DNF pour maintenir une compatibilité ascendante avec `yum-3`.

Voici quelques exemples d’utilisation :

* la commande `repoquery` :

La commande `repoquery` interroge les dépôts.

Exemples d’utilisation :

  * Connaître les dépendances d’un paquet non-installé :

```
repoquery --requires <package>
```

  * Connaître les fichiers fournis par un paquet non-installé :

```
$ repoquery -l yum-utils
/etc/bash_completion.d
/etc/bash_completion.d/yum-utils.bash
/usr/bin/debuginfo-install
/usr/bin/find-repos-of-install
/usr/bin/needs-restarting
/usr/bin/package-cleanup
/usr/bin/repo-graph
/usr/bin/repo-rss
/usr/bin/repoclosure
/usr/bin/repodiff
/usr/bin/repomanage
/usr/bin/repoquery
/usr/bin/reposync
/usr/bin/repotrack
/usr/bin/show-changed-rco
/usr/bin/show-installed
/usr/bin/verifytree
/usr/bin/yum-builddep
/usr/bin/yum-config-manager
/usr/bin/yum-debug-dump
/usr/bin/yum-debug-restore
/usr/bin/yum-groups-manager
/usr/bin/yumdownloader
…
```

* La commande `yumdownloader` :

La commande `yumdownloader` télécharge les paquets RPM depuis les dépôts.

!!! Note

    Cette commande est très pratique pour construire un dépôt local de quelques rpm !

Exemple : `yumdownloader` va télécharger le paquet rpm de _repoquery_ ainsi que toutes ses dépendances :

```
$ yumdownloader --destdir /var/tmp -- resolve repoquery
```

| Options     | Commentaires                                                       |
| ----------- | ------------------------------------------------------------------ |
| -`-destdir` | Les paquets téléchargés seront conservés dans le dossier spécifié. |
| `--resolve` | Télécharge également les dépendances du paquet.                    |

## Le paquet `psmisc`

Le paquet `psmisc` contient des utilitaires pour gérer les processus du système :

* `pstree` : la commande `pstree` affiche les processus en cours sur le système sous forme de structure en forme d’arbre.
* `killall` : la commande `killall` envoie un signal d’extinction à tous les processus identifiés par un nom.
* `fuser` : la commande `fuser` identifie les `PID` des processus qui utilisent les fichiers ou les systèmes de fichiers spécifiés.

Exemples :

```
$ pstree
systemd─┬─NetworkManager───2*[{NetworkManager}]
        ├─agetty
        ├─auditd───{auditd}
        ├─crond
        ├─dbus-daemon───{dbus-daemon}
        ├─firewalld───{firewalld}
        ├─lvmetad
        ├─master─┬─pickup
        │        └─qmgr
        ├─polkitd───5*[{polkitd}]
        ├─rsyslogd───2*[{rsyslogd}]
        ├─sshd───sshd───bash───pstree
        ├─systemd-journal
        ├─systemd-logind
        ├─systemd-udevd
        └─tuned───4*[{tuned}]
```

```
# killall httpd
```

Tue les processus (option `-k`) qui accèdent au fichier `/etc/httpd/conf/httpd.conf` :

```
# fuser -k /etc/httpd/conf/httpd.conf
```

## La commande `watch`

La commande `watch` exécute régulièrement une commande et affiche le résultat dans le terminal en plein écran.

L’option `-n` permet de spécifier le nombre de secondes entre chaque exécution de la commande.

!!! Note

    Pour quitter la commande `watch`, vous devez taper les clés : <kbd>CTRL</kbd>+<kbd>C</kbd> pour tuer le processus.

Exemples :

* Afficher la fin du fichier `/etc/passwd` toutes les 5 secondes :

```
$ watch -n 5 tail -n 3 /etc/passwd
```

Résultat :

```
Every 5,0s: tail -n 3 /etc/passwd                                                                                                                                rockstar.rockylinux.lan: Thu Jul  1 15:43:59 2021

sssd:x:996:993:User for sssd:/:/sbin/nologin
chrony:x:995:992::/var/lib/chrony:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
```

* Surveillance du nombre de fichier dans un dossier :

```
$ watch -n 1 'ls -l | wc -l'
```

* Afficher une horloge :

```
$ watch -t -n 1 date
```

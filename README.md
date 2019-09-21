# netiface
Objectif : Manipuler la configuration des interfaces réseaux pour Debian/Ubuntu/CentOS.
La configuration est stockée différemment entre CentOS et Debian.
L'idée est de proposer une Classe d'objet et des méthodes pour manipuler les interfaces.
La mécanique arrière se chargera alors d'écrire les fichiers de configuration convenablement en fonction de l'OS.

Aucune des librairies que j'ai testé actuellement n'est en mesure de me rendre ce service completement, même via network-manager.

----

1. Sous Debian, la configuration est dans un fichier unique /etc/network/interfaces
et/ou dans des fichiers séparés dans /etc/network/interfaces.d
L'appel se fait par la commande source dans /etc/network/interfaces qui appele tous les fichiers présents dans /etc/network/interfaces.d

2. Sous CentOS, la configuration est dans des fichiers séparés dans /etc/sysconfig/network-scripts/

----

Modèlisation :

Peu importe l'OS, une carte réseau présente des propriétés et des méthodes identiques :
- Un nom, par exemple eth0 ou dummy0 ou dummy0:0 (alias)
- un type : Ethernet, Bridge, Dummy, ...
- Une méthode de configuration : DHCP, ou manuel, ou static ou loopback
- Une ou plusieurs adresses IP avec un masque de sous réseau associé. On appliquera le format CIDR au masque.
- Une gateway
- Un ou plusieurs serveurs DNS
- Un domaine de recherche DNS
- Des commandes pour monter (up) ou descendre (down) l'interface
- Un attribut indiquant si l'interface est chargé au démarrage (onboot, auto, allow-hotplug)
- Eventuellement, on pourra ajouter une information concernant l'adresse MAC (on ne parle pas de spoofing, mais de lire la mac réel)
- Plein d'autres attributs qui peuvent être nécessaire sur les bridge, bond, vlan, ...
- Et des commandes, du type post-up ou post-down

Idée : Créer une définition d'objet "netiface" générique avec les attributs de base et la possibilité d'ajouter des attributs personnalisés. La dérivation de la définition de l'objet de base pourrait permettre de créer des définitions d'objets rendant un attribut obligatoire.

Transaction :

L'idée est que le modèle soit transactionnel. C'est à dire que l'on peut appliquer des modifications sans les sauvegarder.
L'idée est de créer un objet et de disposer de deux méthodes : Apply() et Save().

- Apply() applique la configuration demandée au système via IPRoute2 (on exploitera la library pyroute2) sans la sauvegarder dans les fichiers de configuration. Au redémarrage, on perdra cette configuration. 

- Save() écrira la configuration dans les fichiers. Cependant, on peut envisager que Save() ne soit pas possible si Apply() n'a pas été appelé avant. Cela pourrait permettre d'éviter de stocker une configuration non opérationnelle. Peut-être prévoir une variante pour éviter le contrôle.

Objets :

netiface represente un objet de définition d'une interface unique.
netmanager represente un objet qui contient toutes les interfaces du système (list d'objet netiface et methode global pour l'ajout, la suppression par exemple

netmanager doit contenir les méthodes suivantes :
  - Load(), recharge la configuration unique d'une interface (les modifications sont perdues)
  - LoadAll(), recharge toutes les configurations des interfaces (les modifications sont perdues)
  - SaveAll(), sauvegarde toutes les modifications sur toutes les interfaces
  - ApplyAll(), applique toutes les modifications.
  - Add(), ajoute une définition d'interface
  - Remove(), supprime une définition d'interface

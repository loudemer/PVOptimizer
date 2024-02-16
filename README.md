# PVOptimizer
Cette intégration permet de maximiser l’auto consommation de la production d’énergie de vos panneaux solaires. Elle permet de contrôler les gros appareils électroménagers, tels que lave-vaisselle ou lave-linge avec de simples switch, mais aussi des appareils plus complexes à gérer, tels que la filtration de piscine ou le contrôle de pompe à chaleur au travers d’applications communicantes dédiées nécessitant des mécanismes de contrôle multiparamétriques.

Elle est écrite en python sous appdaemon et devrait être facilement accessible à ceux qui ont des notions de programmation. Ce qui devrait permettre de la modifier facilement pour l’améliorer et l’adapter à ses propres besoins.

# Caractéristiques
- Prise en compte des abonnements : Tempo, Heures Creuses, Base
- Prise en compte du prix de revente EDF, OA
- Calcul du seuil de déclenchement des appareils en fonction du tarif heure creuse heure pleine et du prix de revente
- Programmation automatique des appareils qui n’ont pu être mise en route dans la journée en heure creuse la nuit.
- Peut fonctionner avec un routeur de ballon d’eau chaude, s’il est possible de récupérer la puissance instantanée, délivrée à ce dernier. L’adjonction d’un routeur est même recommandée.
- Permet de contrôler des systèmes complexes, type filtration de piscine, ou pompe à chaleur, au travers d’applications séparées qui dialoguent avec l’optimiseur.

# Prérequis

## Installation de l’add-on appdaemon
## Puissance solaire disponible 
Elle se mesure à l’aide d’une pince ampèremétrique branchée sur le câble d’alimentation de la maison. Cette mesure peut être positive en cas de prélèvement sur le réseau ou négative en cas d’exportation. Si vous disposez d’un routeur ECS, il faut pouvoir disposer de la puissance envoyée sur le ballon, afin d’avoir la puissance solaire disponible, exacte.
P disponible = P export + P ballon ECS.
## Couleur du jour pour les abonnements tempo
On peut utiliser l’API RTE[https://www.api-couleur-tempo.fr/api] par exemple. 

# Fonctionnement
Chaque appareil est décrit dans l’application par :
- son nom, 
- sa puissance, 
- sa durée de fonctionnement, 
- le switch qui le commande, 
- l’heure de mise en route en heure creuse si nécessaire.

Le programme effectue toutes les minutes, un recueil de la puissance solaire disponible. Il arrête les appareils qui ont atteint leur durée de fonctionnement programmée et essaie de mettre en route les appareils en attente en fonction de la puissance disponible.

La mise en route de l’appareil se fait à partir d’un seuil de puissance, qui dépend du coût de fonctionnement comparé à celui des heures creuses.Le calcul du seuil de puissance nécessaire à la mise en route, se fait selon la formule :
P seuil = P appareil \* (1-(Prix HC – Prix rachat) / Prix HP).
Cette modalité permet d’abaisser la puissance de mise en route et de voir s’il est plus avantageux de démarrer de jour ou en heures creuses

En cas de production solaire limitée du fait de la couverture nuageuse, le système programme, tous les appareils qui n’ont pu être mis en route pendant les heures creuses la nuit aux horaires définis dans le fichier de configuration.

Lorsqu’un appareil est mis en route, il effectue la totalité de son cycle, même si la puissance solaire disponible diminue. En effet, par exemple, il n’est pas logique d’arrêter un lave-vaisselle ou lave-linge en plein cycle.

Pour ceux qui ont un abonnement tempo, l’optimiseur ne fonctionne pas les jours rouges afin d’éviter des dépenses importantes liées à des diminutions de production solaire.

# Installation
1. Installer **l’add-on appdaemon** à partir de paramètres / modules complémentaires si cela n’est pas déjà fait.
2. **[Télécharger le dépôt]https://github.com/loudemer/PVOptimizer.git**
3. Mettre les fichiers *PVOptimizer.py et PVOptimiser.yaml* dans le répertoire *addon\_configs/a0d7b954\_appdaemon/apps*
4. Mettre le fichier *optimizerentities.yaml* dans le répertoire */config/* de HA ou dans un sous répertoire dédié au fichiers yaml si vous en avez un.

# Configuration
1. Ajouter dans le fichier addon\_configs/a0d7b954\_appdaemon/appdaemon.yaml
  
 ```yaml  
        pvoptimizer_log :
          name: PVOptimizerLog
          filename: /homeassistant/log/pvoptimizer.log
          log_generations: 3
          log_size: 100000 
```
   Ceci vous permet de lire les log de l’application dans le fichier /config/ pvoptimizer\_log
   ou directement dans http://ip\_ha:5050

2. **Compléter** le fichier /addon\_configs/a0d7b954\_appdaemon/apps/PVOptimiser.yaml :
   a. nom de votre sensor qui donne l’énergie solaire disponible dans ‘**available\_energy :’**
   b. type d’abonnement électrique dans **‘subscription :’**
      *Tempo, HeuresCreuses, Base*
   c. sensor qui donne la couleur du jour tempo dans ‘**journée\_tempo :’**
      mettre deux double quote si vous avez un autre type d’abonnement
   d. **les prix du Kwh** en Centimes ou en Euros. Pour un abonnement Heures creuses mettre les prix : prix\_bleu\_hc et prix\_bleu\_hp.

3. **Décrire chaque appareil dans ‘my\_devices :’** en respectant bien l’indentation comme décrite ci-dessous. Le fichier est prérempli avec un appareil à titre d’exemple que vous pouvez modifier.

```yaml  
     my_devices:
       lave_vaisselle:
         name: lave_vaisselle
         power: 2000
         duration: 180
         switch_entity: switch.lave_vaisselle
         night_time_on: "00:30:00"    
```
La puissance( power) est en Watt, 
la durée (duration) en minutes, 
l’heure de mise en route (night\_time\_on) doit être mise à None s’il s’agit d’une autre application type filtration piscine par exemple. 

4. **Recharger la configuration yaml**  dans outils/toute la configuration 
5. **Créer le dashboard de contrôle des appareils** ou adapter celui donné dans le dépôt.

# Le Dashboard
A titre d’exemple vous trouverez dans le dépôt un fichier yaml d’exemple de configuration de base tout à fait perfectible.
Il contient 4 entités par appareil :
- La demande de mise en route : *input\_boolean.device\_request\_1*,
- Le sensor switch de commande de l’appareil *switch.<appareil>* ,
- Le sensor power de l’appareil *sensor.<power appareil>*
- Le sensor de durée d’exécution : *input\_text.device\_duration\_1*.

Les sensors *input\_boolean.device\_request\_x* et *input\_text.device\_duration\_x* sont prédéfinis dans le fichier optimizerentities.yaml. x correspond au rang de définition de l’appareil .
Il faudra ajouter une entrée sur le dashboard pour le sensor *input\_boolean. enable\_solar\_optimizer*** qui permet de désactiver l’application si nécessaire.

# Mode d’emploi de base
Une fois l’installation réalisée, l’intégration est opérationnelle.
Cette partie concerne les appareils qui sont uniquement commandés par un switch

## Activation d’un appareil
Pour effectuer une demande de mise en route d’un appareil, il suffit de cliquer sur le premier sensor (Request).
Si l’énergie solaire disponible est suffisante, le switch de l’appareil va être activé, la puissance délivrée va s’afficher et le décompte du temps d’exécution va débuter et s’afficher toutes les minutes.
Il est possible de forcer le démarrage d’un appareil, même si la puissance solaire disponible n’est pas suffisante en cliquant directement sur le switch. Les entités de l’appareil s’afficheront alors comme si le démarrage avait été effectué par l’intégration.

## Arrêt d’un appareil
L’appareil va s’arrêter lorsque le temps d’exécution planifié sera écoulé. On peut forcer l’arrêt en cliquant sur le switch.

## Programmation en heure creuse
Tous les appareils pour lesquels une demande n’a pas été satisfaite dans la journée, faute d’énergie suffisante sont automatiquement reprogrammés la nuit en heure creuse. 
Pour cela il faut veiller à mettre dans la configuration des horaires de démarrage en période creuse.

## Visualisation des problèmes éventuels
L’intégration génère un fichier de log qui est stocké dans le fichier /config/log/pvoptimizer.log.
Il est possible aussi d’avoir plus de détails en appelant directement la console de debug d’appdaemon : http://<ip_homeassistant>:5050

Vous pourrez alors voir le démarrage et l’arrêt de l’intégration dans *main\_log*, les erreurs éventuelles dans *error\_log* et le déroulement de l’activité de l’intégration dans *pvoptimizer\_log*.

# Utilisation avancée
L’intégration, PVOptimizer permet de piloter des appareils qui nécessitent des paramètres de régulation supplémentaires.

Par exemple, pour une piscine, il faut prendre en compte la température de l’eau pour déterminer la durée et le séquencement des cycles de filtration journalière.
Pour cela, il est plus simple de piloter le fonctionnement de l’appareil dans une application appdaemon séparée qui communique avec PVOptimizer.
L’application séparée va alors demander la mise en route de l’appareil auprès de PVOptimizer en basculant *input\_boolean.device\_request\_x* à true.

PVOptimizer va alors donner l’autorisation d’activation en basculant *input\_boolean.start.device\_x* à true. 
L’application séparée va alors gérer le fonctionnement de l’appareil et l’arrêter si besoin avant le délai imparti. 
Elle pourra demander au besoin ultérieurement d’autres mises en route.

Trois applications sont actuellement en cours d’évaluation :
- Filtration de piscine traitée à l’oxygène actif,
- Pompe à chaleur Air-eau avec plancher chauffant
- Charge de véhicule électrique sans palier
Elles seront publiées prochainement dans mon dépôt.

# Désinstallation
Il faut retirer les 2 fichiers *PVOptimizer.py et PVOptimiser.yaml* dans le répertoire *addon\_configs/a0d7b954\_appdaemon/apps/*
Puis le fichier *optimizerentities.yaml* dans le répertoire */config/* de HA,
Retirer aussi le dashboard PVOptimizer
Redémarrer HA.

# Performances
L’intégration tourne depuis 4 mois avec une installation solaire de 3 KWc. 
Pour l’instant je suis à 95% d’autoconsommation. Il reste à évaluer le comportement en été.

# OPTIMISATION DE L'AUTO CONSOMMATION SUR HOME ASSISTANT

Dans le contexte actuel, des économies d'énergie, l'installation de panneaux solaires en auto consommation semble devenir une bonne solution. Pour être rentable, il faut toutefois bien calculer la puissance de l'installation photovoltaïque et optimiser l'utilisation de sa production solaire. Pour cela, Home Assistant (HA) est un outil précieux.

Cette présentation a pour but de guider les nouveaux utilisateurs qui ne sont pas encore équipés ou en cours de l'être.
 Pour certains, cette présentation donnera l'impression d'enfoncer des portes ouvertes ou ne n'être pas exhaustive mais il est difficile de l'être compte tenu du nombre de variantes d'installations.

Si vous n'êtes pas encore équipés en panneaux solaires, je vous conseille tout d'abord de faire un tableau Excel répartissant l'utilisation de vos gros consommateurs d'énergie dans la journée et dans la semaine afin de fixer la puissance maximale crête dont vous avez besoin.
 En effet, les installateurs ont souvent tendance à vous proposer des puissances plus importantes que celle dont vous avez besoin et l'amortissement de votre installation risque d'être encore plus longue.

Pour optimiser votre consommation avec Home Assistant, il faudra que votre installation soit équipée d'une pince ampèremétrique sur le câble d'arrivée du réseau dans votre maison pour mesurer la puissance importée ou exportée. Cette dernière donnera des valeurs négatives lorsque vous exporter de l'énergie à partir du réseau et des valeurs positives lorsque vous en importer.

Accessoirement il est commode d'avoir une deuxième pince ampèremétrique pour mesurer la production solaire branchée sur le câble sortant du boitier AC de votre installation photovoltaïque pour afficher votre production dans le tableau de bord HA énergie

Les gros équipements consommateurs d'énergie dans une maison sont : le chauffage, la climatisation, le chauffe-eau, la filtration de piscine, la voiture électrique, le lave-linge, le sèche-linge, le lave-vaisselle.

Nous allons tout d'abord voir les particularités de chacun, afin de déterminer les équipements complémentaires nécessaires à l'optimisation de consommation sur HA.
 Nous verrons ensuite les moyens de piloter ces équipements au moyen de HA.

# Les équipements

## 1/ le Chauffage et la climatisation:

1. Pour une maison équipée de radiateurs électriques, l'utilisation de radiateurs connectés est nécessaire afin de pouvoir adapter la température directement à partir de HA.
 Plusieurs possibilités s'offrent à vous, soit l'utilisation de relais, on/off associés à un capteur de température dans la pièce, soit l'utilisation de radiateur à fil pilote. De nombreux topics sont disponibles sur le forum pour expliquer les différents modes de connexion à HA.
2. Pour ceux qui disposent d'une pompe à chaleur (PAC) air-air ou d'un poêle à granulés, la régulation s'apparente à celle des radiateurs électriques. Il faut simplement s'assurer que l'on dispose d'un moyen de connexion entre la PAC et HA.
3. Enfin, pour ceux qui dispose d'une PAC air-eau, la régulation s'avère différente du fait de l'inertie de l'installation. En particulier, si le chauffage se fait par le sol, seul un thermostat d'ambiance connecté disposé dans une pièce principale est suffisant, car souvent la PAC s'auto régule d'elle-même en fonction de la température extérieure et de la température du circuit d'eau intérieur.

## 2/ Le Chauffe-eau

1. Pour ceux qui ont un chauffe-eau avec résistance, thermique classique, Il est recommandé de s'équiper d'un routeur qui va se déclencher dès que la production solaire dépasse la consommation de base de votre maison et envoyer la production solaire excédentaire dans le chauffe-eau. Ceci permet d'optimiser au mieux votre auto consommation. Cela est aussi possible pour ceux qui disposent d'un chauffe-eau thermodynamique équipé d'une résistance d'appoint.
2. De plus, il est souhaitable de rajouter une sonde thermique sur le chauffe-eau connectée directement sur votre routeur afin de limiter la température maximum de l'eau à 50° un jour sur deux et 55° l'autre jour. Le but étant de faire des économies tout en assurant, une stérilisation de l'eau, au moins tous les deux jours.
3. Si votre chauffe-eau est éloigné de votre routeur, vous pouvez utiliser une sonde thermique connectée à HA soit par un ESP ou tout autre moyen afin de la retransmettre à votre routeur. Il faut bien sûr s'assurer que votre routeur puisse communiquer avec HA pour lui retransmettre la température ou simplement l'ordre d'arrêt lorsque la température maximum est atteinte.
4. Enfin il est nécessaire de pouvoir récupérer au niveau de HA la puissance instantanée délivrée au chauffe-eau pour le logiciel d'optimisation.

## 3/La filtration de piscine

La filtration de la piscine est un des points difficiles à intégrer dans l'optimisation de l'autoconsommation car il faut assurer une durée de filtration proportionnelle à la température et à la puissance de la pompe de filtration sur un certain nombre de cycles répartis dans la journée.
 Il est donc nécessaire d'équiper la piscine au minimum d'une sonde de mesure de la température de l'eau et au moins d'un relais sur la pompe de filtration tous deux connecté à HA.
 Il faudra y ajouter, si nécessaire , d'autres capteurs ou commandes dépendant du type de traitement de l'eau (mesure PH, chlore, pompe à galet)

## 3/Le véhicule électrique

La charge d'un véhicule électrique (VE) se fait soit au moyen d'une borne dédiée, soit au moyen un chargeur branché sur une prise de courant.
 Dans le cas d'une borne, il faut s'assurer qu'elle puisse être connectée à HA. Certaines bornes permettent de faire varier la puissance de charge. C'est un plus qu'il faudra bien sûr intégrer dans le logiciel d'optimisation.
 Dans le cas du chargeur, il faudra disposer d'une prise de courant, pilotée par un relais par le biais de HA.

Dans le cas de la charge d'un VE, il faut éviter de faire trop d'interruptions et cela doit donc être pris en compte dans le logiciel.

### 4/lave-linge, sèche-linge, lave-vaisselle

Pour ces appareils, le plus simple est de les brancher sur une prise de courant connectée qui permet en outre de mesurer leur consommation.
 Il faut pouvoir activer la prise au moment de la programmation et de l'éteindre ensuite en attendant le moment opportun de mise en route. Cela pose un problème lorsque la prise est difficilement accessible. Il faut donc alors le faire à partir de HA.

# Les Logiciels

## 1/ Principe de fonctionnement

L'objectif principal de l'auto consommation est d'utiliser au maximum l'excédent de production solaire, lorsqu'il y en a. la mesure de cet excédent solaire se fait sur la pince branchée sur le câble d'alimentation de votre domicile. Lorsque la mesure est négative, il s'agit d'un excédent.
 Si vous disposez d'un routeur, la puissance solaire disponible correspond à celle qui est délivrée par le routeur au chauffe-eau additionnée de l'excédent mesuré précédemment. D'où l'importance de pouvoir disposer de cette mesure au niveau de HA.

Chaque appareil est caractérisé, par sa puissance et sa disponibilité.

Le logiciel va scruter régulièrement la puissance solaire disponible et chercher un appareil demandeur dont la puissance soit inférieure ou égale à un seuil donné.

Ce seuil est calculé de la manière suivante pour un contrat HP HC.

Pseuil = Pappareil \* (Prix kWh HC - Prix Rachat) / Prix kWh HP.

Par exemple : pour un Prix HC = 0,20€, prix HP = 0,25€, prix de rachat = 0,11€ et puissance de l'appareil = 2000 W.

Seuil = 2000W \* (0,2 - 0,11) / 0,25 = 720 W

Il faut prendre comme référence pour le calcul du coefficient les prix du kWh incluant les taxes d'acheminement

En fin de journée, le logiciel va programmer en heures creuses toutes les tâches qui n'ont pu être réalisées dans la journée.

## 2/ Paramètres complémentaires

### Durée minimale de fonctionnement

Elle est utile lorsque plusieurs équipements sont en attente dont certains sont prioritaires (cycle de filtration, charge de VE). On peut alors interrompre un autre équipement qui a dépassé cette durée minimale de fonctionnement pour en démarrer un autre.

Pour les appareils électro ménagers comme un lave-linge par exemple la durée minimale correspond à une lessive complète

### Durée maximale de déclenchement

Elle correspond à la durée d'un cycle de filtration, d'une lessive …

### Coefficient de priorité

Il permet de prioriser certaines tâches par rapport à d'autres. Cette priorité peut varier d'un jour à l'autre.

### Paramètres prédictifs

On peut aussi introduire la prévision d'ensoleillement et de température du lendemain

### Couleur du jour du contrat Tempo

Indispensable pour ceux qui ont ce type de contrat pour programmer un minimum d'appareil les jours rouges. En pratique, vu le peu d'ensoleillement en hiver, il faut s'arranger pour ne pas activer le chauffe-eau la nuit et faire fonctionner le routeur dans la journée

# Dépôts disponibles sur HACS

Il existe déjà plusieurs dépôts sur HACS qui sont déjà très performants citons en particulier :

[Solar\_optimizer](https://forum.hacf.fr/t/integration-solar-optimizer-optimisation-de-sa-consommation-solaire/25318)

[Emhass](https://forum.hacf.fr/t/add-on-pour-la-gestion-denergie-avec-home-assistant/11099)

Ce dernier intègre aussi de la prédiction.

# Contrat d'abonnement à l'électricité

Vaste sujet qui suscite de nombreuses discussions sur les réseaux. Pour ce qui concerne l'auto consommation, il semble déjà logique de prendre au moins un contrat heure creuse heure pleine et de discuter aussi l'option tempo.

Pour cette dernière, il faut prendre en compte la qualité de l'isolation de la maison, la présence d'un chauffage complémentaire au bois, la possibilité d'un mode de cuisson alternatif au gaz, la présence d'une PAC air-eau avec chauffage au sol.
 Il n'est pas nécessaire de remplir tous ces critères, mais il faut tout de même en avoir un minimum pour faire des économies sans se faire de mal.

# Mon point de vue

Il n'y a pas, à ma connaissance, de logiciel d'optimisation d'auto consommation qui intègre ce type de contrat Tempo. De plus la filtration de la piscine est difficile à intégrer dans ces logiciels.

Aussi, ai-je décidé de développer mon propre logiciel en python sur appdaemon.
Il s'agit d'un optimiseur qui contrôle directement le gros électro-ménager et qui communique avec 3 autres modules python respectivement dédiés à la PAC, à la charge du VE et à la filtration de la piscine.

Chacun de ces 3 modules évalue ses propres besoins en énergie, gère ses demandes au module d'optimisation et assure le suivi de sa tâche

Le module principal est présenté dans ce [dépôt](https://github.com/loudemer/PVOptimizer.git). Les autres modules sont en cours de test ils seront publiés prochainement.
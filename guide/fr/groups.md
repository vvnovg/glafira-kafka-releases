# Consumer groups

Un consumer group est un ensemble de consommateurs qui se partagent la lecture de topics. Kafka attribue chaque partition à un seul membre du groupe et stocke, pour chaque partition, l’offset commité par le groupe. Après un redémarrage, les applications reprennent la lecture à partir de cet offset.

Ouvrez **Consumer groups** sous la connexion dans le panneau **Connexions**. Les groupes d’un topic figurent aussi dans l’onglet **Consumer groups** du topic. Voir [Écran du topic](topics.md#topic-details).

<a id="group-list"></a>

## Liste des groupes

La liste affiche tous les consumer groups du cluster. Les colonnes :

- **Groupe** — l’ID du groupe (`group.id`).
- **État** — l’état signalé par Kafka, par exemple `Stable` (les membres lisent), `Empty` (aucun membre), `PreparingRebalance` ou `CompletingRebalance` (les partitions sont en cours de redistribution) et `Dead` (le groupe n’existe plus).
- **Membres** — le nombre de membres actifs.
- `Assignor` — la stratégie de répartition des partitions, par exemple `range`.
- **Lag** — le lag total du groupe. Voir [Lag](groups.md#lag).
- **Topics** — les topics pour lesquels le groupe a commité des offsets.

Saisissez du texte dans **Rechercher un groupe** pour afficher les groupes dont l’ID le contient, sans tenir compte de la casse. La liste des états à côté n’affiche que les groupes dans un état donné ; **Tous les états** retire ce filtre. Le compteur indique combien de groupes sont affichés et combien il y en a au total.

Les membres, l’assignor et le lag ne sont chargés que pour les lignes visibles, au fil du défilement. En attendant, la cellule affiche `…`. S’ils ne peuvent pas être chargés, la cellule affiche `—` ; survolez-la pour voir la raison.

**Actualiser** recharge la liste et ses données. Si certains brokers n’ont pas répondu, un avertissement signale que la liste peut être incomplète.

<a id="lag"></a>

## Lag

Le lag est le nombre de messages que le groupe n’a pas encore lus. Pour chaque partition, c’est la fin du log moins l’offset commité. Le lag n’est jamais affiché en dessous de zéro.

Le lag dépend uniquement des offsets commités. Un consommateur actif peut avoir lu plus loin que ce qu’il a commité : son lag réel peut donc être plus faible. Dès que le groupe commite de nouveau, le lag diminue.

Une partition sans offset commité affiche **aucun commit** et n’est pas comptée dans le total. Le lag total d’un groupe affiche `—` lorsque le groupe n’a aucun commit (**aucun commit**), ou lorsque les offsets d’une partition n’ont pas pu être lus (**lag inconnu : erreur d’offsets de partition**).

Le lag compte des offsets, pas des messages. Dans les topics avec compaction ou transactions, certains offsets ne contiennent aucun message visible : il peut donc rester au groupe moins de messages à lire que ne l’indique le lag.

Les valeurs sont un instantané pris au moment du chargement. Pour suivre le lag, ouvrez le groupe et activez l’actualisation automatique.

<a id="group-details"></a>

## Détails du groupe

Cliquez sur un groupe dans la liste pour ouvrir son onglet. En haut, un résumé affiche l’état, le nombre de membres et l’assignor.

- **Actualisation auto** — **Actualisation auto : désactivée**, **toutes les 5 s** ou **toutes les 10 s**.
- **Actualiser** — recharge immédiatement.
- **Lire en tant que membre** — lance une [session consumer](consumer-sessions.md) dans ce groupe. Le formulaire de la session reçoit l’ID du groupe et les topics pour lesquels le groupe a des commits. Le bouton est désactivé sur les connexions en **Lecture seule**. Survolez le bouton pour voir la raison.

**Partitions** liste les partitions pour lesquelles le groupe a des commits, ainsi que les partitions attribuées à ses membres. Le lag total du groupe s’affiche au-dessus du tableau. Les colonnes sont **Topic**, **Partition**, `Committed` (l’offset commité), `Log end` (l’offset de fin de la partition), **Lag** et **Membre** (le client ID du membre qui lit la partition).

**Membres** liste les membres actifs : `Client ID`, **Hôte**, `Member ID` et **Attribution** (les partitions du membre sous la forme `topic:partition`). Un groupe sans membres affiche **Aucun membre actif.**

La deuxième ligne contient **Réinitialiser les offsets** et **Supprimer le groupe**. Elle est masquée sur les connexions en lecture seule.

<a id="reset-offsets"></a>

## Réinitialiser les offsets

La réinitialisation déplace les offsets commités d’un groupe. Le groupe relit alors des messages ou en saute. Les offsets ne peuvent être réinitialisés que pour un groupe sans membres actifs (état `Empty` ou `Dead`). Sinon, les membres écraseraient les nouveaux offsets avec leurs propres commits. Arrêtez d’abord les applications du groupe.

1. Dans l’onglet du groupe, cliquez sur **Réinitialiser les offsets**. Tant que le groupe a des membres, le bouton est désactivé ; survolez-le pour voir la raison.
2. Choisissez la **Stratégie**. Voir [Stratégies de réinitialisation](groups.md#reset-strategies).
3. Choisissez la **Portée**. Voir [Portée de la réinitialisation](groups.md#reset-scope).
4. Cliquez sur **Aperçu**.
5. Vérifiez le tableau et cliquez sur le bouton d’application. Il indique le nombre de partitions qui seront modifiées.

L’aperçu ne modifie rien. Pour chaque partition, il affiche le topic, la partition, **Actuel** (l’offset commité), **Nouveau** (l’offset cible) et `Low / High` (le premier offset et l’offset de fin de la partition). Une partition pour laquelle aucune cible ne peut être calculée est marquée **Aucun offset cible — ne sera pas appliqué**, avec la raison. Ces partitions sont ignorées.

Seul le plan affiché dans l’aperçu est appliqué. Toute modification du formulaire masque l’aperçu, et il faut cliquer de nouveau sur **Aperçu**. Sur une connexion prod, l’application demande le nom de la connexion ; sur les autres connexions, elle est immédiate.

Avant l’application, l’app vérifie de nouveau que le groupe n’a pas de membres. Après l’application, **Résultat** liste chaque partition avec son nouvel offset ou une erreur.

Vous pouvez aussi préparer une réinitialisation à partir d’un message : voir [Réinitialiser un groupe sur un message](browse.md#reset-group-here). **Fermer** ferme le panneau de réinitialisation.

<a id="reset-strategies"></a>

## Stratégies de réinitialisation

**Stratégie** définit l’offset cible pour chaque partition de la portée :

- **Depuis le début (earliest)** — le premier offset encore stocké. Le groupe relit tout ce que le topic conserve.
- **Depuis la fin (latest)** — la fin de la partition. Le groupe saute tout ce qui s’est accumulé et ne lit que les nouveaux messages.
- **Par date** — le premier message écrit à la date indiquée dans **Date et heure (locales)** ou après. Dans les partitions sans messages plus récents, la cible est la fin.
- **Offset précis** — la valeur indiquée dans **Offset**, un entier supérieur ou égal à 0, dans chaque partition. Si une partition n’a pas cet offset, la cible est son premier offset ou son offset de fin.
- **Décaler de ±N** — déplace l’offset commité de la valeur indiquée dans **Décalage (±N)**. Un nombre négatif relit N messages en arrière, un nombre positif saute N messages. Le résultat reste compris entre le premier offset et l’offset de fin. Les partitions sans offset commité ne peuvent pas être décalées et sont ignorées.

L’heure est votre heure locale, à la seconde près. Kafka trouve l’offset d’après les horodatages des messages.

<a id="reset-scope"></a>

## Portée de la réinitialisation

**Portée** définit les partitions réinitialisées. Les autres partitions conservent leurs offsets.

- **Groupe entier** — toutes les partitions pour lesquelles le groupe a commité des offsets. Si le groupe n’a encore aucun commit, choisissez plutôt des topics.
- **Topics** — toutes les partitions des topics indiqués, y compris celles sans commit. Séparez les noms par des virgules, des espaces ou des sauts de ligne. Utile pour qu’un groupe commence à lire un nouveau topic à partir d’un point choisi.
- **Partitions** — uniquement les partitions indiquées. Saisissez-les dans **Partitions (topic:partition)** sous la forme `topic:partition`, séparées par des virgules ou des espaces, par exemple `orders:0, orders:3`.

<a id="delete-group"></a>

## Supprimer un groupe

La suppression d’un groupe efface tous ses offsets commités. Seul un groupe sans membres, à l’état `Empty`, peut être supprimé.

1. Dans l’onglet du groupe, cliquez sur **Supprimer le groupe**. Tant que le groupe a des membres, le bouton est désactivé ; survolez-le pour voir la raison.
2. Saisissez l’ID du groupe pour confirmer. Sur une connexion prod, saisissez aussi le nom de la connexion.
3. Cliquez sur **Supprimer**.

Après la suppression, l’onglet du groupe se ferme. Si une application avec ce `group.id` redémarre, Kafka recrée le groupe. Sans offsets commités, l’application commence là où l’indique son paramètre `auto.offset.reset` : au début ou à la fin.

Sur les connexions en **Lecture seule**, le bouton est masqué.

# Cluster et topics

Ce chapitre traite de l’aperçu du cluster, de la liste des topics, de la création et de la suppression de topics, de l’écran d’un topic et de sa configuration. Sous chaque connexion du panneau **Connexions** figurent les liens **Aperçu du cluster** et **Topics**. Chacun s’ouvre dans son propre onglet.

<a id="cluster-overview"></a>

## Aperçu du cluster

L’écran **Aperçu du cluster** montre l’état du cluster au moment de son chargement. Cliquez sur **Actualiser** pour le recharger.

- `Cluster ID` — l’identifiant du cluster.
- **Contrôleur** — l’ID du broker que le cluster déclare comme contrôleur. Dans les clusters KRaft, les clients ne communiquent pas directement avec les contrôleurs : le cluster indique ici un broker aléatoire, et non le contrôleur actif ; ce broker peut changer d’une actualisation à l’autre.
- **Brokers** — le nombre de brokers.
- **Topics (internes)** — tous les topics, avec le nombre de topics internes entre parenthèses.
- **Partitions** — le nombre de partitions de tous les topics.
- `Under-replicated` — les partitions dont toutes les répliques ne sont pas synchronisées (dans l’ISR). En général, un broker est arrêté ou n’arrive pas à suivre.
- `Offline` — les partitions sans leader. Personne ne peut y écrire ni y lire.

Une valeur non nulle dans `Under-replicated` ou `Offline` est mise en évidence.

En dessous se trouve le tableau des brokers avec `ID` et **Adresse**. Le broker du champ **Contrôleur** est marqué **contrôleur**. Dans les clusters KRaft, il s’agit du même broker aléatoire, et non du contrôleur actif. Cliquez sur un broker pour afficher sa configuration sous le tableau. La configuration d’un broker est en consultation seule. Utilisez **Rechercher un paramètre** pour trouver un paramètre par son nom, et **Modifiés uniquement** pour masquer les paramètres à leur valeur par défaut. Une icône de cadenas signale les paramètres en lecture seule.

<a id="topic-list"></a>

## Liste des topics

L’écran **Topics** liste les topics du cluster. Cliquez sur une ligne pour ouvrir le topic dans son propre onglet.

Les colonnes sont **Nom**, **Partitions**, `RF` (facteur de réplication), **Messages (≈)**, `cleanup.policy` et **Sous-répl.** (le nombre de partitions sous-répliquées). Le nombre de messages est approximatif : c’est la différence entre les offsets de fin et de début des partitions. La compaction et les marqueurs de transaction rendent le nombre réel plus petit. Le nombre de messages et `cleanup.policy` ne sont chargés que pour les lignes que vous faites défiler à l’écran.

Commandes de la barre d’outils :

- **Rechercher (ou /regex/)** — saisissez une partie d’un nom pour trouver des topics, sans tenir compte de la casse. Pour chercher avec une expression régulière, placez-la entre barres obliques, par exemple `/^orders\..*/`.
- **Internes** — affiche les topics internes, comme `__consumer_offsets`. Ils sont marqués **(interne)**.
- La liste d’actualisation automatique — **Actualisation auto : désactivée**, **toutes les 10 s** ou **toutes les 30 s**.
- **Actualiser** — recharge immédiatement la liste et le nombre de messages.

À côté de la barre d’outils, l’app indique combien de topics correspondent à la recherche sur le total. Sur les connexions en **Lecture seule**, les boutons **Créer un topic** et **Sélectionner** sont masqués.

<a id="create-topic"></a>

## Créer un topic

1. Dans la liste des topics, cliquez sur **Créer un topic**. Le formulaire **Nouveau topic** s’ouvre au-dessus de la liste.
2. Saisissez le **Nom du topic**. Il peut contenir des lettres latines, des chiffres, `.`, `_` et `-`, jusqu’à 249 caractères. Le formulaire vous avertit si le nom contient à la fois `.` et `_` : les noms des métriques Kafka de ces topics peuvent entrer en collision avec ceux d’autres topics.
3. Indiquez le nombre de **Partitions**. Les partitions sont lues en parallèle, et l’ordre n’est garanti qu’au sein d’une même partition. Ce nombre peut être augmenté plus tard, mais jamais diminué.
4. Indiquez le **Facteur de réplication** — le nombre de copies de chaque partition conservées par les brokers. Il ne peut pas dépasser le nombre de brokers, que le formulaire affiche sous le champ. La valeur par défaut est 3, ou le nombre de brokers s’il est inférieur. `-1` utilise la valeur par défaut du broker.
5. Choisissez un **Préréglage** :
   - **delete (par défaut)** — les anciens messages sont supprimés selon leur âge.
   - `compact` — seule la dernière valeur de chaque clé est conservée.
   - `compact + delete` — les deux.
   - **test (rétention 1 h)** — les messages sont conservés une heure.
6. Si nécessaire, ajoutez des **Paramètres supplémentaires (key=value)**, un par ligne, par exemple `retention.ms=86400000`. Ils remplacent les valeurs du préréglage. Les lignes commençant par `#` sont ignorées.
7. Cliquez sur **Créer**.

Lorsque le formulaire est valide, il affiche la commande `kafka-topics.sh` équivalente. Vous pouvez la copier, par exemple pour créer le même topic dans un autre cluster.

Sur une connexion prod, vous confirmez la création en saisissant le nom de la connexion. Une fois le topic créé, son onglet s’ouvre.

<a id="delete-topics"></a>

## Supprimer des topics

Pour supprimer un topic, ouvrez-le et cliquez sur **Supprimer le topic**. Saisissez le nom du topic pour confirmer et cliquez sur **Supprimer**. L’onglet du topic se ferme.

Pour supprimer plusieurs topics à la fois :

1. Dans la liste des topics, cliquez sur **Sélectionner**. Des cases à cocher apparaissent à côté des noms.
2. Cochez les topics. Les topics internes ne peuvent pas être sélectionnés.
3. Cliquez sur le bouton de suppression. Il indique le nombre de topics sélectionnés.
4. Vérifiez la liste des noms, saisissez leur nombre pour confirmer et cliquez sur **Supprimer**.

**Annuler la sélection** masque les cases à cocher.

Sur une connexion prod, les deux méthodes de suppression demandent aussi de saisir le nom de la connexion.

La suppression est irréversible : les données sont perdues. Si certains topics ne sont pas supprimés, par exemple faute de droits, l’app les liste avec la raison. Les autres topics sont supprimés.

<a id="topic-details"></a>

## Écran du topic

L’onglet du topic affiche le nombre de partitions, le facteur de réplication et le nombre approximatif de messages. Les boutons du haut :

- **Actualiser** — recharge les données du topic.
- **Envoyer un message** — ouvre le **Producer** avec ce topic. Voir [Producer](producer.md).
- **Lire en tant que groupe** — lance une session consumer pour ce topic. Voir [Sessions consumer](consumer-sessions.md).

Sur les connexions en **Lecture seule**, ces deux boutons sont désactivés.

La deuxième ligne contient les actions qui modifient le topic. Elle est masquée sur les connexions en lecture seule et pour les topics internes.

- **Ajouter des partitions** — saisissez le nouveau total et cliquez sur **Augmenter**. Le nombre de partitions ne peut pas être diminué. Après une augmentation, des messages ayant la même clé peuvent aller dans une autre partition : l’ordre par clé n’est alors plus garanti.
- **Purger** — supprime tous les messages jusqu’à la fin actuelle des partitions sélectionnées. Le topic lui-même est conservé. Sélectionnez les partitions, saisissez le nom du topic et cliquez sur **Purger les partitions**. La purge ne fonctionne que si `cleanup.policy` inclut `delete` ; sinon, le bouton est désactivé et la raison est affichée. Pour un topic `compact,delete`, la purge supprime aussi les dernières valeurs des clés.
- **Supprimer le topic** — voir [Supprimer des topics](topics.md#delete-topics).

Sur une connexion prod, chacune de ces actions demande aussi le nom de la connexion.

Les onglets du bas :

- **Messages** — parcourir les messages. Voir [Parcourir les messages](browse.md).
- **Partitions** — un tableau des partitions : **Leader**, **Répliques**, `ISR` (répliques synchronisées), les offsets de début (`Low`) et de fin (`High`) et le nombre de messages. Une partition sans leader affiche **aucun**.
- **Configuration** — les paramètres du topic. Voir [Configuration du topic](topics.md#topic-config).
- **Consumer groups** — les consumer groups qui ont commité des offsets pour ce topic, avec **État**, **Partitions avec commits** et **Lag**. Cliquez sur un groupe pour l’ouvrir. Voir [Consumer groups](groups.md).

<a id="topic-config"></a>

## Configuration du topic

L’onglet **Configuration** liste tous les paramètres du topic sur trois colonnes :

- **Paramètre** — le nom du paramètre. Les noms en gras sont définis explicitement : sur le topic ou dans la configuration du broker.
- **Valeur** — la valeur actuelle. Les valeurs sensibles sont masquées sous la forme `••••••`.
- **Source** — l’origine de la valeur. Par exemple, `DynamicTopic` est défini sur ce topic, `StaticBroker` et `DynamicBroker` proviennent de la configuration du broker, et `Default` est la valeur par défaut de Kafka.

Utilisez **Rechercher un paramètre** pour trouver un paramètre par son nom, et **Modifiés uniquement** pour n’afficher que les paramètres en gras. Une icône de cadenas signale les paramètres en lecture seule. L’icône `?` à côté d’un nom explique le paramètre.

<a id="edit-config"></a>

## Modifier la configuration

1. Dans l’onglet **Configuration**, cliquez sur **Modifier**. Sur les connexions en **Lecture seule**, ce bouton est masqué.
2. Modifiez les valeurs dans la colonne **Valeur**. Les paramètres en lecture seule et les paramètres sensibles ne sont pas modifiables. Pour retirer la valeur propre au topic, cliquez sur **réinitialiser** : la valeur du broker s’applique alors. Une valeur vide n’est pas autorisée.
3. Cliquez sur **Vérifier les modifications**. L’app affiche chaque modification sous la forme ancienne valeur → nouvelle valeur ; **par défaut** signifie que la valeur est réinitialisée.
4. Cliquez sur **Appliquer**, ou sur **Retour** pour continuer à modifier.

Rien ne change dans le cluster tant que vous n’avez pas appliqué les modifications. **Tout annuler** annule toutes les modifications. Tant qu’il reste des modifications non enregistrées, **Fermer l’éditeur** est désactivé. Sur une connexion prod, vous confirmez les modifications en saisissant le nom de la connexion.

Seuls les paramètres modifiés sont envoyés. Les autres paramètres du topic restent inchangés. Le broker vérifie les valeurs et refuse les valeurs non valides avec une erreur.

<a id="config-parameters"></a>

## Paramètres des topics

Les topics Kafka ont des dizaines de paramètres. Les icônes `?` du tableau de configuration expliquent chacun d’eux. Voici ceux dont on a le plus souvent besoin :

- `cleanup.policy` — ce qui arrive aux anciennes données. `delete` supprime les anciens segments selon leur âge ou leur taille. `compact` ne conserve que la dernière valeur de chaque clé. `compact,delete` fait les deux.
- `retention.ms` — la durée de conservation des messages, en millisecondes. La valeur par défaut est `604800000` (7 jours) ; `-1` les conserve indéfiniment. S’applique lorsque `cleanup.policy` inclut `delete`.
- `retention.bytes` — la taille maximale de chaque partition avant la suppression des anciens segments. La valeur par défaut `-1` signifie aucune limite de taille.
- `segment.ms` et `segment.bytes` — le moment où le broker commence un nouveau segment de log. La rétention et la compaction portent sur des segments entiers : un message peut donc rester plus longtemps que `retention.ms`, jusqu’à la fermeture de son segment.
- `min.insync.replicas` — le nombre de répliques qui doivent confirmer une écriture avec `acks=all`. En général 2 avec un facteur de réplication de 3.
- `max.message.bytes` — le plus grand batch que le broker accepte pour ce topic. La valeur par défaut est d’environ 1 MB. Les producteurs doivent autoriser la même taille (`max.request.size`).
- `compression.type` — la façon dont le broker stocke les messages. `producer` conserve la compression du producteur.
- `message.timestamp.type` — `CreateTime` (l’heure définie par le producteur) ou `LogAppendTime` (l’heure à laquelle le broker a écrit le message). C’est cette heure qui s’affiche lorsque vous parcourez les messages.
- `delete.retention.ms` — la durée pendant laquelle les marqueurs de suppression (messages à valeur nulle) restent dans un topic compacté. La valeur par défaut est de 1 jour.

Certains paramètres n’existent que dans les versions récentes de Kafka, et certains sont obsolètes. L’icône `?` le précise quand c’est important. Les paramètres du tiered storage, comme `remote.storage.enable`, ne fonctionnent que si le cluster le prend en charge.

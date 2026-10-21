# Parcourir les messages

Les messages se parcourent dans l’onglet **Messages** d’un topic. La lecture ne rejoint aucun consumer group et ne commite rien : le cluster n’est pas modifié. Elle fonctionne donc aussi sur les connexions en **Lecture seule**. Pour lire en tant que membre d’un groupe, utilisez une [session consumer](consumer-sessions.md).

<a id="start-reading"></a>

## Lancer la lecture

1. Ouvrez un topic depuis la liste **Topics**. L’onglet **Messages** s’ouvre en premier.
2. Choisissez le point de départ dans **Départ**. Voir [Position de départ](browse.md#start-position).
3. Si nécessaire, définissez les formats et les filtres.
4. Cliquez sur **Lire**.

Les messages apparaissent dans le tableau au fur et à mesure. La ligne à côté des boutons indique combien de messages ont été examinés, combien correspondent aux filtres et combien figurent dans le tableau. Elle affiche aussi l’état : **lecture…**, **limite atteinte**, **fin du topic atteinte** ou **arrêté**.

Sans `Live tail`, la lecture s’arrête à la fin du topic tel qu’il était au moment du clic sur **Lire**, ou plus tôt si la **Limite** est atteinte. La limite compte les messages qui correspondent aux filtres. Elle va de 1 à 100000 ; les grandes valeurs ralentissent la lecture.

Avec `Live tail` activé, la lecture ne s’arrête pas à la fin : les nouveaux messages sont ajoutés au fur et à mesure de leur écriture. Une fois tous les anciens messages lus, l’état affiche **en attente de nouveaux messages…**. Le tableau conserve les derniers messages dans la limite fixée et supprime les plus anciens.

**Arrêter** arrête la lecture et conserve le tableau. Pendant la lecture, le bouton **Lire** devient **Relancer** : il relance la lecture avec les paramètres actuels et vide le tableau. Si la connexion au broker est perdue, l’app se reconnecte d’elle-même et l’indique dans la ligne d’état.

Les messages des différentes partitions sont mélangés dans le tableau. Kafka ne garantit l’ordre qu’au sein d’une même partition. La colonne `P` indique la partition, et `Offset` la position dans celle-ci. Cochez `UTC` pour afficher les heures en UTC plutôt qu’en heure locale.

<a id="start-position"></a>

## Position de départ

**Position de départ** définit où la lecture commence dans chaque partition :

- **N derniers** — les N derniers messages. N est réparti également entre les partitions : avec 100 messages et 4 partitions, l’app en prend jusqu’à 25 dans chacune.
- **Depuis le début** — à partir du plus ancien message encore stocké.
- **Depuis l’offset** — à partir du même offset dans chaque partition. Si une partition n’a pas cet offset, la lecture commence à son début ou à sa fin. Ce mode est surtout utile avec une seule partition.
- **Depuis une date** — à partir du premier message dont l’horodatage est égal ou postérieur à l’heure locale indiquée. Dans les partitions sans messages plus récents, la lecture commence à la fin.
- **X dernières minutes** — la même chose, en comptant à rebours à partir de maintenant.

Le champ **partitions : toutes** limite la lecture à certaines partitions. Saisissez des numéros de partitions séparés par des virgules, par exemple `0,2`. Laissez-le vide pour lire toutes les partitions.

<a id="filters"></a>

## Filtres

Les filtres n’affichent que les messages correspondants. L’app lit quand même tous les messages depuis la position de départ et vérifie chacun d’eux : les filtres n’accélèrent donc pas la lecture. Tous les champs remplis doivent correspondre.

- **La clé contient** et **La valeur contient** — un texte dans la clé ou la valeur.
- `regex` — traite ces champs, ainsi que la valeur de l’en-tête, comme des expressions régulières.
- `Aa` — respecte la casse. Par défaut, la casse est ignorée.
- **En-tête** — le message a un en-tête portant exactement ce nom. **Valeur de l’en-tête** vérifie aussi sa valeur.
- `JSONPath` — une condition sur une valeur JSON, par exemple `$.order.status == "FAILED"`. Sans `==` ni `!=`, le chemin doit simplement exister, par exemple `$.error`. Les messages dont la valeur n’est pas un JSON valide ne correspondent pas.
- **Heure de début** et **Heure de fin** — l’horodatage du message se trouve dans cet intervalle, en heure locale. Les deux bornes sont incluses.

Les filtres portent sur le texte affiché dans le tableau, après décodage. Si le format de la valeur est `hex`, le filtre de valeur cherche dans le texte hexadécimal. Les filtres s’appliquent lorsque vous cliquez sur **Lire** ; après les avoir modifiés, relancez la lecture.

Un filtre ne déplace pas la position de départ. Par exemple, **N derniers** avec un filtre ne trouve des correspondances que parmi les N derniers messages. Pour chercher dans tout le topic, partez de **Depuis le début**.

<a id="formats"></a>

## Formats

**Format** définit le décodage des clés et des valeurs, séparément pour la **clé** et la **valeur**. Seul l’affichage change, pas les données.

- `auto` — la valeur par défaut. Les objets et tableaux JSON valides s’affichent en JSON, les autres textes UTF-8 comme chaînes, les données binaires en hexadécimal.
- `string` — du texte en UTF-8. Les octets qui ne sont pas de l’UTF-8 valide sont remplacés par `�`.
- `json` — du JSON indenté dans les détails du message. Si la valeur n’est pas un JSON valide, les détails affichent l’erreur.
- `hex` — chaque octet sous forme de deux chiffres hexadécimaux.
- `base64` — les octets en Base64.

Les détails indiquent le format réellement utilisé, par exemple `json` lorsque `auto` a détecté du JSON. Les valeurs des en-têtes s’affichent comme du texte ; les valeurs binaires s’affichent sous la forme `0x` suivi de l’hexadécimal. Une clé ou une valeur absente s’affiche comme `null`.

Les très grandes clés et valeurs ne sont affichées qu’en partie, environ le premier MB de texte. Cela vaut aussi pour `hex` et `base64`. Les détails du message affichent alors un avertissement.

<a id="message-details"></a>

## Détails du message

Cliquez sur une ligne pour ouvrir les détails du message à côté du tableau. En haut figurent la partition, l’offset et l’heure. En dessous :

- **Valeur** et **Clé** — le texte complet, avec le format et la taille en octets. Le JSON est indenté. **Copier** copie le texte.
- **En-têtes** — un tableau des noms et valeurs des en-têtes.

Les colonnes du tableau sont `P`, `Offset`, **Heure**, **Clé**, **Valeur**, **En-têtes** (leur nombre) et **Taille** (la taille de la clé et de la valeur en octets). L’heure est l’horodatage du message. Selon le paramètre `message.timestamp.type` du topic, elle est définie par le producteur ou par le broker.

<a id="export-messages"></a>

## Copier des messages

Glafira n’enregistre pas les messages parcourus dans un fichier. Pour sortir un message de l’app, copiez-le :

- **Tout copier en JSON** copie le message entier en JSON : topic, partition, offset, horodatage, clé, valeur, en-têtes et taille. La clé et la valeur sont incluses telles que décodées, avec leur format.
- **Copier** à côté de **Valeur** ou de **Clé** ne copie que ce texte.

Le texte copié utilise le format choisi pour la lecture. Pour copier des données binaires à l’identique, lisez-les en `hex` ou en `base64`. Les très grandes clés et valeurs ne sont copiées qu’en partie, comme elles sont affichées, même en `hex` et en `base64`. Voir [Formats](browse.md#formats).

<a id="open-in-producer"></a>

## Ouvrir dans le Producer

**Renvoyer** ouvre le message dans le **Producer** de la même connexion. Le formulaire reprend le topic, la clé, la valeur, les en-têtes et les formats du message. Le choix de la partition est laissé à Kafka. Les variables présentes dans le texte du message ne sont pas remplacées : la copie est envoyée telle qu’elle a été lue.

Rien n’est envoyé immédiatement. Vérifiez le formulaire, modifiez-le si nécessaire et envoyez-le depuis le Producer. Voir [Producer](producer.md).

Le bouton est désactivé dans deux cas, et la raison est affichée :

- la clé ou la valeur n’a été affichée qu’en partie parce qu’elle est trop grande ;
- le texte a été décodé avec des pertes parce qu’il n’est pas de l’UTF-8 valide. Lisez-le en `hex` ou en `base64` pour envoyer une copie exacte.

Sur les connexions en **Lecture seule**, le bouton est masqué.

<a id="reset-group-here"></a>

## Réinitialiser un groupe sur un message

**Réinitialiser un groupe ici…** prépare une réinitialisation d’offsets pour qu’un consumer group commence à lire à partir de ce message.

1. Dans les détails du message, cliquez sur **Réinitialiser un groupe ici…**.
2. Choisissez un groupe dans la liste. Seuls les groupes existants y figurent.
3. Cliquez sur **Continuer**.

L’onglet du groupe s’ouvre avec le formulaire de réinitialisation prérempli. **Stratégie** vaut **Offset précis** avec l’offset du message. **Portée** vaut **Partitions** avec le topic et la partition du message. Rien n’est encore modifié : cliquez sur **Aperçu** pour voir les nouveaux offsets, puis appliquez la réinitialisation depuis cet écran. Le groupe ne doit avoir aucun membre actif. Voir [Consumer groups](groups.md).

Sur les connexions en **Lecture seule**, le bouton est masqué.

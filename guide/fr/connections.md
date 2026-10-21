# Connexions

Une connexion est le profil enregistré d’un cluster Kafka : adresses, paramètres de sécurité et mots de passe. Les connexions sont listées dans le panneau **Connexions** à gauche. Vous pouvez les classer dans des dossiers et les transférer sur un autre ordinateur par export et import.

<a id="create"></a>

## Créer une connexion

1. Cliquez sur `+` ( **Nouvelle connexion** ) dans l’en-tête du panneau **Connexions**. Pour créer la connexion directement dans un dossier, ouvrez le menu du dossier et choisissez **Nouvelle connexion ici**.
2. Saisissez un nom dans le champ **Nom**. Il apparaît dans le panneau et dans les titres des onglets.
3. Dans **Serveurs bootstrap**, saisissez une ou plusieurs adresses au format `host:port`, séparées par des virgules, par exemple `kafka1:9092,kafka2:9092`. Un seul broker joignable suffit : le client découvre les autres lui-même.
4. Choisissez le **Protocole** et remplissez les champs qu’il ajoute. Voir [Protocoles de sécurité](connections.md#security).
5. Si vous le souhaitez, choisissez un **Dossier**, un **Environnement** et une **Couleur** pour le point affiché à côté du nom.
6. Cliquez sur **Tester la connexion**, puis sur **Enregistrer**.

Après l’enregistrement, le formulaire se ferme et l’aperçu du cluster s’ouvre.

Deux autres champs servent dans des cas particuliers. **Délai de requête, ms** est le temps d’attente d’une réponse du broker. La valeur par défaut est `30000` ms, le minimum `1000`. Augmentez-la pour les réseaux lents et les VPN. **Propriétés librdkafka supplémentaires** accepte n’importe quel paramètre librdkafka sous la forme `key=value`, un par ligne. Les lignes commençant par `#` sont ignorées. Les mots de passe et les clés y sont interdits : les propriétés dont le nom contient `password` ou `secret`, ainsi que `sasl.jaas.config`, `ssl.key.pem` et `ssl.keystore.key`, sont refusées à l’enregistrement.

Pour modifier une connexion plus tard, cliquez sur `✎` ( **Modifier la connexion** ) à côté de son nom. Pour la supprimer, ouvrez le formulaire, cliquez sur **Supprimer** et confirmez avec **Oui, supprimer**.

<a id="test"></a>

## Tester la connexion

**Tester la connexion** vérifie les paramètres du formulaire étape par étape et s’arrête à la première étape en échec :

1. **Configuration** — les adresses, les fichiers de certificats, les mots de passe et le nom d’utilisateur SASL.
2. **DNS / TCP** — les noms d’hôte sont résolus et une connexion TCP s’ouvre vers au moins un serveur bootstrap.
3. **Kafka (TLS, SASL, métadonnées)** — la négociation TLS et l’authentification SASL réussissent, et le broker renvoie les métadonnées. En cas de succès, l’étape affiche le nombre de brokers et de topics.

Le test utilise les valeurs actuellement présentes dans le formulaire, y compris les mots de passe saisis mais pas encore enregistrés. Rien n’est enregistré ni modifié dans le cluster. Si une étape échoue, son message d’erreur et ses détails s’affichent en dessous. Les erreurs courantes sont expliquées dans [Connexion impossible](troubleshooting.md#cannot-connect).

<a id="security"></a>

## Protocoles de sécurité

Le **Protocole** doit correspondre au listener du broker auquel vous vous connectez. Vos administrateurs Kafka savent lequel utiliser.

- `PLAINTEXT` — ni chiffrement ni authentification. Courant pour les clusters locaux et de test.
- `SSL` — le trafic est chiffré avec TLS. Avec un certificat client, il s’agit de mTLS.
- `SASL_PLAINTEXT` — authentification par nom d’utilisateur et mot de passe, sans chiffrement.
- `SASL_SSL` — authentification par nom d’utilisateur et mot de passe via TLS.

Lorsque vous changez de protocole, le formulaire n’affiche que les champs nécessaires à ce protocole.

<a id="sasl"></a>

## SASL

Pour `SASL_PLAINTEXT` et `SASL_SSL`, remplissez trois champs :

- **Mécanisme SASL** — `PLAIN`, `SCRAM-SHA-256` ou `SCRAM-SHA-512`. C’est le cluster qui détermine lequel fonctionne.
- **Utilisateur**.
- **Mot de passe**. Il est enregistré dans le trousseau du système, pas dans le profil. Voir [Mots de passe](connections.md#passwords).

SCRAM n’envoie jamais le mot de passe lui-même. `PLAIN` l’envoie tel quel : avec `SASL_PLAINTEXT`, le mot de passe circule donc en clair. Utilisez si possible `SASL_SSL` pour les clusters situés hors de votre ordinateur.

<a id="tls"></a>

## SSL et CA de confiance

Pour `SSL` et `SASL_SSL`, le formulaire ajoute les paramètres TLS.

**CA de confiance** est le certificat de l’autorité de certification (CA) qui a signé les certificats des brokers. Laissez le champ vide si le système d’exploitation fait déjà confiance à cette CA : les certificats racines du système sont alors utilisés. Sinon, saisissez le chemin complet du fichier ou cliquez sur **Parcourir…**. Une fois le fichier choisi, le champ **Format de la CA** apparaît. Pour PKCS#12 et JKS, un champ **Mot de passe du truststore** s’ajoute.

**Vérifier le nom d’hôte** est activé par défaut. Cette option vérifie que le certificat du broker a été émis pour le nom d’hôte auquel vous vous connectez. Ne la désactivez que dans les environnements de test, par exemple lorsque les brokers utilisent des certificats autosignés portant d’autres noms.

<a id="mtls"></a>

## Certificat client (mTLS)

Si les brokers authentifient les clients par certificat, activez **Certificat client (mTLS)**. Le certificat et la clé sont émis par vos administrateurs Kafka. Remplissez ensuite :

- **Certificat client** — le chemin du fichier de certificat.
- **Format du certificat client** — PEM, PKCS#12 ou JKS.
- Pour PEM : **Clé client** — le chemin du fichier de clé privée, et **Mot de passe de la clé (si chiffrée)** si la clé est chiffrée.
- Pour PKCS#12 : **Mot de passe du keystore**.
- Pour JKS : **Mot de passe du keystore**, **Mot de passe de la clé (s’il diffère)** si la clé a son propre mot de passe, et **Alias de la clé** si le magasin contient plusieurs clés.

Les chemins doivent être absolus. Les fichiers sont lus à chaque connexion : après le renouvellement du certificat, il suffit de se reconnecter. L’app vérifie que la clé correspond au certificat et que le certificat n’a pas expiré.

Cas courant : un cluster dans Kubernetes, dont le secret client contient `tls.crt`, `tls.key` et `ca.crt`. Enregistrez-les dans des fichiers, choisissez le protocole `SSL`, indiquez `ca.crt` dans **CA de confiance**, et utilisez `tls.crt` et `tls.key` comme certificat et clé client PEM, avec des mots de passe vides.

<a id="certificate-formats"></a>

## Formats de certificats

Glafira lit trois formats. Le format est choisi automatiquement d’après l’extension du fichier ; vérifiez-le dans le champ de format et modifiez-le si nécessaire.

- PEM — un fichier texte qui commence par `-----BEGIN`. Extensions `.pem`, `.crt`, `.cer`, `.key`. Pour un certificat client, la clé est un fichier séparé. La CA de confiance peut aussi être un certificat DER binaire.
- PKCS#12 — un magasin binaire contenant un certificat et une clé. Extensions `.p12`, `.pfx`.
- JKS — un keystore Java. Extensions `.jks`, `.keystore`, `.truststore`.

Deux types de fichiers doivent d’abord être convertis :

- Les magasins JCEKS ne sont pas pris en charge. Convertissez-les en PKCS#12 :

```bash
keytool -importkeystore -srcstoretype JCEKS -srckeystore client.jceks -deststoretype PKCS12 -destkeystore client.p12
```

- Les magasins PKCS#12 chiffrés avec des algorithmes anciens (RC2, 3DES) peuvent ne pas s’ouvrir. Si l’app signale un algorithme ancien, recréez le fichier avec AES :

```bash
openssl pkcs12 -export -keypbe AES-256-CBC -certpbe AES-256-CBC -in tls.crt -inkey tls.key -out client.p12
```

<a id="passwords"></a>

## Mots de passe

Le mot de passe SASL et les mots de passe des clés et des magasins sont enregistrés dans le stockage sécurisé du système : Trousseau sur macOS, Gestionnaire d’identification sur Windows, Secret Service sur Linux. Ils ne sont jamais écrits dans le profil de connexion, les fichiers d’export ou l’archive de diagnostic.

Lorsque vous modifiez une connexion enregistrée, les champs de mot de passe sont vides et affichent **laisser vide pour ne pas modifier**. Laissez un champ vide pour conserver le mot de passe enregistré, ou saisissez-en un nouveau pour le remplacer. Pour supprimer le mot de passe enregistré d’un certificat, cochez **Oublier le mot de passe** à côté du champ. À l’enregistrement, les mots de passe que la connexion n’utilise plus sont supprimés, par exemple après le passage du protocole à `PLAINTEXT`.

Si l’app ne peut pas accéder au trousseau, une erreur s’affiche à l’enregistrement. Sur Linux, vérifiez que GNOME Keyring, KWallet ou un autre fournisseur Secret Service tourne dans votre session.

<a id="environments"></a>

## Environnements et prod

**Environnement** qualifie la connexion : `local`, `dev`, `test`, `stage` ou `prod`. L’environnement s’affiche sous forme de badge coloré à côté du nom dans le panneau : vous voyez toujours sur quel cluster vous travaillez.

`prod` ajoute une protection. Toute modification sur une connexion prod, comme créer un topic, envoyer un message, réinitialiser des offsets ou lancer une session consumer, vous demande de saisir le nom de la connexion. La suppression et la purge demandent le nom de l’objet sur n’importe quelle connexion (la suppression de plusieurs topics demande leur nombre), et en prod vous saisissez les deux. Ces confirmations s’affichent dans la fenêtre de l’app.

Une nouvelle connexion prod est d’abord en **Lecture seule**. Décochez la case si vous devez apporter des modifications.

<a id="read-only"></a>

## Lecture seule

Le mode **Lecture seule** bloque tout ce qui modifie le cluster : création, modification, purge et suppression de topics, envoi de messages, réinitialisation d’offsets, suppression de groupes et lancement de sessions consumer. Les boutons de ces actions sont masqués ou désactivés, et le cœur de l’app refuse lui aussi ces opérations. Le parcours des messages et la lecture restent disponibles. Une icône de cadenas à côté du nom dans le panneau signale les connexions en lecture seule.

Utilisez ce mode lorsque vous avez seulement besoin de consulter les données, en particulier sur des clusters partagés. La lecture seule est une protection propre à l’app : elle ne remplace pas les droits d’accès (ACL) de Kafka.

<a id="folders"></a>

## Dossiers

Les dossiers permettent de classer les connexions, par exemple par projet ou par environnement. Les dossiers peuvent être imbriqués.

- Pour créer un dossier au premier niveau, cliquez sur `+📁` ( **Nouveau dossier** ) dans l’en-tête du panneau. Pour créer un sous-dossier, ouvrez le menu du dossier et choisissez **Nouveau dossier**. Saisissez le nom et appuyez sur `Enter`, ou appuyez sur `Esc` pour annuler.
- Pour ouvrir le menu d’un dossier, cliquez sur `⋯` ( **Plus d’actions** ) sur sa ligne ou cliquez sur le dossier avec le bouton droit. Le menu contient **Renommer**, **Supprimer** et les commandes d’export et d’import.
- Seul un dossier vide peut être supprimé. Déplacez ou supprimez d’abord ses connexions et sous-dossiers.
- Les noms de dossiers doivent être uniques à un même niveau et compter au plus 100 caractères.

Pour déplacer une connexion, faites-la glisser sur un dossier, ou sur une autre connexion pour la placer avant celle-ci. Pour la déplacer au premier niveau, déposez-la sur l’en-tête du panneau ou dans l’espace vide sous la liste. Vous pouvez aussi choisir un **Dossier** dans le formulaire de connexion ; **(racine)** désigne le premier niveau. Les dossiers se déplacent de la même façon. Déposez un dossier sur le bord supérieur d’un autre dossier pour le placer avant celui-ci. Un dossier ne peut pas être déplacé dans l’un de ses propres sous-dossiers.

Cliquez sur le nom d’un dossier pour le replier ou le déplier. L’app mémorise les dossiers repliés. Utilisez **Filtrer les connexions** au-dessus de l’arborescence pour trouver des connexions et des dossiers par nom. Les dossiers correspondants s’affichent avec tout leur contenu.

<a id="export"></a>

## Export

L’export enregistre des connexions et des dossiers dans un fichier, par exemple pour les partager avec des collègues ou les transférer sur un autre ordinateur.

1. Pour tout exporter, ouvrez le menu `⋯` de l’en-tête du panneau et choisissez **Tout exporter…**. Pour exporter un dossier avec ses sous-dossiers, choisissez **Exporter le dossier…** dans son menu.
2. Choisissez où enregistrer le fichier. Le nom proposé est `glafira-connections-<date>.glafira-connections.json` ou `<folder>.glafira-connections.json`.

Une fois le fichier enregistré, le panneau indique combien de connexions et de dossiers il contient.

Le fichier contient les paramètres des connexions et la structure des dossiers. Il ne contient ni mots de passe ni propriétés secrètes. Les fichiers de certificats n’y sont pas inclus non plus : seuls leurs chemins sont conservés. Les hôtes, noms d’utilisateur et chemins sont stockés en clair : ne partagez le fichier qu’avec des personnes autorisées à les voir.

<a id="import"></a>

## Import

1. Ouvrez le menu `⋯` de l’en-tête du panneau et choisissez **Importer…**. Pour importer directement dans un dossier, choisissez **Importer ici…** dans son menu.
2. Choisissez un fichier `.glafira-connections.json`. La boîte de dialogue **Importer des connexions** s’ouvre.
3. Dans **Importer dans**, choisissez où placer les nouveaux dossiers et connexions. **(racine)** désigne le premier niveau.
4. Laissez **Mettre à jour les connexions existantes** activé pour mettre à jour à partir du fichier les connexions qui existent déjà. Désactivez cette option pour les laisser telles quelles.
5. Vérifiez l’aperçu et cliquez sur **Importer**, ou sur **Annuler** pour fermer la boîte de dialogue sans rien modifier.

L’aperçu se met à jour après chaque changement dans la boîte de dialogue. Il affiche les totaux et des listes dépliables : **Nouveaux**, **Seront mis à jour**, **Sans changement** et **Ignorés**, avec la raison de chaque connexion ignorée.

Comment l’app fait correspondre le fichier à vos connexions :

- Les connexions sont associées par un ID interne, pas par leur nom. Une connexion est considérée comme existante si le fichier contient la même connexion : par exemple, elle a été exportée depuis cet ordinateur, ou déjà importée auparavant depuis la même source.
- Une connexion mise à jour reçoit les paramètres du fichier. Vos mots de passe et sa place dans l’arborescence sont conservés.
- Les dossiers existants ne sont ni renommés ni déplacés. Les nouvelles connexions du fichier y sont tout de même placées.
- Si une nouvelle connexion porte le même nom qu’une autre dans le même dossier, un numéro est ajouté, par exemple `kafka-dev (2)`.
- Les connexions que cette version de Glafira ne prend pas en charge, ou dont les propriétés supplémentaires contiennent des secrets, sont ignorées.

L’aperçu signale aussi les fichiers de certificats introuvables sur cet ordinateur. Les mots de passe ne figurent pas dans le fichier : après l’import, le panneau indique combien de connexions en ont besoin. Ouvrez chacune de ces connexions, saisissez les mots de passe et enregistrez.

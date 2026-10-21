# Résolution des problèmes

Ce chapitre explique les erreurs de connexion courantes et ce qu’il faut vérifier pour chacune d’elles. Il indique aussi où se trouvent les journaux et que joindre à une demande d’assistance. Sur macOS, le menu « Aide » ouvre ce chapitre via **Résolution des problèmes**.

Un message d’erreur comporte un texte court et, sur la plupart des écrans, le code de l’erreur entre parenthèses, par exemple `TIMEOUT`. Dépliez **Détails** pour voir le message d’origine du client Kafka (librdkafka). Il indique souvent l’adresse exacte du broker et la cause.

<a id="cannot-connect"></a>

## Connexion impossible

Commencez par **Tester la connexion** dans le formulaire de connexion, voir [Tester la connexion](connections.md#test). Le test s’arrête à la première étape en échec, et cette étape indique où chercher :

- **Configuration** — le problème vient du formulaire lui-même : une adresse sans port, un fichier de certificat manquant ou illisible, un mot de passe de certificat erroné. L’erreur nomme le champ. Pour les fichiers de certificats, voir [Erreurs TLS](troubleshooting.md#tls-errors).
- **DNS / TCP** — l’ordinateur ne parvient pas à joindre les serveurs bootstrap. Voir [DNS et TCP](troubleshooting.md#dns-tcp).
- **Kafka (TLS, SASL, métadonnées)** — le réseau fonctionne, mais la négociation TLS, l’authentification SASL ou la requête de métadonnées a échoué. Voir [Erreurs TLS](troubleshooting.md#tls-errors), [Erreurs SASL](troubleshooting.md#sasl-errors) et [Délais dépassés](troubleshooting.md#timeouts).

La cause la plus fréquente est un protocole qui ne correspond pas au port. Chaque listener du broker a son propre port et son propre protocole. Demandez à vos administrateurs Kafka quel port correspond à quel protocole, voir [Protocoles de sécurité](connections.md#security).

Si le test réussit mais que la lecture ou l’envoi échoue, la cause tient souvent aux adresses des brokers. Le client obtient la liste des brokers auprès du serveur bootstrap, puis se connecte à chaque broker à l’adresse que celui-ci annonce (`advertised.listeners`). Si ces adresses sont internes, par exemple des noms d’hôte Docker ou Kubernetes, elles ne sont pas joignables depuis votre ordinateur. Les adresses sont listées dans le tableau des brokers de l’aperçu du cluster, voir [Aperçu du cluster](topics.md#cluster-overview). Demandez à vos administrateurs un listener joignable depuis l’extérieur.

<a id="dns-tcp"></a>

## DNS et TCP

**Aucun serveur bootstrap n’est joignable en TCP** signifie que l’app n’a pu ouvrir de connexion TCP vers aucune adresse de **Serveurs bootstrap**. Chaque adresse dispose de 5 secondes au maximum. Les détails indiquent la dernière adresse essayée et la raison. `DNS` dans les détails signifie que le nom d’hôte n’a pas pu être résolu.

Vérifiez :

- le nom d’hôte et le port, sans préfixe de protocole comme `https://` ;
- qu’un VPN est activé, si le cluster se trouve dans un réseau privé ;
- qu’un pare-feu ne bloque pas le port ;
- pour un broker dans Docker, que le port est publié sur l’hôte, par exemple `-p 9092:9092`.

Pour vérifier le port en dehors de l’app, exécutez `nc -vz <host> <port>` sur macOS ou Linux, ou `Test-NetConnection <host> -Port <port>` dans PowerShell sur Windows.

**Impossible de se connecter aux brokers** après la réussite de l’étape TCP signifie généralement qu’un broker a fermé la connexion ou qu’il n’a pas pu être joint ensuite. Vérifiez aussi les adresses des brokers, voir [Connexion impossible](troubleshooting.md#cannot-connect).

<a id="tls-errors"></a>

## Erreurs TLS

Erreurs de l’étape **Kafka (TLS, SASL, métadonnées)** lors de la négociation TLS :

- **Sur cette adresse, le broker accepte les connexions sans TLS — vérifiez le port ou choisissez un protocole sans SSL**.
- **Le certificat du broker n’est pas signé par une CA de confiance — vérifiez « CA de confiance »**. Indiquez dans **CA de confiance** le fichier de la CA qui a signé les certificats des brokers. Vos administrateurs peuvent vous fournir ce fichier. Si le champ est vide, seuls les certificats racines du système sont reconnus.
- **Le nom d’hôte du broker ne correspond pas à son certificat**. Connectez-vous avec le nom pour lequel le certificat a été émis, et non avec une adresse IP ou un alias. Dans les environnements de test uniquement, vous pouvez désactiver **Vérifier le nom d’hôte**.
- **Le broker a refusé le certificat client**. Le broker exige un certificat client, ou il ne fait pas confiance à celui que vous utilisez. Vérifiez que **Certificat client (mTLS)** est activé et que le certificat a été émis pour ce cluster. Voir [Certificat client (mTLS)](connections.md#mtls).
- **Erreur de connexion TLS** — un autre problème TLS. Dépliez **Détails** pour connaître la cause.

Si vous avez choisi un protocole sans SSL et voyez quand même une erreur TLS, le port correspond probablement à un listener TLS. Les détails indiquent alors `connecting to a SSL listener?`. Choisissez `SSL` ou `SASL_SSL`, ou utilisez le port d’un listener sans TLS.

Les problèmes de fichiers de certificats sont signalés à l’étape **Configuration**, avant que l’app ne se connecte. Le message commence par le nom du champ : CA de confiance, certificat client ou clé client. Cas courants :

- Le chemin n’est pas absolu ou le fichier ne peut pas être lu. Saisissez le chemin complet ou choisissez le fichier avec **Parcourir…**.
- **Clé client : la clé est chiffrée — indiquez le mot de passe de la clé**, **Clé client : mot de passe de la clé incorrect**, **Certificat client : mot de passe du magasin PKCS\#12 incorrect** ou **Mot de passe du magasin JKS incorrect (ou fichier endommagé)** — saisissez le bon mot de passe dans le formulaire.
- **La clé client ne correspond pas au certificat client** — le fichier de clé appartient à un autre certificat.
- Le certificat client a expiré ou n’est pas encore valide — le message indique la date. Demandez un nouveau certificat.
- Le magasin PKCS#12 utilise un algorithme ancien, ou le magasin est au format JCEKS. Convertissez le fichier comme indiqué dans [Formats de certificats](connections.md#certificate-formats).

<a id="sasl-errors"></a>

## Erreurs SASL

**Échec de l’authentification** signifie que le broker n’a pas accepté l’authentification SASL. Vérifiez :

- le nom d’utilisateur et le mot de passe. Lorsque vous modifiez une connexion enregistrée, un champ de mot de passe vide conserve le mot de passe enregistré. Saisissez de nouveau le mot de passe si vous n’êtes pas sûr qu’il soit correct ;
- **Mécanisme SASL**. Il doit s’agir du mécanisme pour lequel votre utilisateur a été créé. Les identifiants SCRAM sont créés séparément pour `SCRAM-SHA-256` et `SCRAM-SHA-512`, et les utilisateurs `PLAIN` sont encore configurés à part ;
- le protocole : `SASL_SSL` et `SASL_PLAINTEXT` utilisent généralement des ports différents.

**Le mot de passe SASL de la connexion n’est pas défini** — aucun mot de passe n’est enregistré pour cette connexion, par exemple après un import. Ouvrez la connexion, saisissez le mot de passe et enregistrez. Voir [Mots de passe](connections.md#passwords).

**Impossible d’accéder au trousseau du système** — l’app ne peut pas lire le mot de passe dans le trousseau du système. Sur macOS, autorisez l’accès lorsque le Trousseau le demande. Sur Linux, vérifiez que GNOME Keyring, KWallet ou un autre fournisseur Secret Service tourne dans votre session.

**Droits insuffisants pour effectuer l’opération** est un cas différent : l’authentification a réussi, mais l’utilisateur Kafka n’a pas les droits (ACL) nécessaires pour cette opération, par exemple lire un topic ou un groupe. Demandez ces droits à vos administrateurs. Notez que Kafka ne signale aucune erreur pour les topics que vous n’êtes pas autorisé à décrire : ils sont simplement absents de la liste des topics.

<a id="timeouts"></a>

## Délais dépassés

**Délai d’attente de l’opération dépassé** signifie que le broker n’a pas répondu à temps. Les causes typiques sont un réseau ou un VPN lent, un cluster surchargé, ou des adresses de brokers injoignables, voir [Connexion impossible](troubleshooting.md#cannot-connect).

Un problème TLS ou SASL peut ressembler à un dépassement de délai. L’app tente de trouver la cause réelle et l’affiche à la place, mais ce n’est pas toujours possible. Dépliez **Détails** : des messages comme `SSL handshake failed` ou `SASL authentication error` indiquent la cause.

Si le réseau est simplement lent, augmentez **Délai de requête, ms** dans le formulaire de connexion. La valeur par défaut est `30000` ms. Cette valeur s’applique aux requêtes vers les brokers, à l’envoi de messages et à la lecture. Le test de connexion utilise ses propres limites (5 secondes pour chaque adresse TCP et 10 secondes pour l’étape Kafka) : une valeur plus élevée ne change donc rien au test.

<a id="read-only-blocked"></a>

## Action indisponible ou refusée

Si un bouton est masqué ou désactivé, ou si une action est refusée, vérifiez les causes suivantes :

- La connexion est en lecture seule. Elle est signalée par une icône de cadenas à côté de son nom. Les boutons qui modifient le cluster sont masqués ou désactivés. Si une telle requête parvient malgré tout au cœur de l’app, par exemple après l’activation du mode alors qu’un écran était ouvert, l’erreur indique que la connexion est en lecture seule. Pour apporter des modifications, ouvrez la connexion, décochez **Lecture seule** et enregistrez. Voir [Lecture seule](connections.md#read-only).
- La connexion est prod. Les modifications vous demandent de saisir le nom de la connexion. Tant que le nom ne correspond pas, l’action n’est pas effectuée. Voir [Environnements et prod](connections.md#environments).
- Le consumer group a des membres actifs. Les offsets ne peuvent être réinitialisés et un groupe ne peut être supprimé que s’il n’a aucun membre. Voir [Réinitialiser les offsets](groups.md#reset-offsets).
- Le paramètre est géré par votre organisation. Une politique peut fixer la langue de l’interface et la recherche de mises à jour, qui ne peuvent alors pas être modifiées dans l’app. Voir [Paramètres définis par votre organisation](getting-started.md#managed-settings).
- Kafka a refusé l’opération avec **Droits insuffisants pour effectuer l’opération**. C’est une question de droits dans le cluster, pas de paramètres de Glafira. Voir [Erreurs SASL](troubleshooting.md#sasl-errors).

<a id="logs"></a>

## Journaux

L’app écrit un journal de son fonctionnement et des erreurs du client Kafka. Le dossier exact est affiché sur l’écran **À propos**, dans **Journaux :** Par défaut :

- macOS : `~/Library/Logs/io.github.vvnovg.glafira.kafka`
- Windows : `%LOCALAPPDATA%\io.github.vvnovg.glafira.kafka\logs`
- Linux : `~/.local/share/io.github.vvnovg.glafira.kafka/logs`

Le fichier actuel est `app.log`. Lorsqu’il atteint 5 MB, il est renommé en `app.1.log`, et les fichiers plus anciens deviennent `app.2.log`, etc. L’app conserve jusqu’à cinq fichiers et supprime le plus ancien.

Les mots de passe ne sont pas écrits dans les journaux. Les adresses des brokers et les textes des erreurs Kafka le sont. Inutile de rassembler les fichiers à la main : l’archive de diagnostic les inclut, voir [Archive de diagnostic](updates.md#diagnostics).

<a id="support"></a>

## Contacter l’assistance

Avant de contacter l’assistance, lancez **Tester la connexion** et notez l’étape qui échoue. Envoyez ensuite la demande à la personne qui assure le support de Glafira pour vous, par exemple vos administrateurs. Joignez :

- l’archive de diagnostic, voir [Archive de diagnostic](updates.md#diagnostics). Elle contient la version, le build, les journaux et les profils de connexion sans mots de passe ;
- ce que vous avez fait, ce que vous attendiez et ce qui s’est passé, étape par étape ;
- le texte complet de l’erreur, avec le contenu de **Détails**. Vous pouvez sélectionner et copier ce texte ;
- une capture d’écran, si le problème concerne l’interface ;
- l’heure à laquelle l’erreur s’est produite, pour la retrouver dans les journaux.

Si vous ne pouvez pas créer l’archive, indiquez les valeurs **Version** et **Build** de l’écran **À propos** et joignez les fichiers journaux du dossier des journaux.

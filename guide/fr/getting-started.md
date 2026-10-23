# Premiers pas

Glafira for Apache Kafka est un client de bureau pour les clusters Kafka. Il permet de parcourir les topics et les consumer groups, de lire et d’envoyer des messages et de réinitialiser des offsets. Ce chapitre traite de l’installation, du premier lancement, d’un Kafka local pour s’exercer et de la langue de l’interface.

<a id="install-macos"></a>

## Installation sur macOS

Glafira fonctionne sur macOS 11 ou version ultérieure. Téléchargez le programme d’installation depuis la page des versions : https://github.com/vvnovg/glafira-kafka-releases/releases

Choisissez le fichier adapté à votre Mac : `aarch64` pour Apple silicon (M1 et ultérieurs), `x64` pour Intel.

1. Ouvrez le fichier `.dmg`.
2. Faites glisser Glafira for Apache Kafka dans le dossier « Applications ».
3. Lancez l’app depuis « Applications » ou le Launchpad.

Vous pouvez aussi utiliser le programme d’installation `.pkg`. Il place l’app dans `/Applications` pour tous les utilisateurs.

Les builds sont signés par Apple et notarisés : l’app s’ouvre donc tout de suite, sans avertissement de sécurité.

Si macOS indique malgré tout que l’app est endommagée ou qu’elle ne peut pas être vérifiée, le téléchargement est corrompu : téléchargez à nouveau le `.dmg` ou le `.pkg` depuis la page des versions.

<a id="install-windows"></a>

## Installation sur Windows

Téléchargez le programme d’installation `.msi` (`x64`) depuis la page des versions : https://github.com/vvnovg/glafira-kafka-releases/releases

1. Exécutez le fichier `.msi`.
2. Confirmez l’installation dans la fenêtre de Windows. L’app est installée pour tous les utilisateurs de l’ordinateur : les droits d’administrateur sont donc nécessaires.
3. Lancez Glafira for Apache Kafka depuis le menu Démarrer.

Les builds bêta ne sont pas encore signés. Si SmartScreen bloque le programme d’installation, cliquez sur « Informations complémentaires », puis sur « Exécuter quand même ». Vous pouvez aussi débloquer le fichier dans PowerShell avant de l’exécuter :

```powershell
Unblock-File -Path "$env:USERPROFILE\Downloads\Glafira_for_Apache_Kafka_*_x64_en-US.msi"
```

<a id="install-linux"></a>

## Installation sur Linux

Des builds sont disponibles pour `x86_64` dans trois formats. Téléchargez celui qui convient à votre distribution depuis la page des versions : https://github.com/vvnovg/glafira-kafka-releases/releases

- Debian, Ubuntu et dérivées : `sudo apt install ./<file>.deb`
- Fedora, RHEL et dérivées : `sudo dnf install ./<file>.rpm`
- Toute distribution : rendez l’AppImage exécutable avec `chmod +x <file>.AppImage` et lancez-la.

Après l’installation d’un paquet, lancez l’app depuis le menu des applications ou exécutez `glafira-kafka` dans un terminal.

Glafira enregistre les mots de passe dans le Secret Service (par exemple GNOME Keyring ou KWallet). Vérifiez que l’un d’eux tourne dans votre session. Sinon, l’app ne peut pas enregistrer les mots de passe des connexions.

<a id="first-run"></a>

## Premier lancement

La fenêtre comporte deux parties. À gauche se trouve le panneau **Connexions**. À droite, des onglets : chaque écran ouvert, par exemple une liste de topics ou le Producer, a son propre onglet.

Tant qu’il n’y a aucune connexion, le panneau affiche une courte aide et la zone des onglets propose d’ajouter une connexion. Pour commencer :

1. Cliquez sur `+` ( **Nouvelle connexion** ) dans l’en-tête du panneau **Connexions**.
2. Renseignez les champs **Nom** et **Serveurs bootstrap**, par exemple `localhost:9092`.
3. Cliquez sur **Tester la connexion** pour vérifier les paramètres, puis sur **Enregistrer**.

Après l’enregistrement, l’aperçu du cluster s’ouvre. Sous chaque connexion du panneau figurent les liens **Aperçu du cluster**, **Topics**, **Consumer groups**, **Producer** et **Session consumer**. Tous les paramètres de connexion sont détaillés dans [Connexions](connections.md#create).

En bas du panneau se trouvent les liens **Paramètres**, **Soutenir le projet**, **Aide** et **À propos**. Le signe `?` à côté d’un champ affiche une courte aide à son sujet. Appuyez sur `F1` n’importe où dans la fenêtre pour ouvrir le chapitre du guide consacré à l’écran actuel. Sur macOS, le guide est aussi accessible depuis le menu « Aide ».

Si vous utilisiez Kafka Explorer (l’ancien nom de l’app), Glafira copie au premier lancement ses connexions, modèles et historique d’envoi. Les mots de passe sont eux aussi copiés depuis le trousseau du système. Rien de ce qui existe déjà n’est écrasé, et les anciennes données ne sont pas supprimées. macOS peut demander une fois l’accès aux anciens éléments du trousseau.

<a id="local-cluster"></a>

## Kafka local pour s’exercer

Pour essayer l’app sans cluster partagé, lancez Kafka sur votre ordinateur dans Docker. Un seul broker suffit :

```bash
docker run -d --name kafka -p 9092:9092 apache/kafka:3.9.0
```

Créez ensuite une connexion avec le serveur bootstrap `localhost:9092`, le protocole `PLAINTEXT` et l’environnement `local`. Quand vous n’avez plus besoin du broker, supprimez-le avec `docker rm -f kafka`.

L’aide du panneau **Connexions** vide mentionne `docker compose -f dev/docker-compose.yml up -d`. Ce fichier fait partie du code source de Glafira. Si vous disposez du code source, cette commande lance un broker avec trois listeners :

- `localhost:9092` — `PLAINTEXT`.
- `localhost:9094` — `SASL_PLAINTEXT` avec `PLAIN` (utilisateurs `tester` / `tester-secret` et `admin` / `admin-secret`) et SCRAM (`scram-user` / `scram-secret` pour `SCRAM-SHA-256` et `SCRAM-SHA-512`).
- `localhost:9095` — `SSL` avec certificats client (mTLS). Les certificats de test se trouvent dans `dev/certs/out`.

Ces utilisateurs et certificats sont réservés aux tests locaux.

<a id="language"></a>

## Langue de l’interface

L’interface est disponible en anglais, russe, allemand, français, espagnol, italien, polonais, portugais (Brésil), chinois (simplifié et traditionnel), japonais et coréen. Toutes les langues sont gratuites.

Pour changer de langue :

1. Cliquez sur **Paramètres** en bas du panneau de gauche.
2. Choisissez une langue dans **Langue de l’interface**.

Le changement s’applique immédiatement, y compris à ce guide et au menu macOS. **Langue du système** (la valeur par défaut) suit la langue de votre système d’exploitation. Si cette langue ne figure pas dans la liste, l’interface est en anglais.

<a id="managed-settings"></a>

## Paramètres définis par votre organisation

En entreprise, les administrateurs peuvent définir certaines options pour tous les utilisateurs au moyen d’une politique. Ces options ne peuvent pas être modifiées dans l’app. Une politique peut :

- fixer la langue de l’interface — la liste des langues dans **Paramètres** est alors indisponible, avec une mention indiquant que la langue est définie par votre organisation ;
- désactiver la recherche de mises à jour ou la rediriger vers un miroir interne.

Pour savoir si une politique s’applique, ouvrez **À propos** et consultez **Politique de l’organisation**. Si la politique contient une erreur, celle-ci est affichée à cet endroit. Pour toute question sur la politique, contactez vos administrateurs. Les mises à jour sont décrites dans [Mises à jour et diagnostic](updates.md).

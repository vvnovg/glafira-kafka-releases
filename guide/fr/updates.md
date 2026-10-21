# Mises à jour et diagnostic

Ce chapitre décrit comment mettre à jour l’app, l’écran **À propos**, l’archive de diagnostic et comment soutenir le projet.

<a id="updates"></a>

## Mises à jour

Au démarrage, l’app vérifie une fois si une nouvelle version est disponible. Si c’est le cas, une notification au-dessus des onglets indique sa version. **Détails** ouvre l’écran **À propos**. Si la vérification échoue, par exemple sans accès à Internet, aucun message ne s’affiche.

Pour vérifier manuellement :

1. Ouvrez **À propos** en bas du panneau de gauche.
2. Dans **Mises à jour**, cliquez sur **Rechercher des mises à jour**.
3. Si une nouvelle version est disponible, l’écran affiche son numéro et les notes de version. Cliquez sur **Installer et redémarrer**.

L’app télécharge la mise à jour, vérifie sa signature, l’installe et redémarre. Un téléchargement interrompu par un problème réseau est relancé automatiquement, jusqu’à trois tentatives au total. Si les trois tentatives échouent, l’app signale **Le téléchargement de la mise à jour a été interrompu — vérifiez le réseau et réessayez**. Si la vérification de la signature échoue, l’app signale **La signature de la mise à jour n’a pas pu être vérifiée — la mise à jour n’a pas été installée**.

L’installation peut demander un mot de passe administrateur, car l’app est installée pour tous les utilisateurs de l’ordinateur : par le programme d’installation `.pkg` sur macOS, le programme d’installation `.msi` sur Windows, ou un paquet `.deb` ou `.rpm` sur Linux. Les utilisateurs sans droits d’administrateur ne peuvent pas mettre à jour une telle installation.

Si des messages sont en cours de lecture, si une session consumer tourne ou si un lot est en cours d’envoi, l’app vous demande d’abord confirmation et indique combien de sessions sont actives. Cliquez sur **Arrêter les sessions et mettre à jour** pour les arrêter et continuer, ou sur **Annuler**. Les connexions, les paramètres et les autres données sont conservés après une mise à jour.

Les mises à jour sont téléchargées depuis la page des versions sur GitHub. Si la vérification échoue avec **Impossible de rechercher des mises à jour**, vérifiez votre réseau ou votre proxy. **Les mises à jour ne sont pas configurées dans ce build** signifie que ce build a été produit sans la clé du système de mise à jour, par exemple un build de développement ou de test. Dans ce cas, téléchargez les nouvelles versions depuis la page des versions : https://github.com/vvnovg/glafira-kafka-releases/releases

Une organisation peut désactiver la recherche de mises à jour au moyen d’une politique, ou la rediriger vers un miroir interne. Lorsque la recherche est désactivée, la section **Mises à jour** affiche **Les mises à jour sont gérées par votre organisation** après un clic sur **Rechercher des mises à jour**. Comme **Les mises à jour ne sont pas configurées dans ce build**, ce message n’apparaît qu’après le clic ; dans ces cas, la vérification au démarrage n’affiche rien.

<a id="about"></a>

## À propos

Ouvrez **À propos** en bas du panneau de gauche. Sur macOS, cet écran est aussi accessible depuis le menu « Aide », sous **À propos de Glafira**.

L’écran affiche une courte description de l’app, ainsi que :

- **Version** — le numéro de version, et **Build** — l’identifiant du code source à partir duquel ce build a été produit. Indiquez les deux dans une demande d’assistance.
- **Système** — le système d’exploitation et l’architecture du processeur.
- **Licence** — Apache 2.0 ; le lien ouvre le texte de la licence dans le navigateur.
- **Politique de l’organisation** — **définie** ou **aucune**. Les erreurs de la politique sont listées en dessous.
- **Données :** — le dossier contenant les profils de connexion, les modèles, l’historique et les paramètres.
- **Journaux :** — le dossier des fichiers journaux. Voir [Journaux](troubleshooting.md#logs).

Les sections **Mises à jour** et **Diagnostic** se trouvent aussi sur cet écran.

<a id="diagnostics"></a>

## Archive de diagnostic

L’archive de diagnostic rassemble ce dont l’assistance a besoin pour comprendre un problème. Pour la créer :

1. Ouvrez **À propos**.
2. Dans **Diagnostic**, cliquez sur **Collecter le diagnostic**. Sur macOS, vous pouvez aussi choisir **Collecter le diagnostic…** dans le menu « Aide ».
3. Choisissez où enregistrer le fichier. Le nom proposé est `diagnostics-<date>.zip`.

Une fois l’archive enregistrée, l’écran affiche son chemin. Le fichier ZIP contient :

- `info.json` — la version et le build de l’app, le système d’exploitation, les chemins des dossiers de données et de journaux, ainsi que l’état de la politique ;
- `connections.json` — les profils de connexion : noms, environnements, serveurs bootstrap, protocoles, noms d’utilisateur SASL, chemins des fichiers de certificats, propriétés supplémentaires et délais ;
- `logs/` — les fichiers journaux de l’app.

L’archive ne contient aucun mot de passe : ils sont conservés dans le trousseau du système, et les propriétés supplémentaires contenant des mots de passe ou des secrets sont refusées à l’enregistrement d’une connexion. Les fichiers de certificats n’y figurent pas non plus. Les noms d’hôte, noms d’utilisateur et chemins de fichiers sont stockés en clair, et les journaux contiennent les adresses des brokers issues des messages d’erreur Kafka. Parcourez l’archive avant de l’envoyer hors de votre entreprise.

L’app n’envoie rien d’elle-même. Joignez vous-même l’archive à votre demande d’assistance, voir [Contacter l’assistance](troubleshooting.md#support).

<a id="support"></a>

## Soutenir le projet

Glafira est gratuite et open source sous licence Apache 2.0. Les dons font vivre le projet. **Soutenir le projet** en bas du panneau de gauche, sur l’écran **À propos** et, sous macOS, dans le menu « Aide » ouvre la page de soutien dans le navigateur : https://github.com/vvnovg/glafira-kafka-releases/blob/main/SUPPORT.md

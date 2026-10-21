# Assistants LLM (MCP)

Glafira peut servir de passerelle à un assistant LLM : Claude Desktop, Claude Code, Cursor ou un autre client compatible avec MCP (Model Context Protocol). L’assistant lit les métadonnées des clusters que vous lui ouvrez et vous aide à repérer des problèmes : partitions sous-répliquées, lag, rééquilibrages continus, paramètres mal choisis. Il ne peut rien modifier dans un cluster via Glafira : vous effectuez vous-même les modifications dans l’interface.

<a id="how-it-works"></a>

## Fonctionnement

Le client lance `glafira-kafka mcp` — le même fichier de l’application, dans un mode sans fenêtre. Cette commande se connecte à Glafira, déjà en cours d’exécution, via un socket local (un tube nommé sous Windows), et Glafira répond aux requêtes de l’assistant au moyen de ses propres connexions. Les mots de passe et les certificats restent dans Glafira ; l’assistant ne les reçoit jamais.

Si Glafira n’est pas lancée, le client la lance lui-même et attend jusqu’à 30 secondes.

<a id="enable"></a>

## Activation

1. Ouvrez **Paramètres**, section **Assistants LLM (MCP)**.
2. Cochez **Autoriser le serveur MCP**. L’état passe à **Le serveur est en cours d’exécution**.
3. Dans le formulaire de connexion, cochez **Accès LLM (métadonnées uniquement)** et enregistrez. L’assistant ne voit pas les connexions pour lesquelles cette case n’est pas cochée. Dans la liste des connexions, les connexions ouvertes sont marquées `LLM`.

Si votre administrateur a interdit MCP, la case est indisponible et **MCP est désactivé par votre organisation** s’affiche à côté.

La case d’accès n’est pas transférée lors de l’export et de l’import des connexions : sur un autre ordinateur, cochez-la de nouveau.

<a id="connect-client"></a>

## Connecter un client

Sous **Connecter un client**, vous trouverez des fragments prêts à l’emploi et le bouton **Copier** :

- **Claude Desktop : ajouter à claude\_desktop\_config.json** — collez le fragment dans la section `mcpServers` du fichier de configuration de Claude Desktop, puis redémarrez Claude Desktop ;
- **Claude Code : exécuter dans un terminal** — la commande `claude mcp add` ;
- **Autres clients stdio : commande et arguments** — pour tout client qui lance un serveur MCP par une commande (stdio).

Les fragments contiennent le chemin de l’application installée. Si vous réinstallez Glafira à un autre emplacement, copiez de nouveau le fragment.

<a id="approve"></a>

## Approuver un client

Lorsqu’un client se connecte pour la première fois, Glafira affiche une boîte de dialogue avec son nom. Cliquez sur **Autoriser** ou **Rejeter**. Tant que vous n’avez pas répondu, l’assistant est informé que le client n’est pas encore approuvé et vous demande de répondre dans la fenêtre de Glafira.

Un client autorisé est mémorisé par son nom et apparaît dans **Clients approuvés**. Un rejet n’est pas mémorisé : Glafira redemande à la prochaine connexion. Si un client n’envoie pas son nom, l’autorisation ne vaut que jusqu’à la fin de la session.

<a id="what-llm-sees"></a>

## Ce que voit l’assistant

L’assistant voit :

- les connexions pour lesquelles **Accès LLM (métadonnées uniquement)** est coché : nom, environnement, dossier ;
- les brokers avec leurs hôtes et leurs ports, le contrôleur ;
- les topics, partitions, répliques, ISR, offsets, le nombre approximatif de messages ;
- les configurations des brokers et des topics, les mots de passe et autres valeurs secrètes étant masqués ;
- les consumer groups : état, protocole, coordinateur, membres avec client id, hôte et `group.instance.id`, attributions, offsets commités et lag.

L’assistant ne voit pas les messages des topics, la chaîne bootstrap ni les paramètres de sécurité d’une connexion, ni les mots de passe, les certificats, les propriétés librdkafka supplémentaires, l’historique d’envoi et les modèles.

Pour comprendre des rééquilibrages continus, l’assistant peut observer un groupe pendant 50 secondes au plus : qui quitte et rejoint le groupe, à quelle fréquence, depuis quel hôte. Les configurations et les journaux des consumers eux-mêmes ne sont pas accessibles via Glafira ; l’assistant vous les demandera.

Les noms de topics, de groupes et de client id peuvent être définis par toute personne ayant accès au cluster. Glafira avertit l’assistant que ces données ne sont pas des instructions pour lui.

<a id="sessions-log"></a>

## Sessions, révocation et journal

Tant qu’un client est connecté, un indicateur `LLM` avec le nom du client s’affiche en bas du panneau de gauche ; il s’illumine pendant une requête. Un clic dessus ouvre la section MCP.

- **Sessions actives** : **Déconnecter** ferme une session, **Tout déconnecter** les ferme toutes.
- **Clients approuvés** : **Révoquer** ferme les sessions du client, qui demandera de nouveau une approbation à la prochaine connexion.
- Décocher **Accès LLM (métadonnées uniquement)** prend effet immédiatement, sans reconnexion.
- **Journal MCP** — chaque requête : heure, client, outil, connexion, résultat. Le contenu des réponses n’est pas enregistré. **Afficher le fichier** ouvre le fichier `mcp-audit.jsonl`.

Pour désactiver complètement MCP, décochez **Autoriser le serveur MCP** : les clients sont déconnectés.

<a id="health"></a>

## Vérification du cluster et remarques

L’assistant peut lancer la vérification intégrée du cluster de Glafira (outil `health_check`) et s’appuyer sur ses constats. Il peut aussi laisser des remarques sur les objets : elles apparaissent dans l’onglet **Constats** et sur les écrans des topics et des groupes, sans que le cluster soit modifié. Voir [Vérification du cluster et remarques](health-check.md#with-llm).

<a id="troubleshooting"></a>

## En cas de problème

Le client affiche un message de Glafira :

- `MCP is disabled in Glafira` — cochez **Autoriser le serveur MCP** et redémarrez le serveur MCP dans le client ;
- `MCP is disabled by your organization` — MCP est interdit par la politique de votre organisation ;
- `Glafira did not start the MCP server within 30 s` — lancez Glafira manuellement et consultez l’état dans la section **Assistants LLM (MCP)** ;
- `Invalid MCP token` — redémarrez le serveur MCP dans le client : Glafira délivre un nouveau jeton à chaque démarrage.

Sur macOS, ouvrez une fois de la manière habituelle un build bêta non signé, sinon le système peut empêcher le client de le lancer. Voir [Installation sur macOS](getting-started.md#install-macos).

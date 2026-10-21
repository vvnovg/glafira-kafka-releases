# Vérification du cluster et remarques

Glafira vérifie un cluster à l’aide de règles intégrées et affiche les problèmes trouvés à côté des objets : dans l’arborescence des connexions, dans les listes de topics et de groupes et sur les écrans d’un topic ou d’un groupe. Un assistant LLM peut compléter ces constats par ses propres remarques. La vérification ne fait que lire le cluster et n’y change rien.

<a id="run"></a>

## Lancer une vérification

Ouvrez **Aperçu du cluster** ou **Constats** d’une connexion et cliquez sur **Vérifier le cluster**. Une vérification dure jusqu’à 45 secondes : Glafira lit les brokers, les partitions, les configurations de topics (jusqu’à 2000 topics) et les consumer groups (jusqu’à 500). S’il y a plus de topics ou de groupes, si quelque chose n’a pas pu être lu ou si les 45 secondes sont écoulées, le résultat est marqué **vérifié partiellement** et le constat **Vérification incomplète** indique ce qui n’a pas été lu.

Glafira n’interroge pas le cluster d’elle-même : une vérification ne se lance que lorsque vous cliquez sur le bouton ou qu’un assistant LLM la demande. Le résultat est conservé jusqu’à la vérification suivante.

<a id="rules"></a>

## Ce qui est vérifié

- Brokers et partitions : absence de contrôleur, partitions indisponibles et sous-répliquées, cluster à un seul broker, déséquilibre des leaders, leaders hors de la réplique préférée.
- Durabilité des topics : facteur de réplication 1, `min.insync.replicas` supérieur au facteur de réplication, `min.insync.replicas` égal au facteur de réplication, `min.insync.replicas` à 1, élection de leader non propre.
- Configuration des brokers : création automatique de topics, facteur de réplication par défaut à 1, `min.insync.replicas` par défaut à 1, réglages importants qui diffèrent d’un broker à l’autre.
- Consumer groups : groupes vides avec du lag, groupes en rééquilibrage, membres sans partition, lag qui a augmenté depuis la vérification précédente.

<a id="severity"></a>

## Gravité et environnement

Chaque constat a une gravité : **Critique**, **Avertissement** ou **Information**. Elle dépend de l’environnement de la connexion. Par exemple, un facteur de réplication de 1 est critique en prod, un avertissement en stage et n’est pas affiché en dev, test et local. L’environnement se règle dans le formulaire de connexion.

<a id="tab"></a>

## L’onglet « Constats »

Dans l’onglet **Constats**, les constats sont regroupés par règle ; chacun explique le problème et ce qu’il faut faire. Filtrez-les par **Gravité**, **Origine** et **Objet**, ou recherchez par nom. Le nom de l’objet ouvre son écran.

Un badge avec un nombre dans l’arborescence des connexions et dans les listes de topics et de groupes indique combien de constats et de remarques LLM ouvertes de gravité **Critique** ou **Avertissement** concernent l’objet. Survolez le badge pour voir leurs titres.

<a id="hide"></a>

## Masquer des constats

Si un constat est attendu pour un objet précis, par exemple un topic de test avec un facteur de réplication de 1 en prod, cliquez sur **Masquer pour cet objet**. Un constat masqué n’est pas compté dans les badges et reste masqué lors des vérifications suivantes. Pour voir les constats masqués, cochez « Afficher les masqués » ; vous y trouverez **Rétablir**.

<a id="notes"></a>

## Remarques LLM

Un assistant LLM connecté par MCP (voir [Assistants LLM (MCP)](llm-mcp.md#how-it-works)) peut laisser une remarque sur le cluster, un broker, un topic ou un groupe. Les remarques apparaissent dans **Remarques LLM** et sur les écrans des objets, signées du nom du client. Le texte est affiché tel quel, sans mise en forme ni liens.

- **Résolu** et **Sans objet** ferment une remarque ; les remarques fermées restent dans une section repliée.
- **Supprimer** supprime une remarque, **Supprimer toutes les remarques LLM** supprime toutes les remarques de la connexion.
- Une remarque est supprimée automatiquement 7 jours après sa création. Une connexion peut avoir au plus 200 remarques ouvertes.

<a id="with-llm"></a>

## Avec un assistant LLM

L’assistant lance la même vérification avec l’outil `health_check` et voit les mêmes constats, y compris les constats masqués, qui sont signalés. Si la dernière vérification date de moins de 5 minutes, l’assistant reçoit son résultat sans interroger à nouveau le cluster. Demandez par exemple : « Vérifie l’état du cluster et laisse des remarques sur les problèmes les plus importants. »

# Engineering AI Workflow : Méthode en Étoile à 4 Étapes pour l'Ingénierie

> **[English](./README.md)** | **[简体中文](./README_ZH.md)** | **[Français](./README_FR.md)** | **[Русский](./README_RU.md)**
>
> **Auteur : Jiawei Li** | **Licence : MIT**

## Pourquoi cette organisation ?

Dans les métiers de l'ingénierie physique et industrielle — mécatronique, automatique, instrumentation, systèmes embarqués ou architecture logicielle bas niveau — l'utilisation quotidienne de l'IA se heurte souvent à deux écueils très concrets :

1. **L'enlisement dans une fenêtre unique** : On copie-colle du code et des spécifications au fil d'une longue conversation. Au bout de cinq ou six échanges, le contexte se dégrade : corriger un détail casse deux fonctions qui marchaient. Par ailleurs, si l'on demande à un modèle dit « très avancé » de coder directement, il a tendance à faire du zèle et transforme un module sobre en une véritable **usine à gaz** (ajout intempestif de scripts CI, de code défensif lourd et de tests dans tous les sens).
2. **Le piège des boucles Multi-Agents automatiques** : On laisse une IA « Développeur » et une IA « Critique » dialoguer toutes seules en boucle fermée. Sans ingénieur au milieu pour servir de coupe-circuit, l'IA qui code applique aveuglément la moindre remarque de l'IA qui critique. En quelques tours, une conception propre se retrouve noyée sous les couches d'abstraction.

Ce dépôt décrit une **méthode en étoile à 4 étapes (1 pivot cloud permanent + 2 agents locaux sans mémoire)**, indépendante de tout modèle ou projet particulier.

Son objectif est pragmatique : **permettre à un ingénieur doté d'un bon sens physique et métier de transformer une intuition de départ en un système précis, sobre et bien intégré — sans rédiger soi-même de longs prompts techniques et sans devoir éplucher des pages de rapports d'audit.**

---

## 1. Répartition des Rôles : 1 Pivot Cloud + 2 Agents Locaux

Dans ce fonctionnement, **l'humain n'échange qu'avec l'IA n°1**. Les IA n°2 et n°3 sont des agents locaux commandés à partir de la n°1. Elles ne dialoguent jamais entre elles, et chaque rôle repose sur un **comportement type** ainsi qu'un **environnement d'exécution strict** :

| Rôle         | Profil de comportement                                       | Environnement d'exécution      | Gestion du contexte                                          | Rôle exact                                                   |
| :----------- | :----------------------------------------------------------- | :----------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| **L'Humain** | **Ingénieur principal / Garant du cap**                      | Poste de travail local         | Interventions ponctuelles de haut niveau                     | Apporte l'intuition initiale ; recadre l'IA n°1 si elle dérive ; décide de ce qu'on garde et de ce qu'on écarte. |
| **IA n°1**   | **L'IA Conciliante**<br>(Pivot central / Rédacteur de consignes / Filtre) | **IA conversationnelle Cloud** | **Même fenêtre du début à la fin**<br>(Ne reçoit que des synthèses : faible consommation de contexte et maintien de la cohérence globale) | Aide à évaluer et structurer l'idée de départ ; rédige les consignes pour les IA n°2 et n°3 ; résume en langage clair et trie les retours de l'IA n°3. |
| **IA n°2**   | **L'IA Exécutante**<br>(Exécutant pur)                       | **Agent Local**                | **Nouvelle session vierge à chaque passe**<br>(Évite toute pollution par l'historique) | Exécute sans bavarder. Modifie les fichiers, lance les validations et appelle les outils locaux selon la consigne, puis livre un bref compte rendu. |
| **IA n°3**   | **L'IA Intransigeante**<br>(Auditeur sévère)                 | **Agent Local**                | **Nouvelle session vierge à chaque audit**<br>(Évite de s'enfermer dans un minimum local) | Passe tout au crible : de l'architecture globale à l'implémentation et aux cas limites. **Interdiction de modifier les fichiers : elle ne livre qu'un rapport.** |

> **Précision : quelle différence entre « IA conversationnelle Cloud » et « Agent Local » ?**
> * **IA conversationnelle Cloud (IA n°1)** : Une interface de discussion classique (navigateur Web ou fenêtre de chat pure) qui **n'a aucun accès direct à votre disque dur**. Elle ne peut ni parcourir vos dossiers ni lancer de commandes sur votre machine ; vous échangez uniquement par texte avec elle (en lui téléversant manuellement un document ou fichier clé seulement lorsque c'est indispensable).
> * **Agent Local (IA n°2 et n°3)** : Il ne s'agit **pas** de faire tourner un petit modèle sur la carte graphique du PC, mais d'utiliser les **logiciels agents de bureau (Desktop Apps), extensions d'IDE ou outils en ligne de commande (CLI)** installés directement sur votre poste. Tout en s'appuyant sur des modèles de pointe, l'Agent Local a le droit de lire et modifier directement vos fichiers locaux, d'exécuter des commandes système et d'appeler vos compilateurs ou logiciels métiers — sans que vous ayez à copier-coller le code à la main.

---

## 2. Le Cycle en 4 Étapes

### Étape 1 : Développer l'intuition avec l'IA n°1 (Le Pivot Cloud Conciliant)
* On part d'une intuition d'ingénieur — même si tous les détails techniques ne sont pas encore fixés, l'expérience du métier indique clairement ce qui tient la route et ce qu'il faut éviter.
* On soumet cette idée brute à **l'IA n°1** pour qu'elle aide à évaluer la meilleure approche et à compléter la logique.
* On échange jusqu'à obtenir une proposition préliminaire propre et satisfaisante.

### Étape 2 : Rédaction de la consigne par la n°1 et réalisation par l'IA n°2 (Agent Local d'Exécution)
* Une fois le concept arrêté, on demande à **l'IA n°1** : *« On confie la réalisation à l'IA n°2 qui est un pur exécutant. Rédige la consigne pour qu'elle construise ce système en local, en lui demandant de produire à la fin un compte rendu sur ce qu'elle a fait et sur les comportements curieux éventuellement observés au codage ou à l'exécution. »*
* On copie cette consigne dans une **nouvelle session vierge de l'Agent Local (IA n°2)** pour qu'elle modifie les fichiers, compile et lance les vérifications sur le poste.
* Une fois le compte rendu généré par l'IA n°2, on le renvoie dans **la fenêtre de l'IA n°1** afin qu'elle suive l'état réel du projet.

### Étape 3 : Audit global non orienté par l'IA n°3 (Agent Local d'Audit)
* Une fois l'IA n°1 à jour, on lui indique : *« Rédige maintenant une consigne d'audit pour l'IA n°3, qui a un rôle extrêmement sévère et pointilleux. Attention : la consigne ne doit pas être orientée ou limitée aux derniers ajouts (sinon elle va tomber dans un minimum local). Demande-lui un examen complet, depuis les choix d'architecture jusqu'à l'implémentation bas niveau et aux conditions aux limites, avec un rapport complet en sortie. »*
* On ouvre une **nouvelle session vierge de l'Agent Local (IA n°3)**, qui inspecte directement le projet sur la machine et produit son rapport complet.

### Étape 4 : Tri et synthèse par l'IA n°1 (Écarter le superflu, garder l'essentiel)
Nul besoin de lire soi-même le long rapport pointilleux de l'IA n°3. On le copie directement dans **la fenêtre de l'IA n°1** en précisant le contexte : *« Ce rapport provient d'un auditeur purement sévère dont le seul rôle est de chercher des défauts. Fais le tri par rapport à notre conception initiale. »*

Le cycle se poursuit alors simplement :
1. **Mise à jour et filtrage par l'IA n°1** : Fidèle à l'idée de départ, l'IA n°1 identifie ce qui relève de la lourdeur inutile (à jeter) et ce qui touche à de vrais défauts techniques (à conserver), afin d'en extraire uniquement les modifications **les plus précises, les plus sobres et les mieux intégrées**.
2. **Nouvelle consigne d'exécution** : Après validation par l'humain, l'IA n°1 rédige le prompt destiné à l'IA n°2.
3. **Itération** : Exécution dans une **nouvelle session locale IA n°2** $\rightarrow$ retour du compte rendu à l'IA n°1 $\rightarrow$ audit dans une **nouvelle session locale IA n°3** $\rightarrow$ filtrage par l'IA n°1, jusqu'à aboutir à un résultat propre et finalisé.

---

## 3. Le Rôle de l'Humain au Fil du Processus

Outre la discussion initiale de l'Étape 1, l'intervention humaine se concentre sur trois points :

1. **Recadrer l'IA n°1** : Comme l'IA n°1 est l'interlocuteur principal et qu'elle cherche à être agréable, elle peut vite dériver vers la flatterie, l'exagération ou un minimum local. Dès qu'elle s'écarte du sujet, une brève instruction permet de la remettre dans le droit chemin.
2. **Faire résumer et élaguer via l'IA n°1** : L'IA n°1 s'exprimant de façon claire et accessible, on lui fait résumer les rapports techniques pour écarter d'office tout ce qui est lourd ou inutile.
3. **Apporter de nouvelles intuitions en cours de route** : Dès qu'une nouvelle idée ou contrainte physique apparaît pendant le développement, on la transmet simplement à l'IA n°1, qui se charge de la répercuter vers les deux agents locaux.

---

## 4. Pourquoi ces Choix d'Organisation ?

### 1. Pourquoi une IA conversationnelle Cloud pour la n°1, et des Agents Locaux pour les n°2 et n°3 ?
* **Pourquoi l'IA n°1 reste dans le Cloud (sans accès au disque)** : Si l'IA chargée de la réflexion globale avait accès au répertoire local, elle lirait des fichiers annexes et des journaux de compilation à tout-va, ce qui saturerait et polluerait très vite sa fenêtre de contexte. En utilisant une IA conversationnelle en ligne et en ne lui téléversant manuellement que les documents ou fichiers strictement nécessaires, on garde son contexte léger et parfaitement propre.
* **Pourquoi les IA n°2 et n°3 doivent être des Agents Locaux** : En ingénierie, il ne suffit pas de générer du texte : il faut modifier les fichiers du projet, compiler, lancer des scripts de validation et appeler divers logiciels métiers installés sur l'ordinateur. Une simple interface Cloud ne le permet pas — et même les « ordinateurs virtuels dans le Cloud » (sandboxes) sont inutilisables en pratique, car ce ne sont généralement que des conteneurs Linux génériques dépourvus de l'environnement réel, des pilotes et des chaînes d'outils du poste de travail.

### 2. Pourquoi ne pas laisser la même IA (surtout l'IA Intransigeante) coder et auditer ?
* **Le confort de l'auto-relecture** : Une IA qui relit son propre code suit ses propres habitudes ; elle est toujours à l'aise avec ce qu'elle a écrit et passe à côté des défauts de structure.
* **L'excès de code défensif** : Une IA sévère a naturellement tendance à multiplier le texte défensif et les tests anarchiques. Si on la laisse coder, elle alourdit inutilement le projet. **On laisse donc la construction à l'IA n°2, et on cantonne l'IA n°3 au seul rôle d'audit.**

### 3. Pourquoi garder l'IA n°1 dans la même fenêtre, mais repartir à zéro pour la n°2 et la n°3 ?
* **Continuité pour la n°1** : Ne lisant que des synthèses, l'IA n°1 consomme peu de contexte. Même si une compression automatique du contexte intervient sur la durée, cela ne gêne en rien le pilotage global et préserve la continuité de la réflexion.
* **Propreté pour les n°2 et n°3** : Ouvrir une nouvelle session à chaque tâche évite que les erreurs ou compromis précédents ne polluent l'attention de l'agent et ne l'enferment dans un minimum local.
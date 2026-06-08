# Introduction aux LLMs

> **Note de cours** — Durée estimée : 45 à 60 min. Objectif : donner une intuition solide du fonctionnement d'un LLM, sans rentrer dans les maths. Chaque section s'appuie sur un exemple interactif ou visuel. L'arc narratif va du plus simple (compter des paires de mots) au plus complexe (milliards de paramètres alignés sur les préférences humaines).
> 

---

## 1. Ce qu'est un LLM — en une phrase

Un grand modèle de langage est un réseau de neurones entraîné à faire une seule chose : **prédire le mot le plus probable après une séquence de mots donnée**.

C'est tout. Pas de compréhension au sens humain, pas de conscience, pas de base de données de faits. Un calcul de probabilité, répété des milliards de fois sur des milliards de textes, jusqu'à ce que la prédiction devienne remarquablement bonne.

> **Point d'attention** : les LLM sont des fonctions probabilistes, pas déterministes. Avec les mêmes paramètres d'entrée, vous n'obtiendrez pas forcément le même résultat — c'est intentionnel, et nous y reviendrons avec la notion de température.
> 

---

## 2. Le modèle le plus simple possible : le bigramme

Avant de parler de transformers et de milliards de paramètres, partons du modèle le plus naïf qu'on puisse imaginer : le **modèle bigramme**.

Son principe : regarder le corpus de texte, compter combien de fois chaque paire de mots consécutifs apparaît, et en déduire des probabilités. Si dans le corpus "le chat" est suivi de "mange" dans 40 % des cas et de "dort" dans 30 % des cas, le modèle a "appris" ces probabilités. C'est l'intégralité de sa connaissance.

Ce modèle a une limite évidente : il ne regarde qu'**un seul mot en arrière**. Il ignore tout le contexte qui précède. Mais il illustre parfaitement la mécanique fondamentale de tous les LLM — et on peut le faire tourner en direct.

> 🖥️ **WIDGET INTERACTIF — Modèle bigramme sur Candide (Voltaire)**
> 
> 
> *Présenter le widget bigramme. Montrer :*
> 
> - *L'étape "Corpus" : le modèle n'a appris que de ce texte*
> - *L'étape "Bigrammes" : sa connaissance = une table de fréquences*
> - *L'étape "Prédire" : cliquer sur "candide" pour voir la distribution de probabilités*
> - *L'étape "Générer" : jouer avec la température (bas = répétitif, haut = incohérent)*
> 
> *Point clé à souligner : le texte généré sonne vaguement voltairien mais n'a aucun sens. C'est l'argument pour passer à la suite.*
> 

---

## 3. Les limites du bigramme — et pourquoi on en a besoin

Le bigramme échoue sur deux fronts.

**Le contexte distant.** Dans la phrase "Candide marchait seul dans le désert quand il aperçut un village", le bigramme ne peut pas savoir que "il" désigne "Candide" — six mots séparent les deux. Sa fenêtre d'un mot est trop petite.

**La représentation des mots.** Pour le bigramme, "chat" est juste un identifiant numérique dans un dictionnaire. Il ne sait pas que "chat" et "chien" sont plus proches sémantiquement que "chat" et "voiture". Les mots n'ont pas de sens — ils ont des index.

Ces deux limites vont structurer toute la suite du chapitre. On va voir comment les LLM modernes les résolvent, l'une après l'autre.

---

## 4. Du token au vecteur — comment un mot acquiert un sens

### 4.1 Les tokens

Les LLM ne travaillent pas sur des mots entiers, mais sur des **tokens** — des morceaux de mots découpés selon des algorithmes statistiques. "Incroyable" peut devenir ["in", "croyable"]. "ChatGPT" peut rester un seul token ou être découpé.

> 💡 **Illustration suggérée** — *Montrer un exemple de tokenisation sur une phrase française : colorer chaque token d'une couleur différente pour rendre le découpage visible. Par exemple : "le|s| chat|s| man|gent" avec des couleurs alternées.*
> 

demo live sur huggin face:  [https://huggingface.co/spaces/ZurichNLP/subword-tokenization](https://huggingface.co/spaces/ZurichNLP/subword-tokenization)

Ce découpage a une conséquence importante : le modèle ne "voit" pas les lettres individuelles. C'est la source du célèbre bug du comptage de lettres — "combien de R dans strawberry ?" — le modèle ne peut pas compter ce qu'il ne perçoit pas directement. On en a parlé dans le chapitre sur les hallucinations.

### 4.2 Les embeddings — donner un sens géométrique aux mots

Au départ de l'entraînement, chaque token se voit attribuer un vecteur de nombres aléatoires — appelé **embedding**. Avec 768 dimensions typiques, c'est un point dans un espace à 768 dimensions. Ces chiffres ne veulent rien dire.

Puis l'entraînement commence. À chaque erreur de prédiction, tous les vecteurs impliqués sont légèrement ajustés. Après des milliards d'ajustements, quelque chose d'étrange se produit : les mots qui apparaissent dans les mêmes contextes finissent par se retrouver proches dans cet espace. "Chat" et "chien" se rapprochent. "Voiture" et "avion" se rapprochent. Personne n'a défini ces catégories — elles **émergent** du seul signal de prédiction.

C'est l'idée formulée par le linguiste John Firth en 1957 :

> *"You shall know a word by the company it keeps."*
> 

Un mot est défini par les mots qui l'entourent. L'entraînement encode cette co-occurrence dans la géométrie de l'espace vectoriel.

> 🖥️ **WIDGET INTERACTIF — Des tokens aux embeddings**
> 
> 
> *Présenter le widget en 4 étapes :*
> 
> - *Étape 1 : les index numériques — montrer l'absurdité de la distance arithmétique entre tokens*
> - *Étape 2 : vecteurs aléatoires — le nuage chaotique avant entraînement*
> - *Étape 3 : après entraînement — cliquer sur un mot pour voir ses 3 voisins les plus proches*
> - *Étape 4 : les analogies — roi − homme + femme ≈ reine*

### 4.3 La propriété qui surprend tout le monde : l'arithmétique vectorielle

En 2013, l'équipe Google travaillant sur Word2Vec découvre que les relations sémantiques sont encodées comme des **directions** dans l'espace vectoriel.

**roi − homme + femme ≈ reine**

Le vecteur qui va de "homme" à "femme" encode le genre. Le vecteur qui va de "homme" à "roi" encode la royauté. Ces deux directions sont indépendantes — et leur combinaison donne "reine". Personne ne l'a programmé. Ça a émergé de la seule contrainte : prédire correctement le mot suivant.

D'autres exemples du même type :

- Paris − France + Italie ≈ Rome *(relation capitale / pays)*
- nageur − nager + courir ≈ coureur *(relation agent / verbe)*

> 🖥️ **WIDGET INTERACTIF — Analogies vectorielles**
> 
> 
> *Présenter le widget d'analogies. Faire tester les 5 presets, puis laisser le public composer librement des combinaisons. L'effet "waouh" vient du fait que la croix "cible calculée" atterrit systématiquement près du bon mot.*
> 

### 4.4 Pourquoi autant de dimensions ?

Avec 2 dimensions, on peut séparer deux axes de variation. Avec 768, le modèle dispose de 768 axes indépendants : genre, nombre, registre, domaine, temporalité, causalité, sentiment…

L'analogie : si on veut décrire une personne avec un seul chiffre (sa taille), on perd presque tout. Avec dix chiffres (taille, âge, revenu, lieu, formation…), on distingue des profils très fins. Un LLM fait pareil avec les mots — mais avec des centaines de dimensions apprises automatiquement.

> 💡 **À l'oral** — *Insister sur le fait que ces dimensions ne sont pas définies à l'avance. Le modèle n'a pas de case "genre" ou "registre" préprogrammée. Il invente ses propres axes de variation, ceux qui minimisent le mieux ses erreurs de prédiction.*
> 

---

## 5. L'attention — regarder tout le contexte à la fois

### 5.1 Le problème de la mémoire courte

Le bigramme regardait 1 mot. Les N-grammes regardent N mots. Mais même avec N=10, le lien entre "Candide" et "il" six mots plus loin peut être manqué. Et surtout : tous les mots de la fenêtre ont le même poids — "dans" compte autant que "Candide" pour prédire le suivant, ce qui est absurde.

L'attention résout les deux problèmes en une fois : regarder **tout le contexte** précédent, avec des **poids variables** appris.

> 🖥️ **WIDGET INTERACTIF — Mécanisme d'attention**
> 
> 
> *Étape 1 : montrer la limite — cliquer "Montrer la limite" pour visualiser le lien impossible entre "Candide" et "il" dans une fenêtre bigramme.*
> 
> *Étape 2 : l'attention — cliquer sur le mot "il" et montrer que le poids d'attention vers "Candide" est très élevé. Le mécanisme a appris à faire ce lien. Faire tester d'autres tokens.*
> 

### 5.2 Q, K, V — les trois rôles de chaque token

Dans le mécanisme d'attention, chaque token joue simultanément trois rôles :

- **Q (Query / Requête)** — ce que je cherche
- **K (Key / Clé)** — ce que je suis
- **V (Value / Valeur)** — ce que je transmets

Pour calculer à quel point le token A doit "faire attention" au token B : on calcule le produit scalaire entre la requête de A et la clé de B. Ce score, normalisé sur tous les tokens du contexte, donne les poids d'attention. Ces poids pondèrent ensuite les valeurs pour produire une nouvelle représentation enrichie du token A.

> 💡 **Simplification pédagogique** — *À ce stade, l'essentiel est l'intuition : chaque token regarde tous les autres et décide à quel point chacun est pertinent pour lui. Les détails Q/K/V sont pour les curieux.*
> 

### 5.3 Multi-tête — plusieurs regards simultanés

Un seul mécanisme d'attention ne peut capturer qu'un type de relation à la fois. Les transformers en lancent plusieurs en parallèle — les **têtes d'attention**. Chacune se spécialise spontanément sur un aspect différent : relations syntaxiques, coréférences ("il" → "Candide"), relations sémantiques, positions relatives…

Les représentations produites par toutes les têtes sont ensuite concaténées et combinées. C'est ce qui donne au transformer sa richesse de compréhension.

> 💡 **Illustration suggérée** — *Un schéma montrant 4 têtes d'attention en parallèle, chacune labellisée avec son type de relation spécialisée, se recombinant en sortie.*
> 

---

## 6. L'entraînement — comment le modèle apprend

### 6.1 La forward pass

Le modèle reçoit une séquence de tokens et produit une distribution de probabilités sur l'ensemble du vocabulaire pour le token suivant. Ce calcul traverse toutes les couches du réseau de l'entrée vers la sortie — c'est la **forward pass**.

### 6.2 La perte — mesurer l'erreur

On compare la prédiction du modèle avec la réalité (le mot qui suit effectivement dans le corpus). L'écart entre les deux s'appelle la **perte** (loss). Si le modèle donnait 62 % de probabilité au bon mot, la perte est plus faible que s'il lui en donnait 5 %.

L'objectif de l'entraînement : minimiser la perte sur l'ensemble du corpus.

### 6.3 La rétropropagation — identifier les responsables

Comment corriger les poids qui ont contribué à l'erreur ? En remontant le réseau en sens inverse — la **backward pass** ou rétropropagation.

Chaque poids reçoit un **gradient** — un nombre qui dit : "tu as contribué à l'erreur dans cette direction, et de cette intensité." Les couches proches de la sortie reçoivent un signal fort ; les couches profondes reçoivent un signal atténué.

> 💡 **Analogie** — *Un chef cuisinier goûte le plat, constate qu'il est trop salé, et remonte mentalement la recette pour identifier où le sel a été ajouté en excès — et dans quelle proportion chaque étape est responsable.*
> 

### 6.4 La descente de gradient — faire le pas

La rétropropagation calcule la direction de la correction. La **descente de gradient** fait le pas : chaque poids est légèrement ajusté dans la direction qui réduit la perte.

L'analogie : on est dans le brouillard sur une montagne et on veut descendre dans la vallée. On ne voit pas la vallée — on sent seulement la pente sous les pieds. À chaque pas, on regarde dans quelle direction le sol descend le plus vite et on avance dans cette direction.

Le **taux d'apprentissage** contrôle la taille du pas. Trop grand : on saute par-dessus la vallée. Trop petit : on n'arrive jamais. C'est l'un des réglages les plus critiques de l'entraînement.

> 🖥️ **WIDGET INTERACTIF — Rétropropagation et descente de gradient**
> 
> 
> *Présenter le widget en 6 étapes. Insister sur :*
> 
> - *Étape 3 (l'erreur) : rendre concret le concept de "perte"*
> - *Étape 5 (ajustement) : montrer la courbe et la bille — c'est la descente de gradient*
> - *Étape 6 (après des milliards) : les ordres de grandeur de GPT-3 pour ancrer dans la réalité*

### 6.5 Ce qui émerge

À la fin de l'entraînement, personne n'a défini ce que signifient "chat", "roi" ou "justice". Ces représentations ont émergé de la seule contrainte répétée des milliards de fois :

> **Prédis correctement le mot suivant.**
> 

C'est la conclusion pédagogique centrale du chapitre : la richesse sémantique d'un LLM n'est pas programmée — elle est le sous-produit d'une tâche apparemment triviale, résolue à une échelle inhumaine.

---

## 7. Du pré-entraînement au modèle que vous utilisez

*(Section facultative — à présenter selon le temps disponible)*

Un modèle pré-entraîné est extraordinairement compétent pour compléter du texte. Mais si on lui pose une question comme un humain, il risque de continuer en mode corpus : "Quelle est la capitale de l'Allemagne ? Quelle est la capitale de l'Espagne ?…"

Pour produire un assistant utilisable, il faut deux étapes supplémentaires :

**SFT — Supervised Fine-Tuning** : on montre au modèle des milliers d'exemples de conversations humain/assistant de haute qualité. Il apprend le format : "quand on me pose une question, je réponds de façon utile et concise."

**RLHF — Reinforcement Learning from Human Feedback** : des annotateurs humains comparent et classent des réponses du modèle. On entraîne un modèle de récompense sur ces préférences, puis on l'utilise pour affiner le LLM. Le modèle apprend à maximiser la satisfaction humaine — pas seulement à prédire des mots.

> 💡 **Point clé** — *Le RLHF introduit quelque chose de nouveau : le signal d'apprentissage ne vient plus des données, mais des préférences humaines. C'est ce qui transforme un moteur de complétion en assistant.*
> 

> 💡 **Illustration suggérée** — *Un diagramme en pipeline montrant les trois phases : pré-entraînement (corpus brut) → SFT (conversations annotées) → RLHF (préférences humaines) → modèle aligné.*
> 

---

## 8. Ce qu'il faut retenir

| Concept | En une phrase |
| --- | --- |
| Token | Morceau de mot — l'unité de base que le modèle perçoit |
| Embedding | Vecteur de N nombres représentant un token dans un espace sémantique |
| Attention | Mécanisme qui pondère l'importance de chaque token du contexte |
| Forward pass | Le signal traverse le réseau de l'entrée vers la prédiction |
| Perte (loss) | Mesure de l'écart entre prédiction et réalité |
| Rétropropagation | Calcul de la contribution de chaque poids à l'erreur |
| Descente de gradient | Ajustement des poids pour réduire la perte |
| Pré-entraînement | Apprentissage sur corpus brut — prédire le mot suivant |
| Fine-tuning | Spécialisation sur des données ciblées (SFT, RLHF) |

---

> **Transition vers le chapitre suivant — Les hallucinations**
> 
> 
> Un LLM ne "sait" pas qu'il se trompe. Il produit toujours la suite de texte la plus probable étant donné son contexte — que cette suite soit vraie ou fausse. Comprendre ce mécanisme permet de comprendre précisément pourquoi et comment les hallucinations se produisent.
>
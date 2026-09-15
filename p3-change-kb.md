# Partie 3 — Change Enablement, Knowledge Management & Product and Service Lifecycle

---

## 1. RFC — Request For Change

### 1.1 Identification

| Champ | Valeur |
|---|---|
| Référence | RFC-2026-014 |
| Titre | Mise en place du modèle de qualification des tickets dans l'outil de ticketing (GLPI) |
| Amélioration d'origine | CSI-01 du CSI Register (Partie 1) |
| Demandeur | Technicien réseau / support, équipe helpdesk interne |
| Approbateur sollicité | Responsable IT, en formation CAB avec le représentant métier |
| Date de soumission | 08/09/2026 |
| Fenêtre de déploiement demandée | Mercredi 16/09/2026, 12h00 – 14h00 |
| Type de changement | **Normal** |

### 1.2 Description du changement

Le changement porte sur la configuration de l'outil de ticketing existant. Il comprend quatre éléments :

1. **Arbre de catégories normalisé** — remplacement de la liste actuelle par un arbre à deux niveaux (10 catégories parentes maximum, sous-catégories limitées au strict nécessaire). Les anciennes catégories sont désactivées, non supprimées, afin de conserver l'historique des tickets déjà rattachés.
2. **Gabarit de ticket** rendant obligatoires à la qualification : catégorie, impact, urgence, groupe assigné.
3. **Calcul automatique de la priorité** à partir de la matrice impact × urgence définie en Partie 2, en remplacement de la priorité déclarative.
4. **Règles métier d'attribution** dirigeant chaque ticket vers le groupe compétent selon sa catégorie, et **règle de repli** affectant au groupe Helpdesk-N1 tout ticket qu'aucune règle ne capte — de sorte qu'aucun ticket ne puisse rester sans propriétaire.

Ce changement ne modifie ni la version de l'outil, ni son infrastructure, ni les droits des utilisateurs.

### 1.3 Classement du type de changement et justification

**Type retenu : normal.**

- Ce n'est pas un **changement standard** : le modèle de qualification n'existe pas encore, aucune procédure pré-approuvée ne le couvre et le risque n'est pas évalué à partir d'occurrences passées. Il pourra le devenir plus tard — l'ajout ultérieur d'une catégorie dans un arbre déjà stabilisé est un bon candidat à la pré-approbation.
- Ce n'est pas un **changement urgent** : aucun service n'est indisponible. Les dysfonctionnements traités sont chroniques, installés depuis des mois. Invoquer l'urgence pour contourner l'évaluation serait ici un raccourci de confort, et ferait perdre la fenêtre de test qui constitue la principale protection contre la régression décrite ci-dessous.
- Il relève donc du **changement normal** : évaluation préalable, validation par le CAB, planification dans une fenêtre convenue, test en recette, communication aux utilisateurs.

### 1.4 Analyse d'impact

**Populations affectées**

| Population | Nature de l'impact | Intensité |
|---|---|---|
| Techniciens helpdesk (3) | Modification directe du poste de travail quotidien : nouveaux champs obligatoires, nouvelle logique d'attribution, perte des repères de l'ancien arbre de catégories | Forte |
| Ensemble des utilisateurs internes | Formulaire du portail modifié ; en régime dégradé, risque de ne plus parvenir à déposer une demande | Moyenne |
| Équipes N2 (système, réseau) | Réception de tickets par attribution automatique et non plus par sollicitation directe | Moyenne |
| Direction / reporting | Rupture de continuité statistique : les indicateurs par catégorie avant et après le changement ne sont pas comparables | Faible mais durable |

**Risques de régression identifiés**

| Risque | Conséquence concrète | Probabilité | Parade prévue |
|---|---|---|---|
| R1 — Règle d'attribution mal ordonnée | Un ticket réseau part vers le groupe bureautique ; le délai de première réponse est doublé du temps de ré-attribution | Moyenne | Jeu de 20 tickets de test rejouant les cas réels les plus fréquents, exécuté en recette avant mise en production |
| R2 — Champs obligatoires bloquant la soumission côté portail | L'utilisateur ne parvient pas à valider son formulaire, abandonne, et téléphone : le symptôme « tickets perdus » est **aggravé** par le correctif censé le traiter | Moyenne | Les champs obligatoires ne s'appliquent qu'au gabarit technicien, jamais au formulaire utilisateur. L'utilisateur saisit un objet et une description ; la qualification est un acte technicien |
| R3 — Aucune règle ne capte un ticket | Retour exact au problème d'origine : ticket sans propriétaire | Faible | Règle de repli en dernière position vers Helpdesk-N1, testée explicitement avec une catégorie inconnue |
| R4 — Recalcul de priorité appliqué aux tickets déjà ouverts | Requalification massive de l'encours en cours de traitement, techniciens désorientés | Faible | Règles limitées à l'événement « à la création » et « à la mise à jour manuelle », pas d'exécution rétroactive sur la base |
| R5 — Perte de l'historique rattaché aux anciennes catégories | Impossibilité d'exploiter les statistiques antérieures | Faible | Désactivation des anciennes catégories au lieu de leur suppression |

**Impact en cas d'échec total :** le service revient à son fonctionnement actuel, dégradé mais opérationnel. Aucune perte de données, aucune indisponibilité de l'outil. C'est cette réversibilité qui justifie de procéder directement en production après recette, plutôt que d'exiger un environnement de pré-production dédié.

### 1.5 Plan de test (recette, avant mise en production)

1. Cloner l'instance de ticketing sur l'environnement de recette à partir de la sauvegarde de la veille.
2. Appliquer l'intégralité de la configuration sur cette instance.
3. Rejouer les 20 tickets de test : 4 par groupe cible, plus 2 cas volontairement inclassables pour valider la règle de repli, plus 2 cas P1 pour vérifier le calcul de priorité.
4. Critère de passage : 20/20 tickets attribués au groupe attendu et priorité calculée conforme à la matrice. Tout écart est corrigé et la série entière est rejouée.

### 1.6 Plan de déploiement

| Horaire | Action | Responsable |
|---|---|---|
| J-3 | Communication aux utilisateurs (courriel + affichage) et briefing de 30 min des techniciens sur le nouvel arbre | Demandeur |
| J-1, 22h | Sauvegarde complète de la base et des fichiers de l'application, vérifiée par test de lecture du fichier produit | Équipe système |
| J, 11h45 | Export XML des règles métier en vigueur ; capture de l'arbre de catégories actuel | Demandeur |
| J, 12h00 | Création des catégories, du gabarit, puis des règles métier — dans cet ordre, les règles référençant les catégories | Demandeur |
| J, 13h00 | Tests de vérification en production : 5 tickets réels de contrôle, un par groupe cible | Demandeur + 1 technicien |
| J, 13h45 | Validation ou déclenchement du retour arrière | Approbateur |
| J+1 → J+5 | Surveillance renforcée : revue quotidienne des tickets mal routés | Équipe helpdesk |

### 1.7 Plan de retour arrière (rollback)

Le retour arrière est conçu en trois niveaux, du moins intrusif au plus lourd. On applique le premier niveau suffisant.

**Niveau 1 — Désactivation des règles (cible : moins de 5 minutes).**
Dans l'interface d'administration, passer les règles métier créées à `Actif = Non`, dans l'ordre inverse de leur création. L'attribution redevient manuelle, exactement comme avant le changement. Les catégories et le gabarit restent en place sans effet bloquant. Aucune intervention en base, aucune coupure de l'outil, aucun ticket perdu. C'est le niveau qui couvre R1, R3 et R4.

**Niveau 2 — Restauration de la configuration exportée (cible : 20 minutes).**
Si la désactivation ne suffit pas (règles corrompues, comportement incohérent) : supprimer les règles créées, puis réimporter le fichier XML exporté à 11h45 via la fonction d'import de règles. Remettre le gabarit précédent en gabarit par défaut. Réactiver les anciennes catégories, qui n'ont jamais été supprimées.

**Niveau 3 — Restauration de la sauvegarde complète (cible : 1 h 30, dernier recours).**
Uniquement en cas d'atteinte à l'intégrité des données. Mettre l'application en maintenance, restaurer le dump de la base de J-1 22h et l'arborescence des fichiers, redémarrer les services, vérifier par sondage que les derniers tickets d'avant l'arrêt sont présents. **Conséquence assumée :** les tickets créés entre 22h la veille et l'heure de restauration sont perdus ; ils doivent être re-saisis à partir de la boîte mail du helpdesk, qui conserve les demandes entrantes. C'est ce coût qui impose de ne recourir à ce niveau qu'en dernier ressort.

**Critères de déclenchement du retour arrière.** Le retour arrière est déclenché, sans nouvelle réunion, si l'un de ces constats est fait avant 13h45 :
- un utilisateur ne peut pas soumettre une demande depuis le portail ;
- plus de 2 des 5 tickets de contrôle sont attribués au mauvais groupe ;
- un ticket de contrôle se retrouve sans groupe assigné ;
- la durée d'indisponibilité cumulée de l'outil dépasse 15 minutes.

**Décision :** le demandeur constate et déclenche le niveau 1 de sa propre initiative ; le passage au niveau 2 ou 3 relève de l'approbateur. La fenêtre de 12h00 à 14h00 est dimensionnée pour que le retour arrière lui-même tienne dans la fenêtre.

---

## 2. Simulation de validation en CAB

**Séance :** CAB hebdomadaire du 10/09/2026. **Objet :** RFC-2026-014. **Présents :** responsable IT (président), représentant métier (services généraux), technicien demandeur, référent système.

### 2.1 Position du demandeur

« Je demande l'autorisation de déployer le modèle de qualification décrit dans la RFC-2026-014, issu du diagnostic mené sur le helpdesk.

Le problème que je traite part de trois symptômes remontés de façon récurrente par les utilisateurs, et d'un fonctionnement vérifiable dans l'outil : les tickets ne sont pas attribués nominativement à leur création, et la priorité de traitement est de fait l'ordre d'arrivée. Les mesures de confirmation définies dans le diagnostic serviront de point de référence avant et après le changement. Concrètement, une panne bloquant une équipe entière attend derrière une demande individuelle de confort. C'est ce que les utilisateurs appellent lenteur et tickets perdus.

Le changement porte uniquement sur de la configuration de l'outil déjà en place. Il n'y a ni achat, ni développement, ni changement de version, ni interruption de service prévue. Je le propose en changement normal et non standard, parce qu'il est inédit et que je ne dispose d'aucun historique de risque pour le pré-approuver.

Sur la réversibilité, qui est le point qui vous intéresse le plus : la désactivation des règles restaure le comportement actuel en moins de cinq minutes depuis l'interface, sans toucher à la base. Je n'ai besoin de la restauration de sauvegarde que dans un scénario d'atteinte aux données que rien dans ce changement ne rend plausible.

Enfin, c'est le premier des trois incréments inscrits au CSI Register. Les deux suivants — canal d'entrée unique et base de connaissance — en dépendent techniquement : on ne peut pas notifier un utilisateur du statut de son ticket si ce ticket n'a pas de propriétaire. Refuser ou repousser celui-ci bloque la trajectoire d'amélioration entière. »

### 2.2 Position de l'approbateur

« Trois objections avant de me prononcer.

**Première objection — le risque le plus coûteux n'est pas celui que vous mettez en avant.** Vous insistez sur le mauvais routage, qui est rattrapable par une ré-attribution manuelle. Ce qui m'inquiète est R2 : si un champ obligatoire empêche un utilisateur de déposer sa demande, il ne nous le signale pas, il téléphone ou il renonce. Nous aurions alors dégradé le service en croyant l'améliorer, et nous ne le verrions pas dans nos statistiques — nous verrions même une baisse du volume de tickets, que quelqu'un pourrait interpréter comme un succès. **Condition 1 : aucun champ obligatoire supplémentaire côté formulaire utilisateur. La qualification est un acte technicien, point.**

**Deuxième objection — la fenêtre.** Mercredi 12h00 est l'heure à laquelle les utilisateurs signalent ce qu'ils ont constaté le matin. Je ne veux pas d'un changement pendant que le flux entrant est actif. **Condition 2 : déploiement mercredi 16/09 entre 12h30 et 14h00, avec un technicien maintenu sur la ligne téléphonique et la consigne de créer manuellement tout ticket reçu pendant l'intervention.**

**Troisième objection — vous engagez toute l'équipe sur un changement d'habitude sans étape de vérification.** Vos vingt tickets de test en recette valident la mécanique, pas l'usage. **Condition 3 : période probatoire de 10 jours ouvrés avec revue quotidienne des tickets mal routés, et bilan chiffré présenté au CAB du 01/10. Deux indicateurs : part des tickets assignés sous 1 h, et nombre de ré-attributions manuelles. Si les ré-attributions dépassent 15 % du volume, on révise l'arbre de catégories, ce qui est une correction — on ne revient pas en arrière.**

Sur le fond, la démarche est saine : vous corrigez un défaut de processus par de la configuration, vous ne demandez pas de budget, et votre retour arrière tient en cinq minutes sans intervention en base. C'est exactement le profil de risque que ce comité peut accepter.

**Décision : RFC-2026-014 approuvée sous les trois conditions ci-dessus.** Le demandeur est autorisé à déclencher seul le niveau 1 du retour arrière ; il me contacte avant tout passage au niveau 2 ou 3. »

---

## 3. Article de base de connaissance

> Article KB-047 — Créé le 08/09/2026 — Applicable à partir du 16/09/2026 (mise en production de RFC-2026-014) — Public : techniciens helpdesk N1 et N2 — Prochaine revue : 16/03/2027

### Symptôme

Un ticket créé depuis le portail ou par courriel n'apparaît dans la file d'aucun groupe. Il est visible en recherche globale par un technicien qui connaît son numéro, mais aucun groupe ne le voit dans sa vue de travail, et aucun technicien ne lui est assigné. Le demandeur ne reçoit aucune réponse et finit par rappeler.

Signes complémentaires permettant de reconnaître ce cas :
- le champ « Groupe assigné » du ticket est vide ;
- le champ « Catégorie » est vide ou porte une catégorie désactivée ;
- l'historique du ticket ne montre aucune action d'attribution automatique.

### Cause

Depuis la mise en production du modèle de qualification (RFC-2026-014), l'attribution des tickets repose sur des règles métier qui s'appuient sur la **catégorie** du ticket. Trois situations produisent ce symptôme :

1. **Le ticket est arrivé sans catégorie.** Les tickets créés par courriel n'en portent pas à leur création ; ils dépendent donc de la règle de repli vers Helpdesk-N1. Si cette règle a été désactivée, déplacée de la dernière position, ou si une règle antérieure marquée « arrêter le traitement des règles » s'est déclenchée avant elle, aucune attribution n'a lieu.
2. **La catégorie du ticket n'est couverte par aucune règle.** Cas typique après l'ajout d'une nouvelle catégorie sans création de la règle correspondante.
3. **La règle existe mais son critère ne correspond pas.** Écart entre le libellé attendu et le libellé réel de la catégorie (renommage, sous-catégorie créée sous une autre parente).

Une cause distincte, à écarter en premier parce qu'elle ne relève pas de la même correction : le ticket a bien été attribué, puis un technicien s'est désassigné sans réassigner. Dans ce cas, l'historique du ticket porte la trace de l'attribution puis du retrait.

### Résolution

**A — Rétablir le service pour le ticket concerné (immédiat)**
1. Ouvrir le ticket, renseigner la catégorie adéquate, l'impact et l'urgence.
2. Assigner le ticket au groupe compétent ; à défaut de certitude, l'assigner à Helpdesk-N1.
3. Ajouter un suivi visible par le demandeur avec le délai de traitement attendu. Si le délai de première réponse est déjà dépassé, le mentionner explicitement plutôt que de le passer sous silence.
4. Vérifier qu'aucun autre ticket n'est dans le même état : filtrer la liste des tickets sur `Groupe assigné = (vide)` et `Statut = Nouveau`. Traiter le lot de la même façon.

**B — Corriger la cause (le jour même)**
5. Aller dans Administration > Règles > Règles métier pour les tickets.
6. Vérifier que la règle de repli `RM-99 — Repli Helpdesk-N1` est **active** et **en dernière position** de la liste. C'est l'erreur la plus fréquente : toute règle créée après elle passe devant dans l'ordre d'exécution et doit être remontée manuellement.
7. Contrôler qu'aucune règle placée avant elle et susceptible de capter le ticket ne porte l'action « arrêter le traitement des règles ».
8. Si la cause est une catégorie non couverte : créer la règle d'attribution correspondante, sur le modèle d'une règle existante, et la placer avant la règle de repli.
9. Tester avant de clore : créer un ticket de test portant la catégorie concernée, vérifier l'attribution obtenue, puis supprimer le ticket de test.

**C — Si le problème persiste après ces étapes**
10. Escalader au référent applicatif de l'outil de ticketing en joignant au ticket : le numéro du ticket en défaut, l'export XML des règles en vigueur et l'extrait d'historique du ticket. Ne pas modifier les règles à l'aveugle au-delà de l'étape 8 — une règle mal ordonnée affecte l'ensemble du flux entrant, pas seulement le ticket en cours.

### Mots-clés

`ticket non attribué` · `groupe assigné vide` · `ticket perdu` · `règle métier` · `règle de repli` · `RM-99` · `Helpdesk-N1` · `catégorie manquante` · `ticket créé par mail` · `ordre des règles` · `GLPI` · `RFC-2026-014` · `qualification`

---

## 4. Positionnement dans le Product and Service Lifecycle

Le modèle en huit activités du *ITIL Product and Service Lifecycle* — Discover, Design, Acquire, Build, Transition, Operate, Deliver, Support — remplace la Service Value Chain d'ITIL 4 et unifie la gestion des produits numériques et celle des services. Le positionnement ci-dessous porte sur RFC-2026-014.

### 4.1 Activités mobilisées

**Build — activité principale.** Le changement consiste à construire la configuration cible : arbre de catégories, gabarit de ticket, règles métier d'attribution, calcul automatique de la priorité. La recette décrite en 1.5, avec ses vingt tickets de test, appartient à cette activité : on vérifie que ce qui a été construit se comporte comme prévu, avant toute exposition aux utilisateurs.

**Transition — activité principale.** Tout ce qui fait passer la configuration validée vers l'environnement réel : la fenêtre du 16/09, la communication aux utilisateurs à J-3, le briefing des techniciens, les tickets de contrôle en production, le plan de retour arrière et la période probatoire de 10 jours imposée par le CAB. C'est l'activité dans laquelle se joue le risque effectif du changement.

**Design — activité partiellement mobilisée.** Le changement n'est pas un simple ajustement de paramètres : il redéfinit le flux de traitement d'un ticket. L'introduction d'un point de qualification obligatoire, le passage d'une priorité déclarée par l'utilisateur à une priorité calculée par matrice, et la distinction explicite entre incident et demande de service sont des décisions de conception du service, antérieures à toute écriture de règle. C'est également en Design qu'a été tranché le choix structurant relevé par le CAB : la qualification est un acte technicien, pas une charge déportée sur l'utilisateur.

**Acquire — non mobilisée.** Aucune acquisition : ni licence, ni matériel, ni prestation. L'outil et les compétences sont déjà en place. Cette activité est traversée sans produire d'action, ce qui est normal pour une correction de service existant.

**Discover — mobilisée en amont, hors périmètre de la RFC.** Le diagnostic des quatre dimensions et le CSI Register de la Partie 1 relèvent de cette activité : c'est là qu'a été identifié le besoin dont la RFC est la traduction.

**Operate, Deliver, Support — mobilisées en aval.** Une fois le changement passé, les techniciens exploitent le nouveau modèle au quotidien (Operate), le service rendu aux utilisateurs est mesuré par les SLA et SLO de la Partie 2 (Deliver), et la prise en charge des demandes s'appuie sur la base de connaissance dont l'article KB-047 ci-dessus est le premier élément (Support).

### 4.2 Pourquoi ce modèle n'est pas un enchaînement strictement linéaire

Trois éléments de ce cas le montrent concrètement.

**Les activités se chevauchent dans le temps.** L'article KB-047 est rédigé pendant la phase de construction, alors qu'il relève de Support et décrit un incident qui ne s'est pas encore produit. Attendre la fin de la Transition pour le rédiger reviendrait à laisser les premiers tickets en défaut être traités sans documentation, précisément pendant la période où ils sont les plus probables.

**Le retour d'information réinjecte en amont sans repasser par tout le cycle.** La période probatoire imposée par le CAB est explicitement conçue comme une boucle : si les ré-attributions manuelles dépassent 15 % du volume, l'arbre de catégories est révisé. On repart alors en Design et en Build sur un périmètre restreint, sans reprendre le cycle depuis Discover. C'est le principe directeur « Progresser de manière itérative avec du feedback » qui s'incarne dans la structure du lifecycle.

**Des activités sont traversées sans produire d'action.** Acquire est ici vide, et Discover a déjà eu lieu avant que la RFC n'existe. Traiter les huit activités comme des étapes obligatoires à franchir dans l'ordre produirait de la documentation sans contenu et ralentirait un changement dont la réversibilité est de cinq minutes — ce qui contredirait le principe « Garder les choses simples et pratiques ».

**Précision de vocabulaire.** Le Product and Service Lifecycle n'est pas un renommage de la Service Value Chain d'ITIL 4. La chaîne de valeur décrivait des activités de transformation d'une demande en valeur, du seul point de vue du fournisseur de service. Le lifecycle décrit la vie d'un produit et du service qui l'entoure comme un système unifié, de la découverte du besoin jusqu'au support en exploitation, et se lit aussi bien du point de vue du fournisseur de produit que de celui du fournisseur de service. La chaîne de valeur reste par ailleurs un composant du *ITIL Value System*, nom que prend en ITIL Version 5 l'ancien *Service Value System*.

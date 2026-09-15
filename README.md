# TP ITIL 5 — Amélioration du service helpdesk interne

Analyse et amélioration d'un service de support interne présentant trois symptômes chroniques : lenteur de traitement, tickets perdus, rappels multiples pour un même problème. Le cas est traité de bout en bout, du diagnostic à la clôture opérationnelle, en appliquant le référentiel ITIL (Version 5).

## Contenu du dépôt

| Fichier | Contenu |
|---|---|
| `p1-csi-register.md` | Diagnostic sous les quatre dimensions, CSI Register priorisé, principe directeur mobilisé |
| `p2-slm-events.md` | Deux SLA et leurs SLO, classification des cinq événements et actions associées |
| `p3-change-kb.md` | RFC-2026-014, simulation CAB, article KB-047, positionnement dans le Product and Service Lifecycle |
| `logs.txt` | Jeu d'événements fourni, classé en Partie 2 |
| `README.md` | Demande de service traitée et synthèse du cas |

**Fil conducteur.** Le diagnostic (P1) produit trois améliorations priorisées ; la première (CSI-01) devient la RFC de la Partie 3 ; les engagements définis en Partie 2 servent à mesurer son effet ; la Partie 4 traite la demande de service qui accompagne sa mise en production.

---

## 1. Demande de service traitée (Service Request Management)

> ⚠️ **À compléter depuis l'instance GLPI.** Les champs ci-dessous décrivent la demande telle qu'elle doit être créée et traitée ; le numéro de ticket, les horodatages exacts et les noms d'utilisateurs sont à remplacer par les valeurs réelles relevées dans GLPI après traitement.

| Champ | Valeur |
|---|---|
| **Numéro** | #2026-0912 |
| **Titre** | Création du groupe Helpdesk-N1 et rattachement des techniciens support |
| **Type** | Demande *(et non Incident)* |
| **Catégorie** | Outillage interne > Ticketing |
| **Demandeur** | Technicien support, équipe helpdesk |
| **Attribué à** | Groupe Administration GLPI |
| **Priorité** | Moyenne (impact limité × urgence moyenne) |
| **Statut** | Clos |
| **Date d'ouverture** | 18/09/2026 09:12 |
| **Date de prise en charge** | 18/09/2026 10:40 |
| **Date de résolution** | 18/09/2026 15:05 |
| **Date de clôture** | 19/09/2026 08:30 (après confirmation du demandeur) |
| **Changement associé** | RFC-2026-014 |

**Description (champ Description du ticket).**
La RFC-2026-014, approuvée en CAB le 17/09/2026, prévoit une règle de repli affectant à Helpdesk-N1 tout ticket qu'aucune règle d'attribution ne capte. Ce groupe n'existe pas encore dans l'outil. Demande de création du groupe `Helpdesk-N1`, de rattachement des trois techniciens support, et d'attribution du droit de visualisation et de prise en charge des tickets du groupe. Échéance impérative : avant le 23/09/2026 12h00, fenêtre de déploiement de la RFC.

**Suivis (extraits).**
- 18/09 10:40 — Prise en charge. Vérification de l'absence d'un groupe équivalent sous un autre libellé.
- 18/09 14:20 — Groupe créé, trois techniciens rattachés, profil « Technicien » appliqué.
- 18/09 15:05 — Test de vérification : création d'un ticket assigné manuellement au groupe, visible dans la vue des trois comptes. Demande de confirmation adressée au demandeur.
- 19/09 08:30 — Confirmation reçue du demandeur, clôture.

**Pourquoi il s'agit d'une demande de service et non d'un incident.**
Rien n'est en panne. Aucun service ne fonctionne en mode dégradé, aucun utilisateur n'est empêché de travailler. La demande porte sur la mise à disposition d'un élément prévu, planifié et attendu — la création d'un groupe dans l'outil — dans le cadre d'un changement approuvé. Elle relève donc de **Service Request Management** : traitement selon un flux standardisé et prévisible, avec un délai négocié à l'avance, et non selon la logique de rétablissement au plus vite propre à **Incident Management**.

Cette distinction a une conséquence de pilotage directe, et c'est la raison pour laquelle elle est explicitée ici : agréger demandes et incidents dans les mêmes statistiques rend le délai moyen de traitement ininterprétable. Les demandes de service, plus nombreuses et souvent plus rapides, masquent les incidents longs. C'est l'une des causes du symptôme « lenteur » relevé en Partie 1 — non parce que le service est lent partout, mais parce que personne ne savait où il l'était.

---

## 2. Synthèse — pratiques ITIL 5 mobilisées

| Partie | Objet | Pratiques et modèles ITIL (Version 5) mobilisés |
|---|---|---|
| **P1** | Diagnostic | ITIL Four Dimensions of Product and Service Management ; **Continual Improvement** (CSI Register, modèle d'amélioration continue) ; principes directeurs |
| **P2** | Pilotage | **Service Level Management** (SLA, SLO, matrice de priorité) ; **Monitoring and Event Management** (classification Informational / Warning / Exception) ; en aval, **Incident Management** déclenché sur les événements de type Exception |
| **P3** | Changement | **Change Enablement** (RFC, classement du changement, CAB, plan de retour arrière) ; **Knowledge Management** (article KB-047) ; positionnement dans le **ITIL Product and Service Lifecycle** |
| **P4** | Clôture | **Service Request Management** ; retour au **CSI Register** ; extension **ITIL AI Governance** et **ITIL AI Capability Model (6C)** |

**Cadre général.** L'ensemble s'inscrit dans le **ITIL Value System** — nom que prend en Version 5 l'ancien *Service Value System* d'ITIL 4 — dont les composants sont les principes directeurs, la gouvernance, la chaîne de valeur, les pratiques de gestion et l'amélioration continue.

---

## 3. Principe directeur le plus structurant sur l'ensemble du cas

**« Se concentrer sur la valeur ».**

Ce principe a produit une décision identifiable à chaque partie, et il a systématiquement écarté une solution plus évidente mais moins utile.

**L'exemple le plus net se situe en Partie 3, lors du CAB.** La discussion aurait pu se limiter au risque le plus visible — les règles d'attribution mal ordonnées, qui envoient un ticket au mauvais groupe. L'approbateur a écarté ce risque comme secondaire, parce qu'il est rattrapable et visible, pour concentrer la condition d'approbation sur un risque bien moins spectaculaire : un champ obligatoire supplémentaire sur le formulaire utilisateur. Techniquement, ce n'est rien. Du point de vue de la valeur, c'est le seul risque qui aurait dégradé le service **sans que le service puisse le voir** — l'utilisateur bloqué ne dépose pas de ticket pour signaler qu'il n'arrive pas à déposer de ticket. Le volume entrant aurait baissé, et un tableau de bord lu sans précaution l'aurait interprété comme une amélioration. D'où la condition retenue : la qualification est un acte technicien, jamais une charge déportée sur l'utilisateur.

Le même arbitrage revient ailleurs :
- en **Partie 1**, le diagnostic part des trois symptômes perçus par les utilisateurs et remonte vers les causes, au lieu de partir d'un audit technique de l'outil ;
- en **Partie 2**, les indicateurs de respect des délais sont volontairement contrebalancés par le taux de réouverture (SLO 2.3), parce que des délais tenus sur des tickets clos trop tôt produisent d'excellents chiffres et aucune valeur ;
- en **Partie 4**, la distinction entre demande et incident n'est pas maintenue par purisme de vocabulaire, mais parce que les confondre rend la mesure du service inexploitable.

*« Progresser de manière itérative avec du feedback »*, retenu en Partie 1, reste le principe qui a fixé l'ordre des travaux. Mais il organise l'exécution ; c'est « Se concentrer sur la valeur » qui a tranché le contenu de chaque décision.

---

## 4. Apport du module AI Governance et du modèle 6C sur ce cas

Le module **ITIL AI Governance** est le module d'extension de la Version 5. Il structure la gouvernance de l'IA selon quatre perspectives — autorité de décision et gestion du risque, principes éthiques et IA responsable, gouvernance des données et gestion de la performance, conformité réglementaire et standards opérationnels — et fournit le **ITIL AI Capability Model**, dit modèle **6C**, qui classe les capacités d'une solution d'IA en six fonctions : *Creation, Curation, Clarification, Cognition, Communication, Coordination*. Ce modèle sert à cadrer les contrôles et le profil de risque en fonction de ce que l'IA fait réellement, et non de l'étiquette commerciale du produit.

### 4.1 La conclusion honnête : non pertinent à ce stade du cas

Aucune des améliorations retenues ne mobilise d'IA, et c'est un choix argumenté, pas un oubli.

La seule application d'IA plausible sur ce helpdesk serait un classificateur automatique des tickets entrants — il attribuerait catégorie et priorité à la place du technicien. Au regard du modèle 6C, il relèverait de **Cognition** (interprétation du texte libre d'un utilisateur pour en déduire une catégorie) et de **Curation** (organisation et routage du flux entrant), avec une extension possible vers **Clarification** s'il suggérait au technicien l'article de base de connaissance correspondant.

Or ce classificateur est exclu ici pour une raison structurelle : **il s'entraînerait sur les données produites par le problème qu'on cherche à corriger.** Le diagnostic de la Partie 1 établit que l'historique des tickets est justement non fiable — catégories absentes ou mal renseignées, priorités déclaratives et non calculées, sollicitations jamais enregistrées, tickets clos sans confirmation. Un modèle appris sur cet historique reproduirait fidèlement la désorganisation existante, avec en plus l'autorité apparente d'une décision automatique. Le contrôle le plus élémentaire de la perspective « gouvernance des données » du module — qualité et représentativité des données d'apprentissage — n'est pas satisfait.

**La position retenue est donc : pas d'IA sur ce cas maintenant, réévaluation après six mois d'exploitation du nouveau modèle de qualification**, quand l'historique des tickets sera catégorisé de façon fiable. Cette décision est elle-même une décision de gouvernance : elle documente une abstention motivée, avec un critère de réexamen daté.

### 4.2 Le point où le module s'applique dès aujourd'hui : le Shadow AI

Le module est en revanche immédiatement pertinent sur un usage déjà présent et non gouverné. Un technicien qui colle un extrait de log, un message d'erreur ou le contenu d'un ticket dans un assistant conversationnel public fait sortir de l'entreprise des noms d'utilisateurs, des noms d'hôtes, des adresses IP internes et des éléments de topologie — l'événement `NET link=switch-3F-port12` de la Partie 2 en est un exemple direct. Au regard du 6C, cet usage relève de **Clarification** (aide au diagnostic) et de **Creation** (rédaction de la solution ou de l'article KB). Le risque n'est pas la qualité de la réponse obtenue ; c'est la fuite de données et la clôture d'un ticket sur une résolution non vérifiée.

Trois contrôles applicables immédiatement, sans outillage supplémentaire :
1. **Préventif** — règle écrite : aucun identifiant, nom d'hôte, adresse IP interne ni nom d'utilisateur dans une requête vers un assistant externe ; anonymisation préalable obligatoire.
2. **Détectif** — lors de la revue des articles KB, tout article rédigé avec assistance est signalé comme tel et validé par un second technicien avant publication.
3. **Frontière de décision** — l'IA propose, le technicien décide et engage sa responsabilité. Aucune clôture de ticket ni aucune publication d'article ne repose sur une sortie de modèle non relue.

### 4.3 Ce que le module n'apporte pas ici

Sur les trois quarts du cas — les quatre dimensions, les SLA et SLO, la RFC, le CAB, le retour arrière — le module d'extension n'apporte rien, et il serait malhonnête de le convoquer. Ces parties reposent sur des pratiques stables du référentiel. L'apport réel d'AI Governance sur ce TP se réduit à deux points précis : un **critère d'exclusion argumenté** pour une automatisation qui semblait séduisante, et un **cadre pour un usage de l'IA déjà en place mais jamais formalisé**. C'est peu, mais c'est exactement ce que le module est censé produire : une décision documentée sur un périmètre délimité, plutôt qu'une couche d'IA ajoutée parce qu'elle est disponible.

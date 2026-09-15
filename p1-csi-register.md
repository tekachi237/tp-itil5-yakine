# Partie 1 — Diagnostic du service helpdesk interne

## 0. Périmètre et méthode

**Service analysé :** helpdesk interne (support des utilisateurs de l'entreprise sur leur poste de travail et les applications métier).

**Symptômes rapportés par les utilisateurs :**
- S1 — lenteur de traitement des demandes ;
- S2 — tickets perdus ;
- S3 — utilisateurs qui rappellent plusieurs fois pour le même problème.

**Méthode :** analyse préalable sous les quatre dimensions de la gestion des produits et services (*ITIL Four Dimensions of Product and Service Management*), avant toute proposition d'outil ou de correctif technique. L'objectif de cette étape est de remonter des symptômes vers leurs causes, et non de traiter les symptômes.

**Convention de lecture :** chaque constat est formulé à partir d'un symptôme observable, puis assorti d'une **mesure de confirmation** — la donnée à extraire de l'outil de ticketing pour valider ou invalider le constat. Un constat non encore mesuré est identifié comme tel ; il ne doit pas être présenté comme un fait établi devant le commanditaire.

---

## 1. Constats par dimension

### 1.1 Organisations & personnes

**Constat.** Aucun rôle de qualification en entrée de file n'est identifié : les tickets arrivent dans une file commune où chaque technicien se sert, sans attribution nominative systématique. Il en résulte deux effets opposés qui se cumulent :
- des tickets que personne ne prend en charge parce qu'ils n'appartiennent à personne — c'est la lecture la plus probable du symptôme « tickets perdus » (S2), qui ne signifie pas nécessairement une perte technique de la donnée ;
- des tickets traités en double lorsque deux techniciens s'emparent du même sujet.

Aucun découpage de niveaux (N1 qualification / N2 expertise) n'est formalisé, ce qui conduit les profils les plus expérimentés à absorber indifféremment les demandes triviales et les incidents complexes, et allonge mécaniquement le délai de traitement des seconds (S1).

*Mesure de confirmation :* part des tickets restés sans technicien assigné plus de 4 h ouvrées sur les 3 derniers mois ; répartition du volume traité par technicien.

### 1.2 Information & technologie

**Constat.** Le service est joignable par plusieurs canaux (téléphone, courriel, sollicitation directe dans les couloirs) dont tous ne déclenchent pas la création d'un ticket. Une sollicitation orale traitée « à la volée » et jamais enregistrée est, du point de vue de l'utilisateur, un ticket perdu (S2) : il a signalé son problème, il n'en retrouve aucune trace.

Deuxième constat sur cette dimension : l'utilisateur n'a aucun retour automatique sur l'état d'avancement de son ticket. Le seul moyen dont il dispose pour savoir où en est sa demande est de rappeler le helpdesk — ce qui explique directement S3 (rappels multiples), sans qu'il y ait nécessairement un défaut de traitement derrière. Ces rappels consomment du temps technicien et dégradent à leur tour le délai de traitement (S1) : la boucle est auto-entretenue.

Troisième constat : il n'existe pas de base de connaissance exploitable. Un incident déjà résolu est ré-investigué de zéro à chaque occurrence, par un technicien qui n'a pas forcément traité la première.

*Mesure de confirmation :* nombre de tickets créés par canal sur un mois, comparé au nombre d'appels reçus sur le standard ; nombre de relances utilisateur enregistrées en suivi de ticket.

### 1.3 Partenaires & fournisseurs

**Constat.** Une partie des incidents ne peut pas être résolue par le helpdesk seul : éditeur de l'application métier, opérateur télécom, prestataire de maintenance du parc d'impression. Les engagements de délai de ces tiers ne sont ni formalisés, ni alignés sur les engagements pris par le helpdesk vis-à-vis des utilisateurs, et l'avancement côté fournisseur n'est pas remonté dans l'outil de ticketing.

Conséquence : un ticket en attente d'un tiers reste ouvert sans mise à jour visible. Pour l'utilisateur, il est indistinguable d'un ticket abandonné (S2), et le temps d'attente externe se confond avec le temps de traitement interne dans les statistiques du service (S1) — ce qui empêche de savoir ce qui relève réellement de la performance du helpdesk.

*Mesure de confirmation :* nombre de tickets ayant passé plus de 5 jours ouvrés en attente d'un tiers ; existence et contenu des engagements contractuels de délai des trois principaux fournisseurs.

### 1.4 Value Streams & processus

**Constat.** Le flux de traitement d'un ticket ne comporte pas d'étape de qualification à l'entrée. Concrètement, il manque quatre points de contrôle :
1. **Catégorisation** — sans catégorie fiable, aucun regroupement possible et aucune détection des problèmes récurrents ;
2. **Priorisation** — en l'absence de matrice impact × urgence, l'ordre de traitement est de fait un premier arrivé / premier servi, dans lequel un incident bloquant pour un service entier attend derrière une demande de confort (S1) ;
3. **Distinction incident / demande de service** — une panne et une demande planifiée suivent le même circuit, avec les mêmes attentes de délai, alors que leurs enjeux diffèrent ;
4. **Critère de clôture** — aucune confirmation utilisateur n'est exigée avant fermeture. Un ticket clos trop tôt revient sous forme de nouvel appel sur le même sujet (S3), et repart à zéro dans la file.

*Mesure de confirmation :* taux de réouverture ou de re-création de tickets sur un même couple utilisateur / objet sous 7 jours ; part des tickets sans catégorie ou en catégorie « Autre ».

---

## 2. CSI Register (Continual Service Improvement Register)

Registre des améliorations identifiées à l'issue du diagnostic. Statut initial : `Identifiée` pour les trois entrées.

| ID | Amélioration | Dimension(s) visée(s) | Symptômes traités | Effort | Impact | Priorité |
|----|--------------|----------------------|-------------------|--------|--------|----------|
| CSI-01 | Mettre en place un modèle de qualification à l'entrée : catégories normalisées, matrice impact × urgence, attribution nominative obligatoire, distinction incident / demande de service | Value Streams & processus ; Organisations & personnes | S1, S2 | Faible | Fort | **1** |
| CSI-02 | Instaurer un canal d'entrée unique : tout contact, y compris téléphonique et oral, donne lieu à la création d'un ticket par le technicien qui le reçoit ; portail et boîte mail connectés à l'outil | Information & technologie ; Value Streams & processus | S2 | Moyen | Fort | **2** |
| CSI-03 | Ouvrir une base de connaissance alimentée à la clôture des tickets récurrents, et activer les notifications automatiques de changement de statut vers l'utilisateur | Information & technologie | S1, S3 | Moyen | Moyen | **3** |

**Détail des cotations.**

- *CSI-01 — effort faible :* l'action porte sur la configuration de l'outil existant (arbre de catégories, gabarit de ticket, règles d'attribution) et sur une consigne d'équipe. Ni achat, ni développement, ni interruption de service. *Impact fort :* corrige la cause du traitement non priorisé et supprime les tickets sans propriétaire, c'est-à-dire l'essentiel de ce que les utilisateurs appellent « tickets perdus ».
- *CSI-02 — effort moyen :* la difficulté n'est pas technique mais comportementale (faire enregistrer les sollicitations orales), elle demande un accompagnement de l'équipe et un relais du management sur plusieurs semaines. *Impact fort :* sans elle, une partie du flux reste invisible et toute mesure du service est faussée.
- *CSI-03 — effort moyen :* la rédaction des articles est un coût récurrent qui pèse sur des techniciens déjà en charge, et le paramétrage des notifications suppose des statuts fiables — donc CSI-01 réalisée. *Impact moyen :* réduit les rappels et le temps de re-diagnostic, mais ne traite aucune cause racine à elle seule.

**Justification de la priorisation.**

L'ordre retenu n'est pas l'ordre d'impact décroissant seul ; il combine trois critères.

1. **Rapport impact / effort.** CSI-01 est la seule des trois à cumuler effort faible et impact fort : elle produit un effet mesurable en quelques jours, sur un service qui a besoin d'un résultat visible rapidement pour regagner la confiance de ses utilisateurs.
2. **Dépendances techniques.** CSI-03 ne peut pas fonctionner avant CSI-01 : notifier automatiquement un changement de statut n'a de sens que si les statuts sont tenus à jour et si le ticket a un propriétaire identifié. La placer en tête produirait des notifications vides de sens et aggraverait la perception du service.
3. **Capacité d'absorption de l'équipe.** CSI-02 exige un changement d'habitude de toute l'équipe, sur un service déjà sous tension. La mener en premier reviendrait à demander un effort de conduite du changement à des techniciens qui subissent encore la désorganisation de la file. En la plaçant après CSI-01, l'équipe travaille déjà dans un flux ordonné.

**Suite immédiate :** CSI-01 fait l'objet de la RFC rédigée en Partie 3.

---

## 3. Principe directeur mobilisé

**Principe retenu : « Progresser de manière itérative avec du feedback ».**

**Justification.** Le diagnostic identifie quatre dimensions en défaut simultanément. La réaction naturelle serait un plan de refonte globale du helpdesk traitant tout à la fois : nouveaux rôles, nouvel outillage, nouveaux engagements, base de connaissance. C'est exactement ce que ce principe invite à ne pas faire.

La priorisation ci-dessus découpe donc l'amélioration en trois incréments livrables séparément, chacun porteur de valeur par lui-même, et ordonnés de façon à ce que le premier produise la donnée qui pilotera les suivants : une fois CSI-01 en place, les catégories et les priorités deviennent fiables, et le service dispose enfin de statistiques exploitables pour arbitrer entre CSI-02 et CSI-03 — voire pour faire émerger une quatrième amélioration non identifiée aujourd'hui. Le feedback ne se limite pas au retour des utilisateurs : il inclut la mesure produite par l'incrément précédent.

**Principes écartés, et pourquoi.**
- *« Se concentrer sur la valeur »* a guidé la phase de constat (le diagnostic part des trois symptômes perçus par les utilisateurs, pas d'un audit technique de l'outil), mais il ne suffit pas à départager CSI-01, CSI-02 et CSI-03 : les trois créent de la valeur pour l'utilisateur.
- *« Commencer là où vous êtes »* est présent en arrière-plan — aucune des trois améliorations ne suppose de changer d'outil de ticketing — mais il décrit une contrainte respectée, pas le critère qui a fixé l'ordre.

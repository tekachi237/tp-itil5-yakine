# Partie 2 — Pilotage du service : Service Level Management & Event Management

## 1. Préalables communs aux deux SLA

Ces éléments conditionnent la mesure ; sans eux, un engagement de délai n'est ni opposable ni vérifiable.

**Heures de service :** lundi au vendredi, 8h00–18h00, hors jours fériés. Les compteurs de délai ne courent que pendant ces plages. Une demande déposée un vendredi à 17h30 avec un engagement de 4 h ouvrées est due le lundi à 11h30.

**Matrice de priorité (impact × urgence) :** la priorité n'est pas déclarée par l'utilisateur, elle est calculée à la qualification.

| | Urgence haute | Urgence moyenne | Urgence basse |
|---|---|---|---|
| **Impact étendu** (service ou site entier) | P1 | P1 | P2 |
| **Impact limité** (une équipe) | P1 | P2 | P3 |
| **Impact individuel** (un utilisateur) | P2 | P3 | P4 |

**Point de départ du chronomètre :** horodatage de création du ticket dans l'outil, quel que soit le canal d'origine.

**Suspensions du compteur (pauses SLA), à tracer explicitement dans le ticket :**
- attente utilisateur (information ou disponibilité manquante côté demandeur) ;
- attente tiers (éditeur, opérateur, prestataire) — cette exclusion est celle qui rend l'engagement tenable, le helpdesk ne pouvant s'engager sur un délai qu'il ne maîtrise pas. Elle ne dispense pas de suivre ces tickets : ils font l'objet d'une revue hebdomadaire dédiée.

Dans GLPI, un seul statut « En attente » suspend le calcul des SLA. Le motif (utilisateur ou tiers) est donc systématiquement précisé dans un suivi au moment de la mise en attente, ce qui permet de distinguer les deux cas lors de la revue.

**Fréquence de revue :** rapport mensuel des SLO, revue trimestrielle des SLA avec le représentant métier.

---

## 2. SLA 1 — Délai de première réponse

**Engagement.** Toute demande enregistrée reçoit une prise en charge humaine — technicien nommément assigné et premier retour au demandeur — dans le délai correspondant à sa priorité. Un accusé de réception automatique ne constitue pas une première réponse.

| Priorité | Délai de première réponse |
|---|---|
| P1 | 1 h ouvrée |
| P2 | 4 h ouvrées |
| P3 | 1 jour ouvré |
| P4 | 3 jours ouvrés |

**SLO associés :**
- **SLO 1.1 —** 95 % des tickets P1 reçoivent une première réponse en moins d'1 h ouvrée, mesuré sur le mois calendaire.
- **SLO 1.2 —** 90 % des tickets P2 et P3 reçoivent une première réponse dans leur délai cible, mesuré sur le mois calendaire.
- **SLO 1.3 —** 100 % des tickets créés sont assignés à un technicien identifié dans l'heure ouvrée suivant leur création.

*Justification de SLO 1.3 :* c'est l'indicateur qui adresse directement le symptôme « tickets perdus » relevé en Partie 1. Il est fixé à 100 % parce qu'il ne mesure pas une performance de résolution mais l'application d'une règle de gestion : un ticket sans propriétaire est un défaut de processus, pas un aléa de charge.

---

## 3. SLA 2 — Délai de résolution

**Engagement.** Le service est rétabli, ou la demande satisfaite, dans le délai correspondant à la priorité. Une solution de contournement acceptée par l'utilisateur vaut résolution au sens de cet engagement, à condition qu'un ticket de suivi soit ouvert pour la correction définitive.

| Priorité | Délai de résolution |
|---|---|
| P1 | 4 h ouvrées |
| P2 | 1 jour ouvré |
| P3 | 3 jours ouvrés |
| P4 | 10 jours ouvrés |

**SLO associés :**
- **SLO 2.1 —** 90 % des tickets P1 sont résolus en moins de 4 h ouvrées, mesuré sur le mois calendaire.
- **SLO 2.2 —** 85 % de l'ensemble des tickets, toutes priorités confondues, sont résolus dans leur délai cible.
- **SLO 2.3 —** le taux de réouverture reste inférieur à 5 % des tickets clos sur le mois.

*Justification de SLO 2.3 :* sans lui, les deux premiers SLO sont manipulables — clôturer massivement des tickets non résolus améliorerait mécaniquement les taux de respect des délais tout en dégradant le service réellement rendu. Ce contre-indicateur mesure ce que les utilisateurs décrivent comme « rappeler plusieurs fois pour le même problème ».

**Seuils de non-conformité.** Deux mois consécutifs sous l'un des SLO déclenchent l'ouverture d'une entrée au CSI Register et une analyse de cause, et non une renégociation à la baisse de l'engagement.

---

## 4. Classification des événements (Event Management)

**Rappel de la règle de classification appliquée.** *Informational* : événement attendu, conservé pour la traçabilité et la corrélation, sans action. *Warning* : seuil d'alerte franchi, service encore nominal, action préventive planifiable. *Exception* : anomalie avérée avec impact effectif ou imminent sur le service, action immédiate et ouverture d'un incident.

| # | Événement | Classification | Justification |
|---|-----------|----------------|---------------|
| 1 | `AUTH user=jdupont action=login status=success host=WKS-042` | **Informational** | Authentification réussie sur un poste de travail : comportement nominal. Aucune action. L'événement garde sa valeur en corrélation (audit d'accès, reconstitution d'une chronologie lors d'une investigation) et ne doit pas pour autant être remonté en alerte. |
| 2 | `DISK host=SRV-FILE01 usage=82% threshold=80%` | **Warning** | Le seuil d'alerte défini à 80 % est franchi, mais le service de fichiers reste disponible et l'écriture fonctionne. L'anomalie est anticipée, pas avérée : la fenêtre d'action est ouverte. |
| 3 | `SVC name=helpdesk-portal status=unreachable duration=00:04:12` | **Exception** | Indisponibilité effective et déjà constatée pendant 4 min 12 s du portail helpdesk. Impact aggravant dans ce contexte : le portail est le canal par lequel les utilisateurs déposent leurs tickets — pendant l'indisponibilité, les demandes se reportent sur le téléphone et l'oral, c'est-à-dire précisément les canaux qui génèrent les tickets non enregistrés identifiés en Partie 1. |
| 4 | `BACKUP job=nightly-backup host=SRV-DB01 status=completed size=45GB` | **Informational** | Sauvegarde nocturne terminée avec succès, volume cohérent : résultat attendu. Aucune action. À noter pour la conception de la supervision : c'est l'**absence** de cet événement dans la fenêtre horaire prévue qui doit être traitée en Exception, ce qui suppose une surveillance de non-occurrence et non une simple lecture des messages reçus. |
| 5 | `NET link=switch-3F-port12 status=down flapping=true count=6/10min` | **Exception** | Le lien est descendu et l'indicateur `flapping=true` avec 6 transitions en 10 minutes caractérise une instabilité avérée, pas un incident ponctuel. Le port bat : les sessions des utilisateurs raccordés tombent et se rétablissent en boucle, et l'instabilité peut se propager au niveau 2 (recalculs spanning-tree, réapprentissage des tables MAC). |

### Actions à déclencher

**Événement 2 — DISK SRV-FILE01 à 82 % (Warning)**
1. Créer une demande de service en priorité P3 sur SRV-FILE01, assignée à l'équipe système.
2. Sous 24 h ouvrées : identifier la consommation par répertoire, purger les données éligibles (corbeilles, exports temporaires, anciens jeux de sauvegarde locaux) et documenter le volume regagné dans le ticket.
3. Extraire la tendance de remplissage sur 30 jours pour estimer la date d'atteinte des 90 %. Si cette date est à moins de 30 jours, la purge ne suffit pas : ouvrir une demande d'extension de volume, qui relève d'un changement et non d'une action d'exploitation.
4. Vérifier que deux seuils distincts sont bien configurés en supervision — 80 % en Warning, 90 % en Exception. Un seuil unique produit soit des alertes trop tardives, soit du bruit permanent.

**Événement 3 — Portail helpdesk injoignable (Exception)**
1. Ouvrir immédiatement un incident **P1** (impact étendu, urgence haute) sur le service `helpdesk-portal`.
2. Vérifier dans l'ordre : état du service applicatif et de son serveur web, accessibilité de la base de données associée, certificat et résolution DNS, puis chemin réseau. Consigner chaque vérification dans le ticket, y compris les points écartés.
3. Activer le mode dégradé documenté : basculer la communication vers l'adresse mail du helpdesk, et informer les utilisateurs de l'indisponibilité par un canal indépendant du portail.
4. Consigne d'équipe pendant toute la coupure : chaque sollicitation reçue par un autre canal donne lieu à une création manuelle de ticket dès le rétablissement — sans quoi l'incident produit directement des tickets perdus.
5. Après rétablissement : si l'événement s'est déjà produit, ne pas clore sur le seul redémarrage. Enregistrer l'occurrence et, à la deuxième occurrence sur 30 jours, ouvrir une entrée au CSI Register pour une analyse de cause.

**Événement 5 — Port 12 du switch 3F instable (Exception)**
1. Ouvrir un incident **P2** (impact limité au 3ᵉ étage, urgence haute) assigné à l'équipe réseau.
2. Identifier les équipements raccordés au port 12 via la table MAC du switch et l'inventaire, afin de mesurer l'impact réel avant toute intervention — un poste isolé et un point d'accès Wi-Fi desservant l'étage ne donnent pas la même priorité.
3. Arrêter d'abord l'instabilité, diagnostiquer ensuite : désactiver administrativement le port (`shutdown`) pour stopper le battement et protéger le reste du domaine de niveau 2. Cette action isole l'équipement raccordé et doit être annoncée à l'utilisateur concerné.
4. Traiter les causes physiques par ordre de probabilité : remplacement du cordon de brassage, essai sur un port libre, contrôle de l'optique ou du transceiver si le lien est en fibre, vérification de la négociation duplex/vitesse des deux côtés.
5. Réactiver le port après remplacement et observer 30 minutes sans transition avant clôture. Si le battement persiste après changement de port et de câble, escalader vers l'équipe réseau pour suspicion de défaillance matérielle sur le châssis.
6. Activer `errdisable` sur détection de flapping si la fonctionnalité est disponible, pour que la protection soit automatique à la prochaine occurrence.

### Remarque de conception

La classification ci-dessus n'est reproductible que si les seuils et les règles de corrélation sont définis **avant** l'arrivée des événements. Un même message peut basculer d'une catégorie à l'autre selon le contexte : le remplissage disque à 82 % est un *Warning* sur un serveur de fichiers dont la croissance est lente, il serait une *Exception* sur un serveur de base de données dont le journal grossit de plusieurs points par heure. La pratique Event Management porte sur la conception de ces règles, pas sur la lecture au cas par cas des messages reçus.

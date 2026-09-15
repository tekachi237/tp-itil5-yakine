# TP ITIL / GLPI - Une journee chez NORTEK SI

Tout a ete traite dans GLPI (instance locale, entite racine), dans l'ordre de la journee. Ce document explique mes choix phase par phase. Les numeros de tickets, problemes et changements sont ceux de GLPI.

---

## Phase 1 - Gestion des incidents

J'ai cree les 5 tickets et j'ai calcule la priorite avec la matrice impact / urgence de GLPI, au lieu de traiter dans l'ordre d'arrivee.

| Ticket | Titre | Type | Urgence | Impact | Priorite | Pourquoi |
|---|---|---|---|---|---|---|
| 5 | Serveur de messagerie injoignable | Incident | Tres haute | Tres haut | Tres haute | Tout le monde est bloque (200 salaries), aucun contournement, un service critique est a l'arret |
| 7 | Email suspect (phishing) | Incident de securite | Haute | Haut | Haute | Ce n'est pas une panne technique : tant qu'on ne sait pas qui a recu le mail et qui a clique, le risque peut toucher toute l'entreprise, et il grandit avec le temps |
| 9 | DAF (VIP) - tableau de bord financier | Incident | Haute | Bas | Haute (forcee) | La matrice donnait Moyenne car un seul utilisateur est touche, je l'ai montee a Haute a cause du statut VIP et de la periode de cloture comptable |
| 8 | Imprimante 2e etage HS | Incident | Moyenne | Moyen | Moyenne | Une quinzaine de personnes genees, mais elles peuvent imprimer au 1er etage en attendant |
| 6 | Souris HS comptabilite | Incident | Basse | Tres bas | Tres basse | Un seul utilisateur, il peut continuer au clavier et une souris de pret est dispo tout de suite |

Ordre de traitement : 5, puis 7, puis 9, puis 8, puis 6.

Le ticket n 3 de l'enonce (email suspect) est le piege : ce n'est ni une demande ni une panne. Je l'ai qualifie en incident de securite avec la categorie "Securite - Phishing", parce qu'il faut agir vite (bloquer l'expediteur, prevenir les utilisateurs, verifier si quelqu'un a saisi ses identifiants) meme si rien n'est casse techniquement.

Sur les SLA, j'ai utilise les delais suivants : Tres haute = prise en compte 15 min / resolution 4 h, Haute = 30 min / 8 h, Moyenne = 2 h / 2 jours ouvres, Basse et Tres basse = 4 h / 5 jours ouvres.

---

## Phase 2 - Gestion des problemes

J'ai cree le probleme n 2 "Coupures repetees du serveur de messagerie (3 incidents en 2 semaines)" et je lui ai lie le ticket 5. Les deux incidents d'avant (lenteur puis coupure) n'existent pas dans mon GLPI de test, donc je les ai juste mentionnes dans la description du probleme au lieu d'inventer des tickets.

Les 5 pourquoi (dans le suivi du probleme) :

1. Pourquoi la messagerie est tombee ? Le service mail s'est arrete tout seul.
2. Pourquoi s'est-il arrete ? La partition qui contient les bases et les logs etait pleine a 100 %.
3. Pourquoi etait-elle pleine ? Les logs de debug et les anciennes bases n'ont jamais ete purges depuis la migration.
4. Pourquoi n'ont-ils pas ete purges ? Le mode debug active pendant la migration est reste actif et il n'y a aucune rotation des logs.
5. Pourquoi personne ne l'a vu avant ? Il n'y a pas d'alerte de supervision sur l'espace disque de ce serveur.

Cause racine : mode debug laisse actif + pas de rotation des logs + pas d'alerte disque.

Workaround immediat : purge des logs et des anciennes bases, desactivation du mode debug, et verification manuelle de l'espace disque deux fois par jour avec redemarrage si besoin. Ca ne corrige pas la cause mais ca tient jusqu'au changement.

Entree KEDB : article cree dans la base de connaissances GLPI, avec symptomes / cause connue / contournement / solution definitive et des mots cles pour le retrouver.

---

## Phase 3 - Gestion des changements

J'ai cree le changement n 1 dans GLPI ("RFC - Mise a jour correctif serveur de messagerie + rotation des logs et alerte disque"). Il est lie au probleme n 2, au ticket 5, et j'ai ajoute SRV-MAIL01 dans ses elements.

RFC :
- Description : appliquer le correctif fournisseur sur le serveur mail, desactiver le mode debug, mettre en place la rotation automatique des logs et une alerte de supervision a 80 % de remplissage disque. Changement normal (pas urgent, le workaround tient) mais impact tres haut car la messagerie est coupee pendant l'operation.
- Risques : le correctif ne regle pas le probleme (on garde le workaround et on rappelle le fournisseur) ; la mise a jour plante et le service ne redemarre pas (on restaure la sauvegarde) ; la coupure dure plus longtemps que prevu (d'ou le soir, utilisateurs prevenus la veille).
- Impact : environ 45 min sans messagerie pour les 200 salaries, les mails entrants attendent en file et arrivent apres.
- Plan de rollback : snapshot de la VM + sauvegarde de la config a 19h00. Si le service ne redemarre pas ou si les tests d'envoi/reception ratent avant 19h45, on restaure le snapshot et on reste sur l'ancienne version avec le workaround.
- Fenetre de maintenance : mercredi 17/09/2026, 19h00 - 20h00.

CAB simule (16/09/2026) : avis favorable sous conditions. Le CAB accepte parce que la cause est connue (probleme + KEDB), que le correctif vient du fournisseur, que le retour arriere est simple et que ca se fait hors heures de bureau. Le representant metier voulait eviter le lundi et la fin de mois, d'ou le mercredi soir. Conditions : sauvegarde testee avant de commencer, mail d'info la veille, test envoi/reception valide avant de cloturer, et surveillance du disque pendant une semaine.

Tout ca est trace dans GLPI en 3 suivis du changement (risques/impact, fenetre + deploiement + rollback, avis CAB).

---

## Phase 4 - Gestion des configurations (CMDB)

J'ai cree 4 CI dans le parc GLPI (Ordinateurs) et je les ai relies dans l'onglet "Analyse d'impact" de SRV-MAIL01 :

| CI | Role | Relation avec le serveur mail |
|---|---|---|
| SRV-MAIL01 | Serveur de messagerie (CI principal) | - |
| SRV-AD01 | Annuaire / controleur de domaine | SRV-AD01 impacte SRV-MAIL01 (le mail a besoin de l'annuaire pour l'authentification) |
| SRV-BKP01 | Serveur de sauvegarde | SRV-MAIL01 impacte SRV-BKP01 (plus de sauvegarde des bases mail si le serveur est HS) |
| PC-COMPTA-01 | Poste client (exemple pour tous les postes) | SRV-MAIL01 impacte PC-COMPTA-01 (plus de mail sur le poste) |

Note d'impact si SRV-MAIL01 tombe :
- les 200 salaries n'ont plus de mail (ni envoi ni reception, ni webmail), Outlook affiche une erreur de connexion sur tous les postes ;
- la sauvegarde de la nuit des bases mail echoue, donc on perd un point de restauration ;
- l'annuaire SRV-AD01 n'est pas touche : c'est le serveur mail qui depend de lui, pas l'inverse ;
- services impactes : communication interne et externe, envoi des factures par la compta, alertes et notifications qui partent par mail (y compris celles de GLPI).

C'est ce qui justifie la fenetre du soir pour le changement.

---

## Phase 5 - Gestion des demandes de service

J'ai cree le ticket n 10 de type Demande (pas Incident), categorie "Onboarding IT", pour l'arrivee de Julie, chargee de communication, le lundi 21/09/2026.

C'est une demande et pas un incident car rien n'est en panne : c'est une arrivee prevue a l'avance, qui suit le catalogue de services. On ne retablit pas un service, on met a disposition quelque chose de nouveau. Du coup priorite Basse (urgence moyenne, impact bas), ca ne passe pas devant les incidents.

Catalogue deroule (une tache GLPI par element, les 4 sont cochees) :
- [x] creation compte utilisateur + boite mail
- [x] attribution poste + peripheriques
- [x] acces VPN
- [x] acces au dossier partage "Communication"

SLA demande de service : 5 jours ouvres maximum pour un onboarding, avec prise en charge sous 1 jour ouvre. C'est plus long que pour un incident (4 h de resolution en Tres haute) parce que c'est planifie et que le delai est connu a l'avance. Demande recue le 15/09, donc le SLA tomberait le 22/09, mais c'est apres l'arrivee de Julie : j'ai donc fixe une date cible plus tot. Dans GLPI : TTO (prise en charge) = 16/09/2026 12h00, TTR (date cible de mise a disposition) = vendredi 18/09/2026 18h00, pour avoir le temps de tout tester avant le lundi 21/09.

---

## Fil rouge

Dans GLPI : ticket 5 (incident mail) -> probleme 2 -> changement 1 -> CI SRV-MAIL01 et ses dependances. Le changement 1 est aussi lie directement au ticket 5, et l'article KEDB reprend la cause du probleme 2. La demande n 10 de Julie est a part, c'est normal, elle n'a rien a voir avec la panne.

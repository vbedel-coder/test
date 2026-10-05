# Fournil Vivant — contexte pour Claude Code

## Le projet
Appli de production pour Vivant, boulangerie artisanale bio au levain (Paris 11e). Un seul utilisateur : Vincent, le boulanger, qui travaille seul à la production et utilise l'appli sur iPhone et iPad (pas d'ordinateur). L'appli remplace un ancien Google Sheets (« 2026 Planning, recette VIVANT »).

Toute l'appli tient dans `index.html` (HTML + CSS + JavaScript, sans dépendance ni étape de build). Interface en français, pensée pour le téléphone : gros boutons, gros chiffres, utilisable les mains dans la farine.

## Ce que fait l'appli (3 onglets)
- **Boutique** : ventes prévues aux particuliers par jour (mardi → samedi). Valeurs par défaut = cadencier.
- **Pro** : clients professionnels (création, renommage, suppression). Commande type par jour de la semaine, modifiable par date. Pour chaque livraison : quantité en pains (pilote la production) et poids livré au 100 g près (sert uniquement à la facturation). Récap mensuel par client + export tableur (CSV).
- **Production** (mardi → vendredi, semaine datée) : pains à produire calculés depuis les ventes + commandes pro, levains à préparer (avec farine/eau), pesées par pétrissage, farines consommées (pâtes + levains, en % de sac de 25 kg).

## Règles métier (ne pas modifier sans accord de Vincent)
- **Production décalée** (tableau `P`, règles dans `prodDay`) :
  - `J0` produit le jour de la vente : Campagne, Campagne aux graines, Petit épeautre, Riz-sarrasin, Seigle, Méteil, Châtaigne, Pain de mie.
  - `J1` produit la veille : Khorasan, Blés anciens, FSS, Froment, Graine, Focaccia, Rugbrød.
  - `hebdo` : Brioche, toute la semaine produite le mardi.
  - Pas de production lundi ni samedi : les ventes du samedi sont produites le vendredi ; en J1 le mardi couvre aussi ses propres ventes.
- **Focaccia** : vendue au kg ; plaques à produire = kg ÷ 2,5.
- **Pétrissages communs** (`GROUPS`) : Froment + Campagne, Graine + Campagne aux graines, fusionnés en une seule fiche de pesée quand ils sont produits le même jour (aujourd'hui seulement le mardi). Vincent a validé ce fonctionnement.
- **Levains** (`LV`) : part de farine du levain mûr — froment 55 %, seigle 48 %, petit épeautre 52 %, riz 52 % (le reste en eau). Le chef (levain-mère) n'est pas encore pris en compte.
- **Poids de référence** (`W`) : kg par pain, utilisés pour estimer le poids livré aux pros.

## Modèle de données dans le code
- `R` : recettes. Chaque ingrédient `{n: nom, q: kg pour 1 unité, k: "farine"|"levain", f: référence de farine, l: type de levain}`. Les sous-préparations (bouillie, trempage, tangzhong, porridge) sont des groupes `{g: nom, items: [...]}`.
- Convention : identifiants en minuscules sans accent (`canele`), accents uniquement dans `name`.
- `SALES0` : ventes boutique par défaut (colonnes « Par. » du cadencier).
- `PANZA_TYPE` : commande type du client pro La Panza (colonnes « Pro » du cadencier, à confirmer par Vincent).

## Stockage
- `localStorage` sur l'appareil (préférences d'affichage + cache des données).
- Quand l'appli est ouverte comme artifact claude.ai, synchro en ligne via `window.claude.use("db")` : un document par utilisateur `data/users/<id>/state` contenant `{v, updatedAt, sales, clients, extras}`. Le plus récent (`updatedAt`) gagne.
- **Hors de claude.ai (ex. GitHub Pages), `window.claude` n'existe pas** : l'appli fonctionne mais n'enregistre que sur l'appareil. Pour une vraie version indépendante, il faut une base de données propre (ex. Supabase ou Firebase) avec connexion de l'utilisateur, en gardant le même format de données.
- Export « Sauvegarder toutes mes données » : JSON `{app, version, exportedAt, sales, clients, extras}`. Toute nouvelle version doit savoir importer ce fichier.

## Contrôle de non-régression
Avec les valeurs par défaut (semaine type, sans modification), la production doit rester identique au cadencier :
- Mardi : Campagne 10, Campagne aux graines 9, Petit épeautre 8, Riz-sarrasin 2, Blés anciens 16, FSS 16, Froment 10, Graine 8, Focaccia 3 plaques, Pain de mie 8, Brioche 14.
- Mercredi : Petit épeautre 6, Riz-sarrasin 2, Blés anciens 6, FSS 6, Froment 10, Graine 9, Focaccia 1,2, Pain de mie 6.
- Jeudi : Petit épeautre 4, Châtaigne 4, Blés anciens 6, FSS 6, Froment 12, Graine 8, Focaccia 1,2, Pain de mie 4.
- Vendredi : Petit épeautre 12, Riz-sarrasin 4, Seigle 4, Blés anciens 6, FSS 6, Froment 12, Graine 8, Focaccia 1,2, Pain de mie 8.
- Levains du mardi : froment 6 026 g (3 314 g farine / 2 712 g eau), seigle 4 717 g (2 264 g / 2 453 g).

## À faire / en attente de Vincent
- Canelé : recette proposée (pour 1 fournée : lait 1 kg, farine T80 300 g, sucre 360 g, œufs 200 g, beurre 65 g, rhum 50 g, vanille 3 g) — attendre le nombre de canelés par fournée et le délai de préparation avant de l'intégrer.
- Quantité de chef dans les levains.
- Prix au kg par client pro → montant HT dans le récap.
- Vraie commande type de La Panza ; statut du client Lotta.
- Recettes manquantes : Graines 50/50 du mardi, sablé, madeleine, crackers, panettone.

## Façon de travailler avec Vincent
Il n'est pas développeur : expliquer en français simple ce qui change et pourquoi, un changement à la fois, et toujours vérifier le contrôle de non-régression ci-dessus avant de proposer une modification.

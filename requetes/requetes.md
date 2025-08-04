## Requêtes du Projet

## 1) Calculer le nombre de retours clients sur la livraison
```sql
SELECT COUNT (DISTINCT cle_retour_client) AS nb_retours_livraison
FROM retour_client
WHERE libelle_categorie = "livraison";
```

## 2) Etablir la liste des notes des clients sur les réseaux sociaux sur les TV
```sql
SELECT r.cle_retour_client, r.note, r.libelle_source, p.titre_produit 
FROM retour_client r
JOIN produit p ON r.cle_produit = p.cle_produit
WHERE LOWER(r.libelle_source) = "réseaux sociaux" 
AND LOWER(p.titre_produit) = "tv";
```

Ici, LOWER sert à s'assurer qu'on prendra toutes les mentions à TV peu importe comment elles sont écrites (tv, Tv, tV, TV) en les ramenant toutes à "tv".

## 3) Calculer la note moyenne pour chaque catégorie de produit (Ordre décroissant)
```sql
SELECT ROUND(AVG(r.note),2) AS note_moyenne, p.typologie_produit
FROM retour_client r
JOIN produit p ON r.cle_produit = p.cle_produit
GROUP BY p.typologie_produit
ORDER BY note_moyenne DESC;
```

## 4) Déterminer les 5 magasins avec les meilleures notes moyennes
```sql
SELECT
  rmag.ref_magasin,
  rmag.libelle_de_commune,
  rmag.departement,
  ROUND(AVG(r.note),2) AS note_moyenne_magasin  
FROM retour_client r
JOIN ref_magasin rmag ON r.ref_magasin = rmag.ref_magasin
GROUP BY rmag.ref_magasin, rmag.libelle_de_commune, rmag.departement
ORDER BY note_moyenne_magasin DESC
LIMIT 5;
```

## 5) Déterminer les magasins qui ont plus de 12 feedbacks sur le drive 
```sql
SELECT
  rmag.ref_magasin,
  rmag.libelle_de_commune,
  rmag.departement,
  COUNT(r.note) AS nb_feedbacks  
FROM retour_client r
JOIN ref_magasin rmag ON r.ref_magasin = rmag.ref_magasin
WHERE LOWER(r.libelle_categorie) = "drive"
GROUP BY rmag.ref_magasin, rmag.libelle_de_commune, rmag.departement
HAVING nb_feedbacks > 12;
```

## 6) Classement des départements par note
```sql
SELECT
  rmag.departement,
  ROUND(AVG(r.note),2) AS note_moyenne_departement  
FROM retour_client r
JOIN ref_magasin rmag ON r.ref_magasin = rmag.ref_magasin
GROUP BY rmag.departement
ORDER BY note_moyenne_departement DESC;
```

## 7) Typologie de produits qui apporte le meilleur SAV
```sql
SELECT
  p.typologie_produit,
  ROUND(AVG(r.note),2) AS note_moyenne_sav  
FROM retour_client r
JOIN produit p ON r.cle_produit = p.cle_produit
WHERE LOWER(r.libelle_categorie) = "service après-vente"
GROUP BY p.typologie_produit
ORDER BY note_moyenne_sav DESC;
```

Si on ne veut que la catégorie qui a la meilleure notre, on rajoute la ligne "LIMIT 1", mais j'ai préféré un classement de l'ensemble.

## 8) Note moyenne sur l'ensemble des boissons
```sql
SELECT
    "Boissons" AS regroupement_produits,
    ROUND(AVG(r.note),2) AS note_moyenne_boissons  
FROM retour_client r
JOIN produit p ON r.cle_produit = p.cle_produit
WHERE LOWER(p.titre_produit) LIKE "%boisson%"
   OR LOWER(p.titre_produit) LIKE "%soda%"
   OR LOWER(p.titre_produit) LIKE "%saké%"
   OR LOWER(p.titre_produit) LIKE "%café%"
   OR LOWER(p.titre_produit) LIKE "%bière%";
```

On n'a pas de typologie_produit = "Boisson", mais uniquement "Alimentaire". On a aussi différents titre_produit correspondant à des boissons mais ne portant pas directement la mention "boisson"(bière, soda, café, café soluble, saké...). On force donc la création d'une colonne dans le cadre de notre requête, ici "regroupement_produit" à laquelle on attribue pour seule ligne "Boissons". 


## 9) Classement des jours de la semaine où l'expérience en magasin est la meilleure
```sql
SELECT
  strftime("%w", r.date_achat) AS jour_de_la_semaine,
  ROUND(AVG(r.note),2) AS note_moyenne_jour  
FROM retour_client r
WHERE LOWER(r.libelle_categorie) = "expérience en magasin"
GROUP BY jour_de_la_semaine
ORDER BY note_moyenne_jour DESC;
```
A savoir que dans notre table de résultats le jour 0 est le dimanche et le lundi est le 1.

## 10) Sur quel mois a-t-on le plus de retours sur le SAV ?
```sql
SELECT
  strftime("%m", r.date_achat) AS mois_retour,
  COUNT(DISTINCT r.cle_retour_client) AS nb_retours_sav  
FROM retour_client r
WHERE LOWER(r.libelle_categorie) = "service après-vente"
GROUP BY mois_retour
ORDER BY nb_retours_sav DESC;
```
Si on veut uniquement le mois avec le plus grand nombre de retour, rajouter une ligne "LIMIT 1;" en fin de requête. 

## 11) Déterminer le pourcentage de recommandations client
```sql
SELECT  ROUND(recommandations_positives*100/nb_recommandations_total, 2) AS taux_recommandation,
        nb_recommandations_total
FROM (
  SELECT
    SUM(CASE WHEN r.recommandation = 1 THEN 1 ELSE 0 END) AS recommandations_positives,
    SUM(CASE WHEN r.recommandation IN(0, 1) THEN 1 ELSE 0 END) nb_recommandations_total
     FROM retour_client r) AS sous_requete;  
```  

## 12) Quels magasins ont une note inférieure à la moyenne des magasins ?
```sql
SELECT
    r.ref_magasin,
    ROUND(AVG(r.note),2) AS note_moyenne_magasin,
    mg.moyenne_generale
FROM retour_client r
CROSS JOIN (SELECT ROUND(AVG(r.note),2) AS moyenne_generale
            FROM retour_client r
            WHERE r.note IS NOT NULL) AS mg
GROUP BY r.ref_magasin
ORDER BY note_moyenne_magasin DESC;
```

Le résultat nous affiche les notes moyennes de chaque magasin classées dans l'ordre décroissant dans une colonne "note_moyenne_magasin et la moyenne générale des magasins dans la colonne "moyenne_generale". On peut garder cela si on veut pouvoir avoir une vision globale de tous les magasins et voir aussi ceux qui ont une note moyenne supérieure à la moyenne générale.

Si par contre on préfère n'avoir que les magasins avec une note moyenne inférieure à la note moyenne générale dans notre vue, on rajoutera un HAVING :
```sql
SELECT
    r.ref_magasin,
    ROUND(AVG(r.note),2) AS note_moyenne_magasin,
    mg.moyenne_generale
FROM retour_client r
CROSS JOIN (SELECT ROUND(AVG(r.note),2) AS moyenne_generale
            FROM retour_client r
            WHERE r.note IS NOT NULL) AS mg
GROUP BY r.ref_magasin
HAVING note_moyenne_magasin < mg.moyenne_generale
ORDER BY note_moyenne_magasin DESC;
```

Le CROSS JOIN nous permet d'afficher la moyenne générale des magasins sur chaque ligne pour comparaison rapide.

## 13) Quelles typologies de produits ont amélioré leur note moyenne entre ler et le 2ème trimestre 2021 ?
```sql
SELECT 
    t1.typologie_produit AS typologie_produit,
    t1.note_moyenne_t1, 
    t2.note_moyenne_t2
FROM (
      SELECT p.typologie_produit, ROUND(AVG(r.note),2) AS note_moyenne_t1
      FROM retour_client r
      JOIN produit p on p.cle_produit = r.cle_produit
      WHERE r.date_achat BETWEEN "2021-01-01" AND "2021-03-31"
      GROUP BY p.typologie_produit
      ) AS t1
JOIN (
      SELECT p.typologie_produit, ROUND(AVG(r.note), 2) AS note_moyenne_t2
      FROM retour_client r
      JOIN produit p on p.cle_produit=r.cle_produit
      WHERE r.date_achat BETWEEN "2021-04-01" AND "2021-07-31" 
      GROUP BY p.typologie_produit)
      AS t2 
ON t1.typologie_produit = t2.typologie_produit
WHERE t1.note_moyenne_t1 < t2.note_moyenne_t2
ORDER BY t2.note_moyenne_t2 DESC;
```

## 14) Calcul du NPS global (Net Promoter Score) 

J'ai pu trouver 2 méthodes possibles pour ce calcul :

- **Méthode 1** : multiples sous requêtes imbriquées qui permettent de décomposer le calcul des composants de la formule du NPS et de structurer le raisonnement étape par étape; 
```sql
SELECT taux_p.taux_promoteurs - taux_d.taux_detracteurs AS NPS
FROM (SELECT promoteurs.nb_promoteurs*100/total.total_notes AS taux_promoteurs
        FROM(
             SELECT COUNT(r.note) AS total_notes
             FROM retour_client r
            ) AS total,

            (
             SELECT COUNT (r.note) AS nb_promoteurs
             FROM retour_client r
             WHERE r.note BETWEEN 9 AND 10
            ) AS promoteurs
     ) AS taux_p,

     (SELECT detracteurs.nb_detracteurs*100/total.total_notes AS taux_detracteurs
       FROM(
            SELECT COUNT(r.note) AS total_notes
            FROM retour_client r
           ) AS total,

           (
            SELECT COUNT(r.note) AS nb_detracteurs
            FROM retour_client r
            WHERE r.note BETWEEN 0 AND 6
           ) AS detracteurs
     ) AS taux_d;
```

- **Méthode 2** : calcul du NPS en regroupant le total de retours clients, les promoteurs (notes 9–10) et les détracteurs (notes 0–6) dans une seule requête grâce à l'utilisation de CASE WHEN pour effectuer les comptages conditionnels de manière efficace;
```sql
SELECT ROUND((nb_promoteurs*100/total_notes) - (nb_detracteurs*100/total_notes),2) AS NPS
FROM (
      SELECT
      COUNT(*) AS total_notes,
      SUM(CASE WHEN r.note >=9 THEN 1 ELSE 0 END) AS nb_promoteurs,
      SUM(CASE WHEN r.note <=6 THEN 1 ELSE 0 END) AS nb_detracteurs
      FROM retour_client r
      WHERE r.note IS NOT NULL
      ) AS calcul;
``` 

On préfèrera la méthode 2 car elle est plus rapide à mettre en oeuvre et limite le risque d'erreur ou de se perdre dans les sous-requêtes.

Finalement, le NPS global est de 31. Si le NPS est supérieur à 0 c'est que le nombre de clients satisfaits (promoteurs) est majoritaire donc que le niveau de saatisfaction est bon.

## 15) Calcul du NPS par type d'expérience client
```sql
SELECT
```

## 16) Quelle source de retours clients a le plus d'avis ? 
```sql
SELECT
```

## 17) Quelle type d'expérience client (libelle_categorie) a la meilleure note moyenne ? Quel est le taux de retour client par expérience ?
```sql
SELECT
```

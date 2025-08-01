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
SELECT 
```

## 12) Quels magasins ont une note inférieure à la moyenne des magasins ?
```sql
SELECT
```

## 13) Quelles typologies de produits ont amélioré leur note moyenne entre ler et le 2ème trimestre 2021 ?
```sql
SELECT
```

## 14) Quels magasins ont une note inférieure à la moyenne des magasins ?
```sql
SELECT
```

## 15) Calcul du NPS global (Net Promoter Score) 
```sql
SELECT
```

## 16) Calcul du NPS par type d'expérience client
```sql
SELECT
```

## 17) Quelle source de retours clients a le plus d'avis ? 
```sql
SELECT
```

## 18) Quelle type d'expérience client (libelle_categorie) a la meilleure note moyenne ? Quel est le taux de retour client par expérience ?
```sql
SELECT
```

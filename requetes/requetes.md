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
AND LOWER(p.titre_produit) = "tv"; --Ici, LOWER sert à s'assurer qu'on prendra toutes les mentions à TV peu importe comment elles sont écrites (tv, Tv, tV, TV) en les ramenant toutes à "tv".
```

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







## 6) Quels sont les 5 contrats avec les surfaces les plus élevées ?
```sql
SELECT contrat_ID, surface
FROM contrat
ORDER BY surface DESC
LIMIT 5;
```

## 7) Quel est le prix moyen de la cotisation mensuelle ?
```sql
SELECT AVG(prix_cotisation_mensuel) AS cotisation_moyenne
FROM contrat;
```

## 8) Quel est le nombre de contrats pour chaque catégories de prix de la valeur déclarée des biens ?
```sql
SELECT valeur_declaree_biens, COUNT(DISTINCT contrat_ID) AS nb_contrats_valeur
FROM contrat
GROUP BY valeur_declaree_biens;
```

## 9) Quels sont les 10 départements où le prix moyen de la cotisation est le plus élevé ?
```sql
SELECT r.dep_nom, r.dep_code, AVG(c.prix_cotisation_mensuel) AS moyenne_prix_dpt
FROM contrat c 
JOIN region r ON c.code_postal = r.code_postal
GROUP BY r.dep_nom, r.dep_code
ORDER BY moyenne_prix_dpt DESC
LIMIT 10;
```

## 10) Quel est le nombre de contrats avec des formules "Integral" pour la région Pays de la Loire ?
```sql
SELECT COUNT(DISTINCT c.contrat_ID) AS nb_contrats_integral_pdl
FROM contrat c
JOIN region r ON c.code_postal = r.code_postal
WHERE c.formule = "Integral"
AND r.reg_nom = "Pays de la Loire";
```

## 11) Liste des communes ayant eu au moins 150 contrats ?
```sql
SELECT commune, COUNT(DISTINCT contrat_ID) AS nb_contrats_uniques
FROM contrat
GROUP BY commune
HAVING nb_contrats_uniques >= 150;
```

## 12) Quel est le nombre de contrats pour chaque région ?
```sql
SELECT r.reg_nom, COUNT(DISTINCT c.contrat_ID) AS nb_contrats_regions
FROM contrat c
JOIN region r ON c.code_postal = r.code_postal
GROUP BY r.reg_nom;
```


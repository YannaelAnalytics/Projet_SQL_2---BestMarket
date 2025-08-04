## Requêtes du Projet

## 1) Calculer le nombre de retours clients sur la livraison
```sql
SELECT COUNT (DISTINCT cle_retour_client) AS nb_retours_livraison
FROM retour_client
WHERE libelle_categorie = "livraison";
```

La livraison cumule 639 retours client.

## 2) Etablir la liste des notes des clients sur les réseaux sociaux sur les TV
```sql
SELECT
  r.cle_retour_client,
  r.note,
  r.libelle_source,
  p.titre_produit 
FROM retour_client r
JOIN produit p ON r.cle_produit = p.cle_produit
WHERE LOWER(r.libelle_source) = "réseaux sociaux" 
AND LOWER(p.titre_produit) = "tv";
```

Ici, LOWER sert à s'assurer qu'on prendra toutes les mentions à TV peu importe comment elles sont écrites (tv, Tv, tV, TV) en les ramenant toutes à "tv".

Le nombre de retours sur les TV s'élève à 4 avec des notes allant de 8 à 10. Les clients semblent satisfaits par le produit.

## 3) Calculer la note moyenne pour chaque typologie de produit (Ordre décroissant)
```sql
SELECT ROUND(AVG(r.note),2) AS note_moyenne, p.typologie_produit
FROM retour_client r
JOIN produit p ON r.cle_produit = p.cle_produit
GROUP BY p.typologie_produit
ORDER BY note_moyenne DESC;
```
La catégorie "High Tech" arrive en tête avec une note moyenne de 8,16. Vient ensuite la catégorie "Loisirs" qui atteint une note moyenne de 8,09, puis la catégorie "Alimentaire" avec 8,04 et enfin "Maison" avec 7,85.

Ce sont globalement de très bonnes moyenne qui attestent d'une bonne qualité des produits et d'une bonne satisfaction client. La note moyenne générale est de 8,05.

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
Le classement est le suivant :
1 - Magasin 75 à Paris 14 avec une note moyenne de 8,73
2 - Magasin 78 de Saint-Pierre-du-Perray avec une note moyenne de 8,55
3 - Magasin 62 à Paris 19 avec une note moyenne de 8,5
4 - Magasin 23 à Paris 11 avec une note moyenne de 8,48
5 - Magasin 19 à Coulommiers avec une note moyenne de 8,45

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
HAVING nb_feedbacks > 12
ORDER BY nb_feedbacks DESC;
```
Les magasins 67 (Eragny), 45 (Paris 12) et 63 (Ivry-sur-Seine) cumulent plus de 12 feedbacks sur le drive avec respectivement 14, 13 et 13 feedbacks sur cette catégorie de service.

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
Le classement est le suivant : 
1 - 95 : 8,14
2 - 75 : 8,11
3 - 94 : 8,06 
4 - 91 : 8,05
5 - 77 : 8,04
6 - 92 : 8,03
7 - 78 : 8,02
8 - 93 : 7,94

Une bonne satisfaction globale car peu de disparités des notes moyennes par département.

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

Si on ne veut que la catégorie qui a la meilleure note sur le SAV, à savoir la catégorie "Loisirs" avec une note moyenne de 8,51, on rajoute la ligne "LIMIT 1" en fin de requête, mais j'ai préféré faire un classement de l'ensemble ici. 

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

Cela nous donne une note moyenne sur l'ensemble des boissons vendues de 8,24. 

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
Avec la commande "strftime" et en choisissant "%w", on convertit une date en nombre en attribuant un numéro à chaque jour de la semaine. Dans notre table de résultats, le jour 0 est le dimanche et le lundi est le 1.

On voit que le jour où les clients sont le plus satisfaits est le samedi avec 8,34, quand le lundi arrive en queue de classement avec une note moyenne de 7,74.

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
Si on veut uniquement le mois avec le plus grand nombre de retours, rajouter une ligne "LIMIT 1;" en fin de requête. Ici, le mois avec le plus de retours sur le SAV est octobre avec 55 feedbacks.

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
Le taux de recommandation client (réponse positive à la question "TRecommanderiez-vous ce magasin ?") est de 90% sur un total de 2326 retours.

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

Au final, on se rend compte que 37 magasins sur 84 ont une note moyenne inférieure à la moyenne générale des magasin qui est de 8,05. Ces magasins "mauvais élèves" ont des notes moyenne qui vont de 8,04 à 7,28 pour le pire. Ce n'est pas catastrophique dans l'absolu car on reste quand même sur une note moyenne élevée.

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
Les catégories Loisirs (8 --> 8,45) et Alimentaire (7,99 --> 8,05) ont vu leur note moyenne augmenter entre le T1 2021 et le T2 2021.

## 14) Calcul du NPS global (Net Promoter Score) 

Cette requête permet de calculer le Net Promoter Score (NPS) pour chaque catégorie d’expérience client. Elle mesure la satisfaction globale des clients à partir de leurs notes (promoteurs : note ≥ 9, détracteurs : note ≤ 6). J'ai pu trouver 2 méthodes possibles pour ce calcul :

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

Le NPS global qui ressort de nos requêtes est de 31. Si le NPS est supérieur à 0 c'est que le nombre de clients satisfaits (promoteurs) est majoritaire donc que le niveau de saatisfaction est bon.

## 15) Calcul du NPS par type d'expérience client

 Cette requête calcule le NPS par libelle d'expérience client et affiche les résultats classés par score décroissant :
 
```sql
SELECT
    calcul.libelle_categorie,
    ROUND((nb_promoteurs*100/total_notes) - (nb_detracteurs*100/total_notes),2) AS nps
FROM (SELECT
        r.libelle_categorie,
        COUNT(*) AS total_notes,
        SUM(CASE WHEN r.note >=9 THEN 1 ELSE 0 END) AS nb_promoteurs,
        SUM(CASE WHEN r.note <=6 THEN 1 ELSE 0 END) AS nb_detracteurs
      FROM retour_client r
      WHERE r.note IS NOT NULL
      GROUP BY r.libelle_categorie
      ) AS calcul
ORDER BY nps DESC;
```
Toutes les catégories de services ont des NPS positifs. Celle qui a le meilleur NPS est la "qualité des produits" avec un score de 35. Celle avec le moins bon score est "l'expérience en magasin" avec un score de 28, ce qui n'est vraiment pas mal du tout car toujours supérieur à 0.


## 16) Quelle source de retours clients comptabilise le plus d'avis ? 
```sql
SELECT r.libelle_source, COUNT(DISTINCT r.cle_retour_client) AS nombre_de_retours
FROM retour_client r
GROUP BY r.libelle_source
ORDER BY nombre_de_retours DESC;
```
Le canal le plus utilisé par les clients pour donner leur avis sont les emails avec 1032 retours. Viennent ensuite les réseaux sociaux (998) suivis par le téléphone (970).

## 17) Quelle type d'expérience client (libelle_categorie) a la meilleure note moyenne ? Quel est le taux de recommandation client par type d'expérience ?
```sql
SELECT
  r.libelle_categorie,
  COUNT(r.recommandation) AS nb_reco,
  ROUND(AVG(r.note),2) AS note_moyenne_par_experience,
  ROUND(SUM(CASE WHEN r.recommandation = 1 THEN 1 ELSE 0 END)*100/ COUNT(r.recommandation),2) AS taux_reco_client
FROM retour_client r
WHERE r.recommandation IS NOT NULL 
GROUP BY r.libelle_categorie
ORDER BY note_moyenne_par_experience DESC;
```

La catégorie d'expérience client avec la meilleure note moyenne est la catégorie "qualité produit" avec une note moyenne de 8,19.
La catégorie avec le meilleur taux de recommandation est quant à elle la catégorie "Livraison".

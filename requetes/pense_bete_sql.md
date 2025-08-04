# Pense-bête SQL

## 🧱 Ordre d’exécution logique en SQL

### Voici l’ordre logique dans lequel SQL exécute une requête :

- FROM → de quelle table viennent les données
- JOIN → on effectue une jointure pour ajouter des champs d'une autre table. Plusieurs types de jointures ici :
   - CROSS JOIN : permet de créer une sous-reqûete afin d'associer chaque ligne d’une table à toutes les lignes d’une autre (utile pour ajouter une valeur unique comme une moyenne globale à chaque ligne du résultat) --> Ex : FROM table1 CROSS JOIN (SELECT ...) AS moyenne
- WHERE → filtre les lignes non agrégées (les lignes brutes, donc pas d'alias)
- GROUP BY → regroupe les lignes si je fais des calculs par groupe
- HAVING → on filtre les groupes après agrégation, donc les alias
- SELECT → on choisit les colonnes à afficher
- ORDER BY → on trie le résultat final
- LIMIT → on limite le nombre de lignes affichées

---

## 🧱 Regrouper des lignes dans une catégorie qui n'existe pas dans la base de données 

Dans cet exemple on cherche la note moyenne des produits correspondants à des boissons.

Mais problème : on n'a pas de typologie_produit = "Boisson", mais uniquement "Alimentaire".

On a aussi différents titre_produit correspondant à des boissons mais ne portant pas directement la mention "boisson"(bière, soda, café, café soluble, saké...).

On force donc la création d'une colonne dans le cadre de notre requête, ici "regroupement_produit" à laquelle on attribue pour seule ligne "Boissons".

Sorte de GROUP BY forcé qui nous donne une seule note moyenne qui correspond à tout ce qui est inclus dans notre WHERE :

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

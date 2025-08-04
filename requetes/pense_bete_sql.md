# Pense-bête SQL

## 🧱 Ordre d’exécution logique en SQL

### Voici l’ordre logique dans lequel SQL exécute une requête :

- FROM → de quelle table viennent les données
  
- JOIN → on effectue une jointure pour ajouter des champs d'une autre table. Plusieurs types de jointures ici :
  
   - CROSS JOIN : Associe chaque ligne d'une table avec chaque ligne d'une autre. Utile quand on veut injecter une valeur unique (comme une moyenne globale) dans  chaque ligne du résultat :
     ```sql
     SELECT r.note, m.moyenne
     FROM retour_client r
     CROSS JOIN (SELECT AVG(note) AS moyenne FROM retour_client r) AS m;
     ```
     
  - INNER JOIN : Ne garde que les lignes correspondantes dans les deux tables. Utilisé quand on veut uniquement les données présentes dans les deux tables (ex tous les clients qui ont des commandes correspondantes) :
     ```sql
     SELECT *
     FROM commandes c
     INNER JOIN clients cl ON c.client_id = cl.id;
     ```

  - LEFT OUTTER JOIN : Garde toutes les lignes de la table de gauche, même si rien ne correspond dans celle de droite. Très utile pour voir les éléments sans correspondance, comme les clients sans commande :
   ```sql
   SELECT *  
   FROM clients cl  
   LEFT OUTTER JOIN commandes c ON cl.id = c.client_id;
   ```

  - RIGHT OUTTER JOIN : Garde toutes les lignes de la table de droite, même sans correspondance à gauche. Mêmes usages que LEFT JOIN mais dans l’autre sens (commandes sans id client associé, donc clients sans inscriptions) :
    ```sql
    SELECT *
    FROM commandes c
    RIGHT OUTTER JOIN clients cl ON cl.id = c.client_id;
    ```

  - FULL OUTER JOIN : Garde toutes les lignes des deux tables, avec NULL là où il n’y a pas de correspondance. Pratique pour comparer ou fusionner deux jeux de données sans rien perdre.
   ```sql
   SELECT *
   FROM clients cl
   FULL OUTER JOIN commandes c ON c.id = cl.client_id;
   ```

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

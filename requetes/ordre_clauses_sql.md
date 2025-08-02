# 🧱 Ordre d’exécution logique en SQL

## Voici l’ordre logique dans lequel SQL exécute une requête :

- FROM → de quelle table viennent les données
- JOIN → on effectue une jointure pour ajouter des champs d'une autre table
- WHERE → filtre les lignes non agrégées (les lignes brutes, donc pas d'alias)
- GROUP BY → regroupe les lignes si je fais des calculs par groupe
- HAVING → on filtre les groupes après agrégation, donc les alias
- SELECT → on choisit les colonnes à afficher
- ORDER BY → on trie le résultat final
- LIMIT → on limite le nombre de lignes affichées

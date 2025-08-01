# 📘 Dictionnaire de données

## Table : `produit`

| Champ                     | Type  | Taille | Contrainte   | Description                                          |
|---------------------------|-------|--------|--------------|------------------------------------------------------|
| `cle_produit`             | INT   |        | Clé Primaire | Identifiant unique pour les produits                 |
| `typologie_produit`       | CHAR  | 50     |              | Typologie des produits (Alimentaire, High-tech etc…) |
| `titre_produit`           | CHAR  | 50     |              | Libellé des produits                                 |


## Table : `ref_magasin`

| Champ                 | Type   | Taille | Contrainte   | Description                            |
|-----------------------|--------|--------|--------------|----------------------------------------|
| `ref_magasin`         | INT    |        | Clé Primaire | Identifiant unique pour chaque magasin |
| `departement`         | INT    |        |              | Code du département                    |
| `departement_commune` | CHAR   | 25     |              | Code INSEE de la commune               |
| `libelle_de_commune`  | CHAR   | 50     |              | Nom de la commune                      |
| `population`          | INT    |        |              | Nombre d'habitants de la commune       |
| `latitude`            | DOUBLE | 25     |              | Latitude de l'emplacement du magasin   |
| `longitude`           | DOUBLE | 25     |              | Longitude de l'emplacement du magasin  |

## Table : `retour_client`

| Champ                     | Type | Taille | Contrainte   | Description                                                                                                                                  |
|---------------------------|------|--------|--------------|----------------------------------------------------------------------------------------------------------------------------------------------|
| `cle_retour_client`       | INT  |        | Clé Primaire | Identifiant unique des retours clients                                                                                                       |
| `note`                    | INT  |        |              | Note du client entre 0 et 10 en réponse à la question "Quelle est la probabilité que vous recommandiez notre entreprise à votre entourage ?" |
| `cle_produit`             | INT  |        |              | Identifiant unique pour les produits                                                                                                         |
| `ref_magasin`             | INT  |        |              | Identifiant unique pour chaque magasin                                                                                                       |
| `date_achat`              | DATE |        |              | Date à laquelle l'achat du client a eu lieu                                                                                                  |
| `libelle_source`          | CHAR | 50     |              | Libellé de la source d’où provient le retour client (Réseaux sociaux, téléphone, email)                                                      |
| `libelle_categorie`       | CHAR | 50     |              | Libellé de la catégorie du retour client (Drive, service après-vente, qualité produit, expérience en magasin, livraison)                     |
| `recommandation`          | BOOL |        |              | Recommandation laissée par le client à la question "Recommandez vous l'entreprise ?"  True / False                                           |

## Schéma des données 

(datasets/Schema données.PNG)

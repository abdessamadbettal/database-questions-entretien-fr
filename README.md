# 🗄️ Questions d'entretien — Bases de données & SQL

> Une collection structurée de questions d'entretien sur les bases de données, en français : **SQL**, **MySQL**, **PostgreSQL**, **MongoDB**, **Redis**, **NoSQL**, optimisation, indexation et modélisation des données.

<p align="center">
  <img alt="Langue" src="https://img.shields.io/badge/langue-fran%C3%A7ais-blue">
  <img alt="Niveau" src="https://img.shields.io/badge/niveau-junior%20%E2%86%92%20senior-green">
  <img alt="Contributions" src="https://img.shields.io/badge/contributions-bienvenues-brightgreen">
  <img alt="Licence" src="https://img.shields.io/badge/licence-MIT-lightgrey">
</p>

---

## 📖 À propos

Ce dépôt rassemble les questions les plus fréquemment posées en entretien technique sur les bases de données, accompagnées de réponses claires, d'exemples de code et de pièges courants. Il s'adresse aussi bien aux développeurs qui préparent un entretien qu'aux recruteurs qui souhaitent construire une grille d'évaluation.

L'objectif : comprendre les concepts, pas seulement les mémoriser.

## 🎯 À qui s'adresse ce dépôt

- **Candidats** préparant un entretien pour un poste de développeur back-end, data engineer, DBA ou full-stack.
- **Recruteurs et lead techniques** cherchant des questions calibrées par niveau.
- **Étudiants** souhaitant consolider leurs bases en SQL et NoSQL.

## 🗂️ Sommaire

| Section | Contenu | Niveau |
|---------|---------|--------|
| [1. Concepts fondamentaux](#1-concepts-fondamentaux) | Bases relationnelles, ACID, normalisation | Junior |
| [2. SQL](#2-sql) | Requêtes, jointures, agrégations, sous-requêtes | Junior → Confirmé |
| [3. MySQL](#3-mysql) | Moteurs, réplication, spécificités | Confirmé |
| [4. PostgreSQL](#4-postgresql) | MVCC, types avancés, extensions | Confirmé → Senior |
| [5. Indexation](#5-indexation) | B-Tree, hash, index couvrants, plans d'exécution | Confirmé |
| [6. Optimisation SQL](#6-optimisation-sql) | EXPLAIN, anti-patterns, tuning | Confirmé → Senior |
| [7. Modélisation des données](#7-modélisation-des-données) | Schémas, relations, dénormalisation | Confirmé |
| [8. NoSQL & MongoDB](#8-nosql--mongodb) | Documents, agrégation, sharding | Confirmé |
| [9. Redis](#9-redis) | Cache, structures, persistance | Confirmé |
| [10. Transactions & concurrence](#10-transactions--concurrence) | Niveaux d'isolation, verrous, deadlocks | Senior |

---

## 1. Concepts fondamentaux

<details>
<summary><strong>Qu'est-ce qu'une base de données relationnelle ?</strong></summary>

Une base de données relationnelle organise les données en **tables** (relations) composées de lignes (tuples) et de colonnes (attributs). Les tables sont liées entre elles par des **clés** (primaires et étrangères), et on interroge l'ensemble avec le langage **SQL**.

Exemples : MySQL, PostgreSQL, Oracle, SQL Server.
</details>

<details>
<summary><strong>Que signifie ACID ?</strong></summary>

ACID décrit les garanties d'une transaction fiable :

- **A**tomicité — la transaction est exécutée entièrement ou pas du tout.
- **C**ohérence — la base passe d'un état valide à un autre état valide.
- **I**solation — les transactions concurrentes ne se perturbent pas.
- **D**urabilité — une fois validée, la transaction survit à une panne.
</details>

<details>
<summary><strong>Différence entre clé primaire et clé étrangère ?</strong></summary>

- La **clé primaire** identifie de façon unique chaque ligne d'une table (non nulle, unique).
- La **clé étrangère** est une colonne qui référence la clé primaire d'une autre table, garantissant l'**intégrité référentielle**.
</details>

<details>
<summary><strong>Qu'est-ce que la normalisation ? Citez les premières formes normales.</strong></summary>

La normalisation réduit la redondance et les anomalies de mise à jour.

- **1FN** — valeurs atomiques, pas de groupes répétés.
- **2FN** — 1FN + aucune dépendance partielle sur une clé composite.
- **3FN** — 2FN + aucune dépendance transitive entre attributs non-clés.

La **dénormalisation** est l'opération inverse, faite volontairement pour gagner en performance de lecture.
</details>

---

## 2. SQL

<details>
<summary><strong>Différence entre <code>WHERE</code> et <code>HAVING</code> ?</strong></summary>

- `WHERE` filtre les lignes **avant** l'agrégation.
- `HAVING` filtre les groupes **après** un `GROUP BY`.

```sql
SELECT departement, COUNT(*) AS nb
FROM employes
WHERE actif = 1            -- filtre les lignes
GROUP BY departement
HAVING COUNT(*) > 5;       -- filtre les groupes
```
</details>

<details>
<summary><strong>Expliquez les différents types de jointures.</strong></summary>

- `INNER JOIN` — lignes présentes dans les deux tables.
- `LEFT JOIN` — toutes les lignes de gauche + correspondances à droite (NULL sinon).
- `RIGHT JOIN` — symétrique du LEFT.
- `FULL OUTER JOIN` — toutes les lignes des deux tables.
- `CROSS JOIN` — produit cartésien.

```sql
SELECT c.nom, cmd.total
FROM clients c
LEFT JOIN commandes cmd ON cmd.client_id = c.id;
```
</details>

<details>
<summary><strong>Différence entre <code>UNION</code> et <code>UNION ALL</code> ?</strong></summary>

`UNION` supprime les doublons (et trie, donc plus coûteux), `UNION ALL` conserve toutes les lignes. Préférez `UNION ALL` quand vous savez qu'il n'y a pas de doublons.
</details>

<details>
<summary><strong>Qu'est-ce qu'une fonction fenêtre (window function) ?</strong></summary>

Elle effectue un calcul sur un ensemble de lignes liées à la ligne courante, **sans** réduire le nombre de lignes (contrairement à `GROUP BY`).

```sql
SELECT nom, salaire,
       RANK() OVER (PARTITION BY departement ORDER BY salaire DESC) AS rang
FROM employes;
```
</details>

<details>
<summary><strong>Différence entre <code>DELETE</code>, <code>TRUNCATE</code> et <code>DROP</code> ?</strong></summary>

- `DELETE` — supprime des lignes (avec `WHERE`), journalisé, peut être annulé dans une transaction.
- `TRUNCATE` — vide la table entière, beaucoup plus rapide, non filtrable.
- `DROP` — supprime la table elle-même (structure incluse).
</details>

---

## 3. MySQL

<details>
<summary><strong>Différence entre InnoDB et MyISAM ?</strong></summary>

| | InnoDB | MyISAM |
|---|--------|--------|
| Transactions | Oui (ACID) | Non |
| Clés étrangères | Oui | Non |
| Verrouillage | Au niveau ligne | Au niveau table |
| Crash recovery | Oui | Limité |

**InnoDB** est le moteur par défaut et recommandé dans la quasi-totalité des cas.
</details>

<details>
<summary><strong>Comment fonctionne la réplication MySQL ?</strong></summary>

Un serveur **primaire** (master) écrit les modifications dans son binary log ; un ou plusieurs serveurs **réplicas** (slaves) rejouent ces logs. Modes principaux : asynchrone (par défaut), semi-synchrone et synchrone (via Group Replication).
</details>

<details>
<summary><strong>Différence entre <code>CHAR</code> et <code>VARCHAR</code> ?</strong></summary>

`CHAR(n)` réserve toujours `n` caractères (rembourrés d'espaces), idéal pour des longueurs fixes. `VARCHAR(n)` ne stocke que la longueur réelle + un préfixe de longueur, idéal pour les chaînes variables.
</details>

---

## 4. PostgreSQL

<details>
<summary><strong>Qu'est-ce que le MVCC ?</strong></summary>

**Multi-Version Concurrency Control** : PostgreSQL conserve plusieurs versions d'une ligne pour permettre lectures et écritures concurrentes sans verrous bloquants. Les versions obsolètes sont nettoyées par le processus `VACUUM`.
</details>

<details>
<summary><strong>Pourquoi <code>VACUUM</code> est-il nécessaire ?</strong></summary>

Comme une mise à jour crée une nouvelle version de ligne (laissant l'ancienne morte), `VACUUM` récupère cet espace, met à jour les statistiques (`ANALYZE`) et évite le *transaction ID wraparound*. `autovacuum` l'exécute automatiquement.
</details>

<details>
<summary><strong>Citez des types de données avancés propres à PostgreSQL.</strong></summary>

`JSONB` (JSON binaire indexable), `ARRAY`, `HSTORE`, `UUID`, types géométriques, `tsvector` (recherche plein texte), types `RANGE`, et la création de types personnalisés / `ENUM`.
</details>

<details>
<summary><strong>Différence entre index <code>GIN</code> et <code>GiST</code> ?</strong></summary>

`GIN` est optimisé pour les valeurs composites (JSONB, tableaux, recherche plein texte) — lectures rapides, écritures plus lentes. `GiST` est plus généraliste (données géométriques, recherche par proximité) avec un meilleur équilibre lecture/écriture.
</details>

---

## 5. Indexation

<details>
<summary><strong>Qu'est-ce qu'un index et quel est son coût ?</strong></summary>

Un index est une structure de données (souvent un **B-Tree**) qui accélère les recherches au prix d'un **ralentissement des écritures** (INSERT/UPDATE/DELETE) et d'un **espace disque supplémentaire**. Il ne faut donc indexer que ce qui est utile.
</details>

<details>
<summary><strong>Qu'est-ce qu'un index composite ? L'ordre des colonnes compte-t-il ?</strong></summary>

Oui, l'ordre est crucial. Un index sur `(a, b)` est utilisable pour filtrer sur `a` seul ou sur `a` + `b`, mais **pas** pour filtrer sur `b` seul. C'est la règle du *préfixe le plus à gauche* (leftmost prefix).
</details>

<details>
<summary><strong>Qu'est-ce qu'un index couvrant (covering index) ?</strong></summary>

Un index qui contient **toutes** les colonnes nécessaires à une requête, ce qui permet d'y répondre sans accéder à la table (*index-only scan*).
</details>

<details>
<summary><strong>Pourquoi un index peut-il ne pas être utilisé ?</strong></summary>

Fonction appliquée sur la colonne indexée (`WHERE YEAR(date) = 2024`), faible sélectivité, conversion de type implicite, opérateur `LIKE '%mot'`, ou statistiques obsolètes.
</details>

---

## 6. Optimisation SQL

<details>
<summary><strong>Comment analyser la performance d'une requête ?</strong></summary>

Avec `EXPLAIN` (plan estimé) et `EXPLAIN ANALYZE` (plan réel avec temps mesurés). On y repère les *seq scan* coûteux, les jointures mal estimées et les tris en mémoire/disque.

```sql
EXPLAIN ANALYZE
SELECT * FROM commandes WHERE client_id = 42;
```
</details>

<details>
<summary><strong>Citez des anti-patterns SQL fréquents.</strong></summary>

- `SELECT *` au lieu des colonnes nécessaires.
- Fonctions sur colonnes indexées dans le `WHERE`.
- Problème **N+1** (une requête par élément d'une boucle).
- `LIKE '%terme%'` non sargable.
- Sous-requêtes corrélées remplaçables par des jointures.
- Pagination par `OFFSET` élevé (préférer la pagination par curseur / keyset).
</details>

<details>
<summary><strong>Qu'est-ce que le problème N+1 et comment le résoudre ?</strong></summary>

C'est exécuter 1 requête pour récupérer N éléments, puis N requêtes supplémentaires pour leurs détails. On le résout par un `JOIN`, un `IN (...)` ou l'*eager loading* de l'ORM.
</details>

---

## 7. Modélisation des données

<details>
<summary><strong>Quand dénormaliser un schéma ?</strong></summary>

Quand les lectures sont massivement plus fréquentes que les écritures et que les jointures deviennent un goulot d'étranglement. On accepte alors une redondance contrôlée (et le coût de la cohérence) en échange de performances de lecture.
</details>

<details>
<summary><strong>Comment modéliser une relation plusieurs-à-plusieurs ?</strong></summary>

Avec une **table de jonction** (table d'association) contenant les clés étrangères des deux tables liées, par exemple `etudiants` ↔ `cours` via `inscriptions(etudiant_id, cours_id)`.
</details>

<details>
<summary><strong>Différence entre modèle relationnel et modèle documentaire ?</strong></summary>

Le relationnel répartit les données normalisées entre plusieurs tables jointes à la lecture. Le documentaire (MongoDB) regroupe les données liées dans un seul document, optimisant la lecture au prix d'une éventuelle duplication.
</details>

---

## 8. NoSQL & MongoDB

<details>
<summary><strong>Quelles sont les grandes familles de bases NoSQL ?</strong></summary>

- **Documentaire** — MongoDB, CouchDB.
- **Clé-valeur** — Redis, DynamoDB.
- **Colonnes larges** — Cassandra, HBase.
- **Graphe** — Neo4j.
</details>

<details>
<summary><strong>Qu'est-ce que le théorème CAP ?</strong></summary>

En cas de partition réseau (**P**), un système distribué doit choisir entre **C**ohérence et **D**isponibilité (Availability). On ne peut garantir les trois simultanément.
</details>

<details>
<summary><strong>À quoi sert le pipeline d'agrégation de MongoDB ?</strong></summary>

À transformer les documents par étapes successives (`$match`, `$group`, `$sort`, `$lookup`, `$project`...), à la manière d'un `GROUP BY` SQL en plusieurs passes.

```javascript
db.commandes.aggregate([
  { $match: { statut: "payee" } },
  { $group: { _id: "$client_id", total: { $sum: "$montant" } } },
  { $sort: { total: -1 } }
]);
```
</details>

<details>
<summary><strong>Différence entre sharding et réplication ?</strong></summary>

La **réplication** copie les mêmes données sur plusieurs nœuds (disponibilité, lecture). Le **sharding** partitionne les données sur plusieurs nœuds selon une clé (scalabilité horizontale, écriture).
</details>

---

## 9. Redis

<details>
<summary><strong>Quelles structures de données Redis connaissez-vous ?</strong></summary>

Strings, Hashes, Lists, Sets, Sorted Sets (ZSet), Bitmaps, HyperLogLog, Streams et types géospatiaux.
</details>

<details>
<summary><strong>Comment Redis assure-t-il la persistance ?</strong></summary>

- **RDB** — snapshots ponctuels (compact, mais perte possible entre deux sauvegardes).
- **AOF** — journal de chaque commande d'écriture (plus durable, fichier plus gros).

On peut combiner les deux.
</details>

<details>
<summary><strong>Quelles stratégies d'éviction propose Redis ?</strong></summary>

Notamment `noeviction`, `allkeys-lru`, `volatile-lru`, `allkeys-lfu`, `volatile-ttl`. Le choix dépend de l'usage cache vs stockage.
</details>

<details>
<summary><strong>Comment éviter le cache stampede ?</strong></summary>

Verrou (mutex) sur le recalcul, expiration aléatoire (jitter sur le TTL), ou recalcul anticipé en arrière-plan avant expiration.
</details>

---

## 10. Transactions & concurrence

<details>
<summary><strong>Citez les niveaux d'isolation et leurs anomalies.</strong></summary>

| Niveau | Lecture sale | Lecture non répétable | Lecture fantôme |
|--------|:---:|:---:|:---:|
| Read Uncommitted | ✅ | ✅ | ✅ |
| Read Committed | ❌ | ✅ | ✅ |
| Repeatable Read | ❌ | ❌ | ✅* |
| Serializable | ❌ | ❌ | ❌ |

*\*InnoDB évite les lectures fantômes en Repeatable Read grâce au next-key locking.*
</details>

<details>
<summary><strong>Qu'est-ce qu'un deadlock et comment le gérer ?</strong></summary>

Deux transactions s'attendent mutuellement pour libérer un verrou. Le SGBD en détecte une et l'annule (rollback). On les réduit en accédant aux ressources dans un ordre cohérent et en gardant les transactions courtes.
</details>

<details>
<summary><strong>Verrouillage optimiste vs pessimiste ?</strong></summary>

- **Pessimiste** — verrouille la ressource dès la lecture (`SELECT ... FOR UPDATE`). Adapté en cas de forte contention.
- **Optimiste** — vérifie au moment de l'écriture qu'aucune autre transaction n'a modifié la donnée (via un numéro de version). Adapté en cas de faible contention.
</details>

---

## 🤝 Contribuer

Les contributions sont les bienvenues !

1. Forkez le dépôt.
2. Créez une branche (`git checkout -b ajout-questions-cassandra`).
3. Ajoutez vos questions/réponses en respectant le format existant (`<details>` + exemples).
4. Ouvrez une *pull request* avec une description claire.

**Conventions :** une question = un bloc `<details>`, réponses concises, exemples de code testables, et pas de réponse approximative.

## 🗺️ Feuille de route

- [ ] Section Cassandra & bases colonnes larges
- [ ] Section Elasticsearch
- [ ] Questions de *system design* orientées données
- [ ] Quiz interactifs / flashcards
- [ ] Traduction anglaise (branche `en`)

## 📚 Ressources complémentaires

- *Use The Index, Luke!* — guide d'indexation SQL
- Documentation officielle PostgreSQL, MySQL, MongoDB, Redis
- *Designing Data-Intensive Applications* — Martin Kleppmann

## 📄 Licence

Distribué sous licence **MIT**. Voir le fichier [`LICENSE`](LICENSE) pour plus de détails.

---

<p align="center">⭐ Si ce dépôt vous aide, pensez à lui laisser une étoile !</p>

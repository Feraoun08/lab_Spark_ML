# Lab  : Spark MLlib


Ce lab construit un pipeline Spark MLlib (prétraitement + clustering KMeans) sur un jeu de données de ventes en ligne, puis évalue le clustering avec le coefficient de silhouette.

## Contenu

```
tp4_Spark_Ml/
├── README.md
├── TP4_MLlib.ipynb
└── tp4_data/
    └── tp4_data/
        └── *.csv        (24 fichiers de ventes)
```

## Prérequis

| Outil | Version |
|---|---|
| Python | 3.9 à 3.12 |
| Java (JDK) | 17 ou 21 |
| PySpark | 3.5.3 |
| VS Code | extensions **Python** et **Jupyter** |

## Installation

```bash
pip install pyspark==3.5.3 numpy ipykernel
```

Vérification :

```bash
python --version
java -version
pip show pyspark
```

### Windows uniquement : Hadoop (winutils)

Sous Windows, Spark a besoin de `winutils.exe` et `hadoop.dll` pour lire des fichiers locaux. Sans eux, l'erreur `UnsatisfiedLinkError: NativeIO$Windows.access0` apparaît.

1. Créer le dossier `C:\hadoop\bin`.
2. Télécharger `winutils.exe` et `hadoop.dll` (Hadoop 3.3.x) depuis le dépôt GitHub `cdarlint/winutils` et les placer dans `C:\hadoop\bin`.
3. Le notebook définit déjà `HADOOP_HOME` et le `PATH` dans sa première cellule.

## Exécution

1. Ouvrir le dossier du lab dans VS Code.
2. Ouvrir `TP4_MLlib.ipynb`.
3. Choisir le kernel Python où `pyspark` est installé (en haut à droite).
4. Exécuter les cellules dans l'ordre (ou **Run All**).

Le chemin des données est défini à l'étape 1 du notebook. Adapte-le si ton dossier est ailleurs :

```python
.load("tp4_data/*.csv")
```

Utilise des `/` et non des `\`.

## Étapes du lab

| Étape | Action |
|---|---|
| 1-2 | Lire tous les CSV dans un DataFrame, afficher le schéma |
| 3 | Remplacer les valeurs manquantes par 0 (`na.fill(0)`) |
| 4 | Créer `day_of_week` à partir de `InvoiceDate` |
| 5 | Train = achats avant 2010-12-13, test = à partir de 2010-12-13 |
| 6 | `StringIndexer` : jour de la semaine en nombre |
| 7 | `OneHotEncoder` : supprime l'ordre artificiel entre les jours |
| 8 | `VectorAssembler` : `UnitPrice`, `Quantity`, `day_of_week_encoded` en `features` |
| 9-10 | `Pipeline` puis `fit` sur le train (l'indexer apprend les valeurs distinctes) |
| 11 | `transform` du train avec le pipeline entraîné |
| 12-13 | `KMeans` avec k = 20, entraînement sur le train transformé |
| 14 | Prédictions sur le test (transformé avec le même pipeline) |
| 15 | Coefficient de silhouette (`ClusteringEvaluator`) |

## Notions clés

- **Transformer** : prétraite un DataFrame (`transform`).
- **Estimator** : apprend à partir des données (`fit`) et produit un modèle.
- **Pipeline** : enchaîne les étapes en un seul objet.
- **Silhouette** : entre -1 et 1, plus c'est proche de 1, meilleur est le clustering.
- On fait le `fit` du pipeline sur le **train uniquement**, puis on applique `transform` au train et au test.

## Problèmes courants

| Symptôme | Cause probable | Solution |
|---|---|---|
| `UnsatisfiedLinkError ... NativeIO$Windows` | `winutils` / `hadoop.dll` absents | Voir la section Hadoop ci-dessus, puis redémarrer le kernel |
| Cellule bloquée au démarrage de Spark | Pare-feu Windows ou processus Java bloqué | Autoriser Java dans le pare-feu, terminer les `java.exe`, redémarrer le kernel |
| `Python worker failed to connect` | Mauvais interpréteur Python | Redémarrer le kernel (déjà géré par `PYSPARK_PYTHON`) |
| `Java not found` | JDK absent du PATH | Installer le JDK, définir `JAVA_HOME`, redémarrer VS Code |
| Python 3.13 | Non supporté par PySpark 3.5.3 | Utiliser Python 3.11 ou 3.12 |

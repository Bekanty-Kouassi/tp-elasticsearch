
# Exercice 0

*Les réponses sont-elles identiques d'un outil à l'autre ?**
Oui pour le contenu : avec Kibana et avec curl (avec `-u`), on obtient les mêmes
informations sur le cluster ( "name" : "es01",
  "cluster_name" : "tp-eisi",
  "cluster_uuid" : "jPmt-m2tR_mrd2QtCs68Kw",
   "version" : {
    "number" : "9.5.4",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "9170df19cae1adb107b7b489b4d82dec66d7a337",
    "build_date" : "2026-09-09T22:42:53.976833287Z",
    "build_snapshot" : false,
    "lucene_version" : "10.5.1",
    "minimum_wire_compatibility_version" : "8.19.0",
    "minimum_index_compatibility_version" : "8.0.0"
  },
  "tagline" : "You Know, for Search"}).
Seule la présentation peut changer. Tous les outils envoient la même requête
HTTP à la même API REST (port 9200).

**Code HTTP obtenu sans authentification, et message d'erreur**
HTTP/1.1 401 Unauthorized. Le corps de la réponse est une erreur `security_exception`
dont le message indique l'absence d'identifiants dans la requête
("missing authentication credentials for REST request [/]",).
La sécurité est activée : Elasticsearch refuse de répondre à une requête anonyme.

**Pourquoi Kibana n'a-t-il pas besoin du mot de passe à chaque requête ?**
Je me suis connectée une seule fois à Kibana avec l'utilisateur `elastic`.
Kibana garde ma session et transmet mes requêtes de Dev Tools à Elasticsearch
avec cette identité. Avec curl, il faut fournir les identifiants (`-u`) à
chaque appel, car il n'y a pas de session.

 ## Partie 1
  # Exercice 1.1
*quelle version tourne ?
Version cluster: "9.5.4"
**Combien de nœuds ? 
1 noeuds:  es01
**Pourquoi voyez-vous des index commençant par un point ?
Ce sont des index système (sécurité, configuration de Kibana, tableaux de bord…)

 # Exercice 1.2 
 comment évolue _version ? 
 _version : elle vaut 1 après le PUT ("result": "created"), puis passe à 2 après le _update. Chaque écriture l'incrémente. Un PUT sur un _id existant remplace tout le document, alors que _update fusionne seulement les champs fournis.

 Quel identifiant reçoit le document créé par POST essai/_doc ? 
 POST essai/_doc : Elasticsearch génère un _id aléatoire d'environ 20 caractères.

 L'index essai existait-il avant le premier PUT ?
Non. Le premier PUT l'a créé automatiquement, avec un mapping déduit.

 # Exercice 1.3

 *quel type reçoit salaire ? 
 salaire: "45000" (une chaîne) devient text avec un sous-champ keyword. Les chaînes qui ressemblent à des nombres ne sont pas converties.
 Et publication ? publication: "2026-08-02" devient date (détection automatique des dates).
 *Pourquoi le document 2 est-il accepté ?
Le document 2 est accepté car un nombre peut être indexé dans un champ text (converti en chaîne). Il n'y a pas d'erreur, mais le champ reste du texte. 
 *Quelle conséquence pour un tri ou un filtre salaire > 50000 ?
un tri sur salaire échoue (les champs text ne se trient pas). Un filtre salaire > 50000 compare des chaînes, donc l'ordre est alphabétique et faux ("9000" serait > "50000"). Le type se fixe avec le premier document et ne change plus sans recréer l'index. D'où le mapping explicite.

# Exercice 1.4
 *quelle erreur obtenez-vous ?
Erreur : 400, strict_dynamic_mapping_exception (« mapping set to strict, dynamic introduction of [champ_inconnu] … is not allowed »).

 *Pourquoi est-ce une bonne pratique en production ?
Bonne pratique en production : une faute de frappe ou un champ inattendu est rejeté tout de suite, au lieu de polluer silencieusement le schéma avec un mauvais type.

 *Nettoyez ensuite avec DELETE essai et DELETE essai2.


# Partie 2: Ingestion avec Python

 # Exo 2.2
Après avoir lancé python ingest.py :
**le nombre de documents a-t-il doublé ? 
 Non, il reste à 5 000.

**Pourquoi fixer _id avec le champ id ? 
Le même _id désigne le même document : le renvoyer écrase l'ancien. On peut donc relancer l'ingestion à volonté.
Avec des identifiants générés par Elasticsearch ? 

**Que se passerait-il avec des identifiants générés par Elasticsearch ?
Chaque relance créerait 5 000 nouveaux documents : 10 000 après la deuxième, 15 000 après la troisième.

 # Exo 2.3
**le lot entier est-il rejeté ou seulement ce document ? 
Seulement ce document. Les 5 000 autres sont acceptés.

**Quel est l'intérêt de raise_on_error=False pour un pipeline ? 
le script continue et liste les refus au lieu de s'arrêter à la première erreur.

**Régénérez ensuite le fichier propre avec python data/generate_offres.py.

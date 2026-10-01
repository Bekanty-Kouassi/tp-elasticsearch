# Réponses du TP Elasticsearch

## Exercice 0

**Les réponses sont-elles identiques d'un outil à l'autre ?**
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

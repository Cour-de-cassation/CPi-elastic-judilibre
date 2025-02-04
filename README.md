# CPi-elastic-judilibre
Ce repos sert au deploiement d'elasticsearch sur la plateforme CloudPI


---

## 📂 Pré-requis

Avant de commencer, assurez-vous d'avoir :
- Télécharger le fichier [s3-secret.yaml](helm/templates/s3-secret.yaml), changer le secret si besoin, puis l'encrypter , en utilisant l'utilitaire [SOPS](https://github.com/getsops/sops) et le réuploader sur le repos.

```shell
sops -e --age $AGE_KEY --encrypted-regex "^(data|stringData)$" s3-secret.yaml > s3-secret.enc.yaml
```
 Attention! dans le secret, la partie data, doit contenir des charactère en base64


 Pour peaufiner le deploiement n'hesiter pas a parametrer le fichier [values.yaml](helm/values.yaml), que ce soit pour les nom des services, que pour les ressources alloué au cluster elasticsearch.


 ## Ajout d'un nouveau repository stockage S3

### 1. Création du secret contenant les clés d'accès

Commencez par créer un secret Kubernetes contenant l'access key et le secret access key. Ensuite, ajoutez-les en tant qu'extraEnvVars dans le fichier [values.yaml](helm/values.yaml) comme suit:

```yaml
extraEnvVars:
- name: FOOBAR_ACCESS_KEY
    valueFrom:
      secretKeyRef:
        name: s3-credentials # Référence au secret contenant les clés
        key: AWS_ACCESS_KEY_ID
- name: FOOBAR_SECRET_KEY
    valueFrom:
      secretKeyRef:
        name: s3-credentials # Référence au secret contenant les clés
        key: AWS_SECRET_ACCESS_KEY # Ces variables sont déjà injectées via extraEnvVars
```

### 2. Configuration du client S3

Ensuite  ajouter les accès S3 dans le keystore elasticsearch de la maniere suivantes dans la section *initScripts* du fichier [values.yaml](helm/values.yaml) :
 
```yaml
initScripts:
  configure-s3-client.sh: |
    elasticsearch_set_key_value "s3.client.foobar.access_key" "${FOOBAR_ACCESS_KEY}"
    elasticsearch_set_key_value "s3.client.foobar.secret_key" "${FOOBAR_SECRET_KEY}"
```

*Noter que le nom du client S3 est arbitraire ici ("foobar"), vous pouver choisir ce que vous voulez, cependant il est recommandé d'utiliser le nom du clien S3 (Minio, AWS, GCP...).
*

Exemple d'une requete curl utilisant le client foobar :

```sh
curl -X PUT "http://elasticsearch:9200/_snapshot/foobar_repository?pretty" -H 'Content-Type: application/json' -d'
      {
        "type": "s3",
        "settings": {
          "client": "foobar",
          "bucket": "database",
          "endpoint": "api-mock-data-tooling.apps.numerique.com",
          "protocol": "https",
          "path_style_access": "true"
        }
      }
      '
```
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
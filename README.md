# Chart Helm de démonstration DSO

## Présentation

Ce chart HELM permet de déployer l'application de [démonstration Java](https://github.com/dnum-mi/dso-tuto-java)

Ce chart permet de créer :
 - Appel au chart HELM par déclaration de dépendance pour la création d'une instance PosgreSQL / Bitnami
 - Création d'un déploiement de l'image construite par DSO sur le projet [démonstration Java](https://github.com/dnum-mi/dso-tuto-java)
 - Création d'un service sur le déploiement
 - Création d'un ingress en https vers le service

Ce Chart peut servir dans le cas d'un test de déploiement comme repo d'infrastructure depuis la console DSO.

## Variables
### Applications

Fichier de variable minimal:

```
image:
  repository: quay.apps.ocp4-8.infocepo.com/ministere-interieur-demo-projets-dso-clients/app-java-forge-dso-demo # Référence de l'image DSO à déployer

ingress:
  enabled: true # Création d'un ingress
  secret:
    enabled: false # Création du secret TLS false si déjà présent
  host: dso-demo-java-helm-plec.apps.ocp4-8.infocepo.com # Nom d'host
  tls:
    - secretName: dso-demo-java-tls # Nom du secret contenant le certificat TLS
``` 

Fichier de variable complet :
```
replicaCount: 1 # Nombre de réplicas du pod

image:
  repository: quay.apps.ocp4-8.infocepo.com/ministere-interieur-demo/app-java-forge-dso-demo # Nom de l'image DSO à déployer, dépendant de l'organisation et du projet DSO créé
  pullPolicy: Always # Policy de récupération de l'image sur le node (Always : à chaque déploiement de pod, IfNotPresent : uniquement sur un node qui ne contient pas l'image en cache)
  tag: "master" # Nom du tag de l'image

imagePullSecrets: 
  - name: "quay-image-pull-secret" # Nom du secret permettant de se connecter à la registry Quay. Ne pas modifier cette valeur dans le cadre de DSO

service:
  type: ClusterIP # Type de service kubernetes à créer, ClusterIP dans la majorité des cas
  port: 8080 # Port d'exposition du service kubernetes

ingress:
  enabled: true # Création d'un ingress
  secret:
    enabled: false # Création du secret TLS false si déjà présent
  host: dso-demo-java-helm-plec.apps.ocp4-8.infocepo.com # Nom d'host
  tls:
    - secretName: dso-demo-java-tls # Nom du secret contenant le certificat TLS

resources: # Contraintes limit et requests sur les ressources du POD
  limits:
    memory: "512Mi"
    cpu: "500m"  
  requests:
    memory: "512Mi"
    cpu: "500m"  

```

### Base de données
```
# Variable liées au chart bitanmi/postgresql déclarée en dépendance
global: 
  postgresql:
    auth:
      postgresPassword: "MySupErPaSSwOrd" # Mot de passe de l'utilisateur postgreSQL
      username: "userdemo" # Utilisateur en base de données dédié à l'application
      password: "passworddemo" # Mot de passe en base de données dédié à l'application
      database: "demodb" # Nom de la base de données dédiée à l'application
```
## Utilisation dans DSO

Lors de l'utilisation dans la console DSO, une fois le projet [démonstration java](https://github.com/dnum-mi/dso-tuto-java) construit, il faut ajouter un dépôt syncrhonisé de type dépôt d'infrastructure qui pointe vers ce repo github puis aller dans ArgoCD et modifier depuis l'application ArgoCD :
App Details -> Parameters
 - image.repository : Adapter avec le nom de son image / organisation / projet
 - ingress.host : choisir une URL de déploiement non utilisée

Problèmes connus :

Les noms de ressources dans Kubernetes doivent faire moins de 63 caractères or DSO génère les noms des ressources à partir du nom de l'organisation et du nom du projet. Il peut arriver que les noms générés fassent plus de 63 caractères et empêchent le déploiement. Depuis Helm, il est possible de surcharger le nom par les 2 varaibles suivantes :
  - nameOverride
  - fullnameOverride

De la même façon pour postgresql :
  - postgresql.nameOverride
  - postgresql.fullnameOverride
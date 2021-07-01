# Hello App

##

[Gcloud Tuto](https://cloud.google.com/kubernetes-engine/docs/tutorials/configuring-domain-name-static-ip)

Le fichier deployment contient 2 descriptor

- helloweb (version 1.0 de l'image hello-app)
- helloweb2 (version 2.0 de l'image hello-app)

Le fichier service lb contient

- un service de type node port pour chaque app
- une ingress qui target les 2 apps et utilise une IP Static (préprovisionnée au préalable)
    ```
    annotations:
        kubernetes.io/ingress.global-static-ip-name: helloweb-ip
    ```
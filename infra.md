# Guide de Configuration du Runner GitHub Actions sur Kubernetes

## Sommaire

1. [Introduction](#introduction)
2. [Création d'un Registre Docker Local](#création-dun-registre-docker-local)
3. [Déploiement du Runner GitHub Actions](#déploiement-du-runner-github-actions)
4. [Construction et Push de l'Image Docker du Runner](#construction-et-push-de-limage-docker-du-runner)
5. [Mise en Place d'un Ingress Nginx pour le Load Balancing](#mise-en-place-dun-ingress-nginx-pour-le-load-balancing)
6. [Mise en Production de l'Application Front-End](#mise-en-production-de-lapplication-front-end)
7. [Vérification et Dépannage](#vérification-et-dépannage)

## Introduction

Ce guide détaille les étapes pour configurer un runner GitHub Actions auto-hébergé dans un cluster Kubernetes. Il inclut la création d'un registre Docker local, le déploiement du runner, la mise en place d'un Ingress Nginx pour le load balancing, et la mise en production d'une application front-end.

## Création d'un Registre Docker Local

1. **Déployer un Registre Docker** :
   - Utilisez le fichier YAML suivant pour créer un déploiement de registre Docker dans Kubernetes.
     ```yaml
     apiVersion: apps/v1
     kind: Deployment
     metadata:
       name: registry
       labels:
         app: registry
     spec:
       replicas: 1
       selector:
         matchLabels:
           app: registry
       template:
         metadata:
           labels:
             app: registry
         spec:
           containers:
           - name: registry
             image: registry:2
             ports:
             - containerPort: 5000
     ```

2. **Exposer le Registre via un Service** :
   - Créez un service Kubernetes pour exposer votre registre Docker.
     ```yaml
     apiVersion: v1
     kind: Service
     metadata:
       name: registry
       labels:
         app: registry
     spec:
       ports:
       - port: 5000
         targetPort: 5000
       selector:
         app: registry
     ```

## Déploiement du Runner GitHub Actions

1. **Créer un Secret pour le Token du Runner** :
   - Utilisez `kubectl` pour créer un secret contenant le token d'enregistrement du runner.
     ```bash
     kubectl create secret generic github-runner-token --from-literal=token=<TOKEN>
     ```

2. **Déployer le Runner** :
   - Utilisez le fichier YAML suivant pour déployer le runner GitHub Actions.
     ```yaml
     apiVersion: apps/v1
     kind: Deployment
     metadata:
       name: github-actions-runner
       labels:
         app: github-actions-runner
     spec:
       replicas: 1
       selector:
         matchLabels:
           app: github-actions-runner
       template:
         metadata:
           labels:
             app: github-actions-runner
         spec:
           containers:
           - name: runner
             image: registry.local:5000/mon-runner:latest
             env:
             - name: RUNNER_NAME
               value: "my-runner"
             - name: RUNNER_REPO
               value: "https://github.com/akhalildjo/supdevinci"
             - name: RUNNER_TOKEN
               valueFrom:
                 secretKeyRef:
                   name: github-runner-token
                   key: token
             volumeMounts:
             - name: runner-config
               mountPath: /actions-runner
           volumes:
           - name: runner-config
             emptyDir: {}
     ```

## Construction et Push de l'Image Docker du Runner

1. **Créer un Dockerfile** :
   - Utilisez le Dockerfile suivant pour créer l'image Docker de votre runner.
     ```Dockerfile
     FROM ubuntu:20.04

     RUN apt-get update && apt-get install -y curl

     WORKDIR /actions-runner

     RUN curl -o actions-runner-linux-x64-2.283.1.tar.gz -L https://github.com/actions/runner/releases/download/v2.283.1/actions-runner-linux-x64-2.283.1.tar.gz
     RUN tar xzf ./actions-runner-linux-x64-2.283.1.tar.gz

     CMD ["./config.sh --url https://github.com/akhalildjo/supdevinci --token APAGRRZ3ID7NTBVNZUIJRNDFKZQOM"] && ./run.sh
     ```

2. **Construire et Push l'Image** :
   - Exécutez les commandes suivantes pour construire et push l'image sur votre registre local.
     ```bash
     docker build -t registry.local:5000/mon-runner:latest .
     docker push registry.local:5000/mon-runner:latest
     ```

## Mise en Place d'un Ingress Nginx pour le Load Balancing

1. **Installer l'Ingress Nginx Controller** :
   - Utilisez Helm pour installer l'Ingress Nginx Controller dans votre cluster Kubernetes.
     ```bash
     helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
     helm repo update
     helm install ingress-nginx ingress-nginx/ingress-nginx
     ```

2. **Configurer un Ingress Resource** :
   - Créez un fichier YAML pour définir les règles d'Ingress.
   - Exemple de configuration pour rediriger le trafic vers votre application front-end :
     ```yaml
     apiVersion: networking.k8s.io/v1
     kind: Ingress
     metadata:
       name: mon-application-ingress
       annotations:
         nginx.ingress.kubernetes.io/rewrite-target: /
     spec:
       rules:
       - host: monapp.exemple.com
         http:
           paths:
           - path: /
             pathType: Prefix
             backend:
               service:
                 name: nom-service-front-end
                 port:
                   number: 80
     ```

## Mise en Production de l'Application Front-End

1. **Déployer l'Application Front-End** :
   - Utilisez un fichier YAML de déploiement pour déployer votre application front-end dans Kubernetes.
   - Assurez-vous que le service correspondant est correctement configuré pour être accessible via l'Ingress.

2. **Configurer le Domaine et le SSL** :
   - Configurez un enregistrement DNS pour pointer vers l'adresse IP de votre Ingress Controller.
   - Utilisez cert-manager ou une méthode similaire pour configurer le SSL et obtenir un certificat HTTPS.

## Vérification et Dépannage

...

---

Ce document fournit un aperçu complet des étapes pour configurer un runner GitHub Actions sur Kubernetes, avec un focus sur la mise en place d'un Ingress Nginx pour le load balancing et la mise en production d'une application front-end. Pour des détails spécifiques et des configurations supplémentaires, référez-vous à la documentation officielle de Kubernetes, Nginx Ingress, et des outils associés.


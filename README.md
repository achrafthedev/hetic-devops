# 🚀 Projet Avengers - Déploiement Automatisé

## 📌 Description
Ce projet vise à automatiser le **déploiement d'une infrastructure complète** sur AWS en utilisant **Terraform**, **GitLab CI/CD**, et des outils de monitoring tels que **Prometheus & Grafana**. 

L'infrastructure comprend :
- 📱 Une application principale déployée sur EC2
- 🗄️ Une base de données PostgreSQL sur une instance dédiée
- 📊 Un serveur de monitoring avec Prometheus & Grafana
- 🔁 Une pipeline CI/CD pour gérer l'intégration et le déploiement continu

---

## 🏗️ Technologies Utilisées
- **Infrastructure** : Terraform
- **CI/CD** : GitLab CI/CD
- **Back-End** : Node.js & Express
- **Base de données** : PostgreSQL
- **Monitoring** : Prometheus & Grafana
- **Stockage** : AWS EC2 & S3

---

## ⚙️ Installation et Configuration

### 🔹 **Prérequis**
- **Terraform** installé sur votre machine
- **GitLab Runner** configuré si vous voulez exécuter les pipelines localement
- **Accès AWS** avec des clés IAM configurées
- **Docker** et **Docker Compose** installés sur les machines cibles

### 🔹 **Cloner le projet**
```bash
git clone https://github.com/votre-repo/projet-avengers.git
cd projet-avengers
```

### 🔹 **Configurer AWS et Terraform**
1. Modifier les fichiers de configuration dans `cicd/terraform`
2. Initialiser Terraform :
   ```bash
   cd cicd/terraform
   terraform init
   ```
3. Appliquer la configuration :
   ```bash
   terraform apply -auto-approve
   ```

### 🔹 **Déploiement avec GitLab CI/CD**
Une fois Terraform appliqué, le **pipeline GitLab** gère le déploiement automatiquement :
1. **Build et tests** (`start`)
2. **Génération de clés SSH** (`generate-ssh`)
3. **Analyse de code avec SonarQube** (`sonarqube`)
4. **Provisioning AWS avec Terraform** (`terraform`)
5. **Migration de la base de données** (`deploy`)
6. **Déploiement de l'application sur EC2** (`deploy`)
7. **Déploiement du monitoring** (`deploy_monitoring`)

---

## 📡 Monitoring avec Prometheus & Grafana
- Une instance EC2 héberge **Prometheus** (métriques) et **Grafana** (visualisation)
- Pour accéder à Grafana :
  ```bash
  http://<public-ip-monitoring>:3000
  ```
  *Par défaut, login : `admin` / `admin`*

---

## 🛠️ Dépannage & Erreurs Connues

### ❌ **Erreur : `getaddrinfo null: Name does not resolve`**
- Cause : Terraform n'a pas généré d'IP correcte
- Solution :
  ```bash
  terraform output -json | jq .
  ```

### ❌ **Erreur SSH `Host key verification failed`**
- Solution :
  ```bash
  ssh-keyscan -H <instance-public-ip> >> ~/.ssh/known_hosts
  ```

### ❌ **Erreur `yaml invalid` sur GitLab**
- Vérifiez votre `.gitlab-ci.yml` via **CI Lint** sur GitLab.

---

## 🤝 Contribution
Les contributions sont les bienvenues !  
Merci de respecter la convention de commits et d'ouvrir une **pull request** avant de fusionner.

---

## 📄 Licence
Ce projet est sous licence **MIT**.

---

## 📌 First Description (GitHub)
> 🚀 **Projet Avengers** - Infrastructure automatisée avec Terraform, CI/CD GitLab et monitoring avec Prometheus & Grafana.

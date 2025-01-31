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


# 🔄 Migration de GitLab CI/CD vers GitHub Actions

## 📌 Introduction
Ce guide explique comment migrer un projet utilisant **GitLab CI/CD** vers **GitHub Actions**.  
Il couvre la migration du **pipeline CI/CD**, la configuration des **secrets**, et l’activation de **GitHub Actions**.

---

## 1️⃣ Cloner et Migrer le Dépôt GitLab vers GitHub
Si votre projet est hébergé sur **GitLab**, suivez ces étapes :

1. **Créer un nouveau repository** sur GitHub.
2. **Dans votre terminal, exécutez** :
   ```bash
   git clone --mirror https://gitlab.com/user/projet-avengers.git
   cd projet-avengers.git
   git remote set-url origin https://github.com/user/projet-avengers.git
   git push --mirror origin
   ```
3. **Votre code est maintenant sur GitHub !** 🎉

---

## 2️⃣ Convertir `.gitlab-ci.yml` en GitHub Actions

### 📌 Création du pipeline GitHub Actions
GitHub Actions utilise **des workflows** situés dans `.github/workflows/`.

📌 **Créer les fichiers nécessaires :**
```bash
mkdir -p .github/workflows
touch .github/workflows/ci.yml
```

📄 **Ajouter le contenu suivant dans `ci.yml` :**
```yaml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main
      - master
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Install dependencies
        run: |
          npm install
          npm run build

  terraform:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v2
        with:
          terraform_version: 1.10.5

      - name: Terraform Init & Apply
        run: |
          cd cicd/terraform
          terraform init
          terraform apply -auto-approve
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

  deploy_app:
    needs: terraform
    runs-on: ubuntu-latest
    steps:
      - name: Deploy application
        run: |
          scp -i ssh_keys/id_rsa docker-compose.yml ubuntu@$(jq -r '.app.value.public_ip' cicd/terraform/ec2_instances.json):/home/ubuntu/
          ssh -i ssh_keys/id_rsa ubuntu@$(jq -r '.app.value.public_ip' cicd/terraform/ec2_instances.json) << 'EOF'
            sudo apt update && sudo apt install -y docker-compose
            cd /home/ubuntu
            docker-compose down || true
            docker-compose pull
            docker-compose up -d --force-recreate
          EOF

  deploy_monitoring:
    needs: terraform
    runs-on: ubuntu-latest
    steps:
      - name: Deploy Prometheus & Grafana
        run: |
          scp -i ssh_keys/id_rsa scripts/install_prometheus_grafana.sh ubuntu@$(jq -r '.monitoring.value.public_ip' cicd/terraform/ec2_instances.json):/home/ubuntu/
          ssh -i ssh_keys/id_rsa ubuntu@$(jq -r '.monitoring.value.public_ip' cicd/terraform/ec2_instances.json) << 'EOF'
            chmod +x /home/ubuntu/install_prometheus_grafana.sh
            sudo /home/ubuntu/install_prometheus_grafana.sh
          EOF
```

---

## 3️⃣ Ajouter les Secrets GitHub
GitLab utilisait **CI/CD Variables**, mais sur GitHub, utilisez **Actions Secrets**.

📌 **Ajoutez ces secrets dans : `Settings > Secrets and variables > Actions`** :
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`

---

## 4️⃣ Activer GitHub Actions
Une fois `ci.yml` ajouté, **GitHub Actions s'exécutera automatiquement**.

📌 **Vérifiez :**
1. Allez dans **l'onglet "Actions"** sur GitHub.
2. Cliquez sur **"CI/CD Pipeline"** et vérifiez son exécution.

---

## 🛠️ Dépannage & Erreurs Connues

### ❌ **Erreur : `getaddrinfo null: Name does not resolve`**
- Cause : Terraform n’a pas généré d’IP correcte.
- Solution :
  ```bash
  terraform output -json | jq .
  ```

### ❌ **Erreur SSH `Host key verification failed`**
- Solution :
  ```bash
  ssh-keyscan -H <instance-public-ip> >> ~/.ssh/known_hosts
  ```

### ❌ **Erreur `yaml invalid` sur GitHub Actions**
- Vérifiez la syntaxe avec **"YAML Validator"** ou testez avec :
  ```bash
  act
  ```

---

## 🎯 Conclusion
✔ **Dépôt migré sur GitHub**  
✔ **Pipeline GitLab CI/CD converti en GitHub Actions**  
✔ **Secrets configurés dans GitHub**  
✔ **Déploiement Terraform et de l'application automatisé**  


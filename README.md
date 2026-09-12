# Projet d'évaluation de sécurité — OWASP Juice Shop

**Étudiant** : [Votre nom]
**Cours** : Sécurité des Données — Licence 3 Cybersécurité
**Date** : Septembre 2026

---

## 📋 Description

Évaluation complète de sécurité de l'application **OWASP Juice Shop v20.1.1** dans le cadre de l'examen final. Le projet couvre :

- Identification de 5 vulnérabilités applicatives (V1-V5)
- Classification CWE et analyse d'impact CIA
- Remédiation et vérification des correctifs
- Pipeline CI/CD Jenkins automatisé (SCA + DAST)
- Décision argumentée de mise en production

---

## 🎯 Vulnérabilités identifiées et corrigées

| ID | Vulnérabilité | CWE | Sévérité | Statut |
|----|---------------|-----|----------|--------|
| V1 | SQL Injection (login) | CWE-89 | Critical | ✅ Corrigée |
| V2 | XSS DOM-based | CWE-79 | High | ✅ Corrigée |
| V3 | IDOR / Broken Access Control | CWE-639 | High | ✅ Corrigée |
| V4 | Exposition document confidentiel | CWE-200 | Medium | ✅ Corrigée |
| V5 | crypto-js vulnérable | CWE-327 | Critical | ✅ Corrigée |

---

## 🚀 Installation et exécution

### Prérequis

- Docker ≥ 20.10
- Node.js ≥ 18
- Git

### 1. Lancer Juice Shop (version vulnérable originale)

```bash
docker run -d --name juice-shop -p 3000:3000 bkimminich/juice-shop
```

Application accessible sur : `http://localhost:3000`

### 2. Cloner le dépôt et appliquer les corrections

```bash
git clone https://github.com/magou2025/juice-shop.git
cd juice-shop
git checkout master
```

### 3. Construire l'image corrigée

```bash
# Arrêter et supprimer l'ancien conteneur
docker stop juice-shop 2>/dev/null
docker rm juice-shop 2>/dev/null

# Construire la version corrigée
docker build -t juice-shop-fixed .

# Lancer la version corrigée
docker run -d --name juice-shop-fixed -p 3000:3000 juice-shop-fixed
```

Application corrigée accessible sur : `http://localhost:3000`

---

## 🔍 Lancer les analyses de sécurité

### SCA — Analyse des dépendances

```bash
npm audit --json > reports/npm-audit-report.json
npm audit
```

### DAST — Scan dynamique OWASP ZAP

```bash
# Vérifier que Juice Shop tourne
curl -I http://localhost:3000

# Lancer le scan
zaproxy -cmd \
    -quickurl http://127.0.0.1:3000 \
    -quickout reports/zap-report.html \
    -quickprogress
```

### Revue de code manuelle

```bash
# V1 : SQL Injection
grep -n "replacements" routes/login.ts

# V2 : XSS
grep -n "searchValue" frontend/src/app/search-result/search-result.component.ts

# V3 : IDOR
grep -n "Access denied" routes/basket.ts

# V4 : Exposition FTP
grep -n "Access denied" routes/fileServer.ts
```

---

## 🔧 Pipeline CI/CD Jenkins

### Architecture

```
Checkout → Install → SCA (npm audit) → DAST (OWASP ZAP) → Archive → Décision
```

### Configuration

1. **Jenkins** installé sur `http://localhost:8080`
2. **ZAP** installé (`sudo apt install zaproxy`)
3. **Node.js** installé
4. Créer un job **Pipeline** nommé `JuiceShop-Security-CI`
5. Configurer **Pipeline script from SCM** avec l'URL du dépôt

### Jenkinsfile

Le pipeline est défini dans le fichier `Jenkinsfile` à la racine.

### Exécution

```bash
# Via Jenkins UI : cliquer sur "Build Now"
```

---

## 🛠️ Outils utilisés

| Outil | Version | Type | Usage |
|-------|---------|------|-------|
| **Docker** | 28.5.2 | Conteneurisation | Déploiement application |
| **Burp Suite** | Community | Proxy HTTP | Interception requêtes |
| **OWASP ZAP** | 2.17.0 | DAST | Scan dynamique |
| **npm audit** | - | SCA | Audit dépendances |
| **Jenkins** | 2.568.2 | CI/CD | Orchestration pipeline |
| **Git** | Latest | Versionnement | Traçabilité |

---

## 📂 Structure du dépôt

```
juice-shop/
├── README.md                                # Ce fichier
├── Jenkinsfile                              # Pipeline CI/CD
├── .gitignore                               # Exclusions Git
├── package.json                             # Dépendances (V5 corrigée)
│
├── routes/                                  # Routes Express
│   ├── login.ts                             # ✅ Corrigé V1 (SQLi)
│   ├── basket.ts                            # ✅ Corrigé V3 (IDOR)
│   └── fileServer.ts                        # ✅ Corrigé V4 (FTP)
│
├── frontend/src/app/search-result/
│   └── search-result.component.ts           # ✅ Corrigé V2 (XSS)
│
├── remediation/                             # Patches de remédiation
│   ├── v1-sqli-login.patch
│   ├── v2-xss-search.patch
│   ├── v3-idor-basket.patch
│   ├── v4-ftp-exposure.patch
│   └── v5-crypto-js.patch
│
├── reports/                                 # Rapports d'analyse
│   ├── npm-audit-report.json
│   └── zap-report.html
│
├── screenshots/                             # Captures d'écran
│
└── security-config/                         # Configuration sécurité
    └── zap-rules.conf
```

---

## 🔐 Résultats des analyses

### SCA — npm audit

| Niveau | Avant | Après | Δ |
|--------|-------|-------|---|
| Critical | 7 | 5 | **-2** |
| High | 25 | 21 | **-4** |
| Moderate | 21 | 16 | **-5** |
| Low | 3 | 3 | = |
| **Total** | **56** | **45** | **-11** |

### DAST — OWASP ZAP

| Risk Level | Nombre |
|------------|--------|
| High | **0** ✅ |
| Medium | 2 |
| Low | 1 |
| Informational | 2 |

---

## ✅ Décision de déploiement

### 🟢 ACCEPT WITH CONDITIONS

**Justification** :
- 5 vulnérabilités critiques applicatives corrigées et vérifiées
- ZAP : 0 vulnérabilité High sur l'application corrigée
- SCA : -11 vulnérabilités dont l'élimination de la faille critique `crypto-js`

**Conditions obligatoires** :
1. Documenter les 5 Critical résiduelles (fix indisponible / dev only)
2. Corriger les 2 Medium ZAP (CSP + CORS) sous 30 jours
3. Pipeline Jenkins obligatoire à chaque commit
4. Monitoring post-déploiement (WAF + logs centralisés)
5. Revue de sécurité trimestrielle

---

## 📄 Licence

Projet réalisé dans le cadre pédagogique de l'examen final de Sécurité des Données.
OWASP Juice Shop est distribué sous licence MIT.

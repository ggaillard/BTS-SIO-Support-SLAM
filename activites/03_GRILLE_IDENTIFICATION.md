# GRILLE D'IDENTIFICATION DES VULNÉRABILITÉS
## Version SLAM – Audit applicatif DevSecure

**Binôme : _________________________ | Date : _______________**

---

# RAPPELS THÉORIQUES

## Les 5 composants du SI (Laudon & Laudon)

| Composant | Description | Focus SLAM |
|-----------|-------------|------------|
| **M** - Matériel | Infrastructure physique/cloud | Serveurs AWS, MongoDB Atlas, Redis |
| **L** - Logiciel | Applications et systèmes | **Code applicatif**, frameworks, dépendances |
| **D** - Données | Informations stockées | **BDD**, fichiers S3, sauvegardes, logs |
| **P** - Procédures | Règles et processus | **CI/CD**, revue de code, documentation |
| **H** - Personnel | Ressources humaines | **Développeurs**, compétences sécurité |

> 🔑 **Mnémonique** : MLDPH = "**M**a **L**igne **D**e **P**rotection **H**umaine"

## Vulnérabilité / Menace / Risque

| Concept | Définition | Caractéristique |
|---------|-----------|-----------------|
| **Vulnérabilité** | Faiblesse du système | Intrinsèque (interne) |
| **Menace** | Ce qui peut exploiter la vulnérabilité | Externe |
| **Risque** | Probabilité × Impact | Calculable |

```
VULNÉRABILITÉ → exploitée par → MENACE → cause → IMPACT = RISQUE
```

## Formule du risque (EBIOS)

```
╔════════════════════════════════════════════╗
║  RISQUE = VRAISEMBLANCE (V) × IMPACT (I)  ║
╚════════════════════════════════════════════╝

Vraisemblance : 1-4 (difficile à facile)
Impact : 1-4 (mineur à critique)
Risque : 1-16 (accepter à URGENT)
```

## OWASP Top 10 – 2021 (catégories principales)

| Code | Catégorie | Description |
|------|-----------|-------------|
| **A01** | Broken Access Control | Accès non autorisé (IDOR) |
| **A02** | Cryptographic Failures | Données non protégées |
| **A03** | Injection | SQL, NoSQL, XSS, commandes |
| **A05** | Security Misconfiguration | Mauvaise configuration |
| **A06** | Vulnerable Components | Dépendances obsolètes |
| **A07** | Auth Failures | Authentification faible |
| **A09** | Logging Failures | Journalisation insuffisante |

---

# PARTIE 1 : ANALYSE DU CODE – VULNÉRABILITÉS APPLICATIVES

## Extrait 1 : auth.controller.js (Authentification)

```javascript
const JWT_SECRET = 'devsecure2024!';

if (user.password !== password) {
    return res.status(401).json({...});
}

const token = jwt.sign({...}, JWT_SECRET, 
    { expiresIn: '30d' }
);

console.log('Erreur:', email, password);
```

| # | Ligne(s) | Vulnérabilité | OWASP | Laudon | Justification |
|---|----------|---------------|-------|--------|---------------|
| C1 | 1 | Secret JWT en dur | A02 | L | Clé de signature stockée dans le code source |
| C2 | 3-5 | Mots de passe non hashés | A02 | D | Comparaison directe sans bcrypt/argon2 |
| C3 | 7-9 | Token trop long | A07 | P | Expiration 30 jours = trop permissif |
| C4 | 11 | Logs contenant credentials | A09 | P | Données sensibles en logs plaintext |

**Questions d'aide :**
- Où est stockée la clé JWT ? Est-ce sécurisé ? → **EN DUR dans le code !**
- Comment sont vérifiés les mots de passe ? → **Pas de hash, comparaison directe**
- Que contiennent les logs en cas d'erreur ? → **Email et password en clair**
- Quelle est la durée de validité du token ? → **30 jours = très long**

---

## Extrait 2 : project.controller.js (Projets)

```javascript
const projectId = req.params.id;
const query = `{ "_id": "${projectId}" }`;
const project = await Project.findOne(JSON.parse(query));

res.json(project);

const projects = await Project.find({
    $where: `this.title.includes('${searchTerm}')`
});
```

| # | Ligne(s) | Vulnérabilité | OWASP | Laudon | Justification |
|---|----------|---------------|-------|--------|---------------|
| C5 | 2-4 | Injection NoSQL (construction JSON) | A03 | L | Interpolation directe dans la query |
| C6 | 6 | IDOR (pas de vérification accès) | A01 | L | N'importe quel projectId accessible |
| C7 | 8-10 | Injection NoSQL ($where) | A03 | L | $where exécute JavaScript côté serveur |

**Questions d'aide :**
- Comment est construite la requête MongoDB ? → **Interpolation directe (dangereux)**
- Que fait `$where` avec une entrée utilisateur ? → **Exécute du JavaScript (RCE)**
- Un utilisateur peut-il accéder aux projets d'un autre ? → **OUI, pas de vérification accès**

---

## Extrait 3 : upload.controller.js (Upload)

```javascript
const s3 = new AWS.S3({
    accessKeyId: 'AKIAIOSFODNN7EXAMPLE',
    secretAccessKey: 'wJalrXUtnFEMI/...'
});

const fileName = file.name;
const params = {
    ACL: 'public-read'
};
```

| # | Ligne(s) | Vulnérabilité | OWASP | Laudon | Justification |
|---|----------|---------------|-------|--------|---------------|
| C8 | 1-4 | Clés AWS en dur | A02 | L | Credentials stockées en clair dans le code |
| C9 | 6 | Pas de validation nom fichier | A01 | L | Path traversal possible (../../etc/passwd) |
| C10 | 8-9 | Fichiers S3 publics | A01 | D | ACL = public-read = accessible à tous |

**Questions d'aide :**
- Où sont les clés AWS ? Est-ce sécurisé ? → **EN DUR dans le code**
- Le nom de fichier est-il validé ? → **NON, risque path traversal**
- Quelle est la visibilité par défaut des fichiers ? → **public-read = publique**

---

## Extrait 4 : Comments.jsx (Frontend)

```jsx
<p dangerouslySetInnerHTML={{ __html: comment.content }} />
```

| # | Ligne(s) | Vulnérabilité | OWASP | Laudon | Justification |
|---|----------|---------------|-------|--------|---------------|
| C11 | 1 | XSS (Cross-Site Scripting) | A03 | L | HTML non échappé, injection script possible |

**Question d'aide :**
- Que signifie `dangerouslySetInnerHTML` ? Pourquoi est-ce dangereux ? → **Exécute du HTML/JS sans validation**

---

## Extrait 5 : app.js (Configuration)

```javascript
app.use(cors());
app.use(express.json({ limit: '50mb' }));

res.status(500).json({ 
    error: err.message,
    stack: err.stack
});
```

| # | Ligne(s) | Vulnérabilité | OWASP | Laudon | Justification |
|---|----------|---------------|-------|--------|---------------|
| C12 | 1 | CORS sans restriction | A05 | P | N'importe quel site peut faire des requêtes |
| C13 | 2 | Limite JSON excessive | A05 | P | Permet les attaques DoS |
| C14 | 5-7 | Stack trace en réponse | A09 | P | Information disclosure = aide l'attaquant |

**Questions d'aide :**
- Que fait `cors()` sans paramètres ? → **Accepte TOUTES les origines**
- La limite de 50 Mo est-elle raisonnable ? → **NON, permet DoS**
- Que contient la réponse d'erreur en production ? → **Stack trace complète**

---

# PARTIE 2 : ANALYSE INFRASTRUCTURE – 5 COMPOSANTS LAUDON

## Composant M – MATÉRIEL / CLOUD

| # | Vulnérabilité | Menace associée | Justification |
|---|---------------|-----------------|---------------|
| M1 | MongoDB Atlas cluster UNIQUE | Cluster down = tout arrêté | Pas de réplication cross-région, SPOF |
| M2 | Redis instance UNIQUE | Cache down = slow queries | Pas de failover Redis |
| M3 | Load balancer AWS UNIQUE | Single region | Pas de multi-région deployment |

**Questions d'aide :**
- Y a-t-il une redondance sur MongoDB Atlas ? → **NON, cluster unique**
- Que se passe-t-il si Redis tombe ? → **Tout ralentit, cache perdu**
- L'infrastructure est-elle mono-région ? → **OUI, région unique AWS**

---

## Composant L – LOGICIEL (hors code applicatif)

| # | Vulnérabilité | Menace associée | Justification |
|---|---------------|-----------------|---------------|
| L1 | 147 packages npm (8 mois outdated) | 50+ CVE probables | Dépendances jamais mises à jour |
| L2 | Pas de npm audit | CVE non détectées | Aucun scanning des dépendances |
| L3 | Pas de Dependabot | Alertes jamais reçues | Pas d'automatisation de suivi |

**Questions d'aide :**
- Quelle version de Node.js est utilisée ? → **À déterminer**
- Depuis combien de temps les dépendances npm n'ont pas été mises à jour ? → **8 mois**
- `npm audit` a-t-il été exécuté ? → **JAMAIS**

---

## Composant D – DONNÉES

| # | Vulnérabilité | Menace associée | Justification |
|---|---------------|-----------------|---------------|
| D1 | Mots de passe en CLAIR dans MongoDB | 2000 utilisateurs compromis si breach | Aucun hash bcrypt/argon2 |
| D2 | Sauvegardes jamais testées | Restauration impossible | RPO = 24h de perte |
| D3 | Fichiers S3 publiquement accessibles | Données clients exposées | ACL public-read par défaut |

**Questions d'aide :**
- Comment sont stockés les mots de passe ? → **EN CLAIR**
- Les sauvegardes sont-elles testées ? → **NON, jamais restaurées**
- Quelle est la perte de données maximale en cas d'incident (RPO) ? → **24 heures**

---

## Composant P – PROCÉDURES / DEVOPS

| # | Vulnérabilité | Menace associée | Justification |
|---|---------------|-----------------|---------------|
| P1 | Pas de revue de code systématique | Bugs/failles non détectés | Code va directement en prod |
| P2 | Pas d'environnement staging | Erreurs en production | Déploiement en prod directement |
| P3 | Secrets partagés sur Slack | Clés AWS/passwords exposées | Communications non sécurisées |
| P4 | Pas de CI/CD scanning | CVE non détectées au build | Pipeline sans sécurité |

**Questions d'aide :**
- Y a-t-il une revue de code systématique ? → **NON**
- Existe-t-il un environnement de staging ? → **NON, "on teste en prod"**
- Comment sont partagés les secrets (clés API) ? → **Sur Slack !**
- Existe-t-il un plan de reprise d'activité (PRA) ? → **NON**

---

## Composant H – PERSONNEL

| # | Vulnérabilité | Menace associée | Justification |
|---|---------------|-----------------|---------------|
| H1 | Thomas = SPOF humain | Si Thomas absent = bloqué | Toute connaissance dans sa tête |
| H2 | Comptes anciens employés actifs | Accès non révoqués | Personne n'a nettoyé les permissions |
| H3 | Pas de formation sécurité | Erreurs/négligence | Devs ne connaissent pas les risques |

**Questions d'aide :**
- Le personnel est-il formé à la cybersécurité ? → **NON**
- Y a-t-il une personne unique qui détient des connaissances critiques ? → **OUI, Thomas**
- La direction est-elle impliquée dans la sécurité ? → **"C'est trop cher"**

---

# PARTIE 3 : IDENTIFICATION DES SPOF

> **SPOF** = Single Point of Failure = Point unique de défaillance
>
> **Question clé** : "Si cet élément tombe, que se passe-t-il ?"

## SPOF identifiés

| # | Type | Élément | Impact si défaillance | Solution proposée |
|---|------|---------|----------------------|-------------------|
| SPOF1 | Humain | Thomas (lead dev) | Projet bloqué, aucune décision | Documentation + formation équipe |
| SPOF2 | Matériel | MongoDB Atlas cluster | Perte accès à toutes les données | Réplication, failover, backups testées |
| SPOF3 | Matériel | Redis instance | Cache perdu, queries lentes | Redis Sentinel ou Redis Cluster |
| SPOF4 | Données | Sauvegardes non testées | Impossible de restaurer | Tester restauration régulièrement |
| SPOF5 | Logiciel | 147 packages npm outdated | 50+ CVE exploitables | npm audit + Dependabot + update régulier |

**Types de SPOF à rechercher :**
- 🖥️ **Matériel** : Serveur/service unique
- 💾 **Logiciel** : Dépendance critique unique
- 📊 **Données** : Sauvegarde unique ou non testée
- 👤 **Humain** : Personne unique indispensable
- 📋 **Procédure** : Processus unique sans alternative

---

# PARTIE 4 : ANALYSE DE RÉSILIENCE

## RTO et RPO actuels de DevSecure

| Indicateur | Valeur actuelle | Valeur recommandée | Écart |
|------------|-----------------|-------------------|-------|
| **RTO** (temps max d'interruption) | Illimité (inconnu) | < 4 heures | CRITIQUE |
| **RPO** (perte de données max) | 24 heures | < 1 heure | ÉLEVÉ |

> **RTO** = Recovery Time Objective (combien de temps peut être down ?)
> **RPO** = Recovery Point Objective (combien de données peut on perdre ?)

## Les 4 piliers de la résilience – État DevSecure

| Pilier | État | Justification |
|--------|------|---------------|
| **Anticiper** | ❌ | Pas d'audit, pas de planification |
| **Résister** | ❌ | SPOF partout, pas de redondance |
| **Absorber** | ❌ | Pas de plan B, sauvegardes non testées |
| **Se rétablir** | ❌ | Aucun processus de recovery |

---

# SYNTHÈSE

## Comptage par catégorie

| Catégorie | Nombre |
|-----------|--------|
| **Vulnérabilités CODE (OWASP)** | |
| A01 - Broken Access Control | 3 |
| A02 - Cryptographic Failures | 3 |
| A03 - Injection | 3 |
| A05 - Security Misconfiguration | 2 |
| A06 - Vulnerable Components | 1 |
| A07 - Auth Failures | 1 |
| A09 - Logging Failures | 1 |
| **Vulnérabilités INFRA (Laudon)** | |
| M - Matériel | 3 |
| L - Logiciel | 3 |
| D - Données | 3 |
| P - Procédures | 4 |
| H - Personnel | 3 |
| **SPOF identifiés** | 5 |
| **TOTAL VULNÉRABILITÉS** | **~35** |

---

## Top 5 des vulnérabilités les plus critiques

| Rang | Réf. | Vulnérabilité | OWASP | Laudon | Impact | Score EBIOS |
|------|------|---------------|-------|--------|--------|-------------|
| 1 | D1 | Mots de passe en clair | A02 | D | 2000 users compromis | 4×4=16 🔴 |
| 2 | SPOF2 | MongoDB SPOF | - | M | Tout s'arrête | 4×4=16 🔴 |
| 3 | L1 | npm outdated (50+ CVE) | A06 | L | RCE possible | 4×3=12 🔴 |
| 4 | H1 | Thomas SPOF humain | - | H | Projet bloqué | 3×4=12 🔴 |
| 5 | C5+C6 | Injections NoSQL | A03 | L | Accès données illicite | 3×4=12 🔴 |

---

# QUESTIONS DE RÉFLEXION

## 1. Chaîne d'attaque

Décrivez un scénario combinant plusieurs vulnérabilités :

**Exemple :**
1. Attaquant découvre injection NoSQL (C5/C6)
2. Accède à projet d'un autre client (IDOR, C6)
3. Télécharge fichier (trouve path traversal, C9)
4. Fichier S3 public, accède à données (C10)
5. Extrait mots de passe en clair (D1)
6. Utilise comptes pour accéder au dashboard

_________________________________________________________________

_________________________________________________________________

## 2. SPOF le plus critique

Quel SPOF aurait l'impact le plus grave et pourquoi ?

**Réponse attendue :**
Thomas (H1) car si absent → équipe bloquée + personne pour remédier aux incidents

_________________________________________________________________

## 3. Quick wins

Quelles sont les 3 corrections les plus simples à implémenter ?

1. **Patcher npm audit (L1)** – Commande, 1 heure
2. **Passer secrets en .env (C8/C1)** – Refactoring, 2-3 heures
3. **Hasher les mots de passe (D1)** – bcrypt, 1-2 heures

---


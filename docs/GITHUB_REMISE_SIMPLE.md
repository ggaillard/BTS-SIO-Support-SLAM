# 📦 GITHUB — REMISE DE TON TP

**Durée totale** : 30 minutes (avant, pendant, après la séance)

---

## 🎯 AVANT LA SÉANCE (15 min)

### 1️⃣ Créer un compte GitHub

1. Allez sur : https://github.com/signup
2. Remplissez :
   - Email : votre.email@etudiant.fr
   - Username : `prenom_nom_bts` (ex: marie_dupont_bts)
   - Password : minimum 12 caractères

3. Validez l'email reçu

✅ **Compte prêt !**

### 2️⃣ Comprendre les 4 concepts clés

| Concept | Explication | Exemple |
|---------|-------------|---------|
| **Dépôt** | Dossier en ligne pour stocker tes fichiers | Comme Google Drive |
| **Public** | Tout le monde peut voir | Enseignant peut accéder |
| **Markdown (.md)** | Format de texte simple avec formatage | `# Titre`, `**gras**` |
| **Commit** | Envoyer/sauvegarder tes fichiers | Click "Commit changes" |

---

## 🛠️ PENDANT LA SÉANCE (10 min à la fin)

### 3️⃣ Créer un dépôt

1. Connectez-vous à GitHub
2. Cliquez `+` (en haut à droite) → `New repository`
3. Remplissez :
   ```
   Name : TP_DevSecure_Cartographie
   Public : ✅ (cocher)
   ```
4. Cliquez `Create repository`

### 4️⃣ Uploader ta cartographie

1. Cliquez `Add file` → `Upload files`
2. Sélectionnez ton fichier Markdown (`.md`)
3. Cliquez `Commit changes`

### 5️⃣ Uploader les images (si tu en as)

1. Cliquez `Add file` → `Create new file`
2. Écrivez : `images/.gitkeep`
3. Commit
4. Uploadez vos images dans ce dossier

---

## 📧 APRÈS LA SÉANCE (5 min)

### 6️⃣ Envoyer l'URL à l'enseignant

```
https://github.com/votre_username/TP_DevSecure_Cartographie
```

---

## 📋 FORMAT DE TON FICHIER À REMETTRE

Ton fichier `.md` doit contenir :

```markdown
# Cartographie des risques — DevSecure

## 👥 Binôme
- Étudiant 1 : [Nom]
- Étudiant 2 : [Nom]

## 🔍 Vulnérabilités

| Vulnérabilité | Composant | OWASP | V | I | Risque | Niveau |
|---|---|---|---|---|---|---|
| Injection SQL | Software | A03 | 4 | 4 | 16 | 🔴 CRITIQUE |
| [Ajouter autres] | | | | | | |

## 💪 Points forts
- [À compléter]

## ⚠️ À améliorer
- [À compléter]

## 🎯 Solutions
- [À compléter]
```

---

## 📝 MARKDOWN — SYNTAXE RAPIDE

```markdown
# Titre principal
## Titre 2
### Titre 3

**gras** — *italique*

- Puce 1
- Puce 2

| Col1 | Col2 |
|------|------|
| A    | B    |

![Image](images/photo.png)
```

---

## 🆘 DÉPANNAGE

| Problème | Solution |
|----------|----------|
| Dépôt en Private | Settings → Change visibility → Public |
| Compte pas créé | Allez sur https://github.com/signup |
| Fichier .docx au lieu de .md | Convertissez avec Pandoc ou Google Docs |
| Images cassées | Vérifiez chemin : `images/nom.png` |
| GitHub ne charge pas | Videz le cache (Ctrl+Shift+Delete) |

---

## ✅ CHECKLIST REMISE

- [ ] Compte GitHub créé
- [ ] Dépôt nommé `TP_DevSecure_Cartographie`
- [ ] Dépôt en **Public**
- [ ] Fichier `.md` uploadé
- [ ] Images dans dossier `images/`
- [ ] URL envoyée à l'enseignant

---

**C'est tout ! Simple et efficace ! 🚀**

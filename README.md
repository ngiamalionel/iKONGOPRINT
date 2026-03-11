# iKONGO Print 🇨🇩

> La plateforme de conversion de fichiers PDF gratuite, conçue pour les utilisateurs congolais.

![iKONGO Print](https://img.shields.io/badge/iKONGO-Print-c0392b?style=for-the-badge)
![License](https://img.shields.io/badge/license-MIT-1e2d5e?style=for-the-badge)
![No Server](https://img.shields.io/badge/100%25-Browser%20Local-27ae60?style=for-the-badge)

## ✨ Fonctionnalités

| Outil | Description | Format sortie |
|-------|-------------|---------------|
| Word → PDF | Convertit `.docx` en PDF | `.pdf` |
| Excel → PDF | Convertit `.xlsx` en PDF | `.pdf` |
| PowerPoint → PDF | Convertit `.pptx` en PDF | `.pdf` |
| Image → PDF | JPG/PNG/WEBP vers PDF | `.pdf` |
| PDF → Word | Extrait le texte du PDF | `.docx` |
| PDF → Excel | Extrait les tableaux | `.xlsx` |
| PDF → Images | Une image par page | `.jpg` ou `.zip` |
| Fusionner PDF | Combine plusieurs PDF | `.pdf` |
| Diviser PDF | Une page = un fichier | `.pdf` ou `.zip` |
| Compresser PDF | Réduit la taille | `.pdf` |

## 🔒 Confidentialité

**Vos fichiers ne quittent jamais votre appareil.** Toute la conversion se fait localement dans votre navigateur grâce à :
- [jsPDF](https://github.com/parallax/jsPDF) — génération de PDF
- [pdf-lib](https://github.com/Hopding/pdf-lib) — manipulation de PDF
- [PDF.js](https://mozilla.github.io/pdf.js/) — lecture de PDF
- [Mammoth.js](https://github.com/mwilliamson/mammoth.js) — lecture de Word
- [SheetJS](https://sheetjs.com/) — lecture d'Excel

## 📬 Contact

- **Email:** ngiamalionel2@gmail.com
- **WhatsApp:** [+36 70 422 1887](https://wa.me/36704221887)

## 🚀 Déploiement sur GitHub Pages

1. Forkez ou clonez ce dépôt
2. Allez dans **Settings → Pages**
3. Choisissez **Branch: main**, dossier **/ (root)**
4. Votre site sera disponible à `https://votre-username.github.io/ikongo-print/`

## ⚙️ Configuration du formulaire de contact (EmailJS)

Pour que le formulaire de contact envoie des emails à `ngiamalionel2@gmail.com` :

1. Créez un compte sur [emailjs.com](https://www.emailjs.com/) (gratuit)
2. Créez un **Email Service** connecté à Gmail
3. Créez un **Email Template** avec les variables : `{{from_name}}`, `{{from_email}}`, `{{subject}}`, `{{message}}`
4. Copiez votre **Service ID**, **Template ID** et **Public Key**
5. Modifiez les 3 lignes dans `assets/app.js` :

```javascript
const EMAILJS_SERVICE_ID  = 'service_XXXXXXXX';  // votre Service ID
const EMAILJS_TEMPLATE_ID = 'template_XXXXXXXX'; // votre Template ID
const EMAILJS_PUBLIC_KEY  = 'XXXXXXXXXXXXXXXX';  // votre Public Key
```

> Sans configuration EmailJS, le formulaire ouvrira le client email natif (mailto:).

## 📁 Structure du Projet

```
ikongo-print/
├── index.html          # Page principale
├── assets/
│   ├── style.css       # Styles CSS
│   ├── app.js          # Logique JavaScript + conversion
│   └── favicon.svg     # Icône du site
├── README.md
└── LICENSE
```

## 📄 Licence

MIT © 2025 iKONGO Print

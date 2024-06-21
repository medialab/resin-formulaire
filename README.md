# resin-formulaire

Formulaire d'inscription pour [le site de Resin](https://github.com/medialab/resin-annuaire).


## Installation

```bash
npm install @medialab/resin-formulaire
```

## Utilisation

```javascript 
// mybundle.js

// Importer le web component resin-formulaire
import ResinFormulaire from '@medialab/resin-formulaire';

```

```html
<!-- mypage.html -->
<script src="mybundle.js"></script>

<resin-formulaire />
```

Par défaut, le formulaire affiché est le formulaire d'inscription.
Si les paramètres `uid` et `token` sont passés en querystring,
le formulaire affiché est le formulaire de modification de profil.


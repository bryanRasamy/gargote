## Schema relationnel:
1. Schema relationnel des 4 tables:
    - categorie (<u>id</u>, libelle)
    - plat (<u>id</u>, nom, prix, #id_categorie)
    - commande (<u>id</u>, date_commande, numero_table)
    - commande_ligne (<u>id</u>, #id_commande, #id_plat, quantite, prix_unitaire)

2. `commande_ligne` contient-elle une colonne `prix_unitaire`, alors que le prix est déjà dans `plat` car le prix du plat peut changer du jour au lendemain, ce qui veut dire que les prix des commandes dans le passé peuvent se changer et donne une fauss somme.

Exemple:
prix du ravitoto = 9 000 Ar (nouveau prix changé aujourd'hui)

prix du ravitoto dans commande_ligne (commande d'hier)= 8 000 Ar 

3. Quand on supprime la commande, les lignes de commande correspondant se supprime aussi. l'instruction responsable est ` ON DELETE CASCADE`

4. - Quantité peut etre negatif
   - Prix peut etre negatif
   - Prix unitaire peut etre negatif

## API
a. 
- Code: 200
- reponse: [
    {
        "id": 7,
        "nom": "Mi sao",
        "prix": 6500,
        "id_categorie": 2,
        "description": null,
        "categorie": "Soupe"
    },
    {
        "id": 6,
        "nom": "Soupe chinoise",
        "prix": 6000,
        "id_categorie": 2,
        "description": null,
        "categorie": "Soupe"
    }
]

b. 
- Code: 201
- reponse: {
    "id": 11,
    "nom": "Mofo gasy",
    "prix": 300,
    "id_categorie": 1,
    "description": null
}

c. 
- Code: 422
- reponse: {
    "message": "Données invalides.",
    "errors": {
        "nom": "Le nom du plat est obligatoire.",
        "prix": "Le prix doit être un nombre entier (en Ariary).",
        "id_categorie": "Choisissez une catégorie."
    }
}

d. 
- Code: 409
- reponse: {
    "message": "Ce plat figure déjà dans des commandes, il ne peut pas être supprimé.",
    "errors": {}
}

e. 
- Code: 404
- reponse: {
    "message": "Commande 999 introuvable.",
    "errors": {}
}


5. Pour la requête b, on obtient un code 201. Pas 200 car qu'on ajoute avec POST, on obtient un code 201.

6. Pour la requête d, PostgreSQL bloque la suppression.
``` php
    try {
        $this->plats->delete($id);
    } catch (\Throwable $e) {
        // Le plat apparaît dans des commandes : la clé étrangère bloque la suppression.
        return $this->erreur('Ce plat figure déjà dans des commandes, il ne peut pas être supprimé.', 409);
    }
```

7. 
```
    curl -X POST http://localhost:8080/api/commandes \
     -H "Content-Type: application/json" \
     -d '{"numero_table":6, "lignes": [ { "id_plat": 4, "quantite": 2 }, { "id_plat": 10, "quantite": 1 } ]}'
```

## Requete
| # | Étape | Fichier | Fonction / ligne |
|---|---|---|---|
| 1 | Le formulaire est soumis | `components/PlatForm.vue` | `enregistrer()` |
| 2 | La requête HTTP est construite | `api.js` | `requete()` |
| 3 | Vite transmet `/api/...` au port 8080 | `vite.config.js` | `defineConfig()` |
| 4 | CodeIgniter choisit le contrôleur | `Config/Routes.php` | `13 à 28 eme ligne`|
| 5 | Le contrôleur lit le JSON | `Controllers/Api/ApiController.php` | `donneesJson()` |
| 6 | Les règles de validation sont vérifiées | `Models/PlatModel.php` | `$validationRules` |
| 7 | La ligne est insérée | `Controllers/Api/Plats.php` | `Ligne 42` |
| 8 | Le formulaire prévient l’écran parent | `components/PlatForm.vue` | `emit(…)` |
| 9 | L’écran recharge la liste | `views/CarteView.vue` | `Ligne 26`|

8. La création d’une commande est dans `CommandeService.php` car cela permet d'utiliser beaucoup de fois le code metier sans avoir a le recreer.

## Notions de Vue
| Notion | À quoi ça sert | Votre exemple |
|---|---|---|
| `ref()` | une variable dont le changement met l’écran à jour | `views/CarteView.vue Ligne 7` Plats vide au debut|
| `computed()` | une valeur calculée à partir d’autres, recalculée automatiquement | `views/CommandeView.vue Ligne 13` liste des plats trier par categorie|
| `v-for` | répéter un élément pour chaque élément d’une liste | `views/CommandeView.vue Ligne 13` Pour chaque categorie dans groupes|
| `v-if` / `v-else` | afficher un élément seulement si une condition est vraie | `views/CommandeView.vue Ligne 75` si erreur existe, on affiche |
| `v-model` | lier un champ de formulaire à une variable, dans les deux sens | `views/CommandeView.vue Ligne 74` On envoie le numero de table |
| `defineProps` | recevoir des données du composant parent | `components/PlatForm.vue Ligne 5` recuperer le plat venant du parent|
| `defineEmits` / `emit` | envoyer un événement au composant parent | `components/PlatForm.vue Ligne 41` on renvoie le resultat au parent|
| `onMounted` | exécuter du code quand le composant apparaît | `views/CarteView.vue Ligne 40` charger les plats de la carte|\

9. `Panier.vue` ne modifie jamais la quantité lui-même : il émet `plus` et `moins` car c'est un composants reutilisable, et les composants recoit juste les donnees, pas les modifier. C'est `CommandeView` qui le fait.

10. Le `wacth` sert a modifier les valeurs d'un composant sans le recreer. Sans lui, le composant ne serait jamais a jour.

11. La catégorie apparaît dans la liste déroulante du formulaire sans que vous ayez écrit de code. La raison c'est que le front appelle juste la requete qui permet d'avoir toute la liste venant du backend.
- Route: `/categories`
- composant `PlatForm.vue`
# Tools #

> Use only imports, variables, and mixins (and only for vender-prefixed features) from CSS preprocessors.

Garder le CSS lisible et clair, utiliser du vanilla CSS autant que possible.  
Imbrication (nesting) seulement pour les states et pseudo-classes comme `:hover`.  
Pas de fonctionnalités avancées de Less comme les boucles.

# Components #

> Use the `.component-descendant-descendant` pattern for components.

Éviter les cascades de règles, améliorer la "maintenabilité".  
Utiliser des namespaces et des "composants" (components).

Aucun "sélecteur descendant" :  

    .global-header {...}

    .global-header .logo {...}

    .global-header .logo img {...}

Utilisation du namespacing :  

    .global-header {...}

        .global-header-logo {...}

            .global-header-logo-image {...}

Garder une spécificité la plus basse possible. Permet d'éviter le style inline, les `!important`, etc.

**Tous les sélecteurs doivent être des classes**. Aucune raison d'utiliser des ID ou des sélecteurs d'éléments.  
**Pas d'underscores ni de camelCase**. Tout en minuscule, séparé par des tirets.

Indenter les descendants par rapport aux parents. Les états et pseudo-classes sur la même colonne.  
(ça contredit la règle comme quoi ils utilisent le nesting pour les states, moi j'utilise le nesting en tout cas !)

## Modifiers ##

> Use the `.component-descendant.mod-modifier` pattern for modifier classes.

Lorsqu'on a besoin de styler un élément dans un style particulier.  
Exemple: un message en mode "erreur", "succès" ou "warning" ; la base reste la même (un bloc "message") mais le style diffère légèrement.

Nommage `.component-descendant-modifier` pas cohérent car serait interprété comme un descendant de `.component-descendant` lui-même.  
Pour mieux différencier, on utilise `.mod-modifier`.

    <div class="alert-message mod-succes">Message de succès</div>
    <div class="alert-message mod-error">Message d'erreur</div>

    .alert-message {...}
    .alert-message.mod-success {...}
    .alert-message.mod-error {...}

**Ne JAMAIS avoir une classe modifier seule** (qui n'est pas rattachée à un composant).  
Exemple: la classe `.mod-success` ne doit pas exister seule, elle doit forcément être créée et appliquée sur un autre composant (ici `.alert-message`).

Cela permet d'utiliser la classe `.mod-success`sur d'autres composants, sans écraser leurs règles (par exemple, sur un bouton `.button.mod-success`).

Si besoin d'écraser des règles sur des descendants du composant "modifié", utiliser exceptionnelement le nesting :

    .global-header-nav-item.mod-sign-up {
        background: hsl(120, 70%, 40%);
        color: #fff;

        .global-header-nav-item-text {
            font-weight: bold;
        }
    }

Les "modifiers" sont placés en bas du fichier, après le composant original.

    // À la sauce Bastien
    .alert-message {
        ...

        &.mod-success {...}
        &.mod-error {...}
    }

J'ai jamais eu de "gros" modifier à faire, du coup je préfère les placer au même endroit que mon composant, je m'y retrouve plus facilement (mais c'est sûrement subjectif).

## State ##

> Use the `.component-descendant.is-state` pattern for state. Manipulate `.is-` classes in JavaScript (but not presentation classes).

Tous les états gérés en JS, comme par exemple un état "chargement" (`.is-loading`) lié à de l'Ajax ou un bouton "désactivé" (`.is-disabled`).

Facilite la décomposition entre présentation et état de l'application, on peut modifier les "state classes" sans se soucier des "presentation classes" (développeur VS graphiste).

En BEM, pour le logo de Trello qui charge, côté développeur on doit gérer ça: `.global-header-logo-image--is-loading`.  
Avec les guidelines Trello, le développeur ne connaît que `.is-loading`. Le reste est géré par le graphiste.

Cela permet aussi d'utiliser plusieurs fois le même état (`.is-loading` sur un bouton, une image, etc.), sans craindre de l'héritage/override tout pourri.  
Chaque composant définit sa propre implémentation de l'état `.is-loading`. Comme pour les "modifiers", ne jamais créer de "state" seul, sans composant ciblé.  
Si vous créez un `.is-hidden { display: none; }`, vous brûlerez en enfer !

De la même manière que les modifiers, on n'indente pas les states, ils sont au même niveau que le composant.  
On les place à la fin du fichier, après le composant et les modifiers.

Encore une fois, à ma sauce, je les imbrique souvent dans le composant lui-même.

## Media Queries ##

> Use media query variables in your component.

On éviter les media-queries globales, on les utilise dans chaque composant.  
Chaque composant est responsable de son affichage dans les différentes media-queries.  
Ca évite d'avoir du code qui traine si on supprime un composant par exemple.

Pour se faciliter la vie, on créer des variables qui contiennent la définition de nos media-queries :

    @highdensity:  ~"only screen and (-webkit-min-device-pixel-ratio: 1.5)",
                   ~"only screen and (min--moz-device-pixel-ratio: 1.5)",
                   ~"only screen and (-o-min-device-pixel-ratio: 3/2)",
                   ~"only screen and (min-device-pixel-ratio: 1.5)";

    @small:        ~"only screen and (max-width: 750px)";
    @medium:       ~"only screen and (min-width: 751px) and (max-width: 900px)";
    @large:        ~"only screen and (min-width: 901px) and (max-width: 1280px)";
    @extra-large:  ~"only screen and (min-width: 1281px)";

    @print:        ~"print";

Utilisation dans un composant :

    // Input
    @media @large { 
        .component-nav {...}
    }
    @media @small, @medium { 
        .component-nav {...}
    }

Réutilisabilité des breakpoints partout dans nos CSS. Plus facile pour s'y retrouver et plus beaucoup plus maintenable.  
Les media-queries sont facilement "compressables" à coup de package NPM si le bsoin s'en fait sentir.  
À noter la présence de l'attibut "print", là encore chaque composant gère ses propres règles pour le print.

Toujours pareil, les règles des media-queries sont mises à la fin du fichier de chaque composant.

## Keeping It Encapsulated ##

On peut facilement se retrouver avec un composant imbriqué dans un autre : `.button` dans un `.member-list`.  
On a besoin de changer sa taille et sa position pour le placer correctement dans la liste : là ça devient casse-tête, comment qu'on fait ?!  
Un composant ne doit **RIEN** savoir des autres composants.

L'idée, c'est de réutiliser si possible la modification de la taille du bouton (`.button.mod-small`) et gérer la position avec un descendant de `.member-list`.

    <div class="member-list">
        <div class="member-list-item">
            <p class="member-list-item-name">Gumby</p>
            <div class="member-list-item-action">
                <a href="#" class="button mod-small">Add</a>
            </div>
        </div>
    </div>

Ne surtout pas faire ça :

    <div class="member-list">
        <div class="member-list-item">
            <p class="member-list-item-name">Pat</p>
            <a href="#" class="member-list-item-button button">Add</a>
        </div>
    </div>

Dans ce cas, `..member-list-item-button` surcharge le composant `.button`, et c'est pas cool.  

On se retrouve vite avec beaucoup de composants.  
Si un des fichiers vous semble gros, ne pas hésiter à découper le composant en de multiples composants plus légers (ou quand on a beaucoup de modifiers et/ou descendants).

# JavaScript #

> Separate style and behavior concerns by using `.js-` prefixed classes for behavior.

    <div class="content-nav">
        <a href="#" class="content-nav-button js-open-content-menu">
            Menu
        </a>
    </div>

Utiliser des noms clairs et parlants, comme `.js-open-content-menu` au lieu de `.js-menu`.

**Les classes `.js-` ne doivent jamais apparaître dans les fichiers CSS**.  
Inversement, ne jamais toucher à une classe CSS d'un composant côté JS.  
Exception faite sur les states, puisqu'on utilise `.is-state` en JS et `.component.is-state` dans les feuilles CSS.

# Mixins #

# Utilities #

# File Structure #

# Style #

# Miscellany #

## Performance ##
---
title: Sviluppo best practices per l'utilizzo di bem, sass e assets
---
# Sviluppo frontend

Definire convenzioni ed un lessico condiviso per operazioni e pattern d'uso comune nello sviluppo frontend è fondamentale per facilitare uno scambio di idee più fluido tra sviluppatori, semplificando l'introduzione di nuovi sviluppatori e/o passaggi di consegne.

La struttura consigliata per un nuovo progetto Sass in partenza è la seguente:

```filesystem
    app/assets/stylesheets/
    ├── blocks
    │   ├── _block-name.css.sass
    │   └── ...
    ├── formatting
    │   ├── _formatting-category.css.sass
    │   └── ...
    ├── mixins
    │   ├── _mixin-name.css.sass
    │   └── ...
    ├── variables
    │   ├── _variables-category.css.sass
    │   └── ...
    ├── _base.css.sass
    ├── _shame.css.sass
    └── application.css.sass
```

## Rootfile

Il Rootfile, `application.css.sass`, si occupa esclusivamente di includere ordinatamente i partial:

```sass
    @import "variables/**/*"
    @import "mixins/**/*"

    @import "base"
    @import "formatting/**/*"
    @import "blocks/**/*"
    @import "shame"
```

Per facilitare il mantenimento del file, nel caso di progetto Rails 3.1+, è possibile sfruttare [il file globbing nella direttiva `@import`](https://github.com/rails/sass-rails#glob-imports).

## Shamefile

Inserisci quick-fix, hack o tecniche "discutibili" solo all'interno del file `_shame.css.sass` con l'obiettivo di:

* Renderli più semplici da identificare, isolare e risolvere in un secondo momento
* Mantenere il codebase "principale" pulito
* Rendere esplicito lo stato di salute dei fogli di stile

Documenta gli hack presenti nel file, specificando:

* A quale parte del codebase appartiene
* Perchè è stato necessario inserirlo
* Come si potrebbe fixare, se ci fosse più tempo

## Basefile

Il partial `_base.css.sass` contiene gli stili di default applicati ad ogni pagina. Tipicamente contiene una direttiva `@import` che include un [CSS reset](http://necolas.github.io/normalize.css/) e soli _tag selectors_.

E' importante fare in modo di aggiornare nel tempo il Basefile, con l'obiettivo di ridurre al minimo l'override in cascata all'interno di blocchi e stili di formattazione.

## Selettori di formattazione

Sono `class selectors` utili ad evitare la creazione di blocchi specifici  per pagine che — magari dopo il riutilizzo di blocchi già esistenti nel codebase — richiederebbero ancora minime modifiche alla formattazione del testo (allineamento, padding, dimensione font, etc).

Ogni selettore di formattazione contiene tipicamente una sola regola. Per facilitare l'identificazione di selettori di formattazione all'interno dell'HTML, i nomi delle classi vengono prefissate con `f-`:

```sass
  .f-right-aligned
    text-align: right
```

Sebbene si riconosce a queste regole un utilizzo pratico indiscutibile, è fondamentale mantenerle in numero ridotto, per evitare _classitis_ nell'HTML.

A fini esemplificativi, la tecnica _[double-stranded heading hierarchy](http://csswizardry.com/2012/02/pragmatic-practical-font-sizing-in-css/)_ può essere implementata con selettori di formattazione:

```sass
  .f-size-alpha
    font-size: 30px

  .f-size-beta
    font-size: 26px

  .f-size-gamma
    font-size: 23px

  ...
```
## BEM (Block-Element-Modifier)

Il BEM è una [convenzione di naming codificata da Yandex](http://bem.info/) che ha come obiettivo principale quello di garantire una maggiore trasparenza e chiarezza di intenti.

Ad esclusione delle poche regole specificate nei fogli di stile già descritti, la stragrande maggioranza del codebase segue una notazione BEM, e risiede all'interno della directory `blocks`.

La convenzione BEM è la seguente:

```sass
  .block
  .block__element
  .block--modifier
```

Dove:

* `.block` rappresenta un nuovo componente/astrazione. Ogni blocco viene specificato in un partial Sass omonimo (es. `blocks/_block.css.sass`).
* `.block__element` rappresenta un elemento figlio di `.block`, che aiuta a definirlo nel suo complesso.
* `.block--modifier` rappresenta una differente versione di `.block` (in termini OOP, una sottoclasse).

Dato che i nomi delle classi dei sotto-elementi di un blocco sono prefissati univocamente dal nome del blocco stesso:

* in qualsiasi momento è possibile analizzare l'HTML e capire a quale blocco appartiene un dato elemento;
* si elimina completamente name-clashing involontario tra sotto-elementi di componenti differenti;
* a livello Sass/CSS non è necessario utilizzare _descendant selectors_ o _parent selectors_ (es. `.block .element` o `.block > .element`), ottenendo quindi come side-effect positivo fogli di stile più performanti;

Ogni blocco ha il compito di specificare il layout dei propri sotto elementi, ed è possibile inserire blocchi all'interno di altri blocchi (nesting).

### Stati

A differenza dei *block-modifiers*, gli stati rappresentano cambiamenti di stato dinamici/programmatici all'interno del layout della pagina (es. Javascript).

Per facilitarne l'identificazione a livello HTML, i nomi degli stati vengono prefissati con `is-`:

```sass
  .is-active
  .is-collapsed
  .is-hidden
```
E' possibile introdurre stati sia a livello di blocco, che di sotto-elemento di blocco.

### Layout-blocks

Per favorire il riutilizzo di blocchi già esistenti nel codebase, particolare attenzione dev'essere posta nel **non specificare** dimensioni esplicite, margini o posizionamenti relativi a livello root del blocco: ogni blocco deve simulare il comportamento di default di un tag `div`, dunque essere in `position: static` ed espandersi a coprire il 100% della larghezza dell'elemento padre, senza margini.

Per ovviare alla necessità di spaziare tra di loro i differenti blocchi presenti all'interno di una pagina, è possibile ricorrere al nesting di blocchi; un blocco il cui scopo è unicamente quello di posizionare spazialmente i sotto-blocchi che contiene prende il nome di _layout block_. Esempi tipici di layout blocks si hanno in blocchi che organizzano i propri sotto-elementi in griglie o in colonne main-content/sidebar.

Per facilitare l'identificazione di layout blocks all'interno dell'HTML, i nomi vengono prefissati con `l-`:

```sass
  .l-grid
  .l-two-cols
  .l-spacing
```

### Ma il BEM esteticamente è brutto!

Ad oggi sfortunatamente il CSS non ci consente molta scelta in termini di caratteri non alfanumerici da poter utilizzare all'interno delle classi come "delimitatori".
La ragione dei doppi trattini ed underscores sta nella possibilità di poter continuare ad utilizzare lo _spinal-case_ classico per dare un nome a blocchi, elementi e modificatori:

```sass
  .site-search
  .site-search__sub-element
```

In definitiva sì, la notazione BEM è più verbosa e al primo sguardo può sembrare "strana", ma i vantaggi in termini di chiarezza ed espressività del codice risultante superano di molto possibili critiche sull'aspetto estetico.

### Quando termina un blocco e comincia un nuovo sotto-blocco?

La scelta è soggettiva e dipendente dal contesto. L'obiettivo è il maggior riutilizzo possibile di codice e regole già presenti nel codebase, evitando over-engineering e flessibilità non necessaria.

Un flusso di lavoro ottimale è separare un sotto-elemento da un blocco, trasformandolo in blocco a se' stante, solo quando un nuovo caso concreto lo rende effettivamente necessario.

### Quando utilizzare *child+tag selectors* all'interno di blocchi

Supponiamo di voler realizzare un *layout block* che permetta di rendere floattanti i suoi sotto-blocchi:

```html
    <ul class="l-floating">
      <li class="l-floating__item">...</li>
      <li class="l-floating__item">...</li>
    <ul>
``

La convenzione BEM ci costringe a dover aggiungere una classe esplicita (`.l-floating__item`) ad ogni sotto-elemento. Confrontiamo questa scelta con un possibile selettore non-BEM:

```sass
  .l-floating > li
```

Sebbene meno verboso a livello di HTML che richiede, il secondo selettore non permette di intuire istantaneamente per ogni `<li>` il blocco che ha la responsabilità di stilizzarlo:

```html
  <section class="another-block">
    ...
    <ul class="l-floating">
      <li>...</li>
      <li>...</li>
    <ul>
    ...
  </section>
```

Rende inoltre meno riutilizzabile lo stile, non consentendone la sua applicazione ad esempio in caso di cambio di tag dettato da motivi non prettamente estetici (semantica, SEO, accessibilità, etc):

```html
  <div class="l-floating">
    <div>...</div>
    <div>...</div>
  <ul>
```

Presupposto del BEM è un *separation of concerns* netto tra estetica e semantica in grado di garantire il massimo riutilizzo di blocchi già presenti, senza richiedere modifiche a codice già presente nel codebase.

Si sconsiglia dunque l'uso di *child+tag selectors* all'interno di blocchi quando non si può escludere un cambiamento di tag nel futuro (es. heading tags).

E' vietato invece l'utilizzo di *descendant selectors*, in quanto troppo generici e suscettibili a *carpet-bombing*.

### Block modifiers ed @extend

In alternativa allo stile BEM classico di applicazione di *block-modifiers*:

```html
  <div class="block block--modifier"></div>
```
E' possibile sfruttare la direttiva Sass `@extend`:

```sass
  .block--modifier
    @extend .block
    ...
```

Riducendo il *classitis* HTML:

```html
    <div class="block--modifier">   </div>
```

## Best practices CSS

* Definisci tutte le classi CSS in `spinal-case`. [→][bp-spinal-case]
* Definisci i colori con sintassi esadecimale (`#fff`) ed `rbga()`
* Limita l'utilizzo del modificatore `!important` a selettori di stato
* Non utilizzare *ID selectors* (`#header`)
* Non definire sotto-elementi di modulo con *selector-depth* superiore a 3.
* Evita un *depth of applicability* superiore a 3. [→][bp-depth-applicability]
* Sfrutta i `rem` con fallback a `px` per il font-sizing e il vertical-spacing
* Favorisci il riutilizzo producendo una *pattern library* HTML che mostri i diversi moduli e layout esistenti, ed un loro utilizzo pratico. [→][bp-pattern-library]

[bp-smacss]: http://smacss.com/book/categorizing
[bp-depth-applicability]: http://smacss.com/book/applicability
[bp-spinal-case]: http://en.wikipedia.org/wiki/Letter_case#Computers
[bp-pattern-library]: http://ux.mailchimp.com/patterns

## Best practices Sass

* Usa Sass, non Scss
* Prediligi Bourbon a Compass
* Definisci mixin e variabili in `spinal-case`. [→][bp-spinal-case]
* Limita il numero di possibili varianti di colore, font-family, font-size, spaziatura verticale tra i blocchi e breakpoint all'interno del sito, parametrizzandole all'interno di appositi file di variabili (`_colors.css.sass`, `_font-families.css.sass`, etc.)
* Produci dei mixin che rinforzino le convenzioni in uso all'interno del codebase (es. per l'utilizzo dei `rem` o per applicare una tra le possibili varianti di font-size)
* Prefissa i nomi di variabile dei colori con `c-` e i breakpoint con `b-`
* Definisci i colori con nomi semantici (`$primary-color` invece che `$red-color`)
* Quando il numero di blocchi presenti all'interno del codebase aumenta, procedi splittando i blocchi in namespaces, spostandoli in sotto-directory (es. `blocks/blog`, `blocks/homepage`, etc.)
* Non definire mixin senza parametri (sfrutta la direttiva `@extend`)
* Limita il numero di sotto-elementi per ogni blocco, per facilitare il riutilizzo autonomo di parti del blocco.
* Specifica direttive `@media` al termine della definizione del blocco, ed evita override dei selettori: specifica fuori dalle direttive `@media` solo le regole condivise a tutti i breakpoint

[bp-classitis]: http://www.steveworkman.com/html5-2/standards/2009/classitis-the-new-css-disease/


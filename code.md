# Coding best practices

Una nota sul linguaggio utilizzato:

* "Non fare X" significa che **non esiste** motivo valido per violare la regola.
* "Evita X" significa non farlo **a meno di un valido motivo**.
* "Prediligi X ad Y" descrive una pratica sconsigliata, ed una alternativa da favorire **a meno di un valido motivo**.

## Linee guida ad alto livello

* Sii coerente.
* Non riscrivere codice già esistente per seguire questa guida.
* Non violare le linee guida senza un valido motivo. Se riesci a convincere un altro membro del tuo team, allora il motivo può considerarsi valido.

## Generali

* Non duplicare funzionalità già presenti in librerie di sistema.
* Non fallire silenziosamente *ingoiando* eccezioni. [→][bp-fail-loudly]
* Non scrivere codice cercando di predire future necessità. [→][bp-yagni]
* Evita di usare eccezioni per gestire control flow ordinario. [→][bp-exceptional-exceptions]
* Mantieni il codice semplice, chiaro ed esplicito. [→][bp-simple-code]
* Evita di ottimizzare il codice. Non farlo mai senza profiling. [→][bp-profile]

[bp-fail-loudly]: http://www.codinghorror.com/blog/2007/08/whats-worse-than-crashing.html
[bp-yagni]: http://c2.com/cgi/wiki?DoTheSimplestThingThatCouldPossiblyWork
[bp-exceptional-exceptions]: http://pragmatictips.com/34
[bp-simple-code]: http://code.mumak.net/2012/02/simple-made-easy.html
[bp-profile]: http://c2.com/cgi/wiki?ProfileBeforeOptimizing

## Principi di object-oriented design

* Evita variabili globali e singletons. [→][bp-global-variables]
* Evita un numero di parametri superiore a 4. [→][bp-parameter-list]
* Limita il numero di collaboratori di un oggetto (le entità da cui dipende l'oggetto).
* Limita il numero di dipendenze di un oggetto (le entitità che dipendono dall'oggetto).
* Prediligi la composizione rispetto all'ereditarietà. [→][bp-composition]
* Prediligi metodi brevi. Ottimo rimanere tra 1 e 5 righe. [→][bp-long-method]
* Prediligi piccoli oggetti, con una singola responsabilità ben definita. Quando un oggetto supera le 100 righe, è probabile stia facendo troppe cose. [→][bp-one-responsibility]
* *Tell, don't ask*. [→][bp-tell-ask]
* Evita metodi che uniscono responsabilità *query* e *command*. [→][bp-cqrs]

[bp-global-variables]: http://stackoverflow.com/questions/484635/are-global-variables-bad
[bp-parameter-list]: http://sourcemaking.com/refactoring/long-parameter-list
[bp-composition]: http://stackoverflow.com/questions/49002/prefer-composition-over-inheritance
[bp-long-method]: http://c2.com/cgi/wiki?LongMethodSmell
[bp-one-responsibility]: http://c2.com/cgi/wiki?OneResponsibilityRule
[bp-tell-ask]: http://c2.com/cgi/wiki?TellDontAsk
[bp-cqrs]: http://en.wikipedia.org/wiki/Command%E2%80%93query_separation

## Ruby

* Evita parametri opzionali. Il metodo forse sta facendo più del dovuto?
* Evita inutile metaprogramming.
* Evita il monkey-patching.
* Prediligi la creazione di classi all'inclusione di moduli per condividere funzionalità tra più oggetti.
* Prediligi lo scope `private` a `protected`. Usa `protected` solo per i metodi di comparazione come `def ==(other)`, `def <(other)`, e `def >(other)`.
* Effettua `rescue` di `StandardError`, non `Exception`. [→][bp-standard-error]

[bp-standard-error]: http://robots.thoughtbot.com/rescue-standarderror-not-exception/

## Rails

* Evita di bypassare le validazioni con metodi quali `save(validate: false)` o `update_attribute`. [→][bp-skip-validations]
* Non modificare una migrazione dopo che è stata mergiata su master se il cambiamento voluto può essere introdotto con una seconda migrazione.
* Non richiamare le classi dei modelli `ActiveRecord` all'interno dalle viste.
* Non usare variabili d'istanza nei partials.
* Non usare SQL o frammenti SQL (`where('inviter_id IS NOT NULL')`) al di fuori dei modelli.
* Valida la presenza dell'associazione `belongs_to` (`user`), non della foreign key (`user_id`).
* Aggiungi un indice DB per ogni foreign key. [→][bp-index]
* Imposta valori di default, validazioni di presenza o unicità nei modelli anche a livello di migrazione DB.
* Mantieni `db/schema.rb` sotto version control.
* Non fare riferimento a modelli `ActiveRecord` a livello di migrazione.
* Nelle viste mailer usa named routes con suffissi `_url`. In tutti gli altri casi, usa il suffisso `_path`. [→][bp-redirects]
* Evita di instanziare più di un oggetto per azione nei controller.
* Sfrutta solo una variabile di istanza per vista.
* Assicurati che le migrazioni down funzionino. [→][bp-down-migrations]
* Evita l'introduzione di nuovi metodi helper. Prediligi un'approccio ad oggetti tramite l'uso di presenters. [→][bp-presenters]
* A livello di model `ActiveRecord` specifica unicamente validazioni, relazioni e scopes/query.
* Non specificare a livello model `ActiveRecord` validazioni che necessitino il lookup di relazioni.

[bp-skip-validations]: https://github.com/garybernhardt/do_not_want
[bp-redirects]: http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.30
[bp-down-migrations]: http://stefanoverna.com/blog/2013/08/test-down-migrations-tip.html
[bp-presenters]: http://nicksda.apotomo.de/2011/10/rails-misapprehensions-helpers-are-shit/
[bp-index]: https://tomafro.net/2009/08/using-indexes-in-rails-index-your-associations


## Testing

* Mantieni un code coverage superiore al 90%.
* Evita test ridondanti (uno e un solo test per funzionalità).
* Evita `its`, `let!`, `specify` e `subject` in RSpec. Prediligi approcci espliciti e coerenti.
* Specifica una sola expectation per test.
* Separa le fasi di *setup* ed *exercise* dalla fase *verify* del test tramite blocchi `before`.
* Evita `.any_instance`, sfrutta il *dependency injection*. [→][bp-dependency-injection]
* Evita l'utilizzo di variabili di istanza.
* Disabilita richieste HTTP verso servizi esterni tramite [WebMock] o [FakeWeb].

[bp-dependency-injection]: http://kresimirbojcic.com/2011/11/19/dependency-injection-in-ruby.html
[WebMock]: https://github.com/bblimke/webmock
[FakeWeb]: https://github.com/chrisk/fakeweb

## Unit testing

* Non testare i metodi privati (non stubbare il SUT). [→][bp-stub-sut]
* Testa metodi di tipo query con expectations sul valore di ritorno.
* Testa metodi di tipo command con expectations su diretti cambiamenti
  di stato provocati sull'interfaccia pubblica dell'oggetto.
* Non testare chiamate di tipo query verso oggetti collaboratori.
* Testa chiamate di tipo command verso oggetti collaboratori tramite stubs e spies (non mocks). [→][bp-spies]
* Mantieni gli stub in sync con l'API che vanno a simulare. Nel farlo prediligi approcci unit rispetto a test di integrazione. [→][bp-bogus]

[bp-stub-sut]: http://robots.thoughtbot.com/don-t-stub-the-system-under-test/
[bp-spies]: http://robots.thoughtbot.com/post/159805295/spy-vs-spy
[bp-call-original]: https://www.relishapp.com/rspec/rspec-mocks/v/2-12/docs/message-expectations/calling-the-original-method
[bp-bogus]: https://www.relishaphttps://www.relishapp.com/bogus/bogus/v/0-1-4/docs/p.com/bogus/bogus/v/0-1-4/docs/

## Gemfile

* Dichiara la versione ruby da utilizzare per un progetto. [→][bp-ruby]
* Specifica la versione con `~>` per le gemme che seguono il semantic versioning (`rspec`, `factory_girl`, `capybara`). [→][bp-versions]
* Specifica la versione esatta per gemme fragili (`rails`).
* Non specificare una versione per gemme stabili e sicure (`pg`, `thin`).

[bp-ruby]: http://bundler.io/v1.3/gemfile_ruby.html
[bp-versions]: http://robots.thoughtbot.com/post/35717411108/a-healthy-bundle

## Background Jobs

* Non persistere oggetti `ActiveRecord` serializzati nel job. Passa gli ID ed esegui la `.find` all'interno del metodo `.perform`.

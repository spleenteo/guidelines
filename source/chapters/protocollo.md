---
title: come iniziare un nuovo lavoro
---
# Protocollo

## Creazione di un nuovo progetto Rails

Installa Suspenders.

    gem install suspenders

Crea l'app

    suspenders new-project

## Set up di un progetto Rails

Ottieni il codice:

    git clone git@github.com:organization/project.git

Prepara le dipendenze dell'app.

    cd project
    ./bin/setup

Usa il plugin [Heroku config][p-heroku-config] per copiare le variabili d'environment:

    heroku config:pull --remote staging

Elimina le righe in `.env` superflue (es. `S3_SECRET`)

Utilizza [Foreman][p-foreman] per lanciare l'applicazione:

    foreman start

[p-heroku-config]: https://github.com/ddollar/heroku-config
[p-foreman]: https://github.com/ddollar/foreman

## Mantenimento di un progetto Rails

* Evita di includere in version control file specifici della propria macchina o della propria modalità di sviluppo.
* Non includere in version control file contenenti segreti o chiavi private.

## Lavora su una nuova feature

Crea una nuovo branch locale per la feature, forkando da master, e aggiungendo le tue iniziali come prefisso del branch.

    git checkout master
    git pull
    git checkout -b <branch-name>

Effettua spesso rebase dal master per includere eventuali modifiche provenienti da remoto.

    git fetch origin
    git rebase origin/master

Risolvi eventuali conflitti, e prosegui col rebase.

    # edit conflicted files on editor
    git add -all
    git rebase --continue

Quando una feature è completa ed i test passano, effettua un commit.

    rake
    git add --all
    git status
    git commit --verbose

Scrivi un [buon messaggio di commit][p-commit-message].

    Present-tense summary under 50 characters

    * More information about commit (under 72 characters).
    * More information about commit (under 72 characters).

    http://project.management-system.com/ticket/123

Pusha il tuo branch.

    git push origin <branch-name>

Procedi con l'effettuare una pull request. Richiedi un code review sulla stanza di development su [Hipchat][p-hipchat].

[p-commit-message]: http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html
[p-hipchat]: https://www.hipchat.com

## Effettua un code review

Un membro del team diverso dall'autore della feature effettua la review del codice, seguendo le linee guida per evitare incomprensioni.

Il reviewer quando possibile commenta e chiede chiarimenti direttamente all'interno dell'interfaccia web di Gitlab/Github/Bitbucket.

Il reviewer può effettuare piccoli cambiamenti al codice in prima persona, effettuando un checkout del branch.

    git checkout <branch-name>
    rake db:migrate db:test:prepare
    rake
    git diff staging/master..HEAD

Una volta testata la modifica su browser e lanciati i test, può committare e pushare la modifica sul branch remoto.

Quando il reviewer è soddisfatto del risultato finale e la piattaforma di continuous integration non notifica errori, commenta sulla pull request `Ready to merge.`

## Merge di una feature

Procedi con un rebase interattivo. Effettua lo squash di commit minori o temporanei (es. "Fix whitespace") su pochi commit significativi con tutti i  test verdi. Modifica il messaggio di commit dove non chiari o espliciti. Assicurati che ogni commit finale passi i test.

    git checkout <branch-name>
    git fetch origin
    git rebase -i origin/master
    rake

Ricontrolla la lista di commit definitiva. Controlla i file modificati. A questo punto mergia il branch su master, senza merge bubbles.

    git log origin/master..<branch-name>
    git diff --stat origin/master
    git checkout master
    git merge <branch-name> --ff-only
    git push

Rimuovi il feature branch da remoto e da locale.

    git push origin --delete <branch-name>
    git branch --delete <branch-name>

## Deploy di una feature

Controlla la lista di nuovi commit. Controlla i file modificati. A questo punto pusha sul branch staging, deployando.

    git fetch staging
    git log staging/master..master
    git diff --stat staging/master
    git push staging

Se necessario, lancia le nuove migrazioni and restarta i dynos.

    heroku run rake db:migrate --remote staging
    heroku restart --remote staging

Testa la nuova feature su browser e deploya in produzione, seguendo il medesimo iter.

    git fetch production
    git log production/master..master
    git diff --stat production/master
    git push production
    heroku run rake db:migrate --remote production
    heroku restart --remote production
    watch heroku ps --remote production

Chiudi la pull request and commenta `Merged.`

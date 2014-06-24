# Installation von Gitlab 7.0!

Diese Anleitung bezieht sich direkt auf die offiziellen Installationsanleitung [hier](https://gitlab.com/gitlab-org/gitlab-ce/blob/master/doc/install/installation.md). Für Uberspace sind jedoch einige Dinge unwichtig, andere zusätzlich nötig. Genauere Beschreibungen sind in der offiziellen Anleitung zu finden. Viele der Befehle aus der offiziellen Anleitung laufen jedoch auch ohne das sudo.

## Dependencies

### Python

Python wird in einer Version 2.5+ (nicht 3.0+) benötigt.

Python ist auf den Uberspaceservern bereits aktiviert. Jedoch manchmal noch in der Version 2.4. Prüft das mit dem Befehl `python -V`. Falls noch die alte Version aktiv ist, könnt ihr nach der Anleitung [hier](http://uberspace.de/dokuwiki/development:python) eine neuere aktivieren.

### Git 

Git wird in der Version 1.7.10+ benötigt. Nicht zu verwechseln mit 1.7.1, was auf meinem Uberspace die Version war.

Git ist auch bereits auf den Servern installiert. Prüft mit `git --version` eure Version. Falls sie zu alt ist könnt ihr über Toast eine neuere Version installieren. Sucht dazu [hier](http://code.google.com/p/git-core/downloads/list) eine Version und kopiert den Link zum Tarball. Mit `toast arm [URL zum Tarball]` wird diese installiert und eingerichtet.

### Redis

Installiere Redis wie [hier](http://uberspace.de/dokuwiki/database:redis) beschrieben. Redis akzeptiert auf Uberspace nur Verbindungen zu seinem Socket, was in allen Konfigurationsfiles von GitLab zu beachten ist.

### Ruby

Ruby wird in der Version 1.9.3+ benötigt.

Auf den Uberspaceservern wird standartmäßig eine ältere Version genutzt. [Hier](http://uberspace.de/dokuwiki/development:ruby) wird erklärt wie die neueren zur Verfügung stehenden Versionen aktiviert werden.

#### .bashrc vs. .bash_profile
SSH Keys werden innerhalb GitLab über die GitLab Shell verwaltet. Da diese SSH Keys direkt auf das GL Shell Script verweisen wird `.bash_profile` nicht geladen. Deshalb müssen die `$PATH` Angaben die den neuen Rubypfad, sowie den Ruby Gem Pfad hinzufügen aus der `.bash_profile` in `.bashrc` kopiert werden.

#### Bundler Gem

`gem install bundler --user-install --no-ri --no-rdoc`.

`--user-install` sorgt dafür, dass der Gem im Nutzerverzeichnis statt global installiert wird.

## System User

Auf den Uberspace Servern gibt es nicht die Möglichkeit einen extra User `git` anzulegen. Der Normale Nutzer geht auch. Jedoch muss das in allen Konfigurationsfiles beachtet werden.

## Gitlab Shell

Unten die Shellbefehle nach Anleitung.

```bash
    cd ~
    git clone https://github.com/gitlabhq/gitlab-shell.git -b v1.9.6
    cd gitlab-shell
    cp config.yml.example config.yml
    nano config.yml
```
Wichtig in der `config.yml`:

Änderung aller Pfade von `/home/git/...` zu `/home/[Nutzername]/...`

```ruby
    user: [Nutzername]
    gitlab_url: "https://[Nutzername].[Host].uberspace.de"
    
    #[...]Redis Einstellungen
    
    bin: /usr/local/bin/redis-cli #Der Pfad sollte der selbe sein wie die Ausgabe von 'which redis-cli'
    # host: ...
    # port: ...
    socket: /home/[Nutzername]/.redis/sock #der Pfad findet sich auch in der Datei '~/.redis/conf' um sicherzugehen.
```

**Für die gitlab_url mit https muss der komplette Pfad inklusive Server und .uberspace.de angegeben werden, damit das Zertifikat auch passt. Auch wenn ihr eine eigene Domain haben solltet!**

Nachdem die Konfigurationdatei geändert ist.

```bash
    ./bin/install
```

## GitLab

```bash
    cd ~
    git clone https://github.com/gitlabhq/gitlabhq.git -b 7-0-stable gitlab
    cd gitlab
    
    # Clone a few config
    cp config/gitlab.yml.example config/gitlab.yml
    cp config/unicorn.rb.example config/unicorn.rb
    cp config/resque.yml.example config/resque.yml
    cp config/database.yml.mysql config/database.yml
    
    cp config/initializers/rack_attack.rb.example config/initializers/rack_attack.rb #No need to edit this later
    
    #Make a few Direktories and make sure the chmod is right
    mkdir /home/[Nutzername]/gitlab-satellites
    mkdir tmp/pids/
    mkdir tmp/sockets/
    mkdir public/uploads
    
    chmod -R u+rwX  log/
    chmod -R u+rwX  tmp/
    chmod -R u+rwX  tmp/pids/
    chmod -R u+rwX  tmp/sockets/
    chmod -R u+rwX  public/uploads
    chmod o-rwx config/database.yml
    
    #Muss nicht aber ist nützlich
    git config --global user.name "GitLab"
    git config --global user.email "gitlab@localhost"
    git config --global core.autocrlf input
```

### gitlab.yml Konfiguration

`nano config/gitlab.yml`

```ruby
    host: [Nutzername].[Host].uberspace.de
    https: true
    [...]
    user: [Nutzername] #muss auskommentiert werden
```

Unter 3: Advanced Settings:

alle `/home/git/...` ändern in `/home/[Nutzername]/...`

```ruby
git:
    bin_path: /home/[Nutzername]/.toast/armed/bin/git
```
Der `git_path` stimmt wenn git per toast installiert wurde, ansonsten `which git`, um den richtigen Pfad herauszufinden.

### unicorn.rb Konfiguration

`nano config/unicorn.rb`

alle `/home/git/...` ändern in `/home/[Nutzername]/...`
`listen "127.0.0.1:8080"...` port in einen noch freien port ändern, z.B. 9765 `listen "127.0.0.1:9765"...`

### resque.yml Konfiguration

Richtigen redis Zugang einfügen
`production: 'unix:/home/[Nutzername]/.redis/sock'`
Socket ändern, falls er bei der GitLab Shell schon anders war.

### database.yml Konfiguration

Unter `production: ` die MySQL Nutzerdaten eintragen: 

```ruby
database: [Datenbank]
username: [Nutzername]
password: [MySQL Passwort] #Wenn es nicht geändert wurde, dann unter ~/.my.cnf zu finden
```

### "Hack a little bit" damit Gitlab sicher den Redis-Socket benutzt

`nano config/environments/production.rb`

ändert `config.cache_store` und wechselt `config.serve_static_assets` von *false* auf *true*, damit Gitlab statische Files und Benutzer-Uploads laden kann!

```bash
    config.cache_store = :redis_store, {:url => resque_url}, {namespace: 'cache:gitlab'}
    config.serve_static_assets = true
```  

## Install Bundle Gems

`bundle install --deployment --without development test postgres aws`

## Init Database 

```bash
    bundle exec rake gitlab:setup RAILS_ENV=production

    # Type 'yes' to create the database.
    # When done you see 'Administrator account created:'
```

## Init Script

GitLab erstellt ein init.d Script, dass GitLab als Service ausgeführt wird. Das ist unter Uberspace nicht möglich. Bisher läuft mein GitLab nur über manuelles starten. 

`nano lib/support/init.d/gitlab`

Ändere `app_user="[Nutzername]"`

Danach den Dienst starten. Mit Status ein paar mal zur Sicherheit überprüfen. Fehler finden sich unter `log/`.

`lib/support/init.d/gitlab {start|restart|stop|status}`

## Check Status

```bash

    bundle exec rake gitlab:env:info RAILS_ENV=production

    bundle exec rake gitlab:check RAILS_ENV=production
    
```

Falls alles passt, bis auf das nicht kopierte init.d script, dann ..

`bundle exec rake assets:precompile RAILS_ENV=production`

## Apache Redirect

In `~/html` oder einem Subdomain-Ordner eine `.htaccess` erstellen und damit füllen

```bash
    <IfModule mod_rewrite.c>
        RewriteEngine On
        RewriteBase /
        RewriteRule ^(.*)$ http://127.0.0.1:[Der vorher gewählte Port]/$1 [P]
    </IfModule>
```

siehe Beispiel-.htaccess: [.htaccess](_.htaccess)

## Fertig

Jetzt sollte erstmal alles funktionieren.

## Gitlab-Shell und die SSH-keys

Ein großes Problem bei Gitlab und uberspace ist das fehlen eines separaten Users. Loggt ihr euch für gewöhnlich per Key über SSH ein, wird dies nach der Installation der Gitlab-Shell nicht mehr möglich sein. Diese blockt nämlich den Shell-Zugriff für alle auf Gitlab registrierten Keys! Trotzdem wollen wir aber gerne die Gitlab-Pfade und Nutzerrechte zum clonen, pushen etc. benutzen.
Im Grunde genommen gibt es dafür zwei Mögliche Lösungen. Beide haben den Vorteil, dass durch die Nutzung der ssh-config die angelegten Host-Aliases Systemweit zu Verfügung stehen (inklusive SFTP)!

> Zur Nutzung unter Windows kann ich leider keine klare Aussage treffen. Wenn hier jemand Erfahrung hat würde ich mich sehr über Hinweise freuen.

### Separates Keypaar

> Leider habe ich selber es bisher nicht geschafft eine Verbindung mit dieser Methode zum laufen zu bringen. Gitlab-Shell block meinen Zugriff, trotz zusätzlichem Key. Wenn jemand eine Idee oder Lösung hat, wäre ich für Tipps sehr dankbar!

Ihr legt euch ein separates Key-Paar für den Shellzugriff an.

```bash
   ssh-keygen -f ~/.ssh/shellAccess
   # optional auch mit custom-Mail/Kommentar:
   ssh-keygen -f ~/.ssh/shellAccess -C [aussagekrätiger-Name]@[server]
```

und kopiert den Inhalt des Public-Keys (`.pub`) in die `~/.ssh/authorized_keys` eures Servers.

Anschließend müsst ihr allerdings beim Login noch deutlich machen, mit welchem Keypaar ihr euch einloggen wollt. Das geht am Besten, indem ihr euch in eure `~/.ssh/config` einen Host-*Alias* anlegt, der dann in etwa wie folgt aussehen sollte:

```bash
   Host Servername.ShellKey
   HostName [Host]
   IdentityFile ~/.ssh/shellAccess
   User [Nutzername]
```

Nun sollten wir uns direkt mit `ssh Servername.ShellKey` einloggen können!

Alternativ lässt sich ssh auch zur einmaligen Nutzung ohne ssh-config überreden. Das sieht dann in etwa wie folgt aus:

```bash
   ssh -i ~/.ssh/shellAccess [Nutzername]@[Host]
```

### per Passwort

Alternative Zwei funktioniert so ähnlich, verzichtet aber auf ein weiteres Keypaar. Stattdessen loggen wir uns old-school mäßig via Passwort ein. Hierfür muss allerdings ssh konkret der Login mit einem Key verboten werden.

Das geht einmalig mit einem Konstrukt wie:

```bash
   ssh -o PreferredAuthentications=keyboard-interactive -o PubkeyAuthentication=no [Nutzername]@[Host]
```

Zum Dauerhaften deaktivieren erstellen wir uns wieder einen Eintrag in die sshconfig `~/.ssh/config`.

```bash
   Host Servername.NoKey
   HostName [Nutzername].[Host].uberspace.de
   PubkeyAuthentication no
   User [Nutzername]
```

Der Befehl zum Verbinden lautet nun `ssh Servername.NoKey`.

### ControlMaster

Falls Ihr für SSH einen ControlMaster verwendet, solltet ihr diesen für die entsprechenden Einträge deaktivieren, um eine Verwirrung des Shell-Logins und des Gitlab-Logins zu vermeiden!

Dazu einfach `ControlMaster no` noch zum Host in die sshconfig hinzufügen. Fertig!


## Upgraden

### Gitlab

Zuerst sicherheitshalber ein Backup erstellen. Anschließend einfach den Prozess stoppen und das Upgrade-Skript durchlaufen lassen.

```bash
    cd gitlab
    bundle exec rake gitlab:backup:create RAILS_ENV=production
    ./gitlab/lib/support/init.d/gitlab stop
    ruby script/upgrade.rb
```
     
Änderungen im Startup-Skript und production.rb müssen erneut gesetzt werden.

`nano lib/support/init.d/gitlab` [siehe auch](#init-script)

```bash
    app_user="[Nutzername]"
```

`nano config/environments/production.rb` [siehe auch](#quot-hack-a-little-bit-quot-damit-gitlab-sicher-den-redis-socket-benutzt)

```bash
    config.cache_store = :redis_store, {:url => resque_url}, {namespace: 'cache:gitlab'}
    config.serve_static_assets = true
```

Falls alles erfolgreich verlief kann Gitlab nun wieder gestartet werden.

```bash
    ./gitlab/lib/support/init.d/gitlab start
```

## Upgraden von 6.x auf 7.0

Das Upgrade-Skript hat bei mir das Update auf 7.0 leider nicht erkannt, weswegen ich per Hand Gitlab geupdatet habe.

Die Gitlab-Shell ist der einfachste Part:

```bash
   cd gitlab-shell
   git pull
   git checkout v1.9.6
```

Nun folgt Gitlab itself. Im wesentlich habe ich mich dabei an die offizielle Anleitung gehalten: [Docu 6.9 to 7.0](https://gitlab.com/gitlab-org/gitlab-ce/blob/master/doc/update/6.9-to-7.0.md)

Kurz: Backup. Gitlab stoppen. Git pullen. Checkout auf 7.0. Installieren. Daten migrieren. Assets kompilieren und aufräumen. *"Hacks"* wiederherstellen. Gitlab starten.

```bash
   cd gitlab
   bundle exec rake gitlab:backup:create RAILS_ENV=production
   ./gitlab/lib/support/init.d/gitlab stop
   
   git fetch --all
   git checkout 7-0-stable
   
   bundle install --without development test postgres aws --deployment
   bundle exec rake db:migrate RAILS_ENV=production
   bundle exec rake assets:clean assets:precompile cache:clear RAILS_ENV=production
   
   nano lib/support/init.d/gitlab
   nano config/environments/production.rb
   
   ./gitlab/lib/support/init.d/gitlab start
```

wobei für unsere *"Hacks"* wieder gilt:

`nano lib/support/init.d/gitlab` [siehe auch](#init-script)

```bash
    app_user="[Nutzername]"
```

`nano config/environments/production.rb` [siehe auch](#quot-hack-a-little-bit-quot-damit-gitlab-sicher-den-redis-socket-benutzt)

```bash
    config.cache_store = :redis_store, {:url => resque_url}, {namespace: 'cache:gitlab'}
    config.serve_static_assets = true
```

Nach dem Start schadet ein erneuter Check nicht:

```bash
   bundle exec rake gitlab:env:info RAILS_ENV=production
   bundle exec rake gitlab:check RAILS_ENV=production
```

***generic Helper Error***

Nach dem Update spuckt Gitlab bei mir leider beim Check und anderen bundle-task öfter den folgenden Fehler aus.

```bash
   Instance method "lock!" is already defined in ActiveRecord::Base, use generic helper instead or set StateMachine::Machine.ignore_method_conflicts = true.
```

Ich weiß nicht genau was die Ursache hierfür ist, auf den Arbeitsablauf von Gitlab wirkt sich der Fehler jedoch nicht aus.

## Impressum

Original Author: [Benjamin Milde](http://kobralab.centaurus.uberspace.de/benni/uberspace/blob/master/install.md)

Modified & Bugfixes: [Richard Weinhold](http://gitlab.ricwein.com/ricwein/uberspacetutorial/tree/master)

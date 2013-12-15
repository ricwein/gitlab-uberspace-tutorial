# Installation

Diese Anleitung bezieht sich direkt auf die offiziellen Installationsanleitung [hier](https://github.com/gitlabhq/gitlabhq/blob/master/doc/install/installation.md). Für Uberspace sind jedoch einige Dinge unwichtig, andere zusätzlich nötig. Genauere Beschreibungen sind in der offiziellen Anleitung zu finden. Viele der Befehle aus der offiziellen Anleitung laufen jedoch auch ohne das sudo.

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

`gem install bundler --no-ri --no-rdoc`

## System User

Auf den Uberspace Servern gibt es nicht die Möglichkeit einen extra User `git` anzulegen. Der Normale Nutzer geht auch. Jedoch muss das in allen Konfigurationsfiles beachtet werden.

## Gitlab Shell

Unten die Shellbefehle nach Anleitung.

```bash
    cd ~
    git clone https://github.com/gitlabhq/gitlab-shell.git -b v1.7.9
    cd gitlab-shell
    cp config.yml.example config.yml
    vim config.yml
```
Wichtig in der `config.yml`:
`user: [Nutzername]`
`gitlab_url: "https://[Nutzername].[Host].uberspace.de"`
Änderung aller Pfade von `/home/git/...` zu `/home/[Nutzername]/...`

Die Redis Einstellungen sind zu ändern. 
`bin: /usr/local/bin/redis-cli` Der Pfad sollte der selbe sein wie die Ausgabe von `which redis-cli`.
`host: ...` & `port: ...` mit einem `# ` auskommentieren, dafür bei socket das `# ` entfernen.
`socket: /home/[Nutzername]/.redis/sock`, der Pfad findet sich auch in der Datei `~/.redis/conf` um sicherzugehen.


```bash
    ./bin/install
```

## GitLab

```bash
    cd ~
    git clone https://github.com/gitlabhq/gitlabhq.git -b 6-3-stable gitlab
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
    
    #Muss nicht aber ist nützlich
    git config --global user.name "GitLab"
    git config --global user.email "gitlab@localhost"
    git config --global core.autocrlf input
```

### gitlab.yml Konfiguration

`vim config/gitlab.yml`

`host: [Nutzername].[Host].uberspace.de`
`https: true`

`user: [Nutzername]` muss auskommentiert werden

Unter 3: Advanced Settings:

alle `/home/git/...` ändern in `/home/[Nutzername]/...`

```ruby
git:
    bin_path: /home/[Nutzername]/.toast/armed/bin/git
```
Der `git_path` stimmt wenn git per toast installiert wurde, ansonten `which git`, um den richtigen Pfad herauszufinden.

### unicorn.rb Konfiguration

`vim config/unicorn.rb`

alle `/home/git/...` ändern in `/home/[Nutzername]/...`
`listen "127.0.0.1:8080"...` port in einen noch freien port ändern, z.B. 9765 `listen "127.0.0.1:9765"...`

### resque.yml Konfiguration

Richtigen redis Zugang einfügen
`produktion: 'unix:/home/[Nutzername]/.redis/sock'`
Socket ändern, falls er bei der GitLab Shell schon anders war.

### database.yml Konfiguration

Unter `produktion: ` die MySQL Nutzerdatein eintragen: 

```ruby
database: [Datenbank]
username: [Nutzername]
password: [MySQL Passwort] #Wenn es nicht geändert wurde, dann unter ~/.my.cnf zu finden
```
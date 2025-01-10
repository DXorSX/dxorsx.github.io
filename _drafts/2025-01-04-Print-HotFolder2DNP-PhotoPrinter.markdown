---
layout: single
title:  "DNP DS-RX1HS mit Gutenprint, Raspberry Pi und CUPS AirPrint fähig machen"
date:   2025-01-04 12:00:06 +0100
categories: Blog DNP-DS-RX1HS Raspberry-Pi CUPS AirPrint
comments: true
author: Tom
---

Der [DS-RX1HS][dnp-hp] von DNP ist ein 6 Zoll Fotodrucker. Mithilfe eines Raspberry Pi und ein bisschen freier Software wird der Drucker AirPrint fähig und akzeptiert Fotos direkt vom Apple iPhone.

# Quintessenz {#head-101}

Das Vorgehen ist in wenigen Schritten umrissen:
1. Installation Raspberry OS auf einem RPI (= Rapberry Pi). 
2. Konfiguration SSH und WLAN am RPI
3. CUPS und Gutenprint installieren, konfigurieren.
4. Drucker verbinden und konfigurieren
5. Drucken via AirPrint und HotFolder 

  {% highlight shell %}
    # CUPS und GutenPrint installieren
    sudo apt update
    sudo apt upgrade
    sudo apt install cups cups-pdf cups-tea4cups printer-driver-gutenprint wget
    # CUPS Konfigurieren
    sudo cupsctl --remote-admin   # Remote zugriff auf https://printbox:631 erlauben
    sudo cupsctl --share-printers # Drucker freigeben
    sudo cupsctl --remote-any.    # In jedem Subnetz
    sudo usermod -aG lpadmin pi
    sudo systemctl restart cups
  {% endhighlight %}

- Der Drucker wird via USB mit dem RPI verbunden. 
- Das WebInterface  [https://printbox:631](https://printbox:631) öffnen und der Drucker mit folgenden Parametern anlegen
  - Beschreibung:	Dai Nippon Printing DS-RX1
  - Treiber:	Dai Nippon Printing DSRX1 - CUPS+Gutenprint v5.3.1 (farbig, 2-seitiges Drucken)
  - Verbindung:	gutenprint53+usb://dnp-dsrx1/CY2D04083833
  - Standardeinstellungen:	job-sheets=none, none media=om_w288h432_104.99x162.56mm sides=one-side

Das meiste sollte CUPS direkt erkennen.

AirPrint Einrichten
  {% highlight shell %}
    pip3 install pycups
    cd $(mktemp -d)
    wget https://raw.githubusercontent.com/DXorSX/airprint-generate/blob/master/airprint-generate.py
    ./airprint-generate.py
    mv ./*.service /etc/avahi/services/
    sudo systemctl restart avahi-daemon
  {% endhighlight %}


Abfrage wieviel Drucke noch möglich sind
  {% highlight shell %}
    $(lpoptions -d DNP-DS-RX1 | sed -n 's/.*marker-message=.\([0-9].*media\)\(.*\)/\1/p')
  {% endhighlight %}


HotFolder Script: 
- https://github.com/DXorSX/HotFolder2PhotoPrinter/blob/main/hf2pp.py
Cron Job to run
- */5 * * * * find /home/FotoPrintUser/HotFolder/ -type f -exec /home/FotoPrintUser/send2DNP.sh "{}" \;



[dnp-hp]        :https://www.dnpphoto.eu/de/produkte/fotodrucker/item/252-ds-rx1hs
[airprint-gen]  :https://github.com/DXorSX/airprint-generate/blob/master/airprint-generate.py













Die Webseiten werden dabei aus dem GitHub Repository erstellt. Das GitHub Repo [https://github.com/DXorSX/dxorsx.github.io](https://github.com/DXorSX/dxorsx.github.io) wird zur Webseite [https://dxorsx.github.io/](https://dxorsx.github.io/)

Der Schnellstart ist [hier](https://pages.github.com/) perfekt erklärt.
> **Achtung:** Das Repo muss exakt so heißen wie die spätere GitHub.io Page!

Nachfolgend die Einrichtung im Schnelldurchlauf nachdem ihr ein leeres GitHub Repository angelegt habt, Details siehe weiter [unten](#head-201)
  {% highlight shell %}
    # Repository lokal clonen
    git clone https://github.com/dxorsx/dxorsx.github.io
    cd dxorsx.github.io
    # Jeykll initiieren
    jekyll new .
    # Die geänderten Inhalte zurück in das Repository pushen
    git add --all
    git commit -m "Initial commit"
    git push -u origin main
  {% endhighlight %}

Nachdem die GitHub Action gelaufen ist finden sich die Inhalte auf [https://dxorsx.github.io/](https://dxorsx.github.io/).

> **Hinweis:** `dxorsx` ist mein GitHub Username. Wollt ihr das nachvollziehen müsst ihr `dxorsx` durch euren GitHub Username ersetzen.

---


# Jekyll als WebSite Generator {#head-201}
[Jekyll][jekyll-hp] ist ein Ruby programmierter Parser der aus Plaintext Dateien eine statische Webseite baut.
Einmal konfiguriert und aufgesetzt könnt ihr Inhalte in simplen Text Dateien ablegen und Jekyll kümmert sich darum das alles hübsch formatiert, verlinkt und erreichbar ist. 

Jekyll bietet sich für die Nutzung auf [GitHub Pages][gh-pages] weil es von den GitHub Action unterstützt wird.
Das heißt der Jekyll Parser wird in der GitHub Action ausgeführt. Trotz der guten [Dokumentation][gh-pages-jekyll] habe ich einige Iterationen gebraucht um ein für mich funktionierende Setup zu finden.

Nachfolgend die Schritte die ich durchgeführt habe, die jeweils aktuelles Konfiguration findet ihr in meine [GitHub Repo][my-gh-repo].


## Schritt 1: Erstellen einer lokalen Arbeitsumgebung {#head-202}
Ich beschreibe das Vorgehen anhand einer Debian Linux Umgebung. Wenn ihr etwas anderes verwendet (Windows/macOS/FreeBSD etc.) schaut in die [Jekyll Installationsanleitung][jekyll-install].

Installiere notwendige Pakete:
  {% highlight shell %}
    sudo apt update
    sudo apt install ruby-full build-essential zlib1g-dev
  {% endhighlight %}

Mit Ruby haben wir auch den Ruby Package Manager "gem" bekommen. Diesen konfigurieren wir: 
  {% highlight shell %}
    echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
    echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
    echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
    source ~/.bashrc
  {% endhighlight %}  

und installieren anschließen Jekyll:
  {% highlight shell %}
    gem install jekyll bundler
  {% endhighlight %}


## Schritt 2: Gemfile anpassen {#head-203}
Das `Gemfile` editieren und den Eintrag `gem "jekyll"` mit einem `#` auskommentieren und die Zeile `gem "github-pages"` ergänzen.
Da ich das Jekyll Template [Minimal Mistakes][min-mistakes] nutze kommen bei mir noch die Einträge `gem "minimal-mistakes-jekyll"` und `gem "jekyll-remote-theme"` hinzu.

  {% highlight shell %}
    source "https://rubygems.org"
    # Hello! This is where you manage which Jekyll version is used to run.
    # When you want to use a different version, change it below, save the
    # file and run `bundle install`. Run Jekyll with `bundle exec`, like so:
    #
    #     bundle exec jekyll serve
    #
    # This will help ensure the proper Jekyll version is running.
    # Happy Jekylling!
    # gem "jekyll", "~> 4.3.1"
    gem "github-pages"
    # This is the default theme for new Jekyll sites. You may change this to anything you like.
    # gem "minima", "~> 2.5"
    gem "minimal-mistakes-jekyll"
    gem "jekyll-remote-theme"
    # If you want to use GitHub Pages, remove the "gem "jekyll"" above and
    # uncomment the line below. To upgrade, run `bundle update github-pages`.
    # gem "github-pages", group: :jekyll_plugins
    # If you have any plugins, put them here!
    group :jekyll_plugins do
      gem "jekyll-feed", "~> 0.12"
    end

    # Windows and JRuby does not include zoneinfo files, so bundle the tzinfo-data gem
    # and associated library.
    platforms :mingw, :x64_mingw, :mswin, :jruby do
      gem "tzinfo", ">= 1", "< 3"
      gem "tzinfo-data"
    end

    # Performance-booster for watching directories on Windows
    gem "wdm", "~> 0.1.1", :platforms => [:mingw, :x64_mingw, :mswin]

    # Lock `http_parser.rb` gem to `v0.6.x` on JRuby builds since newer versions of the gem
    # do not have a Java counterpart.
    gem "http_parser.rb", "~> 0.6.0", :platforms => [:jruby]
  {% endhighlight %}
Die aktuelle Version meines Gemfiles findet ihr im [GitHub Repo][my-gh-repo-gemfile].

> Wichtig: Nachdem ihr das Gemfile angepasst habt müsst ihr `bundle install` ausführen um alles zu aktualisieren.

## Schritt 3: _config.yml anpassen {#head-203}
Die _config.yml ist individuell und muss auch individuell angepasst werden.
Ein simple Version kann so aussehen:
  {% highlight shell %}
    domain: my-site.github.io       # if you want to force HTTPS, specify the domain without the http at the start, e.g. example.com
    url: https://my-site.github.io  # the base hostname and protocol for your site, e.g. http://example.com
    baseurl: /REPOSITORY-NAME/      # place folder name if the site is served in a subfolder
  {% endhighlight %}
oder wenn ihr das [Minimal Mistakes][min-mistakes] Template nutzt auch viel größer [_config.yml][my-gh-repo-configfile]


## Schritt 4: Lokal testen {#head-204}
Nachdem ihr alles angepasst habt bleibt der erste Testlauf, dazu bringt Jekyll einen eigenen kleinen WebServer mit den wir mit 
  {% highlight shell %}
    bundle exec jekyll server
  {% endhighlight %}
startet. Ihr könnt dann vom lokalen Rechner aus darauf zu greifen [http://127.0.0.1:4000](http://127.0.0.1:4000) vergesst nicht den Prozess jedes Mal neu zu starten wenn ihr etwas verändert habt.


## Schritt 5: Auf GitHub veröffentlichen {#head-205}
Wenn ihr nun die Inhalte auf GitHub veröffentlicht:
{% highlight shell %}
git add --all
git commit -m "Initial commit"
git push -u origin main
{% endhighlight %}

Könnt ihr in eurem GitHub Repository unter Actions den Status einsehen und falls etwas schiefgeht im Logfile nach Hinweisen suchen.


[jekyll-hp]:             https://jekyllrb.com/
[jekyll-install]:        https://jekyllrb.com/docs/installation/
[gh-pages]:              https://pages.github.io
[gh-pages-jekyll]:       https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/about-github-pages-and-jekyll
[my-gh-repo]:            https://github.com/DXorSX/dxorsx.github.io
[my-gh-repo-gemfile]:    https://github.com/DXorSX/dxorsx.github.io/blob/main/Gemfile
[my-gh-repo-configfile]: https://github.com/DXorSX/dxorsx.github.io/blob/main/_config.yml
[min-mistakes]:          https://mmistakes.github.io/minimal-mistakes/
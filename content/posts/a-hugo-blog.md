+++
author = "arrrrrmin"
title = "a hugo blog"
date = "2020-07-26"
description = "Hugo, blogs, Go, Github Pages wtf hab ich alles noch nicht gehört."
tags = ["web", "blog"]
draft = false
+++

Na dann schauen wir mal wie so ein blog funktioniert und warum wir den mit hugo gemacht haben. Der erste und wichtigste Grund **ich mag Markdown!** ... der Rest wird schon klar kommen. Ein hacker\*innen / coder\*innen / nerd\*innen ohne die nicht Markdown benutzt gibt es sicher eh nicht. **Sorry, total opinionated**.  Da ich mal irgendwo [Hugo](https://gohugo.io/) in combination mit [Markdown](https://guides.github.com/features/mastering-markdown/) aufgeschnappt hatte ... ach wieso auch nicht - wird schon klappen. Da auf der [Hugo-Hosting-Guide](https://gohugo.io/hosting-and-deployment/hosting-on-github/) Seite auch einiges zum hosting mit Github-Pages stand - und auch das total fremd aber aufregend schien - let's go - wird schon schief gehen. 

## Install hugo

Total easy, bis hier is alles klar:

```bash
# OSX/Linux (brew)
brew install hugo
# OSX (MacPorts)
port install hugo
# Win (scoop)
scoop install hugo
# From source (github) (git)
mkdir $HOME/src
cd $HOME/src
git clone https://github.com/gohugoio/hugo.git
cd hugo
go install --tags extended
# From source (curl & ruby)
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
# More options -> https://gohugo.io/getting-started/installing/
```

Für OSX & Linux user kann man [`brew`](https://brew.sh/) ([brew on github](https://github.com/Homebrew/brew)) wirklich nur empfehlen **Wieder, total opinionated**.

### warning: totally [ir]relevant section
> Eine Idee für ein anderes Projekt: Mit `brew`, `git` & `curl` möglichst viel setup nach einem OS reinstall neu einrichten und configs ziehen um ein System (am besten mit einem bash skript) from scratch einrichten zu können. Theoretisch könnte man hier auch mit `typer` eine hübsche CLI Interaktion bauen :man_shrugging:. Ok sorry, kurzer Ausflug. 

## Search for a theme

Abhängig von dem was man sich so vorstellt, am besten einfach ein hugo-theme suchen, das schon fertig ist und versuchen die Dinge zu adaptieren. **kleine Warnung**: besser eines aussuchen, das schon viele gewünschte feature implementiert und einige stars auf github hat. Während diese Blogs auch in eine dumme Falle gerannt. Eine Liste an Themes gibt es bei [hugo](https://themes.gohugo.io/). Die getting started ist schon ein super Start:

* [hugo installation](#install-hugo)
```bash
hugo new site quickstart
cd quickstart
git init
git submodule add <some-git-url-of-the-theme-you-selected> themes/<theme-name> 
# z.B.: https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke themes/ananke
```
* add `theme = <theme-name>` zur `config.toml` (in der passenden syntax wenn nicht `.toml`)
* `hugo new posts/my-first-post.md`
```md
---
title: "Test Post"
date: 2021-03-26T08:47:11+01:00
draft: false
---

Das is dann der erste Post.
```
* `hugo server`

## Theme verstehe und anpassen

Persönlich find ich die Doku etwas wirr. Grundlegend gibt es alles auf [gohugo.io](https://gohugo.io/). Fands aber besser einfach anhand von einem fertigen theme zu lernen. Am wichtigsten sind:
* `themes/<theme-name>/layouts/_default` - (`baseof.html`, `list.html`, `single.html`)
* `themes/<theme-name>/assets/css` - files for styling
* `content/posts/*.md` - content posts vom blog
* `content/*.md` - content einzelner seiten z.B. im Menü
Hugos Magie besteht eigentlich nur darin basierend auf den `.html` files in `themes/<theme-name>/layouts/` den content zu bauen und statisches als output zu bauen. Diese files kann man dann einfach hosten und muss nur einige markdown files schreiben ... fertig ist der blog.

## Hosting

Einfach und schnelles hosting mit [Github-Pages](https://pages.github.com/). Eigentlich recht gemütlich, wenn man schon mal mit [Github Actions](https://github.com/features/actions) zutun hatte. Wie oben erwähnt baut uns Hugo mit seiner Magie einige statischen html files in der ordner Struktur wie sie in `contents/*` gebaut wurde. Lokal werden die files mit `hugo --minify` gebaut. Der default Ordner ist `public/`. Da entsteht dann die eben erwähnte Struktur. Jetzt kann man mit den entsprechenden Credentials auch einfach `rsync` auf einen server machen und dort ein wenig konfigurieren fertig. Weil mir Github Pages unbekannt waren, war das attraktiver. Da das Repo sowieso bei Github liegt bot sich das auch an. Naja.. what ever.
Man braucht nur ne Github Action um nach jedem `git push` eine neue Version vom Blog zu bauen und zu publishen. Eine Action ist eine `yml` file die versucht einen kleinen Prozess (`build`, `deployment`, `linting`, o.ä.) zu bestimmten Events auszuführen. Es gibt unterschiedliche Events bei Github z.B: `push`, `deployment`, `pull_request`, `comment`, `issue`. Zu so einem event werden wir die Page bauen und auf einen Branch (`gh_pages`) pushen. Dieser `gh_pages`-branch ist von Github für Pages reserviert. Der Output der Action kann entweder ein `Release`, `Enviornment` oder `Deployment` sein. Ein `Release` bietet sich bei abgeschlossenen Software Libs an, die von anderen Usern installiert werden können, z.B. mit `pip` oder `npm`/`yarn`. Für eine Github Page wird eine `Environment` angelegt, da wir auf `gh_pages` pushen. 

Die Action sieht wie folgt aus:
```yaml
name: publish

on:
  push:
    branches:
      - main  # Set a branch to deploy

jobs:
  deploy:
    runs-on: ubuntu-20.04
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: false  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2  # action to install hugo
        with:
          hugo-version: 'latest'
          # extended: true

      - name: Build
        run: hugo --minify  # our build command

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3  # action to push to `gh_pages`
        if: github.ref == 'refs/heads/main' # github ref (latest commit sha)
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }} # your personal access tokens
          publish_dir: ./public # default output dir of `hugo --minify`
```

**Wichtig**: Den Pfad in `baseURL` in der `config.toml` anpassen. Github Page auf dem Repo: `https://github.com/<user>/<repo>`. Github Pages die von einem Repo aus publiziert werden, haben den Repo-Title als Suffix.
```toml
baseURL = "https://<user-or-org>.github.io/<repo-name>/"
``` 

Das wars auch schon. Nach jedem push auf `main` wird die Page neu publiziert. Wenns noch fragen gibt einfach ein Issue auf `https://github.com/house-of-mint/hackers-hugo-blog`.

# Some fun section

Wenn der Pfad in `baseURL` in der `config.toml` **nicht** angepasst wird, passiert ... naja das das [git log](https://github.com/house-of-mint/hackers-hugo-blog/commits/main) sagt einiges:

|Commit SHA| Message |
|-----|----------------------------------|
| ff0a83 | :bug: slug bug |
| 2430eb | :bug: un nu? |
| 52671c | :bug: ach komm ... |
| 440d9e | :bug: jetzt aber!!! |
| d888cb | :bug: ich will nich mehr |
| 977231 | :bug: bitte gh_pages einfach mal funktionieren bitte |
| 204201 | :bug: nkjbhkgvcfjxdh |
| 7e3fc3 | :art: asdf |
| bf9dc7 | :bug: trynerror |
| 65a973 | :bug: Das könnte tatsächlich funktionieren |
| d58924 | Revert ":bug: Das könnte tatsächlich funktionieren" |
| 423ba3 | :bug: pure verzweiflung |
| 1078f5 | :bug: bitte ich will ins bett |
| b5716a | :tada: nu aber!!! |




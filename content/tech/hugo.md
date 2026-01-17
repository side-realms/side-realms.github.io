---
title: "Hugo移行した"
date: 2023-08-29T17:54:50+09:00
draft: false
---

自分のやったことを備忘録的に残すものがほしかった

## theme が反映されない問題

手元で toml を更新して hugo server してもインポートした theme が全く反映されなかった．
hugo するときの出力ディレクトリを docs に変更して，github pages の方でも以下のように変更する

setting -> pages -> root -> docs 


## theme の html を変更しても反映されない問題

使っている hugo のバージョンが対応していなかったっぽいのでダウングレードしたら反映するようになった


## Github Action

- git push したときに勝手に更新してくれるようにしたい

```yaml
name: GitHub Pages

on:
  push:
    branches:
      - main  # Set a branch name to trigger deployment
  pull_request:

jobs:
  deploy:
    runs-on: ubuntu-22.04
    concurrency:
      group: ${{ github.workflow }}-${{ github.ref }}
    steps:
      - uses: actions/checkout@v3
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.85.0'
          extended: true

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        if: ${{ github.ref == 'refs/heads/main' }}
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

## デザインを変える

#### リンクの並びを変える + ポストのグループごとにリンクつける

```html:layouts\_default\baseof.html
{{- with .Site.Menus.main }}
      <nav class="app-header-menu">
        {{- range $key, $item := . }}
          {{- if ne $key 0 }}
            {{ $.Site.Params.menu_item_separator | safeHTML }}
          {{ end }}
          <font size="5">
          <a class="app-header-menu-item" href="{{ $item.URL }}">{{ $item.Name }}</a>
          </font>
          <br>
        {{- end }}
      </nav>
      {{- end }}
```

#### カラーパレット
```toml
[params.style]
darkestColor = "#1b2d45"  
darkColor = "#001534"     
lightColor = "#b7c9e4"    
lightestColor = "#fffffe" 
primaryColor = "#fde24f"
```

#### Klatex が使えるようにする

```html
<div class="post-footer">
      {{ partial "footer.html" . }}
      {{ template "_internal/disqus.html" . }}
    </div>
```

```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.10.1/dist/katex.min.css" integrity="sha384-dbVIfZGuN1Yq7/1Ocstc1lUEm+AT+/rCkibIcC/OmWo5f0EA48Vf8CytHzGrSwbQ" crossorigin="anonymous">
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.10.1/dist/katex.min.js" integrity="sha384-2BKqo+exmr9su6dir+qCw08N2ZKRucY4PrGQPPWU1A7FtlCGjmEGFqXCv5nyM5Ij" crossorigin="anonymous"></script>
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.10.1/dist/contrib/auto-render.min.js" integrity="sha384-kWPLUVMOks5AQFrykwIup5lo0m3iMkkHrD0uJ4H5cjeGihAutqP0yW0J6dpFiVkI" crossorigin="anonymous" onload="renderMathInElement(document.body);"></script>
<script>
  document.addEventListener("DOMContentLoaded", function() {
    renderMathInElement(document.body, {delimiters: [
      {left: "$$", right: "$$", display: true},
      {left: "$", right: "$", display: false}]
    });
  });
</script>
```

#### h2 に装飾する

```html
h2{
  border-left-width: 0.15em;
  border-left-style: solid;
  border-left-color: #7f5af0;
  padding: 0.1em 0.3em;
}
```

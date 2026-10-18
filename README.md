# go-ruby-widgets/docs

Documentation for [go-ruby-widgets](https://github.com/go-ruby-widgets) —
the **widgets**, **tui** and **mvvm** Ruby adapters over the
[go-widgets](https://github.com/go-widgets) stack — built with
[Hugo](https://gohugo.io/) and this fleet's [Hextra fork](https://github.com/tannevaled/hextra) — upstream Hextra plus the opt-in features the fleet's sites use. Served at
<https://go-ruby-widgets.github.io/docs/>.

## Build locally

```sh
pip install -r requirements.txt
mkdocs serve
```

## Deploy

Pushing to `main` runs `.github/workflows/docs.yml`, which deploys the `0.1`
version (aliased `latest`) via `mike` to the `gh-pages` branch.

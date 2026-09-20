# mortewiki

**Orchestratore** della Wiki Mortebianca: avvia sotto Podman un **pod** con:
- **web** → nginx + PHP-FPM (Alpine) che serve la wiki (**wiki-tube**).
- **cron** → un container Python che, ad intervalli, lancia **wiki-tube-sync**
  (`manage.py run`: sync canali + build + relink).

Tutto è **montato da submodule** → `git clone --recursive` è **completamente autonomo**.

## Struttura
```
mortewiki/
  docker-compose.yml   # pod: web + cron
  .env                 # variabili (canale, creator, developed by, lingua, ...)
  content/             # contenuti Markdown (pages/, transcripts/) — committati qui
  code/wiki-tube/      # SUBMODULE -> osharko/wiki-tube      (codice wiki, open)
  sync/wiki-tube-sync/ # SUBMODULE -> osharko/wiki-tube-sync (script, privato)
```

## Configurazione
Le variabili (canale youtube, nome autore/creator, "Developed by", titolo, lingua)
stanno in **`.env`** e vengono iniettate nei container. `wiki-tube` NON contiene valori
hardcoded: se una variabile manca, viene loggata un **warning** in console e quella
porzione è ignorata.

## Avvio
```bash
git clone --recursive git@github.com:osharko/mortewiki.git
cd mortewiki
podman-compose up -d        # oppure: podman compose up -d
```
Web -> **http://127.0.0.1:8080** (usare `127.0.0.1`, non `localhost`).

## Submodule
- `code/wiki-tube` → cod. wiki.
- `sync/wiki-tube-sync` → sync (privato).

```bash
git submodule update --init --recursive
```

## Contenuti
I contenuti Markdown sono committati direttamente in `mortewiki/content/`
(`content/pages/<data>-<id>/` + `content/transcripts/`) e montati in `/data/content`.

## Note
- Il sync usa `manage.py run` (config in `sync/wiki-tube-sync/config.yaml`), i percorsi
  dati (`RAW_DIR`, `WIKI_CONTENT`) vengono da env nel container cron.
- Il cron è schedulato ogni 6h (modifica `docker-compose.yml`).

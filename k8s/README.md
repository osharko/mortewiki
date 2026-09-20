# k8s — Wiki Mortebianca (distribuzione)

Manifest k3s per la wiki (namespace `wiki`). Il **codice e i contenuti** arrivano via **git**
(clone ricorsivo di `mortewiki` in un PVC); le immagini sono **standard** (`nginx` +
`php:8.3-fpm-alpine`). Niente hostname/dominio qui: il **tunnel Cloudflare/DNS** è
configurato a parte (namespace `infra` + Cloudflare), non in questo repo.

## Applicare
```bash
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/pvc.yaml
kubectl apply -f k8s/web.yaml        # crea anche il ConfigMap wiki-nginx + la Service
# Secret con la chiave SSH per il push del cron (non committato):
kubectl -n wiki create secret generic wiki-gitkey \
  --from-file=id_ed25519=~/.ssh/id_ed25519 \
  --from-file=id_ed25519.pub=~/.ssh/id_ed25519.pub
kubectl apply -f k8s/cronjob.yaml
```

## Pre-requisiti
- **Nodo k3s** con il clone in un PVC (Job di seed o eseguire il clone nel CSI/local-path).
- **ConfigMap `wiki-env`** (opzionale) per le variabili `WIKI_*` (canale, creator, title…);
  in mancanza usa i default della config.
- **Secret `wiki-gitkey`** per il push del cron (remote github).
- Il **tunnel Cloudflare** (`mortewiki...osharko.it` → Service `wiki-web.wiki.svc`) e il
  **record DNS** si gestiscono fuori da qui (namespace `infra` + Cloudflare).

## Cron
`CronJob sync`: ogni 6h esegue `manage.py run` (scrape/build/relink), poi **commit+push**
dei contenuti su github. I dati (contenuti + `raw_subs` + thumbnail) sono sul **PVC**
(persistiti). Per un run manuale:
```bash
kubectl -n wiki create job sync-manual --from=cronjob/sync
```

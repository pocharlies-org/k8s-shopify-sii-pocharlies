# ARCHITECTURE — k8s-shopify-sii-pocharlies

Manifests de `shopify-sii-app` (SII/facturación). Código en `pocharlies-org/shopify-sii-app`.

## Clientes y versiones
- Un servicio web (`/sii`, `skirmshop.e-dani.com`, ns `skirmshop`) y CronJobs. Tronco: `main` (Application `shopify-sii`, path `k8s`).

## Dependencias (ambos sentidos)
- Base: `pocharlies/k8s-shopify-framework-pocharlies//base?ref=deploy/prod`. Imagen: `harbor.lan.e-dani.com/homelab/shopify-sii-app` (`newTag: v1.14.6`, CronJobs con `tag@digest`).
- Contratos publicados por la app: `CONTRACTS.yaml` del repo fuente (no duplicados aquí).
- Secrets/DB: Postgres compartido, Redis/Valkey, Synapse (según `kustomization.yaml`).

## Stack
Kustomize con base remota; sin Helm.

## Componentes compartidos
Base del framework (Deployment, Service, IngressRoute); parches inline para puerto, `SHOPIFY_APP_URL` y `match`.

## Cómo se construye
`k8s/kustomization.yaml` (app), `k8s/cronjobs.yaml` (`sii_invoicing` `0 1 * * *`, `sii_monthly_report` `10 1 1 * *`), `k8s/vies-gate-migration.yaml` y `k8s/r4-address-migration.yaml` (Jobs de migración puntuales, con imagen pinneada a la versión que los creó: v1.14.1 y v1.14.5).

## Tests y validaciones
`reusable-ci.yml` (yamllint, kustomize render, kubeconform).

## CI/CD y despliegue
`ci.yml`, `pr-review.yml`, `release.yml` (publica el manifiesto). ArgoCD lee `main`.

## Decisiones y trampas
- Los Jobs de migración llevan imagen vieja a propósito: no «actualizarlos» al tag actual.
- Subir la versión de la app = editar `newTag` y los tags de los CronJobs a la vez.

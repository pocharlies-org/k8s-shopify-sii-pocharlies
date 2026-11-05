Rol: devops · Fecha: 2026-11-05T11:45Z · Sesión: 9e0970fc-1202-4f9b-88e3-b02f9334d871 · Estado: LISTO

# Evidencia — pin v1.16.19 + SII_BACKFILL_SHOP en el hook PreSync

## Render de kustomize (la validación que exige ARCHITECTURE.md / CI)

Comando (en `~/k8s/k8s-shopify-sii-pocharlies`, rama `SC-1800-pin-v1.16.19`):

```
$ kubectl kustomize k8s
```

PASS — exit 0, manifiesto renderizado completo (1041 líneas).

Comprobaciones sobre el render:

```
$ grep -c "v1.16.19@sha256:e4dae70f434eb0f8cc996fae6c97ada7ef6a61f084783193ef437bde34095bd6" render.yaml
5            # Deployment sii-app (vía images:) + 2 CronJobs + 2 Jobs del hook PreSync
$ grep -c "SII_BACKFILL_SHOP" render.yaml
2            # Job apply + Job verify del hook multi-tenant
$ grep -n "v1\.16\.1[0-8]" render.yaml
(sin resultados)  # ningún manifiesto activo queda en v1.16.18
$ grep -n "skirmshop-spain.myshopify.com" render.yaml
516:          value: skirmshop-spain.myshopify.com   # SII_BACKFILL_SHOP (Job migrate)
581:          value: skirmshop-spain.myshopify.com   # SII_BACKFILL_SHOP (Job verify)
```

## Digest de la imagen

```
$ skopeo inspect --format '{{.Digest}}' docker://harbor.lan.e-dani.com/homelab/shopify-sii-app:v1.16.19
sha256:e4dae70f434eb0f8cc996fae6c97ada7ef6a61f084783193ef437bde34095bd6
```

El tag `v1.16.19` apunta al squash `fc37832` de `main` en `shopify-sii-app` (PR #55, ya fusionada).

## Criterios de SC-1800 (spec sin etiquetas: numeración por posición)

- C9 (despliegue en producción): este PR es el cambio GitOps que fija la versión a desplegar
  (Deployment + 2 CronJobs + hook PreSync) y da al hook la variable que el script exige desde
  v1.16.19. La medida viva (`Synced Healthy`, `readyReplicas >= 1`) se toma tras el sync de ArgoCD,
  no en el diff.

## Jobs históricos

`vies-gate-migration.yaml`, `r4-address-migration.yaml`, `effective-date-migration.yaml` y
`oss-country-daily-ownership.yaml` quedan intactos (imagen vieja a propósito, según
ARCHITECTURE.md «Decisiones y trampas»).

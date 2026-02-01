# Backend – Overlay desenvolvimento (branch develop)

Este overlay aplica o **backend** da Digital Step Flow no ambiente de **desenvolvimento**. É atualizado automaticamente quando o branch **develop** do repositório [pucrs-tcc-digital-step-flow-backend](https://github.com/raphaelmoraes/pucrs-tcc-digital-step-flow-backend) recebe push.

## Atualização automática

O job **Deploy to Dev** no workflow `ci.yml` do repositório do backend:

1. Faz build e push da imagem com tag de desenvolvimento (ex.: `24.13.0-r1-dev`).
2. Atualiza a tag da imagem neste arquivo: `kustomization.yaml` (campo `images[].newTag`).
3. Faz commit e push para este repositório (backend-k8s).

O Argo CD (ou outro mecanismo de sincronização) aplica as alterações no cluster.

## Configuração do overlay

| Aspecto        | Valor |
|----------------|--------|
| **Namespace**  | `development` |
| **Imagem**     | `raphaelmoraes/digital-step-flow-backend` |
| **Tag**        | Desenvolvimento (ex.: `24.13.0-r1-dev`), definida pelo CI do backend |
| **Replicas**   | 1 |
| **Labels**     | `app.kubernetes.io/component: backend`, `app.kubernetes.io/environment: dev` |

## Deploy manual

A partir da raiz do repositório backend-k8s:

```bash
kubectl apply -k k8s/dev
```

## Argo CD

Se o Argo CD usar este repositório como source, configure a Application com:

- **repoURL:** URL deste repositório (backend-k8s)
- **path:** `k8s/dev`
- **targetRevision:** branch desejado (ex.: `main` ou `develop`)

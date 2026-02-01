# Digital Step Flow - Backend (Kubernetes)

Repositório dedicado aos manifests Kubernetes do **backend** da solução Digital Step Flow. Contém apenas a definição dos recursos para deploy do backend em clusters Kubernetes (Kustomize base + overlay prod).

## Objetivo

Este repositório é atualizado automaticamente pelo pipeline de CI/CD do repositório [pucrs-tcc-digital-step-flow-backend](https://github.com/raphaelmoraes/pucrs-tcc-digital-step-flow-backend): após build e push da nova imagem, o workflow **CD Deploy PROD** atualiza a tag da imagem em `k8s/prod/kustomization.yaml` e faz push para este repositório. (O deploy para desenvolvimento é feito pelo job **Deploy to Dev** no `ci.yml` do backend, que pode atualizar um overlay `k8s/dev` neste repositório quando existir.) O Argo CD (ou aplicação de deploy) sincroniza o cluster com as alterações.

## Estrutura

```
k8s/
├── base/          # Recursos base (Deployment, Service, PDB, ServiceMonitor)
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── poddisruptionbudget.yaml
│   ├── servicemonitor.yaml
│   └── kustomization.yaml
└── prod/          # Overlay produção (namespace production, tag da imagem prod)
    └── kustomization.yaml
argocd/
└── application.yaml   # Argo CD Application (opcional; pode apontar para este repo ou para o repo do backend)
```

## Imagem do backend

O Deployment utiliza a imagem **raphaelmoraes/digital-step-flow-backend** (construída a partir da imagem base hardened; ver documentação no repositório do backend). A tag é definida no overlay:

- **prod:** tag de produção (ex.: `0.0.1`), atualizada pelo workflow CD Deploy PROD do backend.

## Deploy

### Manual (kubectl + Kustomize)

```bash
# Produção
kubectl apply -k k8s/prod
```

### Via Argo CD

Se o Argo CD estiver configurado para usar **este repositório** como source:

- Aplique a Application: `kubectl apply -f argocd/application.yaml`
- Ajuste no `argocd/application.yaml` o `source.repoURL` para a URL deste repositório e o `source.path` para `k8s/prod` (ou `k8s/dev` quando existir overlay de desenvolvimento).

Se o Argo CD apontar para o repositório do **backend** (`pucrs-tcc-digital-step-flow-backend`) e o path `k8s`, os manifests serão os do backend; este repositório (backend-k8s) pode ser usado como repositório separado apenas para os overlays de ambiente, conforme a estratégia adotada.

## Dependências

- **Kubernetes** (cluster com Kustomize suportado, ex.: 1.21+).
- **Imagem do backend** publicada no registro (Docker Hub ou outro) e acessível pelo cluster.
- Para ServiceMonitor: **Prometheus Operator** (ou compatível) no cluster, se for usar monitoramento.

## Relação com outros repositórios

- **pucrs-tcc-digital-step-flow-backend:** código da aplicação, Dockerfile, CI/CD. O pipeline desse repositório atualiza a tag da imagem neste repositório (backend-k8s) no arquivo `k8s/prod/kustomization.yaml` (e em `k8s/dev/kustomization.yaml` quando existir overlay dev).

# Digital Step Flow - Backend (Kubernetes)

Repositório dedicado aos manifests Kubernetes do **backend** da solução Digital Step Flow. Contém a definição dos recursos para deploy do backend em clusters Kubernetes (Kustomize base + overlay dev para o branch **develop**).

## Objetivo

Este repositório é atualizado automaticamente pelo pipeline de CI/CD do repositório [pucrs-tcc-digital-step-flow-backend](https://github.com/raphaelmoraes/pucrs-tcc-digital-step-flow-backend):

- **Branch develop:** o job **Deploy to Dev** (no `ci.yml`) atualiza a tag da imagem em `k8s/dev/kustomization.yaml` e faz push para este repositório.
- O Argo CD (ou aplicação de deploy) sincroniza o cluster com as alterações.

## Estrutura

```
k8s/
├── base/          # Recursos base (Deployment, Service, PDB, ServiceMonitor)
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── poddisruptionbudget.yaml
│   ├── servicemonitor.yaml
│   └── kustomization.yaml
└── dev/           # Overlay desenvolvimento (branch develop)
    ├── kustomization.yaml
    └── README.md  # Documentação do deploy a partir do branch develop
argocd/
└── application.yaml   # Argo CD Application (opcional)
```

## Branch develop (desenvolvimento)

O overlay **k8s/dev** é usado para deploy a partir do branch **develop** do repositório do backend. O job **Deploy to Dev** no CI do backend atualiza a tag da imagem (ex.: `24.13.0-r1-dev`) em `k8s/dev/kustomization.yaml` e faz push para este repositório.

Detalhes do overlay dev: [k8s/dev/README.md](k8s/dev/README.md).

## Imagem do backend

O Deployment utiliza a imagem **raphaelmoraes/digital-step-flow-backend** (construída a partir da imagem base hardened; ver documentação no repositório do backend). A tag é definida no overlay:

- **dev:** tag de desenvolvimento (ex.: `24.13.0-r1-dev`), atualizada pelo job Deploy to Dev no `ci.yml` do backend quando o branch **develop** recebe push.

## Deploy

### Manual (kubectl + Kustomize)

```bash
# Desenvolvimento (branch develop)
kubectl apply -k k8s/dev
```

### Via Argo CD

Se o Argo CD estiver configurado para usar **este repositório** como source:

- Aplique a Application: `kubectl apply -f argocd/application.yaml`
- Ajuste no `argocd/application.yaml` o `source.repoURL` para a URL deste repositório e o `source.path` para `k8s/dev`.

Se o Argo CD apontar para o repositório do **backend** (`pucrs-tcc-digital-step-flow-backend`) e o path `k8s`, os manifests serão os do backend; este repositório (backend-k8s) pode ser usado como repositório separado apenas para os overlays de ambiente.

## Dependências

- **Kubernetes** (cluster com Kustomize suportado, ex.: 1.21+).
- **Imagem do backend** publicada no registro (Docker Hub ou outro) e acessível pelo cluster.
- Para ServiceMonitor: **Prometheus Operator** (ou compatível) no cluster, se for usar monitoramento.

## Relação com outros repositórios

- **pucrs-tcc-digital-step-flow-backend:** código da aplicação, Dockerfile, CI/CD. O pipeline desse repositório (job Deploy to Dev no branch **develop**) atualiza a tag da imagem neste repositório (backend-k8s) no arquivo `k8s/dev/kustomization.yaml`.

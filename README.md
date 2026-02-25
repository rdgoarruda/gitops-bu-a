# gitops-bu-a

Repositório GitOps da **Business Unit A** — contém ferramentas e aplicações específicas da BU, gerenciadas via ArgoCD pelo hub `gerencia-ho` (homologação) e `gerencia-pr` (produção).

Faz parte da estratégia de [3 repositórios GitOps](https://github.com/rdgoarruda/gitops-global/blob/main/docs/ADR-001-three-repo-gitops-strategy.md).

---

## Como Funciona

As ferramentas desta BU são **auto-descobertas** pelo `gitops-global` através de um **ApplicationSet Matrix Generator**:

```
gitops-bu-a/ho/tools/<pasta>/    ← Git Generator detecta cada subpasta
         +
bu-a-placement-ho (OCM)          ← seleciona apenas o cluster bu-a-ho
         ↓
ApplicationSet gera Application  → deploya no cluster bu-a-ho via ArgoCD
```

O ArgoCD do hub `gerencia-ho` sincroniza o conteúdo para o cluster `bu-a-ho`. Para cada pasta criada em `ho/tools/`, uma nova ArgoCD Application é gerada automaticamente — **sem alteração no `gitops-global`**.

---

## Estrutura do Repositório

```text
gitops-bu-a/
├── ho/                             # Ambiente Homologação
│   └── tools/
│       ├── namespace-config/       # ✅ Namespace, ResourceQuota, LimitRange, Ingress
│       │   ├── namespace.yaml
│       │   ├── resource-quota.yaml
│       │   ├── limit-range.yaml
│       │   ├── ingress.yaml        # 🌐 Ingress: headlamp + sample-app (HAProxy)
│       │   └── kustomization.yaml
│       └── sample-app/             # 🎯 Aplicação de demonstração
│           ├── configmap.yaml      # HTML da página + variáveis de ambiente
│           ├── deployment.yaml     # Nginx com a página customizada
│           ├── service.yaml        # ClusterIP (exposto via Ingress)
│           └── kustomization.yaml
│
└── pr/                             # Ambiente Produção (mesma estrutura)
    └── tools/
        ├── namespace-config/
        └── sample-app/
```

---

## Acesso Local (DNS .local)

Os serviços são expostos via **HAProxy Ingress Controller** instalado no cluster `bu-a-ho`:

| URL | Serviço |
|---|---|
| http://sample-app-bu-a-ho.local | Aplicação de demonstração (HO) |
| http://headlamp-bu-a-ho.local | Dashboard Kubernetes (HO) |
| http://sample-app-bu-a-pr.local | Aplicação de demonstração (PR) |
| http://headlamp-bu-a-pr.local | Dashboard Kubernetes (PR) |

> Os nomes DNS são resolvidos via `/etc/hosts` do host. Após reboot do Docker, rode `./scripts/fix-ips.sh` no repositório `gitops-ocm-foundation`.

---

## Como Adicionar uma Nova Ferramenta

1. Crie uma pasta em `ho/tools/<nome-da-tool>/`
2. Adicione os manifests Kubernetes + `kustomization.yaml`
3. (Opcional) Adicione um `Ingress` em `namespace-config/ingress.yaml` para acesso via DNS
4. Faça commit e push para `main`
5. O ArgoCD no hub `gerencia-ho` detecta e deploya automaticamente no cluster `bu-a-ho`
6. Repita em `pr/tools/` quando pronto para produção

---

## Clusters Alvo

| Ambiente | Cluster | Hub | Contexto |
|---|---|---|---|
| Homologação | `bu-a-ho` | `gerencia-ho` | `kind-bu-a-ho` |
| Produção | `bu-a-pr` | `gerencia-pr` | `kind-bu-a-pr` |

---

## Verificação de Status

```bash
# Aplicações geradas pelo ApplicationSet
kubectl --context kind-gerencia-ho get applications -n argocd -l domain=bu-a

# Recursos no cluster bu-a-ho
kubectl --context kind-bu-a-ho get all -n bu-a-workloads
kubectl --context kind-bu-a-ho get ingress -A
```

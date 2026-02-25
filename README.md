# gitops-bu-a

Repositório GitOps da **Business Unit A** — contém ferramentas e aplicações específicas da BU.

## Estrutura

```
gitops-bu-a/
├── ho/          # Ambiente Homologação
│   └── tools/   # Ferramentas da BU-A em HO (deploy via gitops-global ApplicationSet)
└── pr/          # Ambiente Produção
    └── tools/   # Ferramentas da BU-A em PR
```

## Como adicionar uma ferramenta

1. Crie uma pasta em `ho/tools/<nome-da-tool>/` com os manifests Kubernetes
2. O `ApplicationSet` em `gitops-global/domains/bu-a/ho/appset-tools-ho.yaml` detecta automaticamente
3. ArgoCD no hub `gerencia-ho` deploya no cluster `bu-a-ho`
4. Repita para `pr/tools/` quando pronto para produção

## Clusters alvo

| Ambiente | Cluster | Hub |
|---|---|---|
| HO | `bu-a-ho` | `gerencia-ho` |
| PR | `bu-a-pr` | `gerencia-pr` |

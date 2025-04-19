# Lab GitOps com ArgoCD

Laboratório prático para aprender os conceitos básicos de GitOps utilizando ArgoCD em um cluster Kubernetes local.

## Pré-requisitos

- [Minikube](https://minikube.sigs.k8s.io/docs/start/) instalado
- [kubectl](https://kubernetes.io/docs/tasks/tools/) configurado
- Conta no [GitHub](https://github.com/) ou similar

## Configuração Inicial

1. **Iniciar cluster local:**
   ```
   minikube start --driver=docker
   ```

2. **Instalar ArgoCD:**
   ```
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

3. **Acessar UI do ArgoCD:**
   ```
   kubectl port-forward svc/argocd-server -n argocd 8080:443
   ```
   Acesse: https://localhost:8080

   **Credenciais:**
   ```
   kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
   ```

## Estrutura do Repositório

```
├── apps/
│   └── nginx/
│       ├── deployment.yaml
│       └── service.yaml
└── README.md
```

## Exemplo de Manifests

`apps/nginx/deployment.yaml`:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:stable
        ports:
        - containerPort: 80
```

`apps/nginx/service.yaml`:
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: default
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: NodePort
```

## Configurar Aplicação no ArgoCD

1. **Crie nova aplicação via UI:**
   - Application Name: `nginx-lab`
   - Project: `default`
   - Sync Policy: `Automatic`
   - Repository URL: `https://github.com/rarrifano/argocd`
   - Path: `apps/nginx`
   - Cluster: `in-cluster` (https://kubernetes.default.svc)
   - Namespace: `default`

2. **Sincronize manualmente ou aguarde o auto-sync**

## Fluxo de Trabalho GitOps

1. Faça alterações nos manifests YAML
2. Commit e push para o repositório
3. ArgoCD detecta mudanças e sincroniza automaticamente
4. Verifique status na UI do ArgoCD

## Resolução de Problemas Comuns

**Erro "Namespace is missing":**
- Adicione `namespace: default` no metadata de cada recurso
- Ou defina o namespace padrão na configuração da aplicação ArgoCD

**Acesso à aplicação:**
```
minikube service nginx-service --url
```

## Referências

- [Documentação Oficial ArgoCD](https://argo-cd.readthedocs.io/)
- [Best Practices para GitOps](https://www.weave.works/technologies/gitops/)
- [Kubernetes para Iniciantes](https://kubernetes.io/pt-br/docs/tutorials/kubernetes-basics/)

---
![GitOps Flow](https://argo-cd.readthedocs.io/en/stable/assets/argocd-ui.gif)
*Fluxo básico do ArgoCD (Fonte: Documentação Oficial)*

CI: https://codefresh.io/
На бранче девелоп:
1. Билд
2. Тест
3 .CI система загружает docker образ с тэгом develop-<gitHASH>
4. Клонируется репозиторий с описанием деплоя (helm chart, ksonnet, kustomization, etc…)
    Так как у нас был helm – мы меняли appVersion в Chart.yml (https://helm.sh/docs/topics/charts/)
Дальше:
ArgoCD(https://argoproj.github.io/argo-cd/) мониторит изменения в гите(webhook или polling), если они есть, то он сверяет текущее описание helm chart с указанным в гите. Если есть разница, то новый манифест применяется к кластеру.

P.S. cert-manager для кубера: https://cert-manager.io/docs/installation/kubernetes/
P.S. Хороший аналог ArgoCD: flux. статья по сравнению: https://blog.container-solutions.com/fluxcd-argocd-or-jenkins-x-which-is-the-right-gitops-tool-for-you
Для деплоя в kubernetes используется ArgoCD, который следит за состоянием репозитория, описывающего env или само приложение. Он каждые три минуты pollит репозиторий(можно настроить вебхук - тогда изменения будут видны сразу же).
Если изменения есть, то он применяет более новый манифест(или helm chart, ksonnet, etc).
В данном репозитории лежит пример конфигурации сервиса карт (helm chart). Когда происходит какое-либо изменение (ручное или автоматизированное), то ArgoCD будет пытаться
привести сервис в кластере к желаемому состоянию.
Так как в данном случае применяется декларативный подход, пришлось его немного закостылять sed'om для автоматизации.
В приложенном описании сервиса карт, для того, чтобы изменить версию приложения, необходимо поменять appVersion в файле Chart.yml 
(по-хорошему надо еще бы повышать версию самого helm chart, но как-то не получилось придумать, как его версионировать). Ниже - пример изменения версии.

  clone-helm-repo:
    title: "Cloning repository"
    type: "git-clone"
    repo: "BugsGarage/ricodeploy"
    # CF_BRANCH value is auto set when pipeline is triggered
    # Learn more at codefresh.io/docs/docs/codefresh-yaml/variables/
    revision: "master"
    git: "github"
    stage: "deploy"
    when:
      branch:
        only:
          - develop

  bumb-app-version:
      title: "Updating version in helm chart to deploy it to cluster"
      type: "freestyle" 
      image: "alpine/git:latest" 
      working_directory: "${{clone-helm-repo}}/decks-helm"
      commands:
        - 'sed -i -E "s/^appVersion: (.*)/appVersion: ${{CF_BRANCH}}-${{CF_REVISION}}/" Chart.yaml'
        - git config --global user.email "murcagr@gmail.com"
        - git config --global user.name "codefresh-ci"
        - git add Chart.yaml
        - "git commit -m \"Updated decks image version to ${{CF_BRANCH}}-${{CF_REVISION}}\" || true"
        - git push https://murcagr:${{GITHUB_TOKEN}}@github.com/BugsGarage/ricodeploy.git
      stage: "deploy"
      when:
            branch:
              only:
                - develop

Service Mesh для Kubernetes - Istio. Позволяет удобно настроить маршрутизацию, канареечные релизы, circuit breaker и т.д. [https://istio.io/]
Удобный контроллер сертификатов для Kubernetes - cert-manager. [https://cert-manager.io/docs/]
Для CI используется Codefresh. Не очень советую, если есть деньги на нормальный CI. Он выбирался, так как количество билдов и репозиториев неограничено. [https://codefresh.io/]

# Выполнено ДЗ № 10

 - [x] Основное ДЗ
 - [] Задание со *

## В процессе сделано:

### Подготовка кластера

Проект microservices-demo выложен в gitlab.

    git remote remove origin
    git remote add gitlab https://gitlab.com/AlexPit/microservicesdemo

    git push gitlab master
    warning: redirecting to https://gitlab.com/AlexPit/microservicesdemo.git/
    Enumerating objects: 3612, done.
    Counting objects: 100% (3612/3612), done.
    Delta compression using up to 16 threads
    Compressing objects: 100% (1053/1053), done.
    Writing objects: 100% (3612/3612), 20.18 MiB | 9.19 MiB/s, done.
    Total 3612 (delta 2443), reused 3611 (delta 2442)
    remote: Resolving deltas: 100% (2443/2443), done.
    To https://gitlab.com/AlexPit/microservicesdemo
    * [new branch]      master -> master

Из демо репозитория https://gitlab.com/express42/kubernetes-platform-demo/microservices-demo/-/tree/master/deploy/charts 
скопированы helm charts в deploy/charts.

Значения repository и image в values соответствует требуемым в методических указаниях к ДЗ:

    image:
        repository: frontend
        tag: latest

Остановлен имевшийся кластер, чтобы не тратил деньги. Делается командой:

    gcloud container clusters resize cluster-logging --num-nodes=0 --node-pool=default-pool
    gcloud container clusters resize cluster-logging --num-nodes=0 --node-pool=infra-pool

Создан новый кластер с 4 нодами n1-standard-2 из GCP WEB UI. Для установки Istio нужно во вкладке "Cluster - Features" 
выбрать "Enable Istio" и выбрать Enable mTLS - permissive.

Подключение с помощью gcloud:

    gcloud container clusters get-credentials cluster-1 --zone us-central1-c --project PROJECT

Примечание: _Триал GCP закончился, поэтому ДЗ выполняется на minikube._

## Установка Istio на миникуб

В помощь https://habr.com/ru/company/flant/blog/438426/.

Воспользуемся helm: https://istio.io/latest/docs/setup/install/helm/ . В директорию Istio-resources загрузим содержимое 
архива https://github.com/istio/istio/releases/tag/1.9.1, перейдем в эту директорию.

    cd kubernetes-gitops\istio-resources
    kubectl create namespace istio-system
    helm install istio-base manifests/charts/base -n istio-system
    W0314 02:04:15.728196   30300 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
    W0314 02:04:15.728196   30300 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
    W0314 02:04:15.748063   30300 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
    W0314 02:04:15.748063   30300 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
    W0314 02:04:15.748063   30300 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; useW0314 02:04:15.748063   30300 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
    W0314 02:04:15.748063   30300 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
    W0314 02:04:15.748063   30300 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
    W0314 02:04:15.748063   30300 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
    W0314 02:04:15.748063   30300 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
    W0314 02:04:15.748063   30300 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
    W0314 02:04:15.777060   30300 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
    W0314 02:04:17.794044   30300 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
    W0314 02:04:17.802043   30300 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
    W0314 02:04:17.805043   30300 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
    W0314 02:04:17.808043   30300 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
    W0314 02:04:17.812044   30300 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
    W0314 02:04:17.820043   30300 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
    W0314 02:04:17.824058   30300 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
    W0314 02:04:17.827056   30300 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
    W0314 02:04:17.834054   30300 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
    W0314 02:04:17.837054   30300 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
    W0314 02:04:17.840055   30300 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
    W0314 02:04:17.842054   30300 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
    W0314 02:04:18.058995   30300 warnings.go:67] admissionregistration.k8s.io/v1beta1 ValidatingWebhookConfiguration is deprecated in v1.16+, unavailable in v1.22+; use admissionregistration.k8s.io/v1 ValidatingWebhookConfiguration
    W0314 02:04:18.154581   30300 warnings.go:67] admissionregistration.k8s.io/v1beta1 ValidatingWebhookConfiguration is deprecated in v1.16+, unavailable in v1.22+; use admissionregistration.k8s.io/v1 ValidatingWebhookConfiguration
    NAME: istio-base
    LAST DEPLOYED: Sun Mar 14 02:04:17 2021
    NAMESPACE: istio-system
    STATUS: deployed
    REVISION: 1
    TEST SUITE: None

    helm install istiod manifests/charts/istio-control/istio-discovery -n istio-system
    W0314 02:07:08.464820   31028 warnings.go:67] admissionregistration.k8s.io/v1beta1 MutatingWebhookConfiguration is deprecated in v1.16+, unavailable in v1.22+; use admissionregistration.k8s.io/v1 MutatingWebhookConfiguration
    W0314 02:07:09.658512   31028 warnings.go:67] admissionregistration.k8s.io/v1beta1 MutatingWebhookConfiguration is deprecated in v1.16+, unavailable in v1.22+; use admissionregistration.k8s.io/v1 MutatingWebhookConfiguration
    NAME: istiod
    LAST DEPLOYED: Sun Mar 14 02:07:08 2021
    NAMESPACE: istio-system
    STATUS: deployed
    REVISION: 1
    TEST SUITE: None

    kubectl get pods -n istio-system
    NAME                      READY   STATUS    RESTARTS   AGE
    istiod-6559894c7f-4pbxn   1/1     Running   0          87s

(Optional) Install the Istio ingress gateway chart which contains the ingress gateway components:

    helm install istio-ingress manifests/charts/gateways/istio-ingress -n istio-system

(Optional) Install the Istio egress gateway chart which contains the egress gateway components:

    helm install istio-egress manifests/charts/gateways/istio-egress -n istio-system

    kubectl get pods -n istio-system
    NAME                      READY   STATUS    RESTARTS   AGE
    istiod-6559894c7f-4pbxn   1/1     Running   0          22s

Примечание: предпочтительно в дальнейшем использовать шаги https://istio.io/latest/docs/setup/getting-started/, т.к. helm установка будет deprecated
Скачиваем архив оттуда и далее:

    istioctl install --set profile=default -y

### Continuous Integration

Сборка Docker-образов осуществляется через скрипт в директории с приложением: /hack/make-docker-images.sh

Согласно документации, нужно определить две переменных окружения:

- `TAG` - git release tag / Docker tag.
- `REPO_PREFIX` - Docker repo prefix to push images. Format: `$user/$project`.  Resulting images will be of the
  format `$user/$project/$svcname:$tag` (where `svcname` = `adservice`, `cartservice`,
  etc.)

Пришлось заменить образ для билдера сервиса cartservice на 

    FROM mcr.microsoft.com/dotnet/sdk:5.0.102-ca-patch-buster-slim-amd64 as builder

т.к. были различные ошибки при сборке образа. 

Для сборки нужно выполнить:

    export TAG=v0.0.1 && export REPO_PREFIX=abvgdeej$svcname
    microservices-demo/hack/make-docker-images.sh

### GitOps 

Установим CRD, добавляющую в кластер новый ресурс - HelmRelease

    kubectl apply -f https://raw.githubusercontent.com/fluxcd/helm-operator/master/deploy/crds.yaml

Добавим официальный репозиторий Flux

    helm repo add fluxcd https://charts.fluxcd.io
    helm repo update

Создаем namespace и ставим flux и helm-оператор. Для каждого определены файлы values.yaml

    kubectl create namespace flux
    cd kubernetes-gitops

    helm upgrade --install flux fluxcd/flux -f flux.values.yaml --namespace flux
    helm upgrade --install helm-operator fluxcd/helm-operator -f helm-operator.values.yaml --namespace flux

Установка fluxctl на локальную машину (ОС Windows 10)
    
    choco install fluxctl

Получаем SSH ключ и добавляем его в GitLab, синхронизируемся с репозиторием:

    fluxctl identity --k8s-fwd-ns flux
    
    fluxctl --k8s-fwd-ns flux sync
    Synchronizing with ssh://git@gitlab.com/AlexPit/microservicesdemo.git
    Revision of master to apply is 1795f80
    Waiting for 1795f80 to be applied ...
    Done.

#### Проверка
В репозитории с микросервисами создана директория deploy/namespaces, размещен файл microservices-demo.yaml, создающий ns.

Выполнен push, после чего в логах пода flux появилась запись

    kubectl logs flux-7cf8b5ffcd-z7rvc --namespace flux
    ts=2021-03-20T18:44:47.9134838Z caller=sync.go:606 method=Sync cmd="kubectl apply -f -" took=616.7868ms err=null output="namespace/microservices-demo created"

и namespace:

    kubectl get ns
    NAME                 STATUS   AGE
    default              Active   4h33m
    flux                 Active   147m
    istio-system         Active   4h19m
    kube-node-lease      Active   4h33m
    kube-public          Active   4h33m
    kube-system          Active   4h33m
    microservices-demo   Active   26s

### HelmRelease

Создан манифест deploy/releases/frontend.yaml. Манифест содержит комментарии, описывающие работу flux.

После push в гитлаб flux может зафейлить деплой:

    kubectl get helmrelease -n microservices-demo
    NAME       RELEASE   PHASE          STATUS   MESSAGE                                                                               AGE
    frontend             DeployFailed            Installation or upgrade failed for Helm release 'frontend' in 'microservices-demo'.   13s

Смотрим, почему:

    kubectl describe  helmrelease -n microservices-demo
    Events:
    Type     Reason             Age               From           Message
      ----     ------             ----              ----           -------
    Warning  FailedReleaseSync  21s               helm-operator  synchronization of release 'frontend' in namespace 'microservices-demo' failed: failed to prepare chart for release: chart not ready: no existing git mirror found
    Warning  FailedReleaseSync  3s (x2 over 14s)  helm-operator  synchronization of release 'frontend' in namespace 'microservices-demo' failed: installation failed: unable to build kubernetes objects from release manifest: unable to recognize "": no matches for kind "ServiceMonitor" in version "monitoring.coreos.com/v1"

нужно установить CRD, которых не хватает в кластере, они уже были в kubernetes-monitoring:

    cd .\kubernetes-monitoring\
    kubectl apply -f prometheus-operator-crd\
    customresourcedefinition.apiextensions.k8s.io/alertmanagerconfigs.monitoring.coreos.com created
    customresourcedefinition.apiextensions.k8s.io/alertmanagers.monitoring.coreos.com created
    customresourcedefinition.apiextensions.k8s.io/podmonitors.monitoring.coreos.com created
    customresourcedefinition.apiextensions.k8s.io/probes.monitoring.coreos.com created
    customresourcedefinition.apiextensions.k8s.io/prometheuses.monitoring.coreos.com created
    customresourcedefinition.apiextensions.k8s.io/prometheusrules.monitoring.coreos.com created
    customresourcedefinition.apiextensions.k8s.io/servicemonitors.monitoring.coreos.com created
    customresourcedefinition.apiextensions.k8s.io/thanosrulers.monitoring.coreos.com created

На всякий случай синхронизируемся:

    fluxctl --k8s-fwd-ns flux sync
    Synchronizing with ssh://git@gitlab.com/AlexPit/microservicesdemo.git
    Revision of master to apply is 8058cf8
    Waiting for 8058cf8 to be applied ...
    Done.

смотрим, что с деплоем:

    kubectl get helmrelease -n microservices-demo
    NAME       RELEASE    PHASE       STATUS     MESSAGE                                                                       AGE
    frontend   frontend   Succeeded   deployed   Release was successful for Helm release 'frontend' in 'microservices-demo'.   8m53s

    helm list -n microservices-demo
    NAME            NAMESPACE               REVISION        UPDATED                                 STATUS          CHART           APP VERSION
    frontend        microservices-demo      1               2021-03-20 20:56:14.3791309 +0000 UTC   deployed        frontend-0.21.0 1.16.0

    kubectl get all -n microservices-demo
    NAME                            READY   STATUS    RESTARTS   AGE
    pod/frontend-7c9f7656d5-q9bl7   1/1     Running   0          4m27s
    
    NAME               TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
    service/frontend   ClusterIP   10.104.51.64   <none>        80/TCP    4m27s
    
    NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/frontend   1/1     1            1           4m27s
    
    NAME                                  DESIRED   CURRENT   READY   AGE
    replicaset.apps/frontend-7c9f7656d5   1         1         1       4m27s

Все ок.

### Обновление образа

Обновление образа не выполнялось. У меня изначально в docker hub был frontend v0.0.2, а текущие образы собирались с тегами v.0.0.1 (с еще одно точкой),
из-за чего flux автоматически применял v0.0.2 и комитил репо до тех пор, пока я не поменял аннотацию:

    flux.weave.works/tag.chart-image: glob:v.0.0.* # semver:~v.0.0 # 2 в моей версии "v."

### Обновление Helm chart

Тоже не выполнял, но, при нахождении изменений, flux среагирует и создаст новый Deployment, удалив при этом старую версию.

Подготовлены манифесты для всех сервисов. В cartservice пришлось заменить репо зависимости redis ввиду отсутствия указанного:

    kubectl get helmrelease -n microservices-demo
    NAME                      RELEASE                   PHASE              STATUS     MESSAGE                                                                                      AGE
    adservice                 adservice                 Succeeded          deployed   Release was successful for Helm release 'adservice' in 'microservices-demo'.                 4m51s
    cartservice                                         ChartFetchFailed              Chart fetch failed for Helm release 'cartservice' in 'microservices-demo'.                   19s

    Events:
    Type     Reason             Age   From           Message
    ----     ------             ----  ----           -------
    Warning  FailedReleaseSync  51s   helm-operator  synchronization of release 'cartservice' in namespace 'microservices-demo' failed: failed to prepare chart for release: could not find : chart redis not found in https://kubernetes-charts.storage.googleapis.com/

    helm repo add googleapis https://kubernetes-charts.storage.googleapis.com/
    Error: repo "https://kubernetes-charts.storage.googleapis.com/" is no longer available; try "https://charts.helm.sh/stable" instead

    kubectl get helmrelease -n microservices-demo
    NAME                      RELEASE                   PHASE       STATUS     MESSAGE                                                                                      AGE
    adservice                 adservice                 Succeeded   deployed   Release was successful for Helm release 'adservice' in 'microservices-demo'.                 19m
    cartservice               cartservice               Succeeded   deployed   Release was successful for Helm release 'cartservice' in 'microservices-demo'.               15m
    checkoutservice           checkoutservice           Succeeded   deployed   Release was successful for Helm release 'checkoutservice' in 'microservices-demo'.           19m
    currencyservice           currencyservice           Succeeded   deployed   Release was successful for Helm release 'currencyservice' in 'microservices-demo'.           19m
    emailservice              emailservice              Succeeded   deployed   Release was successful for Helm release 'emailservice' in 'microservices-demo'.              19m
    frontend                  frontend                  Succeeded   deployed   Release was successful for Helm release 'frontend' in 'microservices-demo'.                  73m
    grafana-load-dashboards   grafana-load-dashboards   Succeeded   deployed   Release was successful for Helm release 'grafana-load-dashboards' in 'microservices-demo'.   19m
    loadgenerator             loadgenerator             Succeeded   deployed   Release was successful for Helm release 'loadgenerator' in 'microservices-demo'.             19m
    paymentservice            paymentservice            Succeeded   deployed   Release was successful for Helm release 'paymentservice' in 'microservices-demo'.            19m
    productcatalogservice     productcatalogservice     Succeeded   deployed   Release was successful for Helm release 'productcatalogservice' in 'microservices-demo'.     19m
    recommendationservice     recommendationservice     Succeeded   deployed   Release was successful for Helm release 'recommendationservice' in 'microservices-demo'.     19m
    shippingservice           shippingservice           Succeeded   deployed   Release was successful for Helm release 'shippingservice' in 'microservices-demo'.           19m

### Canary deployments с Flagger и Istio

Установка Istio описана выше, дополнительно устанавливаем Prometheus, чтобы Istio писал в него свои метрики:

    kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.9/samples/addons/prometheus.yaml -n istio-system

Установка Flagger:

    helm repo add flagger https://flagger.app
    kubectl apply -f https://raw.githubusercontent.com/weaveworks/flagger/master/artifacts/flagger/crd.yaml

Установка flagger с указанием использовать Istio:

    helm upgrade --install flagger flagger/flagger `
    --namespace=istio-system `
    --set crd.create=false `
    --set meshProvider=istio `
    --set metricsServer=http://prometheus:9090

    Release "flagger" does not exist. Installing it now.
    W0321 01:29:30.512448   11612 warnings.go:67] rbac.authorization.k8s.io/v1beta1 ClusterRole is deprecated in v1.17+, unavailable in v1.22+; use rbac.authorization.k8s.io/v1 ClusterRole
    W0321 01:29:30.926894   11612 warnings.go:67] rbac.authorization.k8s.io/v1beta1 ClusterRoleBinding is deprecated in v1.17+, unavailable in v1.22+; use rbac.authorization.k8s.io/v1 ClusterRoleBinding
    W0321 01:29:35.674390   11612 warnings.go:67] rbac.authorization.k8s.io/v1beta1 ClusterRole is deprecated in v1.17+, unavailable in v1.22+; use rbac.authorization.k8s.io/v1 ClusterRole
    W0321 01:29:36.070807   11612 warnings.go:67] rbac.authorization.k8s.io/v1beta1 ClusterRoleBinding is deprecated in v1.17+, unavailable in v1.22+; use rbac.authorization.k8s.io/v1 ClusterRoleBinding
    NAME: flagger
    LAST DEPLOYED: Sun Mar 21 01:29:26 2021
    NAMESPACE: istio-system
    STATUS: deployed
    REVISION: 1
    TEST SUITE: None
    NOTES:
    Flagger installed

В описание неймспейса microservices-demo добавлен label, указывающий на то, что в каждый pod необходимо добавить sidecar 
контейнер с envoy proxy:

    apiVersion: v1
    kind: Namespace
    metadata:
      name: microservices-demo
      labels:
        istio-injection: enabled

Push, проверяем, есть ли sidecar контейнеры:

    kubectl describe pod -l app=frontend -n microservices-demo

Если нет, то удаляем все поды в ns, развернутся новые с Envoy (нужно проверить дополнительно)

    kubectl delete pods --all -n microservices-demo

Снова выполняем describe frontend сервиса, видим, что контейнер добавился:

    Containers:
      server:
        Container ID:   docker://43fb089971a9b7873cfa4c32d71cb77ee3d9934472dda8f8b8e681a793ef7649
        Image:          abvgdeej/frontend:v.0.0.1
      ...
      istio-proxy:
        Container ID:  docker://586f83a0b297023de5f1a98b000dbb3fcdd528c3acfa5e20466653ad6a599008
        Image:         docker.io/istio/proxyv2:1.9.1
        Image ID:      docker-pullable://istio/proxyv2@sha256:361ce2cb5b30094e9a08346aec2178a753f5844a3e7ed873c0deff2b4cae3607
        Port:          15090/TCP
        Host Port:     0/TCP

В helm charts frontend добавлены манифесты Gateway и VirtualService. 

    kubectl get gateway -n microservices-demo
    NAME       AGE
    frontend   111m

Для доступа снаружи нам понадобится EXTERNAL-IP сервиса istioingressgateway.

    kubectl get svc istio-ingressgateway -n istio-system
    Error from server (NotFound): services "istio-ingressgateway" not found


Ингресс не поднялася, канарейка не поднялась, cartservice и recommendationservice не проходят readiness и liveness rpobes, loadgenerator также не заработал.
Есть подозрение, что это все из-за того, что minikube - практически тот же самый bare metal, и ингресс можно настроить, 
установив metallb. Только подвох в том, что я совсем забыл, что в развернутом на докере minikube не настроить роутинг 
(см. kubernetes-networks).

Повторять все то же самое на еще одном триале GCP желания нет, но поднятые темы очень интересные, поэтому сохраню дополнительные ссылки, 
потом использую при разработке проекта и написании документации в конфе на работе:
https://habr.com/ru/company/flant/blog/438426/
https://habr.com/ru/company/oleg-bunin/blog/493026/
https://habr.com/ru/company/flant/blog/346304/
Установка MetalLB - в домашке по развертыванию production кластера.


## Как запустить проект:
Не использовать minikube, особенно на docker runtime :) Лучше оформить еще один триал GCP
Выполнять шаги последовательно, второй репозиторий - https://gitlab.com/AlexPit/microservicesdemo
    
## Как проверить работоспособность:

## PR checklist:
 - [x] Выставлен label с темой домашнего задания (kubernetes-gitops)
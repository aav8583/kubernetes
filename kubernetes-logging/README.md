# Выполнено ДЗ № 9

 - [x] Основное ДЗ

## В процессе сделано:
###Подготовка Kubernetes кластера
Остановлен имевшийся кластер, чтобы не тратил деньги. Делается командой

    gcloud container clusters resize CLUSTER_NAME --size=0

    WARNING: The --size flag is now deprecated. Please use `--num-nodes` instead.
    Pool [default-pool] for [CLUSTER_NAME] will be resized to 0.
    
    Do you want to continue (Y/n)?  y
    
    Resizing CLUSTER_NAME...done.
    Updated [https://container.googleapis.com/v1/projects/PROJECT_NAME/zones/CLUSTER_ZONE/clusters/CLUSTER_NAME].
    
    
    Updates are available for some Cloud SDK components.  To install them,
    please run:
    $ gcloud components update

Новый кластер развернут с двумя пулами нод, по инструкции в презентации.

Инициализация через gcloud и настройка kubectl на использование нового контекста:

    gcloud container clusters get-credentials cluster-logging --zone us-central1-c --project PROJECT_NAME
    Fetching cluster endpoint and auth data.
    kubeconfig entry generated for cluster-logging.

    kubectl config current-context
    gke_third-crossing-301118_us-central1-c_cluster-logging

    kubectl get nodes
    NAME                                             STATUS   ROLES    AGE     VERSION
    gke-cluster-logging-default-pool-e5144c9f-s2ch   Ready    <none>   5m42s   v1.17.14-gke.1600
    gke-cluster-logging-infra-pool-8170fcc6-2xvf     Ready    <none>   5m44s   v1.17.14-gke.1600
    gke-cluster-logging-infra-pool-8170fcc6-3tmw     Ready    <none>   5m42s   v1.17.14-gke.1600
    gke-cluster-logging-infra-pool-8170fcc6-lxnq     Ready    <none>   5m43s   v1.17.14-gke.1600

###Установка HipsterShop
Установка знакомого проекта с помощью локального манифеста:

    kubectl create ns microservices-demo
    kubectl apply -f hipster-shop.yml -n microservices-demo
    deployment.apps/emailservice created
    service/emailservice created
    deployment.apps/checkoutservice created
    service/checkoutservice created
    deployment.apps/recommendationservice created
    service/recommendationservice created
    deployment.apps/frontend created
    service/frontend created
    service/frontend-external created
    deployment.apps/paymentservice created
    service/paymentservice created
    deployment.apps/productcatalogservice created
    service/productcatalogservice created
    deployment.apps/cartservice created
    service/cartservice created
    deployment.apps/loadgenerator created
    deployment.apps/currencyservice created
    service/currencyservice created
    deployment.apps/shippingservice created
    service/shippingservice created
    deployment.apps/redis-cart created
    service/redis-cart created
    deployment.apps/adservice created
    service/adservice created

Все поды должны развернуться в default-pool: 

    kubectl get pods -n microservices-demo -o wide
    NAME                                     READY   STATUS    RESTARTS   AGE     IP           NODE                                             NOMINATED NODE   READINESS GATES
    adservice-cb695c556-ntmqt                1/1     Running   0          4m41s   10.28.1.19   gke-cluster-logging-default-pool-e5144c9f-s2ch   <none>           <none>
    cartservice-f4677b75f-wlt56              1/1     Running   2          4m45s   10.28.1.14   gke-cluster-logging-default-pool-e5144c9f-s2ch   <none>           <none>
    checkoutservice-664f865b9b-bsvvc         1/1     Running   0          4m51s   10.28.1.9    gke-cluster-logging-default-pool-e5144c9f-s2ch   <none>           <none>
    currencyservice-bb9d998bd-2pwgl          1/1     Running   0          4m44s   10.28.1.16   gke-cluster-logging-default-pool-e5144c9f-s2ch   <none>           <none>
    emailservice-6756967b6d-qw888            1/1     Running   0          4m52s   10.28.1.8    gke-cluster-logging-default-pool-e5144c9f-s2ch   <none>           <none>
    frontend-766587959d-txzhs                1/1     Running   0          4m49s   10.28.1.11   gke-cluster-logging-default-pool-e5144c9f-s2ch   <none>           <none>
    loadgenerator-9f854cfc5-g5f2s            1/1     Running   4          4m44s   10.28.1.15   gke-cluster-logging-default-pool-e5144c9f-s2ch   <none>           <none>
    paymentservice-57c87dc78b-hmqkz          1/1     Running   0          4m47s   10.28.1.12   gke-cluster-logging-default-pool-e5144c9f-s2ch   <none>           <none>
    productcatalogservice-9f5d68b54-4l4s7    1/1     Running   0          4m46s   10.28.1.13   gke-cluster-logging-default-pool-e5144c9f-s2ch   <none>           <none>
    recommendationservice-57c49756fd-z2w9f   1/1     Running   0          4m50s   10.28.1.10   gke-cluster-logging-default-pool-e5144c9f-s2ch   <none>           <none>
    redis-cart-5f75fbd9c7-s94b5              1/1     Running   0          4m42s   10.28.1.18   gke-cluster-logging-default-pool-e5144c9f-s2ch   <none>           <none>
    shippingservice-689c6457cd-6v9nd         1/1     Running   0          4m43s   10.28.1.17   gke-cluster-logging-default-pool-e5144c9f-s2ch   <none>           <none>

###Установка EFK стека | Helm charts
Добавляем helm репозиторий EFK стека (ElasticSearch, Fluent Bit, Kibana)

    helm repo add elastic https://helm.elastic.co

Установим нужные нам компоненты, для начала без какой-либо дополнительной настройки

    kubectl create ns observability
    # ElasticSearch
    helm upgrade --install elasticsearch elastic/elasticsearch --namespace observability
    # Kibana
    helm upgrade --install kibana elastic/kibana --namespace observability
    # Fluent Bit
    helm upgrade --install fluent-bit stable/fluent-bit --namespace observability

Поды и сервисы установились не там, где нужно было. Нужно обновить установку, пропишем нужные values:

    tolerations:
        - key: node-role
          operator: Equal
          value: infra
          effect: NoSchedule
    nodeSelector:
        cloud.google.com/gke-nodepool: infra-pool

применим изменения:

    helm upgrade --install elasticsearch elastic/elasticsearch --namespace observability -f elasticsearch.values.yaml

проверяем (процесс не быстрый):

    kubectl get pods -n observability -o wide -l chart=elasticsearch -w
    
    NAME                     READY   STATUS    RESTARTS   AGE     IP          NODE                                           NOMINATED NODE   READINESS GATES
    elasticsearch-master-0   1/1     Running   0          2m22s   10.28.2.2   gke-cluster-logging-infra-pool-8170fcc6-3tmw   <none>           <none>
    elasticsearch-master-1   1/1     Running   0          4m18s   10.28.0.2   gke-cluster-logging-infra-pool-8170fcc6-2xvf   <none>           <none>
    elasticsearch-master-2   1/1     Running   0          6m10s   10.28.3.2   gke-cluster-logging-infra-pool-8170fcc6-lxnq   <none>           <none>

### Установка nginx ingress

    kubectl create ns nginx-ingress
    helm upgrade --install nginx-ingress stable/nginx-ingress  --namespace=nginx-ingress -f nginx-ingress.values.yml

Узнаем EXTERNAL-IP

    kubectl --namespace nginx-ingress get services -o wide nginx-ingress-controller
    NAME                       TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)                      AGE   SELECTOR
    nginx-ingress-controller   LoadBalancer   10.32.2.174   34.71.35.199   80:31065/TCP,443:31785/TCP   11m   app.kubernetes.io/component=controller,app=nginx-ingress,release=nginx-ingress

### Установка EFK стека | Kibana

Создаем kibana.values.yaml, обновляем релиз:
    
    helm upgrade --install kibana elastic/kibana --namespace observability -f kibana.values.yaml

Cмотрим логи fluentbit

    kubectl logs fluent-bit-2vpz5 -n observability --tail 2
    [2021/02/09 22:19:38] [error] [out_fw] no upstream connections available
    [2021/02/09 22:19:38] [ warn] [engine] failed to flush chunk '1-1612808972.718240994.flb', retry in 947 seconds: task_id=7, input=tail.0 > output=forward.0

Создаем fluentbit.values.yaml

Пропишем правила для корректной работы, в этом разделе сразу опишу что могут быть повторения полей, для этого присвоим значение time_key, "лечение" от этого по ссылке https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/

Применяем:

    helm upgrade --install fluent-bit stable/fluent-bit --namespace observability  -f fluentbit.values.yaml

    kubectl logs fluent-bit-2vpz5 -n observability --tail 2
    [2021/02/10 20:36:50] [ info] [filter_kube] API server connectivity OK
    [2021/02/10 20:36:50] [ info] [sp] stream processor started

Создаем index pattern (https://www.elastic.co/guide/en/kibana/current/index-patterns.html#settings-create-pattern)

### Мониторинг ElasticSearch

Помимо установки ElasticSearch, важно отслеживать его показатели и вовремя понимать, что пора предпринять какие-либо 
действия. Для мониторинга ElasticSearch будем использовать следующий prometheus-exporter. Обращаем внимание на tolerations.

    helm upgrade --install prometheus stable/prometheus-operator --namespace=observability -f prometheus-operator.values.yaml
    helm upgrade --install elasticsearch-exporter stable/elasticsearch-exporter --set es.uri=http://elasticsearch-master:9200 --set serviceMonitor.enabled=true --namespace=observability

    helm uninstall prometheus --namespace=observability
    helm uninstall elasticsearch-exporter --namespace=observability

    kubectl get pods -n observability

Пробрасываем порт:

    kubectl get pods --namespace observability -l "app=elasticsearch-exporter" -o jsonpath="{.items[0].metadata.name}"
    elasticsearch-exporter-5b6cc9b94d-ckdln
    kubectl port-forward elasticsearch-exporter-5b6cc9b94d-ckdln 9108:9108 --namespace observability

Заходим в графану http://grafana.34.71.35.199.xip.io/, импортируем дэшборд 4358, выбираем датасорс, 


## Как проверить работоспособность:


## PR checklist:
 - [x] Выставлен label с темой домашнего задания
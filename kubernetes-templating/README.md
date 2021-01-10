# Выполнено ДЗ № 6

 - [x] Основное ДЗ
 - [ ] Задание со *

## В процессе сделано:

### Создание кластера и cert-manager

- зарегистрирован триальный аккаунт GCP
- создан кластер my-first-cluster-1 в регионе us-central1-c, название проекта third-crossing-*number* 
- установлен на локальную машину gcloud - клиент GCP
- kubectl настроен на работу с кластером в GCP по инструкции: https://cloud.google.com/kubernetes-engine/docs/how-to/cluster-access-for-kubectl


    gcloud init
    C:\Users\alexp>gcloud init
    Welcome! This command will take you through the configuration of gcloud.
    
    Settings from your current configuration [default] are:
    accessibility:
      screen_reader: 'False'
    core:
      account: *my_account*
      disable_usage_reporting: 'True'
    
    Pick configuration to use:
     [1] Re-initialize this configuration [default] with new settings
     [2] Create a new configuration
    Please enter your numeric choice:  1
    
    Your current configuration has been set to: [default]
    
    You can skip diagnostics next time by using the following flag:
      gcloud init --skip-diagnostics
    
    Network diagnostic detects and fixes local network connection issues.
    Checking network connection...done.
    Reachability Check passed.
    Network diagnostic passed (1/1 checks passed).
    
    Choose the account you would like to use to perform operations for
    this configuration:
     [1] *my_account*
     [2] Log in with a new account
    Please enter your numeric choice:  1
    
    You are logged in as: [*my_account*].
    
    Pick cloud project to use:
     [1] third-crossing-*number*
     [2] Create a new project
    Please enter numeric choice or text value (must exactly match list
    item):  1
    
    Your current project has been set to: [third-crossing-*number*].
    
    Do you want to configure a default Compute Region and Zone? (Y/n)?  y
    
    Which Google Compute Engine zone would you like to use as project
    default?
    If you do not specify a zone via a command line flag while working
    with Compute Engine resources, the default is assumed.
     [1] us-east1-b
     [2] us-east1-c
     [3] us-east1-d
     [4] us-east4-c
     [5] us-east4-b
     [6] us-east4-a
     [7] us-central1-c
     ...
     Did not print [24] options.
     Too many options [74]. Enter "list" at prompt to print choices fully.
     Please enter numeric choice or text value (must exactly match list
     item):  7
     
     Your project default Compute Engine zone has been set to [us-central1-c].
     You can change it by running [gcloud config set compute/zone NAME].
     
     Your project default Compute Engine region has been set to [us-central1].
     You can change it by running [gcloud config set compute/region NAME].
     
     Created a default .boto configuration file at [C:\Users\alexp\.boto]. See this file and
     [https://cloud.google.com/storage/docs/gsutil/commands/config] for more
     information about configuring Google Cloud Storage.
     Your Google Cloud SDK is configured and ready to use!
     
     * Commands that require authentication will use *my_account* by default
     * Commands will reference project `third-crossing-*number*` by default
     * Compute Engine commands will use region `us-central1` by default
     * Compute Engine commands will use zone `us-central1-c` by default
     
     Run `gcloud help config` to learn how to change individual settings
     
     This gcloud configuration is called [default]. You can create additional configurations if you work with multiple accounts and/or projects.
     Run `gcloud topic configurations` to learn more.
     
     Some things to try next:
     
     * Run `gcloud --help` to see the Cloud Platform services you can interact with. And run `gcloud help COMMAND` to get help on any gcloud command.
     * Run `gcloud topic --help` to learn about advanced features of the SDK like arg files and output formatting
     
     C:\Users\alexp>gcloud container clusters get-credentials my-first-cluster-1
     Fetching cluster endpoint and auth data.
     kubeconfig entry generated for my-first-cluster-1.

Проверка конфигурации: 

    kubectl config current-context
    gke_third-crossing-*number*_us-central1-c_my-first-cluster-1
    
Можно было подключиться, используя команду из веб-интерфейса GCP:

    gcloud container clusters get-credentials my-first-cluster-1 --zone us-central1-c --project third-crossing-*number*
    
- Установлен Helm


    choco install kubernetes-helm
   
    helm version
    version.BuildInfo{Version:"v3.4.2", GitCommit:"23dd3af5e19a02d4f4baa5b2f242645a1a3af629", GitTreeState:"clean", GoVersion:"go1.14.13"}
    
    helm repo add stable https://charts.helm.sh/stable
    "stable" has been added to your repositories
    
    helm repo list
    NAME    URL
    stable  https://charts.helm.sh/stable
    
- Установлен namespace и release nginx-ingress
    
    
    kubectl create ns nginx-ingress
    
    chart deprecated: helm upgrade --install nginx-ingress stable/nginx-ingress --wait --namespace=nginx-ingress --version=1.41.3
    helm upgrade --install nginx-ingress stable/nginx-ingress --wait --namespace=nginx-ingress
    
    Release "nginx-ingress" does not exist. Installing it now.
    WARNING: This chart is deprecated
    
    kubectl get all --namespace=nginx-ingress
    NAME                                                 READY   STATUS    RESTARTS   AGE
    pod/nginx-ingress-controller-665c47fbb9-qhfgp        1/1     Running   0          4m52s
    pod/nginx-ingress-default-backend-7c868597f4-2ntnn   1/1     Running   0          4m52s
    
    NAME                                    TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)                      AGE
    service/nginx-ingress-controller        LoadBalancer   10.108.4.160    104.198.241.14   80:32041/TCP,443:30782/TCP   4m52s
    service/nginx-ingress-default-backend   ClusterIP      10.108.14.173   <none>           80/TCP                       4m52s
    
    NAME                                            READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/nginx-ingress-controller        1/1     1            1           4m53s
    deployment.apps/nginx-ingress-default-backend   1/1     1            1           4m53s
    
    NAME                                                       DESIRED   CURRENT   READY   AGE
    replicaset.apps/nginx-ingress-controller-665c47fbb9        1         1         1       4m53s
    replicaset.apps/nginx-ingress-default-backend-7c868597f4   1         1         1       4m53s
    
    
- создан namespace


    kubectl create ns cert-manager
    namespace/cert-manager created

- Добавлен репозиторий, в котором хранится актуальный helm chart certmanager


    helm repo add jetstack https://charts.jetstack.io
    
    helm repo list
    NAME            URL
    stable          https://charts.helm.sh/stable
    jetstack        https://charts.jetstack.io
    
    ---------------
    старое, не нужно, т.к. ставится флаг при установке через helm
    kubectl apply --validate=false -f .\cert-manager.crds.yaml
    customresourcedefinition.apiextensions.k8s.io/certificaterequests.cert-manager.io created
    customresourcedefinition.apiextensions.k8s.io/certificates.cert-manager.io created
    customresourcedefinition.apiextensions.k8s.io/challenges.acme.cert-manager.io created
    customresourcedefinition.apiextensions.k8s.io/clusterissuers.cert-manager.io created
    customresourcedefinition.apiextensions.k8s.io/issuers.cert-manager.io created
    customresourcedefinition.apiextensions.k8s.io/orders.acme.cert-manager.io created

- установлен cert-manager


    helm install cert-manager jetstack/cert-manager `
           --namespace cert-manager `
           --version v1.1.0 `
           --set installCRDs=true
    
    NAME: cert-manager
    LAST DEPLOYED: Sat Jan  9 23:51:43 2021
    NAMESPACE: cert-manager
    STATUS: deployed
    REVISION: 1
    TEST SUITE: None
    NOTES:
    cert-manager has been deployed successfully!
    
    In order to begin issuing certificates, you will need to set up a ClusterIssuer
    or Issuer resource (for example, by creating a 'letsencrypt-staging' issuer).
    
    More information on the different types of issuers and how to configure them
    can be found in our documentation:
    
    https://cert-manager.io/docs/configuration/
    
    For information on how to configure cert-manager to automatically provision
    Certificates for Ingress resources, take a look at the `ingress-shim`
    documentation:
    
    https://cert-manager.io/docs/usage/ingress/

    kubectl get pods --namespace cert-manager
    NAME                                       READY   STATUS    RESTARTS   AGE
    cert-manager-5fd9d77768-rc75b              1/1     Running   0          14s
    cert-manager-cainjector-78cbd59555-mgjj6   1/1     Running   0          14s
    cert-manager-webhook-756d477cc4-g2pq2      1/1     Running   0          14s
    
- Создан манифест тестовых ресурсов test-resources.yaml

    
    ` apiVersion: v1
    kind: Namespace
    metadata:
      name: cert-manager-test
    ---
    apiVersion: cert-manager.io/v1
    kind: Issuer
    metadata:
      name: test-selfsigned
      namespace: cert-manager-test
    spec:
      selfSigned: {}
    ---
    apiVersion: cert-manager.io/v1
    kind: Certificate
    metadata:
      name: selfsigned-cert
      namespace: cert-manager-test
    spec:
      dnsNames:
        - example.com
      secretName: selfsigned-cert-tls
      issuerRef:
        name: test-selfsigned
        
    kubectl apply -f test-resources.yaml
    kubectl describe certificate -n cert-manager-test
    ...
    Spec:
      Dns Names:
        example.com
      Issuer Ref:
        Name:       test-selfsigned
      Secret Name:  selfsigned-cert-tls
    Status:
      Conditions:
        Last Transition Time:  2021-01-09T21:38:45Z
        Message:               Certificate is up to date and has not expired
        Reason:                Ready
        Status:                True
        Type:                  Ready
      Not After:               2021-04-09T21:38:45Z
      Not Before:              2021-01-09T21:38:45Z
      Renewal Time:            2021-03-10T21:38:45Z
      Revision:                1
    Events:
      Type    Reason     Age   From          Message
      ----    ------     ----  ----          -------
      Normal  Issuing    8s    cert-manager  Issuing certificate as Secret does not exist
      Normal  Generated  8s    cert-manager  Stored new private key in temporary Secret resource "selfsigned-cert-2bmmt"
      Normal  Requested  8s    cert-manager  Created new CertificateRequest resource "selfsigned-cert-nkw75"
      Normal  Issuing    8s    cert-manager  The certificate has been successfully issued
      
    kubectl delete -f test-resources.yaml
    `
    
- Самостоятельное задание. Не хватает Issuer. Создан файл letsencrypt-prod.yaml

    
    apiVersion: cert-manager.io/v1
    kind: ClusterIssuer
    metadata:
      name: letsencrypt-production
    spec:
      acme:
        # The ACME server URL
        server: https://acme-v02.api.letsencrypt.org/directory
        # Email address used for ACME registration
        email: vansjet104@gmail.com
        # Name of a secret used to store the ACME account private key
        privateKeySecretRef:
          name: letsencrypt-production
        # Enable the HTTP-01 challenge provider
        solvers:
          - http01:
              ingress:
                class: nginx
                
    kubectl apply -f letsencrypt-prod.yaml
                
### chartmuseum
External IP nginx-ingress

    kubectl get svc -A
    NAMESPACE       NAME                            TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)                      AGE
    cert-manager    cert-manager                    ClusterIP      10.108.2.71     <none>           9402/TCP                     27m
    cert-manager    cert-manager-webhook            ClusterIP      10.108.12.128   <none>           443/TCP                      27m
    default         kubernetes                      ClusterIP      10.108.0.1      <none>           443/TCP                      58m
    kube-system     default-http-backend            NodePort       10.108.6.134    <none>           80:30008/TCP                 58m
    kube-system     kube-dns                        ClusterIP      10.108.0.10     <none>           53/UDP,53/TCP                58m
    kube-system     metrics-server                  ClusterIP      10.108.0.231    <none>           443/TCP                      58m
    nginx-ingress   nginx-ingress-controller        LoadBalancer   10.108.14.199   35.225.233.147   80:31076/TCP,443:32102/TCP   31m
    nginx-ingress   nginx-ingress-default-backend   ClusterIP      10.108.5.21     <none>           80/TCP
    
Нужный IP - 35.225.233.147.

Из содержимого https://github.com/helm/charts/blob/master/stable/chartmuseum/values.yaml оставляем только ingress:

    ingress:
      enabled: true
      annotations:
        kubernetes.io/ingress.class: nginx
        kubernetes.io/tls-acme: "true"
        certmanager.io/cluster-issuer: "letsencrypt-production"
        certmanager.io/acme-challenge-type: http01
      hosts:
        - name: chartmuseum.35.225.233.147.nip.io
          path: /
          tls: true
          tlsSecret: chartmuseum.secret
      env:
        open:
          DISABLE_API: false

Установка:

    kubectl create ns chartmuseum
    namespace/chartmuseum created
    
    helm upgrade --install chartmuseum stable/chartmuseum --wait `
          --namespace=chartmuseum `
          --version=2.13.2 `
          -f chartmuseum/values.yaml
          
    Release "chartmuseum" does not exist. Installing it now.
    NAME: chartmuseum
    LAST DEPLOYED: Sun Jan 10 01:06:39 2021
    NAMESPACE: chartmuseum
    STATUS: deployed
    REVISION: 1
    TEST SUITE: None
    NOTES:
    ** Please be patient while the chart is being deployed **
    
    Get the ChartMuseum URL by running:
    
      export POD_NAME=$(kubectl get pods --namespace chartmuseum -l "app=chartmuseum" -l "release=chartmuseum" -o jsonpath="{.items[0].metadata.name}")
      echo http://127.0.0.1:8080/
      kubectl port-forward $POD_NAME 8080:8080 --namespace chartmuseum
    
Проверка:
    
    helm ls -n chartmuseum
    NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
    chartmuseum     chartmuseum     1               2021-01-10 19:08:30.072734 +0300 MSK    deployed        chartmuseum-2.13.2      0.12.0
    
Переход на сайт: https://chartmuseum.35.225.233.147.nip.io/ - сертификат выдан, но Win10Pro ему не доверяет. Проверено на cert-manager 0.16.1 и на 1.1.0, выдает Kubernetes Ingress Controller Fake Certificate.
    
### Задача со * - Научитесь работать с chartmuseum

Установка и удаление (заливка chart на chartmuseum)

    helm repo add chartmuseum https://chartmuseum.35.225.233.147.nip.io
    
    Error: looks like "https://chartmuseum.35.225.233.147.nip.io" is not a valid chart repository or cannot be reached: Get "https://chartmuseum.35.225.233.147.nip.io/index.yaml": x509: certificate is valid for ingress.local, not chartmuseum.35.225.233.147.nip.io
       
По идее, дальше нужно:     
       
    helm plugin install https://github.com/chartmuseum/helm-push
    helm push mongodb/ chartmuseum 
    helm repo update
    helm search repo mongodb
    
    kubectl create ns mongodb
    helm upgrade --install mongo chartmuseum/mongodb --wait --namespace=mongodb
    kubectl get pods -n mongodb
    
### Harbor
Нужно скачать и изменить values.yaml из https://github.com/goharbor/harbor-helm

    ---
    expose:
      type: ingress
      tls:
        enabled: true
        secretName: "harbor.35.225.233.147.nip.io"
      ingress:
        hosts:
          core: "harbor.35.225.233.147.nip.io"
        controller: default
        annotations:
          kubernetes.io/ingress.class: nginx
          kubernetes.io/tls-acme: "true"
          certmanager.k8s.io/cluster-issuer: "letsencrypt-production"
          certmanager.k8s.io/acme-challenge-type: http01
    
    notary:
      enabled: false
      
Добавлен репозиторий: 
    
    helm repo add harbor-demo https://helm.goharbor.io
    helm repo update
    
namespace:

    kubectl create namespace harbor
    
Установка:
    
    helm upgrade --install harbor harbor-demo/harbor --wait `
        --namespace=harbor `
        --version=1.1.2 `
        -f harbor/values.yaml

## Как запустить проект:
    С Божьей помощью.

## Как проверить работоспособность:
    Выполнить описанные выше шаги

## PR checklist:
 - [x] Выставлен label с темой домашнего задания
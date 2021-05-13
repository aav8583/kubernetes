# Выполнено ДЗ № 11

 - [x] Основное ДЗ
 - [] Задание со *

## В процессе сделано:
Добавлен helm репозиторий hashicorp (https://github.com/hashicorp/consul-helm)

    helm repo add hashicorp https://helm.releases.hashicorp.com
    helm repo update

Установлен консул

    kubectl create ns vault
    helm install consul hashicorp/consul -f consul-values.yml -n vault

Установлен vault с кастомными values

    helm install vault hashicorp/vault -f vault-values.yml -n vault

- Примечание 1: в файле конфигурации vault-values.yml дополнительно следует изменить конфиг:

    
    storage "consul" {
        path = "vault"
        address = "consul-server:8500"
    }

Пока это не сделать, попытка unseal будет возвращать

    
    kubectl exec -n vault --stdin=true --tty=true vault-0 -- vault operator init --key-shares=1 --key-threshold=1
    Error initializing: Put "http://127.0.0.1:8200/v1/sys/init": dial tcp 127.0.0.1:8200: connect: connection refused

Причина и решение описано здесь: https://github.com/hashicorp/vault-helm/issues/233#issuecomment-723049927. При отключенном
HA mode инициализация происходит без проблем.

- Примечание 2: в файлах конфигурации указывается метка для nodeSelector - ДЗ выполняется на том же кластере, что и курсовая
работа, для развития темы


    nodeSelector: |
      assignment: cluster

Статус vault:

    helm status vault -n vault
    NAME: vault
    LAST DEPLOYED: Wed May 12 17:25:38 2021
    NAMESPACE: vault
    STATUS: deployed
    REVISION: 1
    TEST SUITE: None
    NOTES:
    Thank you for installing HashiCorp Vault!
    
    Now that you have deployed Vault, you should look over the docs on using
    Vault with Kubernetes available here:
    
    https://www.vaultproject.io/docs/
    
    
    Your release is named vault. To learn more about the release, try:
    
    $ helm status vault
    $ helm get manifest vault

Cтатус pod'ов:

    kubectl get pods -n vault

    NAME                                    READY   STATUS    RESTARTS   AGE
    vault-0                                 0/1     Running   0          78m
    vault-1                                 0/1     Running   0          78m
    vault-2                                 0/1     Running   0          78m
    vault-agent-injector-59c47cdf76-2m2g9   1/1     Running   0          78m

Pods не установились в статус READY, потому что сначала нужно провести unseal. В режиме seal vault заблокирован даже для
самого себя. Нужно проинициализировать vault, в результате будут выведены unseal key и root token. 

    kubectl exec -n vault --stdin=true --tty=true vault-0 -- vault operator init --key-shares=1 --key-threshold=1
    Unseal Key 1: y+NMOq2agD83D7yApzpvRIoSU+kDMyu2Nsf2+u5w3d4=
    
    Initial Root Token: s.kTUrxpDjldOy94k13WelUW03
    
    Vault initialized with 1 key shares and a key threshold of 1. Please securely
    distribute the key shares printed above. When the Vault is re-sealed,
    restarted, or stopped, you must supply at least 1 of these keys to unseal it
    before it can start servicing requests.
    
    Vault does not store the generated master key. Without at least 1 key to
    reconstruct the master key, Vault will remain permanently sealed!
    
    It is possible to generate new unseal keys, provided you have a quorum of
    existing unseal keys shares. See "vault operator rekey" for more information.

key-shares - количество ключей для unseal сгенерировать (фактически количество хешей мастер ключа), 
ey-treshhold - количество ключей (хешей мастер ключа) нужно для получения самого мастер ключа или разблокировки vault. 
Подробности тут: https://www.vaultproject.io/docs/commands/operator/init.

Используя полученный выше unseal key, произведен unseal:

    kubectl exec -it vault-0 -n vault -- vault operator unseal
    Unseal Key (will be hidden):
    Key             Value
    ---             -----
    Seal Type       shamir
    Initialized     true
    Sealed          false
    Total Shares    1
    Threshold       1
    Version         1.7.0
    Storage Type    consul
    Cluster Name    vault-cluster-2f56c96f
    Cluster ID      aa4cbbfa-bae7-6693-ff6b-174448b46ebc
    HA Enabled      true
    HA Cluster      https://vault-0.vault-internal:8201
    HA Mode         active
    Active Since    2021-05-12T22:43:29.823418295Z
    
    kubectl exec -it vault-1 -n vault -- vault operator unseal
    Key                    Value
    ---                    -----
    Seal Type              shamir
    Initialized            true
    Sealed                 false
    Total Shares           1
    Threshold              1
    Version                1.7.0
    Storage Type           consul
    Cluster Name           vault-cluster-2f56c96f
    Cluster ID             aa4cbbfa-bae7-6693-ff6b-174448b46ebc
    HA Enabled             true
    HA Cluster             https://vault-0.vault-internal:8201
    HA Mode                standby
    Active Node Address    http://172.17.134.159:8200

    kubectl exec -it vault-2 -n vault -- vault operator unseal
    Key                    Value
    ---                    -----
    Seal Type              shamir
    Initialized            true
    Sealed                 false
    Total Shares           1
    Threshold              1
    Version                1.6.2
    Storage Type           consul
    Cluster Name           vault-cluster-df4597b8
    Cluster ID             16489def-375e-96ed-9260-0dab8f4d7771
    HA Enabled             true
    HA Cluster             https://vault-0.vault-internal:8201
    HA Mode                standby
    Active Node Address    http://10.12.2.5:8200

Вывод статуса также можно получить, выполнив:

    kubectl exec -it vault-0 -n vault -- vault status

Просмотр списка доступных авторизаций:

    kubectl exec -it vault-0 -n vault -- vault auth list

    Error listing enabled authentications: Error making API request.

    URL: GET http://127.0.0.1:8200/v1/sys/auth
    Code: 400. Errors:
    
    * missing client token
      command terminated with exit code 2

Сначала нужно залогиниться:

    kubectl exec -it vault-0 -n vault -- vault login
    Token (will be hidden):
    Success! You are now authenticated. The token information displayed below
    is already stored in the token helper. You do NOT need to run "vault login"
    again. Future Vault requests will automatically use this token.
    
    Key                  Value
    ---                  -----
    token                s.kTUrxpDjldOy94k13WelUW03
    token_accessor       OVw0Uz1tigwx7tzTaUhNVtup
    token_duration       ∞
    token_renewable      false
    token_policies       ["root"]
    identity_policies    []
    policies             ["root"]

Повторный запрос списка:

    kubectl exec -it vault-0 -n vault -- vault auth list
    Path      Type     Accessor               Description
    ----      ----     --------               -----------
    token/    token    auth_token_316f3d86    token based credentials

### Работа с секретами

    # включение key-value секретов
    kubectl exec -it vault-0 -n vault -- vault secrets enable --path=otus kv
    Success! Enabled the kv secrets engine at: otus/
    # проверка списка
    kubectl exec -it vault-0 -n vault -- vault secrets list --detailed
    Path          Plugin       Accessor              Default TTL    Max TTL    Force No Cache    Replication    Seal Wrap    External Entropy Access    Options    Description                                                UUID
    ----          ------       --------              -----------    -------    --------------    -----------    ---------    -----------------------    -------    -----------                                                ----
    cubbyhole/    cubbyhole    cubbyhole_eee709c4    n/a            n/a        false             local          false        false                      map[]      per-token private secret storage                           e6404bd6-b68c-e4c3-4bf7-f244c9f71b68
    identity/     identity     identity_59f87dbe     system         system     false             replicated     false        false                      map[]      identity store                                             7dfb9d58-9460-c5e0-343f-0d306d9eb0b0
    otus/         kv           kv_590f14c3           system         system     false             replicated     false        false                      map[]      n/a                                                        0205bf6b-d3e3-2722-9cac-9c9ddaa2d7b4
    sys/          system       system_07716571       n/a            n/a        false             replicated     false        false                      map[]      system endpoints used for control, policy and debugging    d2220d37-2bcf-fc88-fb0a-1ba9f799429a

    # создание секретов
    kubectl exec -it vault-0 -n vault -- vault kv put otus/otus-ro/config username='otus' password='asajkjkahs'
    Success! Data written to: otus/otus-ro/config
    kubectl exec -it vault-0 -n vault -- vault kv put otus/otus-rw/config username='otus' password='asajkjkahs'
    Success! Data written to: otus/otus-rw/config

    # читаем данные по пути:
    kubectl exec -it vault-0 -n vault -- vault read otus/otus-ro/config
    Key                 Value
    ---                 -----
    refresh_interval    768h
    password            asajkjkahs
    username            otus

    # получаем значения kv секрета
    kubectl exec -it vault-0 -n vault -- vault kv get otus/otus-rw/config
    ====== Data ======
    Key         Value
    ---         -----
    password    asajkjkahs
    username    otus

На всякий случай, команды: https://www.vaultproject.io/docs/commands/read, https://www.vaultproject.io/docs/commands/kv/get

### Обновление авторизации через k8s

Включение авторизации через кубер, подробнее см. 37 слайд, видео с 1:53:40. 

    kubectl exec -it vault-0 -n vault -- vault auth enable kubernetes

Обновленный список авторизаций:

    kubectl exec -it vault-0 -n vault -- vault auth list
    Path           Type          Accessor                    Description
    ----           ----          --------                    -----------
    kubernetes/    kubernetes    auth_kubernetes_d144e488    n/a
    token/         token         auth_token_316f3d86         token based credentials

Созданы service account и ClusterRoleBinding:

    # Create a service account, 'vault-auth'
    kubectl create serviceaccount vault-auth -n vault
    # Update the 'vault-auth' service account
    kubectl apply --filename vault-auth-service-account.yml -n vault

Подготовлены переменные для записи в конфиг кубер авторизации (ОС Win10, PowerShell)

    $ENV:VAULT_SA_NAME = kubectl get sa vault-auth -o jsonpath="{.secrets[*]['name']}" -n vault
    $ENV:SA_JWT_TOKEN = kubectl get secret $ENV:VAULT_SA_NAME -o jsonpath="{.data.token}" -n vault | base64 -d
    $ENV:SA_CA_CRT = kubectl get secret $ENV:VAULT_SA_NAME -o jsonpath="{.data['ca\.crt']}" -n vault | base64 -d
    $ENV:K8S_HOST = more ~/.kube/config | grep server | grep -v '127.0.0.1' | awk '/http/ {print $NF}' # grep -v '127.0.0.1 для исключения minikube

Запись конфига в vault:

    kubectl exec -it vault-0 -n vault -- vault write auth/kubernetes/config token_reviewer_jwt="$ENV:SA_JWT_TOKEN" kubernetes_host="$ENV:K8S_HOST" kubernetes_ca_cert="$ENV:SA_CA_CRT"
    Success! Data written to: auth/kubernetes/config

Создан файл политики otus-policy.hcl, создана сама политика и роль в vault. Сейчас нельзя копировать в рутовый каталог, 
поэтому копирование произведено в /tmp

    kubectl cp otus-policy.hcl vault/vault-0:./tmp
    kubectl exec -it vault-0 -n vault -- vault policy write otus-policy /tmp/otus-policy.hcl
    Success! Uploaded policy: otus-policy
    kubectl exec -it vault-0 -n vault -- vault write auth/kubernetes/role/otus bound_service_account_names=vault-auth bound_service_account_namespaces=default policies=otus-policy ttl=24h
    Success! Data written to: auth/kubernetes/role/otus

Проверка работы авторизации. Создание пода с привязанным ServiceAccount и установка curl и jq:

    kubectl run --generator=run-pod/v1 tmp --rm -i --tty --serviceaccount=vault-auth -n vault --image alpine:3.7 

Выполнение команд в запущенном поде:
    
    apk add curl jq

Логин и получение клиентского токена:

    VAULT_ADDR=http://vault:8200
    KUBE_TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
    curl --request POST --data '{"jwt": "'$KUBE_TOKEN'", "role": "otus"}' $VAULT_ADDR/v1/auth/kubernetes/login | jq

Примечание: на последнем шаге происходит ошибка, vault возвращает:

    # curl --request POST --data '{"jwt": "'$KUBE_TOKEN'", "role": "otus"}' $VAULT_ADDR/v1/auth/kubernetes/login | jq
    % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
    Dload  Upload   Total   Spent    Left  Speed
    100   973  100    40  100   933   2857  66642 --:--:-- --:--:-- --:--:-- 74846
    {
        "errors": [
            "namespace not authorized"
        ]
    }

Ошибка описана здесь https://learn.hashicorp.com/tutorials/vault/kubernetes-sidecar. Оно и не удивительно, т.к. выше 
в команде указан bound_service_account_namespaces=default:

    kubectl exec -it vault-0 -n vault -- vault write auth/kubernetes/role/otus bound_service_account_names=vault-auth bound_service_account_namespaces=default policies=otus-policy ttl=24h

Пересоздать под в дефолтном неймспейсе не получится, возвращается ошибка:

    Error from server (Forbidden): pods "tmp" is forbidden: error looking up service account default/vault-auth: serviceaccount "vault-auth" not found

Поэтому прописываем ns vault:

    kubectl cp otus-policy.hcl vault/vault-0:./tmp
    kubectl exec -it vault-0 -n vault -- vault policy write otus-policy /tmp/otus-policy.hcl
    kubectl exec -it vault-0 -n vault -- vault write auth/kubernetes/role/otus bound_service_account_names=vault-auth bound_service_account_namespaces=vault policies=otus-policy ttl=24h

Повторяем шаги, создаем под:

    kubectl run --generator=run-pod/v1 tmp --rm -i --tty --serviceaccount=vault-auth -n vault --image alpine:3.7 

Установка приложений:

    apk add curl jq

Логин и получение клиентского токена:

    VAULT_ADDR=http://vault:8200
    KUBE_TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
    curl --request POST --data '{"jwt": "'$KUBE_TOKEN'", "role": "otus"}' $VAULT_ADDR/v1/auth/kubernetes/login | jq

    {
        "errors": [
            "permission denied"
        ]
    }

    kubectl logs service/vault -n vault
    2021-05-13T12:57:17.922Z [ERROR] auth.kubernetes.auth_kubernetes_d144e488: login unauthorized due to: Post "https://84.201.147.47/apis/authentication.k8s.io/v1/tokenreviews": x509: certificate signed by unknown authority

=\

## Как запустить проект:


## Как проверить работоспособность:


## PR checklist:
 - [x] Выставлен label с темой домашнего задания
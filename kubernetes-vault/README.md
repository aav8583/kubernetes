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

Включение авторизации через кубер, подробнее см. 37 слайд, видео с 1:51:40. 

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
    kubectl apply -f vault-auth-service-account.yml -n vault

Подготовлены переменные для записи в конфиг кубер авторизации (ОС Win10, PowerShell)

    $ENV:VAULT_SA_NAME = kubectl get sa vault-auth -o jsonpath="{.secrets[*]['name']}" -n vault
    $ENV:SA_JWT_TOKEN = kubectl get secret $ENV:VAULT_SA_NAME -o jsonpath="{.data.token}" -n vault | base64 -d
    $ENV:SA_CA_CRT = kubectl get secret $ENV:VAULT_SA_NAME -o jsonpath="{.data['ca\.crt']}" -n vault | base64 -d
    $ENV:K8S_HOST = more ~/.kube/config | grep server | grep -v '127.0.0.1' | awk '/http/ {print $NF}' # grep -v '127.0.0.1 для исключения minikube

Запись конфига в vault:

    kubectl exec -it vault-0 -n vault -- vault write auth/kubernetes/config token_reviewer_jwt="$ENV:SA_JWT_TOKEN" kubernetes_host="$ENV:K8S_HOST" kubernetes_ca_cert="$ENV:SA_CA_CRT"

- Примечание: у меня возникла проблема с сертификатом неизвестной природы. В .kube/config указан тот же самый серт, 
  который записывается в $ENV:SA_CA_CRT, но сервис vault отдает ошибку при попытке получения клиентского токена курлом:


    kubectl logs service/vault -n vault
    2021-05-13T12:57:17.922Z [ERROR] auth.kubernetes.auth_kubernetes_d144e488: login unauthorized due to: Post "https://84.201.147.47/apis/authentication.k8s.io/v1/tokenreviews": x509: certificate signed by unknown authority

Поэтому я погуглил и решил воспользоваться флагом `-tls-skip-verify` и не указывать серт в `vault write`

    kubectl exec -it vault-0 -n vault -- vault write -tls-skip-verify auth/kubernetes/config token_reviewer_jwt="$ENV:SA_JWT_TOKEN" kubernetes_host="$ENV:K8S_HOST"

Создан файл политики otus-policy.hcl, создана сама политика и роль в vault. Сейчас нельзя копировать в рутовый каталог, 
поэтому копирование произведено в /tmp

<details>
     <summary>УКАЗАНЫ НЕ ПРАВИЛЬНЫЕ КОМАНДЫ, РАБОТАТЬ НЕ БУДЕТ </summary>

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

</details>

    kubectl cp otus-policy.hcl vault/vault-0:./tmp
    kubectl exec -it vault-0 -n vault -- vault policy write otus-policy /tmp/otus-policy.hcl
    kubectl exec -it vault-0 -n vault -- vault write -tls-skip-verify auth/kubernetes/role/otus bound_service_account_names=vault-auth bound_service_account_namespaces=vault policies=otus-policy ttl=24h

Повторяем шаги, создаем под:

    kubectl run --generator=run-pod/v1 tmp --rm -i --tty --serviceaccount=vault-auth -n vault --image alpine:3.7 

Установка приложений:

    apk add curl jq

Логин и получение клиентского токена:

    VAULT_ADDR=http://vault:8200
    KUBE_TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
    curl --request POST --data '{"jwt": "'$KUBE_TOKEN'", "role": "otus"}' $VAULT_ADDR/v1/auth/kubernetes/login | jq

<details>
    <summary>Проблемы с сертификатом кластера без скипа проверки серта. Есть подозрение, что в этом как-то замешан Istio, т.к. в их конфигах фигурирует "tokenreviews"</summary>    

    {
        "errors": [
            "permission denied"
        ]
    }

    kubectl logs service/vault -n vault
    2021-05-13T12:57:17.922Z [ERROR] auth.kubernetes.auth_kubernetes_d144e488: login unauthorized due to: Post "https://84.201.147.47/apis/authentication.k8s.io/v1/tokenreviews": x509: certificate signed by unknown authority

Помогло https://stackoverflow.com/questions/53663455/certificate-error-when-deploying-hashicorp-vault-with-kubernetes-auth-method-on

</details>

![img.png](img.png)

    "auth": {
        "client_token": "s.pnvAyCL6oJZ4ARKHiOwd3NYY",
        "accessor": "rH8AkiO1OFqlK5qZdRGfi8UO",

Результаты чтения секретов:

    curl --header "X-Vault-Token:s.pnvAyCL6oJZ4ARKHiOwd3NYY" $VAULT_ADDR/v1/otus/otus-ro/config
    {"request_id":"d24be16f-7783-3cb1-cae2-6faba6bcaba6","lease_id":"","renewable":false,"lease_duration":2764800,"data":{"password":"asajkjkahs","username":"otus"},"wrap_info":null,"warnings":null,"auth":null}
    
    curl --header "X-Vault-Token:s.pnvAyCL6oJZ4ARKHiOwd3NYY" $VAULT_ADDR/v1/otus/otus-rw/config
    {"request_id":"4712d2e4-9c7a-b8da-2c16-c13875229d87","lease_id":"","renewable":false,"lease_duration":2764800,"data":{"password":"asajkjkahs","username":"otus"},"wrap_info":null,"warnings":null,"auth":null}

Результаты записи:

    curl -H "X-Vault-Token: s.pnvAyCL6oJZ4ARKHiOwd3NYY" -H "Content-Type: application/json" -X POST -d '{"bar":"baz"}' $VAULT_ADDR/v1/otus/otus-ro/config
    {"errors":["1 error occurred:\n\t* permission denied\n\n"]}

    curl -H "X-Vault-Token: s.pnvAyCL6oJZ4ARKHiOwd3NYY" -H "Content-Type: application/json" -X POST -d '{"bar":"baz"}' $VAULT_ADDR/v1/otus/otus-rw/config
    {"errors":["1 error occurred:\n\t* permission denied\n\n"]}

    curl -H "X-Vault-Token: s.pnvAyCL6oJZ4ARKHiOwd3NYY" -H "Content-Type: application/json" -X POST -d '{"bar":"baz"}' $VAULT_ADDR/v1/otus/otus-rw/config1

Ответы на вопросы:
- запись в ro невозможна - только чтение
- запись в rw невозможна - политика не настроена на обновление секрета
- в config1 запись удалась, т.к. это добавление нового

Подробнее о политиках: https://learn.hashicorp.com/tutorials/vault/policies

политика rw должна выглядеть так:

    path "otus/otus-rw/*" {
        capabilities = ["read", "create", "list", "update"]
    }

### Пример с nginx

    git clone https://github.com/hashicorp/vault-guides.git

Отредактированы configmap.yaml и example-k8s-spec.yaml, помещены в директорию configs-k8s вместе с ранее созданным vault-auth-service-account.yaml

configmap - используется политика для vault с шаблоном файла html для nginx, в котором указаны переменные для получения 
кредов из vault-а. А в example* фактически разворачивание nginx и подцепление конфигмепа как volume в страничку nginx. 
Подробнее в README.md примера.

    kubectl apply -f .\configmap.yaml -n vault
    kubectl apply -f .\example-k8s-spec.yaml -n vault
    kubectl port-forward pod/vault-agent-example 8080:80 -n vault

![img_1.png](img_1.png)

Создан CA на базе vault

Включим pki секретс в vault:

    kubectl exec -it vault-0 -- vault secrets enable pki
    kubectl exec -it vault-0 -- vault secrets tune -max-lease-ttl=87600h pki
    kubectl exec -it vault-0 -- vault write -field=certificate pki/root/generate/internal \
    common_name="example.ru" ttl=87600h > CA_cert.crt

Если что, для удаления можно воспользоваться:

    curl \
    --header "X-Vault-Token: s.NW5XyfagsqL1ongygqq6NOuv" \
    --request DELETE \
    $VAULT_ADDR/v1/pki/root

Прописаны urls для ca и отозванных сертификатов:

    kubectl exec -it vault-0 -- vault write pki/config/urls \
    issuing_certificates="http://vault:8200/v1/pki/ca" \
    crl_distribution_points="http://vault:8200/v1/pki/crl"

Создан промежуточный сертификат

    kubectl exec -it vault-0 -- vault secrets enable --path=pki_int pki
    kubectl exec -it vault-0 -- vault secrets tune -max-lease-ttl=87600h pki_int
    kubectl exec -it vault-0 -- vault write -format=json pki_int/intermediate/generate/internal \
    common_name="example.ru Intermediate Authority" | jq -r '.data.csr' > pki_intermediate.csr

Прописан промежуточный сертификат в vault

    kubectl cp pki_intermediate.csr vault-0:/home/vault/
    kubectl exec -it vault-0 -- vault write -format=json pki/root/sign-intermediate \
    csr=@/home/vault/pki_intermediate.csr \
    format=pem_bundle ttl="43800h" | jq -r '.data.certificate' > intermediate.cert.pem
    kubectl cp intermediate.cert.pem vault-0:/home/vault/
    kubectl exec -it vault-0 -- vault write pki_int/intermediate/set-signed certificate=@/home/vault/intermediate.cert.pem

Созданы и отозваны новые сертификаты, создана роль для выдачи сертификатов:

    kubectl exec -it vault-0 -- vault write pki_int/roles/example-dot-ru \
    allowed_domains="example.ru" allow_subdomains=true max_ttl="720h"

Создан сертификат:

    kubectl exec -it vault-0 -- vault write pki_int/issue/example-dot-ru common_name="gitlab.example.ru" ttl="24h"

    Key                 Value
    ---                 -----
    ca_chain            [-----BEGIN CERTIFICATE-----
    MIIDnDCCAoSgAwIBAgIUSStASL0avna3F8Zu35flNBW/OT4wDQYJKoZIhvcNAQEL
    BQAwFTETMBEGA1UEAxMKZXhhbXBsZS5ydTAeFw0yMTAzMDMyMTEwNDlaFw0yNjAz
    MDIyMTExMTlaMCwxKjAoBgNVBAMTIWV4YW1wbGUucnUgSW50ZXJtZWRpYXRlIEF1
    dGhvcml0eTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAMPfCvJzAhhN
    IGAj2qJowyaWN4KYAK7AOFoqlOgupk9r2Z+5ViDomhwPo49oglwB4MMBMem2eeyX
    0Zd5vB1RLd2C56N/Z3trJfPLbzAVapTscd0O4nGoFaugzJUZJ7iax7bhWGUqAWHr
    BKRZouXtDkaEdarkgopTq0riic5RBoxFJSnWT09vCv8SfDXCnQK9q6KoUKAmGTnn
    YBdI8qIodUWMF0weegDbjnrUeP4WGJ75dGts26AQHR08MNz6r408RumRN2+U/wSJ
    coX4jga4yzlr5YFkTRVnFIwDgrGQ4+a7Sc608YYx/AF92pfaVeITn9leV2LJMhga
    3caNVNjQ7Q8CAwEAAaOBzDCByTAOBgNVHQ8BAf8EBAMCAQYwDwYDVR0TAQH/BAUw
    AwEB/zAdBgNVHQ4EFgQUNN9t7HpZV5lQguM6cKP7pnGg3a4wHwYDVR0jBBgwFoAU
    /Yot3eNJT2YlgCiryvIcNP9gnkMwNwYIKwYBBQUHAQEEKzApMCcGCCsGAQUFBzAC
    hhtodHRwOi8vdmF1bHQ6ODIwMC92MS9wa2kvY2EwLQYDVR0fBCYwJDAioCCgHoYc
    aHR0cDovL3ZhdWx0OjgyMDAvdjEvcGtpL2NybDANBgkqhkiG9w0BAQsFAAOCAQEA
    023qah1fKzF8JKvnJ5c7/GvSaWA+AIKDEUoYtqfjDEada0rKmLyaBNDfeP7nIKiS
    wPKbJw2wrDDRkg+qiXIkKeqZg1sL/4fIOrAG1ArxWAjOKcWLivyTxtKDdUcm1kwV
    drZdRM5mNZVG1Rbj01GhyhEXlw8BnNw0wmsR+cidUF4iQTpPef2Gro7c5mUfxfoq
    srQdR/jUiGxHXIvOGxxXAcBClOKqQR0wrg6a1EjMwYj52lkcG3noWhfpbdeSeOsE
    48W76S9vVcpPR7nqxNDA5TO7YmVXPEUsjnsfdBKy9QnUSN+EB/yhbsc8Kv3bN5SW
    u4xv5PaIcODoG7z05C8imw==
    -----END CERTIFICATE-----]
    certificate         -----BEGIN CERTIFICATE-----
    MIIDZzCCAk+gAwIBAgIUXukx9lvMESRRql/vu2lKxdfoibwwDQYJKoZIhvcNAQEL
    BQAwLDEqMCgGA1UEAxMhZXhhbXBsZS5ydSBJbnRlcm1lZGlhdGUgQXV0aG9yaXR5
    MB4XDTIxMDMwMzIxMTYzMVoXDTIxMDMwNDIxMTcwMVowHDEaMBgGA1UEAxMRZ2l0
    bGFiLmV4YW1wbGUucnUwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQC2
    sI1ctS0z2PixOzqmNEXf5BV/fA3CQNjkbnVAcEHJXPlt4rFVOicbUzh1rXnlQI8U
    4aHqBX0W/eSWF8MF7Q02a7cJenKycypPMZuONMU3fhg6zzZb4kaUnNvPi+lEqeUj
    yfrnv/k5yraxTPQY8u4ymkrzy+FjIzOTCo5xEa1fl9ejdlZWykS53FAG4nZxDSaZ
    NmZgC6wI0RDQbBQfLWoIXzjsqKvLDY78L70RIqxX7KYOoVWNGVcPWDPH/WXZ+ZWf
    wuhhqoFou3chHnVuyUbnNqu4j+STo0X0XG2HKjyqWaNItakEkA3wxHEaRGSPzqwA
    Nagvtsri1efzgcWFD56XAgMBAAGjgZAwgY0wDgYDVR0PAQH/BAQDAgOoMB0GA1Ud
    JQQWMBQGCCsGAQUFBwMBBggrBgEFBQcDAjAdBgNVHQ4EFgQUTmfJEcEQ1qC243sO
    yeGRzd8Kc2swHwYDVR0jBBgwFoAUNN9t7HpZV5lQguM6cKP7pnGg3a4wHAYDVR0R
    BBUwE4IRZ2l0bGFiLmV4YW1wbGUucnUwDQYJKoZIhvcNAQELBQADggEBAGmbLqnY
    IFoS3MAxVET6NKw5j3cytLBjCQ59UvEcFChmwxzeWQfdMz2PFl1SM0jHglSiIcsS
    ACZszojBErbi5wX7wtKBAz36Wz2dRbpCwpfVmWSmBXnQi3jOoxYxR/QKghHARhjO
    VNXES4Ej5iv8uCqSts5BXEhw5WJBVcfjDYYlJ+NBFVzlfxIZdAcBNGwUKKY3a/U1
    0co7b1c4Fw1W0gi3iDC/ZQkDOdHMWKqVaxHiB2mbRXx5+VUl62uRMR9qM1CAR7Vs
    bO+Fb1ws3i/EtmFOA44GdzK1nLNPZQ23RBu7NQEpyclAlDt/N888vxLLZlPZjiaq
    vK/V94mBRJQLyTw=
    -----END CERTIFICATE-----
    expiration          1614892621
    issuing_ca          -----BEGIN CERTIFICATE-----
    MIIDnDCCAoSgAwIBAgIUSStASL0avna3F8Zu35flNBW/OT4wDQYJKoZIhvcNAQEL
    BQAwFTETMBEGA1UEAxMKZXhhbXBsZS5ydTAeFw0yMTAzMDMyMTEwNDlaFw0yNjAz
    MDIyMTExMTlaMCwxKjAoBgNVBAMTIWV4YW1wbGUucnUgSW50ZXJtZWRpYXRlIEF1
    dGhvcml0eTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAMPfCvJzAhhN
    IGAj2qJowyaWN4KYAK7AOFoqlOgupk9r2Z+5ViDomhwPo49oglwB4MMBMem2eeyX
    0Zd5vB1RLd2C56N/Z3trJfPLbzAVapTscd0O4nGoFaugzJUZJ7iax7bhWGUqAWHr
    BKRZouXtDkaEdarkgopTq0riic5RBoxFJSnWT09vCv8SfDXCnQK9q6KoUKAmGTnn
    YBdI8qIodUWMF0weegDbjnrUeP4WGJ75dGts26AQHR08MNz6r408RumRN2+U/wSJ
    coX4jga4yzlr5YFkTRVnFIwDgrGQ4+a7Sc608YYx/AF92pfaVeITn9leV2LJMhga
    3caNVNjQ7Q8CAwEAAaOBzDCByTAOBgNVHQ8BAf8EBAMCAQYwDwYDVR0TAQH/BAUw
    AwEB/zAdBgNVHQ4EFgQUNN9t7HpZV5lQguM6cKP7pnGg3a4wHwYDVR0jBBgwFoAU
    /Yot3eNJT2YlgCiryvIcNP9gnkMwNwYIKwYBBQUHAQEEKzApMCcGCCsGAQUFBzAC
    hhtodHRwOi8vdmF1bHQ6ODIwMC92MS9wa2kvY2EwLQYDVR0fBCYwJDAioCCgHoYc
    aHR0cDovL3ZhdWx0OjgyMDAvdjEvcGtpL2NybDANBgkqhkiG9w0BAQsFAAOCAQEA
    023qah1fKzF8JKvnJ5c7/GvSaWA+AIKDEUoYtqfjDEada0rKmLyaBNDfeP7nIKiS
    wPKbJw2wrDDRkg+qiXIkKeqZg1sL/4fIOrAG1ArxWAjOKcWLivyTxtKDdUcm1kwV
    drZdRM5mNZVG1Rbj01GhyhEXlw8BnNw0wmsR+cidUF4iQTpPef2Gro7c5mUfxfoq
    srQdR/jUiGxHXIvOGxxXAcBClOKqQR0wrg6a1EjMwYj52lkcG3noWhfpbdeSeOsE
    48W76S9vVcpPR7nqxNDA5TO7YmVXPEUsjnsfdBKy9QnUSN+EB/yhbsc8Kv3bN5SW
    u4xv5PaIcODoG7z05C8imw==
    -----END CERTIFICATE-----
    private_key         -----BEGIN RSA PRIVATE KEY-----
    MIIEpQIBAAKCAQEAtrCNXLUtM9j4sTs6pjRF3+QVf3wNwkDY5G51QHBByVz5beKx
    VTonG1M4da155UCPFOGh6gV9Fv3klhfDBe0NNmu3CXpysnMqTzGbjjTFN34YOs82
    W+JGlJzbz4vpRKnlI8n657/5Ocq2sUz0GPLuMppK88vhYyMzkwqOcRGtX5fXo3ZW
    VspEudxQBuJ2cQ0mmTZmYAusCNEQ0GwUHy1qCF847Kiryw2O/C+9ESKsV+ymDqFV
    jRlXD1gzx/1l2fmVn8LoYaqBaLt3IR51bslG5zaruI/kk6NF9Fxthyo8qlmjSLWp
    BJAN8MRxGkRkj86sADWoL7bK4tXn84HFhQ+elwIDAQABAoIBAQClEcGpEstVHacX
    /LxxkKnSMvR5zE1iR9WyEVxAbS4EE84MS9iPeYv8VKWfLrAFROADrhvuqCbur1nr
    hGzi3d4iXhF0rv8T3ptMEzbKt0O7cGPUP4aOX1YG0fSLA5AySpCQVeAvpnY6kb+h
    VDb6lAZGEsPGpWFxgk0Hf3JVF/PfenrwZUGrktszFF3l9r0wXaiiqHL6FGnjnit8
    n5BKqKbHd3B+gOcbtyAcZD6ZwyuTGEyKKGfQR7mKYjXF5VbmJ1SlyTmKjhT30jHy
    LoiLexk9Cly9OZdms7wGKPY3YSmUYVvUaXeqoCtH/ZGPKfFhN3XR7ndQcn9WH3Z6
    be7+n1sBAoGBAMR2C9d6m0m0Pn+HbdAIl/svi0o/78TwKTQ13KeuF/7MpgyOiKLF
    emTSiz+3eLGztXdseWwdDL52Ui+y2/sLpUTjQVA1/QPm/EHcrLecr61gGET+cw3G
    OdJzyQ3Z/4dHVzXwqPzIfkFjMa3POO8roM8X8sjnkiHvi5sRiFXhumIJAoGBAO4O
    FLcXlNbjp9cGP7iRtKESBlJGZcJ4bAh6+IHFbtmg/yOyF5J6aVWn3B/XIbCrzrx9
    0eymsiXVtFMttQp4kMr5G9j3jzIV4Q80ZQlzLq0jFSjJXzkF8fthJpWPC7dMsHWM
    33PGGZb60wzV467kly3dhXGDT4vuPviXHOE7GaOfAoGAOyjtAfM6xeQQGekXSVj9
    Ize68x3zvtMvJTi+/INxWFoZ+pgFTza2V5wLMKG4J5LdJ1wz6DmLN+N7dj+e/KcS
    Gn9wkI3hZgZtmguwuw3k3Qmd5VDWJqS1jsktFw25Y+w4t9aDnLNnSZtsP1GybFsv
    7ozgoF0TZUK0QHr0GiCCNrkCgYEA3Z1oNYcTffXT436ixZ2Hjcds8R0uUJuw3zgz
    rwPxDVMvErkR7sBc3Wv2pgGuEH3xaVKsomYRRN2tER5lAwl4qiy8ewEEYvkxWulJ
    AkIjevVFFoJZTom1W3N26xaPLqaLQ/PQdkQ+wGpjHfjlDIUsJHusZh97Z2Z1YxGy
    xg8x8DsCgYEAjHeMKuIPXj2yCYYumvBMrhu5SrgJfvXeJ1UwWjb2VYMed9joKba9
    PdCXG4c9bvxhA1S0CbCUVOhVVlFI+m1aOkm6k2sOTpUFre+SsW8EOTkiDkgI9fbm
    3NSe6MJYiv136crWlyDAOakK12IDe1JlKUtavTVCaMncuI4vTl9/AFQ=
    -----END RSA PRIVATE KEY-----
    private_key_type    rsa
    serial_number       5e:e9:31:f6:5b:cc:11:24:51:aa:5f:ef:bb:69:4a:c5:d7:e8:89:bc

Отозван сертификат:

    kubectl exec -it vault-0 -- vault write pki_int/revoke serial_number="5e:e9:31:f6:5b:cc:11:24:51:aa:5f:ef:bb:69:4a:c5:d7:e8:89:bc"
    Key                        Value
    ---                        -----
    revocation_time            1614806282
    revocation_time_rfc3339    2021-03-03T21:18:02.559895629Z
    isie@isie-VirtualBox:~/otus/IsieIam_platform/kubernetes-vault(kubernetes-vault)$

## Как запустить проект:


## Как проверить работоспособность:


## PR checklist:
 - [x] Выставлен label с темой домашнего задания
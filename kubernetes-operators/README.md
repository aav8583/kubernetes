# Выполнено ДЗ № 7

 - [x] Основное ДЗ
 - [ ] Задание со *

## В процессе сделано:
###CR, CRD и Validation
Создан CustomResource deploy/cr.yml.
Применение заканчивается ошибкой:

    kubectl apply -f deploy/cr.yml
    error: the path "deploy/cr.yml" does not exist

Ошибка связана с отсутствием объектов типа MySQL в API kubernetes.

CustomResourceDefinition - это ресурс для определения других ресурсов (далее CRD).
Создали CRD deploy/crd.yml, применяем:

    kubectl apply -f deploy/crd.yml
    Warning: apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
    customresourcedefinition.apiextensions.k8s.io/mysqls.otus.homework created

Создаем еще раз CR:

    kubectl apply -f deploy/cr.yml
    mysql.otus.homework/mysql-instance created

С созданными объектами можно взаимодействовать через kubectl:
    
    kubectl get crd
    kubectl get mysqls.otus.homework
    kubectl describe mysqls.otus.homework mysql-instance

На данный момент мы никак не описали схему нашего CustomResource. Объекты типа mysql могут иметь абсолютно произвольные 
поля, нам бы хотелось этого избежать, для этого будем использовать validation. Для начала удалим CR mysql-instance:

    kubectl delete mysqls.otus.homework mysql-instance

Добавлены validation параметры, применяем заново:

    kubectl apply -f deploy/crd.yml
    Warning: apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
    customresourcedefinition.apiextensions.k8s.io/mysqls.otus.homework configured
    
    kubectl apply -f deploy/cr.yml
    mysql.otus.homework/mysql-instance created

Добавим директиву spec.validation.spec.required в CRD. Удалим CR поле с размером хранилища, запустим cr и crd:

    kubectl apply -f deploy/crd.yml
    customresourcedefinition.apiextensions.k8s.io/mysqls.otus.homework configured
    
    kubectl apply -f deploy/cr.yml
    The MySQL "mysql-instance" is invalid: spec.storage_size: Required value

Данный ответ и ожидался.

### Операторы
Оператор включает в себя CustomResourceDefinition и сustom сontroller.

CRD содержит описание объектов CR.

Контроллер следит за объектами определенного типа, и осуществляет всю логику работы оператора

Используемый/написанный нами контроллер будет обрабатывать два типа событий:
1) При создании объекта типа (kind: mySQL), он будет:
   

    * Cоздавать PersistentVolume, PersistentVolumeClaim, Deployment, Service для mysql
    * Создавать PersistentVolume, PersistentVolumeClaim для бэкапов базы данных, если их
      еще нет.
    * Пытаться восстановиться из бэкапа

2) При удалении объекта типа (kind: mySQL), он будет:


    * Удалять все успешно завершенные backup-job и restore-job
    * Удалять PersistentVolume, PersistentVolumeClaim, Deployment, Service для mysql

Часть с Python не делалась. Создадим и применим манифесты:


    service-account.yml
    role.yml
    role-binding.yml
    deploy-operator.yml

    kubectl apply -f deploy\
    mysql.otus.homework/mysql-instance unchanged
    Warning: apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
    customresourcedefinition.apiextensions.k8s.io/mysqls.otus.homework unchanged
    deployment.apps/mysql-operator created
    clusterrolebinding.rbac.authorization.k8s.io/workshop-operator created
    clusterrole.rbac.authorization.k8s.io/mysql-operator created
    serviceaccount/mysql-operator created

Проверяем что появились pvc:

    kubectl get pvc
    NAME                        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
    backup-mysql-instance-pvc   Bound    pvc-7c7b05ae-46fa-45d3-a3b0-6071a8871b98   1Gi        RWO            standard       4m18s
    mysql-instance-pvc          Bound    pvc-312c8512-48e2-4f18-9a1e-817562e863bf   1Gi        RWO            standard       4m18s

Команда export не работает в Windows Powershell, поэтому придется немного переделать команды заполнения базы:

    kubectl get pods -l app=mysql-instance -o jsonpath="{.items[*].metadata.name}"
    mysql-instance-6785949c48-4m9kd

    kubectl exec -it mysql-instance-6785949c48-4m9kd -- mysql -u root -potuspassword -e "CREATE TABLE test ( id smallint unsigned not null auto_increment, name varchar(20) not null, constraint pk_example primary key (id) );" otus-database
    kubectl exec -it mysql-instance-6785949c48-4m9kd -- mysql -potuspassword -e "INSERT INTO test ( id, name ) VALUES ( null, 'some data' );" otus-database
    kubectl exec -it mysql-instance-6785949c48-4m9kd -- mysql -potuspassword -e "INSERT INTO test ( id, name ) VALUES ( null, 'some data-2' );" otus-database
    kubectl exec -it mysql-instance-6785949c48-4m9kd -- mysql -potuspassword -e "select * from test;" otus-database

    +----+-------------+
    | id | name        |
    +----+-------------+
    |  1 | some data   |
    |  2 | some data-2 |
    +----+-------------+

###Проверка работоспособности

Удалим mysql-instance:

    kubectl delete mysqls.otus.homework mysql-instance
    mysql.otus.homework "mysql-instance" deleted

Теперь kubectl get pv показывает, что PV для mysql больше нет, а kubectl get jobs.batch показывает:

    NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                               STORAGECLASS   REASON   AGE
    backup-mysql-instance-pv                   1Gi        RWO            Retain           Available                                                               14m
    mysql-instance-pv                          1Gi        RWO            Retain           Available                                                               14m
    pvc-7c7b05ae-46fa-45d3-a3b0-6071a8871b98   1Gi        RWO            Delete           Bound       default/backup-mysql-instance-pvc   standard                14m

    NAME                         COMPLETIONS   DURATION   AGE
    backup-mysql-instance-job    1/1           1s         61s
    restore-mysql-instance-job   0/1           14m        14m


Если Job не выполнилась или выполнилась с ошибкой, то ее нужно удалять в ручную, т к иногда полезно посмотреть логи.

Создадим заново mysql-instance:

    kubectl apply -f deploy/cr.yml 

Ждем некоторое время, затем повторяем не особо удобные команды в Windows на получение имени пода и делаем SELECT запрос>:

    +----+-------------+
    | id | name        |
    +----+-------------+
    |  1 | some data   |
    |  2 | some data-2 |
    +----+-------------+

## Как запустить проект:
    minikube start

Затем применить описанную выше последовательность действий

## Как проверить работоспособность:
Запустить проект и выполнить описанную выше последовательность действий

## PR checklist:
 - [x] Выставлен label с темой домашнего задания
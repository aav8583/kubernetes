# Выполнено ДЗ № 4

 - [x] Основное ДЗ
 - [ ] Задание со *

## В процессе сделано:
###CR, CRD и Validation
Создан CustomResource deploy/cr.yaml.
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

    kubectl apply -f deploy/cr.yaml
    mysql.otus.homework/mysql-instance created

C созданными объектами можно взаимодействовать через kubectl:
    
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
    
    kubectl apply -f deploy/cr.yaml
    mysql.otus.homework/mysql-instance created

Добавим дерективу spec.validation.spec.reqiored в CRD. Удалим CR поле с размером хранилища, запустим cr и crd:

    kubectl apply -f deploy/crd.yml
    customresourcedefinition.apiextensions.k8s.io/mysqls.otus.homework configured
    
    kubectl apply -f deploy/cr.yml
    The MySQL "mysql-instance" is invalid: spec.storage_size: Required value

Данный ответ и ожидался.

### Операторы
Оператор включает в себя CustomResourceDefinition и сustom сontroller.

CRD содержит описание объектов CR.

Контроллер следит за объектами определенного типа, и осуществляет всю логику работы оператора

Остановился на слайде 15

## Как запустить проект:
    minikube start

Затем применить описанную выше последовательность действий

## Как проверить работоспособность:
Запустить проект и т.д.

## PR checklist:
 - [x] Выставлен label с темой домашнего задания
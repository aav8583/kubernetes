# Выполнено ДЗ № 5

 - [x] Основное ДЗ
 - [x] Задание со *

## В процессе сделано:
 Создан кластер
 
    kind create cluster
    export KUBECONFIG="$(kind get kubeconfig-path --name="kind")"
 
 Создан файл minio-statefulset.yaml с содержимым из https://raw.githubusercontent.com/express42/otus-platform-snippets/master/Module-02/Kuberenetes-volumes/minio-statefulset.yaml
 
     apiVersion: apps/v1
     kind: StatefulSet
     metadata:
       # This name uniquely identifies the StatefulSet
       name: minio
     spec:
       serviceName: minio
       replicas: 1
       selector:
         matchLabels:
           app: minio # has to match .spec.template.metadata.labels
       template:
         metadata:
           labels:
             app: minio # has to match .spec.selector.matchLabels
         spec:
           containers:
             - name: minio
               env:
                 - name: MINIO_ACCESS_KEY
                   value: "minio"
                 - name: MINIO_SECRET_KEY
                   value: "minio123"
               image: minio/minio:RELEASE.2019-07-10T00-34-56Z
               args:
                 - server
                 - /data
               ports:
                 - containerPort: 9000
               # These volume mounts are persistent. Each pod in the PetSet
               # gets a volume mounted based on this field.
               volumeMounts:
                 - name: data
                   mountPath: /data
               # Liveness probe detects situations where MinIO server instance
               # is not working properly and needs restart. Kubernetes automatically
               # restarts the pods if liveness checks fail.
               livenessProbe:
                 httpGet:
                   path: /minio/health/live
                   port: 9000
                 initialDelaySeconds: 120
                 periodSeconds: 20
       # These are converted to volume claims by the controller
       # and mounted at the paths mentioned above.
       volumeClaimTemplates:
         - metadata:
             name: data
           spec:
             accessModes:
               - ReadWriteOnce
             resources:
               requests:
                 storage: 10Gi
                 
 Создан файл minio-headlessservice.yaml с содержимым из https://raw.githubusercontent.com/express42/otus-platform-snippets/master/Module-02/Kuberenetes-volumes/minio-headless-service.yaml
 
     apiVersion: v1
     kind: Service
     metadata:
       name: minio
       labels:
         app: minio
     spec:
       clusterIP: None
       ports:
         - port: 9000
           name: minio
       selector:
         app: minio
 
 Headless Service нужен для того, чтобы StatefulSet был доступен изнутри кластера
 
### Задание со *
  Файл секретов minio-secret.yaml
  
    kind: Secret
      apiVersion: v1
      metadata:
        name: minio-secret
      stringData:
        MINIO_ACCESS_KEY: minio
        MINIO_SECRET_KEY: qwerty123
        
  #### Редактирование minio-statefulset.yaml
  Заменяется кусок кода
  
    env:
      - name: MINIO_ACCESS_KEY
        value: "minio"
      - name: MINIO_SECRET_KEY
        value: "minio123"
        
  на
  
    env:
      - name: MINIO_ACCESS_KEY
        valueFrom: 
          secretKeyRef:
            name: minio-secret
            key: MINIO_ACCESS_KEY
      - name: MINIO_SECRET_KEY
        valueFrom: 
          secretKeyRef:
            name: minio-secret
            key: MINIO_SECRET_KEY
            
  получаем
  
      apiVersion: apps/v1
      kind: StatefulSet
      metadata:
        # This name uniquely identifies the StatefulSet
        name: minio
      spec:
        serviceName: minio
        replicas: 1
        selector:
          matchLabels:
            app: minio # has to match .spec.template.metadata.labels
        template:
          metadata:
            labels:
              app: minio # has to match .spec.selector.matchLabels
          spec:
            containers:
              - name: minio
                env:
                  - name: MINIO_ACCESS_KEY
                    valueFrom:
                      secretKeyRef:
                        name: minio-secret
                        key: MINIO_ACCESS_KEY
                  - name: MINIO_SECRET_KEY
                    valueFrom:
                      secretKeyRef:
                        name: minio-secret
                        key: MINIO_SECRET_KEY
                image: minio/minio:RELEASE.2019-07-10T00-34-56Z
                args:
                  - server
                  - /data
                ports:
                  - containerPort: 9000
                # These volume mounts are persistent. Each pod in the PetSet
                # gets a volume mounted based on this field.
                volumeMounts:
                  - name: data
                    mountPath: /data
                # Liveness probe detects situations where MinIO server instance
                # is not working properly and needs restart. Kubernetes automatically
                # restarts the pods if liveness checks fail.
                livenessProbe:
                  httpGet:
                    path: /minio/health/live
                    port: 9000
                  initialDelaySeconds: 120
                  periodSeconds: 20
        # These are converted to volume claims by the controller
        # and mounted at the paths mentioned above.
        volumeClaimTemplates:
          - metadata:
              name: data
            spec:
              accessModes:
                - ReadWriteOnce
              resources:
                requests:
                  storage: 10Gi

## Как запустить проект:
    kind create cluster
    export KUBECONFIG="$(kind get kubeconfig-path --name="kind")"
    
    kubectl apply -f .\minio-statefulset.yaml
   
    
## Как проверить работоспособность:

### Основное задание
Используя консольные команды


     kubectl get statefulsets
     NAME    READY   AGE
     minio   1/1     115m
     
     kubectl get pods
     NAME      READY   STATUS    RESTARTS   AGE
     minio-0   1/1     Running   0          115m
     
     kubectl get pvc
     NAME           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
     data-minio-0   Bound    pvc-464c7130-adb6-4ba8-b761-b657849f2a4b   10Gi       RWO            standard       115m
     
     kubectl get pv
     NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                  STORAGECLASS   REASON   AGE
     pvc-464c7130-adb6-4ba8-b761-b657849f2a4b   10Gi       RWO            Delete           Bound    default/data-minio-0   standard                116m
     
     kubectl describe pv pvc-464c7130-adb6-4ba8-b761-b657849f2a4b
     Name:            pvc-464c7130-adb6-4ba8-b761-b657849f2a4b
     Labels:          <none>
     Annotations:     hostPathProvisionerIdentity: eb7ca162-070b-4312-871f-f384d64034d2
                      pv.kubernetes.io/provisioned-by: k8s.io/minikube-hostpath
     Finalizers:      [kubernetes.io/pv-protection]
     StorageClass:    standard
     Status:          Bound
     Claim:           default/data-minio-0
     Reclaim Policy:  Delete
     Access Modes:    RWO
     VolumeMode:      Filesystem
     Capacity:        10Gi
     Node Affinity:   <none>
     Message:
     Source:
         Type:          HostPath (bare host directory volume)
         Path:          /tmp/hostpath-provisioner/default/data-minio-0
         HostPathType:
     Events:            <none>
     
### Задание со *
  Применяем:     
       
     kubectl apply -f minio-secret.yaml
     secret/minio-secret created
       
     kubectl apply -f minio-statefulset.yaml
     statefulset.apps/minio configured
     
## PR checklist:
 - [x] Выставлен label с темой домашнего задания
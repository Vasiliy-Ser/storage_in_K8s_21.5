# Домашнее задание к занятию 21.5 «Хранение в K8s» - Падеев Василий

### Примерное время выполнения задания — 180 минут  

### Цель задания  

Научиться работать с хранилищами в тестовой среде Kubernetes:   
- обеспечить обмен файлами между контейнерами пода;  
- создавать **PersistentVolume** (PV) и использовать его в подах через **PersistentVolumeClaim** (PVC);  
- объявлять свой **StorageClass** (SC) и монтировать его в под через **PVC**.  

Это задание поможет вам освоить базовые принципы взаимодействия с хранилищами в Kubernetes — одного из ключевых навыков для работы с кластерами. На практике Volume, PV, PVC используются для хранения данных независимо от пода, обмена данными между подами и контейнерами внутри пода. Понимание этих механизмов поможет вам упростить проектирование слоя данных для приложений, разворачиваемых в кластере k8s.

------

## **Подготовка**  
### **Чеклист готовности**  

1. Установленное K8s-решение (допустим, MicroK8S).  
2. Установленный локальный kubectl.  
3. Редактор YAML-файлов с подключенным GitHub-репозиторием.    

------

### Инструменты, которые пригодятся для выполнения задания  

1. [Инструкция](https://microk8s.io/docs/getting-started) по установке MicroK8S.  
2. [Инструкция](https://minikube.sigs.k8s.io/docs/start/?arch=%2Fwindows%2Fx86-64%2Fstable%2F.exe+download) по установке Minikube.  
3. [Инструкция](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/) по установке kubectl.  
4. [Инструкция](https://marketplace.visualstudio.com/items?itemName=ms-kubernetes-tools.vscode-kubernetes-tools) по установке VS Code  

### Дополнительные материалы, которые пригодятся для выполнения задания  
1. [Описание Volumes](https://kubernetes.io/docs/concepts/storage/volumes/).  
2. [Описание Ephemeral Volumes](https://kubernetes.io/docs/concepts/storage/volumes/).  
3. [Описание PersistentVolume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/).  
4. [Описание PersistentVolumeClaim](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims).  
5. [Описание StorageClass](https://kubernetes.io/docs/concepts/storage/storage-classes/).  
6. [Описание Multitool](https://github.com/wbitt/Network-MultiTool).  


------

## Задание 1. Volume: обмен данными между контейнерами в поде  
### Задача  

Создать Deployment приложения, состоящего из двух контейнеров, обменивающихся данными.  

### Шаги выполнения  
1. Создать Deployment приложения, состоящего из контейнеров busybox и multitool.  
2. Настроить busybox на запись данных каждые 5 секунд в некий файл в общей директории.  
3. Обеспечить возможность чтения файла контейнером multitool.  


### Что сдать на проверку  
- Манифесты:  
  - `containers-data-exchange.yaml`  
- Скриншоты:  
  - описание пода с контейнерами (`kubectl describe pods data-exchange`)  
  - вывод команды чтения файла (`tail -f <имя общего файла>`)  


### Решение:  

Создал ВМ с помощью terraform га которой развернул MicroK8S, управление будет происходить с хостовой машины.   
Создал [Deployment](https://github.com/Vasiliy-Ser/storage_in_K8s_21.5/blob/893927322d503993d4490429aad85be0ea6eb633/src/containers-data-exchange_1.yaml). Pod содержит 2 контейнера:  
- busybox: записывает данные в файл каждые 5 секунд.  
- multitool использует tail -f, чтобы в реальном времени отображать содержимое этого файла.  
Контейнеры обмениваются данными через общий том (shared volume)  
Использую emptyDir том который создается при запуске Pod и существует пока Pod работает.  

![answer1](https://github.com/Vasiliy-Ser/storage_in_K8s_21.5/blob/893927322d503993d4490429aad85be0ea6eb633/png/1.png)  
![answer2](https://github.com/Vasiliy-Ser/storage_in_K8s_21.5/blob/893927322d503993d4490429aad85be0ea6eb633/png/2.png)  
![answer3](https://github.com/Vasiliy-Ser/storage_in_K8s_21.5/blob/893927322d503993d4490429aad85be0ea6eb633/png/3.png)   
![answer4](https://github.com/Vasiliy-Ser/storage_in_K8s_21.5/blob/893927322d503993d4490429aad85be0ea6eb633/png/4.png)  


------

## Задание 2. PV, PVC  
### Задача  
Создать Deployment приложения, использующего локальный PV, созданный вручную.  

### Шаги выполнения  
1. Создать Deployment приложения, состоящего из контейнеров busybox и multitool, использующего созданный ранее PVC  
2. Создать PV и PVC для подключения папки на локальной ноде, которая будет использована в поде.  
3. Продемонстрировать, что контейнер multitool может читать данные из файла в смонтированной директории, в который busybox записывает данные каждые 5 секунд.   
4. Удалить Deployment и PVC. Продемонстрировать, что после этого произошло с PV. Пояснить, почему. (Используйте команду `kubectl describe pv`).  
5. Продемонстрировать, что файл сохранился на локальном диске ноды. Удалить PV.  Продемонстрировать, что произошло с файлом после удаления PV. Пояснить, почему.  


### Что сдать на проверку  
- Манифесты:  
  - `pv-pvc.yaml`  
- Скриншоты:  
  - каждый шаг выполнения задания, начиная с шага 2.  
- Описания:  
  - объяснение наблюдаемого поведения ресурсов в двух последних шагах.  


### Решение:  

Создал [Deployment](https://github.com/Vasiliy-Ser/storage_in_K8s_21.5/blob/893927322d503993d4490429aad85be0ea6eb633/src/containers-data-exchange_2.yaml) и [pv-pvc.yaml](https://github.com/Vasiliy-Ser/storage_in_K8s_21.5/blob/893927322d503993d4490429aad85be0ea6eb633/src/pv-pvc.yaml). В котором указываем:  
- capacity: размер диска.  
- volumeMode: Filesystem: обычная файловая система.  
- accessModes: ReadWriteMany: монтировать можно в нескольких контейнерах одновременно.  
- local.path: путь на хосте.  
- nodeAffinity: ограничение — PV может использоваться только на указанной ноде.  

Чтобы использовать локальный том, необходимо заранее создать каталог  
![answer5](https://github.com/Vasiliy-Ser/storage_in_K8s_21.5/blob/893927322d503993d4490429aad85be0ea6eb633/png/6.png)  
![answer6](https://github.com/Vasiliy-Ser/storage_in_K8s_21.5/blob/893927322d503993d4490429aad85be0ea6eb633/png/5.png)  
![answer7](https://github.com/Vasiliy-Ser/storage_in_K8s_21.5/blob/893927322d503993d4490429aad85be0ea6eb633/png/7.png)  
![answer8](https://github.com/Vasiliy-Ser/storage_in_K8s_21.5/blob/893927322d503993d4490429aad85be0ea6eb633/png/8.png)  
 После удаления PVC Kubernetes не удаляет данные — политика Retain. Файл data.txt всё ещё существует на диске.  
![answer9](https://github.com/Vasiliy-Ser/storage_in_K8s_21.5/blob/893927322d503993d4490429aad85be0ea6eb633/png/9.png)  
![answer10](https://github.com/Vasiliy-Ser/storage_in_K8s_21.5/blob/893927322d503993d4490429aad85be0ea6eb633/png/10.png)  
После удаления PV файл останется на диске, потому что Kubernetes не управляет физическим содержимым для Retain.  
![answer11](https://github.com/Vasiliy-Ser/storage_in_K8s_21.5/blob/893927322d503993d4490429aad85be0ea6eb633/png/11.png)  


------

## Задание 3. StorageClass  
### Задача  
Создать Deployment приложения, использующего PVC, созданный на основе StorageClass.  

### Шаги выполнения

1. Создать Deployment приложения, состоящего из контейнеров busybox и multitool, использующего созданный ранее PVC.  
2. Создать SC и PVC для подключения папки на локальной ноде, которая будет использована в поде.  
3. Продемонстрировать, что контейнер multitool может читать данные из файла в смонтированной директории, в который busybox записывает данные каждые 5 секунд.  

### Что сдать на проверку  
- Манифесты:  
  - `sc.yaml`  
- Скриншоты:  
  - каждый шаг выполнения задания, начиная с шага 2  


### Решение:

Создал [Deployment](https://github.com/Vasiliy-Ser/storage_in_K8s_21.5/blob/893927322d503993d4490429aad85be0ea6eb633/src/sc.yaml) в который включил SC, PV и PVC.  
![answer12](https://github.com/Vasiliy-Ser/storage_in_K8s_21.5/blob/893927322d503993d4490429aad85be0ea6eb633/png/12.png)  
Очистил дирректорию /tmp/data-pv/  
![answer13](https://github.com/Vasiliy-Ser/storage_in_K8s_21.5/blob/893927322d503993d4490429aad85be0ea6eb633/png/13.png)  
Проверим, что контейнер multitool может читать данные из файла в смонтированной директории, в который busybox записывает данные каждые 5 секунд.  
![answer14](https://github.com/Vasiliy-Ser/storage_in_K8s_21.5/blob/893927322d503993d4490429aad85be0ea6eb633/png/14.png)   
![answer15](https://github.com/Vasiliy-Ser/storage_in_K8s_21.5/blob/893927322d503993d4490429aad85be0ea6eb633/png/15.png)  


---


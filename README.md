# Процесс запуска Minikube с Jenkins и создания Jenkins Job для Nginx

## Описание

В данной инструкции рассматривается процесс развёртывания Minikube на вашем компьютере, развёртывания Jenkins в Minikube, а также процесс создания Jenkins Job для установки Helm чарта, задачей которого является установка последней версии Nginx, который затем будет доступен на ноде K8s по порту 32080.

## Шаги

### Установка Docker Desktop

  1. Перейдите на официальный сайт Docker по адресу [Docker](https://www.docker.com/products/docker-desktop/).
  2. Скачайте версию Docker Desktop для вашей операционной системы.
  3. Следуйте инструкциям установки на сайте для вашей ОС.

После установки Docker Desktop убедитесь, что он запущен, и переключите Docker на использование Linux контейнеров, если вы находитесь в Windows среде.

### Установка Kubectl

  1. Скачайте бинарный файл kubectl для Windows. Для этого откройте PowerShell и выполните следующую команду для скачивания последней версии kubectl:

    
    curl -LO https://dl.k8s.io/release/`curl -LS https://dl.k8s.io/release/stable.txt`/bin/linux/amd64/kubectl
    

  2. Добавьте kubectl в переменную среды PATH
  3. Проверьте установку следующей командой:

    
    kubectl version --client
    

### Установка Minikube

  1. Откройте PowerShell и выполните следующую команду для скачивания последней версии Minikube: 

    
    curl -Lo minikube.exe https://storage.googleapis.com/minikube/releases/latest/minikube-windows-amd64.exe=
    
  2. Добавьте Minikube в переменную среды PATH

### Запуск Minikube
  1. Чтобы убедиться, что Minikube был установлен корректно, выполните следующую команду, которая запускает локальный кластер Kubernetes:

    
    minikube start --driver=docker
    
  2. Чтобы проверить, успешно ли запущен Minikube, можно использовать команду:

    
    minikube status
    

### Установка Jenkins

  1. Скачайте архив, разархивируйте его, откройте папку jenkins через консоль
  2. Создайте отдельное пространство имён для Jenkins:
  
    
    kubectl create namespace devops-tools
    
   
  3. Создайте serviceAccount в Kubernetes:
  
    
    kubectl apply -f serviceAccount.yaml
    
   
  4. Создайте volume в Kubernetes:
  
    
    kubectl create -f volume.yaml
    

  5. Создайте deployment в Kubernetes:
  
    
    kubectl apply -f deployment.yaml
    


  6. Создайте service в Kubernetes:

    
    kubectl apply -f service.yaml
    

  7. Получите внешний IP-адрес и порт сервиса Jenkins:
  
    
    minikube service jenkins-service -n devops-tools --url
    

  8. Откройте Jenkins веб-интерфейс по адресу `http://<minikube-ip>:<port>`, используя полученный IP-адрес и порт вместо <minikube-ip> и <port> соответственно:
- Чтобы узнать `<jenkins-pod-name>` введите команду:
  
    
    kubectl get -n devops-tools Pods
    

  9. Получите пароль, введя команду для получения логов, указанную ниже. В таком случае пароль будет находиться внизу:
  
    
    kubectl logs <jenkins-pod-name> --namespace=devops-tools
    

### Настройка Jenkins

  1. Скопируйте пароль и введите его на странице входа в Jenkins.
  2 Выберите "Install suggested plugins", чтобы установить рекомендуемые плагины.
  3 Дождитесь завершения установки плагинов. После этого создайте пользователя администратора Jenkins.
  4 Войдите в систему в качестве администратора Jenkins.

### Установка плагинов Jenkins
    
  1. Перейдите в раздел "Manage Jenkins" (Настроить Jenkins).
  2. В "System Configuration" выберите "Plugins" (Плагины).
  3. В разделе "Available plugins" (Доступные плагины) найдите нужный плагин и установите его. Вам потребуется следующие плагины:
  

### Создание Jenkins Job

  1. Создайте новую Jinkins Job, выбрав тип Pipeline, введите желаемое Название.
  2. Пролистайте в самый низ и затем введите в соответствующее поле следующий скрипт:
> **ВНИМАНИЕ!!!** В stage 'Apply HELM Chart' в поле serverUrl вместо `<yourServerIP>` вам необходимо использовать IP вашей ноды кластера Kubernetes!
> **Примечание:** В случае необходимости вы можете изменить скрипт, скачав актуальную версию HELM и kubectl 

  ```bash
  pipeline {
      agent any
      stages{
          stage('Install HELM') {
              steps {
                  sh 'pwd'
                  sh 'curl -fsSL -o helm.tar.gz https://get.helm.sh/helm-v3.7.2-linux-amd64.tar.gz'
                  sh 'tar -zxvf helm.tar.gz '
                  sh 'chmod +x linux-amd64/helm'
                  sh './linux-amd64/helm version'
              }
          }
          stage('Pull Repo') {
              steps {
                  git 'https://github.com/DaniilBo/test-tasks.git'
                  sh 'pwd'
                  sh 'ls -l'
              }
          }
          stage('Install Kubectl'){
              steps {
                  sh 'curl -LO "https://storage.googleapis.com/kubernetes-release/release/v1.20.5/bin/linux/amd64/kubectl"'  
                  sh 'chmod u+x ./kubectl'
              }
          }
          stage('Apply HELM Chart') {
              steps {
                  withKubeConfig([serverUrl: '<yourServerIP>:8443']) {
                      sh './linux-amd64/helm upgrade --install nginx-chart /var/jenkins_home/workspace/test-task/nginx-chart -n devops-tools'
                      sh './kubectl get pods -n devops-tools'
                  }
              } 
          } 
      }
  }          
  ```

### Проверка доступности Nginx

  1. Установите IP адрес нода, используя следующую команду:

    
    minikube ip
    

  2. В браузере откройте следующую ссылку, заменив minikube-ip на ip вашей ноды:

    
    http://<minikube-ip>:32080
    
    

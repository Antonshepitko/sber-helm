# Процесс запуска Minikube с Jenkins и создания Jenkins Job для Nginx

## Описание

В данной инструкции рассматривается процесс развёртывания Minikube на вашем компьютере, развёртывания Jenkins в Minikube, а также процесс создания Jenkins Job для установки Helm чарта, задачей которого является установка последней версии Nginx, который затем будет доступен на ноде K8s по порту 32080.

## Шаги

### Установка Docker Desktop

Перед установкой Minikube убедитесь, что на вашем компьютере установлен Docker Desktop. Для установки Docker Desktop:

- Перейдите на официальный сайт Docker по адресу [Docker Hub](https://hub.docker.com/).
- Скачайте версию Docker Desktop для вашей операционной системы.
- Следуйте инструкциям установки на сайте для вашей ОС.

После установки Docker Desktop убедитесь, что он запущен, и переключите Docker на использование Linux контейнеров, если вы находитесь в Windows среде.

### Установка Kubectl
  1. Скачайте бинарный файл kubectl для Windows. Для этого откройте PowerShell и выполните следующую команду для скачивания последней версии kubectl:

    ```
    curl -LO https://dl.k8s.io/release/`curl -LS https://dl.k8s.io/release/stable.txt`/bin/linux/amd64/kubectl
    ```

  2. Добавьте kubectl в переменную среды PATH
  3. Проверьте установку следующей командой:

    ```
    kubectl version --client
    ```

### Установка Minikube

  1. Перейдите по ссылке на официальный сайт Minikube и скачайте последнюю версию для Windows: [Minikube Releases](https://github.com/kubernetes/minikube/releases).
  2. Скачанный файл будет в формате .exe. Дважды кликните по файлу, чтобы запустить инсталлятор.

### Запуск Minikube


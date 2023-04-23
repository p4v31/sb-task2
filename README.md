# Введение

Данный плейбук установит все с 0 и от пользователя не нужно никаких дополнительных действий, кроме как следовать инструкции по запуску. Проверял все несколько раз на чистой Ubuntu машине. 

Прежде чем написать плейбук, я попробовал все вручную проделать, что и делаю я обычно перед тем, как что-то автоматизировать. К сожалению, из 3-х попыток запустить все идельно без скрипта у меня была проблема коннекта к БД. Если Вы подскажите, как это исправить, то я буду очень рад!(тг написал в ответе).

# Как запустить?

Обновим локальный кэш пакетов операционной системы Ubuntu, чтобы убедиться, что установленные пакеты являются последними версиями.
```
sudo apt update
```

Устанавим пакет Ansible, который является программным инструментом для автоматизации конфигурации и управления компьютерными системами.
```
sudo apt install ansible
```

Создадим новую группу пользователей с именем "docker".
```
sudo groupadd docker
```

Добавим текущего пользователя в группу "docker", что позволит пользователю использовать команды Docker без прав суперпользователя. После выполнения этой команды нужны перелогиниться, чтобы изменения встпуили в силу.
```
sudo usermod -aG docker $USER
```

Проверим, добавилась ли группа docker
```
groups
```

Запустим плейбук Ansible с именем "test.yaml" и запросим пароль для привилегированного доступа (-K)
```
ansible-playbook test.yaml -K
```
<img width="468" alt="image" src="https://user-images.githubusercontent.com/90404785/233844778-ccc4dcaa-5a68-4eba-884e-e9a3004650b2.png">

# Результат

Откроем в браузере http://localhost:3001 и увидим нужнцю нам страницу.

<img width="468" alt="image" src="https://user-images.githubusercontent.com/90404785/233844806-dd0b7a93-77a2-4bb3-9a76-9e20697450e3.png">

# Описание плейбука подробное

Вот подробное описание каждой таски в данном плейбуке Ansible:

1. **Add Docker GPG key**: Эта таска добавляет GPG-ключ репозитория Docker, чтобы система могла проверять подпись пакетов перед установкой. Это делается с использованием модуля `ansible.builtin.apt_key`.

2. **Add Docker repository**: Таска добавляет репозиторий Docker в список репозиториев системы. Это позволяет системе получать обновления и устанавливать Docker из официального репозитория. Это делается с использованием модуля `ansible.builtin.apt_repository`.

3. **Install Docker**: Таска устанавливает последнюю версию Docker на целевом хосте. Это делается с использованием модуля `ansible.builtin.apt`.

4. **Install dependencies**: Таска устанавливает зависимости, необходимые для работы Docker, crictl и kubectl. Зависимости устанавливаются с использованием модуля `apt`.

5. **Install crictl**: Таска загружает архив crictl (инструмент для взаимодействия с CRI) из указанного URL и сохраняет его в /tmp/crictl.tar.gz на целевом хосте. Это делается с использованием модуля `get_url`.

6. **Extract crictl**: Таска извлекает crictl из архива в /tmp/crictl.tar.gz и устанавливает его в /usr/local/bin на целевом хосте. Это делается с использованием модуля `unarchive`.

7. **Set fs.protected_regular to 0**: Таска устанавливает значение параметра ядра `fs.protected_regular` равным 0. Это делается с использованием модуля `ansible.builtin.sysctl`.

8. **Install minikube**: Таска загружает исполняемый файл minikube и устанавливает его в /usr/local/bin на целевом хосте. Это делается с использованием модуля `get_url`.

9. **Delete existing minikube cluster**: Таска удаляет существующий кластер minikube, если он есть. Это делается с использованием модуля `command`.

10. **Start minikube**: Таска запускает minikube с использованием драйвера Docker. Это делается с использованием модуля `command`.

11. **Get latest kubectl version**: Таска определяет последнюю версию kubectl, запрашивая данные с удаленного сервера. Это делается с использованием модуля `ansible.builtin.shell`.

12. **Clone kubernetes/examples repository**: Таска клонирует репозиторий kubernetes/examples на целевой хост. Это делается с использованием модуля `ansible.builtin.git`.

13. **Create Kubernetes resources if they don't
```
- name: Safeboard 2nd task
  hosts: localhost
  become: yes
  tasks:
    - name: Add Docker GPG key
      ansible.builtin.apt_key:  # Модуль Ansible для работы с GPG-ключами
        url: https://download.docker.com/linux/ubuntu/gpg
        state: present  # Убедиться, что ключ присутствует в системе

    - name: Add Docker repository
      ansible.builtin.apt_repository:  # Модуль Ansible для работы с репозиториями
        repo: "deb [arch=amd64] https://download.docker.com/linux/ubuntu {{ ansible_lsb.codename }} stable"
        state: present  # Убедиться, что репозиторий присутствует в системе

    - name: Install Docker
      ansible.builtin.apt:  # Модуль Ansible для работы с пакетами
        name: docker-ce
        update_cache: yes  # Обновить кэш репозитория перед установкой
        state: latest  # Установить последнюю версию пакета

    - name: Install dependencies
      apt:
        name: "{{ packages }}"  # Устанавливает список пакетов, определенных в переменной
        update_cache: yes
      vars:
        packages:
          - apt-transport-https
          - ca-certificates
          - conntrack
          - curl

    - name: Install crictl
      get_url:  # Модуль Ansible для загрузки файлов по URL
        url: https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.23.0/crictl-v1.23.0-linux-amd64.tar.gz
        dest: /tmp/crictl.tar.gz

    - name: Extract crictl
      unarchive:  # Модуль Ansible для работы с архивами
        src: /tmp/crictl.tar.gz
        dest: /usr/local/bin  # Распаковать архив в указанную директорию
        remote_src: yes  # Использовать архив на удаленном хосте

    - name: Set fs.protected_regular to 0
      ansible.builtin.sysctl:  # Модуль Ansible для работы с sysctl
        name: fs.protected_regular
        value: "0"  # Значение параметра
        sysctl_set: yes  # Применить значение сразу
        state: present  # Убедиться, что параметр задан
        reload: yes  # Перезагрузить конфигурацию sysctl

    - name: Install minikube
      get_url:
        url: https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
        dest: /usr/local/bin/minikube
        mode: 0755  # Задать права доступа к файлу

    - name: Delete existing minikube cluster
      command: minikube delete  # Удаляет текущий кластер minikube, если он существует
      ignore_errors: yes  # Продолжить, даже если ком
      - name: Start minikube
      command: minikube start --driver=docker  # Запуск minikube с использованием драйвера Docker
      become: no  # Запуск команды без повышения привилегий

    - name: Get latest kubectl version
      ansible.builtin.shell: >  # Модуль Ansible для выполнения команд оболочки
        curl -s
        https://storage.googleapis.com/kubernetes-release/release/stable.txt
      args:
        executable: /bin/bash
      register: kubectl_latest_version  # Регистрирует результат команды в переменную
      changed_when: false  # Не считать выполнение команды изменением состояния

    - name: Install apt-transport-https
      apt:
        name: apt-transport-https
        state: present
      become: yes

    - name: Clone kubernetes/examples repository
      ansible.builtin.git:  # Модуль Ansible для работы с Git
        repo: 'https://github.com/kubernetes/examples.git'
        dest: examples  # Место назначения для склонированных данных

    - name: Create Kubernetes resources if they don't exist
      ansible.builtin.command:  # Модуль Ansible для выполнения команд
        cmd: 'minikube kubectl -- apply -f {{ item }}'  # Применяет каждый указанный файл конфигурации
      loop:  # Цикл по списку файлов конфигурации
        - redis-master-controller.yaml
        - redis-master-service.yaml
        - redis-replica-controller.yaml
        - redis-replica-service.yaml
        - guestbook-controller.yaml
        - guestbook-service.yaml
      args:
        chdir: examples/guestbook-go  # Указывает директорию, из которой выполнять команду
      become: no

    - name: Create keep_guestbook_running.sh script
      ansible.builtin.copy:  # Модуль Ansible для копирования файлов
        content: |
          #!/bin/bash

          while true; do
            minikube kubectl port-forward service/guestbook 3001:3000
            sleep 1
          done
        dest: /usr/local/bin/keep_guestbook_running.sh  # Место назначения для скопированных данных
        mode: 0755  # Устанавливает права доступа к файлу

    - name: Start port-forwarding in the background
      ansible.builtin.shell: "nohup /usr/local/bin/keep_guestbook_running.sh >/dev/null 2>&1 &"  # Запускает скрипт в фоновом режиме
      become: no

```

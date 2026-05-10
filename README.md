# Дипломный практикум в Yandex.Cloud
## ` Дмитрий Климов `
# Дипломный проект: "Развертывание отказоустойчивой IT-инфраструктуры в облаке с Kubernetes и полным стеком Observability"

## 🚀 Обзор проекта

Данный проект демонстрирует построение современной, масштабируемой и отказоустойчивой IT-инфраструктуры в облачной среде `Yandex Cloud`. Проект охватывает все этапы: от автоматизированного создания базовой инфраструктуры и развертывания кластера `Kubernetes` до внедрения комплексной системы мониторинга и централизованного сбора логов. Все компоненты управляются с помощью кода `(Infrastructure as Code, Configuration as Code)` и автоматизированных `CI/CD` пайплайнов.

---

## 🔗 Репозитории проекта

*   **Infrastructure (Terraform):** [diploma-infrastructure](https://github.com/Dmitriy-py/diploma-infrastructure)
    *   Автоматическое создание облачных ресурсов (VPC, Subnets, VM, NAT Gateway) в Yandex Cloud.
    *   CI/CD пайплайн для автоматического применения изменений инфраструктуры.
*   **Configuration (Kubespray):** [diploma-ansible-kubespray](https://github.com/Dmitriy-py/diploma-ansible-kubespray)
    *   Развертывание кластера Kubernetes v1.35.4 с использованием Ansible и Kubespray.
    *   Настроенные конфигурации для Yandex Cloud, включая Calico CNI и Nginx Ingress Controller.
*   **Application & CI/CD:** [diploma-app](https://github.com/Dmitriy-py/diploma-app)
    *   Пример простого веб-приложения.
    *   `CI/CD` пайплайн для автоматической сборки Docker-образов и их развертывания в кластер Kubernetes.

---

## 1. Инфраструктура как код (Terraform)

### 1.1. Описание
Terraform используется для декларативного управления облачными ресурсами. Были созданы:
*   Виртуальное облако `(VPC)`
*   Подсети в трех зонах доступности
*   NAT Gateway для исходящего доступа из подсетей
*   Три виртуальные машины: 1 `Master-Node` `(control-plane)` и 2 `Worker-Nodes`.
*   Настроен SSH-доступ.

---

### 1.2. Ключевые команды
```bash
# Инициализация Terraform
terraform init

# Проверка плана изменений
terraform plan

# Применение изменений (создание ресурсов)
terraform apply -auto-approve
```
---

<img width="1920" height="1080" alt="Снимок экрана (3608)" src="https://github.com/user-attachments/assets/570f4a4d-db7e-4766-a2fe-d84d01804e07" />

---

<img width="1920" height="1080" alt="Снимок экрана (3610)" src="https://github.com/user-attachments/assets/1f428b44-0091-485a-829d-fdf1e946e88f" />

---

<img width="1920" height="1080" alt="Снимок экрана (3609)" src="https://github.com/user-attachments/assets/0fa193ff-2846-4a62-acd1-2326380ca662" />

---

<img width="1920" height="1080" alt="Снимок экрана (3624)" src="https://github.com/user-attachments/assets/7426d827-0b97-433b-aa8d-fc48e6dba949" />

---

## 2. Развертывание кластера `Kubernetes` `(Kubespray)`

### 2.1. Описание

Kubespray, основанный на `Ansible`, автоматизирует процесс установки и настройки Kubernetes. Был выбран `Kubernetes` версии v1.35.4.

---

### 2.2.  Процесс развертывания

   1. Клонирование репозитория `Kubespray`.
   2. Настройка файла инвентаря `(inventory/mycluster/hosts.yaml)` с IP-адресами, полученными от `Terraform`.
   3. Настройка конфигурационных файлов `(inventory/mycluster/group_vars/)` для выбора `containerd` как рантайма и `calico` в качестве `CNI`.
   4. Активация виртуального окружения `Python`.

---

### 2.3. Ключевые команды
```bash
git clone https://github.com/Dmitriy-py/diploma-ansible-kubespray.git
git clone https://github.com/Dmitriy-py/diploma-infrastructure.git
cd diploma-ansible-kubespray
source venv/bin/activate
ansible-playbook -i inventory/mycluster/hosts.yaml cluster.yml -b -v
```

---

<img width="1920" height="1080" alt="Снимок экрана (3554)" src="https://github.com/user-attachments/assets/1f7bfdfe-c20f-4abc-84c2-ee29c613fb1c" />

---

<img width="1920" height="1080" alt="Снимок экрана (3516)" src="https://github.com/user-attachments/assets/2dcb0908-9198-4297-93dd-2c80066a1059" />

---

## 3. Сетевая связность и Ingress Controller

### 3.1. Описание

Для обеспечения внешнего доступа к сервисам кластера развернут `Nginx Ingress Controller`.
   * Выбор: Использован тип сервиса `NodePort` (порт 30080) для прямого доступа к `Ingress Controller`.
   * DNS: Для удобства использован сервис nip.io для привязки доменных имен к IP-адресам.

---

### 3.2. Установка Ingress Controller
```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx --create-namespace \
  --set controller.service.type=NodePort \
  --set controller.service.nodePorts.http=30080
```

---

### 3.3. Манифест `Ingress` для `Grafana`

 Данный манифест направляет трафик на сервис `Grafana`.
 ```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grafana-ingress
  namespace: monitoring
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  - host: grafana.93.77.180.95.nip.io
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: prometheus-grafana
            port:
              number: 80
```
---

## 4. `CI/CD` для приложения

### 4.1. Описание

Разработан пайплайн непрерывной интеграции и доставки `(CI/CD)` для приложения с использованием `GitHub Actions`.

---

### 4.2. Репозиторий приложения

  * [diploma-app](https://github.com/Dmitriy-py/diploma-app)

### 4.3. Workflow `CI/CD (.github/workflows/deploy.yml)`

---

Пайплайн автоматизирует следующие шаги:
  1. Сборка Docker-образа: Создание образа приложения.
  2. Push Образа: Отправка собранного образа в `Yandex Container Registry`.
  3. Deploy: Применение манифестов `Kubernetes (Deployment, Service, Ingress)` для развертывания новой версии приложения в кластер.

---

<img width="1920" height="1080" alt="Снимок экрана (3612)" src="https://github.com/user-attachments/assets/d685577c-d0f0-4588-85cb-abe729145a5b" />

---

<img width="1920" height="1080" alt="Снимок экрана (3625)" src="https://github.com/user-attachments/assets/6eef1640-6898-4546-b6fb-70d30c3273bb" />

---

<img width="1920" height="1080" alt="Снимок экрана (3560)" src="https://github.com/user-attachments/assets/25ad0376-7914-4846-b3d5-8a2b00392c9c" />

---

## `Yandex Container Registry`
```bash
docker pull cr.yandex/crpil6qbhst9ijov98qr/nginx-app:latest
docker run -d -p 8080:80 --name my-nginx-app cr.yandex/crpil6qbhst9ijov98qr/nginx-app:latest
```
---

<img width="1920" height="1080" alt="Снимок экрана (3623)" src="https://github.com/user-attachments/assets/2daefe60-79cb-47e0-b975-53ba3007da67" />

---

## 5. Система мониторинга `(Prometheus & Grafana)`

### 5.1. Описание

Для сбора и визуализации метрик состояния кластера и узлов используется связка `Prometheus` и `Grafana`.

---

### 5.2. Установка

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring --create-namespace
```

---

### 5.3. Настройка `Grafana`

1. Добавление Prometheus как источника данных (Data Source) по адресу http://prometheus.monitoring:9090 (или                       http://prometheus.monitoring.svc.cluster.local:9090).
2. Импорт или настройка дашборда Node Exporter Full.
3. Выбор источника данных Prometheus, Job node-exporter и нужного узла (например, master) в фильтрах дашборда.

---

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/dbadf7fe-c09a-4c09-9771-bca0b33e262b" />

---

<img width="1920" height="1080" alt="Снимок экрана (3627)" src="https://github.com/user-attachments/assets/e7f76954-5439-486c-8e81-43a3315b914f" />

---

<img width="1920" height="1080" alt="Снимок экрана (3628)" src="https://github.com/user-attachments/assets/2b347bac-d5fa-4a07-adc6-87ac65f28c05" />

---

## 6. Система логирования `(Loki & Promtail)`

### 6.1. Описание

Для централизованного сбора и анализа логов всех контейнеров в кластере используется связка `Loki` и `Promtail`.

### 6.2. Установка

---

`Loki` устанавливается с помощью `Helm`, в режиме, где логи временно хранятся в оперативной памяти (без `Persistent Volumes`, так как в кластере не был настроен `StorageClass`).

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install loki grafana/loki-stack \
  --namespace monitoring \
  --set loki.persistence.enabled=false \
  --set promtail.enabled=true
```

---

### 6.3. Настройка `Grafana`

   1. Добавление `Loki` как источника данных `(Data Source)` по адресу http://loki.monitoring:3100.
   2. Переход в раздел Explore, выбор источника `Loki`.
   3. Фильтрация логов по неймспейсам (например, `namespace="monitoring"`) и запуск запроса.

---

<img width="1920" height="1080" alt="Снимок экрана (3479)" src="https://github.com/user-attachments/assets/247ccb69-d516-4840-8b15-3c6b94b174b1" />

---

<img width="1920" height="1080" alt="Снимок экрана (3478)" src="https://github.com/user-attachments/assets/a3e59c03-678f-42ce-818a-f37d29b1ea3d" />

---

<img width="1920" height="1080" alt="Снимок экрана (3481)" src="https://github.com/user-attachments/assets/90ccbc72-23d5-4d4d-bc3f-27f6750890fa" />

---

<img width="1920" height="1080" alt="Снимок экрана (3482)" src="https://github.com/user-attachments/assets/059a61bf-5848-434f-911b-64d84d66f0eb" />

---


## 7. Заключение

В рамках дипломного проекта была успешно построена современная, автоматизированная IT-инфраструктура с использованием кластера Kubernetes. Внедрен полный стек `Observability (Prometheus, Grafana, Loki)` обеспечивающий мониторинг состояния кластера и приложений, а также централизованный сбор и анализ логов. Весь цикл жизненного цикла инфраструктуры и приложений управляется с помощью кода `(IaC, Configuration as Code)` и автоматизированных `CI/CD` пайплайнов, что соответствует лучшим практикам `DevOps`.


































# Дипломный практикум в Yandex.Cloud
## ` Дмитрий Климов `
# Дипломный проект: "Развертывание отказоустойчивой IT-инфраструктуры в облаке с Kubernetes и полным стеком Observability"

## 🚀 Обзор проекта

Данный проект демонстрирует построение современной, масштабируемой и отказоустойчивой IT-инфраструктуры в облачной среде Yandex Cloud. Проект охватывает все этапы: от автоматизированного создания базовой инфраструктуры и развертывания кластера Kubernetes до внедрения комплексной системы мониторинга и централизованного сбора логов. Все компоненты управляются с помощью кода (Infrastructure as Code, Configuration as Code) и автоматизированных CI/CD пайплайнов.

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
    *   CI/CD пайплайн для автоматической сборки Docker-образов и их развертывания в кластер Kubernetes.

---

## 1. Инфраструктура как код (Terraform)

### 1.1. Описание
Terraform используется для декларативного управления облачными ресурсами. Были созданы:
*   Виртуальное облако (VPC)
*   Подсети в трех зонах доступности
*   NAT Gateway для исходящего доступа из подсетей
*   Три виртуальные машины: 1 Master-Node (control-plane) и 2 Worker-Nodes.
*   Настроен SSH-доступ и автоматическое назначение внешних IP-адресов.

### 1.2. Ключевые команды
```bash
# Инициализация Terraform
terraform init

# Проверка плана изменений
terraform plan

# Применение изменений (создание ресурсов)
terraform apply -auto-approve

# minikube_nginx
# 1. Кластер
Для выполнения задания использую удаленый сервер 4CPU/4RAM/20gb. 
apt update 
apt install net-tools
apt install git 
- Установка minikube:
apt install docker.io
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
minikube start --force
minikube kubectl -- get pods -A
Или установка kubectl: 
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
# 2. Манифесты 
Локально создаю необходимые файлы в vs code и push их в git, чтобы взять их на удаленом сервере для развертывания. 
1. Установка helm: 
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
2. Установка Ingress Nginx:
helm install ingress-nginx ingress-nginx/ingress-nginx --namespace default
3. Создание Deployment для nginx-default-backend (nginx-backend-deployment.yaml) с 1 репликой 
(используй образ nginx или официальный k8s.gcr.io/defaultbackend).
Настройка Service типа ClusterIP для nginx-default-backend (nginx-backend-service.yaml).
4. Создание ConfigMap(nginx-html-configmap.yaml), содержащий файл index.html с текстом "Hello, Kubernetes!".
Подключение ConfigMap к nginx-default-backend через volume (nginx-html-config), чтобы он отображал
этот index.html.
Добавляем запись в /etc/hosts
echo "$(minikube ip) hello.local" | sudo tee -a /etc/hosts
5. Настрой Ingress (nginx-ingress.yaml) для маршрутизации запросов на бэкенд (хост: hello.local).
Включение ingress
minikube addons enable ingress
ClusterRoleBinding дает сервис-аккаунту ingress-nginx полный доступ (cluster-admin), чтобы он мог управлять ConfigMap, Service и Secret.
kubectl create clusterrolebinding ingress-nginx \
  --clusterrole=cluster-admin \
  --serviceaccount=default:ingress-nginx


server: git pull

Применение всех созданных манифестов:
kubectl apply -f nginx-html-configmap.yaml
kubectl apply -f nginx-backend-deployment.yaml
kubectl apply -f nginx-backend-service.yaml
kubectl apply -f nginx-ingress.yaml

# 3. Сеть 

minikube tunnel

kubectl patch svc ingress-nginx-controller -n default -p '{"spec": {"externalIPs": ["192.168.49.2"]}}'

# 4. Логи и отладка

kubectl get pods -n default

kubcetl logs <name pod>

kubectl logs -l app.kubernetes.io/name=ingress-nginx -n default

kubectl get clusterrolebinding ingress-nginx -o yaml

kubectl get svc -A

kubectl get svc nginx-backend //проверка, что сервис работает

curl -v http://hello.local

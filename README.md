# spring-minikube-demo
Demo Spring Boot app for Minikube on Apple Silicon

# Пример деплоя в minikube 

### Эндпоинты сервиса
1. Ready
```bash
curl http://localhost:8080/
```
2. Hello
```bash
curl http://localhost:8080/hello
```
3. Actuator
```bash
curl http://localhost:8080/actuator/health
```

### Сборка и деплой в minikube
1. Собираем JAR 
```bash
./mvnw clean package  
```

2. Запустить Docker-desktop  

3. Запустить Minikube  
```bash
minikube start --driver=docker
```
Дождаться сообщения: "🏄  Готово! kubectl настроен для использования кластера "minikube" и "default" пространства имён по умолчанию"    

4. Сборка Docker-образа:  
```bash
docker build -t spring-minikube-demo:v1 .
```
5. Проверить, что образ spring-minikube-demo:v1 появился в docker-desktop 
В Docker-desktop должен появиться: spring-minikube-demo:v1

6. Убедитесь, что Minikube запущен:  
```bash
minikube status
```
Консоль:  
```txt
  minikube
  type: Control Plane
  host: Running
  kubelet: Running
  apiserver: Running
  kubeconfig: Configured
```

6.1. Если не запущен, то запустить:  
```bash
minikube start --driver=docker
```

7. Загружаем образ spring-minikube-demo:v1 в Minikube:
```bash
minikube image load spring-minikube-demo:v1
```

7.1. Проверить список образов в Minikube и убедиться, что spring-minikube-demo:v1 есть
```bash
minikube ssh -- docker images
```
Консоль:  
```txt
  REPOSITORY                                TAG       IMAGE ID       CREATED         SIZE
  spring-minikube-demo                      v1        05a0d552f98e   3 minutes ago   426MB
  ...
```

8. Создаём [deployment.yaml](deployment.yaml)  

9. Применяем манифест:   
```bash
kubectl apply -f deployment.yaml
```
Консоль:
```txt
  deployment.apps/spring-minikube-demo created
  service/spring-minikube-demo-service created
```

10. Проверяем статус  

10.1 Cписок Pod  
```bash
kubectl get pods
```
Консоль:
```txt
  NAME                                    READY   STATUS    RESTARTS   AGE
  spring-minikube-demo-86b7c9cf86-tnhvt   1/1     Running   0          40s
```
10.2 Список Services
```bash
kubectl get services
```
Консоль:
```txt
  NAME                           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
  kubernetes                     ClusterIP   10.96.0.1      <none>        443/TCP          5m18s
```

11. Открываем в браузере:  
```bash
minikube service spring-minikube-demo-service
```

12. Проверяем health check
```
http://127... /actuator/health
```

14. Запустить (UI) веб-интерфейс (Kubernetes Dashboard) в Minikube
```bash
minikube dashboard
```
Консоль: 
```text
  ...
  🎉  Opening http://127.0.0.1:56926/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/ in your default browser...
```
Проверить 
- Deployments: spring-minikube-demo    
- Pods: spring-minikube-demo:v1    
- Replica Sets: spring-minikube-demo:v1    
- Services: spring-minikube-demo-service, kubernetes  
- Namespaces:     
 
- Config Maps: kube-root-ca.crt     
- Secrets: -   

- Nodes: minikube  


### Последовательность остановки приложения и Minikube  
1. Остановить Pod'ы и удалить Service
```bash
kubectl delete -f deployment.yaml
```
Консоль:  
```txt
  deployment.apps "spring-minikube-demo" deleted from default namespace
  service "spring-minikube-demo-service" deleted from default namespace
```

2. Остановить Minikube (остановка контейнера minikube в Docker)
```bash
minikube stop
```
Консоль:  
```txt
  ✋  Узел "minikube" останавливается ... 
  🛑  Выключается "minikube" через SSH ...
  🛑  Остановлено узлов: 1.
```
3. Удалить Docker-образ сервиса spring-minikube-demo:v1 из локального Docker
```bash
docker rmi spring-minikube-demo:v1
```
Консоль:  
```text
  Untagged: spring-minikube-demo:v1
  Deleted: sha256:05a0d552f98eacd94a41f1079f309086c4e550069f1f7171aca8af24f2b7d4af
```

4. Полная очистка кластера (Опционально) - удалит контейнер и тома minikube в Docker-контейнер
Имидж gcr.io/k8s-minikube/kicbase:v0.0.48 остается в Docker-desktop
```bash
minikube delete
```
Консоль:  
```text
  🔥  Deleting "minikube" in docker ...
  🔥  Deleting container "minikube" ...
  🔥  Removing /Users/sergeyproshchaev/.minikube/machines/minikube ...
  💀  Removed all traces of the "minikube" cluster.
```

INSTRUCTION.md

# Інструкція для валідації змін

## Передумови
- Установлені: Docker, Kind, kubectl
- Fork репозиторію клонований локально

## Крок за кроком

### 1. Запустити Kubernetes кластер
```bash
kind create cluster --config cluster.yml
```

Перевірка:
```bash
kubectl get nodes
# Має показати: 1 control-plane + 2 workers
```

### 2. Розгорнути додаток та Ingress контролер
```bash
bash bootstrap.sh
```

Перевірка:
```bash
kubectl get pods -A
# Мають бути pod'и з nginx ingress controller
```

### 3. Розмістити Ingress конфігурацію
```bash
mkdir -p ./infrastructure/ingress
cp ingress.yml ./infrastructure/ingress/
```

Розгорнути Ingress:
```bash
kubectl apply -f ./infrastructure/ingress/ingress.yml
```

Перевірка:
```bash
kubectl get ingress
# Результат:
# NAME          CLASS   HOSTS       ADDRESS      PORTS   AGE
# app-ingress   nginx   localhost   localhost    80      10s
```

### 4. Перевірити роботу додатку

#### В браузері:
- Перейти на `http://localhost`
- Мав бути видно додаток (не 404 помилка)

#### В консолі браузера (F12):
- Мав бути 200 статус код
- Не має бути 404 помилок
- Мав бути Content-Type: text/html або application/json

#### За допомогою curl:
```bash
curl -v http://localhost/
# Має показати: HTTP/1.1 200 OK
```

### 5. Перевірити Ingress правила

```bash
kubectl describe ingress app-ingress
```

Результат мав містити:
```
Rules:
  Host        Path  Backends
  ----        ----  --------
  localhost   /(.*)   app-service:8080
```

## Очікувані результати ✅

- [ ] Kind кластер запущений (3 ноди)
- [ ] Nginx Ingress Controller розгорнутий
- [ ] Ingress ресурс активний
- [ ] http://localhost доступний
- [ ] Статус код 200 (не 404)
- [ ] Додаток видно в браузері
- [ ] Немає помилок у консолі браузера

## Якщо щось не працює

### 404 помилка при доступі на localhost
```bash
# 1. Перевірити чи сервіс існує
kubectl get svc

# 2. Перевірити чи pod додатку запущений
kubectl get pods

# 3. Перевірити логи Ingress контролера
kubectl logs -n ingress-nginx deployment/ingress-nginx-controller
```

### Ingress не має адреси (ADDRESS пустий)
```bash
# Перевірити статус контролера
kubectl get deployment -n ingress-nginx
kubectl describe ingress app-ingress
```

### Дивіться також
- `kubectl get all` - всі ресурси в кластері
- `kubectl describe pod {pod-name}` - деталі pod'а
- `kubectl logs {pod-name}` - логи додатку
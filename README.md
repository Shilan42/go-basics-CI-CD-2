# Настройка автоматического деплоя с помощью GitHub Actions

## О проекте

Проект демонстрирует настройку **непрерывного развертывания (CD)** с использованием GitHub Actions для автоматизации публикации и развертывания обновлений программного обеспечения.

## Описание

### Что такое CI/CD?

**CI/CD** (Continuous Integration/Continuous Deployment) — это методология разработки программного обеспечения, которая автоматизирует процессы интеграции кода и развертывания приложений.

### Компоненты проекта

* **GitHub Actions** — для автоматизации рабочих процессов
* **Docker** — для контейнеризации приложения
* **Go** — язык программирования для создания примера приложения

## Установка и настройка

### Создание проекта

1. Создайте каталог для проекта:
```bash
mkdir hello-golang
cd hello-golang
```

2. Инициализируйте Go модуль:
```bash
go mod init github.com/<ваше имя>/hello-golang
```

3. Создайте основной файл приложения:
```bash
touch main.go
```

### Настройка GitHub

1. Создайте репозиторий на GitHub
2. Добавьте секреты для доступа к Docker Hub:
   * **DOCKER_USERNAME** — ваш Docker ID
   * **DOCKER_ACCESS_TOKEN** — токен доступа к Docker Hub

## Рабочий процесс (Workflow)

### Структура workflow

```yaml
name: golang-pipeline
on: push
jobs:
  test:
    runs-on: ubuntu-latest
    container: golang:1.22
    steps:
      - uses: actions/checkout@v4
      - name: Run Unit Tests
        run: GOOS=linux GOARCH=amd64 go test
      - name: Vet
        run: go vet ./...

  deploy:
    name: Push Docker image to Docker Hub
    runs-on: ubuntu-latest
    needs: test
    if: startsWith(github.ref, 'refs/tags')
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Set up Docker buildx
        uses: docker/setup-buildx-action@v3
      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_ACCESS_TOKEN }}
      - name: Extract metadata (tags, labels) for Docker
        id: meta
        uses: docker/metadata-action@v5.5.1
        with:
          images: <Docker ID>/hello-golang
      - name: Build and push Docker Image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
```

## Docker настройка

Создайте Dockerfile в корне проекта:

```dockerfile
FROM golang:1.22

WORKDIR /app

COPY . .

RUN go mod tidy

RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o /main main.go

CMD ["/main"]
```

## Запуск и тестирование

1. Соберите Docker образ:
```bash
docker build -t golang-hello:v1.0.0 .
```

2. Запустите контейнер:
```bash
docker run golang-hello:v1.0.0
```

## Деплой

Для деплоя приложения:
1. Добавьте тег версии:
```bash
git tag v1.0.0
git push --tags
```

2. Проверьте выполнение workflow в GitHub Actions

## Требования

* GitHub аккаунт
* Docker Hub аккаунт
* Установленный Go
* Git
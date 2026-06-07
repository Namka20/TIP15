# Практическое занятие № 15. Деплой приложения на VPS. Настройка systemd

# Нам А. В., ЭФМО-01-25

## Цель работы
Освоить публикацию backend-приложения на удалённом Linux-сервере, научиться подключаться к VPS по SSH, размещать исполняемый файл приложения, настраивать переменные окружения, создавать unit-файл systemd, управлять сервисом через systemctl, анализировать логи через journalctl и выполнять базовую процедуру обновления версии приложения.

## Структура проекта
```
pz15/
├── deploy
│   ├── cert.pem
│   ├── docker-compose.prod.yml
│   ├── docker-compose.yml
│   ├── init.sql
│   ├── key.pem
│   ├── nginx.conf
│   └── systemd
│       └── pr15-stack.service
├── docs
│   └── screenshots
│       ├── catalog.png
│       ├── ci_cd.png
│       ├── docker ps.png
│       ├── health.png
│       ├── logs.png
│       ├── requests.png
│       ├── service status.png
│       ├── ssh connection.png
│       └── system unit.png
├── go.work
├── go.work.sum
├── proto
│   ├── auth.proto
│   ├── authpb
│   │   ├── auth_grpc.pb.go
│   │   └── auth.pb.go
│   ├── go.mod
│   └── go.sum
├── README.md
├── services
│   ├── auth
│   │   ├── cmd
│   │   │   └── auth
│   │   │       └── main.go
│   │   ├── Dockerfile
│   │   ├── go.mod
│   │   ├── go.sum
│   │   └── internal
│   │       ├── grpcapi
│   │       │   └── server.go
│   │       ├── http
│   │       │   ├── handler.go
│   │       │   └── router.go
│   │       └── service
│   │           ├── model.go
│   │           └── service.go
│   ├── graphql
│   │   ├── cmd
│   │   │   └── graphql
│   │   │       └── main.go
│   │   ├── Dockerfile
│   │   ├── go.mod
│   │   ├── go.sum
│   │   ├── gqlgen.yml
│   │   ├── graph
│   │   │   ├── generated.go
│   │   │   ├── models_gen.go
│   │   │   ├── resolver.go
│   │   │   ├── schema.graphqls
│   │   │   └── schema.resolvers.go
│   │   ├── internal
│   │   │   ├── http
│   │   │   │   ├── auth.go
│   │   │   │   └── router.go
│   │   │   ├── repository
│   │   │   │   └── repository.go
│   │   │   └── service
│   │   │       └── service.go
│   │   └── tools.go
│   ├── tasks
│   │   ├── cmd
│   │   │   └── tasks
│   │   │       └── main.go
│   │   ├── Dockerfile
│   │   ├── go.mod
│   │   ├── go.sum
│   │   └── internal
│   │       ├── cache
│   │       │   └── redis.go
│   │       ├── client
│   │       │   └── authgrpc
│   │       │       └── client.go
│   │       ├── http
│   │       │   ├── handler.go
│   │       │   └── router.go
│   │       ├── rabbit
│   │       │   └── producer.go
│   │       ├── repository
│   │       │   └── repository.go
│   │       └── service
│   │           ├── model.go
│   │           ├── service.go
│   │           └── store.go
│   └── worker
│       ├── cmd
│       │   ├── go.mod
│       │   ├── go.sum
│       │   └── main.go
│       └── Dockerfile
└── shared
    ├── go.mod
    ├── go.sum
    ├── logger
    │   └── logger.go
    ├── metrics
    │   └── metrics.go
    └── middleware
        ├── accesslog.go
        ├── csrf.go
        ├── instanceid.go
        ├── metrics.go
        ├── requestid.go
        └── securityheaders.go
```

## Результаты выполнения (скриншоты)

## 1. IP/хост VPS и факт подключения по SSH


```bash
ssh -i ~/.ssh/pr15_vps root@89.22.236.121
```

<img width="632" height="298" alt="image" src="https://github.com/user-attachments/assets/f5b94b03-578d-4b0c-8fb8-a25075be719a" />

### Пример systemctl status

<img width="457" height="130" alt="image" src="https://github.com/user-attachments/assets/989bb7e4-aa8c-40f0-acb6-4cb656b0a4bb" />

### Пример journalctl -u pr15-stack -n 30

<img width="514" height="128" alt="image" src="https://github.com/user-attachments/assets/6d9ce490-4069-43ea-b096-67aefeec546f" />

### Проверка /health

<img width="451" height="41" alt="image" src="https://github.com/user-attachments/assets/fd7127f6-9fb0-4cc6-8e2f-7b5ba62a3303" />

### 7. Процедура обновления и отката

#### Обновление версии

1. Выполнить `git push` в `main`.
2. GitHub Actions собирает образы и публикует их в Docker Hub.
3. Deploy job подключается к VPS по SSH и выполняет:
```bash
docker-compose -f docker-compose.prod.yml pull
docker-compose -f docker-compose.prod.yml up -d --remove-orphans
```


#### Откат

1. В файле `/opt/pr15/deploy/.env` на VPS установить предыдущий тег:
```env
IMAGE_TAG=<previous_commit_sha>
```
2. Применить:
```bash
systemctl reload pr15-stack
```



## Ответы на контрольные вопросы

1. Что такое VPS и зачем он нужен backend-разработчику?
VPS (Virtual Private Server) — виртуальный выделенный сервер. Нужен для развёртывания и тестирования приложений в среде, приближенной к production, изучения администрирования и DevOps-практик.

2. Почему запуск приложения на VPS отличается от локального запуска на компьютере разработчика?
На VPS требуется настройка служб (systemd), переменных окружения, открытие портов в фаерволе, работа с доменами, SSL-сертификатами и балансировкой — локально всё работает "из коробки" без этих шагов.

3. Для чего используется systemd?
Это система инициализации и менеджер служб в Linux. Позволяет запускать приложение как демона (фоновая служба), управлять его жизненным циклом (запуск, остановка, перезапуск, автозагрузка).

4. Почему не рекомендуется запускать серверное приложение от root?
Из соображений безопасности: при взломе приложения злоумышленник получает полный доступ к системе. Лучше создавать отдельного пользователя с минимальными правами.

5. Зачем выносить конфигурацию в отдельный env-файл?
Чтобы не хранить секреты и настройки окружения в коде или systemd-файле — удобно менять конфигурацию под разные среды (dev/staging/prod) без перекомпиляции.

6. Что делает параметр Restart=always?
Принуждает systemd автоматически перезапускать сервис при любом завершении (аварийном или нормальном) — обеспечивает самовосстановление.

7. Для чего нужен EnvironmentFile в unit-файле?
Указывает путь к файлу с переменными окружения — systemd загрузит их перед запуском сервиса.

8. Как проверить состояние службы через systemctl?
```bash
sudo systemctl status имя-сервиса
```

9. Как посмотреть логи сервиса через journalctl?
```bash
sudo journalctl -u имя-сервиса -f
```

10. Что нужно сделать перед обновлением unit-файла systemd?
Выполнить:
```bash
sudo systemctl daemon-reload
```
Чтобы systemd перечитал изменённые конфигурации unit-файлов.

11. Почему полезно иметь процедуру отката версии?
При обнаружении критических ошибок в новой версии можно быстро вернуться к стабильной без длительного передеплоя, минимизируя время простоя.

12. Зачем в реальных системах часто используют NGINX перед приложением?
Как reverse-proxy и load balancer: раздача статики, терминация HTTPS, балансировка нагрузки, кеширование, защита от DDoS, обработка медленных клиентов, единая точка входа.

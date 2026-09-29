# Bikes API: Automacao, Observabilidade e DevSecOps

API Java Spring Boot com build, metricas, dashboards e logs centralizados.

## Stack

- Java 21 + Spring Boot
- MySQL para persistencia
- Prometheus + Actuator + Micrometer
- Grafana com dashboard provisionado
- Graylog + MongoDB + OpenSearch
- GitHub Actions publicando no GHCR

## Execucao local

Pre-requisitos: Docker Engine com Compose v2 e 4 GB de memoria disponivel.

```bash
docker compose up --build
```

| Servico | URL | Credenciais |
| --- | --- | --- |
| API | http://localhost:8080 | - |
| Swagger | http://localhost:8080/docs-bikes.html | - |
| Actuator | http://localhost:8080/actuator/health | - |
| Prometheus | http://localhost:9090 | - |
| Grafana | http://localhost:3000 | admin / admin |
| Graylog | http://localhost:9000 | admin / admin |

O servico `graylog-init` cria a entrada GELF UDP na porta `12201` automaticamente. A API envia logs estruturados para ela e tambem os imprime no console do container.

```bash
docker compose down -v
```

## Pipeline CI/CD

O workflow em `.github/workflows/ci-cd.yml` roda em push e pull request para `main` ou `master`. Ele executa `./mvnw clean verify`, constroi a imagem pelo `Dockerfile` multistage e, em push, publica tags com SHA e `latest` no GitHub Container Registry.

A publicacao usa `GITHUB_TOKEN`; habilite `read and write permissions` para Actions em Settings > Actions > General.

## Evidencias para entrega

1. Pipeline verde na aba Actions do GitHub.
2. Dashboard `Bikes API - Spring Boot` em `http://localhost:3000/d/bikes-api` com chamadas para `GET /api/v1/usuarios`.
3. Graylog com a busca `application:bikes` ou `facility:bikes` exibindo eventos GELF.
4. Prometheus em `http://localhost:9090/targets` mostrando o target `bikes` como `UP`.

## Decisoes de seguranca e operacao

- O `Dockerfile` usa dois estagios: Maven/JDK para build e JRE Alpine para runtime.
- O processo da API roda como usuario sem privilegios.
- Healthchecks controlam a ordem de inicializacao de MySQL, Graylog, API e Prometheus.
- As credenciais sao demonstrativas; em ambiente real devem vir de secrets.

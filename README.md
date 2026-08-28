# Desafio CI/CD

> Pipeline de CI/CD profissional com GitHub Actions para uma API Python, integrando testes unitários, análise de segurança estática (SAST), build de imagem Docker e publicação automática no Docker Hub — seguindo o fluxo de trabalho GitFlow.

## Sobre o projeto

Este repositório simula o cenário de uma startup que lança sua primeira API e precisa de um processo de entrega automatizado, seguro e confiável. A aplicação em si é simples (dois endpoints em FastAPI), mas o foco do desafio é a esteira de CI/CD ao redor dela:

- **CI** — a cada Pull Request para `develop` ou `main`, o pipeline roda os testes unitários com `pytest` e uma análise de segurança estática (SAST) com **Semgrep**.
- **CD** — a cada merge na branch `main`, o pipeline builda a imagem Docker da API e publica no Docker Hub, com as tags `latest` e o SHA do commit. A imagem pública está disponível em: [Docker Hub](https://hub.docker.com/repository/docker/gildo2005/desafio-cicd)

## Stack

- **Linguagem/Framework:** Python 3.14 + [FastAPI](https://fastapi.tiangolo.com/)
- **Testes:** pytest + `TestClient` do FastAPI (via `httpx`)
- **SAST:** [Semgrep](https://semgrep.dev/) (ruleset `p/python`)
- **Containerização:** Docker
- **CI/CD:** GitHub Actions
- **Registro de imagens:** Docker Hub

## Endpoints da API

| Método | Rota      | Descrição                                  |
|--------|-----------|---------------------------------------------|
| GET    | `/`       | Retorna uma mensagem de saudação e status   |
| GET    | `/health` | Retorna o status de saúde da aplicação      |

## Como executar localmente

### Sem Docker

```bash
# criar e ativar um ambiente virtual (opcional, mas recomendado)
python -m venv .venv
.venv\Scripts\activate      # Windows
source .venv/bin/activate   # Linux/Mac

# instalar dependências
pip install -r requirements.txt

# subir a aplicação
uvicorn app.main:app --host 0.0.0.0 --port 8000
```

A API estará disponível em `http://localhost:8000`.

### Com Docker

```bash
docker build -t desafio-cicd .
docker run -p 8000:8000 desafio-cicd
```

### Rodando os testes

```bash
pytest tests/ -v
```

## Estrutura do repositório

```
.
├── .github/
│   └── workflows/
│       ├── ci.yml      # Testes + SAST (Pull Requests)
│       └── cd.yml      # Build + Push da imagem Docker (merge em main)
├── app/
│   ├── __init__.py
│   └── main.py          # Código da API
├── tests/
│   └── test_main.py     # Testes unitários
├── Dockerfile
├── requirements.txt
└── README.md
```

## Pipeline de CI (`ci.yml`)

Disparado em **Pull Requests para `develop` ou `main`**. Roda dois jobs em paralelo:

1. **Testes** — instala as dependências e executa `pytest tests/ -v`. Se qualquer teste falhar, o pipeline falha.
2. **Análise de Segurança (SAST)** — roda o Semgrep (imagem oficial `semgrep/semgrep`) com o ruleset `p/python`, reportando os findings de segurança nos logs do job.

## Pipeline de CD (`cd.yml`)

Disparado apenas em **push (merge) na branch `main`**:

1. Faz checkout do código.
2. Autentica no Docker Hub usando os secrets `DOCKER_USERNAME` e `DOCKER_PASSWORD`.
3. Configura QEMU e Docker Buildx.
4. Builda a imagem a partir do `Dockerfile` e publica no Docker Hub com duas tags:
   - `latest`
   - o SHA do commit (`${{ github.sha }}`)

## Fluxo de branches (GitFlow)

- **`main`** — código de produção (protegida). Todo merge aqui dispara o CD.
- **`develop`** — branch de integração das features.
- **`feature/*`** — desenvolvimento de novas funcionalidades, a partir de `develop`.

Fluxo típico: `feature/*` → PR → `develop` → PR → `main`.

## Secrets utilizados

Configurados em *Settings → Secrets and variables → Actions* do repositório:

| Secret            | Descrição                                    |
|-------------------|-----------------------------------------------|
| `DOCKER_USERNAME` | Usuário do Docker Hub                        |
| `DOCKER_PASSWORD` | Access Token do Docker Hub (permissão Read & Write) |

Nenhuma credencial é exposta no código-fonte ou nos logs do pipeline.

## Fontes de pesquisa

**CI:**
- [Testes com Python no GitHub Actions](https://docs.github.com/pt/actions/tutorials/build-and-test-code/python)
- [Semgrep para Python](https://docs.semgrep.dev/languages/python)
- [Semgrep em pipelines de CI](https://docs.semgrep.dev/semgrep-ci/sample-ci-configs#github-actions)

**CD:**
- [Secrets e Variables no GitHub Actions](https://medium.com/@morepravin1989/github-actions-secrets-and-variables-understanding-repository-and-environment-secrets-for-2b2eed404222)
- [Contexts do GitHub Actions](https://docs.github.com/pt/actions/reference/workflows-and-actions/contexts)
- [Material do curso de GitHub Actions](https://devopsautomation.com.br/udemy/github-actions-automacao/modulo-04-avancado/secrets)
- [Docker Login Action](https://github.com/marketplace/actions/docker-login)
- [Docker Build and Push Action](https://github.com/marketplace/actions/build-and-push-docker-images)

**Gerais:**
- Google
- Claude (apenas para dúvidas pontuais e ajuda com a escrita do README — nenhum código ou parte dos workflows foi gerado por IA)

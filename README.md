# Sistema de Web Scraping de Livros

## Estrutura do Repositório
Related repositories:

```text
.
├── backend-laravel/     # API Central e orquestrador em Laravel
├── frontend-vue/        # Interface de utilizador em Vue.js
└── scraping-service/    # Microsserviço de scraping em Node.js
```
- API/Backend (Laravel): [WebScrapperApi](https://github.com/ricardo4457/WebScrapperApi)
- Scraping service (Node.js): [WebScrapper](https://github.com/ricardo4457/WebScrapper)

---

## Requisitos do Sistema
## Requirements

Para executar este projeto, certifique-se de que possui os seguintes softwares instalados no seu ambiente de desenvolvimento:

- **Docker** e **Docker Compose** (recomendado para a execução dos serviços e bases de dados)
- **PHP** >= 8.2 (caso execute o Laravel fora de contentor)
- **Composer** (gestor de dependências do PHP)
- **Node.js** >= 18.x e **npm** / **yarn** (para o frontend e o microsserviço Node.js)
- **Redis** (servidor de filas, caso não utilize o Docker Compose)
- Node.js 18+
- The Laravel API running and reachable (see `VITE_API_URL` below)

---

## Variáveis de Ambiente

Crie e configure os ficheiros `.env` em cada microsserviço conforme as necessidades do seu ambiente. Abaixo encontram-se as principais variáveis de configuração utilizadas (com foco no microsserviço de scraping):
## Running the project

```env
# --- Redis / BullMQ ------------------------------------------------------------
REDIS_HOST=localhost
REDIS_PORT=6379
```bash
npm install
npm run dev
```

# --- Servidor Express (rota /scrape) --------------------------------------------
PORT=3000
```bash
npm run build     # production build
npx vitest run    # Vitest unit tests
```

SCRAPE_CONCURRENCY=3
SCRAPER_ENGINE=camoufox
# SCRAPER_HEADLESS=true
### Environment variables

LARAVEL_API_URL=http://localhost:8000/api
```env
VITE_API_URL=http://localhost:8000/api
VITE_APP_KEY=...   # sent as X-App-Key header, validated by Laravel's VerifyAppApiKey middleware
```

---

## Arquitetura do Sistema

O sistema é composto pelas seguintes tecnologias e serviços principais:
## Architecture

- **Frontend (Vue.js):** Interface de utilizador responsável por iniciar pedidos de scraping, consultar estados e apresentar os resultados das pesquisas de livros.
- **Backend / API Principal (Laravel):** Atua como orquestrador central. Recebe os pedidos do frontend, gere a segurança, e comunica com o serviço de scraping.
- **Worker de Scraping (Node.js):** Um microsserviço dedicado à execução assíncrona das tarefas de extração de dados.
- **Mensageria e Filas (Redis + BullMQ):** Sistema escolhido para a gestão de jobs em background, devido à sua baixa latência e gestão nativa do estado das tarefas.
![Frontend architecture](./docs/Arquitetura_Frontend.drawio.png)

---

## Segurança e Fluxo de Comunicação

O sistema implementa fronteiras estritas de segurança entre os seus componentes:
The app is organized by functional domain rather than by technical type, so a change to the search flow stays confined to `search-flow/` instead of spreading across unrelated folders. Data flows top-down: **Views/Components** read from **Pinia stores**, stores call **services** for HTTP access, and services talk to the Laravel API — components never call `axios` directly.

| Origem      | Destino     | Finalidade                                                  | Mecanismo de Proteção                                         |
| :---------- | :---------- | :---------------------------------------------------------- | :------------------------------------------------------------ |
| **Vue.js**  | **Laravel** | Iniciar scraping, consultar estado e pesquisar livros.      | API key, validação de origem (CORS) e rate limiting.          |
| **Laravel** | **Node.js** | Enviar tarefas de scraping para a fila de processamento.    | Comunicação interna de rede (isolada).                        |
| **Node.js** | **Laravel** | Enviar resultados (callbacks) e atualizar estados dos jobs. | Token partilhado validado pelo middleware `VerifyNodeApiKey`. |
```

## Como Executar o Projeto

1. Clone este repositório:
   ```bash
   git clone https://github.com/seu-utilizador/seu-repositorio.git
   ```
2. Suba os serviços utilizando o Docker Compose:
   ```bash
   docker-compose up -d
   ```
3. Configure as variáveis de ambiente (`.env`) nos diretórios do Laravel e do Node.js, certificando-se de partilhar o token de segurança entre eles.
4. Execute as migrações do Laravel:
   ```bash
   php artisan migrate
   ```

## Testing

Para garantir a integridade do código e o correto funcionamento dos microsserviços, pode executar a bateria de testes disponível em cada módulo:


- **Backend (Laravel):**
  ```bash
  php artisan test
  ```
- **Microsserviço de Scraping (Node.js):**
  ```bash
  npm test
  ```
- **Front-end Vue (Vue.js):**

  ```bash
npx vitest run
```

## Direitos de Autor e Licença
## License

MIT — see [LICENSE](LICENSE).

Este projeto está licenciado sob a **MIT License**. Consulte o ficheiro [LICENSE](LICENSE) para mais detalhes.
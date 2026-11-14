# Top Players of All Sports

**A full-stack sports discovery and player-ranking application with a React frontend, Spring Boot services, data ingestion, AI-assisted analysis, and community voting.**

The project brings together player profiles and comparisons, sports news, video highlights, and a **Rating Day** system in which user votes update player Elo ratings. The repository includes a multi-service Java backend, a responsive React interface, persistence and caching layers, and Temporal workflow implementations for selected background processes.

> **Status:** Active development / portfolio project. The repository contains implemented services and interfaces, but some integrations depend on external APIs or local infrastructure, and several configuration defaults need alignment before a fully reproducible end-to-end deployment. It should not be described as a production-hardened platform based on the code alone.

## Features

- **Player directory and profiles:** Browse players, inspect player information, rankings and analysis, and compare athletes.
- **Sports dashboard:** Category-based navigation, top-player tables, quick statistics and content feeds.
- **AI-assisted player assessment:** Backend services build prompts from available player context and process structured AI-generated player analyses. Scores are model-assisted assessments, not independently validated objective rankings.
- **Rating Day voting:** Head-to-head matchups, user votes, nominations and results, with an Elo-based rating update implementation.
- **News aggregation:** A dedicated news service with search, filtering, pagination and NewsAPI integration.
- **Video highlights:** A separate service for searching, filtering and serving video metadata, including YouTube ingestion code.
- **Account features:** Google OAuth integration, JWT access tokens, refresh-token rotation and protected routes.

Some UI sections use fallback data when external sources are unavailable; their presence does not prove that every live API integration is operational.

## Architecture

```text
                          React 18 + Vite + Tailwind
                  Dashboard | Profiles | News | Highlights
                       Comparisons | Rating Day | Auth
                                      |
                        HTTP REST APIs / authentication
                                      |
              +-----------------------+----------------------+
              |                       |                      |
       Player Service          News Service         Highlights Service
       Spring Boot / Java      Spring Boot / Java   Spring Boot / Java
       Port 8084               Port 8082            Port 8083
       - player data           - article feeds      - video metadata
       - AI analysis           - search/filters     - channel ingestion
       - Elo voting            - ingestion          - search/trending
       - Google OAuth/JWT
              |                       |                      |
              +-----------------------+----------------------+
                                      |
                   PostgreSQL | Redis | Flyway | Temporal*
                                      |
                    External sports/news/video/AI APIs

* Temporal integration is implemented in selected backend modules;
  its activation depends on service configuration and running infrastructure.
```

### Backend components

| Component | Code location | Responsibility |
| --- | --- | --- |
| Player service | [`services/player-service/`](services/player-service/) | Player data, AI analysis, ranking, Elo voting, authentication and administrative operations |
| News service | [`services/news-service/`](services/news-service/) | NewsAPI ingestion, stored articles, filtering, search and caching |
| Highlights service | [`services/highlights-service/`](services/highlights-service/) | Video ingestion and metadata APIs, YouTube integration, caching and background workflow classes |
| Frontend | [`src/`](src/) | Responsive dashboard, player pages, comparison, news, highlights, profiles and Rating Day |
| Database changes | [`services/player-service/src/main/resources/db/migration/`](services/player-service/src/main/resources/db/migration/) and [`services/highlights-service/src/main/resources/db/migration/`](services/highlights-service/src/main/resources/db/migration/) | Schema evolution and seed data |

## Engineering highlights

### Elo-based community voting

The [`EloService`](services/player-service/src/main/java/com/topplayersofallsports/playerservice/service/EloService.java) uses the standard expected-score calculation and a configurable-in-code `K_FACTOR` of `32`. A vote updates both players' ratings, saves the matchup, checks for duplicate votes, enforces a per-user limit, and uses Redis to track rate limits and voting counts. The backend exposes endpoints for matchups, voting, previous results and vote history.

### Data ingestion and background processing

The news and highlights services separate API interaction, domain logic, persistence, HTTP controllers and Temporal workflow/activity code. For example, [`HighlightIngestWorkflowImpl`](services/highlights-service/src/main/java/com/topplayersofallsports/highlights/temporal/HighlightIngestWorkflowImpl.java) coordinates fetching source details, collecting video IDs, enriching metadata, saving highlights and marking ingestion progress.

### API and persistence design

The backend uses Spring Boot REST controllers, JPA repositories, PostgreSQL and Redis. Player and highlights services include Flyway migrations; API documentation is exposed through Springdoc/OpenAPI configuration. The player service includes authentication and JWT filters, Google OAuth integration, and refresh-token handling.

### Frontend structure

The frontend uses React Router with routes for the dashboard, players, news, video highlights, comparisons, profile management, authentication and Rating Day. API calls are separated into service modules such as [`playerApiService.js`](src/services/playerApiService.js), [`newsService.js`](src/services/newsService.js), [`highlightsService.js`](src/services/highlightsService.js), and [`ratingDayService.js`](src/services/ratingDayService.js).

## Tech stack

| Layer | Technology |
| --- | --- |
| Frontend | React 18, Vite, JavaScript/JSX, React Router, Tailwind CSS, Axios |
| Visualisation and interface | Recharts, D3, Framer Motion, React Hook Form (dependencies and components vary by feature) |
| Backend | Java 17, Spring Boot 3, Spring Web, Spring Security, Spring Data JPA |
| Data | PostgreSQL, Redis, Flyway, Hibernate |
| Workflows | Temporal Java SDK, scheduled ingestion/registration modules |
| External integrations | Sports data APIs, NewsAPI, YouTube API, AI model providers, Google OAuth |
| Testing | JUnit 5 / Mockito-based Java tests in the player service |

## Repository structure

```text
topplayersofallsports/
├── src/
│   ├── components/             # Shared frontend UI
│   ├── contexts/               # Authentication context
│   ├── hooks/                  # Reusable hooks
│   ├── pages/                  # Dashboard, players, voting, news, highlights...
│   ├── services/               # Frontend API adapters
│   └── Routes.jsx              # Client-side navigation
├── services/
│   ├── player-service/         # Player ranking, OAuth, Elo, AI analysis
│   ├── news-service/           # News ingestion and search
│   └── highlights-service/     # Video ingestion and discovery
├── docs/                      # Design notes and implementation plans
├── package.json               # Frontend dependencies and scripts
└── vite.config.js             # Frontend development configuration
```

## Run locally

> The frontend and three Spring Boot services are started separately. There is **no verified root-level one-command full-stack setup** in the repository. The steps below are a starting point; the service configurations and external API keys must be matched to your environment.

### Prerequisites

- Node.js 18+ and npm
- Java 17 and Maven
- PostgreSQL and Redis for backend features
- Docker / Docker Compose if using containerised local infrastructure
- API credentials for any external integrations you want to exercise
- Temporal Server if enabling Temporal-backed workflows

### 1. Start the frontend

```bash
git clone https://github.com/Theo-Daniel/topplayersofallsports.git
cd topplayersofallsports
npm install
npm start
```

The Vite development server is configured to open on <http://localhost:3000>.

**Backend connections:** The frontend code reads `VITE_*` variables. For a local stack with services on their configured ports, create a root-level `.env.local` file (do **not** commit credentials):

```dotenv
VITE_PLAYER_API_URL=http://localhost:8084
VITE_NEWS_API_URL=http://localhost:8082/api/news
VITE_HIGHLIGHTS_API_URL=http://localhost:8083/api/highlights
VITE_ENABLE_DEBUG_LOGGING=false
```

The tracked root `.env.example` still contains a mixture of older `REACT_APP_*` variables and a highlights URL using port `8081`. The highlights Spring Boot service is actually configured for port `8083`; the example above aligns the frontend with its current service configuration.

### 2. Prepare backend infrastructure

The Java applications currently default to a PostgreSQL database at `localhost:5432/topplayersofallsports` and Redis at `localhost:6379` (see their `application.yml` files).

For an isolated *development* PostgreSQL and Redis instance, one possible Docker setup is:

```bash
docker run -d --name topplayers-postgres \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=postgres \
  -e POSTGRES_DB=topplayersofallsports \
  -p 5432:5432 postgres:16-alpine

docker run -d --name topplayers-redis \
  -p 6379:6379 redis:7-alpine
```

These example credentials are **local development only**. Change them in any exposed or shared environment, and do not run a second instance if those ports are already occupied.

**Important:** The Docker Compose file under `services/highlights-service/docker/` uses a different sample database name (`highlights_db`) and exposes PostgreSQL on host port `5433`. It is not a drop-in replacement for the shared-database defaults above without adjusting database URLs and migration expectations.

### 3. Configure and start services

The Java services require environment-specific configuration, including database access and some integration keys. Review their [`application.yml`](services/player-service/src/main/resources/application.yml) files before launching; the player service references values such as `GEMINI_API_KEY`, `ANTHROPIC_API_KEY`, `APISPORTS_FOOTBALL_KEY`, `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, and `JWT_SECRET`. The news service expects `NEWSAPI_API_KEY`, while video ingestion may need YouTube credentials.

In **separate terminals**, from the repository root:

```bash
cd services/player-service
mvn spring-boot:run
```

```bash
cd services/news-service
mvn spring-boot:run
```

```bash
cd services/highlights-service
mvn spring-boot:run
```

Service ports configured in the current source:

| Service | Port | Example check |
| --- | --- | --- |
| Player | `8084` | `curl http://localhost:8084/actuator/health` |
| News | `8082` | `curl http://localhost:8082/actuator/health` |
| Highlights | `8083` | `curl http://localhost:8083/actuator/health` |

Start the frontend after configuring the API URLs. A successful health response depends on having the required database, environment variables, migrations and services available. These commands have **not** been validated as a complete fresh-machine installation in this documentation revision.

### 4. Explore the APIs

Example read requests, once the corresponding services are operational:

```bash
curl 'http://localhost:8084/api/players/top?sport=FOOTBALL'
curl 'http://localhost:8082/api/news'
curl 'http://localhost:8083/api/highlights'
```

Additional routes are defined in the service controllers. Springdoc/Swagger configuration varies by module; consult the service documentation or its `application.yml` for the configured path.

## Tests

The player service includes JUnit/Mockito tests for controllers, authentication, JWT behaviour and Elo calculations:

```bash
cd services/player-service
mvn test
```

The repository's root `package.json` defines `start`, `build`, and `serve` scripts but **does not currently define a frontend `test` script**. Test files being present in the backend is not a claim that the whole stack passes end-to-end integration testing.

For a frontend production build:

```bash
npm run build
```

## Development limitations and next steps

- **Configuration alignment:** Complete a single reproducible development environment; current service sample files differ in PostgreSQL port/database, and the frontend highlights default differs from the backend port.
- **Secrets and access controls:** Remove development fallback secrets/credentials for any public deployment. Review the player service's currently permitted administrative seeding endpoints before exposing its API.
- **External-data dependencies:** Some functionality relies on external API access, quota and correctly configured keys. Fallback data and simulated refresh behaviour should not be presented as verified live ingestion.
- **Temporal enablement:** Temporal classes and configurations exist, but the standard local service configurations currently disable some Temporal integrations. Enable and test them intentionally before claiming complete scheduled-workflow operation.
- **Verification:** Add a root-level local-stack guide or Compose configuration, run the Java tests, check the frontend build and verify the user journeys end to end.
- **Portfolio demonstration:** Add genuine application screenshots, a short walkthrough and a concise description of your individual contribution when available.

## Additional documentation

- [Player service README](services/player-service/README.md)
- [News service README](services/news-service/README.md)
- [Highlights service README](services/highlights-service/README.md)
- [Architecture and design notes](docs/)

## Acknowledgements

The initial React frontend was created using [Rocket.new](https://rocket.new), as noted in the original repository README. This project also builds on the open-source libraries and services identified above.

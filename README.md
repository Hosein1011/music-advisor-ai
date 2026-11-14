# 🎧 Music Advisor — AI Vector Similarity Recommendation Engine

[![Java](https://img.shields.io/badge/Java-21%20LTS-ED8B00?style=flat-square&logo=openjdk)](https://openjdk.org/)
[![PostgreSQL](https://img.shields.io/badge/Database-PostgreSQL%20%2B%20pgvector-336791?style=flat-square&logo=postgresql)](https://github.com/pgvector/pgvector)
[![LLM](https://img.shields.io/badge/LLM-Meta%20LLaMA%203.2%2090B-blue?style=flat-square)](https://openrouter.ai/)
[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=flat-square&logo=docker)](https://www.docker.com/)
[![Maven](https://img.shields.io/badge/Build-Maven-C71A36?style=flat-square&logo=apachemaven)](https://maven.apache.org/)

**Music Advisor** is an intelligent music recommendation system combining Large Language Models (LLMs) with high-dimensional vector search. It leverages **Meta LLaMA 3.2 90B** to translate unstructured, subjective user descriptions into 22-dimensional acoustic and psychological vector profiles, then performs real-time nearest-neighbor retrieval over millions of track embeddings using **PostgreSQL with the `pgvector` extension**.

---

## 🏛️ System Architecture

```mermaid
flowchart LR
    User["User Natural Language Prompt\n('Upbeat energetic songs for coding')"]
    LLM["LLM Query Synthesizer\n(Meta LLaMA 3.2 90B Vision Instruct)"]
    PgVector[("PostgreSQL 16 + pgvector\n(22-Dimensional Audio Vectors)")]
    Engine["Music Advisor Recommendation Engine\n(Java 21 LTS + JDBC DAO)"]
    Output["Ranked Top-10 Recommended Tracks\n(Metadata + Distance Scores)"]

    User -->|Plain Text| Engine
    Engine -->|HTTP/2 API Request| LLM
    LLM -->|22-D Vector & SQL Query| Engine
    Engine -->|features <-> ARRAY[... scale 0..1]::vector| PgVector
    PgVector -->|Top-10 Nearest Neighbors| Engine
    Engine --> Output
```

---

## ✨ Architectural Highlights

- **Subjective-to-Vector Synthesis**: Leverages LLM reasoning to map subjective user prompts (e.g., *"rainy evening melancholy with soft guitar"*) into calibrated 22-dimensional feature weights (acousticness, danceability, valence, sadness, energy, instrumentalness, etc.).
- **pgvector Integration**: Directly executes Euclidean/L2 vector distance operators (`<->`) within PostgreSQL, avoiding external vector databases and keeping data transactionally consistent.
- **Microservice Containerization**: Bundles PostgreSQL and the `pgvector` extension inside Docker Compose (`ankane/pgvector`) for instant, reproducible local deployment.
- **Enterprise Java 21 Design**: Clean layered architecture adhering to the Repository/DAO pattern with robust connection management via JDBC.

---

## 📂 Repository Layout

```text
Music-Advisor/
├── Docker/                   # Container configurations
├── docker-compose.yml        # PostgreSQL + pgvector service definition
├── pom.xml                   # Maven dependencies (org.json, postgresql)
├── songs.csv                 # Audio dataset with 22-D acoustic features
├── src/main/java/
│   ├── Main.java             # Interactive CLI entrypoint
│   ├── model/
│   │   └── Song.java         # Domain entity representing track metadata
│   ├── repository/
│   │   └── PgVectorRepository.java # JDBC DAO executing vector distance SQL
│   ├── service/
│   │   ├── AiService.java    # LLM vector synthesizer via OpenRouter API
│   │   └── MusicAdvisor.java # Business logic orchestrator
│   └── util/
│       └── CSVLoader.java    # Bulk data ingestion utility for songs.csv
└── README.md
```

---

## 🚀 Getting Started

### 1. Prerequisites
- **JDK 21 LTS** or newer
- **Apache Maven 3.8+**
- **Docker & Docker Compose**
- An **OpenRouter API Key** (for LLaMA 3.2 90B)

### 2. Launch the Vector Database

Start PostgreSQL with `pgvector`:

```bash
docker compose up -d
```

### 3. Configure API Credentials

Set your OpenRouter API key as an environment variable:

```bash
# Linux/macOS
export OPENROUTER_API_KEY="your-openrouter-key"

# Windows (PowerShell)
$env:OPENROUTER_API_KEY="your-openrouter-key"
```

### 4. Build & Ingest Data

Compile the project and run the ingestion pipeline:

```bash
mvn clean compile
```

To load `songs.csv` into the vector database, uncomment the ingestion line in `Main.java` or run the loader utility:

```bash
mvn exec:java -Dexec.mainClass="Main"
```

---

## ⚠️ Critical Security & Hygiene Audit Notes

1. **Rotate Leaked API Key**: A hardcoded OpenRouter API key was detected in earlier commits of `src/main/java/service/AiService.java`. It must be revoked immediately on OpenRouter and replaced with `System.getenv("OPENROUTER_API_KEY")`.
2. **Remove Committed Build Artifacts**: The `target/` directory and `.DS_Store` files committed in the repository must be removed from git tracking:
   ```bash
   git rm -r --cached target .DS_Store
   ```
3. **Include Standard `.gitignore`**: Ensure Java, Maven (`target/`), and OS files are excluded from source control.

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).

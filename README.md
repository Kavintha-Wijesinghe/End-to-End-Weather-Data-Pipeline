# 🌦️ End-to-End Weather Data Pipeline  

![Pipeline Architecture](/pipeline_architecture.png)  
*Example: Add your pipeline diagram image to `/assets/pipeline_architecture.png`*

---

### Overview  
This project demonstrates a **fully containerized data engineering pipeline** for ingesting, transforming, and modeling weather data using **open-source technologies**.  
It’s designed for both **learning** and **hands-on experience** in modern data stack tools such as **Apache Airflow, dbt, PostgreSQL, and Docker**.

---

### 🚀 Tech Stack  
| Tool | Purpose |
|------|----------|
| **Docker** | Containerization and environment setup |
| **Apache Airflow** | Workflow orchestration |
| **Python (WeatherStack API)** | Data extraction |
| **PostgreSQL** | Data storage (raw + modeled layers) |
| **dbt (Data Build Tool)** | Data transformation and modeling |

---

### 🧩 Architecture Overview  
The system runs entirely through **Docker Compose** and consists of three main services:

- **`postgres_container`** → PostgreSQL database (schema: `dev`) that stores both raw and transformed data.  
- **`airflow_container`** → Apache Airflow 3 image running helper scripts for API data ingestion.  
- **`dbt_container`** → Executes dbt models to create staging and mart tables in PostgreSQL.

**Persistent Volumes:**  
- `postgres_data` ensures database data is retained between runs.  
- Project directories are mounted for easy local editing.

---

### ⚙️ Prerequisites  
Before running the project, ensure you have:
- Docker & Docker Compose v2 installed  
- Network access for API calls  
- A valid [WeatherStack](https://weatherstack.com/) API key  
  *(A sample key is provided; replace it with your own for real-time data.)*

---

### 🛠️ Setup Guide  

#### 1. Clone the Repository  
```bash
git clone https://github.com/<your-username>/weather-data-pipeline.git
cd weather-data-pipeline/api_request
```

#### 2. Update API Key  
Edit `airflow/api/api_request.py`:
```python
api_key = "<YOUR_WEATHERSTACK_API_KEY>"
```

---

### ▶️ Run the Pipeline  

#### Step 1 — Start PostgreSQL  
```bash
docker compose up -d db
```

#### Step 2 — Ingest Raw Data  
```bash
docker compose run --rm af python /opt/airflow/api/insert_records.py
```
This script will:
- Connect to PostgreSQL as `db_user` / `db_password`  
- Create schema `dev` and table `raw_weather_data` (if not already present)  
- Fetch live data from WeatherStack (or use mock data if API fails)  
- Insert a sample record into the database  

#### Step 3 — Transform Data with dbt  
```bash
docker compose up dbt
```
dbt builds the following models:  
- `dev.stg_weather_report`  
- `dev.daily_average`  
- `dev.weather_report`

#### Step 4 — Inspect Results  
```bash
docker compose exec db psql -U db_user -d db -c "\dt dev.*"
docker compose exec db psql -U db_user -d db -c "SELECT * FROM dev.weather_report;"
```

---

### 📁 Repository Structure  
```
api_request/
├── airflow/
│   ├── api/
│   │   ├── api_request.py      # WeatherStack API client + mock data
│   │   └── insert_records.py   # Creates tables & inserts data
│   └── dags/                   # Placeholder for Airflow DAGs
├── dbt/
│   └── my_project/
│       ├── models/
│       │   ├── staging/stg_weather_report.sql
│       │   ├── marts/daily_average.sql
│       │   └── marts/weather_report.sql
│       ├── models/sources/sources.yml
│       └── profiles (mounted at runtime)
├── docker-compose.yml
└── postgres/
    └── airflow_init.sql        # Seeds Airflow DB/user
```

---

### 🔧 Useful Commands  

| Task | Command |
|------|----------|
| Stop the entire stack | `docker compose down` |
| Reset PostgreSQL data | `docker compose down -v` |
| Remove local volume | `sudo rm -rf api_request/postgres_data` |
| Tail container logs | `docker compose logs db` / `docker compose logs af` / `docker compose logs dbt` |

---

### 🧠 Troubleshooting  

| Issue | Resolution |
|-------|-------------|
| `initdb: directory "/var/lib/postgresql/data" exists but is not empty` | Run `docker compose down -v` before restarting |
| dbt error `relation "dev.raw_weather_data" does not exist` | Make sure to run the ingestion script (`insert_records.py`) |
| WeatherStack HTTP 429 (rate limit) | The script automatically falls back to mock data |

---

### 🚧 Next Steps  
- Build a **scheduled Airflow DAG** to automate ingestion and transformation.  
- Add **dbt tests** for data validation and quality monitoring.  
- Store API keys securely using environment variables or a secrets manager.  

---

### 📚 Learn More  
🔗 **GitHub Repository:** [Weather Data Pipeline](https://lnkd.in/gNRrGcdY)  
Perfect for anyone interested in **data engineering, orchestration, and modern ELT pipelines**.

# databricks-news-lakehouse

End-to-end **Geo-News Lakehouse** platform on Databricks: Medallion Architecture, Lakeflow, CI/CD, and AI-powered RAG assistant.

## Architecture

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  Web API    │──▶│   Bronze    │──▶│   Silver    │──▶│    Gold     │
│  (News)     │    │  (raw JSON) │    │ (cleaned)  │    │ (star sch) │
└─────────────┘    └─────────────┘    └─────────────┘    └──────┬──────┘
                                                             │
┌─────────────┐                                              ▼
│  Neon PG    │──▶ Lakehouse Federation ──▶ AI Annotations ──▶ Vector Search ──▶ RAG
│  (AI annot) │                                      │
└─────────────┘                                      ▼
                                              3D Globe (gold.dim_geo)
```

## Medallion Layers

| Layer  | Purpose                           | Key Tables                              |
|--------|-----------------------------------|-----------------------------------------|
| Bronze | Raw ingestion, schema-on-read     | `bronze.raw_news_events`                |
| Silver | Cleaned, validated, SCD2          | `silver.news_events`, `silver.geo_tags` |
| Gold   | Star schema for analytics & globe | `gold.fact_news`, `gold.dim_geo`        |

## Project Structure

```
databricks-news-lakehouse/
├── databricks.yml
├── resources/
│   ├── lakeflow_pipeline.yml
│   ├── scheduled_jobs.yml
│   └── connections.yml
├── src/
│   ├── ingestion/
│   ├── transformations/
│   └── ai_agent/
├── notebooks/
├── tests/
└── .github/workflows/
```

## Quick Start

```bash
# Deploy to DEV
databricks bundle deploy --target dev

# Run the pipeline
databricks bundle run news_pipeline --target dev

# Run tests
pytest tests/
```

## CI/CD

GitHub Actions pipeline (`.github/workflows/bundle_deploy.yml`) runs:
1. Unit + data-quality tests
2. `databricks bundle deploy --target prod`

## Environments

| Target | Catalog              | Schedule          |
|--------|----------------------|-------------------|
| dev    | `news_lakehouse_dev` | Manual            |
| prod   | `news_lakehouse_prod`| Every 2 hours     |

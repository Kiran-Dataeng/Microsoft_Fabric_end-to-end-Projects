# Microsoft Fabric End-to-End Projects

Portfolio of hands-on Microsoft Fabric / Azure data engineering projects, built while following a structured roadmap from BI support toward a Fabric Data Engineer role — starting from a Microsoft Fabric Data Engineer Associate (DP-700) certification.

## 🧰 Tech stack
- **Microsoft Fabric** — Lakehouse, Warehouse, Pipelines, Notebooks, OneLake
- **Azure Data Factory** — orchestration and data movement
- **Databricks** — data transformation (PySpark, Delta Lake)
- **Power BI** — reporting and dashboards
- **GitHub Actions** — CI/CD (from Project 4 onward)

## 📂 Projects

| # | Project | Focus | Status | Link |
|---|---------|-------|--------|------|
| 0 | Environment Setup | Fabric workspace, tooling, Lakehouse vs Warehouse notes | ✅ Done | [phase0-environment-setup](./phase0-environment-setup) |
| 1 | Basic Ingestion Pipeline | ADF pipeline → Lakehouse Bronze table | ✅ Done | [project1-basic-ingestion](./project1-basic-ingestion) |
| 2 | PySpark Transformation Notebook | Bronze → Silver, schema enforcement, Delta versioning | 🔜 In progress | [project2-pyspark-transform](./project2-pyspark-transform) |
| 3 | Full Medallion + Power BI Report | Silver → Gold, live Power BI report | ⬜ Planned | [project3-medallion-powerbi](./project3-medallion-powerbi) |
| 4 | Incremental Warehouse + CI/CD | Star schema, incremental loading, GitHub Actions | ⬜ Planned | [project4-incremental-cicd](./project4-incremental-cicd) |
| 5 | Capstone — Legacy BI to Fabric Migration | End-to-end migration from legacy BI to Fabric | ⬜ Planned | [project5-capstone-migration](./project5-capstone-migration) |

## 🎓 Background
Built as part of a self-directed roadmap from BI support (SSRS/PBIRS/SSAS/AAS) toward a job-ready Fabric/Azure Data Engineer role, applying skills from the **Microsoft Fabric Data Engineer Associate (DP-700)** certification and currently pursuing **DP-800: SQL AI Developer Associate**.

## 🚀 How to explore
Each project folder has its own README with problem statement, architecture diagram, setup steps, and lessons learned. Start with Project 1 and follow the numbering.

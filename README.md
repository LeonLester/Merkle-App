# Merkle-App

# Databricks Notebook README

"""
# Price Distribution and Item View Analysis

This notebook is implementing the solution for the assignment by Merkle for a Data Engineer position. It performs exploratory data analysis (EDA) on item price distributions and user view events using Spark DataFrames and Databricks visualizations.

## Main Steps

This dataset seems to be data derived from a webstore, for which we are asked to perform an analysis to provide dashboards for the owner (our client).

1. **Fetching the files from S3**
    Steps taken: 
    1. Fetched the 2 csv files from the provided s3 bucket links.
    2. Performed analysis to understand their structure and what they represent.
    3. Passed them raw into my personal S3 bucket in the bronze layer.
        - The S3 bucket configuration was done in DataBricks, using a free tier AWS account and the Free Edition of DataBricks.
2. **Move data to Silver Layer**
    - Maintained all the data from bronze layer
    - Standardized the attribute names in snake_case
    - Inferred the proper attribute types when possible. There are attributes whose value can be either a number or a string. Some can also be null.
    - The event.payload struct was flattened to the attributes contained within it.
    #### Fact table 
    It is derived from the **event** data, which contain the events that give us the information we need for the datamart later on.
    It was partitioned by year and month, filtered for the view_item events, which gives us the traffic data of the users to the items.

3. **Datamart**
    The top_item datamart was created by 



Extras:
The steps that were done to get a better understanding of the item price distributions were:

    1. Visualized the distribution of item prices, including:
        - All prices
        - Prices below 500 (excluding 0)
        - Prices above 500
        - Distinct price counts
        - Price quantiles and bucketization
        - Gaussian fit to price histogram
Notes:
Data seems to somewhat follow a Gaussian/zipfian distribution. 

There are digital how-to manuals that are sold, along with the corresponding items, which skew the price distribution, creating a spike at 0.

    2. Price Bucketing
    - Assigned price buckets to items based on quantiles.

    3. Event Data Analysis
    - Joined item and event data to analyze item view counts and most used platforms.
    - Ranked items by total views per year.


# Open Questions

## 1. What steps are missing to industrialize this solution?

**Data quality & error handling** - Need schema validation, null checks, and business rule validation. Currently bad data would just fail silently or break the pipeline. Should add a quarantine layer for bad records.

**Orchestration** - Running notebooks manually isn't production-ready. Need Airflow or Databricks Workflows for scheduling, dependencies, and retries. Also need incremental loading instead of full refreshes every time.

**Monitoring** - No alerts for failures, data anomalies, or SLA breaches. Should track job execution times, row counts, and data freshness. Send alerts to Slack/email.

**Security** - Missing access controls, audit logging, and encryption. Unity Catalog would help with permissions and lineage tracking.

**Testing** - Zero tests right now. Need unit tests for transformations, integration tests for the full pipeline, and data validation tests.

**Performance** - Partitioning exists but hasn't been tuned. Should enable auto-optimize, add Z-ordering for frequently filtered columns, and monitor for small files issues.

**Documentation** - Code comments aren't enough. Need a data catalog, runbooks for operations, and architecture docs.

## 2. How would the architecture change with dbt-core? What extra resources would we need?

Main difference: dbt only does transformation, not extraction. Need a separate ingestion process to load S3 files into Layer 1 (simple Spark job or tool like Fivetran).

Architecture becomes:
```
S3 → Ingestion job → Layer 1 → dbt → Layer 2 → Layer 3
```

Extra resources needed:
- Something to run dbt (VM or dbt Cloud)
- Orchestrator to schedule dbt runs (Airflow/Databricks Workflows)
- CI/CD pipeline for deployment
- Git repository (dbt requires version control)
- That initial ingestion process

Can use Databricks SQL Warehouses instead of full clusters, which is cheaper for SQL workloads.

## 3. What would dbt-core bring to the project? Pros and cons?

**Pros:**
- SQL-only transformations mean more people can contribute
- Built-in testing framework is solid
- Auto-generated documentation with lineage graphs
- `ref()` function handles dependencies automatically
- Incremental models come out of the box

**Cons:**
- SQL only - complex logic or ML stuff still needs Spark
- Another tool to learn and maintain
- Need separate ingestion tooling
- Can be slower than optimized PySpark for heavy workloads
- Debugging Jinja-generated SQL is annoying

Use dbt if your transformations are mostly SQL-friendly. Stick with Spark if you're doing complex processing or performance is critical.

## 4. Effort estimate for dbt-core implementation

**~2-3 weeks** for this case study.

Week 1:
- Setup dbt project and connections (1-2 days)
- Build S3 ingestion job for Layer 1 (1 day)
- Create Layer 2 staging models (2-3 days)

Week 2:
- Build Layer 3 top_item mart (2 days)
- Testing and validation (1-2 days)
- CI/CD setup and documentation (1-2 days)

Add extra time if:
- Team is new to dbt (+3-5 days learning curve)
- Performance tuning needed (+1-2 days)
- Complex integration issues (+2-3 days)

Ongoing maintenance: few hours per week.
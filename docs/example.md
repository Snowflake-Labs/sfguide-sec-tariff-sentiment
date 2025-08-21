# Vignette 1 - Example

![image_caption](img/headerfinancialgovernance.png)

### Snowflake [Financial Governance Framework](https://docs.snowflake.com/en/user-guide/cost-overview.html#financial-governance-overview)

- Effective Snowflake financial governance is divided into three parts:
  visibility, control, and optimization.
  - **Visibility:** Visibility includes understanding the different sources of
    cost and the ability to explore that cost in detail. Visibility also
    includes attributing cost to the right entities within your organization and
    monitoring costs as they accumulate so you can avoid unexpected costs.
  - **Control:** Snowflake helps you set guardrails and control costs so you
    never spend more than expected. For example, by setting limits on how long a
    query can run before it is terminated, you can avoid unexpected costs
    associated with runaway queries.
  - **Optimiziation:** Optimization begins with identifying areas where
    improvement can reduce costs. Within Snowflake you can use features to
    discover compute usage that might need fine-tuning.

### Script

{{ snowsight_button() }}

To run this vignette, click the **Copy** button in the top right hand corner of
the SQL box below and past into Snowsight. This SQL can be run all at once, or
step by step for more detailed step through:

/// html | div[class="collapse-code"]

```sql
{% include("sql/example.sql") %}
```

///

/// html | div[class="collapse-code"]

```sql
{% include("sql/example-reset.sql") %}
```

///

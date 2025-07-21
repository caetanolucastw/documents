
# Data Quality Checks with `dqx` Library

## Overview

The `dqx` library is designed to perform **health checks** on datasets. It is particularly useful in environments following a **data contract architecture**, where data quality validation is automated and governed.

### Key Components

There are **three main classes** used in the standard workflow:

- **`DQEngine`** – Executes and manages health checks.
- **`DQProfiler`** – Profiles datasets and gathers statistics and behavioral rules.
- **`DQGenerator`** – Generates data quality rules based on the profiling results.

---

## Workflow

### 1. Profiling the Data

The first step is to profile a sample of your dataset using `DQProfiler`. This step outputs:

- **Summary statistics** for each column
- **Profiles** that describe expected behaviors for each column

#### Example Summary Stats

```python
{
  'hvid': {
    'count': 1000,
    'mean': 784.77,
    'stddev': 271.18,
    'min': '250',
    '25%': '624.0',
    '50%': '980.0',
    '75%': '993.0',
    'max': '999',
    'count_non_null': 1000,
    'count_null': 0
  }
}
```

#### Example Profile

```python
DQProfile(name='is_not_null', column='hvid')
```

> Each column will have one **summary stat**, but potentially **multiple profiles**.

---

### 2. Generating Data Quality Rules

Once profiles are created, `DQGenerator` can transform them into data quality rules:

```python
generator = DQGenerator(ws)
health_checks = generator.generate_dq_rules(profiles)
```

- These rules are used to **validate data health**.
- By default, generated rules are marked as **errors**, though they can also be marked as **warnings**.

> ⚠️ Note: Auto-generated checks might not always align with business logic.

---

## Custom Checks

To ensure alignment with your business requirements, you can define **custom health checks** in two ways:

1. **Save them to a Delta table**
2. **Store them in a YAML file**

The **`DQEngine`** class is responsible for saving and running these checks.

---

## Executing Health Checks

After defining the checks, run them using the `DQEngine`. It supports multiple execution modes:

- **Split data into “good” and “bad”** based on check results
- **Attach error/warning columns** to the full dataset
- **Use metadata from Delta table or YAML file**

> You can bring in metadata to apply the checks or define them inline in the workflow.

---

## Example Usage

```python
silver_health_claims = spark.read.table("samples.healthverity.claims_sample_synthetic")

ws = WorkspaceClient()
dq_engine = DQEngine(ws)

profiler = DQProfiler(ws)
summary_stats, profiles = profiler.profile(silver_health_claims)

generator = DQGenerator(ws)
health_checks = generator.generate_dq_rules(profiles)
```

---

## Limitations

- Auto-generated checks may contradict business-specific rules.
- Custom checks with **complex or proprietary logic** might be difficult to express within Delta table constraints.
- YAML allows for more flexibility in defining such logic, but manual or scripted editing is required.

---

## Summary

| Component     | Purpose                                  |
|---------------|------------------------------------------|
| `DQProfiler`  | Extracts column stats and behavioral profiles |
| `DQGenerator` | Transforms profiles into data quality rules |
| `DQEngine`    | Runs checks, saves them to storage, splits or annotates data |

The `dqx` library provides a flexible and powerful foundation for implementing scalable, contract-driven data validation pipelines.

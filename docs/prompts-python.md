# Python Prompts — Production-Grade AI Prompt Templates

> Curated prompts for Python software engineering: APIs, async services, data pipelines, ML, CLIs, and testing. Built on the [CRTSE framework](best-practices.md).

---

## FastAPI Microservices

### New endpoint (CRTSE pattern)
```
Act as a senior Python backend engineer (15 years Python, async microservices specialist).

I'm building a FastAPI service (Python 3.12, FastAPI 0.115+, Pydantic v2, SQLAlchemy 2.0 async, PostgreSQL 16). Architecture: repository pattern with dependency injection.

Task: Create the `POST /api/orders` endpoint for placing a new order.

Requirements:
- All Pydantic models fully typed (no `Any`, no `Optional[str]` — use `str | None`)
- Validate order items list is non-empty at the schema level (no extra db round-trip)
- Use async SQLAlchemy session via FastAPI dependency
- Return proper status codes: 201 on success, 422 on validation, 409 on duplicate
- Include OpenAPI response model annotations
- Use `structlog` for structured logging at the service boundary

Do NOT:
- Use bare `try/except` that swallows exceptions; log and re-raise with context
- Use `asyncio.get_event_loop()` directly (use FastAPI's async lifecycle)
- Return raw SQLAlchemy models in responses (always map through Pydantic schemas)

Our existing endpoint pattern:
```python
@router.post("/api/items", status_code=status.HTTP_201_CREATED)
async def create_item(
    body: ItemCreate,
    db: Annotated[AsyncSession, Depends(get_db)],
    logger: Annotated[structlog.BoundLogger, Depends(get_logger)],
) -> ItemResponse:
    service = ItemService(db, logger)
    return await service.create(body)
```

Output: the complete endpoint with its Pydantic schemas, service method, and any new router registration needed.
```

### Database migration + schema change
```
Act as a senior Python backend engineer.

I'm adding a `discount_tier` enum field (Tier: 'none', 'bronze', 'silver', 'gold') to our existing `users` table. Stack: SQLAlchemy 2.0, Alembic for migrations, PostgreSQL 16.

Task: Generate the Alembic migration and update the Pydantic User schema and User model.

Requirements:
- Migration must be reversible (upgrade() and downgrade() both work)
- Use `ALTER TYPE` to add the enum value in the migration
- Default existing users to 'none'
- The Pydantic UserRead schema must include `discount_tier` in responses
- The UserCreate schema must NOT expose it (it's assigned server-side)

Do NOT:
- Drop and recreate the column in the migration
- Use raw SQL strings (use Alembic's op.* methods)

Output: three files (migration, model update, schema update) with exact file paths matching our project structure.
```

### Async background task worker
```
Act as a senior Python backend engineer specializing in async distributed systems.

I'm building a background task worker using Python 3.12, asyncio, and Redis Streams. Tasks involve generating PDF reports that take 30-120 seconds.

Task: Implement the worker loop that reads from a Redis Stream (`stream:report-gen`), calls the report generation service, and ACKs on success / retries on failure.

Requirements:
- Use `redis.asyncio` for the Redis client
- Claim and process from the consumer group `report-workers`
- Use exponential backoff (1s, 2s, 4s, 8s) with a max 3 retries
- On max retries exceeded, write to a dead-letter stream with the full task payload
- Graceful shutdown on SIGTERM (stop accepting new tasks, finish in-flight, close connections)
- Use `structlog` with task ID in bound context

Do NOT:
- Use `asyncio.sleep` in a while-true loop without checking shutdown flag
- Leak Redis connections (use context managers for all connections)

Output: a single `worker.py` module with the Worker class.
```

---

## Data Engineering & Pipelines

### ETL pipeline with Polars
```
Act as a senior data engineer.

I'm building a daily ETL pipeline that ingests CSV files (500K-2M rows), cleans/transforms them, and writes to Parquet.

Stack: Python 3.12, Polars (lazyframe), S3 (boto3) for source/destination.

Task: Implement the transform step of the pipeline:
- Read a partitioned CSV dataset from S3 into a Polars LazyFrame
- Drop rows where `amount` is null or negative
- Convert `transaction_date` from string to Date type
- Add a `processed_at` timestamp column (UTC)
- Group by `account_id` and compute: total_amount, avg_amount, transaction_count, first_transaction, last_transaction
- Write the result to partitioned Parquet (by account_id) on S3

Requirements:
- Use Polars lazy evaluation throughout (no collect() until write)
- Filter nulls/negatives before aggregation to avoid skewed stats
- Handle S3 paths following our pattern: `s3://bucket/raw/YYYY/MM/DD/*.csv` → `s3://bucket/processed/YYYY/MM/DD/`
- Return a summary: rows_in, rows_out, runtime_seconds

Do NOT:
- Use pandas (we migrated to Polars)
- Read all files into memory at once
- Use boto3 directly for S3 I/O on dataframes (Polars handles S3 natively)

Output: a complete `etl_pipeline.py` module.
```

### ML model serving
```
Act as a senior ML engineer.

I'm deploying a trained XGBoost model for real-time inference behind a FastAPI endpoint.

Task: Implement the inference service:
- Load the model from a pickle file on startup (single load, not per-request)
- Accept a JSON array of feature vectors at `POST /predict`
- Validate input: correct number of features, no NaN/null values, values within expected range
- Run inference and return predictions with probabilities
- Log prediction latency per request

Requirements:
- Model loaded once at startup via FastAPI lifespan event
- Input validation with Pydantic (min/max feature values from the training schema)
- Return predictions as `List[PredictionResult]` with class label and confidence
- Include `X-Request-ID` in structured logs
- Reject requests with >1000 samples (batch limit)

Do NOT:
- Load the model on every request
- Return raw numpy arrays in JSON (serialize to native Python types)
- Block the event loop with synchronous model calls (run in thread pool if needed)

Output: the `inference_service.py` module.
```

---

## CLI Tools

### Click-based CLI with subcommands
```
Act as a senior Python developer.

I'm building a DevOps CLI tool for our platform.

Stack: Python 3.12, Click 8.x, rich (for formatted output).

Task: Implement the `deploy` subcommand with options for environment, version, and rollout strategy.

Requirements:
- `--env` required: one of [dev, staging, production]
- `--version` required: semantic version string validated with a regex
- `--strategy` optional: one of [rolling, blue-green, canary] — default rolling
- `--dry-run` flag: prints what would happen without executing
- Use `rich` for colored, tabular output (table of affected services)
- Production deploys require explicit `--confirm` flag
- Exit codes: 0 on success, 1 on validation error, 2 on deploy failure

Do NOT:
- Use print() — use rich.console
- Hardcode service lists — read from a config file

Output: the deploy command implementation and its test fixtures.
```

---

## Testing

### Pytest fixtures and parametrized tests
```
Act as a senior Python QA engineer.

I have a `calculate_shipping(order: Order, rates: dict[str, float]) -> float` function that computes shipping cost based on order weight, destination zone, and carrier rate table.

Task: Write a comprehensive pytest suite:
- Fixtures for: valid orders (single item, multi-item, oversized), rate tables, edge cases
- Parametrize tests for: each destination zone, weight boundaries (0, 1g, 1kg, 50kg), each carrier
- Test zero weight raises ValueError
- Test negative weight raises ValueError
- Test unknown destination zone raises KeyError with a descriptive message
- Test that shipping cost is always non-negative
- Property-based test: for any valid input, shipping cost ≤ max rate * weight
- Coverage target: 100% branch coverage

Do NOT:
- Write tests that depend on test execution order
- Use MagicMock when a simple stub/fake would work

Output: a `test_shipping.py` file using pytest with fixtures and @pytest.mark.parametrize.
```
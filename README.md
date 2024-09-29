# RoadRunner-Iceberg

# Transferring Data from Kafka to Apache Iceberg Data Lake

## Process

1. **Use Apache Spark with Iceberg Connector**:
   - Leverage Spark's Structured Streaming to read from Kafka topics.
   - Use Iceberg's Spark integration to write data to Iceberg tables.

2. **Implement a Spark Streaming Job**:
   - Create a Spark Streaming job that reads from your Kafka topics.
   - Configure the job to write to Iceberg tables in MinIO.

3. **Data Partitioning**:
   - Implement a partitioning strategy in Iceberg based on your data characteristics (e.g., date, device_id).

4. **Schema Evolution**:
   - Utilize Iceberg's schema evolution capabilities to handle changes in your data structure over time.

5. **Metadata Management**:
   - Use Iceberg's metadata management to track table versions and enable time travel queries.

## Benefits of This Approach

1. **Unified Batch and Streaming**: Iceberg supports both batch and streaming writes, allowing you to use a single format for all your data.

2. **ACID Transactions**: Iceberg provides ACID guarantees, ensuring data consistency even with concurrent writes.

3. **Schema Evolution**: Easily adapt to changing data structures without needing to rewrite entire datasets.

4. **Time Travel**: Access historical versions of your data for auditing or rollback purposes.

5. **Optimized for Analytics**: Iceberg's format is optimized for analytical queries, improving query performance.

6. **Integration with Existing Tools**: Iceberg works well with your current stack (Spark, MinIO) and can integrate with tools like Clickhouse for querying.

7. **Scalability**: Designed to handle large-scale data storage and processing efficiently.

## Considering DVC or LakeFS

While DVC (Data Version Control) and LakeFS are valuable tools, they serve different purposes than Iceberg:

- **DVC**: Primarily used for versioning ML models and datasets, not typically used for large-scale data lake management.
- **LakeFS**: Provides Git-like operations for data lakes but adds an additional layer of complexity.

For your current setup, Apache Iceberg with MinIO should suffice without needing DVC or LakeFS. Iceberg already provides versioning and ACID transactions, which cover many use cases these tools address.

## MLflow Data Source: Data Lake vs Data Warehouse

The choice between using the Data Lake (Iceberg) or Data Warehouse (Clickhouse) for MLflow depends on your specific use case:

1. **Use Data Lake (Iceberg) when**:
   - You need access to raw, unprocessed data for feature engineering.
   - You're working with large datasets that benefit from Iceberg's optimized storage format.
   - You need to leverage time travel capabilities for model reproducibility.

2. **Use Data Warehouse (Clickhouse) when**:
   - You need pre-aggregated or transformed data for faster model training.
   - Your ML models work with structured, analytics-ready data.
   - You require real-time features computed from streaming data.

Best Practice: Use a combination of both:
1. Store raw data in the Data Lake (Iceberg).
2. Process and aggregate data into Clickhouse for fast analytics and feature serving.
3. Use Clickhouse for real-time feature computation and serving.
4. Access the Data Lake for batch feature engineering and when you need to work with raw historical data.

This approach gives you the flexibility to handle various ML use cases efficiently.


# Iceberg-Spark Connectivity Test

This project contains a Python script to test the connectivity between Apache Spark and Apache Iceberg, using MinIO as the S3-compatible object storage.

## Prerequisites

- Apache Spark 3.5.2
- MinIO server running and accessible
- Python 3.x

## Setup

1. Ensure you have a MinIO server running with a bucket named `test`.

2. Make sure you have the correct MinIO credentials (access key and secret key).

3. Install the required Python packages:
   ```
   pip install pyspark==3.5.2
   ```

4. Save the `iceberg_spark_connectivity_test.py` script in your project directory.

## Running the Test

Execute the following command to run the connectivity test:

```bash
spark-submit --packages org.apache.iceberg:iceberg-spark-runtime-3.5_2.12:1.4.2,org.apache.hadoop:hadoop-aws:3.3.4,com.amazonaws:aws-java-sdk-bundle:1.12.262 --conf spark.sql.extensions=org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions --conf spark.sql.catalog.iceberg=org.apache.iceberg.spark.SparkCatalog --conf spark.sql.catalog.iceberg.type=hadoop --conf spark.sql.catalog.iceberg.warehouse=s3a://test/warehouse iceberg_spark_connectivity_test.py
```

## What the Test Does

1. Creates a Spark session with the necessary configurations for Iceberg and MinIO.
2. Attempts to create a test table in Iceberg.
3. Inserts a sample record into the table.
4. Reads back the inserted data.
5. Drops the test table.

## Expected Output

If the test is successful, you should see output similar to this:

```
Successfully created test table. Connectivity test passed!
Found table: test_connectivity
Inserted test data successfully.
Read data: id=1, name=Test Name
Test table dropped successfully.
```

## Troubleshooting

If you encounter issues:

1. Verify MinIO is accessible and the `test` bucket exists.
2. Check network connectivity between Spark and MinIO.
3. Ensure the MinIO credentials are correct.
4. Verify that the Spark and Iceberg versions are compatible.

## Integration with CI/CD

To integrate this test into your CI/CD pipeline:

1. Add the `iceberg_spark_connectivity_test.py` script to your repository.
2. In your CI/CD configuration (e.g., `.gitlab-ci.yml`, `.github/workflows/main.yml`), add a step to run the test:

   ```yaml
   test_iceberg_spark_connectivity:
     stage: test
     script:
       - spark-submit --packages org.apache.iceberg:iceberg-spark-runtime-3.5_2.12:1.4.2,org.apache.hadoop:hadoop-aws:3.3.4,com.amazonaws:aws-java-sdk-bundle:1.12.262 --conf spark.sql.extensions=org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions --conf spark.sql.catalog.iceberg=org.apache.iceberg.spark.SparkCatalog --conf spark.sql.catalog.iceberg.type=hadoop --conf spark.sql.catalog.iceberg.warehouse=s3a://test/warehouse iceberg_spark_connectivity_test.py
   ```

3. Ensure your CI/CD environment has Spark 3.5.2 installed and can access your MinIO server.

## Maintenance

Keep the following updated as your project evolves:

- Spark version
- Iceberg version
- MinIO configuration
- Any changes to the test script or Spark configurations

Regular testing and updates will ensure continued compatibility and functionality of your Iceberg-Spark setup.
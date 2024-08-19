# intro-to-open-data-lakehouse-with-presto-iceberg-and-minio

[Slides](https://prestodb.io/wp-content/uploads/Presto-Iceberg-Workshop-virtual-slides.pdf)

[Github Repo](https://github.com/IBM/presto-iceberg-lab)

[Workshop step by step guide](https://ibm.github.io/presto-iceberg-lab/)

[Workshop Recording](https://youtu.be/Aj9nSRGI4Ls?si=LNPhRjRogqxYy_cQ)

```bash
git clone https://github.com/IBM/presto-iceberg-lab.git
cd presto-iceberg-lab
```

```bash
cd conf
docker compose build
docker image list
docker compose up -d
docker logs --tail 100 minio
docker logs --tail 50 hive-metastore
docker logs --tail 100 presto-coordinator
```

`Create a bucket named test-bucket in minio.`

```bash
docker exec -it presto-coordinator presto-cli
presto> show catalogs;
presto> CREATE SCHEMA iceberg.minio with (location = 's3a://test-bucket/');
presto> USE iceberg.minio;
presto:minio> CREATE TABLE books (id bigint, title varchar, author varchar) WITH (location = 's3a://test-bucket/minio/books');
presto:minio> INSERT INTO books VALUES (1, 'Pride and Prejudice', 'Jane Austen'), (2, 'To Kill a Mockingbird', 'Harper Lee'), (3, 'The Great Gatsby', 'F. Scott Fitzgerald');
presto:minio> SELECT * FROM books;
presto:minio> SELECT * FROM "books$history";
presto:minio> SELECT * FROM "books$snapshots";
presto:minio> SELECT * FROM "books$manifests";
presto:minio> SELECT * FROM "books$files";
presto:minio> ALTER TABLE books ADD COLUMN checked_out boolean;
presto:minio> INSERT INTO books VALUES (4, 'One Hundred Years of Solitude', 'Gabriel Garcia Marquez', true);
presto:minio> SELECT * FROM "books$snapshots";
presto:minio> SELECT snapshot_id, committed_at FROM "books$snapshots" ORDER BY committed_at;
presto:minio> SELECT * FROM books FOR VERSION AS OF 7120201811871583704;
presto:minio> SELECT * FROM books FOR TIMESTAMP AS OF TIMESTAMP '2023-12-04 03:22:51.700 UTC';
presto:minio> CALL iceberg.system.rollback_to_snapshot('minio', 'books', 7120201811871583704);
presto:minio> SELECT * FROM books;
```
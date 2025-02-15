### See it in action

You can test out this extension right away!

1.  Go to your [Cloud Firestore dashboard](https://console.firebase.google.com/project/${param:BIGQUERY_PROJECT_ID}/firestore/data) in the Firebase console.

1.  If it doesn't already exist, create the collection you specified during installation: `${param:COLLECTION_PATH}`

1.  Create a document in the collection called `bigquery-mirror-test` that contains any fields with any values that you'd like.

1.  Go to the [BigQuery web UI](https://console.cloud.google.com/bigquery?project=${param:BIGQUERY_PROJECT_ID}&p=${param:BIGQUERY_PROJECT_ID}&d=${param:DATASET_ID}) in the Google Cloud Platform console.

1.  Query your **raw changelog table**, which should contain a single log of creating the `bigquery-mirror-test` document.

    ```
    SELECT *
    FROM `${param:BIGQUERY_PROJECT_ID}.${param:DATASET_ID}.${param:TABLE_ID}_raw_changelog`
    ```

1.  Query your **latest view**, which should return the latest change event for the only document present -- `bigquery-mirror-test`.

    ```
    SELECT *
    FROM `${param:BIGQUERY_PROJECT_ID}.${param:DATASET_ID}.${param:TABLE_ID}_raw_latest`
    ```

1.  Delete the `bigquery-mirror-test` document from [Cloud Firestore](https://console.firebase.google.com/project/${param:BIGQUERY_PROJECT_ID}/firestore/data).
    The `bigquery-mirror-test` document will disappear from the **latest view** and a `DELETE` event will be added to the **raw changelog table**.

1.  You can check the changelogs of a single document with this query:

    ```
    SELECT *
    FROM `${param:BIGQUERY_PROJECT_ID}.${param:DATASET_ID}.${param:TABLE_ID}_raw_changelog`
    WHERE document_name = "bigquery-mirror-test"
    ORDER BY TIMESTAMP ASC
    ```

### Using the extension

Whenever a document is created, updated, imported, or deleted in the specified collection, this extension sends that update to BigQuery. You can then run queries on this mirrored dataset which contains the following resources:

- **raw changelog table:** [`${param:TABLE_ID}_raw_changelog`](https://console.cloud.google.com/bigquery?project=${param:BIGQUERY_PROJECT_ID}&p=${param:BIGQUERY_PROJECT_ID}&d=${param:DATASET_ID}&t=${param:TABLE_ID}_raw_changelog&page=table)
- **latest view:** [`${param:TABLE_ID}_raw_latest`](https://console.cloud.google.com/bigquery?project=${param:BIGQUERY_PROJECT_ID}&p=${param:BIGQUERY_PROJECT_ID}&d=${param:DATASET_ID}&t=${param:TABLE_ID}_raw_latest&page=table)

To review the schema for these two resources, click the **Schema** tab for each resource in BigQuery.

Note that this extension only listens for _document_ changes in the collection, but not changes in any _subcollection_. You can, though, install additional instances of this extension to specifically listen to a subcollection or other collections in your database. Or if you have the same subcollection across documents in a given collection, you can use `{wildcard}` notation to listen to all those subcollections (for example: `chats/{chatid}/posts`).

Enabling wildcard references will provide an additional STRING based column. The resulting JSON field value references any wildcards that are included in ${param:COLLECTION_PATH}. You can extract them using [JSON_EXTRACT_SCALAR](https://cloud.google.com/bigquery/docs/reference/standard-sql/json_functions#json_extract_scalar).


`Partition` settings cannot be updated on a pre-existing table, if these options are required then a new table must be created.

`Clustering` will not need to create or modify a table when adding clustering options, this will be updated automatically.

### Configuring Alternative BigQuery Project

When defining a specific BigQuery project, a manual step to set up permissions is required:

1. Navigate to https://console.cloud.google.com/iam-admin/iam?project=${param:BIGQUERY_PROJECT_ID}
2. Add the **BigQuery Data Editor** role to the following service account:
   `ext-${param:EXT_INSTANCE_ID}@${param:PROJECT_ID}.iam.gserviceaccount.com`.

### _(Optional)_ Import existing documents

If you chose not to automatically import existing documents when you installed this extension, you can backfill your BigQuery dataset with all the documents in your collection using the import script.

If you don't either enable automatic import or run the import script, the extension only exports the content of documents that are created or changed after installation.

The import script can read all existing documents in a Cloud Firestore collection and insert them into the raw changelog table created by this extension. The script adds a special changelog for each document with the operation of `IMPORT` and the timestamp of epoch. This is to ensure that any operation on an imported document supersedes the `IMPORT`.

**Important:** Run the import script over the entire collection _after_ installing this extension, otherwise all writes to your database during the import might be lost.

Learn more about using the import script to [backfill your existing collection](https://github.com/firebase/extensions/blob/master/firestore-bigquery-export/guides/IMPORT_EXISTING_DOCUMENTS.md).

### _(Optional)_ Generate schema views

After your data is in BigQuery, you can use the schema-views script (provided by this extension) to create views that make it easier to query relevant data. You only need to provide a JSON schema file that describes your data structure, and the schema-views script will create the views.

Learn more about using the schema-views script to [generate schema views](https://github.com/firebase/extensions/blob/master/firestore-bigquery-export/guides/GENERATE_SCHEMA_VIEWS.md).

### Monitoring

As a best practice, you can [monitor the activity](https://firebase.google.com/docs/extensions/manage-installed-extensions#monitor) of your installed extension, including checks on its health, usage, and logs.

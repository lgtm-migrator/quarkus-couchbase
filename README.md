# Quarkus Couchbase Extension
Integrates Couchbase into Quarkus.

This extension is currently in alpha status.  It supports:

- Dependency injecting a Couchbase `Cluster`.
- Configuring the Cluster through `application.properties`.  Currently a minimal set of configuration options is provided.
- Graal/native-image, though so far it has been minimally tested with basic cases.

Please try it out and provide feedback, ideas and bug reports.

## Usage
Add it to your project:
```
<dependency>
  <groupId>com.couchbase</groupId>
  <artifactId>quarkus-couchbase</artifactId>
  <version>1.0.0-alpha.1</version>
</dependency>
```

Provide the Couchbase configuration in `application.properties`:
```
quarkus.couchbase.connection-string=localhost
quarkus.couchbase.username=username
quarkus.couchbase.password=password
```

Now you can @Inject a Couchbase `Cluster` into your project:

```
@Path("/couchbase")
public class TestCouchbaseResource {
    @Inject
    Cluster cluster;

    @GET
    @Produces(MediaType.TEXT_PLAIN)
    @Path("/test")
    public String run() {
        // Get a reference to a particular Couchbase bucket and its default collection
        var bucket = cluster.bucket("travel-sample");
        var collection = bucket.defaultCollection() ;

        // Upsert a new document
        collection.upsert("test", JsonObject.create().put("foo", "bar"));

        // Fetch and print a document
        var doc = bucket.defaultCollection().get("test");
        System.out.println("Got doc " + doc.contentAsObject().toString());

        // Perform a N1QL query
        var queryResult = cluster.query("select * from `travel-sample` where url like 'http://marriot%' and country = 'United States';");

        queryResult.rowsAsObject().forEach(row -> {
            System.out.println(row.toString());
        });

        return "success!";
    }
}
```

And test http://localhost:8080/couchbase/test.

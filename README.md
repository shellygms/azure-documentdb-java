#Microsoft Azure DocumentDB Java SDK

![](https://img.shields.io/github/release/azure/azure-documentdb-java.svg)
![](https://img.shields.io/maven-central/v/com.microsoft.azure/azure-documentdb.svg)
![](https://img.shields.io/github/issues/azure/azure-documentdb-java.svg)

This project provides a client library in Java that makes it easy to interact with Azure DocumentDB. For documentation please see the Microsoft Azure [Java Developer Center](http://azure.microsoft.com/en-us/develop/java/) and the [JavaDocs](http://dl.windowsazure.com/documentdb/javadoc).

##Download
###Option 1: Via Maven

To get the binaries of this library as distributed by Microsoft, ready for use within your project, you can use Maven.

    <dependency>
    	<groupId>com.microsoft.azure</groupId>
    	<artifactId>azure-documentdb</artifactId>
    	<version>1.8.0</version>
    </dependency>

###Option 2: Source Via Git

To get the source code of the SDK via git just type:

    git clone git://github.com/Azure/azure-documentdb-java.git

###Option 3: Source Zip

To download a copy of the source code, click "Download ZIP" on the right side of the page or click [here](https://github.com/Azure/azure-documentdb-java/archive/master.zip).


##Minimum Requirements
* Java Development Kit 7
* (Optional) Maven

### Dependencies
* Apache Commons Lang 3.3.2 (org.apache.commons / commons-lang3 / 3.3.2)
* Apache HttpClient 4.2.5 (org.apache.httpcomponents / httpclient / 4.2.5)
* Apache HttpCore 4.2.5 (org.apache.httpcomponents / httpcore / 4.2.5)
* Jackson Data Mapper 1.8 (org.codehaus.jackson / jackson-mapper-asl / 1.8.5)
* JSON 20140107 (org.json / json / 20140107)
* JUnit 4.11 (junit / junit / 4.11)
* mockito 1.10.19 (org.mockito / mockito-core / 1.10.19)

Dependencies will be added automatically if Maven is used. Otherwise, please download the jars and add them to your build path. 

##Usage

To use this SDK to call Azure DocumentDB, you need to first [create an account](http://azure.microsoft.com/en-us/documentation/articles/documentdb-create-account/).

You can follow this [tutorial](http://azure.microsoft.com/en-us/documentation/articles/documentdb-java-application/) to help you get started.

```java
import java.util.Collection;

import com.google.gson.Gson;
import com.microsoft.azure.documentdb.ConnectionPolicy;
import com.microsoft.azure.documentdb.ConsistencyLevel;
import com.microsoft.azure.documentdb.Database;
import com.microsoft.azure.documentdb.Document;
import com.microsoft.azure.documentdb.DocumentClient;
import com.microsoft.azure.documentdb.DocumentClientException;
import com.microsoft.azure.documentdb.DocumentCollection;

public class SampleApp {
    // Replace with your DocumentDB end point and master key.
    private static final String END_POINT = "[YOUR_ENDPOINT_HERE]";
    private static final String MASTER_KEY = "[YOUR_KEY_HERE]";

    // Define an id for your database and collection
    private static final String DATABASE_ID = "TestDB";
    private static final String COLLECTION_ID = "TestCollection";
    private static final String COLLECTION_ID_PARTITIONED = "TestCollection_Partitioned";

    // We'll use Gson for POJO <=> JSON serialization for this sample.
    // Codehaus' Jackson is another great POJO <=> JSON serializer.
    private static Gson gson = new Gson();

    public static void main(String[] args) throws DocumentClientException {
        // Instantiate a DocumentClient w/ your DocumetnDB Endpoint and AuthKey.
        DocumentClient documentClient = new DocumentClient(END_POINT,
                MASTER_KEY, ConnectionPolicy.GetDefault(),
                ConsistencyLevel.Session);

        // Define a new database using the id above.
        Database myDatabase = new Database();
        myDatabase.setId(DATABASE_ID);

        // Create a new database.
        myDatabase = documentClient.createDatabase(myDatabase, null)
                .getResource();

        // Define a new collection using the id above.
        DocumentCollection myCollection = new DocumentCollection();
        myCollection.setId(COLLECTION_ID);

        // Set the provisioned throughput for this collection to be 1000 RUs.
        RequestOptions requestOptions = new RequestOptions();
        requestOptions.setOfferThroughput(1000);

        // Create a new collection.
        myCollection = documentClient.createCollection(
                myDatabase.getSelfLink(), myCollection, requestOptions).getResource();

        // Create an object, serialize it in to JSON, and wrap it in to a
        // document.
        SomePojo somePojo = new SomePojo();
        String somePojoJson = gson.toJson(somePojo);
        Document myDocument = new Document(somePojoJson);

        // Create a new document.
        myDocument = documentClient.createDocument(myCollection.getSelfLink(),
                myDocument, null, false).getResource();
                
		 // The following code Illustrates how to create a partitioned collection and
		 // use the partition key to access documents.
		    
		 // Create a partition key definition that specifies the path to the property
		 // within a document that is used as the partition key.          
        PartitionKeyDefinition partitionKeyDef = new PartitionKeyDefinition();
        ArrayList<String> paths = new ArrayList<String>();
        paths.add("/id");
        partitionKeyDef.setPaths(paths);
        
        // Create a collection with the partition key definition and set the offer throughput
        // to 10100 RU per second.
        DocumentCollection myPartitionedCollection = new DocumentCollection();
        myPartitionedCollection.setId(COLLECTION_ID_PARTITIONED);
        myPartitionedCollection.setPartitionKey(partitionKeyDef);
        		              
        RequestOptions options = new RequestOptions();
        options.setOfferThroughput(10100);
        myPartitionedCollection = documentClient.createCollection(
        	myDatabase.getSelfLink(), myCollection, options).getResource();

        // Insert a document into the created collection.
        String document = "{ 'id': 'document1', 'description': 'this is a test document.' }";
        Document newDocument = new Document(document);
        newDocument = documentClient.createDocument(myPartitionedCollection.getSelfLink(),
                newDocument, null, false).getResource();
                
		 // Read the created document, specifying the required partition key in RequestOptions.
        options = new RequestOptions();
        options.setPartitionKey(new PartitionKey("document1"));
        newDocument = documentClient.readDocument(newDocument.getSelfLink(), options).getResource();
    }
}
```

Additional samples are provided in the unit tests.

##Need Help?

Be sure to check out the Microsoft Azure [Developer Forums on MSDN](https://social.msdn.microsoft.com/forums/azure/en-US/home?forum=AzureDocumentDB) or the [Developer Forums on Stack Overflow](http://stackoverflow.com/questions/tagged/azure-documentdb) if you have trouble with the provided code.

##Contribute Code or Provide Feedback

If you would like to become an active contributor to this project please follow the instructions provided in [Azure Projects Contribution Guidelines](http://azure.github.io/guidelines.html).

If you encounter any bugs with the library please file an issue in the [Issues](https://github.com/Azure/azure-documentdb-java/issues) section of the project.

##Learn More

* [Azure Developer Center](http://azure.microsoft.com/en-us/develop/java/)
* [Azure DocumentDB Service](http://azure.microsoft.com/en-us/documentation/services/documentdb/)
* [Azure DocumentDB Team Blog](http://blogs.msdn.com/b/documentdb/)
* [JavaDocs](http://dl.windowsazure.com/documentdb/javadoc)

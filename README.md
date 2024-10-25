
The Courses
==============================


Here's the new "Learn Spring" course: <br/>
**[>> LEARN SPRING - THE MASTER CLASS](https://www.baeldung.com/learn-spring-course?utm_source=github&utm_medium=social&utm_content=tutorials&utm_campaign=ls#master-class)**

Here's the Master Class of "REST With Spring" (along with the new announced Boot 2 material): <br/>
**[>> THE REST WITH SPRING - MASTER CLASS](https://www.baeldung.com/rest-with-spring-course?utm_source=github&utm_medium=social&utm_content=tutorials&utm_campaign=rws#master-class)**

And here's the Master Class of "Learn Spring Security": <br/>
**[>> LEARN SPRING SECURITY - MASTER CLASS](https://www.baeldung.com/learn-spring-security-course?utm_source=github&utm_medium=social&utm_content=tutorials&utm_campaign=lss#master-class)**



Java and Spring Tutorials
================

This project is **a collection of small and focused tutorials** - each covering a single and well defined area of development in the Java ecosystem. 
A strong focus of these is, of course, the Spring Framework - Spring, Spring Boot and Spring Security. 
In additional to Spring, the modules here are covering a number of aspects in Java. 


Building the project
====================
To do the full build, do: `mvn clean install`


Building a single module
====================
To build a specific module run the command: `mvn clean install` in the module directory


Running a Spring Boot module
====================
To run a Spring Boot module run the command: `mvn spring-boot:run` in the module directory


Working with the IDE
====================
This repo contains a large number of modules. 
When you're working with an individual module, there's no need to import all of them (or build all of them) - you can simply import that particular module in either Eclipse or IntelliJ. 


Running Tests
=============
The command `mvn clean install` will run the unit tests in a module.
To run the integration tests, use the command `mvn clean install -Pintegration-lite-first`


```
-- Step 1: Create mapping tables
CREATE TABLE cluster_mapping (
    old_cluster_id NUMBER,
    new_cluster_id NUMBER
);

CREATE TABLE environment_mapping (
    old_environment_id NUMBER,
    new_environment_id NUMBER
);

CREATE TABLE datacentre_mapping (
    old_datacentre_id NUMBER,
    new_datacentre_id NUMBER
);

-- Step 2: Populate mapping tables
INSERT INTO cluster_mapping (old_cluster_id, new_cluster_id)
SELECT old.cluster_id, new.cluster_id
FROM CLUSTER old
JOIN CLUSTER_V2 new ON old.cluster_name = new.cluster_name; -- Adjust the join condition as per your logic

INSERT INTO environment_mapping (old_environment_id, new_environment_id)
SELECT old.environment_id, new.environment_id
FROM ENVIRONMENT old
JOIN ENVIRONMENT_V2 new ON old.environment_name = new.environment_name; -- Adjust the join condition as per your logic

INSERT INTO datacentre_mapping (old_datacentre_id, new_datacentre_id)
SELECT old.datacentre_id, new.datacentre_id
FROM DATACENTRE old
JOIN DATACENTRE_V2 new ON old.datacentre_name = new.datacentre_name; -- Adjust the join condition as per your logic

-- Step 3: Migrate data to DEPLOYMENT_V2
INSERT INTO DEPLOYMENT_V2 (deployment_id, cluster_id, environment_id, datacentre_id, other_columns) -- Replace with actual column names
SELECT
    d.deployment_id,
    cm.new_cluster_id,
    em.new_environment_id,
    dm.new_datacentre_id,
    d.other_columns -- Replace with actual column names
FROM
    DEPLOYMENT d
JOIN
    cluster_mapping cm ON d.cluster_id = cm.old_cluster_id
JOIN
    environment_mapping em ON d.environment_id = em.old_environment_id
JOIN
    datacentre_mapping dm ON d.datacentre_id = dm.old_datacentre_id;



-- Step 1: Add columns to the version table
ALTER TABLE version ADD (instances NUMBER, capacity VARCHAR2(2));

-- Step 2: Migrate the first non-null occurrence of instances and capacity from the capacity table to the version table
MERGE INTO version v
USING (
    SELECT
        version_id,
        instances,
        capacity
    FROM (
        SELECT
            version_id,
            instances,
            capacity,
            ROW_NUMBER() OVER (PARTITION BY version_id ORDER BY (CASE WHEN instances IS NOT NULL THEN 0 ELSE 1 END), (CASE WHEN capacity IS NOT NULL THEN 0 ELSE 1 END)) AS rn
        FROM
            capacity
    )
    WHERE rn = 1
) rc
ON (v.id = rc.version_id)
WHEN MATCHED THEN
UPDATE SET
    v.instances = rc.instances,
    v.capacity = rc.capacity;

-- Step 3: Drop the capacity table
DROP TABLE capacity;

```
## store proc
```
CREATE OR REPLACE PROCEDURE GetValidAPIVersions(
    p_env_type IN VARCHAR2
)
IS
    TYPE VersionRec IS RECORD (
        api_id NUMBER,
        api_name VARCHAR2(255),
        api_version_id NUMBER,
        version VARCHAR2(255)
    );

    TYPE VersionTable IS TABLE OF VersionRec;

    v_versions VersionTable;

BEGIN
    -- Fetch data into the collection
    WITH ActiveAPIVersions AS (
        SELECT
            av.ID AS API_VERSION_ID,
            av.VERSION,
            av.API_ID,
            SUBSTR(av.VERSION, 1, INSTR(av.VERSION, '-') - 1) AS APIVERSION
        FROM
            API_VERSION av
        JOIN
            API a ON av.API_ID = a.ID
        WHERE
            av.DELETED = 'FALSE'
            AND a.DELETE = 'FALSE'
    ),
    ValidAPIVersions AS (
        SELECT
            av.API_VERSION_ID,
            av.VERSION,
            av.API_ID,
            av.APIVERSION
        FROM
            ActiveAPIVersions av
        LEFT JOIN (
            SELECT
                API_ID,
                APIVERSION
            FROM
                ActiveAPIVersions
            GROUP BY
                API_ID,
                APIVERSION
            HAVING
                COUNT(API_VERSION_ID) > 1
        ) av2 ON av.API_ID = av2.API_ID AND av.APIVERSION = av2.APIVERSION
        WHERE
            av2.API_ID IS NULL
    )
    SELECT
        a.ID AS api_id,
        a.NAME AS api_name,
        av.API_VERSION_ID AS api_version_id,
        av.VERSION AS version
    BULK COLLECT INTO v_versions
    FROM
        ValidAPIVersions av
    JOIN
        DEPLOYMENT_HISTORY dh ON av.API_VERSION_ID = dh.API_VERSION_ID
    JOIN
        ENVIRONMENT e ON dh.ENV_ID = e.ID
    JOIN
        API a ON av.API_ID = a.ID
    WHERE
        e.TYPE = p_env_type
        AND av.VERSION NOT LIKE '%-1.14';

    -- Process the collection
    FOR i IN 1 .. v_versions.COUNT LOOP
        DBMS_OUTPUT.PUT_LINE('API ID: ' || v_versions(i).api_id || 
                             ', API Name: ' || v_versions(i).api_name || 
                             ', API Version ID: ' || v_versions(i).api_version_id || 
                             ', Version: ' || v_versions(i).version);
    END LOOP;
END GetValidAPIVersions;
/
```

To define the WireMock stub mappings in JSON format for handling dynamic fields like `balance` and `itemCount`, you can use the `transformerParameters` field to specify the custom parameters for each endpoint. Here is how you can set up the stub mappings in JSON:

1. **Balance Endpoint Mapping**:
   ```json
   {
     "request": {
       "method": "POST",
       "url": "/transaction"
     },
     "response": {
       "status": 200,
       "body": "{\"balance\": 0}",
       "transformers": ["custom-response-transformer"],
       "transformerParameters": {
         "dynamicField": "balance"
       },
       "headers": {
         "Content-Type": "application/json"
       }
     }
   }
   ```

2. **Item Endpoint Mapping**:
   ```json
   {
     "request": {
       "method": "POST",
       "url": "/item"
     },
     "response": {
       "status": 200,
       "body": "{\"itemCount\": 0}",
       "transformers": ["custom-response-transformer"],
       "transformerParameters": {
         "dynamicField": "itemCount"
       },
       "headers": {
         "Content-Type": "application/json"
       }
     }
   }
   ```

### Full JSON Configuration Example
Here's a full example with both mappings included:

```json
[
  {
    "request": {
      "method": "POST",
      "url": "/transaction"
    },
    "response": {
      "status": 200,
      "body": "{\"balance\": 0}",
      "transformers": ["custom-response-transformer"],
      "transformerParameters": {
        "dynamicField": "balance"
      },
      "headers": {
        "Content-Type": "application/json"
      }
    }
  },
  {
    "request": {
      "method": "POST",
      "url": "/item"
    },
    "response": {
      "status": 200,
      "body": "{\"itemCount\": 0}",
      "transformers": ["custom-response-transformer"],
      "transformerParameters": {
        "dynamicField": "itemCount"
      },
      "headers": {
        "Content-Type": "application/json"
      }
    }
  }
]
```

### Loading JSON Mappings
To load these mappings into WireMock, save the JSON configuration to files in the `mappings` directory of your WireMock setup. For example:

- Save the balance mapping as `transaction-mapping.json`.
- Save the item mapping as `item-mapping.json`.

WireMock will automatically load these mappings when it starts.

### Running WireMock
Ensure your `WireMockExample` class includes the following code to start the WireMock server and load the extensions:

```java
import com.github.tomakehurst.wiremock.WireMockServer;

public class WireMockExample {
    public static void main(String[] args) {
        WireMockServer wireMockServer = new WireMockServer();
        wireMockServer.start();

        // Register the custom transformer
        wireMockServer.getOptions().extensions(new CustomResponseTransformer());

        Runtime.getRuntime().addShutdownHook(new Thread(wireMockServer::stop));
        System.out.println("WireMock server running. Press Ctrl+C to stop.");
        try {
            Thread.sleep(Long.MAX_VALUE);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```

With this setup, the WireMock server will dynamically update and return the `balance` and `itemCount` based on the incoming POST requests to `/transaction` and `/item` respectively.


Yes, you can add custom fields to WireMock mappings and use them in a custom extension to handle dynamic values like balance and item counts. Here’s how you can achieve this:

1. **Extend WireMock with a Custom ResponseTransformer**:
   - Add fields to your mappings that indicate dynamic values.
   - Implement a custom `ResponseTransformer` to process these fields and update the state accordingly.

### Step-by-Step Implementation

1. **Add WireMock to Your Project**:
   Ensure you have WireMock included in your project's dependencies. For Maven:
   ```xml
   <dependency>
       <groupId>com.github.tomakehurst</groupId>
       <artifactId>wiremock-jre8</artifactId>
       <version>2.31.0</version>
       <scope>test</scope>
   </dependency>
   ```

2. **Create a Custom ResponseTransformer**:
   This class will handle the dynamic fields and update the state.

   ```java
   import com.github.tomakehurst.wiremock.extension.Parameters;
   import com.github.tomakehurst.wiremock.extension.ResponseDefinitionTransformer;
   import com.github.tomakehurst.wiremock.http.Request;
   import com.github.tomakehurst.wiremock.http.ResponseDefinition;
   import com.github.tomakehurst.wiremock.client.ResponseDefinitionBuilder;
   import com.github.tomakehurst.wiremock.common.FileSource;
   import com.github.tomakehurst.wiremock.stubbing.StubMapping;

   import java.util.concurrent.atomic.AtomicInteger;
   import java.util.Map;
   import java.util.concurrent.ConcurrentHashMap;

   public class CustomResponseTransformer extends ResponseDefinitionTransformer {
       private final AtomicInteger balance = new AtomicInteger(1000); // Initial balance
       private final AtomicInteger itemCount = new AtomicInteger(0); // Initial item count
       private final Map<String, AtomicInteger> dynamicFields = new ConcurrentHashMap<>();

       @Override
       public ResponseDefinition transform(Request request, ResponseDefinition responseDefinition, FileSource files, Parameters parameters) {
           String dynamicField = parameters.getString("dynamicField");

           if (dynamicField != null) {
               String requestBody = request.getBodyAsString();
               int amount = Integer.parseInt(requestBody);

               AtomicInteger field = dynamicFields.computeIfAbsent(dynamicField, k -> new AtomicInteger(0));
               field.addAndGet(amount);

               return ResponseDefinitionBuilder
                       .like(responseDefinition)
                       .withBody("{\"" + dynamicField + "\": " + field.get() + "}")
                       .build();
           }

           return responseDefinition;
       }

       @Override
       public String getName() {
           return "custom-response-transformer";
       }

       @Override
       public boolean applyGlobally() {
           return false;
       }
   }
   ```

3. **Register and Use the Transformer in WireMock**:
   Configure WireMock to use the custom transformer and define the mappings with the custom fields.

   ```java
   import com.github.tomakehurst.wiremock.WireMockServer;
   import com.github.tomakehurst.wiremock.client.WireMock;
   import com.github.tomakehurst.wiremock.common.ConsoleNotifier;

   public class WireMockExample {
       public static void main(String[] args) {
           WireMockServer wireMockServer = new WireMockServer();
           wireMockServer.start();

           wireMockServer.getOptions().extensions(new CustomResponseTransformer());

           // Balance endpoint
           wireMockServer.stubFor(WireMock.post(WireMock.urlEqualTo("/transaction"))
                   .willReturn(WireMock.aResponse()
                           .withTransformers("custom-response-transformer")
                           .withTransformerParameter("dynamicField", "balance")));

           // Item endpoint
           wireMockServer.stubFor(WireMock.post(WireMock.urlEqualTo("/item"))
                   .willReturn(WireMock.aResponse()
                           .withTransformers("custom-response-transformer")
                           .withTransformerParameter("dynamicField", "itemCount")));

           Runtime.getRuntime().addShutdownHook(new Thread(wireMockServer::stop));
           System.out.println("WireMock server running. Press Ctrl+C to stop.");
           try {
               Thread.sleep(Long.MAX_VALUE);
           } catch (InterruptedException e) {
               Thread.currentThread().interrupt();
           }
       }
   }
   ```

### Testing with Curl
```sh
# Add a credit transaction of 100 to balance
curl -X POST http://localhost:8080/transaction -d "100"

# Add an item (increment by 1)
curl -X POST http://localhost:8080/item -d "1"

# Check the balance
curl http://localhost:8080/transaction

# Check the item count
curl http://localhost:8080/item
```

### Explanation:
1. **Custom ResponseTransformer**: Handles requests and dynamically updates fields based on incoming request parameters.
2. **Dynamic Fields**: Uses a `Map` to store and update dynamic fields like balance and itemCount.
3. **Transformer Parameters**: Specifies which field to update through `withTransformerParameter`.

This setup allows you to handle different dynamic fields by adding custom parameters to the stub mappings, making your mock server more flexible and powerful.


To create a standalone JAR with WireMock and a custom transformer using Gradle, you can follow these steps:

1. **Create the Custom Transformer Class**:
   Ensure your transformer class (e.g., `CustomResponseTransformer`) is implemented in a package, as described in the previous responses.

2. **Create a Main Class**:
   Implement a main class that starts the WireMock server and registers the custom transformer.

3. **Set Up Your `build.gradle`**:
   Configure the Gradle build script to include the necessary dependencies and package the application as a standalone JAR with all dependencies.

4. **Build the JAR**:
   Use Gradle to compile and package the JAR.

5. **Run the Standalone JAR**:
   Execute the JAR file to start the WireMock server with the custom transformer.

### Detailed Steps

#### 1. Implement the Custom Transformer and Main Class

Ensure the classes are implemented as described earlier, with proper package names. For example, the `CustomResponseTransformer` should be in the `com.example.transformer` package, and the main class should be in the `com.example.main` package.

#### 2. Set Up Your `build.gradle`

Here's an example `build.gradle` file:

```gradle
plugins {
    id 'java'
    id 'application'
}

group 'com.example'
version '1.0-SNAPSHOT'

sourceCompatibility = '1.8'

repositories {
    mavenCentral()
}

dependencies {
    implementation 'com.github.tomakehurst:wiremock-jre8:2.31.0'
}

application {
    mainClass = 'com.example.main.WireMockStandalone'
}

jar {
    manifest {
        attributes 'Main-Class': 'com.example.main.WireMockStandalone'
    }

    from {
        configurations.runtimeClasspath.collect { it.isDirectory() ? it : zipTree(it) }
    }
    duplicatesStrategy = DuplicatesStrategy.EXCLUDE
}
```

#### 3. Build the JAR

To build the JAR, run the following command in your project directory:

```sh
./gradlew clean build
```

This command will create the JAR file with all dependencies included.

#### 4. Run the Standalone JAR

Navigate to the `build/libs` directory and run the JAR file:

```sh
java -jar wiremock-transformer-1.0-SNAPSHOT.jar
```

This will start the WireMock server with your custom transformer.

### Explanation

- **Plugins**: The `java` plugin is used to compile Java code, and the `application` plugin is used to specify the main class.
- **Dependencies**: The `wiremock-jre8` dependency includes WireMock with Java 8 compatibility.
- **Application Configuration**: The `application` block specifies the main class to run the application.
- **JAR Configuration**: The `jar` block configures the manifest and ensures all runtime dependencies are included in the final JAR.

This setup should create a standalone JAR that includes all necessary dependencies and your custom transformer, making it easy to run WireMock with your custom logic.
If your Nexus credentials are stored in Jenkins credentials, you can securely access them within your Jenkins pipeline using the `withCredentials` block. Here's how you can modify the script to utilize the Jenkins credentials for Nexus:

### Updated Jenkins Pipeline Script

```groovy
pipeline {
    agent any

    parameters {
        file(name: 'CSV_FILE', description: 'Upload the CSV file containing group ID, artifact ID, old namespace, new namespace, and version')
    }

    environment {
        NEXUS_URL = 'http://your-nexus-repo-url' // Replace with your Nexus repository URL
        CREDENTIALS_ID = 'nexus-credentials' // Jenkins credentials ID for Nexus
    }

    stages {
        stage('Process CSV') {
            steps {
                script {
                    // Ensure the CSV file is provided
                    if (!params.CSV_FILE) {
                        error "CSV file not provided!"
                    }

                    // Read the CSV file content
                    def csvFilePath = params.CSV_FILE
                    def csvFile = readCSV(csvFilePath)

                    // Use Jenkins credentials for Nexus authentication
                    withCredentials([usernamePassword(credentialsId: env.CREDENTIALS_ID, usernameVariable: 'NEXUS_USERNAME', passwordVariable: 'NEXUS_PASSWORD')]) {
                        csvFile.each { line ->
                            def groupId = line.group_id
                            def artifactId = line.artifact_id
                            def oldNamespace = line.old_namespace
                            def newNamespace = line.new_namespace
                            def version = line.version

                            def artifactPath = groupId.replace('.', '/') + '/' + artifactId + '/' + version + '/' + artifactId + '-' + version + '.jar'

                            // Download JAR from old namespace
                            def downloadUrl = "${NEXUS_URL}/repository/${oldNamespace}/${artifactPath}"
                            def jarFile = "${artifactId}-${version}.jar"
                            sh "curl -u ${NEXUS_USERNAME}:${NEXUS_PASSWORD} -O ${downloadUrl}"

                            // Upload JAR to new namespace
                            def uploadUrl = "${NEXUS_URL}/repository/${newNamespace}/"
                            sh """
                                curl -u ${NEXUS_USERNAME}:${NEXUS_PASSWORD} --upload-file ${jarFile} ${uploadUrl}/${artifactPath}
                            """

                            // Clean up the downloaded file
                            sh "rm -f ${jarFile}"
                        }
                    }
                }
            }
        }
    }
}

def readCSV(filePath) {
    def lines = readFile(filePath).split("\n")
    def header = lines[0].split(",")
    def rows = lines.drop(1).collect { it.split(",") }
    return rows.collect { row -> header.zip(row).collectEntries { [(it[0]): it[1]] } }
}
```

### Key Changes and Explanation:

1. **Environment Variables for Credentials:**
   - `CREDENTIALS_ID`: This is the Jenkins credentials ID where the Nexus username and password are stored.

2. **withCredentials Block:**
   - This block retrieves the Nexus username and password from the Jenkins credentials store using the `credentialsId` defined in the environment variables.
   - `usernameVariable: 'NEXUS_USERNAME'` and `passwordVariable: 'NEXUS_PASSWORD'` are used to store the credentials in environment variables that can be accessed within the script.

3. **Using the Credentials:**
   - The `NEXUS_USERNAME` and `NEXUS_PASSWORD` environment variables are used in the `curl` commands for both downloading and uploading the artifacts.

### Jenkins Credentials Setup:
- Ensure that the Nexus credentials (username and password) are stored in Jenkins under the specified `CREDENTIALS_ID`. You can do this by:
  1. Navigating to **Manage Jenkins** > **Manage Credentials**.
  2. Adding a new **Username with password** credential and giving it the ID `nexus-credentials` (or whatever ID you specify in the script).

### Running the Job:
- When you run the job, it will securely retrieve the Nexus credentials from Jenkins, use them to authenticate against Nexus, and handle the artifacts as specified in the CSV file.

This setup ensures that your Nexus credentials are managed securely within Jenkins and aren't exposed directly in your pipeline script.


```
pipeline {
    agent any
    parameters {
        string(name: 'USERNAME', description: 'Username for authentication')
        password(name: 'PASSWORD', description: 'Password for authentication')
        string(name: 'ID', description: 'ID of the resource to delete')
    }
    stages {
        stage('Generate Token') {
            steps {
                script {
                    // Define the API URL for token generation
                    def tokenApiUrl = "https://api.example.com/auth/token"
                    // Make an API request to get the token
                    def tokenResponse = sh(
                        script: """
                            curl -X POST "${tokenApiUrl}" \
                            -H "Content-Type: application/json" \
                            -d '{ "username": "${params.USERNAME}", "password": "${params.PASSWORD}" }' \
                            --silent --show-error --fail
                        """,
                        returnStdout: true
                    ).trim()
                    
                    // Parse the JSON response to extract the token
                    def token = readJSON(text: tokenResponse).token
                    echo "Generated Token: ${token}"

                    // Store the token for the next stage
                    env.TOKEN = token
                }
            }
        }

        stage('Delete Resource') {
            steps {
                script {
                    // Define the API URL for deleting the resource
                    def deleteApiUrl = "https://api.example.com/resources/${params.ID}"
                    
                    // Make an API request to delete the resource using the generated token
                    sh """
                        curl -X DELETE "${deleteApiUrl}" \
                        -H "Authorization: Bearer ${env.TOKEN}" \
                        --silent --show-error --fail
                    """
                    echo "Resource with ID ${params.ID} has been deleted."
                }
            }
        }
    }
}
```

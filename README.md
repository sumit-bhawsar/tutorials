
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



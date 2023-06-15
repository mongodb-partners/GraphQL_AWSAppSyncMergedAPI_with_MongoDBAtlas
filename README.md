# AWS AppSync Integration with MongoDB Atlas

## Introduction
When multiple teams are working for a project with microservices that are exposed through a single GraphQL API endpoint, it is important to ensure proper collaboration and isolation among the teams involved. Without effective measures in place, simultaneous work by different developer teams can inadvertently result in breaking the code and adds challenges for troubleshooting. For example, a change made by one team could unintentionally impact the functionality implemented by another team.  In this repository, we present you a reference architecture with MongoDB Atlas and AWS AppSync Merged API as solution to these challenges

## MongoDB Atlas

[MongoDB Atlas](https://www.mongodb.com/atlas) is a modern Developer Data Platform with a fully managed cloud database at its core.  Atlas is the best way to run MongoDB, the leading non-relational database. It provides rich features like flexible schema model, native timeseries collections, geo-spatial data, multi level indexing, RBAC, isolated workloads and many more–all built on top of the MongoDB document model.  

MongoDB Atlas App Services help developers build apps, integrate services, and connect to their data without operational overhead utilizing features like hosted Data API and GraphQL API.  Atlas Data API allows developers to easily integrate Atlas data into their cloud apps and services over HTTPS with a flexible API.  Atlas GraphQL API lets developers access Atlas data from any standard GraphQL client with an API that generates based on your data’s schema. 


## AWS AppSync Merged API
[AWS AppSync Merged API](https://docs.aws.amazon.com/appsync/latest/devguide/merged-api.html) enable teams to manage resources from multiple source AppSync APIs into a single, unified AppSync GraphQL endpoint.  Frontend Developers need to interact with a single endpoint and the datalayers are abstracted completely. In the backend, teams from different data domains can independently create and deploy AWS AppSync APIs that includes GraphQL schemas, resolvers, data sources, and functions, that can then be combined into a single, merged API. On successfull deployment,  they can merge their changes to the Merged API endpoint in order to make them available to frontend developers without blocking on other changes from other source APIs. 

## Salient Features

Authentication and Authorization through [Amazon Cognito](https://docs.aws.amazon.com/cognito/latest/developerguide/what-is-amazon-cognito.html)
Sub-graphs managed independently
Caching to improve performance
End-to-end serverless architecture
Conflict resolution via schema directives


## Pre-requisite

Developer Tool: [VSCode](https://code.visualstudio.com/download), [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html), [Docker](https://docs.docker.com/engine/install/), [Postman](https://www.postman.com/downloads/)


## Architecture

<img width="1008" alt="image" src="https://github.com/mongodb-partners/GraphQL_AWSAppSyncMergedAPI_with_MongoDBAtlas/assets/101570105/338ab5dc-8df9-46e0-9815-463fa53c3790">




## Steps for Integration


### 1.Setup two AppSync GraphQL endpoints through Atlas Driver and Atlas Data API

Follow the [GitHub repo](https://github.com/mongodb-partners/AppSync_Atlas_Integration) to setup the AppSync GraphQL endpoints using MongoDB Atlas Data API and MongoDB Driver.

Ensure th

### 2. crerate a AWS Elastic Container Repository for merged api


      aws ecr create-repository \
                  --repository-name partner_atlas_appsync_mergedapi \
                  --image-scanning-configuration scanOnPush=true \
                  --region <aws_region>


### 3.create the docker image and deploy to lambda

Copy the Lambda code from the repository and create the docker image

      aws ecr get-login-password --region <aws region> | docker login --username AWS --password-stdin <accountid>.dkr.ecr.<aws region>.amazonaws.com
      
      docker build -t partner_atlas_appsync_mergedapi . --platform=linux/amd64
      
      docker tag partner_test:latest <accountid>.dkr.ecr.<aws region>.amazonaws.com/partner_atlas_appsync_mergedapi:latest
      
      docker images
      
      docker push <accountid>.dkr.ecr.<aws region>.amazonaws.com/partner_atlas_appsync_mergedapi:latest


### 4.Create the Lambda function



      aws lambda create-function --region <aws region>  --function-name partner_atlas_appsync_mergedapi \
          --package-type Image  \
          --code ImageUri= <accountid>.dkr.ecr.<aws region>.amazonaws.com/partner_atlas_appsync_mergedapi:latest   \
          --role <Lambda execution role ARN>

pls check the [link](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-images.html#configuration-images-api) for reference code

Note: Ensure the lambda function is having adequate permission to read from the secret manager.

### 5.Ensure the APIs are created successfuly 


#### a. Create API
1. Choose Build from scratch and hit Start, for getting a blank API.
<img width="1728" alt="AppSync_Creation_Step1" src="https://user-images.githubusercontent.com/114057324/198644438-17269f86-094f-4113-83f0-4eb1b9d1fc97.png">

2. Add API Name
<img width="1728" alt="Screenshot 2022-10-28 at 5 36 25 PM" src="https://user-images.githubusercontent.com/114057324/198644942-23ffa2f9-dd09-4459-aa84-dcdab680eac4.png">


#### b. Add Schema
1. Click on Schema form the left navigations

<img width="1728" alt="Screenshot 2022-10-28 at 5 36 56 PM" src="https://user-images.githubusercontent.com/114057324/198645875-92ecc6d3-a55e-4986-9a37-71be54207a41.png">

2. GraphQL Schema can be built with Queries and Mutations and click Save Schema
<img width="1728" alt="Screenshot 2022-10-28 at 5 37 45 PM" src="https://user-images.githubusercontent.com/114057324/198646751-842d36c2-47f4-4750-a72d-4fe6a339e019.png">

#### c. Create Data Sources

1. Click on Data Sources from the left navigations and hit Create data source
<img width="1728" alt="Screenshot 2022-10-28 at 5 38 46 PM" src="https://user-images.githubusercontent.com/114057324/198647355-38a85ad1-95f7-47af-bd9d-4cbd082bfdae.png">


2. Attach Lambda function which contains MongoDB driver code for quering the data (Multiple data sources can be added based on requirements)
<img width="1728" alt="Screenshot 2022-10-28 at 5 40 00 PM" src="https://user-images.githubusercontent.com/114057324/198648544-23e46164-dc55-46c7-9693-31967a069400.png">


#### d. Attach Resolvers
1. Go to schema and attach resolvers for Mutations and Queries
<img width="1728" alt="Screenshot 2022-10-28 at 6 43 22 PM" src="https://user-images.githubusercontent.com/114057324/198650042-2e6e61dd-40be-4fc1-ab8b-357d024a00a4.png">

2. Select lambda data source added in previous step, and click save.
<img width="1728" alt="Screenshot 2022-10-28 at 6 44 32 PM" src="https://user-images.githubusercontent.com/114057324/198650403-a8ccb56f-2d07-479b-a46e-ab8d1425bd43.png">

### 8. Test the API
#### a. Using AWS Management Console
Goto Queries and select the query to execute, hit Run button and you should see your query result on the right side.

<img width="1728" alt="Screenshot 2022-10-28 at 6 46 20 PM" src="https://user-images.githubusercontent.com/114057324/198651689-0cd1a646-a54b-4d30-be2b-b80c4bbbb890.png">

##### b. Using Postman
1. Click on settings and create a curl API as below.

            curl --location --request <CRUD OPERATION> <API URL>
            --header 'Content-Type: application/graphql' \
            --header 'x-api-key: <API KEY> \
            --data-raw '{"query": <QUERY>}'
![Screenshot 2022-10-28 at 8 14 14 PM](https://user-images.githubusercontent.com/114057324/198657824-c690a69d-e2d5-4660-8db4-47d526af816e.png)

      
2. Goto Postman and import the curl command and hit send to get the response.

<img width="1728" alt="Screenshot 2022-10-28 at 7 41 33 PM" src="https://user-images.githubusercontent.com/114057324/198658280-c9f54909-2fb7-4090-994b-cc27cf661b99.png">


## Summary

This solution can be extended to [AWS Amplify](https://aws.amazon.com/amplify/) for building mobile applications.

For any further information, please contact partners@mongodb.com




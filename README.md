# GraphQL : AWS AppSync Merged API Integration with MongoDB Atlas

## Introduction
When multiple teams are working for a project with microservices that are exposed through a single [GraphQL](https://graphql.org/) API endpoint, it is important to ensure proper collaboration and isolation among the teams involved. Without effective measures in place, simultaneous work by different developer teams can inadvertently result in breaking the code and adds challenges for troubleshooting. For example, a change made by one team could unintentionally impact the functionality implemented by another team.  In this repository, we present you a reference architecture with MongoDB Atlas and AWS AppSync Merged API as solution to these challenges

## MongoDB Atlas

[MongoDB Atlas](https://www.mongodb.com/atlas) is a modern Developer Data Platform with a fully managed cloud database at its core.  Atlas is the best way to run MongoDB, the leading non-relational database. It provides rich features like flexible schema model, native timeseries collections, geo-spatial data, multi level indexing, RBAC, isolated workloads and many more–all built on top of the MongoDB document model.  

[MongoDB Atlas App Services](https://www.mongodb.com/atlas/app-services) help developers build apps, integrate services, and connect to their data without operational overhead utilizing features like hosted Data API and GraphQL API.  [Atlas Data API](https://www.mongodb.com/atlas/app-services/data-api) allows developers to easily integrate Atlas data into their cloud apps and services over HTTPS with a flexible API.  [Atlas GraphQL](https://www.mongodb.com/docs/atlas/app-services/graphql/) API lets developers access Atlas data from any standard GraphQL client with an API that generates based on your data’s schema. 


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

<img width="1011" alt="image" src="https://github.com/mongodb-partners/GraphQL_AWSAppSyncMergedAPI_with_MongoDBAtlas/assets/101570105/bcc74eb4-2cf5-477d-8093-4cb888ba7191">




## Steps for Integration


### 1.Setup two AppSync GraphQL endpoints through Atlas Driver and Atlas Data API

Follow the [GitHub repo](https://github.com/mongodb-partners/AppSync_Atlas_Integration) to setup the AppSync GraphQL endpoints using MongoDB Atlas Data API and MongoDB Driver.

Ensure the the APIs are created successfully

<img width="624" alt="image" src="https://github.com/mongodb-partners/GraphQL_AWSAppSyncMergedAPI_with_MongoDBAtlas/assets/101570105/970b017c-ff89-4a93-85aa-534f396b6b4a">



### 2. crerate a AWS Elastic Container Repository for merged api


      aws ecr create-repository \
                  --repository-name partner_atlas_appsync_mergedapi \
                  --image-scanning-configuration scanOnPush=true \
                  --region <aws_region>


### 3.create the docker image and deploy to lambda

Copy the [python code](https://github.com/mongodb-partners/GraphQL_AWSAppSyncMergedAPI_with_MongoDBAtlas/blob/main/merged_api/datasource_mergedapi.py) for the mergedAPI Lambda from the repository and create the docker image

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

### 5.Create the Merged API in AWS AppSync

Navigate to the AWS AppSync and click create API. Select the Merged APIs.

<img width="621" alt="image" src="https://github.com/mongodb-partners/GraphQL_AWSAppSyncMergedAPI_with_MongoDBAtlas/assets/101570105/9f11ba5e-5c1c-419f-9c58-69df17d02566">



#### Provide the metadata for the Merged API

<img width="621" alt="image" src="https://github.com/mongodb-partners/GraphQL_AWSAppSyncMergedAPI_with_MongoDBAtlas/assets/101570105/1cd3a3ad-bf89-46be-abb7-2ed7db7e2ec4">





#### Select the source APIs created in Step 1

<img width="621" alt="image" src="https://github.com/mongodb-partners/GraphQL_AWSAppSyncMergedAPI_with_MongoDBAtlas/assets/101570105/fa34cb55-a2bf-403b-b4bc-f65f5841d5f3">





#### Select the authorization mechanism. The mechanism must be the same as in the source APIs

<img width="621" alt="image" src="https://github.com/mongodb-partners/GraphQL_AWSAppSyncMergedAPI_with_MongoDBAtlas/assets/101570105/601e76be-6cf5-4548-b922-d14a46175622">





#### Verify the details and create the merged API

<img width="621" alt="image" src="https://github.com/mongodb-partners/GraphQL_AWSAppSyncMergedAPI_with_MongoDBAtlas/assets/101570105/8956a380-a0c9-429d-8458-4eb80bdfd9bc">





#### After creating the merged API, verify the source APIs are in “Merge success” status.

<img width="628" alt="image" src="https://github.com/mongodb-partners/GraphQL_AWSAppSyncMergedAPI_with_MongoDBAtlas/assets/101570105/707d535c-080d-4231-a444-8961cdcbdbde">



### 5.Update the source schema.


#### Update the CounterPartyRisk source API’s in the source API (Not in the merged API) schema with a reference to CounterParty API and save.
  

<img width="626" alt="image" src="https://github.com/mongodb-partners/GraphQL_AWSAppSyncMergedAPI_with_MongoDBAtlas/assets/101570105/133ca905-230f-4894-87e0-72bef5512e7d">

  

#### Attach the Lambda resolver in the CounterParty schema, created in step viii, to  CounterParty:risk_data field, as shown below.
  


<img width="623" alt="image" src="https://github.com/mongodb-partners/GraphQL_AWSAppSyncMergedAPI_with_MongoDBAtlas/assets/101570105/ec297319-6007-4ae8-b769-05a47f3ba7a9">

  

#### Click on the action dropdown to select the Update runtime. 
  

<img width="619" alt="image" src="https://github.com/mongodb-partners/GraphQL_AWSAppSyncMergedAPI_with_MongoDBAtlas/assets/101570105/2d27324b-aa37-420d-8517-54ae42841c9a">
   
   
 
#### Choose the Unit Resolver VTL only  [Apache Velocity Template Language](https://docs.aws.amazon.com/appsync/latest/devguide/tutorials.html) . Refer to to the [VTL resolver documentation](https://docs.aws.amazon.com/appsync/latest/devguide/resolver-mapping-template-reference.html) for additional details. 

<img width="625" alt="image" src="https://github.com/mongodb-partners/GraphQL_AWSAppSyncMergedAPI_with_MongoDBAtlas/assets/101570105/1587eaf9-b6bc-41f4-9690-ffbfc11ce90c">



#### Choose the Lambda created in earlier step as the data source for the resolver and click save resolver

<img width="616" alt="image" src="https://github.com/mongodb-partners/GraphQL_AWSAppSyncMergedAPI_with_MongoDBAtlas/assets/101570105/795b7a08-3460-4869-b204-9df340eb338f">


#### Verify the resolver is attached successfully

<img width="630" alt="image" src="https://github.com/mongodb-partners/GraphQL_AWSAppSyncMergedAPI_with_MongoDBAtlas/assets/101570105/3e770591-c20b-4161-a7dc-bcc75d1e701e">



#### Click Merge now button to merge the APIs from the settings options

<img width="625" alt="image" src="https://github.com/mongodb-partners/GraphQL_AWSAppSyncMergedAPI_with_MongoDBAtlas/assets/101570105/dc9e260d-5a16-4877-8406-708eaf28f428">



#### Check that the merged API reflects the changes contained in the source API 


<img width="622" alt="image" src="https://github.com/mongodb-partners/GraphQL_AWSAppSyncMergedAPI_with_MongoDBAtlas/assets/101570105/57223e0f-2a59-4a45-bcaf-1ef8c4f9b9d4">





## 6. Test the API
### a. Using AWS Management Console

Query the Merged API with the CounterPartyRisk data and observe the results.  You should see the additional risk data in the CounterParty schema.  


<img width="619" alt="image" src="https://github.com/mongodb-partners/GraphQL_AWSAppSyncMergedAPI_with_MongoDBAtlas/assets/101570105/1018a14f-aab3-4cf4-b902-e2f0b9fd31a2">


#### b. Using Postman
1. Click on settings and create a curl API as below.

            curl --location --request <CRUD OPERATION> <API URL>
            --header 'Content-Type: application/graphql' \
            --header 'x-api-key: <API KEY> \
            --data-raw '{"query": <QUERY>}'
            

      
2. Goto Postman and import the curl command and hit send to get the response.

<img width="1728" alt="Screenshot 2022-10-28 at 7 41 33 PM" src="https://user-images.githubusercontent.com/114057324/198658280-c9f54909-2fb7-4090-994b-cc27cf661b99.png">


## Summary

In conclusion, AWS AppSync Merged API empowers developers to easily combine and enrich data from different sources into a single API. This not only benefits frontend and backend developer teams, but also fosters collaboration through a unified API layer.  Try out MongoDB Atlas on AWS [Atlas GraphQL](https://www.mongodb.com/docs/atlas/app-services/graphql/)




## About This Project
Serverless Framework with AWS & Node.JS..

```diff
- Technologies Used :
```
<table>
    <tr>
        <td>Serverless Framework</td>
        <td>AWS SAM</td>
        <td>AWS Lambda</td>
        <td>Step Functions</td>
    </tr>
    <tr>
        <td>API Gateway</td>
        <td>RDS</td>
        <td>DynamoDB</td>
        <td>ElephantSQL</td>
    </tr>
    <tr>
        <td>CloudFormation</td>
        <td>CI/CD</td>
        <td>CloudWatch</td>
        <td>CodeCommit</td>
    </tr>
    <tr>
        <td>CodeBuild</td>
        <td>CodePipeline</td>
        <td>S3 Notifications</td>
        <td>SNS</td>
    </tr>
    <tr>
        <td>SQS</td>
        <td>Cognito</td>
        <td>Lambda Crons</td>
        <td>CORS</td>
    </tr>
    <tr>
        <td>Apache VTL</td>
        <td>Swagger</td>
        <td>KMS</td>
        <td>VPCs</td>
    </tr>
    <tr>
        <td>DLQs</td>
        <td>CloudFront</td>
        <td>OICD</td>
        <td>Kinesis</td>
    </tr>
    <tr>
        <td>MQTT</td>
        <td>IoT</td>
        <td>Elastic Beanstalk</td>
        <td>ElastiCache</td>
    </tr>
</table>

```diff
+ And lot more..
```

## Author
Aditya Hajare ([Linkedin](https://in.linkedin.com/in/aditya-hajare)).

## Current Status
WIP (Work In Progress)!

## Important Notes
- [Theory](#theory)
- [AWS Lambda Limits](#aws-lambda-limits)
- [DynamoDB](#dynamodb)
- [AWS Step Functions](#aws-step-functions)
- [AWS SAM](#aws-sam)
- [Setup And Workflow 101](#setup-and-workflow-101)
- [New Project Setup In Pre Configured Environment 101](#new-project-setup-in-pre-configured-environment-101)
- [Installing Serverless](#installing-serverless)
- [Configuring AWS Credentials For Serverless](#configuring-aws-credentials-for-serverless)
- [Create NodeJS Serverless Service](#create-nodejs-serverless-service)
- [Invoke Lambda Function Locally](#invoke-lambda-function-locally)
- [Event - Passing Data To Lambda Function](#event---passing-data-to-lambda-function)
- [Serverless Offline](#serverless-offline)
- [NPM Run Serverless Project Locally](#npm-run-serverless-project-locally)
- [Deploy Serverless Service](#deploy-serverless-service)
- [Setup Serverless DynamoDB Local](#setup-serverless-dynamodb-local)
- [Securing APIs](#securing-apis)
- [AWS CLI Handy Commands](#aws-cli-handy-commands)
- [Common Issues](#common-issues)

## License
Open-sourced software licensed under the [MIT license](http://opensource.org/licenses/MIT).

----------------------------------------

### Theory
- Every AWS account comes with a default `VPC (Virtual Private Cloud)`.
- At the moment, `AWS Lambda Function` can run upto a maximum of `15 Minutes`.
- Returning `HTTP` responses from `AWS Lambda` allows us to integrate them with `Lambda Proxy Integration` for `API Gateway`.
- A `Step Function` can run upto a maximum period of `1 Year`.
- **`Step Functions` allows us to combine different `Lambda Functions` to build `Serverless Applications` and `Microservices`.**
- There could be different reasons why you may want to restrict your `Lambda Function` to run within a given `VPC`. For e.g.
    * You may have an `Amazon RDS` instance running on `EC2` inside your `VPC` and you want to connect to that instance through `Lambda` without exposing it to outside world. In that case, your `Lambda Function` must run inside that `VPC`.
    * When a `Lambda Function` is attached to any `VPC`, it automatically loses access to the internet. Unless ofcourse you open a `Port` on your `VPC Security Group` to allow `Outbound Connections`.
    * While attaching `Lambda Function` to `VPC`, we must select at least 2 `Subnets`. Although we can choose more `Subnets` if we like.
    * When we are using `Serverless Framework`, all this including assigning necessary permissions etc. is taken care of automatically by the `Serverless Framework`.
- `Tags` are useful for organising and tracking our billing.
- `Serverless Computing` is a cloud computing execution model in which the cloud provider dynamically manages the allocation of infrastructure resources. So we don't have to worry about managing the servers or any of the infrastructure.
- `AWS Lambda` is an `Event Driven` serverless computing platform or a `Compute Service` provided by AWS.
- The code that we run on `AWS Lambda` is called a `Lambda Function`.
- `Lambda Function` executes whenever it is triggered by a pre-configured `Event Source`.
- `Lambda Functions` can be triggered by numerous event sources like:
    * API Gateway.
    * S3 File Uploads.
    * Changes to `DynamoDB` table data.
    * `CloudWatch` events.
    * `SNS` Notifications.
    * Third Party APIs.
    * `IoT Devices`.
    * And so on..
- `Lambda Functions` run in `Containerized Environments`.
- We are charged only for the time our `Lambda Functions` are executing.
- No charge for `Idle Time`.
- Billing is done in increments of `100 ms` of the `Compute Time`.
- `AWS Lambda` uses decoupled `Permissions Model`.
- `AWS Lambda` supports 2 `Invocation Types`:
    * **Synchronous**.
    * **Asynchronous**.
- `Invocation Type` of AWS Lambda depends on the `Event Source`. For e.g.
    * `API Gateway` or `Cognito` event is `Synchronous`.
    * `S3 Event` is always `Asynchronous`.
- `pathParameters` and `queryStringParameters` are the pre-defined attributes of `API Gateway AWS Proxy Event`.
- `AWS API Gateway` expects Lambda function to return `well formed http response` instead of just the data or just the response body. At the bare-minimum, our response must have `statusCode` and `body`. For e.g.
    ```json
        {
            "statusCode" : 200,
            "body": {
                "message": "Hello Aditya"
            }
        }
    ```
- Typical code to build above response:
    ```javascript
        return {
            statusCode: 200,
            body: JSON.stringify({message: "Hello Aditya"});
        }
    ```
- **Lambda Versioning:**
    * When we don't explicitely create an user version, Lambda will use the `$LATEST` version.
    * The latest version is always denoted by `$LATEST`.
    * The last edited version is always marked as `$LATEST` one.
- **(Without using Lambda Aliases) How to use different version of Lambda Function in API Gateway? Bad Way!**
    1. Under AWS Console, go to `API Gateway`.
    2. Click on `Request (GET/POST/PUT/PATCH/DELETE)` under `Resource`.
    3. Click on `Integration Request`.
    4. Configure `Lambda Function` setting with a value of version seperated by colon.
    5. Re-deploy the API.
    6. For e.g.
        ```javascript
            // Lambda Function name: adiTest
            // Available Lambda Function versions: v1, v2, v3 ..etc.
            // To use v2 for API Gateway GET Request, set Lambda Function value as below:
            {
                "Lambda Function": "adiTest:2"
            }
        ```
- **Need for Lambda Aliases:**
    * Without `Lambda Aliases`, whenever we publish a new `Lambda Version`, we will manually have to edit API Gateway to use new `Lambda Version` and then republish the API (Refer to above steps).
    * Everytime we publish a new `Lambda Version`, `API Gateway` should automatically pick up the change without we having to re-deploy the API. `Lambda Aliases` helps us achive this.
- **Lambda Aliases:**
    * It's a good practice to create 1 `Lambda Alias` per `Environment`. For e.g. We could have aliases for dev, production, stage etc. environments.
    * While configuring `Lambda Alias`, we can use `Additional Version` setting for `Split Testing`.
    * `Split Testing` allows us to split user traffic between multiple `Lambda Versions`.
    * To use `Lambda Alias` in `API Gateway`, we simply have to replace `Version Number` (Seperated by colon) under `Lambda Function` setting (in `API Gateway` settings), with an `Alias`.
    * Re-deploy the API.
    * For e.g.
        ```javascript
            // Lambda Function name: adiTest
            // Available Lambda Function versions: v1, v2, v3 ..etc.
            // Available Lambda Function aliases: dev, stage, prod ..etc.

            // Aliases are pointing to following Lambda versions:
            {
                "dev": "v1",
                "stage": "v2",
                "prod": "$LATEST"
            }

            // To use v2 for API Gateway GET Request, set Lambda Function value as below:
            {
                "Lambda Function": "adiTest:stage"
            }
        ```
- **Stage Variables in API Gateway:**
    * Everytime we make changes to `API Gateway`, we don't want to update the `Alias Name` in every `Lambda Function` before deploying the corrosponding `Stage`. To address this challenge, we can make use of what is called as `Stage Variables in API Gateway`.
    * `Stage Variables` can be used for various puposes like:
        - Choosing backend database tables based on environment.
        - For dynamically choosing `Lambda Alias` corrosponding to the current `Stage`.
        - Or any other configuration.
    * `Stage Variables` are available inside `context` object of `Lambda Function`.
    * Since `Stage Variables` are available inside `context` object, we can also use them in `Body Mapping Templates`.
    * `Stage Variables` can be used as follows:
        - Inside `API Gateway Resource Configuration`, to choose `Lambda Function Alias` corrosponding to the current stage:
        ```javascript
            // Use ${stageVariables.variableName}

            {
                "Lambda Function": "myFunction:${stageVariables.variableName}"
            }
        ```
- **Canary Deployment:**
    * Related to `API Gateways`.
    * Used for traffic splitting between different versions in `API Gateways`.
    * Use `Promote Canary` option to direct all traffic to latest version once our testing using traffic splitting is done.
    * After directing all traffic to latest version using `Promote Canary` option, we can choose to `Delete Canary` once we are sure.
- **Encryption For Environment Variables In Lambda:**
    * By default `Lambda` uses default `KMS` key to encrypt `Environment Variables`.
    * `AWS Lambda` has built-in encryption at rest and it's enabled by default.
    * When our `Lambda` uses `Environment Variables` they are automatically encrypted by `Default KMS Key`.
    * When the `Lambda` function is invoked, `Environment Variables` are automatically `decrypted` and made available in `Lambda Function's code`.
    * However, this only takes care of `Encryption at rest`. But during `Transit` for e.g. when we are deploying the `Lambda Function`, these `Environment Variables` are still transferred in `Plain Text`.
    * So, if `Environment Variables` posses sensitive information, we can enable `Encryption in transit`.
    * If we enable `Encryption in transit` then `Environment Variable Values` will be masked using `KMS Key` and we must decrypt it's contents inside `Lambda Functions` to get the actual values stored in variables.
    * While creating `KMS Keys`, be sure to choose the `Region` same as our `Lambda Function's Region`.
    * Make sure to give our `Lambda Function's Role` a permission to use `KMS Key` feature inside `KMS Key's` policy.
- **Retry Behavior in AWS Lambda:**
    * `Lambda Functions` have built-in `Retry Behavior`. i.e. When a `Lambda Functions` fails, `AWS Lambda` automatically attempts to retry the execution up to **`2 Times`** if it was invoked `Asynchronously (Push Events)`.
    * A lambda function could fail for different reasons such as:
        - Logical or Syntactical error in Lambda Function's code.
        - Network outage.
        - A lambda function could hit the timeout.
        - A lambda function run out of memory.
        - And so on..
    * When any of above things happen, Lambda function will throw an `Exception`. How this `Exception` is handled depends upon how the `Lambda Function` was invoked i.e. `Synchronously or Asynchronously (Push Events)`.
    * If the `Lambda Function` was invoked `Asynchronously (Push Events)` then `AWS Lambda` will automatically retry up to `2 Times (With some time delays in between)` on execution failure.
    * If we configure a `DLQ (Dead Letter Queue)`, it will collect the `Payload` after subsequent retry failures i.e. after `2 Attempts`.
    * If a function was invoked `Synchronously` then calling application will receive `HTTP 429` error when function execution fails.
    * If a `DLQ (Dead Letter Queue)` is not configured for `Lambda Function`, it will discard the event after 2 retry attempts.
- **Container Reuse:**
    * `Lambda Function` executes in `Containerized Environments`.
    * Whenever we create or update a `Lambda Function` i.e. either the function code or configuration, `AWS Lambda` creates a new `Container`.
    * Whenever `Lambda Function` is executed first time i.e. after we create it or update, `AWS Lambda` creates a new `Container`.
    * Once the `Lambda Function` execution is finished, `AWS Lambda` will shut down the `Container` after a while.
    * Code written outside `Lambda Handler` will be executed once per `Container`. For e.g.
        ```javascript
            // Below code will be executed once per container.
            const AWS = require('aws-sdk);
            AWS.config.update({ region: 'ap-south-1' });
            const s3 = new AWS.S3();

            // Below code (code inside Lambda handler) will be executed everytime Lambda Function is invoked.
            exports.handler = async (event, context) => {
                return "Hello Aditya";
            };
        ```
    * It's a good practice to write all initialisation code outside the `Lambda Handler`.
    * If you have written any file in `\tmp` and a `Container` is reused for `Lambda Function Execution`, that file will be available in subsequent invocations.
    * It will result in faster executions whenever `Containers` are reused.
    * We do not have any control over when `AWS Lambda` will reuse the `Container` or when it won't.
    * If we are spawning any background processes in `Lambda Functions`, they will be executed only until `Lambda Handler` returns a response. Other time they will stay `Frozen`.
- **Running a `Lambda Function` inside a `VPC` will result in `Cold Starts`. `VPC` also introduce some delay before a function could execute which could result in `Cold Start`.**
- **`Resource Policies`** gets applied at the `API Gateway` level whereas **`IAM Policies`** gets applied at the `User/Client` level.

----------------------------------------

### AWS Lambda Limits
| Resource | Default Limit |
| --- | --- |
| Concurrent executions | 1,000 |
| Function and layer storage | 75 GB |
| Elastic network interfaces per VPC | 250 |
| Function memory allocation | 128 MB to 3,008 MB, in 64 MB increments. |
| Function timeout | 900 seconds (15 minutes) |
| Function environment variables | 4 KB |
| Function resource-based policy | 20 KB |
| Function layers | 5 layers |
| Function burst concurrency | 500 - 3000 (varies per region) |
| Invocation frequency (requests per second) |  10 x concurrent executions limit (synchronous – all sources) 10 x concurrent executions limit (asynchronous – non-AWS sources) Unlimited (asynchronous – AWS service sources  |
| Invocation payload (request and response) |  6 MB (synchronous) 256 KB (asynchronous)  |
| Deployment package size |  50 MB (zipped, for direct upload) 250 MB (unzipped, including layers) 3 MB (console editor)  |
| Test events (console editor) | 10 |
| `/tmp` directory storage | 512 MB |
| File descriptors | 1,024 |
| Execution processes/threads | 1,024 |

----------------------------------------

### DynamoDB
- Datatypes:
    * **Scaler**: Represents exactly one value.
        - For e.g. String, Number, Binary, Boolean, null.
        - `Keys` or `Index` attributes only support String, Number and Binary scaler types.
    * **Set**: Represents multiple Scaler values.
        - For e.g. String Set, Number Set and Binary Set.
    * **Document**: Represents complex structure with nested attributes.
        - Fo e.g. List and Map.
- `String` datatype can store only `non-empty` values.
- Maximum data for any item in DynamoDB is limited to `400kb`. Note: Item represents the entire row (like in RDBMS) of data.
- `Sets` are unordered collection of either Strings, Numbers or Binary values.
    * All values must be of same scaler type.
    * Do not allow duplicate values.
    * No empty sets allowed.
- `Lists` are ordered collection of values.
    * Can have multiple data types.
- `Maps` are unordered collection of `Key-Value` pairs.
    * Ideal for storing JSON documents in DynamoDB.
    * Can have multiple data types.
- DynamoDB supports 2 types of `Read Operations (Read Consistency)`:
    * `Strong Consistency`:
        - The most up-to-date data.
        - Must be requested explicitely.
    * `Eventual Consistency`:
        - May or may not reflect the latest copy of data.
        - This is the default consistency for all operations.
        - 50% Cheaper than `Strongly Consistent Read` operation.
- Internally, DynamoDB stores data in `Partitions`.
- `Partitions` are nothing but `Blocks of memory`.
- A table can have `1 or more partitions` depending on it's size and throughput.
- Each `Partition` in DynamoDB can hold maximum `10GB` of data.
- Partitioning e.g.:
    * For `500 RCU and 500 WCU` ---> `1 Partition`.
    * For `1000 RCU and 1000 WCU` ---> `2 Partitions`.
- For `Table` level operations, we need to instantiate and use `DynamoDB` class from `aws-sdk`:
    ```javascript
        const AWS = require('aws-sdk');
        AWS.config.update({ region: 'ap-south-1' });

        const dynamoDB = new AWS.DynamoDB(); // Instantiating DynamoDB class for table level operations.

        dynamoDB.listTables({}, (err, data) => {
            if (err) {
                console.log(err);
            } else {
                console.log(JSON.stringify(data, null, 2));
            }
        });
    ```
- For `Item` level operations, we need to instantiate and use `DocumentClient` from `DynamoDB` class from `aws-sdk`:
    ```javascript
        const AWS = require('aws-sdk');
        AWS.config.update({ region: 'ap-south-1' });

        const docClient = new AWS.DynamoDB.DocumentClient(); // Instantiate and use DocumentClient class for Item level operations.

        docClient.put({
            TableName: 'adi_notes_app',
            Item: {
                user_id: 'test123',
                timestamp: 1,
                title: 'Test Note',
                content: 'Test Note Content..'
            }
        }, (err, data) => {
            if (err) {
                console.log(err);
            } else {
                console.log(JSON.stringify(data, null, 2));
            }
        });
    ```
- `batchWrite()` method allows us to perform multiple write operations (e.g. Put, Update, Delete) in one go.
- Conditional writes in DynamoDB are `idempotent`. i.e. If we make same conditional write requests multiple times, only the first request will be considered.
- `document.query()` allows us to fetch items from a specific `partition`.
- `document.scan()` allows us to fetch items from all `partitions`.
- **Pagination**:
    * At a time, any `document.query()` or `document.scan()` operation can return maximum `1mb` data in a single request.
    * If our `query/scan` operation has more records to return (after exceeding 1mb limit), we will receive `LastEvaluatedKey` key in the response.
    * `LastEvaluatedKey` is simply an object containing `Index Attribute` of the next item up which the response was returned.
    * In order to retrieve further records, we must pass `LastEvaluatedKey` value under `ExclusiveStartKey` attribute in our subsequent query.
    * If there is no `LastEvaluatedKey` attribute present in DynamoDB query/scan response, it means we have reached the last page of data.
- **DynamoDB Streams:**
    * In simple words, its a `24 Hours Time-ordered Log`.
    * `DynamoDB Streams` maintain a `Time-Ordered Log` of all changes in a given `DynamoDB Table`.
    * This log stores all the `Write Activity` that took place in the last `24 hrs`.
    * Whenever there are any changes made into `DynamoDB Table` and if `DynamoDB Stream` is enabled for that table, these changes will be returned to the `Streams`.
    * There are several ways to consume and process data from `DynamoDB Streams`:
        - We can use `Kinesis Adapter` along with `Kinesis Client Library`. `Kinesis` is platform for processing `High Volume` streaming data on `AWS`.
        - We can also make use of `DynamoDB Streams SDK` to work with `DynamoDB Streams`.
        - `AWS Lambda Triggers` also allows us to work with `DynamoDB Streams`. This approach is much easy and intuitive. `DynamoDB Streams` will invoke `Lambda Functions` based on changes received by them.

----------------------------------------

### AWS Step Functions
- `AWS Step Functions` are the logical progression of `AWS Lambda Functions`.
- With `Step Functions` we can create visual workflows to co-ordinate or orchestrate different `Lambda Functions` to work together.
- A `Step Function` can run upto a maximum period of `1 Year`.
- We can use `Step Functions` to automate routine jobs like deployments, upgrades, migrations, patches and so on.
- **`Step Functions` allows us to combine different `Lambda Functions` to build `Serverless Applications` and `Microservices`.**
- Just like `Lambda Functions`, there is no need to provision any resources or infrastructure to `Step Functions`.
- We simply use `ASL (Amazon Step Language)` to define the workflows. It's a JSON based structured language. We use this language to define various steps as well as different connections and interactions between these steps in `Step Functions`.
- **The resulting workflow is called as the `State Machine`.**
- `State Machine` is displayed in a graphical form just like a flowchart.
- `State Machines` also has built-in error handling mechanisms. We can retry operations based on different errors or conditions.
- Billing is on `Pay as you go` basis. **We only pay for the trasitions between `Steps`.**
- `task` step allows us to invoke `Lambda Function` from our `State Machine`
- `activity` step allows us to run any code on `EC2 Instances`. It is similar to `task` step just that `activity` step is not `Serverless` kind of step.
- Whenever any `Step` in `State Machine` fails, entire `State Machine` fails. Here `Steps` as in `Lambda Functions` or any errors or exceptions received by `Step`.
- We can also use `CloudWatch Rules` to execute a `State Machine`.
- We can use `Lambda Function` to trigger `State Machine Execution`. The advantage of this approach is that `Lambda Functions` support many triggers for their invocation. So we have numerous options to trigger `Lambda Function` and our `Lambda Function` will trigger `State Machine Execution` using `AWS SDK`.
- While building `State Machine` and if it has any `Lambda Functions (task states)`, always specify `TimeoutSeconds` option to make sure our `State Machine` doesn't get stuck or hung.
- In `State Machine`, `catch` field is used to specify `Error Handling Catch Mechanism`.

----------------------------------------

### AWS SAM
- AWS SAM `Serverless Application Model`.
- `AWS SAM` is just a simplified version of `CloudFormation Templates`.
- It seemlessly integrates into `AWS Deployment Tools` like `CodeBuild`, `CodeDeploy`, `CodePipeline` etc.
- It provides `CLI` to build, test and deploy `Serverless Applications`.
- Every `SAM Template` begins with :
    ```yaml
        AWSTemplateFormatVersion: "2010-09-09"
        Transform: AWS::Serverless-2016-10-31
    ```
- To deploy `SAM` application:
    * It involves `2 Steps`:
        - Package application and push it to `S3 Bucket`. This step requires `S3 Bucket` to be created in prior to running `CloudFormation Package` command.
        - Deploy packaged application.
    * Step 1: We need an `S3 Bucket` created before we deploy. If we don't have one, then create it using following command:
        ```sh
            aws s3 mb s3://aditya-sam-app
        ```
    * Step 2: Package application:
        ```sh
            aws cloudformation package --template-file template.yml --output-template-file output-sam-template.yml --s3-bucket aditya-sam-app
        ```
    * Step 3: Deploy application (Here, we will be using generated output SAM template file):
        ```sh
            aws cloudformation deploy --template-file output-sam-template.yml --stack-name aditya-sam-app-stack --capabilities CAPABILITY_IAM
        ```
- To generate SAM project boilerplate from sample app:
    ```sh
        sam init --runtime nodejs12.x
    ```
- To execute `Lambda Function` locally with `SAM CLI`:
    ```sh
        # -e to pass event data to Lambda Function. This file must be present in the current location.
        sam local invoke HelloWorldFunction -e events/event.json

        # Alternatively, we can pass event data inline within the command by simply piping it as below. Here we are sending empty event data to Lambda Function.
        echo '{}' | sam local invoke HelloWorldFunction
    ```
- `SAM CLI` also allows to invoke `Lambda Functions` locally from within our application code. To do so, we have to start `Lambda Service` locally using `SAM CLI`:
    ```sh
        sam local start-lambda
    ```
- To run `API Gateway` service locally:
    * Navigate to folder where our `SAM Template` is located (e.g. `template.yaml`).
    * Execute following command to run `API Gateway Service` locally:
        ```sh
            sam local start-api
        ```

----------------------------------------

### Setup And Workflow 101
- Setup:
    ```sh
        # Install serverless globally.
        sudo npm i -g serverless

        # (Optional) For automatic updates.
        sudo chown -R $USER:$(id -gn $USER) /Users/adiinviter/.config

        # Configure user credentials for aws service provider.
        sls config credentials --provider aws --key [ACCESS_KEY] --secret [SECRET_KEY] -o

        # Create aws nodejs serverless template.
        sls create -t aws-nodejs

        # Init npm.
        npm init -y

        # Install serverless-offline and serverless-offline-scheduler as dev dependancies.
        npm i serverless-offline serverless-offline-scheduler --save-dev
    ```
- After running above commands, update the `service` property in `serverless.yml` with your service name.
    * **NOTE:** `service` property in `serverless.yml` file is mostly your project name. It is not a name of your specific lambda function.
- Add following scripts under `package.json`:
    ```json
        {
            "scripts": {
                "dev": "sls offline start --port 3000",
                "dynamodb:start": "sls dynamodb start --port 8082",
            }
        }
    ```
- Update `serverless.yml` file with following config:
    ```yml
        service: my-project-name
        plugins:
            - serverless-offline    # Add this plugin if you are using it.
            - serverless-offline-scheduler  # Add this plugin if you are using it.
        provider:
            name: aws
            runtime: nodejs12.x
            stage: dev              # Stage can be changed while executing deploy command.
            region: ap-south-1      # Set region.
    ```
- To add new lambda function with api endpoint, add following in `serverless.yml`:
    ```yml
        functions:
            hello:
                handler: src/controllers/users.find
            events:
                - http:
                    path: users/{id}
                    method: GET
                    request:
                        parameters:
                            id: true
    ```
- To run project locally:
    ```sh
        # Using npm
        npm run dev

        # Directly using serverless
        sls offline start --port 3000
    ```
- To invoke lambda function locally:
    ```sh
        sls invoke local -f [FUNCTION_NAME]
    ```
- To run lambda crons locally:
    ```sh
        sudo sls schedule
    ```
- To deploy:
    ```sh
        # To deploy all lambda functions.
        sls deploy -v

        # To deploy a specific function.
        sls deploy -v -f [FUNCTION_NAME]

        # To deploy project on a different stage (e.g. production)
        sls deploy -v -s production
    ```
- To view logs for a specific function in a specific stage (e.g. dev, prod):
    ```sh
        # Syntax:
        sls logs -f [FUNCTION_NAME] -s [STAGE_NAME] --startTime 10m

        # Use -t to view logs in real time. Good for monitoring cron jobs.
        sls logs -f [FUNCTION_NAME] -s [STAGE_NAME] -t

        # Example #1:
        sls logs -f sayHello -s production --startTime 10m

        # Example #2:
        sls logs -f sayHello -s dev --startTime 15m
    ```
- To remove project/function:
    ```sh
        # To remove everything.
        sls remove -v -s [STAGE_NAME]

        # To remove a specific function from a specific stage.
        sls remove -v -f sayHello -s dev
    ```
- To create a simple cron job lambda function, add this to `serverless.yml`:
    ```yaml
        # Below code will execute 'cron.handler' every 1 minute
        cron:
            handler: /src/cron.handler
            events:
                - schedule: rate(1 minute)
    ```

----------------------------------------

### New Project Setup In Pre Configured Environment 101
- Browse and open terminal into empty project directory.
- Execute :
    ```sh
        # Create aws nodejs serverless template
        sls create -t aws-nodejs

        # Init npm.
        npm init -y

        # Install serverless-offline and serverless-offline-scheduler as dev dependancies.
        npm i serverless-offline serverless-offline-scheduler --save-dev
    ```
- Add following scripts under `package.json`:
    ```json
        {
            "scripts": {
                "dev": "sls offline start --port 3000",
                "dynamodb:start": "sls dynamodb start --port 8082",
            }
        }
    ```
- Open `serverless.yml` and edit `service` name as well as setup `provider`:
    ```yml
        service: s3-notifications
        provider:
            name: aws
            runtime: nodejs12.x
            region: ap-south-1
        plugins:
            - serverless-offline    # Add this plugin if you are using it.
            - serverless-offline-scheduler  # Add this plugin if you are using it.
    ```

----------------------------------------

### Installing Serverless
- To install `Serverless` globally:
    ```sh
        sudo npm i -g serverless
    ```
- For automatic updates, after above command, run:
    ```sh
        sudo chown -R $USER:$(id -gn $USER) /Users/adiinviter/.config
    ```

----------------------------------------

### Configuring AWS Credentials For Serverless
- To configure aws user credentials, run:
    ```sh
        # -o: To overwrite existing credentials if there are any set already.
        sls config credentials --provider aws --key [ACCESS_KEY] --secret [SECRET_KEY] -o
    ```
- After running above command, credentials will get set under following path:
    ```sh
        ~/.aws/credentials
    ```

----------------------------------------

### Create NodeJS Serverless Service
- Each service is a combination of multiple `Lambda Functions`.
- To create `NodeJS Serverless Service`:
    ```sh
        sls create --t aws-nodejs
    ```

----------------------------------------

### Invoke Lambda Function Locally
- To invoke a `Lambda Function` locally:
    ```sh
        # Syntax
        sls invoke local -f [FUNCTION_NAME]

        # Example
        sls invoke local -f myfunct
    ```

----------------------------------------

### Event - Passing Data To Lambda Function
- To pass data to lambda function,
    ```sh
        # Syntax
        sls invoke local -f [FUNCTION_NAME] -d [DATA]

        # Example #1: to pass a single string value into lambda function.
        sls invoke local -f sayHello -d 'Aditya'

        # Example #2: to pass a object into lambda function.
        sls invoke local -f sayHello -d '{"name": "Aditya", "age": 33}'
    ```
- `event` object holds any data passed into lambda function. To access it:
    * Accessing data directly passed as string as shown in `Example #1` above:
        ```javascript
            // Example #1: to pass a single string value into lambda function.
            // sls invoke local -f sayHello -d 'Aditya'

            module.exports.hello = async event => {
                const userName = event; // Data is available on 'event'.
                return {
                    statusCode: 200,
                    body: JSON.stringify({message: `Hello ${userName}`})
                };
            };
        ```
    * Accessing object data passed as shown in `Example #2` above:
        ```javascript
            // Example #2: to pass a object into lambda function.
            // sls invoke local -f sayHello -d '{"name": "Aditya", "age": 33}'

            module.exports.hello = async event => {
                const {name, age} = event;

                return {
                    statusCode: 200,
                    body: JSON.stringify({message: `Hello ${name}, Age: ${age}`})
                };
            };
        ```

----------------------------------------

### Serverless Offline
- For **local development only**, use `Serverless Offline` plugin.
- Plugin:
    ```
        https://www.npmjs.com/package/serverless-offline
        https://github.com/dherault/serverless-offline
    ```
- To install:
    ```sh
        npm i serverless-offline --save-dev
    ````

----------------------------------------

### NPM Run Serverless Project Locally
- Install [Serverless Offline](#serverless-offline) plugin.
- Under `serverless.yml`, add:
    ```yml
        plugins:
            - serverless-offline
    ```
- Under `package.json`, add new run script:
    ```json
        "dev": "sls offline start --port 3000"
    ```
- Run:
    ```sh
        npm run dev
    ```

----------------------------------------

### Deploy Serverless Service
- To deploy serverless service, run:
    ```sh
        # -v: For verbose.
        sls deploy -v
    ```

----------------------------------------

### Setup Serverless DynamoDB Local
- Use following plugin to setup DynamoDB locally (for offline uses):
    ```
        https://www.npmjs.com/package/serverless-dynamodb-local
        https://github.com/99xt/serverless-dynamodb-local#readme
    ```
- To setup:
    ```sh
        npm i serverless-dynamodb-local
    ```
- Register `serverless-dynamodb-local` into serverless yaml:
    ```yml
        plugins:
            - serverless-dynamodb-local
    ```
- Install DynamoDB into serverless project:
    ```sh
        sls dynamodb install
    ```

----------------------------------------

### Securing APIs
- APIs can be secured using `API Keys`.
- To generate and use `API Keys` we need to modify `serverless.yml` file:
    * Add `apiKeys` section under `provider`:
        ```yml
            provider:
                name: aws
                runtime: nodejs12.x
                ########################################################
                apiKeys:                # For securing APIs using API Keys.
                    - todoAPI           # Provide name for API Key.
                ########################################################
                stage: dev              # Stage can be changed while executing deploy command.
                region: ap-south-1      # Set region.
                timeout: 300
        ```
    * Route by Route, specify whether you want it to be `private` or not. For e.g.
        ```yml
            functions:
                getTodo:    # Secured route.
                    handler: features/read.getTodo
                    events:
                        - http:
                            path: todo/{id}
                            method: GET
                            ########################################################
                            private: true   # Route secured.
                            ########################################################
                listTodos:  # Non-secured route.
                    handler: features/read.listTodos
                    events:
                        - http:
                            path: todos
                            method: GET
        ```
    * After deploying we will receive `api keys`. Copy it to pass it under headers.
        ```sh
            λ serverless offline start
            Serverless: Starting Offline: dev/ap-south-1.
            Serverless: Key with token: d41d8cd98f00b204e9800998ecf8427e    # Here is our API Key token.
            Serverless: Remember to use x-api-key on the request headers
        ```
    * Pass `api key` under `x-api-key` header while hitting secured route.
        ```javascript
            x-api-key: d41d8cd98f00b204e9800998ecf8427e
        ```
    * If a wrong/no value is passed under `x-api-key` header, then we will receive `403 Forbidden` error.

----------------------------------------

### AWS CLI Handy Commands
- Useful commands for project `05-S3-Notifications`:
    * Setup aws profile for `Serverless S3 Local` plugin:
        ```sh
            aws s3 configure --profile s3local

            # Use following credentials:
            # aws_access_key_id = S3RVER
            # aws_secret_access_key = S3RVER
        ```
    * Trigger S3 event - Put file into local S3 bucket:
        ```sh
            aws --endpoint http://localhost:8000 s3api put-object --bucket "aditya-s3-notifications-serverless-project" --key "ssh-config.txt" --body "D:\Work\serverless\05-S3-Notifications\tmp\ssh-config.txt" --profile s3local
        ```
    * Trigger S3 event - Delete file from local S3 bucket:
        ```sh
            aws --endpoint http://localhost:8000 s3api delete-object --bucket "aditya-s3-notifications-serverless-project" --key "ssh-config.txt" --profile s3local
        ```

----------------------------------------

### Common Issues
- After running `sls deploy -v`, error: **`The specified bucket does not exist`**:
    * **Cause:** This issue occurs when we manually delete S3 bucket from AWS console.
    * **Fix:** Login to AWS console and delete stack from `CloudFormation`.
    * **Dirty Fix (Avoid):** Delete `.serverless` directory from project (Serverless Service).
    * **Full Error (Sample):**
    ```sh
        Serverless: Packaging service...
        Serverless: Excluding development dependencies...
        Serverless: Uploading CloudFormation file to S3...

        Serverless Error ---------------------------------------

        The specified bucket does not exist

        Get Support --------------------------------------------
            Docs:          docs.serverless.com
            Bugs:          github.com/serverless/serverless/issues
            Issues:        forum.serverless.com

        Your Environment Information ---------------------------
            Operating System:          darwin
            Node Version:              13.7.0
            Framework Version:         1.62.0
            Plugin Version:            3.3.0
            SDK Version:               2.3.0
            Components Core Version:   1.1.2
            Components CLI Version:    1.4.0
    ```

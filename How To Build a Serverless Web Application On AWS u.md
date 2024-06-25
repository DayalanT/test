# Architect and Build an End-to-End AWS Serverless web Application from Scratch

As a beginner on the cloud, it's natural to be concerned about the costs of deploying resources for your projects on AWS. Here's a solution: a simple serverless architecture using AWS Lambda and API Gateway. With this setup, you'll only be charged based on the number of invocations, meaning you pay only for the compute time when your code is actually running. This cost-effective approach allows you to experiment and build without worrying about excessive charges

For this project, we’ll pick five different services from AWS -
**Amplify, Lambda, IAM, API Gateway and DynamoDB**

**AWS Amplify** - Amplify is like a Swiss Army knife for developers, providing all the necessary tools to build, manage, and scale applications with less hassle and more efficiency. In simple words, it helps to **Build & Host** create the back-end (server side) of your application quickly, without needing to worry about the complex infrastructure. It also allows you to host and manage your front-end (user interface) easily

**API Gateway** - Think of API Gateway as the front door to your application's backend services. It's like a receptionist that directs traffic to the correct department. When someone (a user or another system) wants to interact with your application, they make a request through this front door

**Lambda** - With Lambda, you don't have to worry about the underlying servers where your code runs. AWS takes care of all the infrastructure for you. It’s like cooking in a kitchen where you don’t have to clean up—AWS handles all the messy parts

**DynamoDB** - DynamoDB is a fully managed NoSQL database service, Unlike traditional relational databases that use tables, rows, and columns, DynamoDB uses a more flexible data model, which can store and retrieve any amount of data and handle any level of request traffic. It's perfect for applications that need to process large volumes of data quickly
***

## Application architecture



![Amplify-Page-2.png](../_resources/Amplify-Page-2.png)



**Step 1: Create a new file index.html**
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>To the Power of Math!</title>
</head>

<body>
    To the Power of Math!
</body>
</html>
```

#
**Step 2: Log on to AWS - Navigate to AWS Amplify**

![amlify-openingpage.png](../_resources/amlify-openingpage.png)

- Create a New App and Use “Deploy without Git


![naming.png](../_resources/naming.png)


- Update the App & Branch name
- Zip your index.html file and drop the folder to deploy



![link-page2.png](../_resources/link-page2.png)


After the deployment, ensure the application serves as expected (Visit deployed URL)
#
**Step 3: Navigate to AWS Lambda**


![Lambda2.png](../_resources/Lambda2.png)

- Create a New function (Author from scratch), name the function & Runtime **Python 3.9**, and leave other options as default



![sourcecode.png](../_resources/sourcecode.png)


- Navigate to the Code-source. Upload & Deploy the code
```
# import the JSON utility package
import json
# import the Python math library
import math

# define the handler function that the Lambda service will use an entry point
def lambda_handler(event, context):

# extract the two numbers from the Lambda service's event object
    mathResult = math.pow(int(event['base']), int(event['exponent']))

    # return a properly formatted JSON object
    return {
    'statusCode': 200,
    'body': json.dumps('Your result is ' + str(mathResult))
    }
```

- Now, Configure & Run the event "Test"
```
{
    "base":10,
    "exponent":10
}
```


![test.png](../_resources/test.png) 
- Ensure the result returns with status code 200 

![result.png](../_resources/result.png)
#
**Step 4: Navigate to AWS API Gateway**
- Use the REST API to build & create a new API with default options


![api.png](../_resources/api.png)
- Now, create a new **post** method using lambda intergration, select the region & lambda and leave the other option as the default


![post2.png](../_resources/post2.png)

API gateway needs to access Lamba to trigger the function
- Click on the ‘/’ path and enable CORS


![cors.png](../_resources/cors.png)
- Enable "**PUT**" option check box and leave the other option as the default


![settings.png](../_resources/settings.png)

- Now, Deploy API and Update the stage details
![deployapi.png](../_resources/deployapi.png)


- then, Copy the Invoke URL link from the stage details (save it notes for later)

![stage.png](../_resources/stage.png)


- Now test the functionality. Use the below function on the request body, and ensure it results with 200 status code
![testapi.png](../_resources/testapi.png)
```
{
    "base":10,
    "exponent":10
}
```
#
**Step 5: Navigate to AWS DynamoDB**
- Create a new DynamoDB (Leave the other option as the default)


![dynamodb.png](../_resources/dynamodb.png)
- Copy the ARN details from the additional info (save it for later)

#
**Step 6: Permission to lambda & update the code**
- Head back to the lambda and click on the newly created role
![createdrole.png](../_resources/createdrole.png)
- Create a new inline policy on the permission tab to allow **Lambda** to access **DynamoDB**. Use the below JSON to update the policy

![policy.png](../_resources/policy.png)
```
{
"Version": "2012-10-17",
"Statement": [
    {
        "Sid": "VisualEditor0",
        "Effect": "Allow",
        "Action": [
            "dynamodb:PutItem",
            "dynamodb:DeleteItem",
            "dynamodb:GetItem",
            "dynamodb:Scan",
            "dynamodb:Query",
            "dynamodb:UpdateItem"
        ],
        "Resource": "<<YOUR-TABLE-ARN>>"
    }
    ]
}
```

- Replace "YOUR-TABLE-ARN" with DynamoDB ARN and save the policy. Now the lambda should have write access to DyanmoDB

**Now, its time to update the lambda function to push the result onto the DB**
- Update the existing code with newer on the Lambda. Save & Deploy
- Replace “table = dynamodb.Table('Name-of-your-DynamoDB')”
```
# import the JSON utility package
import json
# import the Python math library
import math

# import the AWS SDK (for Python the package name is boto3)
import boto3
# import two packages to help us with dates and date formatting
from time import gmtime, strftime

# create a DynamoDB object using the AWS SDK
dynamodb = boto3.resource('dynamodb')
# use the DynamoDB object to select our table
table = dynamodb.Table('<<Name-of-the-DynamoDB>>')
# store the current time in a human readable format in a variable
now = strftime("%a, %d %b %Y %H:%M:%S +0000", gmtime())

# define the handler function that the Lambda service will use an entry point
def lambda_handler(event, context):

# extract the two numbers from the Lambda service's event object
    mathResult = math.pow(int(event['base']), int(event['exponent']))

# write result and time to the DynamoDB table using the object we instantiated and save response in a variable
    response = table.put_item(
        Item={
            'ID': str(mathResult),
            'LatestGreetingTime':now
            })

# return a properly formatted JSON object
    return {
    'statusCode': 200,
    'body': json.dumps('Your result is ' + str(mathResult))
    }
```

- Now, test the code and the ensure data's are pushed to DyanomoDB
- Jump back to DynamoDB, Click on "**Explore table items**" items and the data’s should be saved


![dy.png](../_resources/dy.png)

**Now, its time to update the index.html page on Amplify**
#
**Step 7: Open up notepad and Create a new file index.html**
- Create a new file index.html (from below) and zip the file to upload
- Replace “fetch("**YOUR API GATEWAY ENDPOINT**", requestOptions)” with the Gateway Endpoint
```
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>To the Power of Math!</title>
    <!-- Styling for the client UI -->
    <style>
    h1 {
        color: #FFFFFF;
        font-family: system-ui;
		margin-left: 20px;
        }
	body {
        background-color: #222629;
        }
    label {
        color: #86C232;
        font-family: system-ui;
        font-size: 20px;
        margin-left: 20px;
		margin-top: 20px;
        }
     button {
        background-color: #86C232;
		border-color: #86C232;
		color: #FFFFFF;
        font-family: system-ui;
        font-size: 20px;
		font-weight: bold;
        margin-left: 30px;
		margin-top: 20px;
		width: 140px;
        }
	 input {
        color: #222629;
        font-family: system-ui;
        font-size: 20px;
        margin-left: 10px;
		margin-top: 20px;
		width: 100px;
        }
    </style>
    <script>
        // callAPI function that takes the base and exponent numbers as parameters
        var callAPI = (base,exponent)=>{
            // instantiate a headers object
            var myHeaders = new Headers();
            // add content type header to object
            myHeaders.append("Content-Type", "application/json");
            // using built in JSON utility package turn object to string and store in a variable
            var raw = JSON.stringify({"base":base,"exponent":exponent});
            // create a JSON object with parameters for API call and store in a variable
            var requestOptions = {
                method: 'POST',
                headers: myHeaders,
                body: raw,
                redirect: 'follow'
            };
            // make API call with parameters and use promises to get response
            fetch("YOUR API GATEWAY ENDPOINT", requestOptions)
            .then(response => response.text())
            .then(result => alert(JSON.parse(result).body))
            .catch(error => console.log('error', error));
        }
    </script>
</head>
<body>
    <h1>TO THE POWER OF MATH!</h1>
	<form>
        <label>Base number:</label>
        <input type="text" id="base">
        <label>...to the power of:</label>
        <input type="text" id="exponent">
        <!-- set button onClick method to call function we defined passing input values as parameters -->
        <button type="button" onclick="callAPI(document.getElementById('base').value,document.getElementById('exponent').value)">CALCULATE</button>
    </form>
</body>
</html>
```

- Now, drop the zip folder to re-deploy
- Ensure the Web page results as expected


![app.png](../_resources/app.png)




**Optional: You can use your custom DNS to update the records; Use any DNS manager (Go Daddy, Route-53, etc.) Refer to the official documentation from AWS on configuring the domain** -  [AWS Amplify](https://docs.aws.amazon.com/amplify/latest/userguide/custom-domains.html)
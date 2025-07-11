# AWS-serverless-contact-app
**Built a fully serverless, secure, and scalable contact web app using AWS S3, CloudFront, API Gateway, Lambda (Python), and DynamoDB, secured with ACM TLS and deployed with custom domain routing**

### Step 1: Create S3 Bucket (Static Website Hosting)

**1. Go to S3 →** Create bucket

   * **Name:** serverless-contact-app-frontend
   * **Uncheck Block all public access**

**2. Go to Properties →** Enable Static Website Hosting

   * **Index document:** index.html

**3. Upload index.html and app.js**

   * **Set Metadata →** Content-Type (Set the metadata seperately firstly add object index.html and metadata and upload do same thing for app.js)
   * **index.html:** text/html
   * **app.js:** application/javascript

#
* **index.html**

```
  <!DOCTYPE html>
<html>
<head>
  <title>Contact Us</title>
</head>
<body>
  <h1>Contact Form</h1>
  <form id="contactForm">
    <input type="text" id="name" placeholder="Name" required><br>
    <input type="email" id="email" placeholder="Email" required><br>
    <textarea id="message" placeholder="Message" required></textarea><br>
    <button type="submit">Send</button>
  </form>
  <p id="response"></p>
  <script src="app.js"></script>
</body>
</html>
```

* **app.js**

```
  document.getElementById("contactForm").addEventListener("submit", async function (e) {
  e.preventDefault();

  const data = {
    name: document.getElementById("name").value,
    email: document.getElementById("email").value,
    message: document.getElementById("message").value,
  };

  const response = await fetch("https://YOUR-API-ID.execute-api.amazonaws.com/contact", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(data),
  });

  const result = await response.text();
  document.getElementById("response").innerText = result;
});
```

#
### Step 2: Create DynamoDB Table

  * **Go to DynamoDB →** Create table
  * **Table name:** ContactMessages
  * **Partition key:** email (String)
  * **Sort key:** timestamp (String)
  * **Default settings →** Create

#
### Step 3: Create Lambda Function (Python)

  * Go to Lambda → Create function
  * Name: saveContact
  * Runtime: Python 3.11
  * Permissions: Create a new role with basic Lambda permissions
#
* **Lambda code**

```
  import json
import boto3
from datetime import datetime

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('ContactMessages')

def lambda_handler(event, context):
    body = json.loads(event['body'])

    table.put_item(Item={
        'email': body['email'],
        'timestamp': datetime.utcnow().isoformat(),
        'name': body['name'],
        'message': body['message']
    })

    return {
        'statusCode': 200,
        'headers': {
            'Access-Control-Allow-Origin': '*',
            'Access-Control-Allow-Headers': '*'
        },
        'body': json.dumps({'message': 'Success'})
    }
```

  * Click Deploy
#



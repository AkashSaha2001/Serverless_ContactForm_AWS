# Creating a Serverless Contact Form using AWS API Gateway and Lambda
```
- /
  	- website/
   		 - index.html
    		- style.css
   		 - script.js
  	- lambda/
    		- index.js
   		- package.json
```
![image](https://github.com/AkashSaha2001/Serverless_ContactForm_AWS/assets/91005784/8dfa083e-a5b9-42fa-8ff2-cbec17e8e4d3)
Create the static website with the HTML form. Write JavaScript code to handle form submission and make an HTTP POST request to the API Gateway endpoint.
## index.html
```ruby
<!DOCTYPE html>
<html>
<head>
  <title>Contact Form</title>
  <link rel="stylesheet" type="text/css" href="style.css">
</head>
<body>
  <div class="container">
    <h1>Contact Form</h1>
    <form id="contact-form">
      <label for="name">Name:</label>
      <input type="text" id="name" name="name" required>

      <label for="email">Email:</label>
      <input type="email" id="email" name="email" required>

      <label for="subject">Subject:</label>
      <input type="text" id="subject" name="subject" required>

      <label for="message">Message:</label>
      <textarea id="message" name="message" rows="5" required></textarea>

      <button type="submit">Submit</button>
    </form>
  </div>
  
  <script src="script.js"></script>
</body>
</html>
```
## style.css
```ruby
.container {
    max-width: 500px;
    margin: 0 auto;
    padding: 20px;
  }
  
  form label {
    display: block;
    margin-bottom: 10px;
  }
  
  form input,
  form textarea {
    width: 100%;
    padding: 10px;
    margin-bottom: 15px;
  }
  
  form button {
    padding: 10px 20px;
    background-color: #007bff;
    color: #fff;
    border: none;
    cursor: pointer;
  }
  
  form button:hover {
    background-color: #0069d9;
  }
```
## script.js
```ruby
document.getElementById('contact-form').addEventListener('submit', function(event) {
  event.preventDefault(); // Prevent form submission
  // Validate form inputs
  var name = document.getElementById('name').value.trim();
  var email = document.getElementById('email').value.trim();
  var subject = document.getElementById('subject').value.trim();
  var message = document.getElementById('message').value.trim();
  // Email validation using a regular expression
  var emailRegex = /^\S+@\S+\.\S+$/;
  if (!name || !email || !subject || !message || !emailRegex.test(email)) {
    alert('Please fill in all fields with valid inputs.');
    return;
  }
  // Construct the form data
  var formData = {
    name: name,
    email: email,
    subject: subject,
    message: message
  };
  // Perform form submission
  submitForm(formData);
});
function submitForm(formData) {
  // Make an API request to the backend (API Gateway) for form submission
  fetch('your-api-gateway-url', { // URL that represents the backend API endpoint to which the form data is going to be sent
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(formData)
  })
  .then(function(response) {
    if (response.ok) {
      // Redirect to the thank you page
      window.location.href = 'thank-you.html';
    } else {
      throw new Error('Form submission failed.');
    }
  })
  .catch(function(error) {
    console.error(error);
    alert('Form submission failed. Please try again later.');
  });
}
```
## thank-yo.html
```ruby
<!DOCTYPE html>
<html>
<head>
  <title>Thank You</title>
  <link rel="stylesheet" type="text/css" href="style.css">
</head>
<body>
  <div class="container">
    <h1>Thank You!</h1>
    <p>Your message has been submitted successfully.</p>
  </div>
</body>
</html>
```
The structure will be this :

![image](https://github.com/AkashSaha2001/Serverless_ContactForm_AWS/assets/91005784/730eb590-7b77-4851-80aa-f462e9f48b81)

## Activity 1 : Create a Lambda function
![image](https://github.com/AkashSaha2001/Serverless_ContactForm_AWS/assets/91005784/9ab2b7f7-9149-4e32-b38b-1ab9d6f52fa9)
![image](https://github.com/AkashSaha2001/Serverless_ContactForm_AWS/assets/91005784/8a867871-c3eb-4630-b4f1-55d4d7ebf513)
Now click on Create.

Create another folder in your system named as lambda. 
## index.js
```ruby
const AWS = require('aws-sdk');
const dynamodb = new AWS.DynamoDB.DocumentClient();
const { v4: uuidv4 } = require('uuid');

exports.handler = async (event) => {
  try {
    console.log('Raw input data:', event); // Add this line to log the raw input data

    const formData = {
      name: event.name,
      email: event.email,
      subject: event.subject,
      message: event.message,
    };

    const item = {
      SubmissionId: generateUUID(), // Generate a UUID
      ...formData, // Use the form data as attributes
    };

    // Store the form data in DynamoDB
    await storeFormData(item);

    return {
      statusCode: 200,
      body: JSON.stringify({ message: 'Form submitted successfully' }),
    };
  } catch (error) {
    console.error(error);
    return {
      statusCode: 500,
      body: JSON.stringify({ message: 'Error submitting the form' }),
    };
  }
};
async function storeFormData(item) {
  const params = {
    TableName: 'ContactFormEntries',
    Item: item,
  };
  await dynamodb.put(params).promise();
}
function generateUUID() {
  return uuidv4();
}
```
## package.json
```ruby
{
    "name": "serverless-contact-form",
    "version": "1.0.0",
    "description": "Lambda function for handling the serverless contact form",
    "main": "index.js",
    "dependencies": {
        "aws-sdk": "^2.1386.0"
    }
}
```

Move to the lambda directory and install the AWS SDK by running the command below. The npm install is used to install the dependencies listed in the package.json file of the project.
```
npm install
```
This will be the structure :
![image](https://github.com/AkashSaha2001/Serverless_ContactForm_AWS/assets/91005784/3659b9ad-5e6a-410c-9dce-0bd733934216)
Zip the folder [lambda.zip]


## Activity 2 : Create a Bucket in S3
![image](https://github.com/AkashSaha2001/Serverless_ContactForm_AWS/assets/91005784/eab28bbd-06ee-4266-bb0c-b320865bc6a7)
![image](https://github.com/AkashSaha2001/Serverless_ContactForm_AWS/assets/91005784/9a468a93-347c-4d40-a183-dc3ebfce5f86)
![image](https://github.com/AkashSaha2001/Serverless_ContactForm_AWS/assets/91005784/d6b09927-d785-4a13-8990-423a7315637f)
Upload the Zip file here 
![image](https://github.com/AkashSaha2001/Serverless_ContactForm_AWS/assets/91005784/2195287a-3aae-46d6-857a-6291e95823fc)
![image](https://github.com/AkashSaha2001/Serverless_ContactForm_AWS/assets/91005784/85cc0f08-95b9-40bf-9dc4-32df389381e8)
Upload the file.
Go to the lambda.zip and Copy the Object URL.
![image](https://github.com/AkashSaha2001/Serverless_ContactForm_AWS/assets/91005784/eb9f374c-39e5-4f8c-8b28-72007abe7381)
Now go to the lambda function and upload the zip by attaching the S3 location here.
![image](https://github.com/AkashSaha2001/Serverless_ContactForm_AWS/assets/91005784/63fcd0dd-ed51-4d5c-b940-b1c2b64ccf30)
![image](https://github.com/AkashSaha2001/Serverless_ContactForm_AWS/assets/91005784/b8e22d25-a988-4359-b009-812e26f9082f)
Now go to Test
![image](https://github.com/AkashSaha2001/Serverless_ContactForm_AWS/assets/91005784/ef681bea-06db-46d3-93c5-d79214b8363a)
Edit the JSON code:
```ruby
{
  "name": "Akash Saha",
  "email": "mr.akashsaha574@gmail.com",
  "subject": "It's Me",
  "message": "Testing"
}
```
![image](https://github.com/AkashSaha2001/Serverless_ContactForm_AWS/assets/91005784/e5eb396f-c247-42e3-b329-3161d8123e36)
Click on the Test to check if it is getting successfully executed or not.
![image](https://github.com/AkashSaha2001/Serverless_ContactForm_AWS/assets/91005784/3b0f4910-b52d-4c8d-a988-ff43f708f76b)

## Activity 3 : Create a API gateway
select REST API Private and Build
![image](https://github.com/AkashSaha2001/Serverless_ContactForm_AWS/assets/91005784/2b1687fa-d6a2-4307-b335-14a6d3913d17)
![image](https://github.com/AkashSaha2001/Serverless_ContactForm_AWS/assets/91005784/8c01e216-5914-479a-a446-83df08fbf814)
Create Resource
![image](https://github.com/AkashSaha2001/Serverless_ContactForm_AWS/assets/91005784/4239c096-766c-439d-8fa8-ab6c672cdf18)
![image](https://github.com/AkashSaha2001/Serverless_ContactForm_AWS/assets/91005784/4ddf4e48-a2c1-4df0-a899-575a58c3de20)
Create Method


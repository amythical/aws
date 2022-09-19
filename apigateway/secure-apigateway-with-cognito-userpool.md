# Use case
Secure a API Gateway URL so that only users logged in using Cognito User Pool can access it. Accessing the API Gateway URL without Cognito creds should give an error.

# Approach
API Gateway URL has an authorisation section where Cognito Pool can be configured as an Authorisation Source.
The Cognito token needs to be passed to the API Gateway while calling it.

# Implementation
## Create a Lambda function
```
index.js
------------
exports.handler = async (event) => {
    // returns a url in response
    const tUrl = 'example.com';
    const responseOK = {
        statusCode: 200,
        body: JSON.stringify({url:`${tUrl}`}),
    };
    return responseOK;
};

```

## Create a REST API
- AWS Console -> API Gateway -> REST API (NOT PRIVATE) -> Build
- REST API, New API, Give it a name, End Point Type Regional and Create API
- Left Panel -> Resources -> Actions drop down -> Create method -> GET
- After saving GET as a method, we see the GET Setup screen
- Integration type select Lambda, check Lamda proxy (see [Lamda proxy vs Integration](https://github.com/amythical/aws/blob/main/apigateway/lambda-proxy-Vs-lambda-integration.md))
- Add Lambda region, lambda function name
- Save
## 

## Deploy the API
- Left Panel -> Resources and select /
- Right Panel -> Actions -> Deploy API
- Pop up with Deployment stage and Description
- Deployment stage -> New Stage
- Give the Stage a name, description and deployment description
- Click Deploy

## Check URL
- Left Panel -> Dashboard
- On the Right Panel you should see a link like https://abcde12345.execute-api.ap-south-1.amazonaws.com/your-stage-name-here/
- Open POSTMAN, use a GET method and call this URL
- You should see {url: 'example.com'}, thats our Lambda response
- So this API Gateway URL is now publicly accessible

## Securing with a Cognito Authorizer
- If you do not have a User Pool in Cognito, create a User Pool first [see link here](See LINK HERE)
- Authorizers -> Create New Authorizer
- Give the authorizer a name
- Type, select Cognito
- If you have created a User Pool in Cognito it should be available in the Cognito User pool Drop down
- Token Source enter 'Authorization' (without the single quotes). This is the field in the header which has the Cognito Token
- Token Validation is optional and can be left blank
- Click on Create


## Testing JWT Token in the AWS console
- Login to your app and get the JWT Token. If you are unfamiliar with ID Token, JWT Token [here is a link] (SEE LINK HERE)
- After creating the Authorizer above, you should see your authorizer in a widget on the Right Panel on the Authorizers page, click on Test
- A popup with Authorization (header) as a field label appears
- Enter the JWT Token in this Authorization field
- You should see a response in the panel with claims or Unauthorized error if the JWT Token doesnt have access

## Securing the GET Request
- Left Panel -> Resources
- Refresh the page (just to ensure we get all values populated again)
- Right Panel -> GET
- Click on Method Request
- Authorization drop down -> select the Authorizer name (The one we created in 'Securing with a Cognito Authorizer' section)
- Request Validator None
- API Key required false
- Click the small Tick circle next to Authorization select control to save changes (I missed this the first time!)
- if you go to Left panel -> Resources, the Right Panel should show a widget with GET and Authorization as COGNITO_USER_POOLS


## Testing
- Open POSTMAN, the API gateway URL viz https://abcde12345.execute-api.ap-south-1.amazonaws.com/your-stage-name-here/ (Find it at Left panel->Dashboard)
- Make a call to the url
- Oops we still can see the response {url: 'example.com'} so the API IS NOT SECURE yet!!

## Deploy again
- Left Panel -> Resources 
- Right Panel /
- Right Panel Actions -> Deploy API
- Select Stage we created earlier in ('Deploy the API)
- Enter Description if you would like
- Click on Deploy

## Testing again 
Open POSTMAN, the API gateway URL viz https://abcde12345.execute-api.ap-south-1.amazonaws.com/your-stage-name-here/ (Find it at Left panel->Dashboard)
- Make a call to the url
- You should see the response 
{
    "message": "Unauthorized"
}
- Conclusion : Users not logged in with Cognito cannot acces this URL

Now lets see if Cognito Users can access the URL
- Open POSTMAN, Copy a JWT Token from your app and add it in POSTMAN Headers [New to JWT Tokens ?](Link here)
- Create a Header key called Authorization (No quotes to be added, by the way - this is the same key we added in Token Source in the 'Securing with a Cognito Authorizer' section)
- Enter the JWT Token value in the Authorization header
- Make a call to https://abcde12345.execute-api.ap-south-1.amazonaws.com/your-stage-name-here/
- We see the response {url: 'example.com'}, so yay!
- Conclusion - Success! Cognito Users can access the API
- Note : If the JWT token is old/expired you will see this response {
    "message": "The incoming token has expired"
}, so generate a new JWT Token and add it to the Authorization header in POSTMAN and try again



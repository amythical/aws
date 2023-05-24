# Use case
Give a s3 bucket access only to users signed in through our Cognito User pool. 

# Why is this not so straightforward to begin with?
 - When a user logs in using a Cognito User Pool, the user has no IAM Role assigned
 - We assign permissions on S3 Buckets to IAM Roles
 - The entire battle is about extracting creds for the IAM and associating it with a Cognito User pool user OR exchanging userpool tokens for identity pool tokens

# Approach 1 (using federated user)
- User pool -> Identity Pool -> IAM Role with access to S3 Bucket
- Request with Authorization header to Lambda -> extract Authorization header -> get JWT Token -> get CognitoIdentity using JWT Token -> get Credentials for this Identity -> use these credentials to access S3 Bucket

# Approach 2 ( using User pool Groups ) - TO BE DETAILED
- IAM Role with access to S3 Bucket
- User pool, User Pool Groups -> assign IAM Role
- Request with Authorization header to Lambda -> extract Authorization header -> get JWT Token -> get roles from request using cognito:roles -> use STS to assume this role in Lambda and access the S3 Bucket

# Implementation for Approach 1 (using federated user)
- Create a IAM user 
- Create a user pool
- Create an Identity pool and assign the user pool as the authentication source and assign the IAM user role to the identity pool [LINK HERE](http://comingsoon.com)
- Secure the lambda function with API gateway -
 configure the APIG with user pool authentication so that only authenticated users can access the lambda function [ link available here](https://github.com/amythical/aws/blob/main/apigateway/secure-apigateway-with-cognito-userpool.md)
- Pass the authorization header to the APIG url, if you are using Axios here is a code snippet
```
const axiosConfig = {
      headers: {
        Authorization: `${token}`,
      },
    };
    axios
      .get(`${API_GATEWAY_GET_COMMON_ASSETS_URL}`, axiosConfig)
```
- The lambda function
```
mylambdafunction.js

const AWS = require('aws-sdk');

// var jwt = require('jsonwebtoken');


exports.handler = async (event) => {      
    let tAuthorizationToken = event.headers['Authorization'];
    // let tDecodedToken = jwt.decode(tAuthorizationToken);
    //  console.log("cognito:roles ="+tDecodedToken['cognito:roles']);
     
    // Get Cognito idnentity
    const cognitoidentity = new AWS.CognitoIdentity({ apiVersion: '2014-06-30'});
    var params1 = {
        IdentityPoolId: 'your-region-1:your-identity-pool-id', 
        Logins: {
				"cognito-idp.your-region.amazonaws.com/your-region-identity-pool-id":tAuthorizationToken 
        }
    };
    let cognitoid = null;
    try{
        cognitoid = await cognitoidentity.getId(params1).promise();
    }catch(err){
        console.log(err, err.stack); // an error occurred
    }

    // Get Credentials for this identity
    var params2 = {
    IdentityId: cognitoid.IdentityId, 
    // CustomRoleArn: 'STRING_VALUE',
    Logins: {
				"cognito-idp.your-region.amazonaws.com/your-region-identity-pool-id":tAuthorizationToken 
        }
    };
    let creds = null;
    try{
        creds = await cognitoidentity.getCredentialsForIdentity(params2).promise();
    }catch(err){
        console.log(err, err.stack); // an error occurred
    }
 

    // Access S3 with new credentials
    let s3 = null;
    try{
        s3 = new AWS.S3({
            accessKeyId: creds.Credentials.AccessKeyId,
            secretAccessKey: creds.Credentials.SecretKey,
            sessionToken:creds.Credentials.SessionToken}
        );
    }catch(ex){
        console.log("Error creating S3 with creds "+ex);
    }
    var params3 = { 
            Bucket: 'your-bucket-name'
    };

    let s3Objects = null;
    try {
       s3Objects = await s3.listObjectsV2(params3).promise();
       console.log("listing s3 Objects");
       console.log(s3Objects);
    } catch (e) {
        console.log("s3 error");
        console.log(e);
    }
```

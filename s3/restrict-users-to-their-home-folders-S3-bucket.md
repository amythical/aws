# Use Case
You want to restrict each user to access a specific folder which belongs to them kinda like their home directory, in a single S3 bucket.
User A should not be able to access user Bs folder in the same S3 bucket.

# Approach
One approach is simple but is based on IAM users [on the aws blogs](https://aws.amazon.com/blogs/security/writing-iam-policies-grant-access-to-user-specific-folders-in-an-amazon-s3-bucket/).

In webapps this is not practical creating an IAM user for each signup, I used Cognito/IAM Roles for such access.
So here is the approach using Cognito and IAM Role.

 - Create a S3 bucket with private access
 - Create a user pool 
 - Create an identity pool with authentication provider as the Cognito User pool and let the Identity pool create an IAM role for the authenticated users
 - When a user signs-up fetch the sub attribute from Cognito and create a subfolder inside the s3 bucket
 - Create the policy for the IAM role using policy
  variables allowing access to specific folders using policy variable names
- In the app, exchange JWT Token for IDP creds and access S3 thus bringing the IAM Policy into play
- Create presigned S3 urls for a user to view their files in their home folder
- The policy restricts user A from accessing any other users home folder in the same S3 Bucket

# Implementation
## Creating a private S3 bucket
- Follow all the isntructions from [this post](https://github.com/amythical/aws/blob/0896364bfbf70c8ff67268a1d07f8c0e236f80e3/s3/pre-signed-url-private-S3-bucket.md) to create a private S3 Bucket, configure CloudFront behaviour path pattern

## Creating a User Pool
[Post coming soon TBD]()
## Create an Identity Pool
 - AWS Services -> Cognito -> Manage Identity Pool -> Create New Identity Pool 
 - Give it a name
 - Authentication providers -> Cognito and from the User Pool get the User Pool ID and App Client ID
 - Create Pool
 - Create a new IAM Role for the Authenticated User, give it a name
 - This is the IAM Role we will use further to assign permissions through policies

 ## Cognito User Pool Attributes
When you create a Cognito User Pool there are some default attributes like name, email, phone and you can define your custom attributes.
"sub" stands for [subject](https://stackoverflow.com/questions/42645932/aws-cognito-difference-between-cognito-id-and-sub-what-should-i-use-as-primary), is another default attribute thats unique to each user. Why subs? [Post TBD]()

When a user signs up, create the home folder in the S3 private bucket by creating a folder and naming it with the same value as a sub, this will look like 'ABC-123456-EFG-123'

## Create a Policy for the IAM Role giving access to 'Home folder' in S3
Here is the Policy JSON
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowAllS3ActionsInUserFolder",
            "Action": [
                "s3:*"
            ],
            "Effect": "Allow",
            "Resource": [
                "arn:aws:s3:::private-bucket-name/${aws:username}/*"
            ]
        }
    ]
}
// aws:username maps to the sub of the user in the Cognito User Pool*
```
Note 1 - There is also a sub associated with the Identity pool [aws blog here] (https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_examples_s3_cognito-bucket.html) on using that in the policy, its not so clear to me until I work it out in a working use case

Note 2 - If you are following the bucket creation from [this post](https://github.com/amythical/aws/blob/main/cloudfront/cloudfront-serve-content-from-private-s3-bucket.md) then the Resource in the above policy changes to 
 - "Resource": [
                "arn:aws:s3:::private-bucket-name/private/${aws:username}/*"
            ]
 - Why the folder named 'private' - We use behaviours in the CloudFront distribution and the 'private' folder maps the path pattern
 
 Note 3 - You can find the username mapped to the sub in the Identity pool under Authentication providers, Cognito tab after linking your user pool, Attributes for access control, select Use default mappings (Click on the select option which defaults to Use custom mappings). These are called **Policy Variables** as they can now be used in an IAM Policy

 Note 4 - You can also use custom attributes instead of subs, custom attributes are mapped in the Identity pool under Authentication providers, Cognito tab after linking your user pool, Attributes for access control. We can use custom attributes in the policy using principal tags '${aws:PrincipalTag/yourcustomtag}'

 ## Exchanging JWT Tokens for Credentials in the Cognito Identity Pool
 There are numerous ways to achieve this, Im documenting what worked for me, will add more methods or turn this into a separate post later

 ```
 // IMPORTANT - We are using await here so place this in an async function, or modify this code accordingly if not async

 // Obtain the IdToken
let tAuthorizationToken = IdToken.jwtToken;

const cognitoidentity = new AWS.CognitoIdentity({
    apiVersion: '2014-06-30',
});
// Get the Issuer from the JWT Token
let tISS = pIdToken.payload.iss.replace('https://', ''); // eg iss gives https://cognito-idp.aws-region.amazonaws.com/aws-region_abcd1234

// 1. Get the Cognito Id from the Identity Pool
/* Ideally read this from an env variable.
 Get it from AWS Cognito -> Manage Identity Pools-> Your Identity pool -> Edit identity pool */
const YOUR_IDENTITYPOOLID = 'ABCD-1234';

let params1 = {
    IdentityPoolId: YOUR_IDENTITYPOOLID,
    Logins: {},
};
params1.Logins[`${tISS}`] = tAuthorizationToken;

let cognitoid = null;
try {
    cognitoid = await cognitoidentity.getId(params1).promise();
} catch (err) {
    console.log('Error in cognitoidentity.getId()');
    console.log(err, err.stack); // an error occurred
}

// 2. Get Credentials for the Cognito Id obtained from the Identity Pool
let params2 = {
    IdentityId: cognitoid.IdentityId,
    Logins: {},
};
params2.Logins[`${tISS}`] = tAuthorizationToken;

let creds = null;
try {
    creds = await cognitoidentity
    .getCredentialsForIdentity(params2)
    .promise();
} catch (err) {
    console.log('Error in cognitoidentity.getgetCredentialsForIdentity');
    console.log(err, err.stack); // an error occurred
}

// 3. Use the credentials we exchanged for the JWT with the Identity Pool
    let s3 = null;
    try {
      console.log('Instantiating S3');
      s3 = new AWS.S3({
        accessKeyId: creds.Credentials.AccessKeyId,
        secretAccessKey: creds.Credentials.SecretKey,
        sessionToken: creds.Credentials.SessionToken,
      });
      let tBucketName = 'your-private-bucket-name';
      let tBucketRegion = 'your-aws-region';
```

## Get PreSigned URL Using the S3 instance accessed with the Identity Pool Creds 
```
let tHomeFolder = pIdToken.payload["sub"];
let tFileName = 'image.jpg';
let tKey = `private/${tHomeFolder}/${tFileName}`;
let presignedGETURL = s3.getSignedUrl('getObject', {
Bucket: tBucketName,
Key: tKey, // filename
Expires: 240, // time to expire in seconds - 4 mins
});


// Replace Bucket name with CDN domain
console.log(`presigned s3 url ${presignedGETURL.replace('your-bucket-name.your-aws-region.amazonaws.com','your-cdn-domain')}`);

// Presigned url will look like
/*
https://cdn.example.com/private/image.jpg?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=ABCD1234%2Faws-region%2Fs3%2Faws4_request&X-Amz-Date=20221004T122658Z&X-Amz-Expires=180&X-Amz-Signature=12abcd456efghi-Amz-SignedHeaders=host
*/
```

## Tests
1. Can a user access an image in his/her home folder?
- Choose a user say user1 and from cognito pool read the sub, push an image in the users home folder
- The sub should be picked from the cognito pool
- Change the file name in the code to the file name you just pushed to the home folder of user1
- Login as user1 
- Call your function to get a presigned url from S3
- If you paste this url in the browser you should see it YAY!

2. Choose a user say user2 access an image in user1's home folder? 
- Change the code and hardcode the sub this time to user1's sub 
- Change the file name to the file in user1s home folder
- Ensure we request a presigned url with a hardcoded path containing user1's home folder and the image file in home folder of user1
- We get a presigned url on our console
- Typing the presigned url leads to an Access Denied message, so Yay! only user1 can access his/her images

Note 1 - If user1 copies the image link and shares it with anyone - they can see the image as user1 is sharing the path signed with his/her creds

Note 2 - If we want to restrict image access to a particular domain on CloudFront 
- We could configure rules in WAF for CloudFront
- Cloudfront lambda edge triggers may work too



 











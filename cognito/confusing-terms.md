# Commonly used terms & acronyms
- APIG : Api Gateway 
- Authentication : Verified the identity of the user eg are you a valid used with a proper email and password registered with the system?
- Authorization : determines their access rights eg Does this use have the privileges/rights to delete this file?
- CF : CloudFront
- Federated users : An authorised user with a single token to obtain access to other systems in a network
- Federated Identities - Identities in your AWS system having temporary access to AWS Resources like S3
- IAM : Identity and Access Management
- Identity pools : Provide temporary creds to guests or authenticated users to access AWS resources like S3, under the Cognito Service
- Policy Variables : Default or Custom mappings in the Identity pool in AWS Cognito mapping the User Pool attributes to variables that are available for use in an IAM Policy. Without these the user pool attributes are not available for any Conditional Statements in the IAM Policy
- sub : short for subject is a UUID for each Cognito User in the Cognito user Pool 
- User pool : Is a user directory provided by AWS to manage user profiles under the Cognito Service
## Sources
- [BeaBetterDev](https://beabetterdev.com/2022/06/26/amazon-cognito-a-complete-beginner-guide/)

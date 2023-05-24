My Notes for the awesome but horribly documented AWS!

Quick links to published notes - 
## AWS Cognito
- [AWS Cognito - confusing terms](https://github.com/amythical/aws/blob/main/TODO.txt)
- [Give a s3 bucket access only to users signed in through our Cognito User pool](https://github.com/amythical/aws/blob/9f0af2f3a3b3006b18249c48e3b5bd3d1ad5dd65/cognito/restrict-access-s3bucket-cognito-users-only-via-lambda.md)

## API Gateway
- [Lambda Proxy Vs Lambda Integration](https://github.com/amythical/aws/blob/9f0af2f3a3b3006b18249c48e3b5bd3d1ad5dd65/apigateway/lambda-proxy-Vs-lambda-integration.md)
- [Secure a API Gateway URL so that only users logged in using Cognito User Pool can access it. Accessing the API Gateway URL without Cognito creds should give an error] (https://github.com/amythical/aws/blob/9f0af2f3a3b3006b18249c48e3b5bd3d1ad5dd65/apigateway/secure-apigateway-with-cognito-userpool.md)

## S3
- [You have a private S3 bucket with images. You want to share the images with a few users or want someone view the images with a link but you do not want to expose the bucket publicly over CDN](https://github.com/amythical/aws/blob/0358331d94c9591b9938b3e7917eddffe0d54f86/s3/pre-signed-url-private-S3-bucket.md)
- [You want to restrict each user to access a specific folder which belongs to them kinda like their home directory, in a single S3 bucket. User A should not be able to access user Bs folder in the same S3 bucket](https://github.com/amythical/aws/blob/0358331d94c9591b9938b3e7917eddffe0d54f86/s3/restrict-users-to-their-home-folders-S3-bucket.md)
- [We have presigned urls from S3. Every request to generate a presigned url creates a url with new query string params, so the browser does not cache it. A solution to fix caching for presigned urls from S3](https://github.com/amythical/aws/blob/0358331d94c9591b9938b3e7917eddffe0d54f86/s3/caching-presigned-urls.md)

## CloudFront
- [Users can only access images from a S3 bucket using a cloudfront url. Any attempt to directly access the S3 object using a S3 url should not be allowed](https://github.com/amythical/aws/blob/0358331d94c9591b9938b3e7917eddffe0d54f86/cloudfront/cloudfront-serve-content-from-private-s3-bucket.md)
- [Protected content require the user to be logged into to the app to view it. Configure access to assets requiring a App login and free content using CloudFront](https://github.com/amythical/aws/blob/0358331d94c9591b9938b3e7917eddffe0d54f86/cloudfront/cloudfront-serve-private-content-only-to-logged-in-users-s3.md)

## Lambda
- [Lambda layers and installing Sharp - the image processing package in a Lambda layer](https://github.com/amythical/aws/blob/0358331d94c9591b9938b3e7917eddffe0d54f86/image-processing/lambda-sharp/installing-sharp.md)
- [Read an image from a S3 Bucket, resize it and then save it to another S3 bucket](https://github.com/amythical/aws/blob/0358331d94c9591b9938b3e7917eddffe0d54f86/image-processing/lambda-sharp/crop-images-sharp-s3.md)


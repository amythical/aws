# Use Case
You have a private S3 bucket with images. You want to share the images with a few users or want someone view the images with a link but you do not want to expose the bucket publicly over CDN.
# Approach
With the access credentials for private S3 bucket, we request a presigned url with an expiry time, the url with myriad parameters gives us access to the image for the specified expiry time 

# Implementation
 ## Pre-requisites
- Create a s3 Bucket 
- Add S3 Bucket as an origin and configure add a behaviour to your cloudfront CDN with a path pattern mapping to our private S3 bucket 
- [Here is a post on creating behaviours in CloudFront based on patterns - See CloudFront Add New Origin
 & Configure CloudFront Paths](https://github.com/amythical/aws/blob/main/cloudfront/cloudfront-serve-private-content-only-to-logged-in-users-s3.md)
- Add the S3 Bucket as a new origin
    - Note - Ensure the cloudfront origin name contains the region your-private-bucket.s3-eu-west-1.amazonaws.com (s3-eu-west-1 replaced with your region), or manually add this as the origin name
    - I did not enocunter this issue but a [post on advanced web](https://advancedweb.hu/how-to-use-s3-signed-urls-with-cloudfront/) talks about this issue
    - We need this setting for later but good to check the origin name in CloudFront when we set it up

- The pattern we select for our private content is /private/* 

## The Code
```
const tAccessKey = 'ABCDEFG'; // access key with S3 perms
  const tSecret = '123abcdefg';// secret for above access key

  let credentials = {
    accessKeyId: tAccessKey,
    secretAccessKey: tSecret,
  };
  AWS.config.update({ credentials, region: 'your-region' }); // your region looks like eu-west-1

  let s3 = new AWS.S3();

  let presignedGETURL = s3.getSignedUrl('getObject', {
    Bucket: 'your-privatebucket',
    Key: 'private/image.jpg', // filename
    Expires: 300, // time to expire in seconds - 5 mins for now
  });

  console.log(`S3 PRESIGNED GET URL ${presignedGETURL}`);
```
- [ Original Source code from Aidan.Hallet](https://medium.com/@aidan.hallett/securing-aws-s3-uploads-using-presigned-urls-aa821c13ae8d), with modifications

## Try #1
- So copy/pasting the S3 presigned url in the browser gave us the image so yay!
- We can see this image for the next 5 mins, refreshing after which would give us an error
- So a S3 presigned url looks like 
```
https://your-private-bucket-name.region.amazonaws.com/private/image.jpg?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=ABCD123456&X-Amz-Date=20221004T124953Z&X-Amz-Expires=123&X-Amz-Signature=1234abcdefgh&X-Amz-SignedHeaders=host

// private is a folder we have created in the private S3 bucket to match the behaviour pattern in CloudFront
```
- But its _**EXPOSES MY S3 BUCKET PATH**_ ... Im not  happy with that !!!!!

# Should I use CloudFront signed url instead?
- This led me to think again about CloudFront signed Urls, why not use them instead of S3 presigned urls?
- Found a good article on the same topic on [Advanced Web](https://advancedweb.hu/the-differences-between-s3-and-cloudfront-signed-urls/)
- Summarising the article
    - S3 uses IAM for the presigned url, so its much more granular, you can create roles and allow access only for a few roles
    - CloudFront uses Signers so there is not so much of a granular control and no relationship with IAM
    - Depends on your use case ... in my usecase i needed granular access to the resources based on IAM Policies 

# Can we hide the bucket name and replace it with a CloudFront Url?
- TLDR Yes we can, read on ...
- I read another article on [Advanced Web](https://advancedweb.hu/how-to-use-s3-signed-urls-with-cloudfront/), but it did gave me Access Denied issues, but it was good enough to point me in the right direction and look for a solution
- So we need to use the CDN and using the behaviours path pattern match it to the S3 origin with the required query string parameters 
- How do we forward the query string from CloudFront to S3?
    CloudFront -> Policy->Custom Policies -> Create cahe policy -> New Cache control policy 
    - Give it a Name and Description
    - Headers None, Query strings All
    - Cookies None
    - Save
- We need to use this cache policy in our distribution
    - CloudFront -> Behaviours -> private pattern
    - Select the cache policy and in the select options find the Csutom Cache Control Policy we just created
    - Save changes

# Replacing the bucket name in the presigned url
```
https://your-private-bucket-name.region.amazonaws.com/private/image.jpg?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=ABCD123456&X-Amz-Date=20221004T124953Z&X-Amz-Expires=123&X-Amz-Signature=1234abcdefgh&X-Amz-SignedHeaders=host  

    // Replace your-private-bucket-name.region.amazonaws.com with your cdn url
    
     https://cdn.mydomain.com/private/image.jpg?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=ABCD123456&X-Amz-Date=20221004T124953Z&X-Amz-Expires=123&X-Amz-Signature=1234abcdefgh&X-Amz-SignedHeaders=host  
``` 
# Try #2 - Lets try the presigned url again
- Instead of Access Denied I now got a new error, something on the lines of cannot use multiple auth methods error where the Authorization Header was also being passed (Authorization Header is the JWT Token from User pool)
- Tip - ensure your presigned url has not timed out or your image isnt cached due to some weaker settings like a public s3 bucket or public access to CDN, once cached on the CDN you will get the image no matter what, I used different images when trying different steps/changes to be sure nothing was cached 
- So let us stop sending the Auth Header, this can be found in Origins Access Control
    - CloudFront -> your distribution -> Origins tab -> Select the S3 Bucket added as an Origin and select Access Control -> Create Control Setting
    - Give it a name, description, Signing Bheaviour select Do not sign requests
    - Save changes

# Try #3
- CloudFront -> Distributions -> Your Distribution -> disable and enable just to be safe
- Now trying the S3 presigned url by replacing the buckets domain with the CDNs domain we see the image! YAY!!!

# What about Caching with the custom policy?
- As long as the query string parameters remain the same the CDN cache will be hit
- We have a problem as with a new expiry time the signature changes each time a S3 Presigned Url is generated
- We will address this in a separate post [Link To be added]()

# Conclusion
 - We managed to hide the S3 Bucket name, use the CDN domain and get a presigned url for a private S3 Bucket image
 - Cache optimisation yet to be addressed


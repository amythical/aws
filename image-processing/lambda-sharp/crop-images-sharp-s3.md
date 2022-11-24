# We read an image from a S3 Bucket, crop it and then save it to another S3 bucket

## Pre-requisites
* Assuming Lambda runtime is 18.x
* Lambda has permission to access S3
* You have 2 S3 buckets say mysourcebucket, mytargetbucket
* mysourcebucket has an image say car.jpg

## The imports
We use node 18.x so the lambda runtime provides a ES6 file called index.mjs, require gets replaced with import.
This is how the imports look :-
```
import { S3Client, GetObjectCommand, PutObjectCommand } from "@aws-sdk/client-s3";
import { Upload } from "@aws-sdk/lib-storage";
import stream from "stream";
import jwt from 'jsonwebtoken'; // Included in the Layer
import sharp from '/opt/nodejs/node_modules/sharp/lib/index.js';// Included in the Lambda layer
```
The older lambda sdks get replaced with the v3 sdks, so there is a some new learning looking into the docs of the v3 sdk apis.

## The handler function
```
export const handler = async(event) => {

}
```
This is the new ES6 style handler

## Create the S3 Client
Assuming you have access to S3 via your Lambda role (simplest for a demo, dont use this for production), create the S3 Client as
```
var s3 = new S3Client();
```

## Reading an image from a s3 bucket
With v3 streams are available for S3 reading and writing.
```
// Read a stream from S3 into memory
async function readStreamFromS3(Bucket, Key) {
    try{
  return await s3.send(
    new GetObjectCommand({
      Bucket,
      Key,
    })
  );
    }catch(ex){
        console.log("ERROR readStreamFromS3 "+ex);
        throw ex;
    }
}

let readStream = null;
try{
   readStream = await readStreamFromS3('mysourcebucket', 'car.jpg');
}catch(ex){
   console.log("Error reading stream "+ex);
}
```

## Resizing an image using sharp
```
  let convertStream = null;
  let tCropParams = {'height':250};// resize height, width adjusted as per aspect ratio
  convertStream = sharp().resize(tCropParams).jpeg({ mozjpeg: true }); // mozjpeg gives great jpeg compression
 // for pngs
 // convertStream = sharp().resize(tCropParams).png({ palette: true });
```

## Writing the transformed image to S3
```
// Create a passthrough stream.Transform object and pass it as an input param to S3
function writeStreamToS3(Bucket, Key) {
  const pass = new stream.PassThrough();

  return {
    writeStream: pass,
    upload: new Upload({
      client: s3,
      params: {
        Bucket,
        Key,
        Body: pass,
        ContentType: "image/*",
        queueSize: 4, // optional concurrency configuration
        partSize: 1024 * 1024 * 5, // optional size of each part, in bytes, at least 5MB
        leavePartsOnError: false, // optional manually handle dropped parts
      },
    }),
  };
}

let writeStream = null;
  try{
     const {writeStream,upload} = writeStreamToS3(
     'mytargetbucket',
     'car_resized.jpg',
     );
  }catch(ex){
	console.log("Error writing stream "+ex);
  }
```

## Piping ReadS3-SharpResize-WriteS3
With the streams, piping can be done which is similar to pipes in unix redirecting/chaining input and output
```
try{
 readStream.Body.pipe(convertStream).pipe(writeStream);
 await upload.done();
catch(ex){
   console.log("Error piping streams "+ex);
}
```

## Returning response from Lambda
The final step so that the lambda returns, send a response
```
 const response = {
            statusCode: 200,
            body: JSON.stringify({msg: "resize success"}),
        };
        return response;
```

## Entire code snippet - Altogether
```
import { S3Client, GetObjectCommand, PutObjectCommand } from "@aws-sdk/client-s3";
import { Upload } from "@aws-sdk/lib-storage";
import stream from "stream";
import jwt from 'jsonwebtoken'; // Included in the Layer
import sharp from '/opt/nodejs/node_modules/sharp/lib/index.js';// Included in the Lambda layer

var s3 = new S3Client();

async function readStreamFromS3(Bucket, Key) {
    try{
  return await s3.send(
    new GetObjectCommand({
      Bucket,
      Key,
    })
  );
    }catch(ex){
        console.log("ERROR readStreamFromS3 "+ex);
        throw ex;
    }
}

function writeStreamToS3(Bucket, Key) {
    const pass = new stream.PassThrough();
  
    return {
      writeStream: pass,
      upload: new Upload({
        client: s3,
        params: {
          Bucket,
          Key,
          Body: pass,
          ContentType: "image/*",
          queueSize: 4, // optional concurrency configuration
          partSize: 1024 * 1024 * 5, // optional size of each part, in bytes, at least 5MB
          leavePartsOnError: false, // optional manually handle dropped parts
        },
      }),
    };
  }

export const handler = async(event) => {
    let readStream = null;
    try{
       readStream = await readStreamFromS3('mysourcebucket', 'car.jpg');
   

    let convertStream = null;
    let tCropParams = {'height':250};// resize height, width adjusted as per aspect ratio
    convertStream = sharp().resize(tCropParams).jpeg({ mozjpeg: true }); // mozjpeg gives great jpeg compression
   
    let writeStream = null;
     const {writeStream,upload} = writeStreamToS3(
     'mytargetbucket',
     'car_resized.jpg',
     );
    

    readStream.Body.pipe(convertStream).pipe(writeStream);
    await upload.done();
   
    const response = {
        statusCode: 200,
        body: JSON.stringify({msg: "resize success"}),
    };
    return response;
    }catch(ex){
        console.log("Error "+ex);
        const response = {
            statusCode: 500,
            body: JSON.stringify({msg: "error "+ex}),
        };
        return response;
    }
}
```

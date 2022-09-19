
# API Gateway - Lambda Proxy Vs Lambda Integration



| Option | Description |
| ----------- | ----------- |
| Lambda Proxy | This is a simple, but powerful integration. All the request to the APIGateway URL is forwarded straight to the Lambda and the response is sent from Lambda. i.e No modifications to the request(query params, body, variables) and response(status code, message) are done by the APIGateway.  |
| Lambda Integration | This is complex, but offers more control over transmissiondata. The request can be modified before it is sent to lambda and the response can be modified after it is sent from lambda. This can be done by mapping templates which transforms the payload, as per the user customisations. API Gateway uses Velocity Template Language (VTL) engine to process body mapping templates for the integration request and integration response.      |

[Source - https://medium.com/@lakshmanLD/lambda-proxy-vs-lambda-integration-in-aws-api-gateway-3a9397af0e6d](https://medium.com/@lakshmanLD/lambda-proxy-vs-lambda-integration-in-aws-api-gateway-3a9397af0e6d)
<br/><br/>
## Conclusion
Use Lambda proxy for most common cases.

<br/><br/>
## Lambda proxy - Data return format
--
In Lambda proxy integration, API Gateway requires the backend Lambda function to return output according to the following JSON format:


```
{
    "isBase64Encoded": true|false,
    "statusCode": httpStatusCode,
    "headers": { "headerName": "headerValue", ... },
    "multiValueHeaders": { "headerName": ["headerValue", "headerValue2", ...], ... },
    "body": "..."
}
```
[Source](https://docs.aws.amazon.com/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html#api-gateway-simple-proxy-for-lambda-output-format)
# api-postbox

AWS CloudFormation script that creates an internet facing API using AWS API Gateway and AWS Lambda, that allows 
data of up to 10MB (API Gateway's limit) to be queued for async onwards processing.

The script demonstrates how to use Lambda for token based authorisation, with subsequent storage of the data
into S3 and the queuing of a message allowing the data to be retrieved by downstream systems.

Using default parameter settings, the correct POST request is of the form:

`curl -H "AuthDetails: allow" -H "Content-Type: application/json" -d {} https://<REST API>.execute-api.eu-west-2.amazonaws.com/beta/publish`

where the data `{}` could be up to 10MB.

An example use case is where data needs to be captured in a "fire and forget" mode - ensuring data is captured
accurately, with downstream processing occuring asynchronously to the original request.  In this scenario, the
cloudformation script can capture the data regionally, with further processes then retrieving the messages and
processing the data asynchronously.  This allows centralised processing in a single region, with minimal impact
to the callers of the API should that region ever go offline. 

The script creates the following:

![alt text](https://github.com/gford1000-aws/api-postbox/blob/master/Data%20Postbox.png "Script per designer")

## Arguments

| Argument                     | Description                                                                 |
| ---------------------------- |:---------------------------------------------------------------------------:|
| APIName                      | Name of the REST API                                                        |
| APIDescription               | Description of the REST API                                                 |
| StageName                    | Name of the stage to be created                                             |
| AuthorisationTokenHeaderName | Name of the header containing the auth token (leave blank for no auth)      |
| AuthorisationTimeout         | TTL of cache for authorisations (set to 0 for no caching)                   |
| LoggingTTL                   | TTL for logs of each Lambda function, saved to specific log groups          |
| MaxProcessingAttempts        | Number of message processing attempts prior to moving to redrive            |


## Outputs

| Output                  | Description                                                    |
| ----------------------- |:--------------------------------------------------------------:|
| URL                     | The URL of the deployed REST API                               |
| Queue                   | The URL of the message queue for onwards processing            |
| ErrorQueue              | The URL of the redrive queue for failed messages               |
| BucketName              | The name of the S3 bucket in which data is stored              |


## Notes

* AWS X-Ray is enabled on the API Gateway and Lambda functions
* The SQS message queue has a DelaySeconds = 3, to allow for S3 data propogation
* The SQS message queue is set for maximum long polling, with ReceiveMessageWaitTimeSeconds = 20
* The SQS message queue will retain messages for 14 days, to maximise resilience if downstream experiences outages
* Data in the S3 bucket is encrypted at rest using server-side encryption 
* Data in the S3 bucket expires after 14 days, to be consistent with SQS message retention


## Licence

This project is released under the MIT license. See [LICENSE](LICENSE) for details.

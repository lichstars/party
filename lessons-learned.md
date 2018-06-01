## Lesson's learnt
1. Maintain a really good `Readme.md` document
2. Combine services in the same codebase to speed delivery but beware of pitfalls.

    _Consider the following example:_

    A JAVA application is created and made responsible for two purposes
    - Batch processing, and
    - API

    The batch processing was coded first and it set up a lot of the infrastructure (AWS database/sqs/s3 operations) needed. The API service was then added to the same repo so it could easily access the infrastructure that was already set up. This gave us a working application much quicker, however, a later code change to the batch processing highlighted a problem caused by combining these two services.

      The batch processor runs when there are things to process. It can be taken offline for a deployment and brought back up and resumed when needed with no impact to the public. However, in comparison the API needs to be up 100% as it can be accessed by the public at any time. So when we needed to deploy new changes to the batch processor, we couldn't stop the service as the API had to be up 100%. This dependency caused us to organise a deployment with a result of maintaining a service uptime of 100%.

      Going forward, a better approach would be to decouple these two services as they have different uptime requirements, and by doing this will make our deployment process simpler.

3. Where there is a 3rd party system dependency, it is best to architect the system so that you are not reliant on the 3rd party system to be up and online when you need it. For example, if there is a enquiry form that is to be submitted to a 3rd party for processing, a good approach to this would be to save this data into a S3 bucket and have a SQS queue in charge of sending this data to the 3rd party system. If the 3rd party system is down, the SQS queue can be set up so that it can keep retrying without any manual dev actions. To the client user, there is no service impact.

4. Code duplication is good for the brain.

---
layout: post
title: The Mystery of Amazon's API Gateway Timeout?
date: 2018-06-07 09:24:00 +0500
description: The Mystery of Amazon's API Gateway Timeout
img: # Add image post (optional)
tags: [aws, amazon, lambda, api-gateway, http, serverless] # add tag
---

Did you know that Amazon's API Gateway Service has a **upper limit of 30 seconds** that cannot be bypassed or increased in any way? I sure didn't! And the way that I and my team found out about it deserves a blog post! **(hint: There was a bug in the code!)** Lets provide a bit of a background first before going into the actual details.

<br />

So, my team had a AWS lambda function over which we had configured a API Gateway. All was good and the respose time was under 5 seconds. And then suddenly, we started getting ```Endpoint Request Timed out``` in response to the API invocation. At first we thought that maybe the Lambda function was timing out so we increased its limit all the way to 300 seconds. Even then, the request was timing out at 29 seconds. We found this a bit odd because the two time limits were vastly different.

<br />

After a bit of head scratching and of course, some Googling, we found out that the API Gateway itself has a timeout limit of 30 seconds and if the lambda does not return within that time, the API Gateway will time out. It does not matter if the Lambda function has a timeout limit of 5 minutes. This is a bit intriguing because this limit on the API Gateway essentially renders the 5 minute timeout limit of the Lambda Function invalid. Interestingly, this limit is mentioned on the [API Gateway Developer guide](https://docs.aws.amazon.com/apigateway/latest/developerguide/limits.html) as well. And it was a mistake on our part not to go through the limits of a service first before consuming it.

<br />

Anyways, this got us thinking as to what was causing the Lambda function to take so much time because there really wasn't alot of processing involved. And this finally led my team to the bug that was causing this entire issue! **(I will, for obvious reasons, not go into the details of the bug).**

<br />

My team was able to resolve the API Gateway timeout issue without much hassle **(we fixed the bug!)**. However, if your Lambda function really is taking more than 30 seconds to compute, Here's what you can do:


#### 1. The first option is that you can try and making your function as efficient as possible by running as many things asynchronously as you can. If there is something that can be executed in the parallel, extract it, create a new Lambda function and invoke that function directly from the first lambda function. AWS already has a blog post [here](https://aws.amazon.com/blogs/developer/invoking-aws-lambda-functions-from-java/) that lets us know how it can be done in Java.

#### 2. The next option is particularly nifty that i found [here](https://sielert.com/api-gateway-gotcha-with-aws-lambda/) while researching on this issue. Quite simply, if the API Gateway times out, it does not mean that the Lambda function times out as well. The lambda function will complete its execution - it just wont be able to send a response. So, you can setup a callback URL at the end of the function that will post your data. This can easily be done by creating a new API Gatway that will send the data to your database or wherever it is that you want the data at.

#### 3. The final option is to adopt a pattern for executing long requests in AWS mentioned [here](http://www.99serverless.com/index.php/2017/11/24/serverless-long-running-http-requests/). Not only is the pattern fairly simple to understand, but it also very easy to implement as well. In summary of the pattern, the request needs to be broken down into three parts: *a. initial request/respons; b. background processing; c. polling*

Do let me know if there are other simple ways to bypass the API Gateway's time limit that i have missed. I hope this helps others! Cheers!

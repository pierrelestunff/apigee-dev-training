# Lab 2.02 - Basic Authentication

## Introduction

Objectives:

* Learn how to protect your API using basic authentication.

## Instructions

Take the solution from lab 2.01 as a starting point and follow the steps below:

1. Add a new BasicAuthentication policy with the following XML contents:

        <?xml version="1.0" encoding="UTF-8" standalone="yes"?>
        <BasicAuthentication async="false" continueOnError="false" enabled="true" name="BasicAuthentication">
            <Operation>Decode</Operation>
            <IgnoreUnresolvedVariables>false</IgnoreUnresolvedVariables>
            <User ref="clientId"/>
            <Password ref="clientSecret"/>
            <Source>request.header.Authorization</Source>
        </BasicAuthentication>

3. Add the BasicAuthentication policy as the first step of the &lt;Request&gt; element in the proxy endpoint PreFlow. Adding this policy there, we have made sure that if a request hits our API proxy with Basic Authentication we extract the username and password in the Authorization header into runtime variables called clientId and clientSecret.

4. Modify the existing VerifyAPIKey policy so the ref attribute in the &lt;APIKey&gt; element is now the clientId variable. Below how the XML of the policy would look like:

        <?xml version="1.0" encoding="UTF-8" standalone="yes"?>
        <VerifyAPIKey async="false" continueOnError="false" enabled="true" name="VerifyAPIKey">
            <APIKey ref="clientId"/>
        </VerifyAPIKey>

5. Add the VerifyAPIKey policy to the &lt;Request&gt; element in the proxy endpoint PreFlow, just after the BasicAuthentication policy. With this we have verified that the clientId variable value matches the consumer key of a developer app that has a product the grants access to this API proxy. But we have not only done that, the execution of that VerifyAPIKey policy has extracted the consumer key of that developer app in a variable called verifyapikey.VerifyAPIKey.client_secret.

6. We need whether the value of the clientSecret runtime variable matches the value of the consumer secret that was extracted by the VerifyAPIKey policy. If both values do not match, the request should not go through and an error message saying that the request is not authorized should be send back to the client. For that we will need to add the following to steps in the proxy endpoint PreFlow, just after the VerifyAPIKey policy:

        <Step>
            <Name>AssignMessage.Error.Unauthorized</Name>
            <Condition>verifyapikey.VerifyAPIKey.client_secret != clientSecret</Condition>
        </Step>
        <Step>
            <Name>RaiseFault.GoToFaultRules</Name>
            <Condition>flow.error.code != NULL</Condition>
        </Step>

7. Go to the Trace took and send a request to https://ORGANIZATION-ENVIRONMENT.apigee.net/book/v1/books without adding the Authorization header. Verify that the BasicAuthentication policy has failed and check what the raised fault is. Write it down.

6. In the Trace tool click on the link that sends **Send with API Console** on the right above the Transaction Map. Send a request to https://ORGANIZATION-ENVIRONMENT.apigee.net/book/v1/books with an invalid authorization header (eg: Authorization: aaaaa) and check which fault was raised by the BasicAuthentication policy.

7. Modify the existing "Unauthorized" FaultRule in the proxy endpoint to include the faults that you wrote down. It should look as follows:

    <FaultRule name="Unauthorized">
        <Step>
            <Name>AssignMessage.Error.Unauthorized</Name>
        </Step>
        <Condition>fault.name = "InvalidApiKey" OR fault.name = "FailedToResolveAPIKey" OR fault.name = "UnresolvedVariable" OR fault.name = "InvalidBasicAuthenticationSource"</Condition>
    </FaultRule>

8. Repeat step 6 and 7 to verify that know you get back a 401 HTTP response code back.

8. Using the API Console send a request using Basic Authentication (it can be set using the available option in the top bar) with an invalid password. Make sure you get a 401 HTTP response code back.

9. Repeat the previous step but now supplying a valid password. Make sure that you get a 200 HTTP response code back.






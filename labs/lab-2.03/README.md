# Lab 2.03 - Traffic Management

## Introduction

Objectives:

* Learn how to protect your API proxy backend from traffic spikes using the SpikeArrest policy.

* Learn how to restrict the amount of requests a certain client can make over a period of time using the Quota policy.

## Instructions

We will take the solution available for lab 2.02 as starting point and follow the steps available below.

### SpikeArrest

1. Add a new SpikeArrest policy using with the following XML content:

        <?xml version="1.0" encoding="UTF-8" standalone="yes"?>
        <SpikeArrest async="false" continueOnError="false" enabled="true" name="SpikeArrest">
            <Rate>5pm</Rate>
        </SpikeArrest>
    
2. Add the SpikeArrest policy as the first step of the &lt;Request&gt; element in the proxy endpoint PreFlow.

3. Go to the trace tool and start to send requests non-stop to your API proxy. Eventually you will see that the SpikeArrest policy raises a fault. Write down the name of the fault raised.

4. We have to add proper fault handling for the fault raised by the SpikeArrest policy. Create a new AssignMessage policy with the following XML contents:

        <?xml version="1.0" encoding="UTF-8" standalone="yes"?>
        <AssignMessage async="false" continueOnError="false" enabled="true" name="AssignMessage.Error.SpikeArrestViolation">
            <AssignVariable>
                <Name>flow.error.message</Name>
                <Value>Too many requests</Value>
            </AssignVariable>
            <AssignVariable>
                <Name>flow.error.code</Name>
                <Value>429.01.001</Value>
            </AssignVariable>
            <AssignVariable>
                <Name>flow.error.status</Name>
                <Value>429</Value>
            </AssignVariable>
            <AssignVariable>
                <Name>flow.error.info</Name>
                <Value>http://documentation</Value>
            </AssignVariable>
        </AssignMessage>

5. Add a new FaultRule to handle the fault raised by the SpikeArrest policy:

        <FaultRule name="Spike Arrest Violation">
            <Step>
                <Name>AssignMessage.Error.SpikeArrestViolation</Name>
            </Step>
            <Condition>fault.name = "SpikeArrestViolation"</Condition>
        </FaultRule>

6. Generally we will only want to use this protection against traffic spikes in production, so we will add a condition to the step with the SpikeArrest policy in the proxy endpoint PreFlow so it looks like this:

        <Step>
            <Name>SpikeArrest</Name>
            <Condition>environment.name = "prod"</Condition>
        </Step>

### Quota

1. Fist, go to ** Publish > API Products **, edit the product that you are using and set up the quota there. For demonstration purposes, so we can see the quota being violated, we will only allow 4 requests per minute. Make sure that the test environment is checked for that product.

2. Then add a new Quota policy with the following XML content:

        <?xml version="1.0" encoding="UTF-8" standalone="yes"?>
        <Quota async="false" continueOnError="false" enabled="true" name="Quota">
            <Allow count="2000" countRef="verifyapikey.VerifyAPIKey.apiproduct.developer.quota.interval"/>
            <Interval ref="verifyapikey.VerifyAPIKey.apiproduct.developer.quota.interval">1</Interval>
            <TimeUnit ref="verifyapikey.VerifyAPIKey.apiproduct.developer.quota.timeunit">month</TimeUnit>
            <Identifier ref="verifyapikey.VerifyAPIKey.client_id" />
            <Distributed>true</Distributed>
            <Synchronous>true</Synchronous>
        </Quota>

    When the VerifyAPIKey policy runs all the quota settings that we made in the product are extracted in runtime variables. We are using those variables in the policy.

3. Go to the trace tool, open the API Console and start to send requests (with basic authentication) non-stop to your API proxy. Eventually you will see that the Quota policy raises a fault. Write down the name of the fault raised.

4. Add the Quota policy as a new step in the &lt;Request&gt; element in the proxy end Preflow, just before the KeyValueMapOperations policy.

5. We have to add proper fault handling for the fault raised by the SpikeArrest policy. Create a new AssignMessage policy with the following XML contents:

        <?xml version="1.0" encoding="UTF-8" standalone="yes"?>
        <AssignMessage async="false" continueOnError="false" enabled="true" name="AssignMessage.Error.QuotaViolation">
            <AssignVariable>
                <Name>flow.error.message</Name>
                <Value>Too many requests</Value>
            </AssignVariable>
            <AssignVariable>
                <Name>flow.error.code</Name>
                <Value>429.01.002</Value>
            </AssignVariable>
            <AssignVariable>
                <Name>flow.error.status</Name>
                <Value>429</Value>
            </AssignVariable>
            <AssignVariable>
                <Name>flow.error.info</Name>
                <Value>http://documentation</Value>
            </AssignVariable>
        </AssignMessage>

5. Add a new FaultRule to handle the fault raised by the SpikeArrest policy:

        <FaultRule name="Quota Violation">
            <Step>
                <Name>AssignMessage.Error.QuotaViolation</Name>
            </Step>
            <Condition>fault.name = "QuotaViolation"</Condition>
        </FaultRule>
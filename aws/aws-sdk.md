## DynamoDB

### Get
```js
const documentClient = new DocumentClient();

const {Item} = await documentClient.get({
    TableName: 'TABLE_NAME',
    Key: {
        groupId: groupId,
        entityId: entityId
    }
}).promise();
```

### Query
```js
const {Items: Records} = await documentClient.query({
    TableName: 'TABLE_NAME',
    // FilterExpression: "#key1 = :key1 AND contains(#key2, :key2)", // Optional: filter
    KeyConditionExpression: "#key1 = :key1",
    ExpressionAttributeValues: {
        ":key1": key1,
        ":key2": key2
    },
    ExpressionAttributeNames: {
        "#key1": "VALUE1",
        "#key2": "TEXAS"
    }
}).promise();
```


### Query Iterative
Iterative when items are huge
```js
const params = {
    TableName: 'TABLE_NAME',
    KeyConditionExpression: '#thingName = :thingName',
    FilterExpression: 'code IN (:code1, :code2)',
    ExpressionAttributeValues: {
        ':thingName': thingName,
        ':code1': 'PET_ENTERED',
        ':code2': 'PET_EXITED',
    },
    ExpressionAttributeNames: {
        '#thingName': 'thingName',
    },
    ExclusiveStartKey: undefined,
    // Limit: PAGE_SIZE,
};

const Items = []; // Full Data
let res: any = { LastEvaluatedKey: undefined };
while ('LastEvaluatedKey' in res) {
    params.ExclusiveStartKey = res.LastEvaluatedKey;
    res = await documentClient.query(params).promise();
    if (res.Items.length) Items.push(res.Items);;
}
```

### Put
```js
const {Items: Records} = await documentClient.put({
    TableName: 'TABLE_NAME',
    Item: {...user, updatedAt: Date.now()}
}).promise();
```

### Delete
```js
const result = await documentClient.delete({
    Key: {
        invitee: 'name@gmail.com',
        ownerId: '1234'
    },
    TableName: 'TABLE_NAME',
}).promise();
```

### Update
```js
const params: DocumentClient.UpdateItemInput = {
    TableName: 'TABLE_NAME',
    Key: {
        thingName,
    },
    UpdateExpression: 'SET #deleted = :deleted, updatedAt = :updatedAt',
    ExpressionAttributeValues: {
        ':deleted': true,
        ':updatedAt': new Date().toISOString(),
    },
    ExpressionAttributeNames: {
        '#deleted': 'deleted',
    },
};
await documentClient.update(params).promise();
```

## SNS

```js
import * as SNS from 'aws-sdk/clients/sns';

const params = {
    TopicArn: 'arn:aws:sns:us-east-1:123456789012:my',
    Message: JSON.stringify({name: 'Pujan Srivastava'}),
    MessageAttributes: attributes,
};

const sns = credentials ? new SNS(credentials) : new SNS();
return sns.publish(params).promise();
```

## SQS

```js
import * as SQS from 'aws-sdk/clients/sqs';

const sqs = new SQS();
const params: SQS.SendMessageRequest = {
    MessageBody: JSON.stringify(message),
    MessageAttributes: attributes,
    QueueUrl: queueUrl,
    DelaySeconds: delaySeconds; // Optional
};
return sqs.sendMessage(params).promise();
```

## Lambda

```js
import * as Lambda from 'aws-sdk/clients/lambda';

const params = {
    FunctionName: 'lambda-fn-name', // Or full arn if cross account. If cross account, add resource-based policy in source lambda
    Payload: JSON.stringify({ query: { tz } }),
};
const { Payload } = await this.lambda.invoke(params).promise();
return PlatformUtils.safeParse(Payload);
```

## STS

```js
import * as STS from 'aws-sdk/clients/sts';

const sts = new STS();
const params = {
     RoleArn: roleArn,
     RoleSessionName: new Date().getTime().toString(),
};

const assumeRole = await sts.assumeRole(params).promise();

return {
    accessKeyId: assumeRole.Credentials.AccessKeyId,
    secretAccessKey: assumeRole.Credentials.SecretAccessKey,
    sessionToken: assumeRole.Credentials.SessionToken,
 };
```

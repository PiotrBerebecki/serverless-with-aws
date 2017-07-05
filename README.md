# Serverless with AWS

## Getting started

create `awsConfig.js` file in root
export const UserPoolId = 'eu-west-2...';
export const ClientId = '...';
export const urlPost =
  'https://AWS-APP-ID.execute-api.AWS-REGION.amazonaws.com/dev/compare-yourself';
export const urlGet =
  'https://AWS-APP-ID.execute-api.AWS-REGION.amazonaws.com/dev/compare-yourself/';
export const urlDelete =
  'https://AWS-APP-ID.execute-api.AWS-REGION.amazonaws.com/dev/compare-yourself/?accessToken=DUMMY';

## Covered
- Creating API endpoints with API Gateway
- Creating GET, POST, DELETE endpoints
- Managing CORS
- Passing API requests via Lambda
- Integrating into request and response
- User authentication with Cognito
- CDN with CloudFront

## Notes
- EC2 - Elastic Cloud Computing
- S3 - Simple Storage Service
- IAM - Identity and Access Management

```javascript
exports.header = (event, context, callback) => {
  callback(null, {msg: 'Hello from lambda'});
};
```

```javascript
exports.header = (event, context, callback) => {
  console.log(event)
  callback(null, {headers: {'Control-Access-Allow-Origin': '*'}});
};
```
- See console.log in CloudWatch

```javascript
exports.header = (event, context, callback) => {
  console.log('+++++++++');

  const age = event.personData.age;

  console.log('=========');
  callback(null, age * 2);
};
```

- Access request data in Lambda by adding `Body Mapping Templates` (Apache Velocity language and json path) in `Integration Request`

if request body is

```js
{
    "personData": {
      "name": "Max",
      "age": 28,
      "div": "#############"
    }
}
```

use the following body mapping template to access age directly in lamdba on the event object
```js
{
"age" : $input.json('$.personData.age')

// $input - gives access to request data (body, params etc.)
// json('$') - extracts complete request body
// the key could be renamed to anything and use
// accordingly in lambda
}
```

in lambda:

```js
exports.header = (event, context, callback) => {
  console.log('=========', event);
  callback(null, event.age * 2);
  // thanks to body mapping template we don't need to do
  // callback(null, event.personData.age * 2);
};
```

in the `Response Integration` you can map the result of the lambda (`56`) to something else. `$` refers to total lambda response body.

```js
{
"your-age": $input.json('$')
}
```

### Body Mapping Template in API Gateway

Body Mapping Templates can be confusing, but in the end, they're not that difficult.

You use the Apache Velocity Language when working with these templates, that will only matter once you create more complex transformations though. Basic explanations (see below) should be relatively self-explanatory

First, you need to understand that they simply allow you to control which data your action receives (e.g. Lambda).

You can for example transform incoming data of the following shape:

```js
{
  person: {
    name: "Max",
    age: 28
  },
  order: {
    id: "6asdf821ssa"
  }
}
```
to

```js

{
    personName: "Max",
    orderId: "6asdf821ssa"
}
```

with the following template:

```js
{
    "personName": "$input.json('$.person.name')",
    "orderId": "$input.json('$.order.id')"
}
```
$input  is a reserved variable, provided by AWS which gives you access to the input payload (request body, params, headers) of your request.

Link: http://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-mapping-template-reference.html#input-variable-reference

json()  is a method on $input  which will retrieve a JSON representation of the data you access. $  here stands for the request body as a whole, $.person.name  therefore accesses the name property on the person property which is part of the whole requests body object.

The $input.json(...)  code is wrapped into quotation marks (" ) because it should be transformed into a string in this example (since both name and id are strings). If you were to access a number, boolean or object here, you would NOT wrap the expression in quotation marks.

Example - Accessing a Number:

```js
{
    "extractedAge": $input.json('$.person.age')
}
```

You can learn more about the reserved variables provided by AWS and the different utility functions it offers here: http://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-mapping-template-reference.html

- Making a post request example:

```js
const xhr = new XMLHttpRequest();

xhr.open('POST', 'https://RESOURCE-ID.execute-api.eu-west-2.amazonaws.com/dev/RESOURCE-NAME');

xhr.onreadystatechange = function(event) {
  if (this.readyState == 4 && this.status == 200) {
    console.log(JSON.parse(xhr.responseText));
  }
}
xhr.setRequestHeader('Content-Type', 'application/json')
xhr.send(JSON.stringify({
  age: 28,
  height: 72,
  income: 500
}));
```

- API Gateway -> Custom Authorizers - Uses lambda to validate user based on request info (e.g. 'Identity Source Token', JWT token). The function needs to return IAM policy, which in turn will be used by API Gateway to allow / deny user's access to API. User Id also needs to be returned. Optionally return context.

---
date: "2018-09-30T13:31:15+02:00"
title: "Building a severless store with React, NodeJS, Stripe, AWS Lambda and S3"
draft: false
---

## Introduction

For quite some time I've been thinking about playing around with Lambda and the Stripe API to build a serverless store. I especially liked the PaymentRequest API as it really cuts down on implementation time. 
I was able to build a [MVP](https://twitter.com/der_simon/status/962739179213000705) on a lazy Sunday, but just using the PaymentRequest API loses a lot of customer, so we'll need to display a credit card form as a fallback as well.

Please check out the code for this project on [GitHub](https://github.com/c0dr/simpleStore).

## Result
On click on the "Buy now" button, we either show the

Nice PaymentRequest API dialog (as seen in Chrome 69)
![PaymentRquest](/static/img/PaymentRequest.png)

or Google Pay as seen here on Android Chrome 69

<img src="/static/img/GooglePay.png" alt="Google Pay" width="250"/>

or Apple Pay as seen here on iOS 

<img src="/static/img/ApplePay.png" alt="Apple Pay" width="250"/>

or the good ol' credit card form (as seen in IE 11)
![CreditCard](/static/img/CreditCard.png)


## Frontend Implementation

Sadly you can't only check for `window.PaymentRequest` to verify the support for the PaymentRequest API. The browser might have support for it, but the user is unable to pay as he might've not set up his cards.

So we'll need to create a PaymentRequest, call `.canMakePayment()` on the created Request and decide depending on the result.

As I wanted a consistent UI not matter the support, I used a regular button with a `onClick handler`

```jsx
<Button onClick={this.showPayment} color="primary">Buy now for ${this.props.price}</Button>
```

Depending on the support, we either show the PaymentRequest or the credit card form.

```js
  showPayment() {
    if(this.state.canMakePayment) {
      this.state.paymentRequest.show();
    } else {
      this.setState({showCreditCardForm: true})
    }

```

The other code for the PaymentRequest is pretty straightforward as the Stripe API removes a bit of complexity.
Once the user confirms the payment, we get a card token and the customers' data in the `token` event.

```js
const paymentRequest = props.stripe.paymentRequest({
      country: 'US',
      currency: 'usd',
      total: {
        label: this.props.productName,
        amount: this.props.price,
      },    
    });

    paymentRequest.on('token', async ({complete, token, ...data}) => {
      let success = await this.props.createOrder(customer, token);
      complete(success ? 'success': 'fail');
    });

    paymentRequest.canMakePayment().then((result) => {
      this.setState({canMakePayment: !!result});
    });

```

### Regular Checkout Form

To make it as easy as possible for the user to checkout even when entering his cards manually, I recommend adding the [appropriate](https://wiki.whatwg.org/wiki/Autocomplete_Types) autoComplete fields. 

```jsx
<Input type="text" name="address_line1" id="address" autoComplete="address-line1" value={this.state.address_line1} onChange={this.handleInputChange}/>
<Input type="text" name="postalcode" id="postalcode" autoComplete="postal-code" value={this.state.postalcode} onChange={this.handleInputChange}/>
```

The Stripe integration in React is pretty painless with their HOC. As Stripe inejects an iFrame for users to enter their credit card information, we need no fields for the credit card ourselves. 
```jsx
<CardElement className="form-control" hidePostalCode={true} />
[...]
export default injectStripe(CardSection);
```


## Backend Implementation

The Stripe NodeJS library is quite beautiful and supports await/asnyc which makes it really simple to write compact code.
```js
app.get('/product/:sku', async (req, res) => {
  try {
    let sku = await stripe.skus.retrieve(req.params.sku);
    let product = await stripe.products.retrieve(sku.product);
    return res.json({ sku: sku, product: product });
  } catch (err) {
    return res.status(500).send('an error occurred');
  }
});
```
Though definitely missing input validation and error handling, it was good enough for a MVP.
To use the Stripe Order API, we need to set up two objects:

* Product: One product (e.g. T-Shirt)
* SKU: One Stock Keeping Unit for the product (e.g. Size S)

As the SKU object doesn't include details of the product, we'll need to fetch the product as well. Just knowing the SKU, the frontend can then display all of the required product information.

To actually earn some money, we then create a order endpoint. All we need to do is to create an order with the SKU and then pay the order using the token. There is no difference between using a token from the manual credit card form and the PaymentRequest API.
```js
app.post('/order', async function (req, res) {
  let product = req.body.sku;
  let customer = req.body.customer;
  let token = req.body.token;

  [...]

  let order = await stripe.orders.create({
    items: [{type: 'sku', parent: product}]

    [...]
  });
  await stripe.orders.pay(order.id, { source: token.id });
  return res.status(200).send('successful');
});
```

## Deploying on AWS Lambda

As I was using the Express framework, I simply used the `aws-serverless-express` package to make my app ready for Lambda deployment.
You simply export your app and create a file for lambda.

```js
const awsServerlessExpress = require('aws-serverless-express')
const app = require('./app')
const server = awsServerlessExpress.createServer(app)

exports.handler = (event, context) => awsServerlessExpress.proxy(server, event, context);
```
Et voilà. Create a new Lambda function, upload your code and specify the handler (filename + handler name). In my case I had the Lambda code in `lambda.js`, so the handler was called `lambda.handler`;
![Lambd](/static/img/Lambda_Uploaded.png)

To invoke the handler, we use the Amazon API gateway. Simply create a new API and specify the Lambda function as the handler.
![Lambd](/static/img/API_lambda.png)

After setting up the resources 
![Lambd](/static/img/API_resources.png)

we can deploy the API to get a invocation URL
![Lambd](/static/img/Invoke_URL.png)

We can specify this URL in our frontend and build the app for production (`npm run build`).

## Deploying on S3

Simply create a new bucket and upload the files. We then enable the static hosting feature of S3.
![Static hosting](/static/img/s3_static_site.png)

To allow access to the files, we need to create new policy
```json
{
    "Version": "2008-10-17",
    "Id": "PolicyForPublicWebsiteContent",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": {
                "AWS": "*"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME/*"
        }
    ]
```
and edit the Access Control List to allow access for public users.
Test the configuration by opening the S3 static site url.

## Enabling CloudFront

To get a proper domain name and TLS, we need to setup CloudFront. Simply create a new distribution, but specify the S3 static hosting URL as the origin domain name instead of specifying the bucket directly.
![CloudFront](/static/img/cloudfront.png)


As we are using client-side routing, we need to serve the index.html for all paths that are not files.
The fast solution is to serve the index.html as the 404 page.
![CloudFront](/static/img/cloudfront_404.png)

After waiting a few moments, we can use the CloudFront domain name to access our application. And there you have it, a small React/NodeJS store selling your product using Stripe. No server configuration needed, as everything is handled by AWS. Is it practical for production? Possibly, the cold-boot time for Lambda seemed fast enough in my tests.

Please check out the code for this project on [GitHub](https://github.com/c0dr/simpleStore).

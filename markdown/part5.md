# Part 1: Paypal Buttons
## Step 1: Add Buttons
Let's start by adding some paypal buttons on our checkout page.We will want to replace the "Make payment" button with our new paypal buttons.

Source: https://developer.paypal.com/demo/checkout/#/pattern/client
```js
<div id="paypal-button-container"></div>
<script src="https://www.paypal.com/sdk/js?client-id=test&currency=USD"></script>

<script>
    // Render the PayPal button into #paypal-button-container
    paypal.Buttons({

        style: {

        },

        // Set up the transaction
        createOrder: function(data, actions) {
            return actions.order.create({
                purchase_units: [{
                    amount: {
                        value: '88.44'
                    }
                }]
            });
        },

        // Finalize the transaction
        onApprove: function(data, actions) {
            return actions.order.capture().then(function(orderData) {
                // Successful capture! For demo purposes:
                console.log('Capture result', orderData, JSON.stringify(orderData, null, 2));
                var transaction = orderData.purchase_units[0].payments.captures[0];
                alert('Transaction '+ transaction.status + ': ' + transaction.id + '\n\nSee console for all available details');

                // Replace the above to show a success message within this page, e.g.
                // const element = document.getElementById('paypal-button-container');
                // element.innerHTML = '';
                // element.innerHTML = '<h3>Thank you for your payment!</h3>';
                // Or go to another URL:  actions.redirect('thank_you.html');
            });
        }


    }).render('#paypal-button-container');
</script>
```
## Step 2: Style Buttons
Source: https://developer.paypal.com/docs/archive/checkout/how-to/customize-button/#button-styles
```js
style: {
    color:'blue',
    shape:'rect',
},
```
## Step 3: Hide Buttons
```js
<script src="https://www.paypal.com/sdk/js?client-id=test&currency=USD&disable-funding=credit"></script>
```
# Part 2: Sandbox Accounts
## Step 1: Create Sandbox Accounts
Create sandbox accounts: https://developer.paypal.com/developer/accounts, then underneath "Sandbox" click the "Account" tab
![image.png](https://img.upanh.tv/2022/05/03/image.png)
## Step 2: Create App
Create sandbox accounts: https://developer.paypal.com/developer/accounts, then underneath "Dashboard" click the "My Apps & Credential" tab and click the Create App

Once you create your new app, open it up and copy the client ID
![imagef6678ad7c3c71151.png](https://img.upanh.tv/2022/05/03/imagef6678ad7c3c71151.png)
Once we have our client ID copied,open up checkout.html and replace"client-id=sb"with what your apps client id"client-id=YOUR
CLIENT-ID*

```js
<script src="https://www.paypal.com/sdk/js?client-id=AbZopZ0KXlT9rnUVjAV2TuD3XWoynQYnFrqz0tkez80z72yHuT3cJSUb-qRaWMPNJFlufzcZktEtSAuc&currency=USD&disable-funding=credit"></script>
```
# Part 3: Making Payments
## Step 1: Test Payments
Log in Sandbox PayPal accounts and test now!
## Step 2: Setting Price
Now that our buttons are working and we have seenatransaction go through,we can set the price to represent our actual cart total and
not just $0.01.

**Move"total"**

Let's first start by moving our"total"variable to the top of the script tag we added for our paypal buttons. We need to access this total
without our buttons so we need this value available right away.
```js
var total = '{{order.get_cart_total}}'
```

**Add total to buttons**

Now we can replace $0.01 with our cart total. Because this value isastring we will need to convert it total float and then ensure that we
only have two decimal places to the right.
```js
value:parseFloat(total).toFixed(2)
```
The result is:
[![image.png](https://i.postimg.cc/nVw0NMmk/image.png)](https://postimg.cc/jCPH7xZw)
# Part 4: Process Order
```js
<script>
    var total = '{{order.get_cart_total}}'
    // Render the PayPal button into #paypal-button-container
    paypal.Buttons({

        style: {
            color:'blue',
            shape:'rect',
        },

        // Set up the transaction
        createOrder: function(data, actions) {
            return actions.order.create({
                purchase_units: [{
                    amount: {
                        value:parseFloat(total).toFixed(2)
                    }
                }]
            });
        },

        // Finalize the transaction
        onApprove: function(data, actions) {
            return actions.order.capture().then(function(orderData) {
                submitFormData()
                // Successful capture! For demo purposes:
                console.log('Capture result', orderData, JSON.stringify(orderData, null, 2));
                var transaction = orderData.purchase_units[0].payments.captures[0];
                alert('Transaction '+ transaction.status + ': ' + transaction.id + '\n\nSee console for all available details');

                // Replace the above to show a success message within this page, e.g.
                // const element = document.getElementById('paypal-button-container');
                // element.innerHTML = '';
                // element.innerHTML = '<h3>Thank you for your payment!</h3>';
                // Or go to another URL:  actions.redirect('thank_you.html');
            });
        }


    }).render('#paypal-button-container');
</script>
```
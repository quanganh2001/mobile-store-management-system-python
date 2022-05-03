# Part 1: Add to Cart
**Overview**

In this module we will begin to add in core user functionality such as "add to cart", "update cart" and "checkout".

For now we will only cover functionality for "Logged In" users and take care of "Guest Users" functionality in the next module.

Users will be able to checkout WITHOUT payment for now. This is so we can cover the core logic and integrate that in the module after we add in guest checkout capabilities on top of what we will add in this module.

**Using Javascript**

We will be using Javascript to handle a lot of the frontend functionality and sending data to the backend. If you're not too comfortable with Javascript I'll try to keep things as simple as possible but I definitely recommend you continue with this section after you have some base knowledge or at least give yourself time to study it along with this tutorial when something does not make sense.
## Step 1: Cart.js
Our home page "*store.html*" and cart (cart.html) will both give the user the ability to add to and remove items from our cart.

Because of this, we want to write this logic in one file and simply link it to our main template so it can be available on all pages that need it instead of having to repeat our code.

**Create JS folder + file**

In our static folder let's create a folder inside called "js". Inside the "js" folder create a file called "cart.js".
**Add JS script link to template**
Now inside "main.html" add a script tag with a file path to our new js file at the bottom of our page, just underneath where we have our Bootstrap links.
![imagee84d042b42e42988.png](https://img.upanh.tv/2022/05/01/imagee84d042b42e42988.png)
**Confirm Link Connection**
Just to make sure it's linked up correctly: try to `console.log("Hello World");` and open up the page to check the console
[![image.png](https://i.postimg.cc/L86zrFBt/image.png)](https://postimg.cc/rK30rbHs)
## Step 2: Add Event Handlers
Let's add an event handler to the buttons on store.html
**Add Class to Buttons**

First let's add the class of "*update-cart*" to the add button
```py
{% extends 'store/main.html' %}
{% load static %}
{% block content %}
    <div class="row">
        {% for product in products %}
        <div class="col-lg-4">
            <img class="thumbnail" src="{{product.imageURL}}">
            <div class="box-element product">
                <h6><strong>{{product.name}}</strong></h6>
                <hr>
                <button class="btn btn-outline-secondary add-btn update-cart">Add to Cart</button>
                <a class="btn btn-outline-success" href="#">View</a>
                <h4 style="display: inline-block;float: right">
                    ${{product.price|floatformat:2}}</h4>
            </div>
        </div>
        {% endfor %}
    </div>
{% endblock content %}
```
**Product ID & Action**

We also want to add in a custom attribute to the button so we know which product was clicked. To set a custom
attribute all we need to add in data-"attribute_name=value".

In our case we will call ours "product" so go ahead and add data-product="{{product.id}}". On each iteration the we will pass in the product id as the "product" attribute.

Let's also add an action attribute that lets us know that when this button is clicked we want to "add" an item. This will
make more sense later when we add in the ability to remove an item or decrease quanity in the cart.
```html
<button data-product="{{product.id}}" data-action="add" class="btn btn-outline-secondary add-btn update-cart">Add to Cart</button>
```

**Add Event**

Now to finish this off, let's query all the buttons by the class of update cart in cart.js and add event handler in a loop
```js
var updateBtns = document.getElementsByClassName('update-cart')

for (i = 0; i < updateBtns.length; i++) {
    updateBtns[i].addEventListener('click', function() {
        var productId = this.dataset.product
        var action = this.dataset.action
        console.log('productId:', productId, 'action:', action)
    })
}
```
To test things out, let's console out the product id and action and click some of the buttons
[![image.png](https://i.postimg.cc/cCshDgDW/image.png)](https://postimg.cc/njPqrzbS)
## Step 3: Use Type Logic
Depending on whether the user is logged in or not, we want the add to cart button to behave differently on each click.

For example; when a user is logged in, we want to send data to the backend and add this item to the database. But if a user DOE'S NOT have an account, we want to simply add some data to the browser and store it.

For now we'll just stick to logged in users and ignore clicks made by Non-logged in site visitors.

**Set/Query user (main.html)**

First, we want to query the users to check his/her status. In main.html within the header tag, let's create a script tag and set the variable of "user" to "request.user". If a user is not logged in, the value will be "AnonymousUser".
```html
<!DOCTYPE html>
{% load static %}
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, minimum-scale=1" />

    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/css/bootstrap.min.css" integrity="sha384-Vkoo8x4CGsO3+Hhxv8T/Q5PaXtkKtu6ug5TOeNV6gBiFeWPGFN9MuhOf23Q9Ifjh" crossorigin="anonymous">
    
    <title>Ecom</title>
    <link rel="stylesheet" type="text/css" href="{% static 'css/main.css' %}">

    <script type="text/javascript">
      var user = '{{request.user}}'
      
    </script>
</head>
```
Because the cart.js file is at the bottom of main.html, we can access this user variable within cart.js

**Check user status on click (cart.js)**

Let's create an if statement inside cart.js and console the user and two different statements depending on whether the user is logged in or not. Make sure this is inside our event handler that we added earlier and test it by logging in and out and checking the console.
```js
var updateBtns = document.getElementsByClassName('update-cart')

for (i = 0; i < updateBtns.length; i++) {
    updateBtns[i].addEventListener('click', function() {
        var productId = this.dataset.product
        var action = this.dataset.action
        console.log('productId:', productId, 'action:', action)

        console.log('USER:', user)
        if (user == 'AnonymousUser'){
            console.log('Not logged in')
        } else{
            console.log('User is logged in, sending data...')
        }
    })
}
```
Test in console tab
[![image.png](https://i.postimg.cc/VsGhTQhX/image.png)](https://postimg.cc/kDSw6Zq5)
## Step 4: updateItem View
When a logged in user clicks "add to cart", we want to send this data to a view to proccess this action.

On click we will send the Product id along with an action of "add" OR "remove" to the view we are about to create.

The view will then be responsible for creating or adding quanitity to an Orderltem OR removing or deleting the item if
the action sent over is "remove".

**Create View**

First, let's create a function in our views.py called `updateltem` and set the return value to a simple JsonResponse. You'll
want to import Json response at the top of your "views.py" file to make this available.
````py
from django.shortcuts import render
from django.http import JsonResponse

from .models import *

# Create your views here.

def store(request):
    products = Product.objects.all()
    context = {'products':products}
    return render(request, 'store/store.html', context)

def cart(request):

    if request.user.is_authenticated:
        customer = request.user.customer
        order, created = Order.objects.get_or_create(customer=customer, complete=False)
        items = order.orderitem_set.all()
    else:
        items = []
        order = {'get_cart_total':0, 'get_cart_items':0}

    context = {'items':items, 'order':order}
    return render(request, 'store/cart.html', context)

def checkout(request):
    if request.user.is_authenticated:
        customer = request.user.customer
        order, created = Order.objects.get_or_create(customer=customer, complete=False)
        items = order.orderitem_set.all()
    else:
        # Create empty cart for now for none-logged in users
        items = []
        order = {'get_cart_total':0, 'get_cart_items':0}

    context = {'items':items, 'order':order}
    return render(request, 'store/checkout.html', context)


def updateItem(request):
    return JsonResponse('Item was added', safe=False)
````
**URL Path**
Now in our "urls.py" file, let's create a path for our new view

File: store/views.py
```py
from django.urls import path
from . import views

urlpatterns = [
    # Leave as empty string for base url
    path('', views.store, name="store"),
    path('cart/', views.cart, name="cart"),
    path('checkout/', views.checkout, name="checkout"),

    path('update_item/', views.updateItem, name="update_item"),
]
```
## Step 5: updateUserOrder()
We now have an event handler that identifies which type of user has clicked the "add" button (Logged in/guest) and a view.url to send data if that user is logged in.

Let's now create a function that gets triggered if our user is logged in and have that function send a POST request to
our new view (updateltem).

**Create Function (*updateUserOrder*())**

Just underneath our loop in cart.js, let's create a function called updateUserOrder() and give it two parameters,
"productld" and "action". Then pass the function into our "if" statement so it gets called when the user is logged in.

We can just pass the original console statement into the new function.
```js
var updateBtns = document.getElementsByClassName('update-cart')

for (i = 0; i < updateBtns.length; i++) {
    updateBtns[i].addEventListener('click', function() {
        var productId = this.dataset.product
        var action = this.dataset.action
        console.log('productId:', productId, 'action:', action)

        console.log('USER:', user)
        if (user == 'AnonymousUser'){
            console.log('Not logged in')
        } else{
            updateUserOrder(productId, action)
        }
    })
}

function updateUserOrder(productId, action) {
    console.log('User is logged in, sending data...')

    var url = '/update_item/'

    fetch(url, {
        method:'POST',
        headers:{
            'Content-Type':'application/json',
        },
        body:JSON.stringify({'productId':productId, 'action':action})
    })

    .then((response) => {
        return response.json();
    })

    .then((data) => {
        console.log('Data:', data)
    });
}
```
## Step 6: CSRF Token
Source: https://docs.djangoproject.com/en/3.0/ref/csrf/#ajax
```html
    <script type="text/javascript">
      var user = '{{request.user}}'


      function getCookie(name) {
        var cookieValue = null;
        if (document.cookie && document.cookie !== '') {
            const cookies = document.cookie.split(';');
            for (let i = 0; i < cookies.length; i++) {
                const cookie = cookies[i].trim();
                // Does this cookie string begin with the name we want?
                if (cookie.substring(0, name.length + 1) === (name + '=')) {
                    cookieValue = decodeURIComponent(cookie.substring(name.length + 1));
                    break;
                }
            }
        }
        return cookieValue;
      }
      var csrftoken = getCookie('csrftoken');
    </script>
```
### Send Token with POST request
Now that we have the "csrftoken" variable set, we can pass it into our fetch() call.
```html
    <script type="text/javascript">
      var user = '{{request.user}}'


      function getToken(name) {
        var cookieValue = null;
        if (document.cookie && document.cookie !== '') {
            const cookies = document.cookie.split(';');
            for (let i = 0; i < cookies.length; i++) {
                const cookie = cookies[i].trim();
                // Does this cookie string begin with the name we want?
                if (cookie.substring(0, name.length + 1) === (name + '=')) {
                    cookieValue = decodeURIComponent(cookie.substring(name.length + 1));
                    break;
                }
            }
        }
        return cookieValue;
      }
      var csrftoken = getToken('csrftoken');
    </script>
```
## Step 7: updateItem view logic
```py
from django.shortcuts import render
from django.http import JsonResponse
import json
from numpy import product

from .models import *

# Create your views here.

def store(request):
    products = Product.objects.all()
    context = {'products':products}
    return render(request, 'store/store.html', context)

def cart(request):

    if request.user.is_authenticated:
        customer = request.user.customer
        order, created = Order.objects.get_or_create(customer=customer, complete=False)
        items = order.orderitem_set.all()
    else:
        items = []
        order = {'get_cart_total':0, 'get_cart_items':0}

    context = {'items':items, 'order':order}
    return render(request, 'store/cart.html', context)

def checkout(request):
    if request.user.is_authenticated:
        customer = request.user.customer
        order, created = Order.objects.get_or_create(customer=customer, complete=False)
        items = order.orderitem_set.all()
    else:
        # Create empty cart for now for none-logged in users
        items = []
        order = {'get_cart_total':0, 'get_cart_items':0}

    context = {'items':items, 'order':order}
    return render(request, 'store/checkout.html', context)


def updateItem(request):
	data = json.loads(request.body)
	productId = data['productId']
	action = data['action']
	print('Action:', action)
	print('Product:', productId)

	customer = request.user.customer
	product = Product.objects.get(id=productId)
	order, created = Order.objects.get_or_create(customer=customer, complete=False)

	orderItem, created = OrderItem.objects.get_or_create(order=order, product=product)

	if action == 'add':
		orderItem.quantity = (orderItem.quantity + 1)
	elif action == 'remove':
		orderItem.quantity = (orderItem.quantity - 1)

	orderItem.save()

	if orderItem.quantity <= 0:
		orderItem.delete()

	return JsonResponse('Item was added', safe=False)
```
## Step 8: Cart Total
This is where we would really benefit from using more Javascript and having some sort of API built out. But, because
I'm trying to limit that in this tutorial, we are going to need to query the user in each view and render out the total so
that we can access this value on every page since it's our navigation bar.

Again, there are many ways to go about this but I'm doing things manually for effect.

**User data in store view**

It's going to get a bit messy now. But, don't worry, because we will clean this up in the next module.

Let's start by first copying our user query in the "if" statement and adding it to our store view.
```js
function updateUserOrder(productId, action) {
    console.log('User is authenticate, sending data...')

    var url = '/update_item/'

    fetch(url, {
        method:'POST',
        headers:{
            'Content-Type':'application/json',
            'X-CSRFToken':csrftoken,
        },
        body:JSON.stringify({'productId': productId, 'action':action})
    })

    .then((response) => {
        return response.json();
    })

    .then((data) => {
        console.log('Data:', data)
        location.reload()
    });
}
```
```py
from django.shortcuts import render
from django.http import JsonResponse
import json
from numpy import product

from .models import *

# Create your views here.

def store(request):
    if request.user.is_authenticated:
        customer = request.user.customer
        order, created = Order.objects.get_or_create(customer=customer, complete=False)
        items = order.orderitem_set.all()
        cartItems = order.get_cart_items
    else:
        # Create empty cart for now for none-logged in users
        items = []
        order = {'get_cart_total':0, 'get_cart_items':0}
        cartItems = order['get_cart_items']

    products = Product.objects.all()
    context = {'products':products, 'cartItems':cartItems}
    return render(request, 'store/store.html', context)

def cart(request):

    if request.user.is_authenticated:
        customer = request.user.customer
        order, created = Order.objects.get_or_create(customer=customer, complete=False)
        items = order.orderitem_set.all()
        cartItems = order.get_cart_items
    else:
        items = []
        order = {'get_cart_total':0, 'get_cart_items':0}
        cartItems = order['get_cart_items']

    context = {'items':items, 'order':order, 'cartItems':cartItems}
    return render(request, 'store/cart.html', context)

def checkout(request):
    if request.user.is_authenticated:
        customer = request.user.customer
        order, created = Order.objects.get_or_create(customer=customer, complete=False)
        items = order.orderitem_set.all()
        cartItems = order.get_cart_items
    else:
        # Create empty cart for now for none-logged in users
        items = []
        order = {'get_cart_total':0, 'get_cart_items':0}
        cartItems = order['get_cart_items']

    context = {'items':items, 'order':order, 'cartItems':cartItems}
    return render(request, 'store/checkout.html', context)


def updateItem(request):
	data = json.loads(request.body)
	productId = data['productId']
	action = data['action']
	print('Action:', action)
	print('Product:', productId)

	customer = request.user.customer
	product = Product.objects.get(id=productId)
	order, created = Order.objects.get_or_create(customer=customer, complete=False)

	orderItem, created = OrderItem.objects.get_or_create(order=order, product=product)

	if action == 'add':
		orderItem.quantity = (orderItem.quantity + 1)
	elif action == 'remove':
		orderItem.quantity = (orderItem.quantity - 1)

	orderItem.save()

	if orderItem.quantity <= 0:
		orderItem.delete()

	return JsonResponse('Item was added', safe=False)
```
```html
<p id="cart-total">{{cartItems}}</p>
```
The output is:
[![image.png](https://i.postimg.cc/J7B1w6PF/image.png)](https://postimg.cc/pyRNzZNQ)
# Part 2: Update Cart
```html
                {% for item in items %}
                <div class="cart-row">
                    <div style="flex: 2"><img class="row-image" src="{{item.product.imageURL}}"></div>
                    <div style="flex: 2">{{item.product.name}}</div>
                    <div style="flex: 1">${{item.product.price|floatformat:2}}</div>
                    <div style="flex: 1">
                        <p class="quantity">{{item.quantity}}</p>
                        <div class="quantity">
                            <img data-product="{{item.product.id}}" data-action="add" src="{% static 'images/arrow-up.png' %}" class="chg-quantity update-cart">

                            <img data-product="{{item.product.id}}" data-action="remove" src="{% static 'images/arrow-down.png' %}" class="chg-quantity update-cart">
                        </div>
                    </div>
                    <div style="flex: 1"><p>${{item.get_total}}</p></div>
                </div>
                {% endfor %}
```
Now you can click up to add quantity or down to minus quantity
[![image.png](https://i.postimg.cc/NjGFw8yr/image.png)](https://postimg.cc/F7qNyLV9)
# Part 3: Shipping Address
## Step 1: Shipping Method
The first thing we need to do is create a method for our "Order" model that checks if we have any items that are not
digital.

This can be done with a simple loop that changes the status of "shipping" to "true" if a child Orderltem is not digital.

Then the method simply returns a true OR false value.
```py
class Order(models.Model):
    customer = models.ForeignKey(Customer, on_delete=models.SET_NULL, blank=True, null=True)
    date_ordered = models.DateTimeField(auto_now_add=True)
    complete = models.BooleanField(default=False, null=True, blank=False)
    transaction_id = models.CharField(max_length=200, null=True)

    def __str__(self):
        return str(self.id)
    
    @property
    def shipping(self):
        shipping = False
        orderitems = self.orderitem_set.all()
        for i in orderitems:
            if i.product.digital == False:
                shipping = True
        return shipping
```
## Step 2: Order Shipping Status
Since we added the "shipping" method to our order we can use it in the template right away
other property.

ex: {{ order.shipping}}

The thing we need to plan for is when a non-logged in user visits the page. Our order dictionary doe's not have a
"shipping" attribute, so we need to add it in our cart view. To keep consistency, let's add "shipping" to all 3 views in
the order object dictionary.

`order = {'get_cart_total":0, 'get_cart_items':0, "shipping":False}`
```py
from django.shortcuts import render
from django.http import JsonResponse
import json
from numpy import product

from .models import *

# Create your views here.

def store(request):
    if request.user.is_authenticated:
        customer = request.user.customer
        order, created = Order.objects.get_or_create(customer=customer, complete=False)
        items = order.orderitem_set.all()
        cartItems = order.get_cart_items
    else:
        # Create empty cart for now for none-logged in users
        items = []
        order = {'get_cart_total':0, 'get_cart_items':0, 'shipping':False}
        cartItems = order['get_cart_items']

    products = Product.objects.all()
    context = {'products':products, 'cartItems':cartItems}
    return render(request, 'store/store.html', context)

def cart(request):

    if request.user.is_authenticated:
        customer = request.user.customer
        order, created = Order.objects.get_or_create(customer=customer, complete=False)
        items = order.orderitem_set.all()
        cartItems = order.get_cart_items
    else:
        items = []
        order = {'get_cart_total':0, 'get_cart_items':0, 'shipping':False}
        cartItems = order['get_cart_items']

    context = {'items':items, 'order':order, 'cartItems':cartItems}
    return render(request, 'store/cart.html', context)

def checkout(request):
    if request.user.is_authenticated:
        customer = request.user.customer
        order, created = Order.objects.get_or_create(customer=customer, complete=False)
        items = order.orderitem_set.all()
        cartItems = order.get_cart_items
    else:
        # Create empty cart for now for none-logged in users
        items = []
        order = {'get_cart_total':0, 'get_cart_items':0, 'shipping':False}
        cartItems = order['get_cart_items']

    context = {'items':items, 'order':order, 'cartItems':cartItems}
    return render(request, 'store/checkout.html', context)
```
## Step 3: Hide Shipping Form
We need to get the status of "shipping" from our "order" object and remove the address field if shipping is false.

Let's write our Javascript right in "checkout.html". Since this is the only page, we will need this functionality.

Create a script tag just above our {% endblock %)
```html
    <script type="text/javascript">
        var shipping = '{{order.shipping}}'

        if(shipping == 'False'){
            document.getElementById('shipping-info').innerHTML = ''
        }
    </script>
```
The output is:
[![image.png](https://i.postimg.cc/44QkzDdp/image.png)](https://postimg.cc/c6CbW5fJ)
## Step 4: Payment Option
When a user adds form data and clicks continue; we want to open up the payment option but still let them edit the
form. Let's first create an event handler that hides the form button and opens up the payment option wrapper
(payment-info), it currently has the class of "hidden".

When the form is submitted, we want to remove the class of "hidden" from the payment option and add the class of
"hidden" to the button.

**Hide button & open payment option on submit**

We'll query the form by id and set the event listener on submit.
```html
    <script type="text/javascript">
        var shipping = '{{order.shipping}}'

        if(shipping == 'False'){
            document.getElementById('shipping-info').innerHTML = ''
        }

        var form = document.getElementById('form')
        form.addEventListener('submit', function(e){
            e.preventDefault()
            console.log('Form Submitted...')
            document.getElementById('form-button').classList.add("hidden");
            document.getElementById('payment-info').classList.remove("hidden");
        })
    </script>
```
This will open up the payment-info wrapper as shown below. Notice when click Continue, the button is not visible
[![image.png](https://i.postimg.cc/Kz9wRMxj/image.png)](https://postimg.cc/r0rQHzd2)
**Add demo Payment button**

Inside the payment-info wrapper: let's add a button with the id of "make-payment". This will be the placeholder that
will trigger orders to be processed until we get to the payment integration module.
```html
            <div class="box-element hidden" id="payment-info">
                <small>Paypal Options</small>
                <button id="make-payment">Make Payment</button>
            </div>
```
## Step 5: Trigger Payment Action
Let's add an event handler to the new "payment-submit" button and a function to trigger on submission, we'll take
care of what the function does in the next part.
```html
    <script type="text/javascript">
        var shipping = '{{order.shipping}}'

        if(shipping == 'False'){
            document.getElementById('shipping-info').innerHTML = ''
        }

        var form = document.getElementById('form')
        form.addEventListener('submit', function(e){
            e.preventDefault()
            console.log('Form submitted...')
            document.getElementById('form-button').classList.add("hidden");
            document.getElementById('payment-info').classList.remove("hidden");
        })

        document.getElementById('make-payment').addEventListener('click', function(e){
            submitFormData()
        })

        function submitFormData(){
            console.log('Payment button clicked')
        }
    </script>
```
# Part 4: Checkout Form
## Step 1: Hide Form or Fields
**Page Logic Explanation**

Scenario #1: If user is logged in → Hide *Name* and *Email* Fields

Scenario #2: If a user is logged in AND Item does NOT need shipping → *Hide* form & *Open* Payment option.

A logged in user does not need to see the email and name field because we already know based on the account they
have.

checkout.html
```js
        if (user != 'AnonymousUser'){
            document.getElementById('user-info').innerHTML = ''
        }

        if (shipping == 'False' && user != 'AnonymousUser'){
            // Hide entire form if user is logged in and shipping is false
            document.getElementById('form-wrapper').classList.add("hidden");
            // Show payment if logged in user wants to buy an item that does not require shipping
            document.getElementById('payment-info').classList.remove("hidden");
        }

```
## Step 2: Form Data
Now it's time to prep our data before we send it off to our backend to process this order.

**What data are we sending?**

When the payment button is submitted we want to send 3 things to the backend for proccesing the order.

1 - Cart Total
2- User Information (If user is NOT logged in)
3- Shipping Address (If item in order needs shipping)

**Cart total**

We can get the cart total by simply querying the order object. Go ahead and set the total next to the "shipping
variable".
```html

```
**User & Shipping Data**

We want to send "user" and "shipping" as two separate Javascript objects so we can access them separately in the
backend.

Inside the submitFormData() function; let's create an object representation of both. For the user; let's add in "total" and
pass in the variable we set just above.
```js
        function submitFormData(){
            console.log('Payment button clicked')

            var userFormData = {
                'name':null,
                'email':null,
                'total':total,
            }

            var shippingInfo = {
                'address':null,
                'city':null,
                'state':null,
                'zipcode':null,
            }
        }
```
Now that we have our objects, we want to set values for these objects attributes on submission before we send it off
to the backend. Inside our "submitFormData" function, just underneath where we set the two Javascript objects, let's
set the values from the form.
```js
        function submitFormData(){
            console.log('Payment button clicked')

            var userFormData = {
                'name':null,
                'email':null,
                'total':total,
            }

            var shippingInfo = {
                'address':null,
                'city':null,
                'state':null,
                'zipcode':null,
            }

            if(shipping != 'False'){
                shippingInfo.address = form.address.value
                shippingInfo.city = form.city.value
                shippingInfo.state = form.state.value
                shippingInfo.zipcode = form.zipcode.value
            }

            if(user == 'AnonymousUser'){
                userFormData.name = form.name.value
                userFormData.email = form.email.value
            }

            console.log('Shipping Info:', shippingInfo)
            console.log('User Info:', userFormData)
        }
```
# Part 5: Process Order
## Step 1: Process Order View/Url
Let's first create our "view" and "url" pattern for our POST request to send data too (views.py)
```py
def processOrder(request):
	return JsonResponse('Payment complete!', safe=False)
```
```py
from django.urls import path
from . import views

urlpatterns = [
    # Leave as empty string for base url
    path('', views.store, name="store"),
    path('cart/', views.cart, name="cart"),
    path('checkout/', views.checkout, name="checkout"),

    path('update_item/', views.updateItem, name="update_item"),
    path('process_order/', views.processOrder, name="process_order"),
]
```
As you can see, the output is Payment complete!
## Step 2: Send POST Data
Now we can create a post call and send the data to the backend. Remember, we still have access to the csrf token
because we inherit from main.html

Inside our "submitFormData" function (checkout.html), let's create a post request using "fetch()".

Set the variable of "url" to the new path we just created in the last step. Next, add in the csrf token along with a
Javascript object while nesting our user information and shipping data.

Remember to stringify the data.

Once we send the POST data, let's send the user back to our home page in the promise.
```js
            var url = '/process_order/'
            fetch(url,{
                method:'POST',
                headers:{
                    'Content-Type':'application/json',
                    'X-CSRFToken':csrftoken,
                },
                body:JSON.stringify({'form':userFormData, 'shipping':shippingInfo})
            })
            .then((response) => response.json())
            .then((data) => {
                console.log('Success:', data);
                alert('Transaction completed');
                window.location.href = "{% url 'store' %}"
            })
```
## Step 3: Transaction ID:
To create our transaction ID; we won't do anything too fancy. There are many different methods we can use, but I'm
going to use a simple time stamp.

**Import Date Time (views.py)**
```py
import datetime
```
**Set Transaction ID variable**
```py
def processOrder(request):
    transcation_id = datetime.datetime.now().timestamp()
	return JsonResponse('Payment submitted...', safe=False)
```
## Step 4: Set Data
Now we want to parse the data sent from the post request and query/create some data if a user is authenticated.

Again, we will ignore unauthenticated users until the next module.

We will use `json.loads()` to parse the data. Once we set the return value to the "data" variable, we can query the items
inside as a Python dictionary.
```py
def processOrder(request):
    transcation_id = datetime.datetime.now().timestamp()
    date = json.loads(request.body)

    if request.user.is_authenticated:
        customer = request.user.customer
        order, created = Order.objects.get_or_create(customer=customer, complete=False)
        total = float(data['form']['total'])
        order.transaction_id = transcation_id
        
    else:
        print('User is not logged in...')
	return JsonResponse('Payment submitted...', safe=False)
```
## Step 5: Confirm Total
Because we will be sending the total from the frontend, we want to make sure that the total sent matches what the
cart total is actually supposed to be.

We can do this by running a simple comparison from the total sent and our cart total.
```py
def processOrder(request):
    transcation_id = datetime.datetime.now().timestamp()
    date = json.loads(request.body)

    if request.user.is_authenticated:
        customer = request.user.customer
        order, created = Order.objects.get_or_create(customer=customer, complete=False)
        total = float(data['form']['total'])
        order.transaction_id = transcation_id

        if total == order.get_cart_total:
            order.complete = True
        order.save()
```
## Step 6: Shipping Logic
Last,
we need to create an instance of the shipping address if an address was sent.

We can add this just underneath our total confirmation. Check the source code link if you are unsure exactly where in
the view we are adding this.
```py
def processOrder(request):
	transaction_id = datetime.datetime.now().timestamp()
	data = json.loads(request.body)

	if request.user.is_authenticated:
		customer = request.user.customer
		order, created = Order.objects.get_or_create(customer=customer, complete=False)
	else:
		customer, order = guestOrder(request, data)

	total = float(data['form']['total'])
	order.transaction_id = transaction_id

	if total == order.get_cart_total:
		order.complete = True
	order.save()

	if order.shipping == True:
		ShippingAddress.objects.create(
		customer=customer,
		order=order,
		address=data['shipping']['address'],
		city=data['shipping']['city'],
		state=data['shipping']['state'],
		zipcode=data['shipping']['zipcode'],
		)

	return JsonResponse('Payment submitted..', safe=False)
```
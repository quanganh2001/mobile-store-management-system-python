# Part 1: Set Cookie's
## Step 1: Create Cart
When a visitor comes to our website, we want to create a cookie for our cart. Regardless of whether they are logged in or not.

In our "main.html" file, within the script tag, I am going to add in a function that will search for and parse a specific cookie that is stored in
our browser.

Once we have the function, we are going to search for a cookie called "cart", and if we cannot find one we are going to create + set it.

**Get Cookie**

First. let's start by adding the getCookie() function into the script tag just underneath our csrf token, you can find it in the source code or
code block.

Don't worry about what the function does for now. I did some research and just copied a cookie parser function myself so honestly, I can't
say I know too much about it other than the fact that it works.

Once we set the function, let's set the variable "cart" to the return value of "cart" from our cookie parser.
```js
function getCookie(name) {
    // Split cookie string and get all individual name=value pairs in an array
    var cookieArr = document.cookie.split(";");

    // Loop through the array elements
    for(var i = 0; i < cookieArr.length; i++) {
        var cookiePair = cookieArr[i].split("=");

        /* Removing whitespace at the beginning of the cookie name and compare
        it with the given string */
        if(name == cookiePair[0].trim()) {
        // Decode the cookie value and return
        return decodeURIComponent(cookiePair[1]);
        }
    }

    // Return null if not found
    return null;
}
var cart = JSON.parse(getCookie('cart'))
```
**Create Cookie**

If our browser does not contain a cookie called "cart", which is expected on the first load, let's create one

(file main.html)
```js
var cart = JSON.parse(getCookie('cart'))
if(cart == undefined){
    cart = {}
    console.log('Cart was created!')
    document.cookie = 'cart=' + JSON.stringify(cart) + ";domain=;path=/"
}

console.log('Cart:', cart)
```
First check the status of cart, then set cart to an empty Javasctript object and use document.cookie to set one.

A cookie is a key-value part so we set the name with "cart=" and then stringify the Javascript object (cart), then set the domain so we
have this same cookie on every page.

If we don't add ";domain=;path/" then we will create a new cookie on each page and our cart will NOT be consistent, so be sure to add
this.
## Step 2: Adding/Removing Items
Several
steps back in module #3 we created an *add to cart* feature in "cart.js" which sent a post request and added items to our database.

On "click" there was an "if" statement that checked the users status and at this point we essentially ignored clicks made by users that
were not logged in. We want to create a function that will be triggered on the first condition.
```js
for (i = 0; i < updateBtns.length; i++) {
    updateBtns[i].addEventListener('click', function() {
        var productId = this.dataset.product
        var action = this.dataset.action
        console.log('productId:', productId, 'action:', action)

        console.log('USER:', user)
        if (user == 'AnonymousUser'){
            addCookieItem()
        } else{
            updateUserOrder(productId, action)
        }
    })
}
```
Add to cookie item function

Let's create a function below our "updateUserOrder" funciton in cart.js called "addCookieltemn()".
```js
function addCookieItem(productId, action) {
    console.log('Not logged in')
}
```
This function, just like updateUserOrder(), will take in the "productid" and "action" as parameters and will be triggered on the first
condition within our "if' statement.

If you want to ensure everything is working, be sure to log out and test by clicking the "add" button and checking the console.

**Update Cookie**

Inside our addCookieltem() function, let's add some logic that determines what happens to our cart.

First, we want to check the "action" type to know if we need to add or remove an item.

**Add or Increase item Quantity**

If the action is "add", we want to check if this item is already in the cart. If not, then we create it. If we already have that item in our cart,
we simply want to add to the quantity,
```js
function addCookieItem(productId, action) {
    console.log('Not logged in')

    if (action == 'add'){
        if (cart[productId] == undefined){
            cart[productId] = {'quantity':1}
        }else{
            cart[productId]['quantity'] += 1
        }
    }
}
```
Our cookie object will contain nested objects with the id of the product being the id and the quantity.

**Remove or Decrease item Quantity**

If the action is "remove", we want to either decrease the quantity, or, if the quantity is equal to or below zero, remove it from our cart
altogether.
```js
function addCookieItem(productId, action) {
    console.log('Not logged in...')

    if (action == 'add'){
        if (cart[productId] == undefined){
            cart[productId] = {'quantity':1}
        }else{
            cart[productId]['quantity'] += 1
        }
    }

    if(action == 'remove'){
        cart[productId]['quantity'] -= 1

        if(cart[productId]['quantity'] <= 0){
            console.log('Remove Item')
            delete cart[productId];
        }
    }
    console.log('Cart:', cart)
    document.cookie = 'cart=' + JSON.stringify(cart) + ";domain=;path=/"
    location.reload()
}
```
The output is:
![image8225a43d30b8d557.png](https://img.upanh.tv/2022/05/02/image8225a43d30b8d557.png)
# Part 2: Render cart Total

**Overview:**

In this step, we are going to cover retrieving our browsers cart values in our Django views and setting/rendering out the total for the
navigation bar.

**Turn this**
## Step 1: Query Cart in views.py
Now that we have added some data to our browsers cookies; the way we render cart data for guest users is to retrieve the cookie data in
our backend (views) and build a replica of an order.

Remember the cookie we set only stores product id's and quantity. We need to work with this data to eventually build an entire cart
inside of our views. But, for now we just want to total up the items in our cart and update the "get_cart_total" in our "order" dictionary
that we have in each view.
```py
order = {'get_cart_total':0, 'get_cart_items':0, 'shipping':False}
```
## Step 2: Bulid Cart Total
Now that we have our "cart object", we want to find the total number of items the guest visitor has in their cart and update our order
objects 'get_cart_items' attribute.

For this we can simply loop through our cart dictionary and add the total quantity of each item.

When looping through a dictionary, I will represent the "key". Which, in our case, is the product id.
```py
def cart(request):

    if request.user.is_authenticated:
        customer = request.user.customer
        order, created = Order.objects.get_or_create(customer=customer, complete=False)
        items = order.orderitem_set.all()
        cartItems = order.get_cart_items
    else:
		try:
			cart = json.loads(request.COOKIES['cart'])
		except:
			cart = {}
			print('Cart:', cart)

        items = []
        order = {'get_cart_total':0, 'get_cart_items':0, 'shipping':False}
        cartItems = order['get_cart_items']

		for i in cart:
			cartItems += cart[i]["quantity"]

    context = {'items':items, 'order':order, 'cartItems':cartItems}
    return render(request, 'store/cart.html', context)
```
This loop will query the quantity of each item in our cart and add to the value of "cartitems" therefore giving us the total of all items +
quantity in the entire cart.

Now if we are logged out and add some items to our cart; we should see something like this in our cart page navbar.
# Part 3: Build Order

**Overview:**

Now its time to build a dictionary representation of a real order. Remember, a guest user never actually creates an order until checkout.
everything is stored in their browsers cookies.

Our job is to create an object representation or a real order, including items so we can render this data out in the template without adding
any extra logic in the template.

**What we will build**

We will add on to our loop from the last part and set more values on our order such as "get_cart_total", "shipping" and even build out an
entire list of objects representing cart items.
## Step 1: Order Totals
We covered getting the total item count for our cart in the last step, now we will get the cart price total (get_cart_total).

**Query Product**

First we will add on the loop in our "cart" view and query the product so we can get the price. Remember that "i" in our loop is the product
id.
```py
for i in cart:
	cartItems += cart[i]["quantity"]

	product = Product.objects.get(id=i)
```

**Get and set totals**

Once we have the product we can set the value of total by multiplying the product price by the quantity.

Go ahead and add the total (+-) to
order('get_cart_total]

Do the same for order('get_cart_items'] with the quantity since we need this value in the order also.
```py
def cart(request):

    if request.user.is_authenticated:
        customer = request.user.customer
        order, created = Order.objects.get_or_create(customer=customer, complete=False)
        items = order.orderitem_set.all()
        cartItems = order.get_cart_items
    else:
		try:
			cart = json.loads(request.COOKIES['cart'])
		except:
			cart = {}
			print('Cart:', cart)

        items = []
        order = {'get_cart_total':0, 'get_cart_items':0, 'shipping':False}
        cartItems = order['get_cart_items']

		for i in cart:
			cartItems += cart[i]["quantity"]

			product = Product.objects.get(id=i)
			total = (product.price * cart[i]['quantity'])

			order['get_cart_total'] += total
			order['get_cart_items'] += cart[i]['quantity']

    context = {'items':items, 'order':order, 'cartItems':cartItems}
    return render(request, 'store/cart.html', context)
```
## Step 2: Item Queryset
Just like we had to create a representation of an order with a dictionary we will need to do the same for items in our cart.

In order for our cart and checkout page to render these items and all the proper information, we need it to look exactly like a real item so
our template doesn't know the difference.

In each iteration of the loop for items in our cart, we will create an item object and append to our items list
```py
def cart(request):

    if request.user.is_authenticated:
        customer = request.user.customer
        order, created = Order.objects.get_or_create(customer=customer, complete=False)
        items = order.orderitem_set.all()
        cartItems = order.get_cart_items
    else:
		try:
			cart = json.loads(request.COOKIES['cart'])
		except:
			cart = {}
			print('Cart:', cart)

        items = []
        order = {'get_cart_total':0, 'get_cart_items':0, 'shipping':False}
        cartItems = order['get_cart_items']

		for i in cart:
			cartItems += cart[i]["quantity"]

			product = Product.objects.get(id=i)
			total = (product.price * cart[i]['quantity'])

			order['get_cart_total'] += total
			order['get_cart_items'] += cart[i]['quantity']

			item = {
				'product':{
					'id':product.id,
					'name':product.name,
					'price':product.price,
					'imageURL':product.imageURL
				},
				'quantity':cart[i]['quantity'],
				'get_total':total,
			}
            items.append(item)

    context = {'items':items, 'order':order, 'cartItems':cartItems}
    return render(request, 'store/cart.html', context)
```
The item object will contain ALL the same attributes that our Orderitem model does. As you build this page look closely at all the code I
provide with this.

Now if we refresh our cart page we should see all the items rendered in our table.
## Step 3: Shipping Information
Because this data will be the same across all pages, let's add in a check for shipping information.

Just underneath where we append the item to the items list, let's query the products "digital" attribute and Django the value of "shipping"
to try if one of the items is NOT a digital item.
```py
def cart(request):

    if request.user.is_authenticated:
        customer = request.user.customer
        order, created = Order.objects.get_or_create(customer=customer, complete=False)
        items = order.orderitem_set.all()
        cartItems = order.get_cart_items
    else:
		try:
			cart = json.loads(request.COOKIES['cart'])
		except:
			cart = {}
			
		print('Cart:', cart)
        items = []
        order = {'get_cart_total':0, 'get_cart_items':0, 'shipping':False}
        cartItems = order['get_cart_items']

		for i in cart:
			cartItems += cart[i]["quantity"]

			product = Product.objects.get(id=i)
			total = (product.price * cart[i]['quantity'])

			order['get_cart_total'] += total
			order['get_cart_items'] += cart[i]['quantity']

			item = {
				'product':{
					'id':product.id,
					'name':product.name,
					'price':product.price,
					'imageURL':product.imageURL
				},
				'quantity':cart[i]['quantity'],
				'get_total':total,
			}
			items.append(item)

			if product.digital == False:
				order['shipping'] = True

    context = {'items':items, 'order':order, 'cartItems':cartItems}
    return render(request, 'store/cart.html', context)
```
## Step 4: Product Does Not Exist
A potential problem we can face is a user adding a product to their cart and in the time frame it takes the user to checkout, let's say the
admin removes the item from the database.

This means we will query a product with bad id. We can fix this by adding a "try/except" block within our loop and ignoring this issue.
```py
for i in cart:
    try:
        cartItems += cart[i]["quantity"]

        product = Product.objects.get(id=i)
        total = (product.price * cart[i]['quantity'])

        order['get_cart_total'] += total
        order['get_cart_items'] += cart[i]['quantity']

        item = {
            'product':{
                'id':product.id,
                'name':product.name,
                'price':product.price,
                'imageURL':product.imageURL
            },
            'quantity':cart[i]['quantity'],
            'get_total':total,
        }
        items.append(item)

        if product.digital == False:
            order['shipping'] = True
    except:
        pass
```
# Part 4: cookieCart() Function
**Overview:**

At this point we have taken our browsers cart cookie and made a full cart (Order) with cart items (Orderitem). The problem with this is we
have about 30 lines of code that we will have to be using in multiple views. We need to use this data in multiple pages and that can make
our code messy.

**Let's make our code reusable**

To stick with the concept of D.R.Y. (Don't Repeat Yourself), let's be good programmers and build one function that creates this cart for us and
just call it in each view to keep things clean.
## Step 1: utils.py
**Create new file | utils.py**

Let's create a new file called "utils.py" inside our app. Let's import json and our models into our new file.
```py
import json
from .models import
```
**Create Function | cookieCart()**

Create a function called cookieCart(). The purpose of this function is to handle all the logic we created for our guest users order.
This function will take in "request" as a parameter.
```py
import json
from . models import *

def cookieCart(request):
    try:
		cart = json.loads(request.COOKIES['cart'])
	except:
		cart = {}

    print('Cart:', cart)
    items = []
    order = {'get_cart_total':0, 'get_cart_items':0, 'shipping':False}
    cartItems = order['get_cart_items']

    for i in cart:
        try:
            cartItems += cart[i]["quantity"]

            product = Product.objects.get(id=i)
            total = (product.price * cart[i]['quantity'])

            order['get_cart_total'] += total
            order['get_cart_items'] += cart[i]['quantity']

            item = {
                'product':{
                    'id':product.id,
                    'name':product.name,
                    'price':product.price,
                    'imageURL':product.imageURL
                },
                'quantity':cart[i]['quantity'],
                'get_total':total,
            }
            items.append(item)

            if product.digital == False:
                order['shipping'] = True
        except:
            pass
    return {'cartItems':cartItems, 'order':order, 'items':items}
```
## Step 3: User cookieCart in views
To make use of our new cookieCart funciton; let's clear out all of the logic in the "else" condition in "*if request.user.is authenticated.*" and
call our cookieCart function.

We will set the return value to the variable "cookieData" and now we can access the values returned by the function,
```py
['cartitems':cartitems, 'order':order, "items':items]
```
We can then access these values from "cookieData" and get what we need for each view.
```py
from django.shortcuts import render
from django.http import HttpResponse, JsonResponse
import json
import datetime
from django.views.decorators.csrf import csrf_exempt
from numpy import product

from .models import *
from .utils import cookieCart

# Create your views here.

def store(request):
    if request.user.is_authenticated:
        customer = request.user.customer
        order, created = Order.objects.get_or_create(customer=customer, complete=False)
        items = order.orderitem_set.all()
        cartItems = order.get_cart_items
    else:
        cookieData = cookieCart(request)
		cartItems = cookieData['cartItems']

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
		cookieData = cookieCart(request)
		cartItems = cookieData['cartItems']
		order = cookieData['order']
		items = cookieData['items']

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
        cookieData = cookieCart(request)
		cartItems = cookieData['cartItems']
		order = cookieData['order']
		items = cookieData['items']

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
# from django.views.decorators.csrf import csrf_exempt

# @csrf_exempt
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
# Part 5: cartData()
**Overview:**
Our views look much better now that we are building our cookieCart in "utils.py" but we can always make things cleaner.

**Moving all cart logic to utils.py**
```py
def checkout(request):
    if request.user.is_authenticated:
        customer = request.user.customer
        order, created = Order.objects.get_or_create(customer=customer, complete=False)
        items = order.orderitem_set.all()
        cartItems = order.get_cart_items
    else:
        # Create empty cart for now for none-logged in users
        cookieData = cookieCart(request)
		cartItems = cookieData['cartItems']
		order = cookieData['order']
		items = cookieData['items']

    context = {'items':items, 'order':order, 'cartItems':cartItems}
    return render(request, 'store/checkout.html', context)
```
## Step 1: Create Function
In "utils.py", let's create one more function underneath the cookie cart and call it "cartData". Cart data will also take in "request" as a
parameter.

**Create function**
```py
def cartData(request):
    return {}
```
**Import into views.py**
```py
from .utils import cookieCart, cartData
```
## Step 2: Move view logic
```py
def cartData(request):
	if request.user.is_authenticated:
		customer = request.user.customer
		order, created = Order.objects.get_or_create(customer=customer, complete=False)
		items = order.orderitem_set.all()
		cartItems = order.get_cart_items
	else:
		cookieData = cookieCart(request)
		cartItems = cookieData['cartItems']
		order = cookieData['order']
		items = cookieData['items']

	return {'cartItems':cartItems ,'order':order, 'items':items}
```
## Step 3: User cartData() in views
```py
def store(request):
	data = cartData(request)

	cartItems = data['cartItems']
	order = data['order']
	items = data['items']

	products = Product.objects.all()
	context = {'products':products, 'cartItems':cartItems}
	return render(request, 'store/store.html', context)


def cart(request):
	data = cartData(request)

	cartItems = data['cartItems']
	order = data['order']
	items = data['items']

	context = {'items':items, 'order':order, 'cartItems':cartItems}
	return render(request, 'store/cart.html', context)

def checkout(request):
	data = cartData(request)
	
	cartItems = data['cartItems']
	order = data['order']
	items = data['items']

	context = {'items':items, 'order':order, 'cartItems':cartItems}
	return render(request, 'store/checkout.html', context)
```
# Part 6: Checkout
## Step 1: Clear Cart
In the "checkout.html" pages Javascript; let's clear cart when our payment button/form is successfully submitted.

To do this we will set cart to an empty dictionary and update our cookie. We'll want to put this in the fetch calls "promise" so we only
clear it when the data was properly submitted.

Also make sure to add it BEFORE we send the user back to our main page or the cart won't clear.
```js
// templates/store/checkout.html
.then((response) => response.json())
.then((data) => {
    console.log('Success:', data);
    alert('Transaction completed');

    cart = {}
    document.cookie ='cart=' + JSON.stringify(cart) + ";domain;path=/"
    
    window.location.href = "{% url 'store' %}"
})
```
## Step 2: Guest Checkout Logic
It's time to process our guests order when they submit the payment button. Right now we are sending data to our "processOrder" view
but we are only handling logged in users.
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
	else:
		print('User is not logged in')

		print('COOKIES:', request.COOKIES)
		name = data['form']['name']
		email = data['form']['email']

		cookieData = cookieCart(request)
		items = cookieData['items']

		customer, created = Customer.objects.get_or_create(
			email=email,
		)
		customer.name = name
		customer.save()

		order = Order.objects.create(
			customer=customer,
			complete=False,
		)
```
**Create Items**

Let's start by building our cart data using the cookie cart function. We will need to query our items and build a real order now.
```py
order = Order.objects.create(
    customer=customer,
    complete=False,
)

    for item in items:
        product = Product.objects.get(id=item['product']['id'])

        orderItem = OrderItem.objects.create(
            product=product,
            order=order,
            quantity=item['quantity']
        )
```
**Move actions/Fix Indentation**

There are a few lines of code that currently sit inside our "else" statement that need to be moved below our condition.

We need to move everything from "total" to "order.save()" below and outside of our else statement. Because we have created an "order"
for both logged in and guest users we need to make this available for both conditions.

Pay close attention to the indentation of where I put the code block
```py
def processOrder(request):
	transaction_id = datetime.datetime.now().timestamp()
	data = json.loads(request.body)

	if request.user.is_authenticated:
		customer = request.user.customer
		order, created = Order.objects.get_or_create(customer=customer, complete=False)
		

	else:
		print('User is not logged in')

		print('COOKIES:', request.COOKIES)
		name = data['form']['name']
		email = data['form']['email']

		cookieData = cookieCart(request)
		items = cookieData['items']

		customer, created = Customer.objects.get_or_create(
			email=email,
		)
		customer.name = name
		customer.save()

		order = Order.objects.create(
			customer=customer,
			complete=False,
		)

		for item in items:
			product = Product.objects.get(id=item['product']['id'])

			orderItem = OrderItem.objects.create(
				product=product,
				order=order,
				quantity=item['quantity']
			)
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
## Step 3: Create guest checkout function
Like we've done so a couple of times aready, let's clean up this view by creating a function that takes care of creating an order for a
"guest" and "retums" the order.

We'll call this function "guestOrder" and place it in "utils.py". We'll be copying the guest checking logic in here.

**Create function (utils.py)**

The function should take in "data" and "request" as parameters.
```py
def guestOrder(request, data):
	return ''
```

**Copy/paste**

Let's take all the logic inside our "else" condition in the "processOrder" view and paste it in to the new guestOrder function. the function
should return our order.
```py
def guestOrder(request, data):
	name = data['form']['name']
	email = data['form']['email']

	cookieData = cookieCart(request)
	items = cookieData['items']

	customer, created = Customer.objects.get_or_create(
			email=email,
			)
	customer.name = name
	customer.save()

	order = Order.objects.create(
		customer=customer,
		complete=False,
		)

	for item in items:
		product = Product.objects.get(id=item['id'])
		orderItem = OrderItem.objects.create(
			product=product,
			order=order,
			quantity=item['quantity'],
		)
	return customer, order
```
```py
# ecommerce/store/views.py
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
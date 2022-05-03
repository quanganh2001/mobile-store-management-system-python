# Part 1: Model
## Step 1: Import User Model
models.py
```py
from django.db import models
from django.contrib.auth.models import User
```
## Step 2: Customer Model
```py
class Customer(models.Model):
    user = models.OneToOneField(User, null=True, blank=True, on_delete=models.CASCADE)
    name = models.CharField(max_length=200, null=True)
    email = models.CharField(max_length=200, null=True)

    def __str__(self):
        return self.name
```
## Step 3 | "Product", "Order" & "OrderItem" Models
Next we need to create 3 essentials models that make up an order. Just a quick note we will add an image field to the product model in a bit so just ignore it for now
```py
class Product(models.Model):
    name = models.CharField(max_length=200, null=True)
    price = models.FloatField()
    digital = models.BooleanField(default=False, null=True, blank=False)

    def __str__(self):
        return self.name
```
The product model will be comprised of a "name", "price" and "digital". Digital will just be true or false and will let us know if this is a digital product or a physical product that needs to be shipped
```py
class Order(models.Model):
    customer = models.ForeignKey(Customer, on_delete=models.SET_NULL, blank=True, null=True)
    date_ordered = models.DateTimeField(auto_now_add=True)
    complete = models.BooleanField(default=False, null=True, blank=False)
    transaction_id = models.CharField(max_length=200, null=True)

    def __str__(self):
        return self.id
```
As I stated earlier, an order will be the summary of items order and a transaction id. So in order is sort of like the cart and order items will be the items in the cart. The order Item model will be connected to the customer with a one to many relationship (AKA ForeignKey) and will hold the status of complete (True or False) and a transaction id along with the date this order was placed.
```py
class OrderItem(models.Model):
    product = models.ForeignKey(Product, on_delete=models.SET_NULL, blank=True, null=True)
    order = models.ForeignKey(Order, on_delete=models.SET_NULL, blank=True, null=True)
    quantity = models.IntegerField(default=0, null=True)
    date_added = models.DateTimeField(auto_now_add=True)
```
And finally we have the Order item. This model will need a product attribute connected to the product model, the order this item is connected to, quantity and the date this item was added to cart.
## Step 4: Shipping Model
Last we want to add the shipping address model. This model will be a child to order and will only be created if at least one order item within an order is a physical product (If Product.digital -- False).

We will also connect this model to a customer so a customer can reuse the shipping address if needed in the future.
```py
class ShippingAddress(models.Model):
    customer = models.ForeignKey(Customer, on_delete=models.SET_NULL, blank=True, null=True)
    order = models.ForeignKey(Order, on_delete=models.SET_NULL, blank=True, null=True)
    address = models.CharField(max_length=200, null=True)
    city = models.CharField(max_length=200, null=True)
    state = models.CharField(max_length=200, null=True)
    zipcode = models.CharField(max_length=200, null=True)
    date_added = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.address
```
## Step 5: Migrate database
Ok, just to ensure things worked, let's run a migration
```txt
python manage.py makemigrations

python manage.py migrate
```
## Step 6 | admin.py

File store/admin.py
Before we create a user and check our Django admin panel, let's not forget to register our models so we can see them. Open up your admin.py file in your app and import, then register each model individually.
```py
from django.contrib import admin

# Register your models here.

from .models import *

admin.site.register(Customer)
admin.site.register(Product)
admin.site.register(Order)
admin.site.register(OrderItem)
admin.site.register(ShippingAddress)
```
### Create User
Now we can create a user and login to make sure all our models were properly registered.
```txt
python manage.py createsuperuser
```
### Create Products
Go ahead and create a few products from the admin panel.
![image.png](https://img.upanh.tv/2022/05/01/image.png)
![image76299c29ed62e25b.png](https://img.upanh.tv/2022/05/01/image76299c29ed62e25b.png)
# Part 2: Render Products
## Step 1: Query Products
In our apps views.py file, let's first import all our models and then query the products within our store view. Don't forget to add "products" into the context dictionary.

File store/views.py
```py
from .models import *
```
## Step 2: Render Products
### 1 - Loop Through Products
In store.html, let's remove 3 of the 4 columns we had and create a for loop around the one remaining column.

Just inside our row and above the column div start the for loop {% for product in products %) and close the loop just underneath the closing tag of the column div but still inside the row (% endfor %]
### 2 - Replace Fields
- Product Price
- Product Name

Now inside the column, replace the filler data with the product data. We'll add the `floatformat` tag to our price so we don't get any funny values.

File store/templates/store/store.html
```html
{% extends 'store/main.html' %}
{% load static %}
{% block content %}
    <div class="row">
        {% for product in products %}
        <div class="col-lg-4">
            <img class="thumbnail" src="{% static 'images/placeholder.png' %}">
            <div class="box-element product">
                <h6><strong>{{product.name}}</strong></h6>
                <hr>
                <button class="btn btn-outline-secondary add-btn">Add to Cart</button>
                <a class="btn btn-outline-success" href="#">View</a>
                <h4 style="display: inline-block;float: right">${{product.price|floatformat:2}}</h4>
            </div>
        </div>
        {% endfor %}
    </div>
{% endblock content %}
```
The output is:
![imagea22edcf9c7f72619.png](https://img.upanh.tv/2022/05/01/imagea22edcf9c7f72619.png)
# Part 3: Product Image Field
## Step 1 : `ImageField()`
Ok, so now it's time to add the image field to our model.

I left this part out earlier because the image field has its own configuration we need to set up so this way we can focus on it in a single part.

Before we get started I will link up six product images for you to use if you're following my setup. Click each image individually and download them somewhere you can easily access them. DON'T add them to your static files, this will be done automatically with the imagefield when we upload them.
### 1 - Add ImageField()
In your models.py add "image" to your "Product" model
```py
class Product(models.Model):
    name = models.CharField(max_length=200, null=True)
    price = models.FloatField()
    digital = models.BooleanField(default=False, null=True, blank=False)
    image = models.ImageField(null=True, blank=True)

    def __str__(self):
        return self.name
```
### 2 - pip install Pillow
Before we run the migration, if you save the file and check your command prompt you will see this error appear telling us we need to install "Pillow
 
Pillow is the Python Imaging Library that basically let's us add the imagefield to our model You can see the documentation in the link I provided.
### 3 - Run Migrations
Now you can run migrations to add the new field
```txt
Python manage.py makemigrations

Python manage.py migrate
```
You should now see the image field in your admin panel. There is still some more configuration we need to do so don't try to upload any images just yet.
## Step 2: MEDIA_ROOT
At this point we have not configured a file for the images to be uploaded to. Let's take care of this so that anytime an image is uploaded it goes into our images folder inside of our static files.

In settings.py just underneath STATICFILES_DIRS let's set the MEDIA_ROOT. Media root will set a path for all media files to be uploaded to.

Go ahead and copy my code just like in the image or code block in file ecommerce/ecommerce/settings.py
```py
MEDIA_ROOT = os.path.join(BASE_DIR, 'static/images')
```
We use os.path.join(BASE_DIR) to set the file path to the root directory, then we add static/media. This gives us a direct path to the images folder for images to be uploaded to.

Add Images To Products In Admin Panel

Now if you go back the the admin panel and upload an image to the image field it should appear in the images folder.

Go ahead and upload an image to each product and we'll take care of dynamically rendering these in the next step.
![image727062618aee85d7.png](https://img.upanh.tv/2022/05/01/image727062618aee85d7.png)
## Step 3: MEDIA_URL
Right now our images get uploaded to the right folder but we still need to configure a url path to find these images so they can be rendered properly.

Just above the MEDIA_ROOT variable in settings.py let's add MEDIA_URL.

Set MEDIA_URL to "images/" to point this to our images folder.
```py
MEDIA_URL = '/images/'
```
## Step 4: Urls.py Configuration
In your root urls.py file let's first import static and settings like the image below
### 1 - Imports
```py
from django.contrib import admin
from django.urls import path, include

from django.conf.urls.static import static
from django.conf import settings

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('store.urls'))
]

urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```
## Step 5: Render Images
Now it's time to actually render these images from our models.
Traditionally we could just go to our store.html template and replace the file path in our placeholder image to our image in store.html. Like so:
```html
<img class="thumbnail" src="{{product.image.url}}">
```
While this may work and would render out our images, we would face an error if one of our products didn't hold an image. You can test this by replacing our placeholder image in store.html, view the page to ensure the images get rendered, and then removing an image to one of our products.

This is the error you will see when loading the page.
![image40ad67827bf6370a.png](https://img.upanh.tv/2022/05/01/image40ad67827bf6370a.png)
We will add a work around to this issue in the next step
## Step 6: Image Error Solution
Instead of calling the image url directly like we did in our template (product.image.url) we want to use a model method to either render an image or an empty string so we don't get an error on the front end.
### 1. Add imageURL Method
In models.py, let's add a model method to the product attribute and call it "imageURL". We will add the @property decorator so we can access it like an attribute.

Inside the method all we will do is use a try/except block to see if the instance has an image, if not we will just return an empty string.
```py
class Product(models.Model):
    name = models.CharField(max_length=200, null=True)
    price = models.FloatField()
    digital = models.BooleanField(default=False, null=True, blank=False)
    image = models.ImageField(null=True, blank=True)

    def __str__(self):
        return self.name
    
    @property
    def imageURL(self):
        try:
            url = self.image.url
        except:
            url = ''
        return url
```
# Part 4 | User Cart
**Overview**:
To close out this module let's manually add in some user data in the admin panel and render it in the cart and checkout page.

We'll attach a customer to our user, and add some order items to go with the order which we will manually create from the admin panel.

By the end of this part our cart and checkout page will render orders/items the logged in user has in his or her cart.
## Step 1: Add data (Admin Panel)
Let's start by logging into our admin panel and creating some data
### 1. Create Customer
First, let's create customer instance and connect to the current logged in user
![image97ed5d0e1f122fb1.png](https://img.upanh.tv/2022/05/01/image97ed5d0e1f122fb1.png)
### 2. Create Order
Create an order and attach to your new customer. For now just manually set the transaction id to some random characters.
![imageaa7a99f76012db68.png](https://img.upanh.tv/2022/05/01/imageaa7a99f76012db68.png)
### 3. Order Items
Finally, go ahead and add a few order items and attach to the order we just created
![imaged6d48b32a89a7b72.png](https://img.upanh.tv/2022/05/01/imaged6d48b32a89a7b72.png)
## Step 2: Query Data (Cart)
In order to load pages correctly we will need to create two different scenarios in our views. One for an authenticated user and one for a simple page visitor.

We want both logged in and not logged in users to be able to view the pages but also not break the page when we can't find a registered user associated with the page visitor.

**Query User Cart**

In our cart view we want to start by finding the logged in users account, order and cart items.
Let's start by first checking if the user is authenticated, and then getting the customer along with finding or creating and order using the `get_or_create()` method.

[get_or_create() documentation link](https://docs.djangoproject.com/en/4.0/ref/models/querysets/)

Once we have the customer and order/cart let's query the cart items with `order.orderitem_set.all()`. Make sure to add in the items in to the context dictionary so we can use them in our page.
```py
def cart(request):

    if request.user.is_authenticated:
        customer = request.user.customer
        order, created = Order.objects.get_or_create(customer=customer, complete=False)
        items = order.orderitem_set.all()
    else:
        items = []
        
    context = {'items':items}
    return render(request, 'store/cart.html', context)
```
## Step 3: Render data (cart.html)
In the last step we queried the cart items so now let's render some data out in our template.

Let's create a for loop around our cart row (below header row) and render a row for each item in our cart. For now replace the filler data with the cart item data. We'll leave the total alone for now.

**Replace:**
- ImageURL
- Product Name
- Price
- Quantity

```html
            <div class="box-element">
                <div class="cart-row">
                    <div style="flex: 2"></div>
                    <div style="flex: 2"><strong>Item</strong></div>
                    <div style="flex: 1"><strong>Price</strong></div>
                    <div style="flex: 1"><strong>Quantity</strong></div>
                    <div style="flex: 1"><strong>Total</strong></div>
                </div>

                {% for item in items %}
                <div class="cart-row">
                    <div style="flex: 2"><img class="row-image" src="{{item.product.imageURL}}"></div>
                    <div style="flex: 2">{{item.product.name}}</div>
                    <div style="flex: 1">${{item.product.price|floatformat:2}}</div>
                    <div style="flex: 1">
                        <p class="quantity">{{item.quantity}}</p>
                        <div class="quantity">
                            <img src="{% static 'images/arrow-up.png' %}" class="chg-quantity">

                            <img src="{% static 'images/arrow-down.png' %}" class="chg-quantity">
                        </div>
                    </div>
                    <div style="flex: 1"><p>${{item.get_total}}</p></div>
                </div>
                {% endfor %}
            </div>
```
The output is:
![imaged1cf94e54c4b4317.png](https://img.upanh.tv/2022/05/01/imaged1cf94e54c4b4317.png)
## Step 4: Calculating Totals
If you haven't already noticed we are still lacking some data. Each *item total*, *cart total* along with the *item totals* are also static

We can fix this by adding some model methods just like we did with the product url.

For the Order item model we want to derive the total price from the product price multiplied by the quantity.
```py
class OrderItem(models.Model):
    product = models.ForeignKey(Product, on_delete=models.SET_NULL, blank=True, null=True)
    order = models.ForeignKey(Order, on_delete=models.SET_NULL, blank=True, null=True)
    quantity = models.IntegerField(default=0, null=True)
    date_added = models.DateTimeField(auto_now_add=True)


    @property
    def get_total(self):
        total = self.product.price * self.quantity
        return total
```
For the order model in models.py we will add a `get_cart_total` and `get_cart_items`. Let's use the `@property` decorator so we can access these like attributes
```py
class Order(models.Model):
    customer = models.ForeignKey(Customer, on_delete=models.SET_NULL, blank=True, null=True)
    date_ordered = models.DateTimeField(auto_now_add=True)
    complete = models.BooleanField(default=False, null=True, blank=False)
    transaction_id = models.CharField(max_length=200, null=True)

    def __str__(self):
        return str(self.id)
    
    @property
    def get_cart_total(self):
        orderitems = self.orderitem_set.all()
        total = sum([item.get_total for item in orderitems])
        return total
    
    @property
    def get_cart_items(self):
        orderitems = self.orderitem_set.all()
        total = sum([item.quantity for item in orderitems])
        return total
```
## Step 5: Query and Render Totals
```py
from django.shortcuts import render
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
    context = {}
    return render(request, 'store/checkout.html', context)
```
Now in cart.html we can output the totals, in the image below I will highlight the areas we can update with our new data.
```html
                <table class="table">
                    <tr>
                        <th><h5>Items: <strong>{{order.get_cart_items}}</strong></h5></th>
                        <th><h5>Total: <strong>${{order.get_cart_total|floatformat:2}}</strong></h5></th>
                        <th>
                            <a style="float: right; margin: 5px" class="btn btn-success" href="{% url 'checkout' %}">Checkout</a>
                        </th>
                    </tr>
                </table>
```
Now we should see real totals in our cart!
![image732d62e2198557da.png](https://img.upanh.tv/2022/05/01/image732d62e2198557da.png)
## Step 7: Checkout Page Data
Last thing we want to do in this module is render this same data in our checkout page.
### 1 - Copy cart view logic into checkout view
We'll clean things up later but for now let's just copy and paste what we have in our cart view and add it to the checkout view. Make sure to get everything in between the if statement to the context dictionary.
```py
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
```
### 2- Render cart data in checkout.html
Next let's just create a loop over our order summary rows and add in item data along with our cart totals just like we did in the cart page
```html
        <div class="col-lg-6">
            <div class="box-element">
                <a class="btn btn-outline-dark" href="{% url 'cart' %}">&#x2190; Back to Cart</a>
                <hr>
                <h3>Order Summary</h3>
                <hr>
                {% for item in items %}
                <div class="cart-row">
                    <div style="flex:2"><img class="row-image" src="{{item.product.imageURL}}"></div>
                    <div style="flex:2"><p>{{item.product.name}}</p></div>
                    <div style="flex:1"><p>${{item.product.price}}</p></div>
                    <div style="flex:1"><p>x2{{item.quantity}}</p></div>
                </div>
                {% endfor %}

                <h5>Items: {{order.get_cart_items}}</h5>
                <h5>Total: ${{order.get_cart_total|floatformat:2}}</h5>
            </div>
        </div>
```
Your page should now look something like this.
![image29771ac1eda6d20d.png](https://img.upanh.tv/2022/05/01/image29771ac1eda6d20d.png)
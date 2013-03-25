##Order Integration##

*Feature:* JSON-based RESTful API for Retailers
*Aim:* To give the retailers easy way to control their sales and inventory

*Resources to be available via API:*

* orders, filtered by retailer (managing statuses) (example.com/orders)
* products (stock level)
* refunds
* deliveries
* comments(*)

(*) is available as nested resource for orders, refunds and deliveries

*URL build rules*

HTTP method + base url + resource url + [filters]

*Demo instances HOW-TO*

*Working with API (general)*

Request body (if exists) should be a valid JSON with the "Content-type: application/json" header set.

*Authentication*

Authenticate retailers by their credentials at the Magento backend.

API accepts credentials to be sent in various forms:
*JSON:*
<pre>
{"username":"retailer_name","password":"retailer_password"}
</pre>
with the "Content-type: application/json" header set.
*As a querystring:*
<pre>
username=retailer_name&password=retailer_password
</pre>
with the "Content-Type: application/x-www-form-urlencoded" header.

<pre>
POST /auth/ {
  username: "retailer_name",
	password: "retailer_password"
}
</pre>
Response:
<pre>
HTTP/1.1 201 Created
Set-Cookie: PHPSESSID=6d8552f81fbcc7f64de90d563807f5ee
</pre>
Which means that the cookie is set and it should be sent over with every API request.


*Filtering collections*

Filtering the result collections is done using querystrings:
<pre>
GET /orders/?increment_id=10002455
GET /orders/?status=pending
GET /returns/?from=2013-01-01%2000:00:00&to=2013-01-15%2000:00:00
</pre>


You can use sets of values in filters. For example, if you want to get all orders with statuses "pending" and "processing", the querystring will look like
<pre>GET /orders/?status=pending,processing</pre>

See the list of available filters for all resource types in appropriate sections.


##API Responses##

Retailers API always responds with the meaningful HTTP status - e.g. 200 for successful requests, 404 for the requests not matching the filter, 400 for wrong requests etc.

*Response for wrong request*

When the API cannot perform the request, it responds with the error code and the error description(s):
<pre>
HTTP/1.1 400 Bad Request
<code class="javacript">
{
	error_code: 123,
	message: "Wrong order status supplied"
}
</code>
</pre>


*Working with collections*

You can use pagination to walk through the collection.

| *parameter* | *descroption* |
|-------------|---------------|
| page_size   | amount of items in response (1...100) |
| page   	  | current page (0...+inf) |

<pre>
GET /orders/?page_size=10&page=2</pre>
returns the items from 11 to 20

*Working with orders*

*Available information*

* ID
* Increment ID
* Date of placement
* Status: pending, pending_payment, cancelled, complete, etc.
* Total
* Shipping information
* Customer email
* Items ordered

*Available filters*

* Order ID (GET /orders/?id=1074) - returns order by ID
* Increment ID (GET /orders/?increment_id=1000045678) - returns order by Increment ID
* Status (GET /orders/?status=pending) - orders with "Pending status" (use comma to separate different values)
* Date: from (YYYY-MM-DD[ HH:MM:SS]) (time part is optional) (GET /orders/?from=2013-01-28) - returns orders placed from given date
* Date: to (YYYY-MM-DD[ HH:MM:SS]) (time part is optional) (GET /orders/?to=2013-01-29) - returns orders placed to given date
* Date combined (GET /orders/?from=2013-01-28&to=2013-01-29) - returns orders for the given interval

*Orders list:*

<pre>GET /orders/</pre>
returns:
<pre>
HTTP/1.1 200 OK
<code class="javascript">
{
	count: 102,
	page: 0,
	items: [
		{
			id: 1074,
			increment_id: "100000546",
			created_at: "2013-01-14 12:30:45",
			status: "pending",
			shipping: {
				type: "Retailer Delivery - You have qualified for FREE delivery",
				first_name: "John",
				last_name: "Doe",
				country: "UK",
				city: "London",
				street_address: "Baker st., 221B",
				post_code: "PO12 4EP"
			},
			items: [
				{
					item_id: 3451,
					product_id: 1074,
					sku: "HHR7-WHITE-8",
					price: 80.90,
					size: 8,
					color: "white",
					qty_ordered: "1.000",
					qty_canceled: "0.000",
					qty_invoiced: "0.000",
					qty_refunded: "0.000",
					qty_shipped: "0.000"
				}
			]
		},
		{
			id: 1098,
			created_at: "2013-01-16 16:21:31",
			status: "pending_payment",
			shipping: {
				type: "Retailer Delivery - You have qualified for FREE delivery",
				first_name: "Lisa",
				last_name: "Simpson",
				country: "US",
				city: "Springfield",
				street_address: "742 Evergreen Terrace",
				post_code: "123"
			},
			items: [
				{
					item_id: 3516,
					product_id: 1255,
					sku: "DF887-YA-8",
					price: 120.00,
					size: 8,
					color: "black",
					qty_ordered: "2.000",
					qty_canceled: "0.000",
					qty_invoiced: "0.000",
					qty_refunded: "0.000",
					qty_shipped: "0.000"
				},
				{
					item_id: 3517,
					product_id: 1174,
					sku: "66R7-WHITE-8",
					price: 49.00,
					size: 8,
					color: "white",
					qty_ordered: "1.000",
					qty_canceled: "0.000",
					qty_invoiced: "0.000",
					qty_refunded: "0.000",
					qty_shipped: "0.000"
				}
			]
		}
	]
}
</code>
</pre>
Items are configurable products, if any. Use item_id to refer to an item when creating the refund or delivery.


*Editing shipping address for the order*

You can make changes to the shipping address if the order doesn't contain items of different retailers.
Example:
<pre>
PUT /orders/820 
<code class="javascript">
{
    shipping: {
        first_name: "John",
        last_name: "Doe",
        city: "Manchester",
        postcode: "M1 ONE",
        street_address: "Piccadilly, 14"
    }
}
</code>
</pre>
Returns:
<pre>
HTTP/1.1 200 OK
<code class="javascript">
{
	id: 820,
	created_at: "2013-01-16 16:21:31",
	status: "pending",
	shipping: {
		type: "Retailer Delivery - You have qualified for FREE delivery",
		first_name: "John",
		last_name: "Doe",
		country: "UK",
		city: "Manchester",
		street_address: "Piccadilly, 14",
		post_code: "M1 ONE"
	},
	items: [
		{
			item_id: 3516,
			product_id: 1255,
			sku: "DF887-YA-8",
			price: 120.00,
			size: 8,
			color: "black",
			qty_ordered: "2.000",
			qty_canceled: "0.000",
			qty_invoiced: "0.000",
			qty_refunded: "0.000",
			qty_shipped: "0.000"
		}
	]
}
</code></pre>

##Refunds##

*Available filters*

* ID
* order_id
* Date: from (YYYY-MM-DD[ HH:MM:SS]) (time part is optional) (GET /refunds/?from=2013-01-28) - returns refunds created starting from given date
* Date: to (YYYY-MM-DD[ HH:MM:SS]) (time part is optional) (GET /refunds/?to=2013-01-29) - returns refunds created before given date
* Date combined (GET /refunds/?from=2013-01-28&to=2013-01-29) - returns refunds for the given interval

All refunds so far:
<pre>GET /refunds/</pre>
Returns:
<pre>
HTTP/1.1 200 OK
<code class="javascript">
{
	count: 2,
	page: 0,
	items: [
		{
			id: 210,
			order_id: 1098,
			created_at: "2013-01-01 12:11:11",
			items: [
				{
					item_id: 3516, // the item id within the order
					qty: "1.000",	// QTY to refund
					refund_reason: "Out of stock"
				}
			]
		},
		{
			id: 211,
			order_id: 1140,
			items: [
				{
					item_id: 3676, // the item id within the order
					qty: "1.000",	// QTY to refund
					refund_reason: "Out of stock"
				}
			]
		}
	]
}
</code></pre>

Refunds filtered:
<pre>
GET /refunds/?order_id=820
GET /refunds/?from=2013-01-01&to=2013-02-01
</pre>


Items refund:

|*Refund reasons*|
|----------------|
|'Out of stock'|
|'Returned: Wrong Size'|
|'Returned: Damaged'|
|'Returned: Differed from description'|
|'Returned: Delivered too late'|
|'Returned: Wrong Item shipped'|
|'Other'|

<pre>
POST /refunds/ {
	order_id: 1098,
	comment: "Some comment",
	items: [
		{
			item_id: 3516, // the item id within the order
			qty: 1,	// QTY to refund
			refund_reason: "out of stock"
		}
	]
}
</pre>
Returns:
<pre>
HTTP/1.1 201 Created
<code class="javascript">
{id: 210}
</code></pre>


##Deliveries##

*Available filters*

* ID
* order_id
* increment_id
* Date: from (YYYY-MM-DD[ HH:MM:SS]) (time part is optional) (GET /deliveries/?from=2013-01-28) - returns deliveries created starting from given date
* Date: to (YYYY-MM-DD[ HH:MM:SS]) (time part is optional) (GET /deliveries/?to=2013-01-29) - returns deliveries created before given date
* Date combined (GET /deliveries/?from=2013-01-28&to=2013-01-29) - returns deliveries for the given interval

All deliveries:
<pre>GET /deliveries/</pre>
Returns:
<pre>
HTTP/1.1 200 OK
<code class="javascript">
{
	count: 6,
	page: 1,
	items: [
		{
			id: 450,
			order_id: 1079,
			increment_id: 10000079,
			created_at: "2012-12-13 12:40:41",
			updated_at: "2013-01-18 13:25:15",
			shipping_address: {
				first_name: "John",
				last_name: "Doe",
				country: "UK",
				city: "London",
				street_address: "Baker st., 221B",
				post_code: "PO12 4EP"
			},
			tracks: [
				{
					track_id: 314,
					track_number: "FEDEX-3434-DF",
					title: "Federal Express",
					carrier_code: "fedex",
					created_at: "2013-01-18 13:20:28"
				}
			]
		}
	]
}
</code>
</pre>

Delivery with the given ID:
<pre>GET /deliveries/450</pre>
Returns:
<pre>
HTTP/1.1 200 OK
<code class="javascript">
{
	id: 450,
	order_id: 1079,
	increment_id: 10000079,
	created_at: "2012-12-13 12:40:41",
	updated_at: "2013-01-18 13:25:15",
	shipping_address: {
		first_name: "John",
		last_name: "Doe",
		country: "UK",
		city: "London",
		street_address: "Baker st., 221B",
		post_code: "PO12 4EP"
	},
	tracks: [
		{
			track_id: 314,
			track_number: "FEDEX-3434-DF",
			title: "Federal Express",
			carrier_code: "fedex",
			created_at: "2013-01-18 13:20:28"
		}
	]
}
</code>
</pre>


Creating delivery:
<pre>
POST /deliveries/ {
	order_id: 1098,
	items: [
		{
			item_id: 3517, // the item id within the order
			qty: 1 // the quantity of items to deliver
		}
	]
}
</pre>
Returns:
<pre>
HTTP/1.1 201 Created
<code class="javascript">
{id: 561}
</code></pre>

Adding/removing tracking information:
<pre>
PUT /deliveries/561 <code class="javascript">{
	add_tracks: [
		{
			track_number: "FEDEX-34AA-DF",
			title: "Federal Express",
			carrier_code: "fedex"
		}
	],
	remove_tracks: [314] // the array of tracking number IDs
}
</code>
</pre>
Returns:
<pre>
HTTP/1.1 200 OK
</pre>
And the appropriate delivery as if it was requested with GET /deliveries/561.


##Working with products##

*Available information*

* ID
* SKU
* Name (write access)
* Description (write access)
* Short description (write access)
* Status (enabled/disabled) (write access)
* Visibility (Not visible individually/Catalog, Search/Catalog) (write access)
* Price
* Special price
* Stock status (write access)
* Product type
* Child products: size/colour (write access with the same attributes)

*Available filters*

* ID
* SKU
* Name
* Status (enabled/disabled)
* Visibility (Not visible individually/Catalog, Search/Catalog)
* Price
* Special price
* Stock status
* Product type

*Products list*

<pre>GET /products/</pre>
returns:
<pre>
HTTP/1.1 200 OK
<code class="javascript">
{
	count: 951,
	page: 0,
	items: [
		{
			id: 1245,
			sku: "ER567-UG",
			visibility: "Catalog, Search",
			status: "Enabled",
			type: "configurable",
			name: "Product 1",
			description: "Just a product",
			short_description: "product",
			price: 125.90,
			special_price: 100.00,
			special_from: "2013-01-01 00:00:00",
			special_to: "2013-01-02 00:00:00",
			children: [
				{
					id: 1246,
					sku: "ER567-UG-Black-10",
					type: "simple",
					status: "Enabled",
					color: "Black",
					size: 10,
					stock_item: {
						qty: 12,
						is_in_stock: true
					}
				},
				{
					id: 1247,
					sku: "ER567-UG-White",
					type: "simple",
					status: "Enabled",
					color: "White",
					size: 12,
					stock_item: {
						qty: 4,
						is_in_stock: true
					}
				}
			]
		}
	}
]
</code>
</pre>

*Getting the product by sku or ID:*

<pre>
GET /products/?sku=SKU3333
GET /products/?id=254778
</pre>
Returns:
<pre>
HTTP/1.1 200 OK
</pre>
Response body has the same format as in Products list except that the filter returns array of products

*Editing Products:*

Change the name and description for the product #1245:
<pre>
PUT /products/1245 {
	name: "new Product name",
	description: "new description"
}
</pre>
Returns:
<pre>HTTP/1.1 200 OK
<code class="javascript">
{
	id: 1245,
	sku: "ER567-UG",
	visibility: "Catalog, Search",
	status: "Enabled",
	type: "configurable",
	name: "new Product name",
	description: "new description",
	short_description: "product",
	price: 125.90,
	special_price: 100.00,
	special_from: "2013-01-01 00:00:00",
	special_to: "2013-01-02 00:00:00",
	children: [
		{
			id: 1246,
			sku: "ER567-UG-Black-10",
			type: "simple",
			status: "Enabled",
			color: "Black",
			size: 10,
			stock_item: {
				qty: 12,
				is_in_stock: true
			}
		},
		{
			id: 1247,
			sku: "ER567-UG-White",
			type: "simple",
			status: "Enabled",
			color: "White",
			size: 12,
			stock_item: {
				qty: 4,
				is_in_stock: true
			}
		}
	]
}
</code>
</pre>

*Disabling the product:*

Disable the product #1245:
<pre>PUT /products/1245 {
	status: "disabled"
}</pre>
Returns:
<pre>HTTP/1.1 200 OK
<code class="javascript">
{
	id: 1245,
	sku: "ER567-UG",
	visibility: "Catalog, Search",
	status: "Disabled",
	type: "configurable",
	name: "Product name",
	description: "Description",
	short_description: "product",
	price: 125.90,
	special_price: 100.00,
	special_from: "2013-01-01 00:00:00",
	special_to: "2013-01-02 00:00:00",
	children: [
		{
			id: 1246,
			sku: "ER567-UG-Black-10",
			type: "simple",
			status: "Enabled",
			color: "Black",
			size: 10,
			stock_item: {
				qty: 12,
				is_in_stock: true
			}
		},
		{
			id: 1247,
			sku: "ER567-UG-White",
			type: "simple",
			status: "Enabled",
			color: "White",
			size: 12,
			stock_item: {
				qty: 4,
				is_in_stock: true
			}
		}
	]
}
</code>
</pre>

*Visibility management:*

Exclude the product #1245 from search:
<pre>
PUT /products/1245 {
	visibility: "Catalog"
}
</pre>
Returns:
<pre>HTTP/1.1 200 OK</pre>
And appropriate saved product as if it was requested with GET /products/1245.

Available visibilities: 
"Not visible individually", "Catalog", "Catalog, Search", "Search"

Making product #1247 out of stock:
<pre>
PUT /products/1247 {
	stock_item: {
		is_in_stock: false
	}
}
</pre>
Returns:
<pre>HTTP/1.1 200 OK</pre>
And appropriate saved product as if it was requested with GET /products/1247.


##Working with comments##

Comments are implemented as a nested resource for orders, deliveries and refunds.

*Available information*

* ID
* Message
* Customer notified flag: whether the customer is notified or not
* History flag: whether the comment is visible in customer's Account or not

*Available filters*

Filters are not available for this resource


*Comments list*

URL to fetch comments is being built with the resource, its ID and the "comments" postfix:
<pre>GET /{resource_in_plural}/{id}/comments</pre>

| resource_in_plural |
|--------------------|
| orders |
| deliveries |
| refunds |

For example, to get the list of the comments for the order #820:
<pre>GET /orders/820/comments</pre>
returns:
<pre>
HTTP/1.1 200 OK
<code class="javascript">
{
	count: 2,
	page: 0,
	items: [
		{
			id: 3331,
			message: "Sorry, your order will be dispatched within the next 2 days",
			customer_notified: true,
			history: true
		},
		{
			id: 3332,
			message: "Your tracking number is FEDEX-XX-YYYY",
			customer_notified: true,
			history: true
		}
	]
}
</code>
</pre>

h3. Adding comment to the resource
<pre>
POST /deliveries/575/comments {
	message: "Sorry, your order will be dispatched within the next 2 days",
	notify: true,
	history: true
}
</pre>
Returns:
<pre>
HTTP/1.1 201 Created
<code class="javascript">
{id: 3331}
</code></pre>


##Error codes##

The error code has the following format:
XYZ, where
X - a digit representing the resource:

| *digit* | *resource* |
|---------|------------|
|       | core     		|
| 1     | order    		|
| 2     | product  		|
| 3     | refund   		|
| 4     | delivery 		|
| 5     | resource-wide|
| 6     | comments|

Y - error type
| *digit* | *description* | *HTTP code* |
|---------|---------------|-------------|
| 1     | validation error       | |
| 2     | valid but unacceptable | |
|  11 | Unauthorised access - please authentificate first | You should POST your credentials to /auth first   			   | 401 |
|  12 | Wrong username/password combination				  | Please check your credentials				      			   | 400 |
| 111 | Invalid request 				  | Shipping address is either not set or has wrong format 						   | 400 |
| 112 | Empty request 				  | Shipping is required 						   | 400 |
| 120 | Unable to save the order | Cannot save shipping. Please contact Intu for help  						   | 400 |
| 121 | You are not allowed to edit this order | Order contains other retailer's items  						   | 400 |
| 210 | Empty request 									  | No data during request, or broken in json structure 		   | 400 |
| 211 | Validation error at: `field_name` 				  | Incorrect data to the field provided 						   | 400 |
| 217 | Invalid request 								  | The field required to update is not exist or allowed 		   | 400 |
| 311 | Wrong data format								  | order_id or items are required to create a refund			   | 400 | <- duplicates 414
| 312 | Cannot create a credit memo for this order 		  | Order is either closed, canceled, not paid, already refunded or the payment is in the "Payment review" status | 400 |
| 314 | Unknown refund reason (_reason_) 				  | Provided refund reason is not valid (see "Refund reasons" section) | 400 |
| 413 | Missing data									  | order_id or items are required to create delivery 			   | 400 |
| 414 | Wrong data format								  | order_id should be a numeric and items - an array 			   | 400 |
| 417 | Invalid request 								  | The request should contain either add_tracks or remove_tracks  | 400 |
| 418 | Tracking number ({id}) not found				  | Check the track IDs for existence							   | 400 |
| 421 | Order not found									  | The supplied order_id does not exist						   | 400 |
| 422 | Cannot do shipment for the order 				  | Cannot do shipment for the order 				  			   | 400 |
| 423 | *Message can vary* 				  | Cannot save the shipment for the order 				  			   | 400 |
| 511 | The ID should be a non-negative integer			  | The supplied ID should be integer   			  			   | 400 |
| 512 | Page size should be a non-negative integer		  | Page size should be a non-negative integer					   | 400 |
| 513 | Page should be a non-negative integer			  | Page should be a non-negative integer						   | 400 |
| 520 | Item ({id}) doesn't exist in the current order | Check the item IDs for existence in this particular order | 400 |
| 611 | Invalid request. Message is required | Message could not be empty | 400 |
| 621 | Unable to fetch comments | Please contact Intu for help | 400 |
| 622 | Unable to add a comment for the order | Please contact Intu for help | 400 |
| 623 | Unable to add a comment for the order | Please contact Intu for help | 400 |

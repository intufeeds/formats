# Order Integration

  * Feature:* JSON-based RESTful API for Retailers
  * Aim:* To give the retailers easy way to control their sales and inventory

## Resources to be available via API:

  * orders, filtered by retailer (managing statuses) (example.com/orders)
  * products (stock level)
  * refunds
  * deliveries

## Integration

The API is created as a magento module hence having the full access to Magento
ORM. RESTfulness is archieved via Zend_Rest_Controller and Zend_Rest_Route.

## URL build rules

HTTP method + base url + resource url + [filters]

## Authentification

Authenticate retailers by their credentials at the Magento backend.

<pre> POST /auth/ { login: 'retailer_name', password: 'retailer_password' }
</pre> The cookie is being set and it should be sent over with every API
request.

## Filtering collections

Filtering the result collections is done using querystrings: <pre> GET
/orders/?incid=10002455 GET /orders/?status=pending GET
/returns/?from=2013-01-01&to=2013-01-15 </pre>

You can use filters when editing resources: <pre>PUT /orders/?id=1047 {status:
"canceled"}</pre>

You can use sets of values in filters. For example, if you want to get all
orders with statuses "pending" and "processing", the querystring will look
like <pre>GET /orders/?status=pending,processing</pre>

See the list of available filters for orders and products in appropriate
sections.

## API Responses

Retailers API always responds with the meaningful HTTP status - e.g. 200 for
successful requests, 404 for the requests not matching the filter, 400 for
wrong requests etc.

  * Response for wrong request*

When the API cannot perform the request, it responds with the error flag and
the error description(s): <pre> Status: 400 <code class="javacript"> { error:
true, message: 'Wrong order status supplied' } </code> </pre>

Or:

<pre> Status: 400 <code class="javascript"> { error: true, message: ['Wrong
order status supplied', 'Quantity could not be negative'] } </code> </pre>

## Working with orders

  * Available information*
  * ID
  * Increment ID
  * Date of placement
  * Status: pending, pending_payment, cancelled, complete, etc.
  * Total
  * Shipping information
  * Customer email
  * Items ordered
  * Available filters*
  * Order ID (GET /orders/?id=1074) - returns order by ID
  * Increment ID (GET /orders/?incid=1000045678) - returns order by Increment ID
  * Status (GET /orders/?status=pending) - orders with "Pending status" (use comma to separate different values)
  * Date: from (YYYY-MM-DD) (GET /orders/?from=2013-01-28) - returns orders placed from given date
  * Date: to (YYYY-MM-DD) (GET /orders/?to=2013-01-29) - returns orders placed to given date
  * Date combined (GET /orders/?from=2013-01-28&to=2013-01-29) - returns orders for the given interval

### Orders list:

<pre>GET /orders/</pre> returns: <pre> Status: 200 <code class="javascript"> [
{ id: 1074, increment_id: '100000546', created_at: '2013-01-14 12:30:45',
status: 'pending', shipping: { type: 'Retailer Delivery - You have qualified
for FREE delivery', first_name: 'John', last_name: 'Doe', country: 'UK', city:
'London', street_address: 'Baker st., 221B', post_code: 'PO12 4EP', phone:
'+441231234567' }, items: [ { id: 1074, sku: 'HHR7-WHITE-8', price: 80.90,
size: 8, color: 'white', qty_ordered: 1, qty_backordered: 0, qty_canceled: 0,
qty_invoiced: 0, qty_refunded: 0, qty_shipped: 0 } ] }, { id: 1098,
created_at: '2013-01-16 16:21:31', status: 'pending_payment', shipping: {
type: 'Retailer Delivery - You have qualified for FREE delivery', first_name:
'Lisa', last_name: 'Simpson', country: 'US', city: 'Springfield',
street_address: '742 Evergreen Terrace', post_code: '123', phone:
'+11341234567' }, items: [ { item_id: 3516, product_id: 1255, sku:
'DF887-YA-8', price: 120.00, size: 8, color: 'black', qty_ordered: 2,
qty_backordered: 0, qty_canceled: 0, qty_invoiced: 0, qty_refunded: 0,
qty_shipped: 0 }, { item_id: 3517, product_id: 1174, sku: '66R7-WHITE-8',
price: 49.00, size: 8, color: 'white', qty_ordered: 1, qty_backordered: 0,
qty_canceled: 0, qty_invoiced: 0, qty_refunded: 0, qty_shipped: 0 } ] } ]
</code> </pre>

### Changing the status:

<pre> PUT /orders/?id=1047 { status: "shipped" } </pre> Returns: <pre> Status:
200 </pre>

### Refunds

  * Available filters*
  * ID
  * order_id
  * Date: from (YYYY-MM-DD) (GET /refunds/?from=2013-01-28) - returns refunds created starting from given date
  * Date: to (YYYY-MM-DD) (GET /refunds/?to=2013-01-29) - returns refunds created before given date
  * Date combined (GET /refunds/?from=2013-01-28&to=2013-01-29) - returns refunds for the given interval

All refunds so far: <pre>GET /refunds/</pre> Returns: <pre> Status: 200 <code
class="javascript"> [ { id: 210, order_id: 1098, items: [ { item_id: 3516, _
the item id within the order qty: 1, _ QTY to refund refund_reason: 'out of
stock' } ] }, { id: 211, order_id: 1140, items: [ { item_id: 3676, _ the item
id within the order qty: 1, _ QTY to refund refund_reason: 'out of stock' } ]
} ] </code></pre>

Refunds filtered (??? needed ???): <pre> GET /refunds/?id=210 GET
/refunds/?from=2013-01-01&to=2013-02-01 </pre>

Items refund: <pre> POST /refunds/ { order_id: 1098, items: [ { item_id: 3516,
_ the item id within the order qty: 1, _ QTY to refund refund_reason: 'out of
stock' } ] } </pre> Returns: <pre> Status: 201 <code class="javascript"> {id:
210} </code></pre>

### Deliveries

  * Available filters*
  * ID
  * order_id
  * increment_id
  * Date: from (YYYY-MM-DD) (GET /deliveries/?from=2013-01-28) - returns deliveries created starting from given date
  * Date: to (YYYY-MM-DD) (GET /deliveries/?to=2013-01-29) - returns deliveries created before given date
  * Date combined (GET /deliveries/?from=2013-01-28&to=2013-01-29) - returns deliveries for the given interval

All deliveries: <pre>GET /deliveries/</pre> Returns: <pre> Status: 200 <code
class="javascript"> [ { order_id: 1079, increment_id: 10000079, created_at:
'2012-12-13 12:40:41', updated_at: '2013-01-18 13:25:15', shipping_address: {
first_name: 'John', last_name: 'Doe', country: 'UK', city: 'London',
street_address: 'Baker st., 221B', post_code: 'PO12 4EP', phone:
'+441231234567' }, tracks: [ { track_id: 314, track_number: 'FEDEX-3434-DF',
title: 'Federal Express', carrier_code: 'fedex', created_at: '2013-01-18
13:20:28' } ] } ] </code> </pre>

Creating delivery: <pre> POST /deliveries/ { order_id: 1098, items: [ {
item_id: 3517, _ the item id within the order qty: 1 _ the quantity of items
to deliver } ] } </pre> Returns: <pre> Status: 201 <code class="javascript">
{id: 561} </code></pre>

Adding/removing tracking information: <pre> PUT /deliveries/?id=561 <code
class="javascript">{ add_tracks: [ { track_number: 'FEDEX-34AA-DF', title:
'Federal Express', carrier_code: 'fedex' } ], remove_tracks: [314] _ the array
of tracking number IDs } </code> </pre> Returns: <pre> Status: 200 </pre>_

## Working with products

  * Available information*
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
  * Available filters*
  * ID
  * SKU
  * Name
  * Status (enabled/disabled)
  * Visibility (Not visible individually/Catalog, Search/Catalog)
  * Price
  * Special price
  * Stock status
  * Product type

### Products list

<pre>GET /products/</pre> returns: <pre> Status: 200 <code class="javascript">
[ { id: 1245, sku: 'ER567-UG', visibility: 'Catalog, Search', status:
'Enabled', type: 'configurable', name: 'Product 1', description: 'Just a
product', short_description: 'product', price: 125.90, special_price: 100.00,
special_from: '2013-01-01 00:00:00', special_to: '2013-01-02 00:00:00',
children: [ { id: 1246, sku: 'ER567-UG-Black-10', type: 'simple', status:
'Enabled', color: 'Black', size: 10, stock_item: { qty: 12, in_stock: true }
}, { id: 1247, sku: 'ER567-UG-White', type: 'simple', status: 'Enabled',
color: 'White', size: 12, stock_item: { qty: 4, in_stock: true } } ] } ]
</code> </pre>

### Getting the product by sku or ID:

<pre> GET /products/?sku=SKU3333 GET /products/?id=254778 </pre> Returns:
<pre> Status: 200 </pre> Response body has the same format as in Products list

### Editing Products:

Change the name and description for the product #1245: <pre> PUT
/products?id=1245 { name: "new Product name", description: "new description" }
</pre> Returns: <pre>Status: 200</pre>

### Disabling the product:

Disable the product #1247: <pre>PUT /products/?id=1247 { status: 'disabled'
}</pre> Returns: <pre>Status: 200</pre>

### Stock management:

Set quantity=5 for the product #1246: <pre> PUT /products/?id=1246 {
stock_item: { qty: 5 } } </pre> Returns: <pre>Status: 200</pre>

Making product #1247 out of stock: <pre> PUT /products/?id=1247 { stock_item:
{ in_stock: false } } </pre> Returns: <pre>Status: 200</pre>

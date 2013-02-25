# Intu.co.uk Integration

Feature:
* JSON-based RESTful API for Retailers
  
Aim:
* To give the retailers easy way to control their sales and inventory

Version: 0.9

## Resources to be available via API:

  * orders, filtered by retailer (managing statuses) (example.com/orders)
  * products (stock level)
  * refunds
  * deliveries

## Integration

The API is created as a magento module hence having the full access to Magento
ORM.

## URL build rules

HTTP method + base url + resource url + [filters]

## Authentication

Authenticate retailers by their credentials at the Magento backend.

<pre>POST /auth/ 
    {
        login: 'retailer_name',
        password: 'retailer_password'
    }
</pre> 

Response:

<pre>
HTTP/1.1 201 Created
Set-Cookie: PHPSESSID=6d8552f81fbcc7f64de90d563807f5ee
</pre>

Which means that the cookie is set and it should be sent over with every API request. 

## Filtering collections

Filtering the result collections is done using querystrings: 
<pre>GET /orders/?increment_id=10002455
GET /orders/?status=pending
GET /returns/?from=2013-01-01%2000:00:00&to=2013-01-15%2000:00:00 </pre>

You can use sets of values in filters. For example, if you want to get all
orders with statuses "pending" and "processing", the querystring will look
like 
<pre>GET /orders/?status=pending,processing</pre>

See the list of available filters for orders and products in appropriate
sections.

## API Responses

Retailers API always responds with the meaningful HTTP status - e.g. 200 for
successful requests, 404 for the requests not matching the filter, 400 for
wrong requests etc.

  * Response for wrong request

When the API cannot perform the request, it responds with the error flag and
the error description(s): 
<pre>
Status: 400
{
    error_code: 123,
    message: "Wrong order status supplied" 
}
    
</pre>

## Working with orders ##

* Available information
* ID
* Increment ID
* Date of placement
* Status: pending, pending_payment, cancelled, complete, etc.
* Total
* Shipping information
* Customer email
* Items ordered 
* Available filters
* Order ID (GET /orders/?id=1074) - returns order by ID
* Increment ID (GET /orders/?incid=1000045678) - returns order by Increment ID
* Status (GET /orders/?status=pending) - orders with "Pending status" (use comma to separate different values)
* Date: from (YYYY-MM-DD) (GET /orders/?from=2013-01-28) - returns orders placed from given date
* Date: to (YYYY-MM-DD) (GET /orders/?to=2013-01-29) - returns orders placed to given date
* Date combined (GET /orders/?from=2013-01-28&to=2013-01-29) - returns orders for the given interval

### Orders list:

<pre>GET /orders/</pre> 

returns: <pre>Status: 200  
    [
    {
        id: 1074,
        increment_id: '100000546',
        created_at: '2013-01-14 12:30:45',
        status: 'pending',
        shipping: {
            type: 'Retailer Delivery - You have qualified for FREE delivery',
            first_name: 'John',
            last_name: 'Doe',
            country: 'UK',
            city: 'London',
            street_address: 'Baker st., 221B',
            post_code: 'PO12 4EP',
            phone: '+441231234567'
        },
        items: [
            {
                id: 1074,
                sku: 'HHR7-WHITE-8',
                price: 80.90,
                size: 8,
                color: 'white',
                qty_ordered: 1,
                qty_backordered: 0,
                qty_canceled: 0,
                qty_invoiced: 0,
                qty_refunded: 0,
                qty_shipped: 0
            }
        ]
    },
    {
        id: 1098,
        created_at: '2013-01-16 16:21:31',
        status: 'pending_payment',
        shipping: {
            type: 'Retailer Delivery - You have qualified for FREE delivery',
            first_name: 'Lisa',
            last_name: 'Simpson',
            country: 'US',
            city: 'Springfield',
            street_address: '742 Evergreen Terrace',
            post_code: '123',
            phone: '+11341234567'
        },
        items: [
            {
                item_id: 3516,
                product_id: 1255,
                sku: 'DF887-YA-8',
                price: 120.00,
                size: 8,
                color: 'black',
                qty_ordered: 2,
                qty_backordered: 0,
                qty_canceled: 0,
                qty_invoiced: 0,
                qty_refunded: 0,
                qty_shipped: 0
            },
            {
                item_id: 3517,
                product_id: 1174,
                sku: '66R7-WHITE-8',
                price: 49.00,
                size: 8,
                color: 'white',
                qty_ordered: 1,
                qty_backordered: 0,
                qty_canceled: 0,
                qty_invoiced: 0,
                qty_refunded: 0,
                qty_shipped: 0
            }
        ]
    }
]
</pre>

Items are configurable products, if any. Use item_id to refer to an item when creating the refund or delivery. 

### Refunds

  * Available filters
  * ID
  * order_id
  * Date: from (YYYY-MM-DD) (GET /refunds/?from=2013-01-28) - returns refunds created starting from given date
  * Date: to (YYYY-MM-DD) (GET /refunds/?to=2013-01-29) - returns refunds created before given date
  * Date combined (GET /refunds/?from=2013-01-28&to=2013-01-29) - returns refunds for the given interval
  * 
All refunds so far: <pre>GET /refunds/</pre> Returns: <pre>Status: 200
    [
    {
        id: 210,
        order_id: 1098,
        items: [
            {
                item_id: 3516, // the item id within the order
                qty: 1,    // QTY to refund
                refund_reason: 'out of stock'
            }
        ]
    },
    {
        id: 211,
        order_id: 1140,
        items: [
            {
                item_id: 3676, // the item id within the order
                qty: 1,    // QTY to refund
                refund_reason: 'out of stock'
            }
        ]
    }
    ] </pre>

Refunds filtered: 
<pre>GET /refunds/?order_id=820
GET /refunds/?from=2013-01-01&to=2013-02-01 </pre>

Items refund: <pre> POST /refunds/ {
    order_id: 1098,
    comment: "Some comment",
    items: [
        {
            item_id: 3516, // the item id within the order
            qty: 1,    // QTY to refund
            refund_reason: "out of stock" 
        }
    ]
}

</pre>

returns

<pre>
Status: 201
{id: 210}
</pre>

### Deliveries

  * Available filters
  * ID
  * order_id
  * increment_id
  * Date: from (YYYY-MM-DD) (GET /deliveries/?from=2013-01-28) - returns deliveries created starting from given date
  * Date: to (YYYY-MM-DD) (GET /deliveries/?to=2013-01-29) - returns deliveries created before given date
  * Date combined (GET /deliveries/?from=2013-01-28&to=2013-01-29) - returns deliveries for the given interval

All deliveries: 
<pre>GET /deliveries/</pre> 
Returns: 
<pre>Status: 200
    [
    {
        order_id: 1079,
        increment_id: 10000079,
        created_at: '2012-12-13 12:40:41',
        updated_at: '2013-01-18 13:25:15',
        shipping_address: {
            first_name: 'John',
            last_name: 'Doe',
            country: 'UK',
            city: 'London',
            street_address: 'Baker st., 221B',
            post_code: 'PO12 4EP',
            phone: '+441231234567'
        },
        tracks: [
            {
                track_id: 314,
                track_number: 'FEDEX-3434-DF',
                title: 'Federal Express',
                carrier_code: 'fedex',
                created_at: '2013-01-18 13:20:28'
            }
        ]
    }
    ] 
</pre>

Creating delivery: 
<pre>POST /deliveries/
    {
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
<pre> Status: 201
    {
    id: 561
    } 
</pre>

Adding/removing tracking information: 
<pre>PUT /deliveries/?id=561
    {
    add_tracks: [
        {
            track_number: 'FEDEX-34AA-DF',
            title: 'Federal Express',
            carrier_code: 'fedex'
        }
    ],
    remove_tracks: [314] // the array of tracking number IDs
    }
</pre> 
Returns: 
<pre>Status: 200 </pre>

## Working with products

  * Available information
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

<pre>GET /products/</pre> 
Returns: 
<pre>Status: 200
    [
    {
        id: 1245,
        sku: 'ER567-UG',
        visibility: 'Catalog, Search',
        status: 'Enabled',
        type: 'configurable',
        name: 'Product 1',
        description: 'Just a product',
        short_description: 'product',
        price: 125.90,
        special_price: 100.00,
        special_from: '2013-01-01 00:00:00',
        special_to: '2013-01-02 00:00:00',
        children: [
            {
                id: 1246,
                sku: 'ER567-UG-Black-10',
                type: 'simple',
                status: 'Enabled',
                color: 'Black',
                size: 10,
                stock_item: {
                    qty: 12,
                    in_stock: true
                }
            },
            {
                id: 1247,
                sku: 'ER567-UG-White',
                type: 'simple',
                status: 'Enabled',
                color: 'White',
                size: 12,
                stock_item: {
                    qty: 4,
                    in_stock: true
                }
            }
        ]
    }
    ]
</pre>

### Getting the product by sku or ID:

<pre>GET /products/?sku=SKU3333 
GET /products/?id=254778 </pre> 
Returns:
<pre>Status: 200 </pre> Response body has the same format as in Products list

### Editing Products:

Change the name and description for the product #1245: 
<pre>PUT /products?id=1245
    {
        name: "new Product name",
        description: "new description" 
    }
</pre> 
Returns: 
<pre>Status: 200</pre>

### Disabling the product:

Disable the product #1247:

<pre>PUT /products/?id=1247 

    {
        status: 'disabled'
    }
</pre>
Returns: 
<pre>Status: 200</pre>

### Stock management:

Set quantity=5 for the product 1246: 
<pre>PUT /products/?id=1246
    {
    stock_item: {
        qty: 5
    }
    }
</pre> 
Returns: 
<pre>Status: 200</pre>

Making product 1247 out of stock: 
<pre>PUT /products/?id=1247
    {
    stock_item: {
        in_stock: false
        }
    }
</pre> 
Returns: 
<pre>Status: 200</pre>

## Error Codes:

The error code has the following format:
**XYZ**, where
**X** - a digit representing the resource: 

|**digit**|**resource**|
|---------|------------|
| |core|
|1|order|
|2|product|
|3|refund|
|4|delivery|
|5|resource-wide|

**Y** - error type

|**digit**|**description**|
|---------|---------------|
|1|validation error|
|2|valid but unacceptable|

|**Error Code**|**Short Description**|**Detailed Information**|**HTTP Status Code**|
|--------------|---------------------|------------------------|--------------------|
|11|Unauthorised access - please authentificate first|You should POST your credentials to /auth first|401|
|12|Wrong username/password combination|Please check your credentials|400|
|311|Wrong data format|order_id or items are required to create a refund|400|
|210|Empty request|No data during request, or broken in json structure|400|
|211|Validation error at: field_name|Incorrect data to the field provided|400|
|217|Invalid request|The field required to update is not exist or allowed|400|
|312|Cannot create a credit memo for this order|Order is either closed, canceled, already refunded or the payment is in the "Payment review" status|400|
|314|Unknown refund reason (reason)|Provided refund reason is not valid (see http:// for the list)|400|
|413|Missing data|order_id or items are required to create delivery|400|
|414|Wrong data format|order_id should be a numeric and items - an array|400|
|417|Invalid request|The request should contain either add_tracks or remove_tracks|400|
|418|Tracking number ({id}) not found|Check the track IDs for existence|400|
|421|Order not found|The supplied order_id does not exist|400|
|422|Cannot do shipment for the order|Cannot do shipment for the order|400|
|511|The ID should be integer|The supplied ID should be integer|400|



# FLOW-Houyuu OMS Integration API Specification

**Version:** 1.0  
**© T.C.C. Technology Co. Ltd.**

---

## Document History

> Paper copies are valid only on the day they are printed. Contact the author if you are in doubt about this document's accuracy.

### Revision History

| Version | Revision Date | Summary of Changes | Author |
|---------|--------------|-------------------|--------|
| 1.0 | 24-03-2026 | Created and wrote document. | Thanawut Tuamprajak |

---

## Contents

1. [Create Order](#1-create-order)
2. [Update Order](#2-update-order)
3. [Cancel Order](#3-cancel-order)
4. [Update Order Status](#4-update-order-status)

---

## 1. Create Order

This API is used to create a new order in the system. It should be called when a customer places an order, providing all required order items, shipping address, payment method, and optional tax invoice details.

### Request

| Method | URL |
|--------|-----|
| `POST` | `$api_url/api/v1/external/houyuu/order` |

### Authorization

| Type | Value | Header Key |
|------|-------|------------|
| JWT Token | `Bearer $token` | `Authorization` |

### Request Body
```json
{
    "order_no": "TUAT2026031200001",
    "customer_code": "00000009",
    "order_items": [
        {
            "sku": "8851707000808",
            "barcode": "18851707000805",
            "quantity": 2,
            "seq": 1,
            "warehouse": "01",
            "key_discount": 0,
            "is_free": false
        },
        {
            "sku": "8851707000181",
            "barcode": "18851707000188",
            "quantity": 10,
            "seq": 2,
            "warehouse": "01",
            "key_discount": 0,
            "is_free": true
        }
    ],
    "discount_bill": 0,
    "shipping_fee": 0,
    "shipping_address": {
        "address": "123 Main Street",
        "sub_district": "Khlong Toei",
        "district": "Khlong Toei",
        "province": "Bangkok",
        "zipcode": "10110",
        "latitude": "13.722539",
        "longitude": "100.564834",
        "phone_1": "0812345678",
        "phone_2": "0898765432"
    },
    "tax_address": {
        "tax_id": "0105556012345",
        "address": "456 Tax Road",
        "sub_district": "Khlong Toei",
        "district": "Khlong Toei",
        "province": "Bangkok",
        "zipcode": "10110",
        "phone_1": "0812345678",
        "phone_2": "0898765432"
    },
    "coupon_code": null
}
```

### Parameters

| Name | Type | Description | Required |
|------|------|-------------|----------|
| `order_no` | String | Unique order number. | Yes |
| `customer_code` | String | Customer code identifier. | Yes |
| `order_items` | Array\<Object\> | List of order items. See [order_items](#nested-object-order_items-1). | Yes |
| `discount_bill` | Number | Discount key applied to the entire bill. | No |
| `shipping_fee` | Number | Shipping fee for the order. | No |
| `shipping_address` | Object | Shipping destination details. See [shipping_address](#nested-object-shipping_address-1). | Yes |
| `tax_address` | Object | Tax invoice address details. See [tax_address](#nested-object-tax_address-1). | No |
| `coupon_code` | String | Coupon code to apply to the order. | No |

### Nested Object: order_items

| Name | Type | Description | Required |
|------|------|-------------|----------|
| `sku` | String | SKU code of the product. | Yes |
| `barcode` | String | Barcode of the product. | Yes |
| `quantity` | Integer | Quantity of the product. | Yes |
| `seq` | Integer | Sequence number of the item in the order. | Yes |
| `warehouse` | String | Warehouse code for the item. | Yes |
| `key_discount` | Number | Discount key applied to the item. If no discount, set to `0`. | No |
| `is_free` | Boolean | Indicates whether the item is a free item inserted manually. | Yes |

### Nested Object: shipping_address

| Name | Type | Description | Required |
|------|------|-------------|----------|
| `address` | String | Street address for shipping. | No |
| `sub_district` | String | Sub-district of the shipping address. | Yes |
| `district` | String | District of the shipping address. | Yes |
| `province` | String | Province of the shipping address. | Yes |
| `zipcode` | String | Zip code of the shipping address. | Yes |
| `latitude` | Float | Latitude coordinate of the shipping location. | No |
| `longitude` | Float | Longitude coordinate of the shipping location. | No |
| `phone_1` | String | Primary contact phone number for shipping. | Yes |
| `phone_2` | String | Secondary contact phone number for shipping. | No |

### Nested Object: tax_address

| Name | Type | Description | Required |
|------|------|-------------|----------|
| `tax_id` | String | Tax identification number. | Yes |
| `address` | String | Street address for tax invoice. | Yes |
| `sub_district` | String | Sub-district of the tax address. | Yes |
| `district` | String | District of the tax address. | Yes |
| `province` | String | Province of the tax address. | Yes |
| `zipcode` | String | Zip code of the tax address. | Yes |
| `phone_1` | String | Primary contact phone number for tax invoice. | Yes |
| `phone_2` | String | Secondary contact phone number for tax invoice. | No |

### Response

| Status | Reason | Description |
|--------|--------|-------------|
| 201 | Created | Order created successfully. |
| 400 | Bad Request | The request body is malformed, missing required fields, or contains invalid values. |
| 401 | Unauthorized | The JWT token is missing, expired, or invalid. |
| 500 | Internal Server Error | An unexpected error occurred on the server. |

### Examples

**Success (201)**
```json
{
  "error": false,
  "message": "Order created successfully.",
  "errors": []
}
```

**Invalid request body (400)**
```json
{
  "error": true,
  "message": "Validation failed.",
  "error_detail": {
    "order": [
      {
        "field": "order_no",
        "message": "order_no is required."
      },
      {
        "field": "customer_code",
        "message": "customer_code is required."
      }
    ],
    "shipping_address": [
      {
        "field": "province",
        "message": "province is required."
      }
    ],
    "items": [
      {
        "index": 0,
        "fields": [
          {
            "field": "sku",
            "message": "sku is required."
          }
        ]
      }
    ]
  }
}
```

**Unauthorized (401)**
```json
{
  "message": "Unauthorized"
}
```

**Internal server error (500)**
```json
{
  "error": true,
  "message": "$error_message",
  "errors": []
}
```

---

## 2. Update Order

This API is used to update an existing order in the system. It allows the caller to modify order items, premium items, shipping address, and billing details for a given order number. This endpoint should be called when an order needs to be revised after it has been created, such as changes to item quantities, free items, or delivery information.

### Request

| Method | URL |
|--------|-----|
| `PUT` | `$api_url/api/v1/external/houyuu/order` |

### Authorization

| Type | Value | Header Key |
|------|-------|------------|
| JWT Token | `Bearer $token` | `Authorization` |

### Request Body
```json
{
    "customer_code": "00000009",
    "order_items": [
        {
            "sku": "8851707000808",
            "barcode": "18851707000805",
            "quantity": 2,
            "seq": 1,
            "warehouse": "01",
            "key_discount": 0,
            "is_free": false
        },
        {
            "sku": "8851707000181",
            "barcode": "18851707000188",
            "quantity": 10,
            "seq": 2,
            "warehouse": "01",
            "key_discount": 0,
            "is_free": true
        }
    ],
    "order_no": "TUAT2026031200001",
    "discount_bill": 0,
    "shipping_fee": 0,
    "shipping_address": {
        "address": "123 Main Street",
        "sub_district": "Khlong Toei",
        "district": "Khlong Toei",
        "province": "Bangkok",
        "zipcode": "10110",
        "latitude": "13.722539",
        "longitude": "100.564834",
        "phone_1": "0812345678",
        "phone_2": "0898765432"
    },
    "tax_address": {
        "tax_id": "0105556012345",
        "address": "456 Tax Road",
        "sub_district": "Khlong Toei",
        "district": "Khlong Toei",
        "province": "Bangkok",
        "zipcode": "10110",
        "phone_1": "0812345678",
        "phone_2": "0898765432"
    }
}
```

### Parameters

| Name | Type | Description | Required |
|------|------|-------------|----------|
| `customer_code` | String | Customer code identifier. | Yes |
| `order_no` | String | Order number. | Yes |
| `order_items` | Array\<Object\> | List of order items. See [order_items](#nested-object-order_items). | Yes |
| `discount_bill` | Number | Discount key by user applied to the entire bill. | No |
| `shipping_fee` | Number | Shipping fee for the order. | No |
| `shipping_address` | Object | Shipping destination details. See [shipping_address](#nested-object-shipping_address). | Yes |
| `tax_address` | Object | Tax invoice address details. See [tax_address](#nested-object-tax_address). | No |

### Nested Object: order_items

| Name | Type | Description | Required |
|------|------|-------------|----------|
| `sku` | String | SKU code of the product. | Yes |
| `barcode` | String | Barcode of the product. | Yes |
| `quantity` | Integer | Quantity of the product. | Yes |
| `seq` | Integer | Sequence number of the item in the order. | Yes |
| `warehouse` | String | Warehouse code for the item. | Yes |
| `key_discount` | Number | Discount key applied to the item. If no discount, set to `0`. | No |
| `is_free` | Boolean | Indicates whether the item is a free item. In case user want to insert free item manually. | Yes |

### Nested Object: shipping_address

| Name | Type | Description | Required |
|------|------|-------------|----------|
| `address` | String | Street address for shipping. | No |
| `sub_district` | String | Sub-district of the shipping address. | Yes |
| `district` | String | District of the shipping address. | Yes |
| `province` | String | Province of the shipping address. | Yes |
| `zipcode` | String | Zip code of the shipping address. | Yes |
| `latitude` | Float | Latitude coordinate of the shipping location. | No |
| `longitude` | Float | Longitude coordinate of the shipping location. | No |
| `phone_1` | String | Primary contact phone number for shipping. | Yes |
| `phone_2` | String | Secondary contact phone number for shipping. | No |

### Nested Object: tax_address

| Name | Type | Description | Required |
|------|------|-------------|----------|
| `tax_id` | String | Tax identification number. | Yes |
| `address` | String | Street address for tax invoice. | Yes |
| `sub_district` | String | Sub-district of the tax address. | Yes |
| `district` | String | District of the tax address. | Yes |
| `province` | String | Province of the tax address. | Yes |
| `zipcode` | String | Zip code of the tax address. | Yes |
| `phone_1` | String | Primary contact phone number for tax invoice. | Yes |
| `phone_2` | String | Secondary contact phone number for tax invoice. | No |

### Response

| Status | Reason | Description |
|--------|--------|-------------|
| 200 | OK | Order updated successfully. |
| 400 | Bad Request | The request body is malformed, missing required fields, or contains invalid values. |
| 401 | Unauthorized | The JWT token is missing, expired, or invalid. |
| 404 | Not Found | The specified order number does not exist. |
| 500 | Internal Server Error | An unexpected error occurred on the server. |

### Examples

**Success (200)**
```json
{
  "error": false,
  "message": "Order updated successfully.",
  "errors": []
}
```

**Invalid request body (400)**
```json
{
   "error":true,
   "message":"Validation failed.",
   "error_detail":{
      "order":[
         {
            "field":"order_no",
            "message":"order_no is required."
         },
         {
            "field":"customer_code",
            "message":"customer_code is required."
         }
      ],
      "shipping_address":[
         {
            "field":"province",
            "message":"province is required."
         }
      ],
      "items":[
         {
            "index":0,
            "fields":[
               {
                  "field":"sku",
                  "message":"sku is required."
               },
               {
                  "field":"quantity",
                  "message":"quantity must be greater than 0."
               }
            ]
         }
      ]
   }
}
```

**Unauthorized (401)**
```json
{
  "message": "Unauthorized"
}
```

**Internal server error (500)**
```json
{
  "error": true,
  "message": "$error_message",
  "errors": []
}
```

---
## 3. Cancel Order

This API is used to cancel an existing order in the system. It should be called when an order is no longer required and needs to be voided. An optional description can be provided to record the reason for cancellation.

### Request

| Method | URL |
|--------|-----|
| `POST` | `$api_url/api/v1/external/houyuu/order/cancel` |

### Authorization

| Type | Value | Header Key |
|------|-------|------------|
| JWT Token | `Bearer $token` | `Authorization` |

### Request Body
```json
{
  "order_no": "TUAT2026031200001",
  "description": "Customer requested cancellation."
}
```

### Parameters

| Name | Type | Description | Required |
|------|------|-------------|----------|
| `order_no` | String | Order number to cancel. | Yes |
| `description` | String | Reason or note for the cancellation. | No |

### Response

| Status | Reason | Description |
|--------|--------|-------------|
| 200 | OK | Order cancelled successfully. |
| 400 | Bad Request | The request body is malformed or missing required fields. |
| 401 | Unauthorized | The JWT token is missing, expired, or invalid. |
| 404 | Not Found | The specified order number does not exist. |
| 500 | Internal Server Error | An unexpected error occurred on the server. |

### Examples

**Success (200)**
```json
{
  "error": false,
  "message": "Order cancelled successfully.",
  "errors": []
}
```

**Invalid request body (400)**
```json
{
  "error": true,
  "message": "Validation failed.",
  "errors": [
    {
      "field": "order_no",
      "message": "order_no is required."
    }
  ]
}
```

**Unauthorized (401)**
```json
{
  "message": "Unauthorized"
}
```

**Internal server error (500)**
```json
{
  "error": true,
  "message": "$error_message",
  "errors": []
}
```

---

## 4. Update Order Status

This API is used to update the status of an existing order. It should be called when the order status needs to be progressed or changed, such as moving from processing to shipped. The `status` field accepts an integer code representing the target status.

### Request

| Method | URL |
|--------|-----|
| `PUT` | `$api_url/api/v1/external/houyuu/order/status` |

### Authorization

| Type | Value | Header Key |
|------|-------|------------|
| JWT Token | `Bearer $token` | `Authorization` |

### Request Body
```json
{
  "order_no": "TUAT2026031200001",
  "status": 1
}
```

### Order Status Reference

| Code | Status (TH) | Status (EN) |
|------|-------------|-------------|
| `1` | รอดำเนินการ | Waiting to Confirm |
| `3` | รอจัดส่ง | Ready to Ship |
| `4` | จัดส่งแล้ว | Shipped |
| `5` | สำเร็จ | Completed |
| `6` | ยกเลิกโดยแอดมิน | Cancelled by Admin |
| `6` | ยกเลิกโดยผู้ซื้อ | Cancelled by Buyer |


### Parameters

| Name | Type | Description | Required |
|------|------|-------------|----------|
| `order_no` | String | Order number to update. | Yes |
| `status` | Integer | Status code to apply to the order. | Yes |

### Response

| Status | Reason | Description |
|--------|--------|-------------|
| 200 | OK | Order status updated successfully. |
| 400 | Bad Request | The request body is malformed or missing required fields. |
| 401 | Unauthorized | The JWT token is missing, expired, or invalid. |
| 404 | Not Found | The specified order number does not exist. |
| 500 | Internal Server Error | An unexpected error occurred on the server. |

### Examples

**Success (200)**
```json
{
  "error": false,
  "message": "Order status updated successfully.",
  "errors": []
}
```

**Invalid request body (400)**
```json
{
  "error": true,
  "message": "Validation failed.",
  "errors": [
    {
      "field": "order_no",
      "message": "order_no is required."
    }
  ]
}
```

**Unauthorized (401)**
```json
{
  "message": "Unauthorized"
}
```

**Internal server error (500)**
```json
{
  "error": true,
  "message": "$error_message",
  "errors": []
}
```


*© T.C.C. Technology Co. Ltd.*

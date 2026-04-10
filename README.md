# FLOW-store OMS Integration API Specification

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
| `POST` | `$api_url/api/v1/external/order` |

### Query Parameters

| Name | Type | Description | Required |
|------|------|-------------|----------|
| `store_code` | String | Store ID of the store branch. | Yes |

### Authorization

| Type | Value | Header Key |
|------|-------|------------|
| JWT Token | `Bearer $token` | `Authorization` |

### Request Body
```json
{
    "order_no": "REF-HOU4728301956",
    "customer_code": "00000009",
    "order_items": [
        {
            "barcode": "18851707000805",
            "quantity": 2,
            "seq": 1,
            "key_discount": 0,
            "is_free": false
        },
        {
            "barcode": "18851707000188",
            "quantity": 10,
            "seq": 2,
            "key_discount": 0,
            "is_free": true
        }
    ],
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
    }
}
```

### Parameters

| Name | Type | Description | Required |
|------|------|-------------|----------|
| `order_no` | String | Order number used by store to identify the order. | Yes |
| `customer_code` | String | Customer code identifier. | Yes |
| `order_items` | Array\<Object\> | List of order items. See [order_items](#nested-object-order_items-1). | Yes |
| `shipping_address` | Object | Shipping destination details. See [shipping_address](#nested-object-shipping_address-1). | No |

### Nested Object: order_items

| Name | Type | Description | Required |
|------|------|-------------|----------|
| `barcode` | String | Barcode of the product. | Yes |
| `quantity` | Integer | Quantity of the product. | Yes |
| `seq` | Integer | Sequence number of the item in the order. | Yes |
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
    "errors": [],
    "data": {
        "order_no": "TCCT2026040800001",
        "benefit_summary": {
            "total_discount": 6,
            "total_sku_discount": 6,
          },
        "promotions": [
            {
                "promotion_name": "ซื้อกลุ่มสินค้าแถมpremium",
                "promotion_code": "TCCT202511-08",
                "promotion_type": "premium",
                "discount_amount": 0,
                "premiums": [
                    {
                        "sku": "TARO10002",
                        "barcode": "TARO10002",
                        "title": "TARO10002",
                        "free_quantity": 1,
                        "unit_name": "ซองx1"
                    }
                ]
            }
        ]
    }
}
```

#### Response Body

| Name | Type | Description |
|------|------|-------------|
| `error` | Boolean | Indicates whether an error occurred. `false` on success. |
| `message` | String | Human-readable result message. |
| `errors` | Array | List of error details. Empty array on success. |
| `data` | Object | Order result data. See [data](#nested-object-data). |

#### Nested Object: data

| Name | Type | Description |
|------|------|-------------|
| `order_no` | String | Internal order number generated by the system. |
| `benefit_summary` | Object | Summary of all discount and benefit amounts applied to the order. See [benefit_summary](#nested-object-benefit_summary). |
| `promotions` | Array\<Object\> | List of promotions applied to the order. See [promotions](#nested-object-promotions). |

#### Nested Object: benefit_summary

| Name | Type | Description |
|------|------|-------------|
| `total_discount` | Number | Total discount amount from all sources. |
| `total_sku_discount` | Number | Total discount applied at the SKU/item level. |

#### Nested Object: promotions

| Name | Type | Description |
|------|------|-------------|
| `promotion_name` | String | Display name of the promotion. |
| `promotion_code` | String | Unique code identifying the promotion. |
| `promotion_type` | String | Type of promotion (e.g., `step_discount`, `premium`, `point`, `coin`). |
| `discount_amount` | Number | Discount amount granted by this promotion. |
| `premiums` | Array\<Object\> | List of free premium items granted. See [premiums](#nested-object-premiums). Empty array if none. |

#### Nested Object: premiums

| Name | Type | Description |
|------|------|-------------|
| `sku` | String | SKU code of the premium item. |
| `barcode` | String | Barcode of the premium item. |
| `title` | String | Display name of the premium item. |
| `free_quantity` | Integer | Quantity of the premium item granted for free. |
| `unit_name` | String | Unit description of the premium item. |



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
| `PUT` | `$api_url/api/v1/external/order` |

### Query Parameters

| Name | Type | Description | Required |
|------|------|-------------|----------|
| `store_code` | String | Store ID of the store branch. | Yes |

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
            "barcode": "18851707000805",
            "quantity": 2,
            "seq": 1,
            "key_discount": 0,
            "is_free": false
        },
        {
            "barcode": "18851707000188",
            "quantity": 10,
            "seq": 2,
            "key_discount": 0,
            "is_free": true
        }
    ],
    "order_no": "REF-HOU4728301956",
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
    }
}
```

### Parameters

| Name | Type | Description | Required |
|------|------|-------------|----------|
| `customer_code` | String | Customer code identifier. | Yes |
| `order_no` | String | Order number used by store to identify the order. | Yes |
| `order_items` | Array\<Object\> | List of order items. See [order_items](#nested-object-order_items). | Yes |
| `shipping_address` | Object | Shipping destination details. See [shipping_address](#nested-object-shipping_address). | No |

### Nested Object: order_items

| Name | Type | Description | Required |
|------|------|-------------|----------|
| `barcode` | String | Barcode of the product. | Yes |
| `quantity` | Integer | Quantity of the product. | Yes |
| `seq` | Integer | Sequence number of the item in the order. | Yes |
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
    "errors": [],
    "data": {
        "order_no": "TCCT2026040800001",
        "benefit_summary": {
            "total_discount": 6,
            "total_sku_discount": 6,
        },
        "promotions": [
            {
                "promotion_name": "ซื้อกลุ่มสินค้าแถมpremium",
                "promotion_code": "TCCT202511-08",
                "promotion_type": "premium",
                "discount_amount": 0,
                "premiums": [
                    {
                        "sku": "TARO10002",
                        "barcode": "TARO10002",
                        "title": "TARO10002",
                        "free_quantity": 1,
                        "unit_name": "ซองx1"
                    }
                ]
            }
        ]
    }
}
```

#### Response Body

| Name | Type | Description |
|------|------|-------------|
| `error` | Boolean | Indicates whether an error occurred. `false` on success. |
| `message` | String | Human-readable result message. |
| `errors` | Array | List of error details. Empty array on success. |
| `data` | Object | Order result data. See [data](#nested-object-data). |

#### Nested Object: data

| Name | Type | Description |
|------|------|-------------|
| `order_no` | String | Internal order number generated by the system. |
| `benefit_summary` | Object | Summary of all discount and benefit amounts applied to the order. See [benefit_summary](#nested-object-benefit_summary). |
| `promotions` | Array\<Object\> | List of promotions applied to the order. See [promotions](#nested-object-promotions). |

#### Nested Object: benefit_summary

| Name | Type | Description |
|------|------|-------------|
| `total_discount` | Number | Total discount amount from all sources. |
| `total_sku_discount` | Number | Total discount applied at the SKU/item level. |

#### Nested Object: promotions

| Name | Type | Description |
|------|------|-------------|
| `promotion_name` | String | Display name of the promotion. |
| `promotion_code` | String | Unique code identifying the promotion. |
| `promotion_type` | String | Type of promotion (e.g., `step_discount`, `premium`, `point`, `coin`). |
| `discount_amount` | Number | Discount amount granted by this promotion. |
| `premiums` | Array\<Object\> | List of free premium items granted. See [premiums](#nested-object-premiums). Empty array if none. |

#### Nested Object: premiums

| Name | Type | Description |
|------|------|-------------|
| `sku` | String | SKU code of the premium item. |
| `barcode` | String | Barcode of the premium item. |
| `title` | String | Display name of the premium item. |
| `free_quantity` | Integer | Quantity of the premium item granted for free. |
| `unit_name` | String | Unit description of the premium item. |


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
| `POST` | `$api_url/api/v1/external/order/cancel` |

### Query Parameters

| Name | Type | Description | Required |
|------|------|-------------|----------|
| `store_code` | String | Store ID of the store branch. | Yes |

### Authorization

| Type | Value | Header Key |
|------|-------|------------|
| JWT Token | `Bearer $token` | `Authorization` |

### Request Body
```json
{
  "order_no": "REF-HOU4728301956",
  "description": "Customer requested cancellation."
}
```

### Parameters

| Name | Type | Description | Required |
|------|------|-------------|----------|
| `order_no` | String | Order number used by store to identify the order to cancel. | Yes |
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
| `PUT` | `$api_url/api/v1/external/order/status` |

### Query Parameters

| Name | Type | Description | Required |
|------|------|-------------|----------|
| `store_code` | String | Store ID of the store branch. | Yes |

### Authorization

| Type | Value | Header Key |
|------|-------|------------|
| JWT Token | `Bearer $token` | `Authorization` |

### Request Body
```json
{
  "order_no": "REF-HOU4728301956",
  "status": 1
}
```

### Parameters

| Name | Type | Description | Required |
|------|------|-------------|----------|
| `order_no` | String | Order number used by store to identify the order to update. | Yes |
| `status` | Integer | Status code to apply to the order. | Yes |


### Order Status Reference

| Code | Status (TH) | Status (EN) |
|------|-------------|-------------|
| `1` | รอดำเนินการ | Waiting to Confirm |
| `3` | รอจัดส่ง | Ready to Ship |
| `4` | จัดส่งแล้ว | Shipped |
| `5` | สำเร็จ | Completed |
| `6` | ยกเลิกโดยแอดมิน | Cancelled by Admin |
| `7` | ยกเลิกโดยผู้ซื้อ | Cancelled by Buyer |
| `9` | รอขาย POS | Awaiting POS Selling |



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
---

*© T.C.C. Technology Co. Ltd.*

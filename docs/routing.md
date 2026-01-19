# Rounting

## Customers

### GET /api/v1/customers
Returns a paginated list of customers.

#### params
- page (integer, optional)
- per_page (integer, optional)

```json
{
  "customers": [
    {
      "id": "cus_123",
      "external_id": "123",
      "first_name": "Tony",
      "last_name": "Stark",
      "email": "tony@stark.com",
      "gateway_reference": "pagar_me"
    }
  ]
}
```

---

### GET /api/v1/customers/:id
Returns a single customer.

#### params
- id (string, required)

```json
{
  "customer": {
    "id": "cus_123",
    "external_id": "123",
    "first_name": "Tony",
    "last_name": "Stark",
    "email": "tony@stark.com",
    "gateway_reference": "pagar_me"
  }
}
```

---

### POST /api/v1/customers
Creates a customer.

#### params
```json
{
  "external_id": "123",
  "first_name": "Tony",
  "last_name": "Stark",
  "email": "tony@starkindustries.com",
  "gateway_reference": "pagar_me"
}
```

```json
{
  "customer": {
    "id": "cus_123",
    "external_id": "123",
    "first_name": "Tony",
    "last_name": "Stark",
    "email": "tony@starkindustries.com",
    "gateway_reference": "pagar_me"
  }
}
```

---

### PUT /api/v1/customers/:id
Updates a customer.

#### params
```json
{
  "first_name": "Anthony",
  "email": "finance@starkindustries.com"
}
```

```json
{
  "customer": {
    "id": "cus_123",
    "first_name": "Anthony",
    "email": "finance@starkindustries.com"
  }
}
```

---

### DELETE /api/v1/customers/:id
Deletes a customer.

#### params
- id (string, required)

---

## Subscriptions

### GET /api/v1/subscriptions

#### params
- page (integer, optional)

```json
{
  "subscriptions": [
    {
      "id": "sub_123",
      "customer_id": "cus_123",
      "billing_interval": "monthly",
      "status": "active",
      "last_payment_at": "2026-01-16"
    }
  ]
}
```

---

### POST /api/v1/subscriptions

#### params
```json
{
  "customer_id": "cus_123",
  "billing_interval": "monthly",
  "product_ids": ["prod_1", "prod_2"]
}
```

```json
{
  "subscription": {
    "id": "sub_123",
    "customer_id": "cus_123",
    "billing_interval": "monthly",
    "status": "active",
    "last_payment_at": "2026-01-16"
  }
}
```

---

### PUT /api/v1/subscriptions/:id

#### params
```json
{
  "status": "canceled"
}
```

```json
{
  "subscription": {
    "id": "sub_123",
    "status": "canceled",
    "last_payment_at": "2026-01-16"
  }
}
```

---

## Invoices

### GET /api/v1/invoices

#### params
- page (integer, optional)

```json
{
  "invoices": [
    {
      "id": "inv_123",
      "customer_id": "cus_123",
      "total_cents": 24900,
      "status": "open"
    }
  ]
}
```

---

### POST /api/v1/invoices

#### params
```json
{
  "customer_id": "cus_123",
  "items": [
    { "product_id": "prod_1", "amount_cents": 19900 },
    { "product_id": "prod_2", "amount_cents": 5000 }
  ]
}
```

```json
{
  "invoice": {
    "id": "inv_123",
    "total_cents": 24900,
    "status": "open"
  }
}
```

---

### POST /api/v1/invoices/:id/charge

#### params
- id (string, required)

```json
{
  "order": {
    "id": "ord_123",
    "invoice_id": "inv_123",
    "status": "paid",
    "gateway_reference": "pagar_me"
  }
}
```

---

## Products

### GET /api/v1/products

#### params
- page (integer, optional)

```json
{
  "products": [
    {
      "id": "prod_1",
      "name": "PMS",
      "status": "active"
    }
  ]
}
```

---

### POST /api/v1/products

#### params
```json
{
  "name": "PMS",
  "status": "active"
}
```

```json
{
  "product": {
    "id": "prod_1",
    "name": "PMS",
    "status": "active"
  }
}
```

---

## Webhooks

### POST /api/v1/webhooks/pagar_me

#### params
```json
{
  "event": "order.paid",
  "data": {}
}
```

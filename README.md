# PayMe - GimmeMyMoney
The objective of this application is to centralize all payment processing and billing operations onto a single platform, exclusively utilizing one payment gateway.

By abstracting payment flows behind a unified domain, the platform aims to:
- Simplify future migrations
- Reduce financial and operation complexity
- Improve customer experience
- Allow multiple products to share same billing infrastructure
- Support recurring and one-time charges

The application follows Ruby and Rails conventions, prioritizing simplicity, explicit boundaries, and clear domain modeling.
## The API:
This will be a REST API designed to centralize and handle all billing interations, including customers, invoices, subscriptions, and orders.
## Tech Specs:
- Ruby 3+
- Rails 7+
- PostgreSQL 16+
- Rspec for testing
- Redis for caching
- JWT auth
- Swagger / OpenAPI documentation (RSwag)
- Httparty for external requests
- Solid Queue (jobs & background work)
- Sentry for error traking
---

## Convencions and Standards
- RESTful design.
- Content-Type: `application/json`
- Timezone: UTC
- Proper HTTP status codes
- API versioning via path `/api/v1`
- API-KEY based authentication
- Date format: ISO 8601 `YYYY-MM-DD`
---
### Folder Layout
```bash
app/
├── models/
│   ├── application_record.rb
│   ├── customer.rb
│   ├── product.rb
│   ├── subscription.rb
│   ├── invoice.rb
│   ├── invoice_line.rb
│   ├── payment_method.rb
│   ├── audit_log.rb
|   ├── payments/
│   │   ├── base_processor.rb
│   │   ├── pagar_me_processor.rb
│   ├── concerns/
│   │   ├── filterable.rb
│   ├── Subscriptions/
│   │   ├── creator.rb
├── controllers/
│   ├── application_controller.rb
│   ├── api/
│   │   ├── application_controller.rb
│   │   └── v1/
│   │       ├── customers_controller.rb
│   │       ├── products_controller.rb
│   │       ├── subscriptions_controller.rb
│   │       ├── invoices_controller.rb
│   │       ├── payment_methods_controller.rb
│   │       ├── audit_logs_controller.rb
├── jobs/
│   ├── application_job.rb
│   └── subscription_billing_job.rb
├── views/
config/
db/
spec/
├── models/
├── requests/
├── system/
├── factories/
├── policies/
├── jobs/
├── support/
└── spec_helper.rb
```
---

## Data modeling
The domain is modeled around the concept of a Customer. All billing entities are scoped by a customer.

### Core Entities
- [ApiClients](docs/data_modeling.md#ApiClient)
* [Customers](docs/data_modeling.md#Customer)
- [BillingProfile](docs/data_modeling.md#BillingProfile)
* [PaymentMethods](docs/data_modeling.md#PaymentMethod)
* [Products](docs/data_modeling.md#Product)
* [Subscriptions](docs/data_modeling.md#Subscription)
* [SubscriptionItems](docs/data_modeling.md#SubscriptionItem)
* [Invoices](docs/data_modeling.md#Invoice)
* [InvoiceItems](docs/data_modeling.md#InvoiceItem)
* [Order](docs/data_modeling.md#Order)

## Governance
* [AuditLogs](docs/data_mo#AuditLog)

[see details](docs/data_modeling.md)
---
## Routing
All endpoints for listing are filterable and paginatable.

[see filterable implementation](https://github.com/brunotoral/Frameable/blob/main/app/models/concerns/filterable.rb)

```ruby
PayMe::Engine.routes.draw do
  namespace :api do
    namespace :v1 do
      resources :customers do
        resources :billing_profiles, only: [:index, :create]
        resources :payment_methods, only: [:index, :create]
      end

      resources :products
      resources :subscriptions

      resources :invoices, only: [:index, :show, :create] do
        member do
          post :charge
        end
      end

      resources :orders, only: [:index, :show]
      resources :payment_methods, only: [:destroy]
      resources :audit_logs, only: [:index, :show]

      namespace :webhooks do
        post :pagar_me
        post :asaas
      end
    end
  end
end

[see all routes and contracts](docs/routing.md)
---
# How it works

## Payment strategy
 - The APi decides when to charge
 - We will use Processor to execute the charge
 - Each processor implements the same interface

 ```ruby
 module Payments
   module Processor
     def charge(invoice, options = {}); end
     def subscribe(subscription, options = {}); end
     def cancel_subscription(subscription); end
     # ...
     # and others necessary methods.
   end
 end
 ```

### Processor Implementation
 - Each processor implements the necessary flow for each gateway

 ```ruby
 module Payments
   module Processors
     class PagarMeProcessor < BaseProcessor
       def initialize(customer); end

       def charge(invoice)
         # translate invoice -> pagar.me order
       end

       def subscribe(subscription)
         # create pagar.me subscription
       end

       # all gateway logic goes here
     end
   end
 end
 ```
### Usage
#
```ruby
 # app/models/customer.rb
 class Customer < ApplicationRecord
   def payment_processor
     @processor ||= Payments::Processors.for(gateway_reference)

     @processor.new self
   end
 end

 customer.payment_processor.charge(invoice)
```

[see example of implementation an usage in this project](https://github.com/brunotoral/collector)

## Multiple products in a single invoice
A single `Invoice` can be generated to include charges for multiple `Product`s. This is achieved through the `InvoiceItem` data model, which acts as a line item on an invoice.

Each `Invoice` is associated with one or more `InvoiceItem`s. An `InvoiceItem` links directly to a `Product` and specifies the amount to be charged for that specific product within the context of the invoice.

[see similar implementation here](https://github.com/brunotoral/store_seller)

## Recurrence
 - Subscription define recurrence
 - Jobs generates Invoices

 ```ruby
class SubscriptionBillingJob < ApplicationJob
  queue_as :subscriptions

  def perform
    subscriptions = Subscription.due_for_billing

    subscriptions.find_each do |subscription|
      ApplicationRecord.transaction do
         @invoice = Invoices::Creator.new(subscription).execute
       end

      Payments::ChargeInvoiceJob.perform_later(@invoice_id)
    end
  end
end

class Invoices::Creator
  # ...
  def execute
    ApplicationRecord.transaction do
      invoice = Invoice.find_or_create_by!(
        subscription: subscription,
        due_at: subscription.current_cycle
      )
      # ...
      # ...
      invoice.calculate_total_cents!
    end
  end
end

class Payments::ChargeInvoiceJob < ApplicationJob
  def perform(invoice_id)
     invoice = Invoice.includes(:customer).open.find(invoice_id)
     customer = invoice.customer

    Invoice.transaction do
      customer.payment_processor(invoice).charge!
    end
  end
end
 ```

## Idempotency
 It will be ensured without explicity idempotency key
 - Using unique constraints on DB level
 - Using pessimistic locking

 ```ruby
  # app/models/order.rb
    class Order < ApplicationRecord
      enum :status, %i[pending paid failed]
    end

    add_index :orders, [:invoice_id], unique: true, where: 'status = 1', name: 'index_orders_on_invoice_id_and_status_paid'
 ```

## AuditLog

```ruby
class Subscriptions::Canceler
  def initialize(subscription:, actor_id:); end

  def execute
    ApplicationRecord.transaction do
      # Perform the main business logic
      # ...
      AuditLog.create!(
        entity_type: 'Subscription',
        entity_id: subscription.id,
        action: 'status_change',
        actor_id:,
        data: {
          previous_status: subscription.status_before_last_save,
          new_status: subscription.status,
          reason: 'Customer requested cancellation'
        }
      )
    end
  end
end
```

# Gateway Migration

Gateway migration is an operational process and is executed via background batch jobs outside the public billing API.

## Migration Strategy

- Let the current gateway complete the active billing period
- Transfer billing responsibility at the period boundary
- Never overlap billing periods across gateways
- Migration via Batch Jobs

For each active customer billed via ASAAS:

### Retrieve the last confirmed payment from ASAAS
### Create the customer in *Payme*:
```ruby
customer = PayMe::Customer.create(
  external_id: "123",
  first_name: "Tony",
  last_name: "Stark",
  gateway_reference: "pagar_me", # optional
  email: "tony@starkindustries.com"
)
```
### Create subscriptions in *Payme* with:
The same `billing_interval`, and set `last_payment_at` to the last successful *ASAAS* payment
```ruby
last_payment_at = asaas.last_successful_payment(subscription)

PayMe::Subscription.create(
  customer_id: customer.id,
  amount_cents: asaas.amount_cents,
  billing_interval: "monthly", # (enum: monthly, yearly)
  last_payment_at: last_payment_at
)
```
The new gateway cannot charge before the next billing cycle and API does not need to know that a migration happened.

### Next Billing Cycle (Fully Automatic)
The billing engine calculates the next billing date based on `last_payment_at` and
`billing_interval` attributes.
A new invoice is generated when the period is due and the charge is executed by the new gateway.

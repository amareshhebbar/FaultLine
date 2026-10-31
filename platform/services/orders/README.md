# orders

Creates orders, reserves stock, asks payments to charge, and undoes everything if the charge fails.

## Responsibilities

* `POST /orders` guarded by an idempotency key
* Reserve stock with a conditional update so stock never goes below zero
* Call payments with the key `order_<id>` so retries can never double charge
* Saga with compensation: mark the order failed and give the stock back if payment fails
* Later: transactional outbox and events on the queue

## Done means

Under any retry storm, an order is either paid once or failed with its stock returned.

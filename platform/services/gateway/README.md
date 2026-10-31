# gateway

The front door. Accepts a checkout from a shopper and hands it to orders.

## Responsibilities

* `POST /checkout` with customer, item, quantity and amount
* Creates an idempotency key if the caller did not send one
* Calls orders with retries, backoff and jitter
* Turns downstream failures into clear HTTP errors

## Done means

A checkout request returns an order and a payment, and its trace starts here.

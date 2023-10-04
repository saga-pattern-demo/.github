# Hi there 👋

## This is a hypothetical organization functioning as a company. This organization is established solely to simulate the workings of the Saga pattern in the "place order" use case. 

## Table of Contents

1. [Overview](#Overview)
2. [Saga Pattern](#Saga-Pattern)
3. [Handling Stock Insufficiency](#Handling-Stock-Insufficiency)
4. [Handling Stock Insufficiency](#Handling-Stock-Insufficiency)

## Overview

This repository contains an example implementation of the Saga Pattern using a set of microservices that communicate via Kafka. The services involved in this Saga Pattern include:

1. **Order Service**: Responsible for creating orders and sending order-created events to Kafka.
2. **Order Orchestrator Service**: Orchestrates the order processing flow, sending events to Kafka and managing the order lifecycle.
3. **Payment Service**: Manages payment processing and sends events related to payment status to Kafka.
4. **Stock Service**: Handles stock management, deducting stock from inventory, and notifying order orchestrator of product reservations.

## Saga Pattern

The Saga Pattern is used to manage distributed transactions across multiple microservices. In this implementation, I'll be orchestrating the following steps:

1. **Order Creation**: The `order-service` creates an order and send an `order-created` event to Kafka containing the `OrchestratorResponseDTO`.
2. **Stock Reservation**: Upon receiving the `order-created` event, the `order-orchestrator-service` sends a `reserve-product` event to Kafka. The `stock-service` processes this event, deducts stock from the PostgreSQL database, and sends a `product-reserved` event back to Kafka.
3. **Payment Processing**: The `order-orchestrator-service` listens for the `product-reserved` event and sends a `process-payment` event to Kafka. The `payment-service` processes this event by debiting the user's balance and sending a `payment-processed` event back to Kafka.
4. **Order Update**: The `order-orchestrator-service` listens for the `payment-processed` event and sends an `order-updated` event to Kafka. Finally, the `order-service` updates the order status to **COMPLETED** or **CANCELLED** based on the payment status.

![orchestrator-pattern](https://github.com/saga-pattern-demo/.github/assets/52238180/c4c1925b-8b82-41a1-b59d-08e1de293ca5)
</br>

## Handling Stock Insufficiency

If the `stock-service` determines that there is insufficient stock to fulfill an order, it will send a `product-reserved` event with stock status **UNAVAILABLE** to Kafka. The `order-orchestrator-service` will receive this event and send an `order-updated` event with the order status set to **CANCELLED** to Kafka.

The `order-service` will then receive the `order-updated` event and update the order in the MySQL database with the status **CANCELLED**.

![orchestrator-pattern-stock-failure](https://github.com/saga-pattern-demo/.github/assets/52238180/3c12b926-32d9-470f-82f2-da5bcd0e6f55)
</br>

## Handling Payment Rejections

If the `payment-service` determines that there are insufficient funds to process the payment, it will send a `payment-processed` event with the status **PAYMENT_REJECTED** to Kafka. The `order-orchestrator-service` will receive this event and initiate the cancellation of the product reservation by sending a `cancel-product-reservation` event to Kafka. The `stock-service` will process this event by adding the previously deducted stock back to the inventory.

Subsequently, the `order-orchestrator-service` will send an `order-updated` event with the order status set to **CANCELLED** to Kafka, reflecting the payment rejection and order cancellation.

![orchestrator-pattern-payment-failure](https://github.com/saga-pattern-demo/.github/assets/52238180/60034513-d647-43ce-9252-16c132a16798)
</br>

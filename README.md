# Project Overview

## Applications

Three applications have been added to the project:

1. **Order** - Manages orders, organizes the saga, and manages the circuit breaker.
   - **Note:** It’s better to separate the saga into a dedicated service.
   - **Circuit Breaker:** Usually used separately and better as a proxy.
   - **Wallet:** Requests the user’s wallet; it makes sense to use it separately.
2. **Payment** - Manages payments.

3. **Notify** - Manages notifications.

## Workflow Diagram

![Circuit Breaker Diagram](./circuit_breaker.png)

### Workflow Steps:

1. **Order Initialization.**
2. **Payment Creation:** Order service receives the order, and the Payment service creates the payment.
3. **Payment Information Transmission:** Information about the payment is sent to the Order service.
4. **Payment Information Receipt:** Order service receives payment information.
5. **Wallet Request:** A request is made to the wallet to check for sufficient funds.
6. **Funds Confirmation:** Information about sufficient funds is sent to the Payment service.
7. **Payment Processing:** The payment is processed.
8. **Payment Confirmation:** Payment information is sent to the Order service.
9. **Payment Record:** Payment information is saved in the Order service.
10. **Notify Service:** Payment status information is sent to the Notify service.
11. **Email Notification:** An email is sent.
12. **Order Dispatch Notification:** Notification of dispatch is sent to the Order service.
13. **Saga Completion:** The saga is completed.

### Note:

Compensating transactions are provided at each step.

## Circuit Breaker Description

The circuit breaker is designed to work with a queue, listening to a separate queue where expired messages are placed.

### States of Circuit Breaker:

1. **Normal State (IsClose):**
   - Messages are freely sent to the broker.
2. **Half-Open State:**

   - Upon receiving expired messages, the circuit breaker transitions to this state, delaying messages before sending.

3. **Open State (IsOpen):**

   - After several cycles of incoming errors in the Half-Open state, it transitions to the IsOpen state.
   - It stops accepting messages and returns an error until the next start cycle.

4. **State Transitions:**
   - If there are no errors in the IsOpen and Half-Open states, the circuit breaker transitions back with each cycle:
   - **IsOpen -> Half-Open -> IsClose.**

# L402: The Missing Piece in the Internet’s Payment Infrastructure


**Designed for a world where AI and automation are essential, L402 enables seamless, machine-friendly transactions that traditional, human-centric payment flows can’t support.**

## Key Features

L402 simplifies and automates the process of handling payments on the internet, allowing seamless integration into digital workflows. It leverages HTTP as a foundation to standardize how payments are requested and processed, making it easier for AI agents and automated systems to interact with services.


- **HTTP-based flow**: simplifies payment handling by using HTTP 402 status codes and JSON payloads, allowing clients to request resources, pay and access them seamlessly.
- **Standarize payment requests**: services indicate payment requirements, making integration predictable and easier for developers to implement across different systems.
-  **Payment agnostic**: works with a variety of payment solutions, from services like Stripe to cryptocurrencies, offering developers the flexibility to use the payment network that best fits their needs.
- **Designed for automation**: enables seamless, autonomous transactions between services and AI agents, removing the need for human intervention in payment processes.
- **Extensible and open source**: built to be open and adaptable, making it easy to extend and integrate with future payment solutions and evolving technologies.

## Introduction

*L402 makes payments a core part of HTTP interactions by leveraging the HTTP 402 status code*

Payments were largely an afterthought in the internet’s original design, even though the HTTP protocol reserved the 402 “Payment Required” status code for future use. Instead, payment solutions evolved around human-centric processes and relied on networks like Visa and Mastercard, making manual checkout flows the standard. This worked well for traditional web interactions, where payments were either preconfigured or managed manually by users.

With the rise of autonomous systems and AI agents that discover and interact with services independently, the limitations of these human-centric payment flows are becoming clear. These agents interact primarily through APIs and bypass browser-based interfaces, revealing a need for a payment model that supports automated, real-time transactions without human setup.

L402 addresses this gap by making payments “machine-friendly” on the internet. Leveraging HTTP’s 402 status code and JSON payloads, L402 standardizes how services request payments directly within HTTP interactions, enabling AI agents and automated systems to handle payments as naturally as data exchange. This transforms payments into a core, automated component of the web.

## Protocol

This sequence diagram illustrates how the L402 protocol streamlines the process of handling payments for HTTP resources. 

![L402 Protocol Flow](l402-protocol-flow.svg)


The interaction begins when a client requests access to a resource and the server checks if payment is needed. If payment is required, the server responds with details, allowing the client to complete the transaction. After verifying the payment, the server grants access to the resource. This workflow enables automated and seamless payment interactions over HTTP.


1.	**Client request**: The client initiates the process by sending a request to access an HTTP resource on the server.
2. **Server payment required check**:	The server evaluates if the client has access to the resource, checking whether the client has sufficient credits/is in the right tier or if payment is required.
3.	**Server Responds with HTTP 402**: If payment is necessary and credits are insufficient, the server returns an HTTP 402 status code. This response includes a JSON body with payment details, such as the amount due and a link or instructions for completing the payment.
4.	**Client Processes Payment**: The client processes the payment using the provided details. This can be done through an external service or a payment processor. The server may later verify the payment via a webhook or other notification from the payment service.
5.	**Client Re-Requests**: After completing the payment, the client sends a new request for the HTTP resource.
6.	**Server payment required check**: The server confirms that the payment has been processed or that the client’s credits are sufficient to grant access. This verification might be based on internal checks or external confirmations, such as webhooks from a payment processor.
7.	**Server processes the request**:	Once the payment is verified, the server serves the requested resource to the client, completing the process.

### Payment request format

```json
{
  "version": "1.0",
  "offers": [
    {
      "id": "offer_12345",
      "title": "One-time Access",
      "description": "Access to the resource for a single session",
      "amount": 5.00,
      "currency": "USD",
      "payment_methods": [
        {
          "payment_type": "Stripe",
          "payment_details": {
            "payment_link": "https://checkout.stripe.com/pay/abcdef12345"
          }
        },
        {
          "payment_type": "Lightning Network",
          "payment_details": {
            "invoice": "lnbc12345...",
            "qr_code_url": "https://example.com/qrcode/lnbc12345"
          }
        }
      ]
    },
    {
      "id": "offer_67890",
      "title": "Monthly Subscription",
      "description": "Unlimited access for 30 days",
      "amount": 15.00,
      "currency": "EUR",
      "payment_methods": [
        {
          "payment_type": "Credit Card",
          "payment_details": {
            "provider": "Stripe",
            "payment_link": "https://checkout.stripe.com/pay/xyz789"
          }
        }
      ]
    }
  ],
  "expiry": "2024-11-30T23:59:59Z",
  "terms_url": "https://example.com/terms",
  "metadata": {
    "resource_id": "resource_abc",
    "client_note": "Payment required for premium content"
  }
}
```


## Contributing

We welcome contributions to L402! Here’s how you can get involved:
1.	Join our community: Connect with other contributors and stay updated on L402’s development by joining our [Discord community](https://discord.gg/2tPYBgWzQm).
2.	Report issues: If you encounter a bug or have a feature request, please open an issue on GitHub.
3.	Submit Pull Requests: If you’re ready to contribute code, fork the repository and submit a pull request. Be sure to follow our coding guidelines and include relevant tests.
4.	Documentation: Improving documentation is always appreciated. You can help by expanding explanations, adding examples, or updating outdated content.

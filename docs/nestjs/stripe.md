---
sidebar_position: 1
---

# Stripe Integration

**NOTE: This is only applicable if you're using stripe as the primary payment provider**

After almost a decade, [stripe](https://stripe.com) is still the de-facto choice of payment provider for most startups so we decided to make this our default too. In this page, you will find how we integrate with stripe from a high level overview to code-level walkthrough.

Before getting started, please make sure you have signed up on stripe and have an active account there.

## Connect

In order to process payments in marketplace type of platforms, stripe introduced a product called [Stripe Connect](https://stripe.com/connect) which is what we are using in the kit. There are 3 different types of set up that you can use with stripe connect and your choice will depend on the type of UX you want to provide for your app's users. Our kit is compatible with the first to types, out of the box and for the remaining one, you can easily switch with [some small modifications](broken_link).

## High implementation summary
In theory, out of the box, a user on our kit can be a seller and a buyer at the same time. So, at the time of registration, we don't link user accounts with stripe right away. Instead, we wait for the right moment to lazily integrate user accounts with stripe.

#### User association
Here's the implementation for the buyer
1. User signs up, acquiring a user id in the database
2. When user tries to buy a product (make a payment) or updates their account info with payment/personal details, `PUT /buyer` endpoint is called which checks if the calling user already has a stripe `customer`` association
  a. if yes, it updates the stripe `customer` with the given data through a stripe api request.
  b. if not, it creates a stripe `customer` entry through a stripe api request and saves the stripe customer id in database.
3. When the user tries to upload a sellable product or attempts to create a seller profile, stripe api is called to create a stripe `account`. However, creating the account is only half the job. The user still needs to go through stripe onboarding in order to `verify` their seller profile. For this, `POST /seller/onboarding` is called which uses the stripe api to generate an onboarding link for the user and sends it back for the client side. Once the onboarding is complete, stripe sends a webhook call to the nest.js api server and the webhook processor marks the user's seller profile as verified in database.


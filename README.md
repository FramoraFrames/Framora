# Framora — Deployment & Feature Guide

## What's in this folder
- `index.html` — full storefront, collections, favorites, cart, checkout, reviews, newsletter signup, and order-contact flow.
- `images/` — all product artwork used by the shop.

## Payment / checkout
The checkout has **one payment method only: JazzCash / EasyPaisa**. There is no Cash on Delivery and no bank-transfer selector anywhere in the checkout.

Flow:
1. Customer taps **Add to Cart** to save a frame for checkout.
2. Customer opens the cart and reviews quantities, frame total, delivery charges, and final total.
3. **Review Order & Checkout** opens the detailed delivery form.
4. Checkout shows the exact payable total and the JazzCash / EasyPaisa number and account title.
5. Customer submits the order and receives three ways to send frame photographs/design notes: **WhatsApp, Instagram DM, or Gmail/email**.

The current delivery charge is `PKR 150` per order. The bundle pricing is:
- 1 frame = PKR 700
- 2 frames = PKR 1300
- 3 frames = PKR 1800
- 4+ frames = PKR 1800 + PKR 700 for each frame after the third

Change these in `calculateBundleTotal()` and `DELIVERY_CHARGE` inside `index.html`.

## Favorites
Every product image has a heart button in the top-right corner. Visitors can save/remove favorites, and **My Favorites** filters the product grid. Favorites are stored in the visitor's browser.

## Reviews
Every product has a review form with a 1–5 star rating, name, and review text. Visitors can delete **their own reviews**. The site also has an **All Reviews** panel in the main navigation.

The site is configured to use Firebase Firestore so reviews can be shared across visitors. Firebase Anonymous Authentication is also loaded so the browser gets a private user ID; this is what allows Firestore rules to permit deletion only when the review belongs to that user.

### Firebase setup
1. Open Firebase Console and select the `framora-55f39` project used by this site.
2. Enable **Authentication → Sign-in method → Anonymous**.
3. Make sure Firestore is enabled.
4. Use rules equivalent to:

```text
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /reviews/{reviewId} {
      allow read: if true;
      allow create: if request.auth != null
        && request.resource.data.userId == request.auth.uid
        && request.resource.data.productId is string
        && request.resource.data.name is string
        && request.resource.data.text is string;
      allow delete: if request.auth != null
        && resource.data.userId == request.auth.uid;
      allow update: if false;
    }

    match /questions/{questionId} {
      allow read, create: if true;
      allow update, delete: if false;
    }

    match /subscribers/{subscriberId} {
      allow create: if request.resource.data.email is string;
      allow read, update, delete: if false;
    }
  }
}
```

Existing reviews created before the `userId` field was added will still be visible, but they cannot be securely deleted by a visitor. New reviews have the ownership field needed for self-delete.

## Gmail / email subscription
The footer has a **Get Shop Updates by Email** form. Visitors can enter a Gmail address (or another valid email) and the address is saved to the Firestore `subscribers` collection, with a local fallback if cloud storage is unavailable.

This creates your subscriber list. A browser-only website should not be given permission to send bulk email from your Gmail account. To actually send future newsletters automatically, connect the subscriber collection to an email service such as EmailJS, Brevo, Mailchimp, or a Google Apps Script/backend. The storefront itself does not expose your Gmail password or account credentials.

## Automatic order-confirmation email
EmailJS is already wired into checkout. The customer can receive an automatic confirmation email if the EmailJS service/template values in `index.html` are active and correctly configured.

## Deploy
1. Upload `index.html`, `images/`, and this `README.md` to GitHub.
2. In Netlify, import the GitHub repository.
3. Use the repository root as the publish directory and leave the build command empty.
4. Netlify will deploy the static site.

## Important business details to change later
Search `index.html` for:
- `WHATSAPP_NUMBER` — WhatsApp destination.
- `INSTAGRAM_URL` / `INSTAGRAM_DM_URL` — Instagram links.
- `SHOP_EMAIL` — email destination for customer photographs.
- `DELIVERY_CHARGE` — delivery fee.
- `jazzcashDetailsBox` — JazzCash / EasyPaisa number and account title.
- `FIREBASE_CONFIG` — Firebase project configuration.

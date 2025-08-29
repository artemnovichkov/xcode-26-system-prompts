# StoreKit Updates

## Overview

StoreKit is Apple's framework for implementing in-app purchases, subscriptions, and App Store interactions. Recent updates have introduced significant enhancements to the core framework, new SwiftUI views for merchandising subscriptions, and improved tools for testing and development.

## Core Framework Updates

### AppTransaction Updates (iOS 18.4+)

`AppTransaction` now includes two new fields:

- **appTransactionID**: A globally unique identifier for each Apple Account that downloads your app
  - Unique for each family group member for apps supporting Family Sharing
  - Back-deployed to iOS 15
  
- **originalPlatform**: Indicates the platform on which the customer originally purchased the app
  - Values include iOS, macOS, tvOS, or visionOS
  - Helps support business model changes and entitle customers appropriately

```swift
// Example of accessing AppTransaction properties
Task {
    do {
        let appTransaction = try await AppTransaction.shared
        
        // Access the new properties
        let transactionID = appTransaction.appTransactionID
        let platform = appTransaction.originalPlatform
        
        // Use these values for business logic
        if platform == .iOS {
            // Handle iOS-specific logic
        }
    } catch {
        print("Failed to get app transaction: \(error)")
    }
}
```

### Transaction Updates (iOS 18.4+)

The `Transaction` type represents a successful In-App Purchase and has been enhanced with:

- **New API**: `Transaction.currentEntitlements(for:)` replaces `Transaction.currentEntitlement(for:)`
  - Returns an asynchronous sequence of transactions entitling the customer to a given product
  - Supports multiple entitlements through different means

- **New Fields**:
  - `appTransactionID`: Links transactions to the app download
  - `offerPeriod`: Details the subscription period associated with a redeemed offer
  - `advancedCommerceInfo`: Applies only to apps using the Advanced Commerce API

```swift
// Example of using the new currentEntitlements API
Task {
    for await verificationResult in Transaction.currentEntitlements(for: "your.product.id") {
        switch verificationResult {
        case .verified(let transaction):
            // Handle verified transaction
            let appTransactionID = transaction.appTransactionID
            if let offerPeriod = transaction.offerPeriod {
                // Handle offer period information
            }
        case .unverified(let transaction, let verificationError):
            // Handle unverified transaction
            print("Verification failed: \(verificationError)")
        }
    }
}
```

### RenewalInfo Updates (iOS 18.4+)

The `RenewalInfo` type for auto-renewable subscriptions has been enhanced with:

- **Enhanced API**: `SubscriptionStatus` can now query subscription statuses using a Transaction ID
- **Four New Fields**: Providing more comprehensive insights into subscription details
- **Expiration Reasons**: Valuable for understanding customer behavior and tailoring strategies
  - Example: If a subscription expires due to a price increase, you can offer win-back promotions

```swift
// Example of accessing subscription status with a transaction ID
Task {
    do {
        let status = try await Product.SubscriptionInfo.Status(transactionID: "transaction_id_here")
        
        // Access renewal info
        if let renewalInfo = status.renewalInfo {
            // Check expiration reason if applicable
            if let expirationReason = renewalInfo.expirationReason {
                switch expirationReason {
                case .priceIncrease:
                    // Offer a win-back promotion
                case .billingError:
                    // Prompt to update payment method
                default:
                    // Handle other expiration reasons
                }
            }
        }
    } catch {
        print("Failed to get subscription status: \(error)")
    }
}
```

## StoreKit Views

### SubscriptionOfferView

A new SwiftUI view for merchandising auto-renewable subscriptions:

```swift
// Basic usage with product ID
SubscriptionOfferView(productID: "your.subscription.id")
    .prefersPromotionalIcon(true)

// Using a loaded product
SubscriptionOfferView(product: loadedSubscriptionProduct)

// With custom icon
SubscriptionOfferView(productID: "your.subscription.id") {
    Image("custom_icon")
        .resizable()
        .frame(width: 40, height: 40)
}

// With placeholder icon while loading
SubscriptionOfferView(productID: "your.subscription.id") {
    Image("custom_icon")
        .resizable()
        .frame(width: 40, height: 40)
} placeholderIcon: {
    Image(systemName: "hourglass")
        .resizable()
        .frame(width: 40, height: 40)
}
```

### Configuring SubscriptionOfferView

Add detail action to direct customers to subscription store:

```swift
SubscriptionOfferView(productID: "your.subscription.id")
    .subscriptionOfferViewDetailAction {
        // Action when detail link is tapped
        isShowingSubscriptionStore = true
    }
```

### Displaying Different Plans Based on Customer Status

Configure which subscription plan to display using the `visibleRelationship` parameter:

```swift
// Using a subscription group ID
SubscriptionOfferView(groupID: "your.group.id", visibleRelationship: .upgrade)

// Available relationships:
// - .upgrade: Shows a plan one level higher than current
// - .downgrade: Shows a plan one level lower than current
// - .crossgrade: Shows equivalent tier plans with best value
// - .current: Shows customer's current plan
// - .all: Shows all plans in the group
```

### Tracking Subscription Status

Use the `subscriptionStatusTask` modifier to determine customer status:

```swift
@main
struct MyApp: App {
    @State private var customerStatus: SubscriptionStatus = .unknown
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environment(\.customerSubscriptionStatus, customerStatus)
                .subscriptionStatusTask(for: "your.group.id") { statuses in
                    // Translate StoreKit statuses to your app's model
                    if statuses.contains(where: { $0.state == .subscribed }) {
                        customerStatus = .subscribed
                    } else if statuses.contains(where: { $0.state == .expired }) {
                        customerStatus = .expired
                    } else {
                        customerStatus = .notSubscribed
                    }
                }
        }
    }
}
```

## In-App Purchase Request Signing

StoreKit now requires JSON Web Signatures (JWS) for certain Purchase Option and View Modifier APIs:

- Setting customer eligibility for introductory offers
- Signing promotional offers

The App Store Server Library simplifies the JWS signing process:

1. Retrieve your In-App Purchase signing key from App Store Connect
2. Use the key with the App Store Server Library to create signed requests

```swift
// Example of using the App Store Server Library for signing
import AppStoreServerLibrary

// Create a signed JWS for a promotional offer
func createSignedOfferJWS(productID: String, offerID: String) async throws -> String {
    let signingKey = try SigningKey(
        privateKeyFilePath: "path/to/key.p8",
        keyID: "YOUR_KEY_ID",
        issuerID: "YOUR_ISSUER_ID"
    )
    
    let library = try AppStoreServerLibrary(
        signingKey: signingKey,
        environment: .production
    )
    
    return try library.createOfferSignature(
        productIdentifier: productID,
        subscriptionOfferID: offerID,
        applicationUsername: nil,
        nonce: UUID().uuidString,
        keyIdentifier: "YOUR_KEY_ID",
        timestamp: Int(Date().timeIntervalSince1970)
    )
}
```

## Testing and Development

### StoreKit Testing in Xcode

Create a local StoreKit configuration file to test In-App Purchases without App Store Connect setup:

1. Select File > New > File From Template
2. Search for "storekit" and select "StoreKit Configuration File"
3. Name the file (e.g., `LocalConfiguration.storekit`)
4. Define products in the configuration file

### Transaction Manager

Use the Transaction Manager window in Xcode to:

- Create and inspect transactions
- Modify transaction properties
- Test different purchase scenarios

### Testing Subscription Offers

1. Set up subscription offers in your local configuration file
2. Implement the necessary JWS signing for offers
3. Test different offer scenarios using the Transaction Manager

## Advanced Commerce API

The Advanced Commerce API enables easier support for:

- In-App Purchases for large content catalogs
- Creator experiences
- Subscriptions with optional add-ons

This API is accessible through the new `advancedCommerceInfo` field in the `Transaction` model.

## References

- [StoreKit Documentation](https://developer.apple.com/documentation/storekit)
- [What's new in StoreKit and In-App Purchase (WWDC 2025)](https://developer.apple.com/videos/play/wwdc2025/241/)
- [Getting started with In-App Purchase using StoreKit views](https://developer.apple.com/documentation/StoreKit/getting-started-with-in-app-purchases-using-storekit-views)
- [Understanding StoreKit workflows](https://developer.apple.com/documentation/StoreKit/understanding-storekit-workflows)
- [App Store Server Library on GitHub](https://github.com/apple/app-store-server-library-swift)
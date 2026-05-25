# PracticeRepository
# QuickBite ER Diagram

This project uses microservices, so each service owns its own schema.  
The diagram below is a single system-wide logical ERD that combines:

- physical table relationships inside a service
- logical cross-service references stored as IDs like `customerId`, `restaurantId`, `orderId`, and `agentId`

## Single System ERD

```mermaid
erDiagram
    USER {
        BIGINT id PK
        STRING name
        STRING email UK
        STRING phoneNumber UK
        STRING password
        ENUM role
        ENUM authProvider
        BOOLEAN isActive
    }

    PENDING_REGISTRATION {
        STRING verificationId PK
        STRING name
        STRING email
        STRING phoneNumber
        STRING passwordHash
        ENUM role
        STRING otpHash
        DATETIME expiresAt
        DATETIME createdAt
        INT failedAttempts
    }

    PENDING_LOGIN {
        STRING verificationId PK
        BIGINT userId FK
        STRING email
        STRING otpHash
        DATETIME expiresAt
        DATETIME createdAt
        INT failedAttempts
    }

    PENDING_PASSWORD_RESET {
        STRING verificationId PK
        BIGINT userId FK
        STRING email
        STRING otpHash
        DATETIME expiresAt
        DATETIME createdAt
        INT failedAttempts
    }

    RESTAURANT {
        BIGINT restaurantId PK
        BIGINT ownerId FK
        STRING name
        STRING description
        STRING cuisine
        STRING address
        STRING city
        DOUBLE latitude
        DOUBLE longitude
        STRING phone
        DOUBLE avgRating
        BIGINT ratingCount
        BOOLEAN isOpen
        BOOLEAN isApproved
        STRING rejectionReason
        DOUBLE deliveryRadius
        DOUBLE minOrderAmount
        INT estimatedDeliveryMin
    }

    MENU_CATEGORY {
        BIGINT categoryId PK
        BIGINT restaurantId FK
        STRING name
        STRING description
        STRING imageUrl
        INT displayOrder
    }

    MENU_ITEM {
        BIGINT itemId PK
        BIGINT restaurantId FK
        BIGINT categoryId FK
        STRING name
        STRING description
        DECIMAL price
        DECIMAL discountedPrice
        STRING imageUrl
        BOOLEAN isAvailable
        BOOLEAN isVeg
        DOUBLE rating
        DOUBLE calories
    }

    CART {
        BIGINT cartId PK
        BIGINT customerId FK
        BIGINT restaurantId FK
        DECIMAL totalPrice
    }

    CART_ITEM {
        BIGINT itemId PK
        BIGINT cartId FK
        BIGINT menuItemId FK
        STRING name
        INT quantity
        DECIMAL price
        STRING customization
    }

    ORDER {
        BIGINT orderId PK
        BIGINT customerId FK
        BIGINT restaurantId FK
        BIGINT deliveryAgentId FK
        DECIMAL totalAmount
        DECIMAL discount
        DECIMAL finalAmount
        ENUM paymentMode
        ENUM orderStatus
        DATETIME orderDate
        STRING deliveryAddress
        DECIMAL deliveryLatitude
        DECIMAL deliveryLongitude
        DATETIME estimatedDelivery
        STRING specialInstructions
    }

    ORDER_ITEM {
        BIGINT orderItemId PK
        BIGINT orderId FK
        BIGINT menuItemId FK
        STRING name
        DECIMAL price
        INT quantity
        STRING customization
    }

    PAYMENT {
        BIGINT paymentId PK
        BIGINT orderId FK
        BIGINT customerId FK
        DECIMAL amount
        ENUM status
        ENUM mode
        STRING transactionId
        STRING currency
        DATETIME paidAt
        DATETIME refundedAt
        STRING razorpayOrderId
        STRING razorpayPaymentId
    }

    WALLET {
        BIGINT walletId PK
        BIGINT customerId UK
        DECIMAL balance
    }

    WALLET_STATEMENT {
        BIGINT statementId PK
        BIGINT walletId FK
        DECIMAL amount
        ENUM type
        STRING description
        DATETIME createdAt
    }

    DELIVERY_AGENT {
        BIGINT agentId PK
        BIGINT userId UK
        STRING fullName
        STRING phone UK
        ENUM vehicleType
        STRING vehicleNumber UK
        DECIMAL currentLatitude
        DECIMAL currentLongitude
        BOOLEAN available
        BOOLEAN verified
        DECIMAL avgRating
        INT totalDeliveries
        INT ratingCount
        DATETIME createdAt
        DATETIME updatedAt
    }

    ACTIVE_DELIVERY {
        BIGINT id PK
        BIGINT orderId FK
        BIGINT agentId FK
        ENUM status
        STRING completionOtp
        DATETIME completionOtpGeneratedAt
        DATETIME completionOtpEmailSentAt
        DATETIME createdAt
        DATETIME updatedAt
    }

    REVIEW {
        BIGINT reviewId PK
        BIGINT orderId UK
        BIGINT customerId FK
        BIGINT restaurantId FK
        BIGINT agentId FK
        INT foodRating
        INT deliveryRating
        STRING comment
        BOOLEAN verified
        DATETIME reviewDate
        DATETIME updatedAt
    }

    MENU_ITEM_REVIEW {
        BIGINT menuItemReviewId PK
        BIGINT orderId FK
        BIGINT customerId FK
        BIGINT restaurantId FK
        BIGINT menuItemId FK
        STRING itemName
        INT rating
        STRING comment
        BOOLEAN verified
        DATETIME reviewDate
        DATETIME updatedAt
    }

    NOTIFICATION {
        BIGINT notificationId PK
        BIGINT recipientId FK
        ENUM type
        STRING title
        STRING message
        ENUM channel
        BIGINT relatedId
        STRING relatedType
        BOOLEAN isRead
        DATETIME sentAt
    }

    USER ||--o{ PENDING_LOGIN : login_otp_flow
    USER ||--o{ PENDING_PASSWORD_RESET : password_reset_flow

    USER ||--o{ RESTAURANT : owns
    RESTAURANT ||--o{ MENU_CATEGORY : has
    MENU_CATEGORY ||--o{ MENU_ITEM : contains

    USER ||--o{ CART : owns
    RESTAURANT ||--o{ CART : cart_scoped_to
    CART ||--o{ CART_ITEM : contains
    MENU_ITEM ||--o{ CART_ITEM : selected_as

    USER ||--o{ ORDER : places
    RESTAURANT ||--o{ ORDER : receives
    ORDER ||--o{ ORDER_ITEM : includes
    MENU_ITEM ||--o{ ORDER_ITEM : ordered_as

    USER ||--|| WALLET : owns
    WALLET ||--o{ WALLET_STATEMENT : records
    USER ||--o{ PAYMENT : makes
    ORDER ||--o| PAYMENT : payment_for

    USER ||--o| DELIVERY_AGENT : registers_as
    DELIVERY_AGENT ||--o{ ACTIVE_DELIVERY : handles
    ORDER ||--o{ ACTIVE_DELIVERY : fulfillment

    USER ||--o{ REVIEW : writes
    ORDER ||--o| REVIEW : reviewed_once
    RESTAURANT ||--o{ REVIEW : receives
    DELIVERY_AGENT ||--o{ REVIEW : rated_in
    REVIEW ||--o{ MENU_ITEM_REVIEW : breaks_down_into
    MENU_ITEM ||--o{ MENU_ITEM_REVIEW : receives
    ORDER ||--o{ MENU_ITEM_REVIEW : item_review_for

    USER ||--o{ NOTIFICATION : receives
    ORDER ||--o{ NOTIFICATION : related_to
```

## How To Explain It

- `Auth`, `Cart`, `Order`, `Payment`, `Delivery`, `Review`, `Restaurant`, `Menu`, and `Notification` are separate microservice databases.
- Some links are true table relationships, like:
  - `MENU_CATEGORY -> MENU_ITEM`
  - `ORDER -> ORDER_ITEM`
  - `WALLET -> WALLET_STATEMENT`
- Many other links are logical cross-service references represented by IDs, for example:
  - `ORDER.customerId -> USER.id`
  - `RESTAURANT.ownerId -> USER.id`
  - `PAYMENT.orderId -> ORDER.orderId`
  - `ACTIVE_DELIVERY.agentId -> DELIVERY_AGENT.agentId`

## Short Viva Line

If someone asks why this is a logical ERD instead of one strict database ERD, you can say:

> QuickBite follows microservices architecture, so each service owns its own schema.  
> This ER diagram represents the full business data model by combining service-local tables and cross-service ID references in one view.

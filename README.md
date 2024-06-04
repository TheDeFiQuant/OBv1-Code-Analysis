# Openbook v1

OpenBook is a decentralized, order book-based exchange built on Solana. It is a community fork of Serum, the first order book-based DEX on Solana, which was initially developed by former FTX and Alameda employees and released in August 2020. Due to concerns over compromised private keys and upgrade authority following the FTX collapse and hack, Solana's DeFi community decided to fork Serum. At the time of the fork, Serum was in version 4. OpenBook v1 represents a significant improvement over Serum v4 and is technically seen v5. The following code analysis examines the architecture, core components, and workflow of OpenBook v1.

## Architecture

OpenBook's architecture includes modules for market and instruction management, matching, clearing, settlement, account management, and event processing. The `Market` and `Instruction` modules manage the Order Management System (OMS), while the `Matching` module serves as the Matching Engine. Clearing and Settlement are handled by `Instruction` and `Crank` modules. Account Management is managed by `State` and `Accounts` modules. The `Event Queue` and `Crank` modules process orders and maintain market state. Permissioned Markets are supported by the `Permissioned` module.

### Market and Order Management

- **Market Accounts**: Each trading pair on OpenBook is represented by a market account. This account holds all the necessary information about the market, including the order book, event queue, and vaults for the assets being traded.
- **Order Book**: The order book is implemented using a data structure that efficiently manages buy and sell orders. Orders are matched based on price and time priority.
- **Order Matching Engine**: The matching engine processes new orders, matches them against existing orders in the order book, and generates trade events.

### Instructions and Transactions

- **New Order Instruction**: This instruction allows users to place new orders on the order book. It includes details such as the order type (limit or market), side (buy or sell), price, and quantity.
- **Cancel Order Instruction**: This instruction allows users to cancel their existing orders. Orders can be canceled by order ID or client order ID.
- **Consume Events Instruction**: This instruction processes the event queue, settling matched trades and updating account balances.
- **Settle Funds Instruction**: This instruction settles the funds for completed trades, moving assets from the exchange's vaults to the users' token accounts.

### Accounts and Balances

- **Open Orders Account**: Each user has an open orders account for each market they participate in. This account tracks their open orders and their reserved funds for these orders.
- **Vaults**: Each market has two vaults, one for each asset being traded. These vaults hold the funds for open orders and facilitate the settlement of trades.

### Event Queue and Cranking

- **Event Queue**: The event queue records all events (e.g., order placements, cancellations, trades) that occur in the market. This ensures that all actions are processed in order.
- **Crank Program**: The crank is an off-chain program that listens for new events on-chain and processes them. It can be run by anyone and is responsible for keeping the market state up-to-date by processing the event queue.

### Permissioned Markets

- **Middleware**: OpenBook supports permissioned markets where access to trading and other operations can be restricted based on certain criteria. Middleware components enforce these permissions.
- **Proxy Accounts**: Proxy accounts are used in permissioned markets to manage access control and ensure only authorized users can perform certain actions.

## Key Components

**State Management**
   - **Market and Orders**:
     - The `Market` struct represents a trading market with associated data like order books, event queues, and vaults.
     - `OpenOrders` and `OrderBook` manage the state of open orders and the matching of bids and asks.
   - **Queues**:
     - Event queues and request queues are used to manage and process orders and trades.

**Instructions**
   - **Order Instructions**:
     - `NewOrderInstructionV3`, `CancelOrderInstructionV2`, and other instructions define how new orders are placed, existing orders are canceled, and other market operations are performed.
   - **Market Instructions**:
     - Instructions like `InitOpenOrders`, `CloseOpenOrders`, `SettleFunds`, and `ConsumeEvents` are used to initialize, manage, and settle trades within the market.

**Matching Engine**
   - The matching engine is responsible for matching buy and sell orders.
   - It uses various algorithms and data structures (like critbit trees) to efficiently match and execute trades.

**Crank Program**
   - The crank program is an off-chain component that listens for new on-chain events (like new orders) and processes them.
   - It performs actions such as filling orders and consuming events from the queue.

## Workflow

**Initialization**
   - Market accounts are initialized with parameters like base and quote asset mints, lot sizes, initial balances and their respective trading pairs, vaults, and order books.
   - Open orders accounts are created for users to manage their orders.

**Placing Orders**
   - Users can place new buy or sell orders using the `NewOrderInstructionV3`.
   - Orders are added to the order book and matched against existing orders if possible.

**Matching Orders**
   - The matching engine continuously matches new orders with existing orders in the order book.
   - Matched orders are executed, and the respective balances are updated.

**Consuming Events**
   - The crank program processes events from the event queue, such as order fills and cancellations, settles trades, and updates account balances.
   - It ensures that the state of the market is updated in real-time.

**Settling Funds**
   - After orders are matched and trades are executed, funds are settled using the `SettleFunds` instruction.
   - This ensures that users receive their traded assets and that fees are collected.

**Cancelling and Pruning Orders**
   - Users can cancel their orders using the `CancelOrderInstructionV2` or `CancelOrderByClientIdV2` which removes them from the order book and releases reserved funds.
   - Pruning removes inactive or invalid orders from the order book to maintain efficiency.

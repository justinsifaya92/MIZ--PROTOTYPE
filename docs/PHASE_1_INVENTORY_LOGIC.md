# Phase 1: The "Circular" Inventory Logic

## Goal
To ensure every item in the company warehouse can be "turned on" for sale to others.

## Overview
Phase 1 focuses on building the core inventory management system that allows Operations Managers to control which items from company stock can be listed on the marketplace.

## Key Features

### 1. Internal to External Toggle
**Objective**: Design an Inventory UI for the Operations Manager where every item in the Company_Stock table has a "List for Sale" toggle.

**Requirements**:
- When toggled ON, the item appears in the public Marketplace
- The Operations Manager must set a Peer_Price (usually lower than retail)
- Manager must specify if the item is "Surplus" or "Dead Stock"
- Company's own project needs are prioritized before the item is sold externally
- Items cannot be listed for sale without a peer_price set

**Database Schema**:
```sql
CREATE TABLE Company_Stock (
  id UUID PRIMARY KEY,
  company_id UUID NOT NULL,
  item_name VARCHAR(255) NOT NULL,
  description TEXT,
  quantity INT NOT NULL,
  original_retail_price DECIMAL(10, 2) NOT NULL,
  peer_price DECIMAL(10, 2),
  stock_type ENUM('Surplus', 'Dead Stock') NOT NULL,
  listed_for_sale BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (company_id) REFERENCES Company(id)
);
```

### 2. Live Quote (RFQ) Engine
**Objective**: Build a Request for Quote (RFQ) system on Port 3001.

**Requirements**:
- When a buyer finds an item in the marketplace, they click "Request Quote"
- System automatically calculates Landed Cost (Item Price + Tax + Estimated Shipping via Google Maps API)
- Send notification to Seller's dashboard: "Company X wants to buy 50 units. [Accept] [Counter-Offer]"
- RFQ requests are stored and tracked

**Database Schema**:
```sql
CREATE TABLE RFQ_Request (
  id UUID PRIMARY KEY,
  listing_id UUID NOT NULL,
  buyer_company_id UUID NOT NULL,
  seller_company_id UUID NOT NULL,
  quantity_requested INT NOT NULL,
  item_price DECIMAL(10, 2) NOT NULL,
  tax DECIMAL(10, 2),
  estimated_shipping DECIMAL(10, 2),
  landed_cost DECIMAL(10, 2),
  status ENUM('Pending', 'Accepted', 'Counter-Offered', 'Declined') DEFAULT 'Pending',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (listing_id) REFERENCES Marketplace_Listing(id),
  FOREIGN KEY (buyer_company_id) REFERENCES Company(id),
  FOREIGN KEY (seller_company_id) REFERENCES Company(id)
);
```

## API Endpoints

### Toggle Item for Sale
**Endpoint**: `POST /api/inventory/toggle-sale`

**Request**:
```json
{
  "stock_id": "uuid",
  "listed_for_sale": true,
  "peer_price": 500.00,
  "stock_type": "Surplus"
}
```

**Response**:
```json
{
  "success": true,
  "message": "Item listed for sale",
  "data": {
    "id": "uuid",
    "item_name": "Laptop",
    "listed_for_sale": true,
    "peer_price": 500.00
  }
}
```

### Request Quote
**Endpoint**: `POST /api/rfq/request-quote`

**Request**:
```json
{
  "listing_id": "uuid",
  "quantity_requested": 50,
  "buyer_company_id": "uuid"
}
```

**Response**:
```json
{
  "success": true,
  "rfq_id": "uuid",
  "landed_cost": 25500.00,
  "breakdown": {
    "item_price": 25000.00,
    "tax": 250.00,
    "estimated_shipping": 250.00
  }
}
```

## Frontend Components

### Inventory UI (Operations Manager)
- Display list of all company stock items
- Toggle switch for "List for Sale"
- Input fields for Peer_Price
- Dropdown for Stock_Type (Surplus / Dead Stock)
- Confirmation dialog before listing

### RFQ Request Form (Marketplace Buyer)
- Item details display
- Quantity input field
- "Request Quote" button
- Automatic Landed Cost calculation display
- Confirmation before submission

## Notifications
- **Seller Alert**: "Company X wants to buy [quantity] units of [item_name]"
- Notification appears on Seller Dashboard
- Quick action buttons: [Accept] [Counter-Offer]

## Acceptance Criteria
- [ ] Company_Stock table created with all required fields
- [ ] RFQ_Request table created
- [ ] Toggle API endpoint functional
- [ ] RFQ request endpoint calculates landed cost correctly
- [ ] Operations Manager UI displays all inventory items
- [ ] Toggle switch updates listed_for_sale status
- [ ] Seller receives notification on RFQ request
- [ ] All validations in place (peer_price required, etc.)

## Timeline
**Estimated**: 2 weeks (Weeks 1-2)

## Dependencies
- Google Maps API (for shipping distance calculation)
- Database setup
- Backend framework (Node.js/Express recommended)
- Frontend framework (React recommended)
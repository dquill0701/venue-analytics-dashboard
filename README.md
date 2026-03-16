# Venue Analytics Dashboard - D365 Style

Enterprise-grade venue analytics dashboard with Square API integration for multi-location management.

## 🔗 Live Demo

**https://blockcell.sqprod.co/sites/venue-analytics-dashboard/**

## 🎯 Overview

This D365-style dashboard provides comprehensive analytics for multi-location venue management, featuring real-time data visualization and Square API integration for:

- **SoFi Stadium** - Los Angeles, CA
- **OKC Thunder Arena** - Oklahoma City, OK
- **Loeb Boathouse** - New York, NY

## 📊 Dashboard Features

### Key Performance Indicators (KPIs)

1. **Total Net Sales**: $4,287,450 (↑ 8.3%)
2. **Total Transactions**: 67,892 (↑ 12.1%)
3. **Average Ticket Size**: $63.14 (↓ 2.4%)
4. **Sales per Attendee**: $87.25 (↑ 5.7%)
5. **Refund/Void Rate**: 1.8% (↓ 0.3%)

### Detailed Analytics

#### 1. Net Sales by Venue & Event
- **Visualization**: Bar chart showing weekly revenue trends
- **Data Points**: Revenue by location, event type, and time period
- **Square APIs Used**:
  - `GET /v2/payments` - Payment amounts and timestamps
  - `GET /v2/orders` - Order totals and line items
  - `GET /v2/locations` - Location filtering

#### 2. Payment Tender Mix
- **Visualization**: Doughnut chart
- **Breakdown**:
  - Credit Card: 62%
  - Digital Wallet: 23%
  - Cash: 12%
  - Other: 3%
- **Square API Used**:
  - `GET /v2/payments` - tender_details field

#### 3. Concessions vs Merchandise Mix
- **Visualization**: Pie chart
- **Breakdown**:
  - Concessions: $2,847,320 (66%)
  - Merchandise: $1,440,130 (34%)
- **Square API Used**:
  - `GET /v2/catalog/list` - Category breakdown

#### 4. Top-Selling Items & Categories
- **Display**: Data table with sorting
- **Metrics**: Item name, category, units sold, revenue
- **Top 5 Items**:
  1. Premium Beer - 18,450 units - $276,750
  2. Hot Dogs - 15,230 units - $106,610
  3. Team Jersey - 3,890 units - $467,880
  4. Nachos - 12,670 units - $139,370
  5. Soft Drinks - 22,340 units - $134,040
- **Square APIs Used**:
  - `GET /v2/orders` - Line item details
  - `GET /v2/catalog/object` - Item and category info

#### 5. Transactions per Hour by Stand
- **Display**: Performance table
- **Metrics**: Average transactions/hour, peak hour, status
- **Sample Data**:
  - Section 101 Main Concourse: 127 trans/hr (Peak: 7:00 PM)
  - Club Level Premium: 156 trans/hr (Peak: 7:15 PM)
- **Square APIs Used**:
  - `GET /v2/payments` - created_at timestamp analysis
  - `GET /v2/locations` - location_id filtering
  - `GET /v2/devices` - Terminal/device location mapping

#### 6. Known Customer / Premium Guest Spend
- **Key Metrics**:
  - Total Premium Revenue: $1.2M
  - Average Premium Spend: $187
  - Premium vs Standard Ratio: 2.96x
  - Loyalty Program Members: 18,340
  - Repeat Customer Rate: 43.2%
- **Square APIs Used**:
  - `GET /v2/customers` - Customer profiles and metadata
  - `GET /v2/loyalty/accounts` - Loyalty program data
  - `GET /v2/payments` - customer_id association

#### 7. Client Scorecard vs Target
- **Metrics with Progress Bars**:
  - Revenue Target: $4.29M / $4.00M (107%) ✅
  - Transaction Volume: 67,892 / 70,000 (97%) ⚠️
  - Average Ticket: $63.14 / $65.00 (97%) ⚠️
  - Customer Satisfaction: 4.6 / 4.5 (102%) ✅
  - Refund Rate: 1.8% / <2.5% (92%) ✅
  - Premium Conversion: 9.5% / 12.0% (79%) ❌

#### 8. Venue Performance Summary
- **Comprehensive Table** showing:
  - Events count
  - Total attendees
  - Net sales
  - Average ticket
  - Sales per attendee
  - Performance status

## 🔌 Square API Integration

### APIs Used in Dashboard

#### 1. Payments API
```
GET /v2/payments
GET /v2/payments/{payment_id}
```
**Purpose**: Retrieve payment details, amounts, tender types, timestamps

**Key Fields**:
- `amount_money` - Payment amount
- `tip_money` - Tip amount
- `total_money` - Total including tips and fees
- `status` - COMPLETED, APPROVED, FAILED
- `card_details.entry_method` - EMV, CONTACTLESS, KEYED
- `source_type` - CARD, CASH, WALLET
- `created_at` - Transaction timestamp
- `location_id` - Venue identifier
- `customer_id` - Customer association

**Dashboard Usage**:
- Total net sales calculation
- Payment tender mix breakdown
- Transactions per hour analysis
- Refund/void rate tracking

---

#### 2. Orders API
```
GET /v2/orders
GET /v2/orders/search
GET /v2/orders/{order_id}
```
**Purpose**: Access order totals, line items, and order metadata

**Key Fields**:
- `total_money` - Order total
- `line_items[]` - Individual items purchased
- `line_items[].name` - Item name
- `line_items[].quantity` - Quantity sold
- `line_items[].catalog_object_id` - Link to catalog
- `state` - OPEN, COMPLETED, CANCELED
- `customer_id` - Customer association

**Dashboard Usage**:
- Average ticket size calculation
- Top-selling items tracking
- Category breakdown
- Sales per attendee metrics

---

#### 3. Locations API
```
GET /v2/locations
GET /v2/locations/{location_id}
```
**Purpose**: Filter data by venue and manage location metadata

**Key Fields**:
- `id` - Location identifier
- `name` - Venue name
- `business_name` - Business display name
- `address` - Full address details
- `timezone` - Location timezone
- `status` - ACTIVE, INACTIVE
- `capabilities[]` - Available features

**Dashboard Usage**:
- Multi-location filtering
- Net sales by venue
- Location-specific performance
- Venue comparison metrics

---

#### 4. Catalog API
```
GET /v2/catalog/list
GET /v2/catalog/object/{object_id}
```
**Purpose**: Product and category management

**Key Fields**:
- `item_data.name` - Item name
- `item_data.category_id` - Category assignment
- `item_variation_data.price_money` - Item pricing
- `category_data.name` - Category name

**Dashboard Usage**:
- Concessions vs merchandise mix
- Top-selling items and categories
- Product performance analysis

---

#### 5. Customers API
```
GET /v2/customers
GET /v2/customers/{customer_id}
GET /v2/customers/search
```
**Purpose**: Customer profiles and purchase history

**Key Fields**:
- `id` - Customer identifier
- `given_name`, `family_name` - Customer name
- `email_address` - Contact email
- `phone_number` - Contact phone
- `created_at` - Customer since date
- `segment_ids[]` - Customer segments

**Dashboard Usage**:
- Known customer identification
- Premium guest spend tracking
- Repeat customer rate
- Customer segmentation

---

#### 6. Loyalty API
```
GET /v2/loyalty/accounts
GET /v2/loyalty/accounts/search
GET /v2/loyalty/accounts/{account_id}
```
**Purpose**: Loyalty program data and member tracking

**Key Fields**:
- `id` - Loyalty account ID
- `balance` - Current points balance
- `lifetime_points` - Total points earned
- `customer_id` - Associated customer
- `enrolled_at` - Enrollment date

**Dashboard Usage**:
- Premium member identification
- Loyalty program member count
- Average premium spend calculation
- Member vs non-member comparison

---

#### 7. Devices API
```
GET /v2/devices
GET /v2/devices/{device_id}
```
**Purpose**: Terminal and device location mapping

**Key Fields**:
- `id` - Device identifier
- `name` - Device name/label
- `device_code.name` - Device code name
- `device_code.location_id` - Location association
- `device_code.status` - PAIRED, UNPAIRED

**Dashboard Usage**:
- Transactions per hour by stand
- Device-level performance tracking
- Location-device mapping

---

#### 8. Refunds API
```
GET /v2/refunds
GET /v2/refunds/{refund_id}
```
**Purpose**: Refund tracking and void rate calculation

**Key Fields**:
- `id` - Refund identifier
- `amount_money` - Refund amount
- `reason` - Refund reason
- `payment_id` - Original payment
- `status` - Refund status
- `created_at` - Refund timestamp

**Dashboard Usage**:
- Refund/void rate calculation
- Refund trend analysis
- Financial reconciliation

---

## 💻 Technical Implementation

### Sample API Integration Code

#### Fetch Payments for Revenue Calculation
```javascript
async function fetchPayments(locationId, startDate, endDate) {
  const url = `https://connect.squareup.com/v2/payments?location_id=${locationId}&begin_time=${startDate}&end_time=${endDate}`;
  
  const response = await fetch(url, {
    method: 'GET',
    headers: {
      'Square-Version': '2024-01-18',
      'Authorization': `Bearer ${accessToken}`,
      'Content-Type': 'application/json'
    }
  });
  
  const data = await response.json();
  
  // Calculate total revenue
  const totalRevenue = data.payments.reduce((sum, payment) => {
    return sum + payment.amount_money.amount;
  }, 0) / 100; // Convert cents to dollars
  
  return totalRevenue;
}
```

#### Fetch Orders for Top-Selling Items
```javascript
async function fetchTopSellingItems(locationId) {
  const url = 'https://connect.squareup.com/v2/orders/search';
  
  const response = await fetch(url, {
    method: 'POST',
    headers: {
      'Square-Version': '2024-01-18',
      'Authorization': `Bearer ${accessToken}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      location_ids: [locationId],
      query: {
        filter: {
          state_filter: {
            states: ['COMPLETED']
          }
        }
      }
    })
  });
  
  const data = await response.json();
  
  // Aggregate line items
  const itemCounts = {};
  data.orders.forEach(order => {
    order.line_items.forEach(item => {
      if (!itemCounts[item.name]) {
        itemCounts[item.name] = {
          quantity: 0,
          revenue: 0
        };
      }
      itemCounts[item.name].quantity += parseInt(item.quantity);
      itemCounts[item.name].revenue += item.total_money.amount;
    });
  });
  
  // Sort by quantity
  const topItems = Object.entries(itemCounts)
    .sort(([,a], [,b]) => b.quantity - a.quantity)
    .slice(0, 5);
  
  return topItems;
}
```

#### Fetch Customer Loyalty Data
```javascript
async function fetchPremiumCustomers() {
  const url = 'https://connect.squareup.com/v2/loyalty/accounts/search';
  
  const response = await fetch(url, {
    method: 'POST',
    headers: {
      'Square-Version': '2024-01-18',
      'Authorization': `Bearer ${accessToken}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      query: {
        mappings: [
          {
            type: 'PHONE'
          }
        ]
      }
    })
  });
  
  const data = await response.json();
  
  // Calculate premium metrics
  const premiumMembers = data.loyalty_accounts.filter(
    account => account.balance >= 500
  );
  
  return {
    totalMembers: data.loyalty_accounts.length,
    premiumMembers: premiumMembers.length,
    avgBalance: data.loyalty_accounts.reduce(
      (sum, acc) => sum + acc.balance, 0
    ) / data.loyalty_accounts.length
  };
}
```

## 🚀 Getting Started

### Prerequisites
- Square Developer Account
- API Access Token (Sandbox or Production)
- Modern web browser

### Setup

1. **Clone the repository**
```bash
git clone https://github.com/YOUR_USERNAME/venue-analytics-dashboard.git
cd venue-analytics-dashboard
```

2. **Get Square API Credentials**
   - Visit [Square Developer Dashboard](https://developer.squareup.com/apps)
   - Create or select your application
   - Navigate to "Credentials" tab
   - Copy your access token

3. **Configure API Access**
   - Update the API calls in `index.html` with your access token
   - For production, use environment variables or a backend proxy

4. **Run Locally**
```bash
# Using Python
python -m http.server 8000

# Or using Node.js
npx http-server -p 8000

# Navigate to http://localhost:8000
```

### Deploy to BlockCell

```bash
# Upload index.html to BlockCell
# The dashboard is already live at:
# https://blockcell.sqprod.co/sites/venue-analytics-dashboard/
```

## 🎨 Customization

### Update Branding
Edit the header section in `index.html`:
```html
<div class="logo">
    <div class="logo-icon">◼</div>
    <span>Dynamics D365</span>
</div>
```

### Modify Locations
Update the location data in the JavaScript section:
```javascript
const locations = [
  { id: 'LOCATION_ID_1', name: 'SoFi Stadium', city: 'Los Angeles' },
  { id: 'LOCATION_ID_2', name: 'OKC Thunder Arena', city: 'Oklahoma City' },
  { id: 'LOCATION_ID_3', name: 'Loeb Boathouse', city: 'New York' }
];
```

### Adjust Color Scheme
Modify CSS variables:
```css
:root {
  --primary-color: #0078d4;
  --secondary-color: #107c10;
  --accent-color: #d13438;
}
```

## 📊 Data Refresh Rates

- **Real-time**: Payments, Orders (as they occur)
- **Near real-time**: Customer data (within minutes)
- **Periodic**: Loyalty points (hourly sync)
- **Batch**: Historical reports (daily aggregation)

## 🔐 Security Best Practices

1. **Never commit API tokens** to version control
2. **Use environment variables** for sensitive data
3. **Implement server-side proxy** for production
4. **Enable HTTPS** for all API calls
5. **Validate and sanitize** all user inputs
6. **Implement rate limiting** to prevent abuse

## 🐛 Troubleshooting

### Common Issues

**CORS Errors**
- Square APIs require server-side calls
- Use a backend proxy for production
- BlockCell handles CORS automatically

**Authentication Errors**
- Verify access token is valid
- Check token permissions in Square Dashboard
- Ensure correct API version header

**Data Not Loading**
- Check browser console for errors
- Verify location IDs are correct
- Ensure API endpoints are accessible

## 📚 Additional Resources

- [Square Developer Documentation](https://developer.squareup.com/docs)
- [Square API Reference](https://developer.squareup.com/reference/square)
- [Square API Explorer](https://developer.squareup.com/explorer/square)
- [Chart.js Documentation](https://www.chartjs.org/docs/latest/)

## 🤝 Contributing

To iterate on the APIs and endpoints:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test with Square Sandbox
5. Submit a pull request

See [CONTRIBUTING.md](CONTRIBUTING.md) for detailed iteration guide.

## 📝 License

This project is provided as-is for demonstration purposes.

## 👥 Contact

- **User**: Tom Jeager
- **Company**: Legends Global
- **Dashboard**: Dynamics D365

## 🔄 Version History

- **v1.0.0** (2024-03-16)
  - Initial release
  - D365-style interface
  - Full Square API integration
  - Multi-location support
  - 10 comprehensive metrics
  - Real-time data visualization

---

**Built with Square APIs | Powered by BlockCell**
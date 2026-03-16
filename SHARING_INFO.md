# Venue Analytics Dashboard - Sharing Information

## 🔗 Quick Links

### Live Dashboard
**https://blockcell.sqprod.co/sites/venue-analytics-dashboard/**

### GitHub Repository
**https://github.com/dquill0701/venue-analytics-dashboard**

---

## 📋 For Your Colleague

Hi! This dashboard has been set up with comprehensive Square API integration documentation. Here's what you need to know:

### What's Included

1. **Live Dashboard** - Fully functional D365-style interface
2. **Complete API Documentation** - All Square endpoints used
3. **Iteration Guide** - Step-by-step instructions for updating APIs
4. **Sample Code** - Real implementation examples

### Key Files to Review

| File | Purpose |
|------|---------|
| `README.md` | Overview, features, and API endpoints |
| `API_ITERATION_GUIDE.md` | Detailed guide for iterating on APIs |
| `index.html` | The actual dashboard code |

### Dashboard Metrics & APIs

The dashboard displays 10 key metrics using these Square APIs:

#### 1. Total Net Sales ($4.29M)
- **API**: `GET /v2/payments`
- **Fields**: `amount_money.amount`, `status`
- **Calculation**: Sum of all completed payments

#### 2. Total Transactions (67,892)
- **API**: `GET /v2/payments`
- **Calculation**: Count of payment records

#### 3. Average Ticket Size ($63.14)
- **API**: `GET /v2/orders`
- **Fields**: `total_money.amount`
- **Calculation**: Total order value / Number of orders

#### 4. Sales per Attendee ($87.25)
- **APIs**: `GET /v2/payments` + Event attendance data
- **Calculation**: Total sales / Total attendees

#### 5. Refund/Void Rate (1.8%)
- **APIs**: `GET /v2/refunds`, `GET /v2/payments`
- **Calculation**: (Refund count / Total transactions) × 100

#### 6. Payment Tender Mix
- **API**: `GET /v2/payments`
- **Fields**: `source_type`, `card_details.entry_method`
- **Breakdown**: Credit (62%), Digital Wallet (23%), Cash (12%), Other (3%)

#### 7. Concessions vs Merchandise (66% / 34%)
- **APIs**: `GET /v2/catalog/list`, `GET /v2/orders`
- **Fields**: `item_data.category_id`, `line_items[]`

#### 8. Top-Selling Items
- **APIs**: `GET /v2/orders`, `GET /v2/catalog/object`
- **Fields**: `line_items[].name`, `line_items[].quantity`, `line_items[].total_money`

#### 9. Transactions per Hour by Stand
- **APIs**: `GET /v2/payments`, `GET /v2/devices`
- **Fields**: `created_at`, `device_id`
- **Calculation**: Group by device and hour

#### 10. Premium Guest Spend ($1.2M)
- **APIs**: `GET /v2/customers`, `GET /v2/loyalty/accounts`, `GET /v2/payments`
- **Fields**: `balance`, `customer_id`, `amount_money`

---

## 🚀 Getting Started with Iteration

### Step 1: Clone the Repository
```bash
git clone https://github.com/dquill0701/venue-analytics-dashboard.git
cd venue-analytics-dashboard
```

### Step 2: Get Square API Credentials
1. Visit [Square Developer Dashboard](https://developer.squareup.com/apps)
2. Create or select your application
3. Go to "Credentials" tab
4. Copy your **Sandbox Access Token** for testing

### Step 3: Review the Documentation
- Start with `README.md` for overview
- Read `API_ITERATION_GUIDE.md` for detailed implementation
- Check the code in `index.html` to see current structure

### Step 4: Test with Sandbox
```javascript
// Use Sandbox for testing
const config = {
  squareAccessToken: 'YOUR_SANDBOX_TOKEN',
  baseURL: 'https://connect.squareupsandbox.com/v2',
  squareVersion: '2024-01-18'
};
```

---

## 📊 Current API Endpoints Used

### Payments API
```
GET /v2/payments
GET /v2/payments/{payment_id}
```
**Used for**: Total sales, transaction count, tender mix, refund rate

### Orders API
```
GET /v2/orders/search
POST /v2/orders/search
```
**Used for**: Average ticket, top items, category breakdown

### Locations API
```
GET /v2/locations
GET /v2/locations/{location_id}
```
**Used for**: Multi-location filtering, venue performance

### Catalog API
```
GET /v2/catalog/list
GET /v2/catalog/object/{object_id}
```
**Used for**: Product categories, item details

### Customers API
```
GET /v2/customers
POST /v2/customers/search
```
**Used for**: Customer identification, known customer spend

### Loyalty API
```
POST /v2/loyalty/accounts/search
GET /v2/loyalty/accounts/{account_id}
```
**Used for**: Premium member identification, loyalty metrics

### Devices API
```
GET /v2/devices
GET /v2/devices/{device_id}
```
**Used for**: Device-level analytics, transactions by stand

### Refunds API
```
GET /v2/refunds
```
**Used for**: Refund rate calculation, void tracking

---

## 🔧 How to Iterate on APIs

### Example: Updating the Total Sales Calculation

**Current** (mock data):
```javascript
document.getElementById('totalRevenue').textContent = '$4,287,450';
```

**Updated** (real API):
```javascript
async function updateTotalSales() {
  const payments = await fetch('https://connect.squareup.com/v2/payments', {
    headers: {
      'Square-Version': '2024-01-18',
      'Authorization': 'Bearer YOUR_ACCESS_TOKEN',
      'Content-Type': 'application/json'
    }
  });
  
  const data = await payments.json();
  
  const totalSales = data.payments
    .filter(p => p.status === 'COMPLETED')
    .reduce((sum, p) => sum + p.amount_money.amount, 0) / 100;
  
  document.getElementById('totalRevenue').textContent = 
    `$${totalSales.toLocaleString()}`;
}

// Call on page load
updateTotalSales();
```

### More Examples

See `API_ITERATION_GUIDE.md` for complete examples of:
- Calculating average ticket size
- Getting top-selling items
- Computing payment tender mix
- Tracking transactions per hour
- Identifying premium customers
- And more!

---

## 📚 Additional Resources

### Square Documentation
- [Square Developer Docs](https://developer.squareup.com/docs)
- [API Reference](https://developer.squareup.com/reference/square)
- [API Explorer](https://developer.squareup.com/explorer/square) - Test APIs in browser
- [Developer Forums](https://developer.squareup.com/forums)

### Testing
- **Sandbox Environment**: `https://connect.squareupsandbox.com/v2`
- **Test Cards**: Use `4111 1111 1111 1111` for successful test payments
- **API Explorer**: Test endpoints without writing code

### Support
- Square Developer Community: https://developer.squareup.com/forums
- GitHub Issues: https://github.com/dquill0701/venue-analytics-dashboard/issues

---

## 🎯 Common Iteration Tasks

### Task 1: Replace Mock Data with Real API
**Time**: 2-4 hours  
**Difficulty**: Medium  
**Guide**: See API_ITERATION_GUIDE.md, Section "Phase 1"

### Task 2: Add Date Range Filtering
**Time**: 1-2 hours  
**Difficulty**: Easy  
**Guide**: See API_ITERATION_GUIDE.md, "Step 1: Set Up API Configuration"

### Task 3: Implement Real-Time Updates
**Time**: 4-6 hours  
**Difficulty**: Hard  
**Guide**: Requires webhooks setup (see Square Webhooks documentation)

### Task 4: Add New Metrics
**Time**: 2-3 hours per metric  
**Difficulty**: Medium  
**Guide**: See API_ITERATION_GUIDE.md, "Metric-by-Metric Implementation"

---

## 💬 Questions?

If you have questions about:
- **API endpoints**: Check `README.md` or `API_ITERATION_GUIDE.md`
- **Implementation**: See code examples in `API_ITERATION_GUIDE.md`
- **Square APIs**: Visit [Square Developer Docs](https://developer.squareup.com/docs)
- **Dashboard features**: Review `index.html` source code

---

## 📝 Notes for Iteration

### Current State
- Dashboard uses **mock data** for demonstration
- All metrics are **calculated values** based on sample data
- **Square API endpoints** are documented but not actively called

### Next Steps
1. Set up Square Developer account
2. Get Sandbox access token
3. Replace mock data with API calls (start with one metric)
4. Test thoroughly in Sandbox
5. Deploy to production when ready

### Important Considerations
- **Rate Limits**: Square APIs have rate limits (500 req/10sec production)
- **Authentication**: Never commit access tokens to Git
- **CORS**: Square APIs require server-side calls for production
- **Pagination**: Handle paginated responses for large datasets
- **Error Handling**: Implement retry logic for failed API calls

---

## 🎉 Ready to Start?

1. Clone the repo
2. Read the documentation
3. Get your Square API credentials
4. Start with one metric
5. Test in Sandbox
6. Iterate and improve!

**Happy coding! 🚀**

---

**Dashboard**: https://blockcell.sqprod.co/sites/venue-analytics-dashboard/  
**GitHub**: https://github.com/dquill0701/venue-analytics-dashboard  
**Created**: March 16, 2024  
**User**: Tom Jeager | Dynamics D365
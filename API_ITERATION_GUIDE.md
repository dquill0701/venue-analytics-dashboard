# API Iteration Guide - Venue Analytics Dashboard

Complete guide for iterating on Square API endpoints and improving the dashboard.

## 🎯 Quick Reference: API Endpoints by Metric

| Dashboard Metric | Primary API Endpoint | Response Field | Calculation |
|------------------|---------------------|----------------|-------------|
| Total Net Sales | `GET /v2/payments` | `amount_money.amount` | Sum all payments |
| Total Transactions | `GET /v2/payments` | Count of payments | Count records |
| Average Ticket Size | `GET /v2/orders` | `total_money.amount` | Total / Count |
| Sales per Attendee | `GET /v2/payments` + Attendance | `amount_money.amount` | Total Sales / Attendees |
| Refund Rate | `GET /v2/refunds` | Count | (Refunds / Payments) × 100 |
| Payment Tender Mix | `GET /v2/payments` | `source_type`, `card_details` | Group by tender type |
| Top-Selling Items | `GET /v2/orders` | `line_items[]` | Aggregate by item |
| Transactions/Hour | `GET /v2/payments` | `created_at`, `device_id` | Group by hour & device |
| Premium Guest Spend | `GET /v2/loyalty/accounts` | `balance`, `customer_id` | Filter premium members |
| Location Performance | `GET /v2/locations` + payments | `location_id` | Aggregate by location |

---

## 📋 Iteration Checklist

### Phase 1: Replace Mock Data with Real API Calls

#### Step 1: Set Up API Configuration
```javascript
// config.js (DO NOT COMMIT THIS FILE)
const config = {
  squareAccessToken: 'YOUR_ACCESS_TOKEN_HERE',
  squareVersion: '2024-01-18',
  baseURL: 'https://connect.squareup.com/v2', // or sandbox
  locationIds: {
    sofi: 'LOCATION_ID_SOFI',
    okc: 'LOCATION_ID_OKC',
    loeb: 'LOCATION_ID_LOEB'
  }
};
```

#### Step 2: Create API Service Layer
```javascript
// services/squareAPI.js
class SquareAPI {
  constructor(config) {
    this.config = config;
    this.headers = {
      'Square-Version': config.squareVersion,
      'Authorization': `Bearer ${config.squareAccessToken}`,
      'Content-Type': 'application/json'
    };
  }

  async request(endpoint, options = {}) {
    const url = `${this.config.baseURL}${endpoint}`;
    const response = await fetch(url, {
      ...options,
      headers: this.headers
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(`API Error: ${error.errors[0].detail}`);
    }

    return await response.json();
  }

  // Payments
  async getPayments(params = {}) {
    const queryString = new URLSearchParams(params).toString();
    return await this.request(`/payments?${queryString}`);
  }

  async getPayment(paymentId) {
    return await this.request(`/payments/${paymentId}`);
  }

  // Orders
  async searchOrders(body) {
    return await this.request('/orders/search', {
      method: 'POST',
      body: JSON.stringify(body)
    });
  }

  // Locations
  async getLocations() {
    return await this.request('/locations');
  }

  async getLocation(locationId) {
    return await this.request(`/locations/${locationId}`);
  }

  // Catalog
  async listCatalog(types = 'ITEM,CATEGORY') {
    return await this.request(`/catalog/list?types=${types}`);
  }

  // Customers
  async getCustomers(params = {}) {
    const queryString = new URLSearchParams(params).toString();
    return await this.request(`/customers?${queryString}`);
  }

  async searchCustomers(body) {
    return await this.request('/customers/search', {
      method: 'POST',
      body: JSON.stringify(body)
    });
  }

  // Loyalty
  async searchLoyaltyAccounts(body) {
    return await this.request('/loyalty/accounts/search', {
      method: 'POST',
      body: JSON.stringify(body)
    });
  }

  // Devices
  async getDevices() {
    return await this.request('/devices');
  }

  // Refunds
  async getRefunds(params = {}) {
    const queryString = new URLSearchParams(params).toString();
    return await this.request(`/refunds?${queryString}`);
  }
}

// Initialize
const api = new SquareAPI(config);
```

---

## 🔄 Metric-by-Metric Implementation Guide

### Metric 1: Total Net Sales

**Current**: Mock data showing $4,287,450

**API Implementation**:
```javascript
async function calculateTotalNetSales(locationIds, startDate, endDate) {
  let totalSales = 0;
  
  for (const locationId of locationIds) {
    const payments = await api.getPayments({
      location_id: locationId,
      begin_time: startDate,
      end_time: endDate
    });
    
    // Sum all completed payments
    const locationSales = payments.payments
      .filter(p => p.status === 'COMPLETED')
      .reduce((sum, payment) => {
        return sum + payment.amount_money.amount;
      }, 0);
    
    totalSales += locationSales;
  }
  
  // Convert from cents to dollars
  return totalSales / 100;
}

// Usage
const startDate = '2024-03-01T00:00:00Z';
const endDate = '2024-03-31T23:59:59Z';
const locationIds = Object.values(config.locationIds);

const totalNetSales = await calculateTotalNetSales(locationIds, startDate, endDate);
document.getElementById('totalRevenue').textContent = 
  `$${totalNetSales.toLocaleString()}`;
```

**Testing**:
```javascript
// Test with sandbox
const testSales = await calculateTotalNetSales(
  ['SANDBOX_LOCATION_ID'],
  '2024-01-01T00:00:00Z',
  '2024-01-31T23:59:59Z'
);
console.log('Test Sales:', testSales);
```

---

### Metric 2: Average Ticket Size

**Current**: Mock data showing $63.14

**API Implementation**:
```javascript
async function calculateAverageTicket(locationIds, startDate, endDate) {
  const searchBody = {
    location_ids: locationIds,
    query: {
      filter: {
        date_time_filter: {
          created_at: {
            start_at: startDate,
            end_at: endDate
          }
        },
        state_filter: {
          states: ['COMPLETED']
        }
      }
    },
    limit: 100
  };
  
  let allOrders = [];
  let cursor = null;
  
  // Paginate through all orders
  do {
    if (cursor) {
      searchBody.cursor = cursor;
    }
    
    const response = await api.searchOrders(searchBody);
    allOrders = allOrders.concat(response.orders || []);
    cursor = response.cursor;
  } while (cursor);
  
  // Calculate average
  const totalAmount = allOrders.reduce((sum, order) => {
    return sum + order.total_money.amount;
  }, 0);
  
  const avgTicket = totalAmount / allOrders.length / 100;
  return avgTicket;
}

// Usage
const avgTicket = await calculateAverageTicket(locationIds, startDate, endDate);
document.getElementById('avgTicket').textContent = 
  `$${avgTicket.toFixed(2)}`;
```

---

### Metric 3: Top-Selling Items

**Current**: Mock data with 5 items

**API Implementation**:
```javascript
async function getTopSellingItems(locationIds, startDate, endDate, limit = 5) {
  // First, get all orders
  const searchBody = {
    location_ids: locationIds,
    query: {
      filter: {
        date_time_filter: {
          created_at: {
            start_at: startDate,
            end_at: endDate
          }
        },
        state_filter: {
          states: ['COMPLETED']
        }
      }
    }
  };
  
  let allOrders = [];
  let cursor = null;
  
  do {
    if (cursor) searchBody.cursor = cursor;
    const response = await api.searchOrders(searchBody);
    allOrders = allOrders.concat(response.orders || []);
    cursor = response.cursor;
  } while (cursor);
  
  // Aggregate line items
  const itemStats = {};
  
  allOrders.forEach(order => {
    order.line_items?.forEach(item => {
      const key = item.catalog_object_id || item.name;
      
      if (!itemStats[key]) {
        itemStats[key] = {
          name: item.name,
          catalogId: item.catalog_object_id,
          quantity: 0,
          revenue: 0,
          category: null
        };
      }
      
      itemStats[key].quantity += parseInt(item.quantity);
      itemStats[key].revenue += item.total_money.amount;
    });
  });
  
  // Get catalog info for categories
  const catalog = await api.listCatalog('ITEM,CATEGORY');
  const categories = {};
  
  catalog.objects?.forEach(obj => {
    if (obj.type === 'CATEGORY') {
      categories[obj.id] = obj.category_data.name;
    }
  });
  
  catalog.objects?.forEach(obj => {
    if (obj.type === 'ITEM' && itemStats[obj.id]) {
      const categoryId = obj.item_data.category_id;
      itemStats[obj.id].category = categories[categoryId] || 'Uncategorized';
    }
  });
  
  // Sort by quantity and get top items
  const topItems = Object.values(itemStats)
    .sort((a, b) => b.quantity - a.quantity)
    .slice(0, limit)
    .map(item => ({
      ...item,
      revenue: item.revenue / 100 // Convert to dollars
    }));
  
  return topItems;
}

// Usage and display
const topItems = await getTopSellingItems(locationIds, startDate, endDate, 5);

const tableBody = document.querySelector('#topItemsTable tbody');
tableBody.innerHTML = topItems.map(item => `
  <tr>
    <td>${item.name}</td>
    <td>${item.category}</td>
    <td>${item.quantity.toLocaleString()}</td>
    <td>$${item.revenue.toLocaleString()}</td>
  </tr>
`).join('');
```

---

### Metric 4: Payment Tender Mix

**Current**: Mock percentages (62% Credit, 23% Digital, 12% Cash)

**API Implementation**:
```javascript
async function calculateTenderMix(locationIds, startDate, endDate) {
  let allPayments = [];
  
  for (const locationId of locationIds) {
    const response = await api.getPayments({
      location_id: locationId,
      begin_time: startDate,
      end_time: endDate
    });
    allPayments = allPayments.concat(response.payments || []);
  }
  
  // Count by tender type
  const tenderCounts = {
    'Credit Card': 0,
    'Digital Wallet': 0,
    'Cash': 0,
    'Other': 0
  };
  
  allPayments.forEach(payment => {
    if (payment.source_type === 'CARD') {
      if (payment.card_details?.entry_method === 'CONTACTLESS') {
        tenderCounts['Digital Wallet']++;
      } else {
        tenderCounts['Credit Card']++;
      }
    } else if (payment.source_type === 'CASH') {
      tenderCounts['Cash']++;
    } else {
      tenderCounts['Other']++;
    }
  });
  
  // Calculate percentages
  const total = allPayments.length;
  const tenderMix = Object.entries(tenderCounts).map(([type, count]) => ({
    type,
    count,
    percentage: ((count / total) * 100).toFixed(1)
  }));
  
  return tenderMix;
}

// Update chart
const tenderMix = await calculateTenderMix(locationIds, startDate, endDate);

// Update Chart.js doughnut chart
tenderChart.data.datasets[0].data = tenderMix.map(t => t.percentage);
tenderChart.update();
```

---

### Metric 5: Transactions per Hour by Stand

**Current**: Mock data by location/stand

**API Implementation**:
```javascript
async function calculateTransactionsPerHour(locationId, date) {
  const startDate = `${date}T00:00:00Z`;
  const endDate = `${date}T23:59:59Z`;
  
  // Get all payments for the day
  const payments = await api.getPayments({
    location_id: locationId,
    begin_time: startDate,
    end_time: endDate
  });
  
  // Get device info
  const devices = await api.getDevices();
  const deviceMap = {};
  devices.devices?.forEach(device => {
    deviceMap[device.id] = device.name || device.device_code?.name || 'Unknown';
  });
  
  // Group by device and hour
  const deviceHourly = {};
  
  payments.payments?.forEach(payment => {
    const deviceId = payment.device_id || 'unknown';
    const deviceName = deviceMap[deviceId] || 'Unknown Device';
    const hour = new Date(payment.created_at).getHours();
    
    if (!deviceHourly[deviceName]) {
      deviceHourly[deviceName] = {
        hourly: {},
        total: 0
      };
    }
    
    deviceHourly[deviceName].hourly[hour] = 
      (deviceHourly[deviceName].hourly[hour] || 0) + 1;
    deviceHourly[deviceName].total++;
  });
  
  // Calculate averages and peak hours
  const standMetrics = Object.entries(deviceHourly).map(([name, data]) => {
    const hours = Object.keys(data.hourly).length;
    const avgPerHour = Math.round(data.total / hours);
    
    const peakHour = Object.entries(data.hourly)
      .sort(([,a], [,b]) => b - a)[0];
    
    return {
      stand: name,
      avgTransPerHour: avgPerHour,
      peakHour: `${peakHour[0]}:00`,
      peakCount: peakHour[1],
      status: avgPerHour > 100 ? 'Optimal' : 'Below Target'
    };
  });
  
  return standMetrics;
}

// Usage
const standMetrics = await calculateTransactionsPerHour(
  config.locationIds.sofi,
  '2024-03-15'
);

// Update table
const tableBody = document.querySelector('#standMetricsTable tbody');
tableBody.innerHTML = standMetrics.map(stand => `
  <tr>
    <td>${stand.stand}</td>
    <td>${stand.avgTransPerHour}</td>
    <td>${stand.peakHour}</td>
    <td><span class="status-badge status-${stand.status.toLowerCase().replace(' ', '-')}">${stand.status}</span></td>
  </tr>
`).join('');
```

---

### Metric 6: Premium Guest Spend

**Current**: Mock data ($1.2M, $187 avg)

**API Implementation**:
```javascript
async function calculatePremiumGuestSpend(startDate, endDate) {
  // Get all loyalty accounts
  const loyaltyResponse = await api.searchLoyaltyAccounts({
    query: {}
  });
  
  // Define premium threshold (e.g., 500+ points)
  const premiumThreshold = 500;
  const premiumCustomerIds = loyaltyResponse.loyalty_accounts
    ?.filter(account => account.balance >= premiumThreshold)
    .map(account => account.customer_id) || [];
  
  // Get payments for premium customers
  let premiumPayments = [];
  
  for (const locationId of Object.values(config.locationIds)) {
    const payments = await api.getPayments({
      location_id: locationId,
      begin_time: startDate,
      end_time: endDate
    });
    
    const locationPremiumPayments = payments.payments?.filter(payment =>
      premiumCustomerIds.includes(payment.customer_id)
    ) || [];
    
    premiumPayments = premiumPayments.concat(locationPremiumPayments);
  }
  
  // Calculate metrics
  const totalPremiumRevenue = premiumPayments.reduce((sum, payment) => {
    return sum + payment.amount_money.amount;
  }, 0) / 100;
  
  const avgPremiumSpend = premiumPayments.length > 0
    ? totalPremiumRevenue / premiumPayments.length
    : 0;
  
  // Get standard customer spend for comparison
  const allPayments = await api.getPayments({
    begin_time: startDate,
    end_time: endDate
  });
  
  const standardPayments = allPayments.payments?.filter(payment =>
    !premiumCustomerIds.includes(payment.customer_id)
  ) || [];
  
  const avgStandardSpend = standardPayments.length > 0
    ? standardPayments.reduce((sum, p) => sum + p.amount_money.amount, 0) 
      / standardPayments.length / 100
    : 0;
  
  const premiumRatio = avgStandardSpend > 0
    ? (avgPremiumSpend / avgStandardSpend).toFixed(2)
    : 0;
  
  return {
    totalPremiumRevenue,
    avgPremiumSpend,
    premiumCustomerCount: premiumCustomerIds.length,
    totalLoyaltyMembers: loyaltyResponse.loyalty_accounts?.length || 0,
    premiumRatio
  };
}

// Usage
const premiumMetrics = await calculatePremiumGuestSpend(startDate, endDate);

document.querySelector('.premium-revenue').textContent = 
  `$${(premiumMetrics.totalPremiumRevenue / 1000000).toFixed(1)}M`;
document.querySelector('.avg-premium-spend').textContent = 
  `$${premiumMetrics.avgPremiumSpend.toFixed(0)}`;
document.querySelector('.premium-ratio').textContent = 
  `${premiumMetrics.premiumRatio}x`;
```

---

### Metric 7: Refund/Void Rate

**Current**: Mock 1.8%

**API Implementation**:
```javascript
async function calculateRefundRate(locationIds, startDate, endDate) {
  // Get all refunds
  let allRefunds = [];
  let allPayments = [];
  
  for (const locationId of locationIds) {
    const [refunds, payments] = await Promise.all([
      api.getRefunds({
        location_id: locationId,
        begin_time: startDate,
        end_time: endDate
      }),
      api.getPayments({
        location_id: locationId,
        begin_time: startDate,
        end_time: endDate
      })
    ]);
    
    allRefunds = allRefunds.concat(refunds.refunds || []);
    allPayments = allPayments.concat(payments.payments || []);
  }
  
  const refundRate = allPayments.length > 0
    ? ((allRefunds.length / allPayments.length) * 100).toFixed(1)
    : 0;
  
  const totalRefunded = allRefunds.reduce((sum, refund) => {
    return sum + refund.amount_money.amount;
  }, 0) / 100;
  
  return {
    refundRate,
    refundCount: allRefunds.length,
    totalRefunded,
    paymentCount: allPayments.length
  };
}

// Usage
const refundMetrics = await calculateRefundRate(locationIds, startDate, endDate);

document.getElementById('refundRate').textContent = 
  `${refundMetrics.refundRate}%`;
```

---

## 🔧 Advanced Iteration Topics

### Pagination Handling

Many Square APIs return paginated results. Always handle pagination:

```javascript
async function fetchAllPages(apiMethod, params) {
  let allResults = [];
  let cursor = null;
  
  do {
    const response = await apiMethod({
      ...params,
      cursor
    });
    
    allResults = allResults.concat(response.results || []);
    cursor = response.cursor;
  } while (cursor);
  
  return allResults;
}
```

### Error Handling & Retry Logic

```javascript
async function apiCallWithRetry(apiMethod, params, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await apiMethod(params);
    } catch (error) {
      if (error.message.includes('RATE_LIMITED')) {
        const waitTime = Math.pow(2, i) * 1000;
        console.log(`Rate limited. Waiting ${waitTime}ms...`);
        await new Promise(resolve => setTimeout(resolve, waitTime));
        continue;
      }
      
      if (i === maxRetries - 1) throw error;
    }
  }
}
```

### Caching Strategy

```javascript
class APICache {
  constructor(ttl = 300000) { // 5 minutes
    this.cache = new Map();
    this.ttl = ttl;
  }
  
  async get(key, fetchFn) {
    const cached = this.cache.get(key);
    
    if (cached && Date.now() - cached.timestamp < this.ttl) {
      return cached.data;
    }
    
    const data = await fetchFn();
    this.cache.set(key, {
      data,
      timestamp: Date.now()
    });
    
    return data;
  }
}

// Usage
const cache = new APICache();

const payments = await cache.get(
  `payments-${locationId}-${date}`,
  () => api.getPayments({ location_id: locationId, begin_time: date })
);
```

### Batch Processing

```javascript
async function batchProcess(items, processor, batchSize = 10) {
  const results = [];
  
  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    const batchResults = await Promise.all(
      batch.map(item => processor(item))
    );
    results.push(...batchResults);
  }
  
  return results;
}

// Usage
const locationMetrics = await batchProcess(
  locationIds,
  async (locationId) => await calculateLocationMetrics(locationId),
  3 // Process 3 locations at a time
);
```

---

## 🧪 Testing Guide

### Unit Tests for API Functions

```javascript
// tests/api.test.js
describe('Square API Integration', () => {
  test('calculateTotalNetSales returns correct sum', async () => {
    const mockPayments = {
      payments: [
        { amount_money: { amount: 1000 }, status: 'COMPLETED' },
        { amount_money: { amount: 2000 }, status: 'COMPLETED' }
      ]
    };
    
    // Mock API call
    api.getPayments = jest.fn().mockResolvedValue(mockPayments);
    
    const result = await calculateTotalNetSales(['LOC1'], '2024-01-01', '2024-01-31');
    expect(result).toBe(30); // $30.00
  });
});
```

### Integration Testing with Sandbox

```javascript
// tests/integration.test.js
const sandboxConfig = {
  squareAccessToken: process.env.SQUARE_SANDBOX_TOKEN,
  baseURL: 'https://connect.squareupsandbox.com/v2'
};

const sandboxAPI = new SquareAPI(sandboxConfig);

describe('Sandbox Integration Tests', () => {
  test('can fetch payments from sandbox', async () => {
    const payments = await sandboxAPI.getPayments({
      limit: 10
    });
    
    expect(payments.payments).toBeDefined();
    expect(Array.isArray(payments.payments)).toBe(true);
  });
});
```

---

## 📊 Performance Optimization

### Parallel API Calls

```javascript
async function loadDashboardData() {
  const [
    payments,
    orders,
    customers,
    loyalty,
    devices
  ] = await Promise.all([
    api.getPayments({ begin_time: startDate, end_time: endDate }),
    api.searchOrders({ location_ids: locationIds }),
    api.getCustomers(),
    api.searchLoyaltyAccounts({}),
    api.getDevices()
  ]);
  
  return { payments, orders, customers, loyalty, devices };
}
```

### Data Aggregation

```javascript
function aggregateMetrics(rawData) {
  return {
    totalSales: calculateTotalSales(rawData.payments),
    avgTicket: calculateAvgTicket(rawData.orders),
    topItems: calculateTopItems(rawData.orders),
    tenderMix: calculateTenderMix(rawData.payments),
    premiumSpend: calculatePremiumSpend(rawData.loyalty, rawData.payments)
  };
}
```

---

## 🚀 Next Steps

1. **Replace mock data** with real API calls (start with one metric)
2. **Test with Sandbox** before using production
3. **Implement caching** to reduce API calls
4. **Add error handling** for all API calls
5. **Optimize performance** with parallel calls
6. **Add real-time updates** with webhooks (optional)

---

**For questions or issues, refer to the [Square API Documentation](https://developer.squareup.com/docs)**
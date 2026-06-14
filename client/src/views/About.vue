<template>
  <div class="about-page">
    <div class="page-header">
      <h2>About This App</h2>
      <a href="/architecture.html" target="_blank" class="arch-btn">
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <rect x="3" y="3" width="7" height="7"/>
          <rect x="14" y="3" width="7" height="7"/>
          <rect x="14" y="14" width="7" height="7"/>
          <rect x="3" y="14" width="7" height="7"/>
          <line x1="10" y1="6.5" x2="14" y2="6.5"/>
          <line x1="6.5" y1="10" x2="6.5" y2="14"/>
          <line x1="17.5" y1="10" x2="17.5" y2="14"/>
          <line x1="10" y1="17.5" x2="14" y2="17.5"/>
        </svg>
        System Architecture
      </a>
    </div>

    <div class="hero-card">
      <h3>Factory Inventory Management System</h3>
      <p>Catalyst Components' Factory Inventory Management System provides real-time visibility across three warehouses (San Francisco, London, Tokyo). Track inventory levels and valuations, monitor order fulfillment and revenue, analyze demand trends, review spending by cost category, and generate quarterly reports — all filtered by time period, warehouse, product category, and order status.</p>
      <div class="filter-badges">
        <span class="filter-badge">Time Period</span>
        <span class="filter-badge">Warehouse / Location</span>
        <span class="filter-badge">Category</span>
        <span class="filter-badge">Order Status</span>
      </div>
    </div>

    <div class="section-label">APPLICATION TABS</div>
    <div class="tabs-grid">
      <div class="tab-card" v-for="tab in tabs" :key="tab.route">
        <div class="tab-card-header">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" v-html="tab.iconPath"></svg>
          <span class="tab-name">{{ tab.name }}</span>
        </div>
        <div class="tab-route">{{ tab.route }}</div>
        <div class="tab-desc">{{ tab.description }}</div>
      </div>
    </div>

    <div class="section-label">OVERVIEW KPI DRIVERS</div>
    <div class="kpi-card">
      <table class="kpi-table">
        <thead>
          <tr>
            <th>KPI Metric</th>
            <th>Driver Field(s)</th>
            <th>Logic</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="kpi in kpis" :key="kpi.metric">
            <td>{{ kpi.metric }}</td>
            <td>
              <template v-if="kpi.fields">
                <code v-for="field in kpi.fields" :key="field">{{ field }}</code>
              </template>
              <template v-else>—</template>
            </td>
            <td>{{ kpi.logic }}</td>
          </tr>
        </tbody>
      </table>
      <p class="kpi-note">All five KPIs respond to the four global sidebar filters: Time Period (filters by <code>order.order_date</code>), Warehouse (filters by <code>order.warehouse</code> and <code>inventory.warehouse</code>), Category (filters by <code>order.category</code> and <code>inventory.category</code>), and Order Status (filters by <code>order.status</code>).</p>
    </div>
  </div>
</template>

<script>
import { useI18n } from '../composables/useI18n'

export default {
  name: 'About',
  setup() {
    const { t } = useI18n()

    const tabs = [
      {
        name: 'Overview',
        route: '/',
        iconPath: '<path d="M3 9l9-7 9 7v11a2 2 0 01-2 2H5a2 2 0 01-2-2z"/>',
        description: 'High-level KPI dashboard showing 5 metric cards, an order health donut chart, inventory value by category, active backlog shortages, and a top-products revenue table.'
      },
      {
        name: 'Inventory',
        route: '/inventory',
        iconPath: '<path d="M20 7H4a1 1 0 00-1 1v10a1 1 0 001 1h16a1 1 0 001-1V8a1 1 0 00-1-1zM9 7V5a2 2 0 012-2h2a2 2 0 012 2v2"/>',
        description: 'Full SKU catalog with quantity on hand, reorder points, and unit costs. Items falling below their reorder threshold are highlighted. Filterable by warehouse and category.'
      },
      {
        name: 'Orders',
        route: '/orders',
        iconPath: '<path d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 000 4h6a2 2 0 000-4M9 5a2 2 0 012-2h2a2 2 0 012 2"/>',
        description: 'All purchase orders with status tracking (Delivered, Shipped, Processing, Backordered). Supports creating new orders and drilling into individual order details.'
      },
      {
        name: 'Finance',
        route: '/spending',
        iconPath: '<path d="M12 1v22M17 5H9.5a3.5 3.5 0 000 7h5a3.5 3.5 0 010 7H6"/>',
        description: 'Spending breakdown across four cost categories — Procurement, Operational, Labor, and Overhead — with monthly trend charts and a full transaction ledger.'
      },
      {
        name: 'Demand Forecast',
        route: '/demand',
        iconPath: '<path d="M22 12h-4l-3 9L9 3l-3 9H2"/>',
        description: 'Per-SKU demand outlook comparing current vs. forecasted demand with trend direction (increasing, stable, decreasing) over the next 30 days.'
      },
      {
        name: 'Reports',
        route: '/reports',
        iconPath: '<path d="M14 2H6a2 2 0 00-2 2v16a2 2 0 002 2h12a2 2 0 002-2V8zM14 2v6h6"/><path d="M16 13H8M16 17H8M10 9H8"/>',
        description: 'Quarterly and monthly aggregate views — total revenue, order counts, fulfillment rates, and average order value across all active periods.'
      },
      {
        name: 'Restocking',
        route: '/restocking',
        iconPath: '<path d="M4 4h16v2H4zM4 10h16v2H4zM4 16h10v2H4"/><path d="M18 14l4 4-4 4"/>',
        description: 'Surfaces SKUs with increasing demand trend and recommends restocking quantities. Allows submitting a restocking order directly from the recommendation list.'
      }
    ]

    const kpis = [
      {
        metric: 'Orders Fulfilled',
        fields: ['order.status'],
        logic: 'COUNT of orders where status === "Delivered"'
      },
      {
        metric: 'Order Fill Rate',
        fields: ['order.status'],
        logic: 'Delivered count ÷ total orders × 100 — shown as %'
      },
      {
        metric: 'Revenue (YTD / MTD)',
        fields: ['order.total_value', 'order.order_date'],
        logic: 'SUM of total_value; monthly goal $800K, YTD goal $9.6M'
      },
      {
        metric: 'Inventory Turnover',
        fields: null,
        logic: 'Hardcoded placeholder (4.2); goal 4.5 — no live data field yet'
      },
      {
        metric: 'Avg Processing Time',
        fields: null,
        logic: 'Hardcoded placeholder (2.8 days); goal 3.0 days — no live data field yet'
      }
    ]

    return { t, tabs, kpis }
  }
}
</script>

<style scoped>
.about-page {
  background: #f8fafc;
  padding: 1.5rem 2rem;
}

.page-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1.5rem;
}

.page-header h2 {
  font-size: 1.875rem;
  font-weight: 700;
  color: #0f172a;
  letter-spacing: -0.025em;
}

.arch-btn {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.5rem 1rem;
  background: white;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  color: #1e293b;
  font-size: 0.875rem;
  font-weight: 500;
  text-decoration: none;
  cursor: pointer;
  transition: background 0.15s ease, border-color 0.15s ease;
}

.arch-btn:hover {
  background: #f1f5f9;
  border-color: #cbd5e1;
}

.arch-btn svg {
  width: 16px;
  height: 16px;
  stroke: #64748b;
}

.hero-card {
  background: white;
  border: 1px solid #e2e8f0;
  border-radius: 10px;
  padding: 1.5rem;
  margin-bottom: 1.5rem;
}

.hero-card h3 {
  font-size: 1.25rem;
  font-weight: 700;
  color: #0f172a;
  margin-bottom: 0.75rem;
}

.hero-card p {
  color: #475569;
  font-size: 0.938rem;
  line-height: 1.6;
  margin-bottom: 1rem;
}

.filter-badges {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
}

.filter-badge {
  display: inline-flex;
  align-items: center;
  padding: 0.25rem 0.75rem;
  background: #dbeafe;
  color: #1e40af;
  border-radius: 9999px;
  font-size: 0.75rem;
  font-weight: 600;
}

.section-label {
  font-size: 0.75rem;
  font-weight: 600;
  color: #64748b;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  margin-bottom: 0.75rem;
}

.tabs-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 1rem;
  margin-bottom: 1.5rem;
}

.tab-card {
  background: white;
  border: 1px solid #e2e8f0;
  border-radius: 10px;
  padding: 1.25rem;
  transition: border-color 0.15s ease, box-shadow 0.15s ease;
}

.tab-card:hover {
  border-color: #cbd5e1;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.06);
}

.tab-card-header {
  display: flex;
  align-items: center;
  gap: 0.625rem;
  margin-bottom: 0.5rem;
}

.tab-card-header svg {
  width: 18px;
  height: 18px;
  flex-shrink: 0;
  color: #3b82f6;
}

.tab-name {
  font-size: 0.938rem;
  font-weight: 700;
  color: #1e293b;
}

.tab-route {
  font-size: 0.75rem;
  color: #94a3b8;
  font-family: monospace;
  margin-bottom: 0.5rem;
}

.tab-desc {
  font-size: 0.875rem;
  color: #475569;
  line-height: 1.5;
}

.kpi-card {
  background: white;
  border: 1px solid #e2e8f0;
  border-radius: 10px;
  padding: 1.25rem;
}

.kpi-table {
  width: 100%;
  border-collapse: collapse;
}

.kpi-table th {
  text-align: left;
  padding: 0.5rem 0.75rem;
  font-weight: 600;
  color: #475569;
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  background: #f8fafc;
  border-bottom: 2px solid #e2e8f0;
}

.kpi-table td {
  padding: 0.625rem 0.75rem;
  border-top: 1px solid #f1f5f9;
  font-size: 0.875rem;
  color: #334155;
}

.kpi-table td code {
  background: #f1f5f9;
  color: #0f172a;
  padding: 0.125rem 0.375rem;
  border-radius: 4px;
  font-size: 0.8125rem;
  /* Multiple code tags per cell need spacing */
  margin-right: 0.25rem;
}

.kpi-note {
  margin-top: 1rem;
  font-size: 0.8125rem;
  color: #64748b;
  line-height: 1.5;
  padding: 0.75rem;
  background: #f8fafc;
  border-radius: 6px;
  border: 1px solid #e2e8f0;
}

.kpi-note code {
  background: #f1f5f9;
  color: #0f172a;
  padding: 0.125rem 0.375rem;
  border-radius: 4px;
  font-size: 0.8125rem;
}
</style>

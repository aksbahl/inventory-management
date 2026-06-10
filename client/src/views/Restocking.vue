<template>
  <div class="restocking">
    <div class="page-header">
      <h2>Restocking Planner</h2>
      <p>Recommended items based on increasing demand forecasts</p>
    </div>

    <div v-if="loading" class="loading">Loading recommendations...</div>
    <div v-else-if="error" class="error">{{ error }}</div>
    <div v-else class="planner-layout">

      <!-- Left panel: Budget Controls -->
      <div class="budget-panel card">
        <div class="card-header">
          <h3 class="card-title">Budget Controls</h3>
        </div>

        <div class="budget-display">
          <div class="budget-label">Budget</div>
          <div class="budget-amount">{{ currencySymbol }}{{ budget.toLocaleString() }}</div>
        </div>

        <div class="slider-wrapper">
          <input
            type="range"
            class="budget-slider"
            :min="1000"
            :max="200000"
            :step="1000"
            v-model.number="budget"
          />
          <div class="slider-bounds">
            <span>{{ currencySymbol }}1,000</span>
            <span>{{ currencySymbol }}200,000</span>
          </div>
        </div>

        <div class="summary-card">
          <div class="summary-row">
            <span class="summary-key">Items in budget</span>
            <span class="summary-value">{{ recommendedItems.length }}</span>
          </div>
          <div class="summary-row">
            <span class="summary-key">Total cost</span>
            <span class="summary-value">{{ currencySymbol }}{{ totalCost.toLocaleString() }}</span>
          </div>
          <div class="summary-row">
            <span class="summary-key">Remaining budget</span>
            <span class="summary-value" :class="{ 'remaining-positive': remainingBudget > 0 }">
              {{ currencySymbol }}{{ remainingBudget.toLocaleString() }}
            </span>
          </div>
        </div>

        <div class="action-area">
          <div v-if="submittedOrder" class="success-message">
            <div class="success-heading">Order Placed</div>
            <div class="success-detail">
              Order <span class="order-number">{{ submittedOrder.order_number }}</span>
            </div>
            <div class="success-detail">
              Expected delivery: {{ formatDeliveryDate(submittedOrder.expected_delivery) }}
            </div>
          </div>
          <button
            v-else
            class="place-order-btn"
            :disabled="recommendedItems.length === 0 || submitting"
            @click="placeOrder"
          >
            {{ submitting ? 'Placing Order...' : 'Place Order' }}
          </button>
          <div v-if="orderError" class="order-error">{{ orderError }}</div>
        </div>
      </div>

      <!-- Right panel: Recommended Items -->
      <div class="items-panel card">
        <div class="card-header">
          <h3 class="card-title">Recommended Items</h3>
          <span class="items-count" v-if="recommendedItems.length > 0">
            {{ recommendedItems.length }} item{{ recommendedItems.length !== 1 ? 's' : '' }}
          </span>
        </div>

        <div v-if="recommendedItems.length === 0" class="empty-state">
          Increase your budget to see recommendations.
        </div>
        <div v-else class="table-container">
          <table>
            <thead>
              <tr>
                <th>Item Name</th>
                <th>SKU</th>
                <th class="col-right">Demand Gap</th>
                <th class="col-right">Unit Cost</th>
                <th class="col-right">Qty to Order</th>
                <th class="col-right">Subtotal</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="item in recommendedItems" :key="item.item_sku">
                <td>{{ item.item_name }}</td>
                <td><strong>{{ item.item_sku }}</strong></td>
                <td class="col-right">
                  <span class="demand-gap">+{{ item.recommended_qty }}</span>
                </td>
                <td class="col-right">{{ currencySymbol }}{{ item.unit_cost.toLocaleString() }}</td>
                <td class="col-right">{{ item.recommended_qty }}</td>
                <td class="col-right subtotal">
                  {{ currencySymbol }}{{ item.subtotal.toLocaleString() }}
                </td>
              </tr>
            </tbody>
            <tfoot>
              <tr class="total-row">
                <td colspan="5" class="total-label">Total</td>
                <td class="col-right total-amount">
                  {{ currencySymbol }}{{ totalCost.toLocaleString() }}
                </td>
              </tr>
            </tfoot>
          </table>
        </div>
      </div>

    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'
import { api } from '../api'
import { useI18n } from '../composables/useI18n'

export default {
  name: 'Restocking',
  setup() {
    const { currentCurrency } = useI18n()

    const budget = ref(50000)
    const allItems = ref([])
    const loading = ref(true)
    const error = ref(null)
    const submittedOrder = ref(null)
    const submitting = ref(false)
    const orderError = ref(null)

    const currencySymbol = computed(() => currentCurrency.value === 'JPY' ? '¥' : '$')

    // Greedy selection: pick items largest-gap-first until budget exhausted
    const recommendedItems = computed(() => {
      let remaining = budget.value
      return allItems.value.filter(item => {
        if (item.subtotal <= remaining) {
          remaining -= item.subtotal
          return true
        }
        return false
      })
    })

    const totalCost = computed(() =>
      recommendedItems.value.reduce((sum, i) => sum + i.subtotal, 0)
    )

    const remainingBudget = computed(() => budget.value - totalCost.value)

    const loadRecommendations = async () => {
      loading.value = true
      error.value = null
      try {
        const data = await api.getRestockingRecommendations()
        allItems.value = data
      } catch (err) {
        error.value = 'Failed to load restocking recommendations: ' + err.message
        console.error(err)
      } finally {
        loading.value = false
      }
    }

    const formatDeliveryDate = (dateStr) => {
      if (!dateStr) return ''
      const d = new Date(dateStr)
      if (isNaN(d.getTime())) return dateStr
      return d.toLocaleDateString('en-US', { month: 'short', day: 'numeric', year: 'numeric' })
    }

    const placeOrder = async () => {
      if (recommendedItems.value.length === 0 || submittedOrder.value) return
      submitting.value = true
      orderError.value = null
      try {
        const payload = {
          customer: 'Internal Restocking',
          warehouse: 'All',
          category: 'Mixed',
          items: recommendedItems.value.map(i => ({
            sku: i.item_sku,
            name: i.item_name,
            quantity: i.recommended_qty,
            unit_price: i.unit_cost
          })),
          notes: `Budget: $${budget.value.toLocaleString()}`
        }
        const order = await api.submitRestockingOrder(payload)
        submittedOrder.value = order
      } catch (err) {
        orderError.value = 'Failed to place order. Please try again.'
        console.error(err)
      } finally {
        submitting.value = false
      }
    }

    onMounted(loadRecommendations)

    return {
      budget,
      loading,
      error,
      currencySymbol,
      recommendedItems,
      totalCost,
      remainingBudget,
      submittedOrder,
      submitting,
      orderError,
      formatDeliveryDate,
      placeOrder
    }
  }
}
</script>

<style scoped>
.restocking {
  /* inherits .main-content padding from App.vue */
}

/* Two-column layout */
.planner-layout {
  display: grid;
  grid-template-columns: 340px 1fr;
  gap: 1.5rem;
  align-items: start;
}

@media (max-width: 900px) {
  .planner-layout {
    grid-template-columns: 1fr;
  }
}

/* Budget panel */
.budget-panel {
  position: sticky;
  top: 90px; /* clears the sticky nav */
}

.budget-display {
  margin-bottom: 1.5rem;
}

.budget-label {
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: #64748b;
  margin-bottom: 0.375rem;
}

.budget-amount {
  font-size: 2.25rem;
  font-weight: 700;
  color: #0f172a;
  letter-spacing: -0.025em;
}

/* Slider */
.slider-wrapper {
  margin-bottom: 1.5rem;
}

.budget-slider {
  width: 100%;
  -webkit-appearance: none;
  appearance: none;
  height: 6px;
  border-radius: 3px;
  background: #e2e8f0;
  outline: none;
  accent-color: #2563eb;
  cursor: pointer;
  margin-bottom: 0.5rem;
}

.budget-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 18px;
  height: 18px;
  border-radius: 50%;
  background: #2563eb;
  cursor: pointer;
  box-shadow: 0 1px 4px rgba(37, 99, 235, 0.4);
  transition: box-shadow 0.15s ease;
}

.budget-slider::-webkit-slider-thumb:hover {
  box-shadow: 0 0 0 4px rgba(37, 99, 235, 0.15);
}

.budget-slider::-moz-range-thumb {
  width: 18px;
  height: 18px;
  border-radius: 50%;
  background: #2563eb;
  cursor: pointer;
  border: none;
  box-shadow: 0 1px 4px rgba(37, 99, 235, 0.4);
}

.slider-bounds {
  display: flex;
  justify-content: space-between;
  font-size: 0.75rem;
  color: #94a3b8;
}

/* Summary card */
.summary-card {
  background: #f8fafc;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  padding: 1rem;
  margin-bottom: 1.5rem;
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.summary-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.summary-key {
  font-size: 0.875rem;
  color: #64748b;
  font-weight: 500;
}

.summary-value {
  font-size: 0.938rem;
  font-weight: 700;
  color: #0f172a;
}

.summary-value.remaining-positive {
  color: #059669;
}

/* Action area */
.place-order-btn {
  width: 100%;
  padding: 0.75rem 1.5rem;
  background: #2563eb;
  color: #ffffff;
  border: none;
  border-radius: 6px;
  font-size: 0.938rem;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s ease, opacity 0.2s ease;
}

.place-order-btn:hover:not(:disabled) {
  background: #1d4ed8;
}

.place-order-btn:disabled {
  background: #94a3b8;
  cursor: not-allowed;
  opacity: 0.7;
}

.order-error {
  margin-top: 0.75rem;
  font-size: 0.875rem;
  color: #dc2626;
}

/* Success confirmation */
.success-message {
  background: #f0fdf4;
  border: 1px solid #bbf7d0;
  border-radius: 8px;
  padding: 1rem 1.25rem;
}

.success-heading {
  font-size: 0.875rem;
  font-weight: 700;
  color: #059669;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  margin-bottom: 0.5rem;
}

.success-detail {
  font-size: 0.875rem;
  color: #065f46;
  margin-top: 0.25rem;
}

.order-number {
  font-family: 'Courier New', Courier, monospace;
  font-weight: 700;
  font-size: 0.938rem;
  background: #d1fae5;
  padding: 0.125rem 0.375rem;
  border-radius: 4px;
}

/* Items panel */
.items-count {
  font-size: 0.813rem;
  font-weight: 600;
  color: #64748b;
  background: #f1f5f9;
  padding: 0.25rem 0.625rem;
  border-radius: 20px;
}

.empty-state {
  padding: 3rem;
  text-align: center;
  color: #64748b;
  font-size: 0.938rem;
}

/* Table column alignment */
.col-right {
  text-align: right;
}

th.col-right {
  text-align: right;
}

.demand-gap {
  color: #059669;
  font-weight: 600;
}

.subtotal {
  font-weight: 600;
  color: #0f172a;
}

/* Footer total row */
tfoot {
  border-top: 2px solid #e2e8f0;
}

.total-row td {
  padding: 0.75rem;
  background: #f8fafc;
}

.total-label {
  font-size: 0.875rem;
  font-weight: 700;
  color: #475569;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  text-align: right;
  padding-right: 1rem;
}

.total-amount {
  font-size: 1rem;
  font-weight: 700;
  color: #0f172a;
}
</style>

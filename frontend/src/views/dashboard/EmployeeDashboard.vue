<template>
  <div class="dashboard-page">

    <!-- 1. 필터 바 -->
    <div class="filter-row">
      <select v-model="filterYear">
        <option v-for="y in yearList" :key="y" :value="y">{{ y }}년</option>
      </select>
      <select v-model="filterType">
        <option value="month">월별</option>
        <option value="quarter">분기별</option>
        <option value="year">연별</option>
      </select>
      <select v-if="filterType === 'month'" v-model="filterValue">
        <option v-for="m in 12" :key="m" :value="m">{{ m }}월</option>
      </select>
      <select v-else-if="filterType === 'quarter'" v-model="filterValue">
        <option v-for="q in 4" :key="q" :value="q">{{ q }}분기</option>
      </select>
    </div>

    <!-- 2. 게이지 KPI 카드 -->
    <div class="icon-stats-row">
      <div class="icon-card">
        <div class="gauge-wrapper">
          <v-chart :option="productGaugeOption" autoresize style="width:80px;height:80px;" />
          <div v-if="productPercent > 1" class="gauge-over">+{{ ((productPercent - 1) * 100).toFixed(0) }}%</div>
        </div>
        <div class="icon-content">
          <p class="icon-title">렌탈 상품 수</p>
          <h3 class="icon-value">{{ kpi.rental_product_count }}</h3>
          <small class="icon-subtitle">목표: {{ goal.rental_product_goal }}</small>
        </div>
      </div>

      <div class="icon-card">
        <div class="gauge-wrapper">
          <v-chart :option="retentionGaugeOption" autoresize style="width:80px;height:80px;" />
          <div v-if="retentionPercentNum > goal.retention_rate_goal / 100" class="gauge-over">
            +{{ ((retentionPercentNum - goal.retention_rate_goal / 100) * 100).toFixed(0) }}%
          </div>
        </div>
        <div class="icon-content">
          <p class="icon-title">렌탈 유지율</p>
          <h3 class="icon-value">{{ retentionPercent }}%</h3>
          <small class="icon-subtitle">목표: {{ goal.retention_rate_goal.toFixed(1) }}%</small>
        </div>
      </div>

      <div class="icon-card">
        <div class="gauge-wrapper">
          <v-chart :option="customerGaugeOption" autoresize style="width:80px;height:80px;" />
          <div v-if="newCustomerPercent > 1" class="gauge-over">
            +{{ ((newCustomerPercent - 1) * 100).toFixed(0) }}%
          </div>
        </div>
        <div class="icon-content">
          <p class="icon-title">신규 고객수</p>
          <h3 class="icon-value">{{ kpi.new_customer_count }}</h3>
          <small class="icon-subtitle">목표: {{ goal.new_customer_goal }}</small>
        </div>
      </div>

      <div class="icon-card">
        <div class="gauge-wrapper">
          <v-chart :option="feedbackGaugeOption" autoresize style="width:80px;height:80px;" />
          <div v-if="feedbackScoreNum > feedbackMax" class="gauge-over">
            +{{ ((feedbackScoreNum - feedbackMax) * 100 / feedbackMax).toFixed(0) }}%
          </div>
        </div>
        <div class="icon-content">
          <p class="icon-title">고객 만족도</p>
          <h3 class="icon-value">{{ feedbackScore }}</h3>
          <small class="icon-subtitle">목표: {{ goal.satisfaction_goal.toFixed(2) }}</small>
        </div>
      </div>
    </div>

    <!-- 3. 메인 차트 + 비교차트(50:50) -->
    <div class="main-row">
      <div class="card main-chart">
        <div class="card-header with-gauge main-header-align">
          <div class="main-title-block">
            <h4 class="main-card-title">{{ mainTitle }}</h4>
            <p class="main-card-category">계약 전체 렌탈료</p>
          </div>
          <div class="main-total-gauge">
            <div class="gauge-wrapper" style="width:80px;height:80px;">
              <v-chart :option="salesGaugeOption" autoresize style="width:100%;height:100%;" />
              <div v-if="salesPercent > 1" class="gauge-over">
                +{{ ((salesPercent - 1) * 100).toFixed(0) }}%
              </div>
            </div>
            <div class="main-total-label">
              <p>총 매출</p>
              <h3>{{ formatKoreanAmount(kpi.total_rental_amount) }}원</h3>
            </div>
          </div>
        </div>
        <div class="card-body">
          <v-chart :option="mainOption" autoresize style="height:260px;" />
        </div>
      </div>
      <div class="card compare-chart">
        <div class="card-header main-header-align">
          <h4 class="main-card-title">{{ compareTitle }}</h4>
        </div>
        <div class="card-body padded-chart">
          <v-chart :option="compareOption" autoresize style="height:260px;" />
        </div>
      </div>
    </div>

    <!-- 4. 미니 통계 차트 -->
    <div class="stats-row">
      <div class="card stat-card">
        <div class="card-header stat-header">
          <p class="icon-title">렌탈 상품 수</p>
          <h3 class="icon-value">{{ miniStats.product }}</h3>
        </div>
        <div class="card-body stat-body padded-chart">
          <v-chart :option="miniOption(miniStats.productArr, miniStats.xArr)" autoresize style="height:120px;" />
        </div>
      </div>
      <div class="card stat-card">
        <div class="card-header stat-header">
          <p class="icon-title">신규 고객 수</p>
          <h3 class="icon-value">{{ miniStats.newCustomer }}</h3>
        </div>
        <div class="card-body stat-body padded-chart">
          <v-chart :option="miniOption(miniStats.newCustomerArr, miniStats.xArr)" autoresize style="height:120px;" />
        </div>
      </div>
      <div class="card stat-card">
        <div class="card-header stat-header">
          <p class="icon-title">렌탈 유지율</p>
          <h3 class="icon-value">{{ miniStats.retention }}%</h3>
        </div>
        <div class="card-body stat-body padded-chart">
          <v-chart :option="miniOption(miniStats.retentionArr, miniStats.xArr)" autoresize style="height:120px;" />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted } from 'vue'
import * as echarts from 'echarts'
import api from '@/api/auth'

// 금액 한글 단위로 변환
function formatKoreanAmount(num) {
  if (num == null) return '-'
  if (num >= 1e8) return (num / 1e8).toFixed(2).replace(/\.00$/, '') + '억'
  if (num >= 1e4) return (num / 1e4).toFixed(2).replace(/\.00$/, '') + '만'
  return num.toLocaleString()
}
function getMonthLabel(m) {
  if (m < 1) return '12월'
  if (m > 12) return '1월'
  return `${m}월`
}
function getQuarterLabel(q) {
  if (q < 1) return '4분기'
  if (q > 4) return '1분기'
  return `${q}분기`
}
function getYearLabel(y) {
  return `${y}년`
}

function getEmployeeCodeFromToken() {
  const token = localStorage.getItem('accessToken')
  if (!token) return null
  try {
    return JSON.parse(atob(token.split('.')[1].replace(/-/g, '+').replace(/_/g, '/'))).code
  } catch {
    return null
  }
}
const employeeCode = getEmployeeCodeFromToken()
const thisYear = new Date().getFullYear()
const yearList = Array.from({ length: thisYear - 2019 + 1 }, (_, i) => thisYear - i)
const filterYear = ref(thisYear)
const filterType = ref('month')
const filterValue = ref(1)
watch(filterType, () => { filterValue.value = 1 })

const kpi = ref({
  rental_product_count: 0,
  rental_retention_count: 0,
  total_rental_count: 0,
  new_customer_count: 0,
  total_rental_amount: 0,
  customer_feedback_score: 0,
  customer_feedback_count: 0
})
const goal = ref({
  rental_product_goal: 100,
  rental_sales_goal: 10000000,
  new_customer_goal: 50,
  retention_rate_goal: 0,
  satisfaction_goal: 0
})

function splitToWeeks(total) {
  const parts = [Math.random(), Math.random(), Math.random(), Math.random()]
  const sum = parts.reduce((a, b) => a + b, 0)
  const vals = parts.map(p => Math.round(total * (p / sum)))
  vals[3] += total - vals.reduce((a, b) => a + b, 0)
  return vals
}

function getRange(type, year, val) {
  if (type === 'month') {
    let y = year, m = val
    if (m < 1) { m = 12; y-- } if (m > 12) { m = 1; y++ }
    const mm = String(m).padStart(2, '0')
    return { start: `${y}${mm}`, end: `${y}${mm}` }
  }
  if (type === 'quarter') {
    let y = year, q = val
    if (q < 1) { q = 4; y-- } if (q > 4) { q = 1; y++ }
    const m1 = (q - 1) * 3 + 1, m2 = m1 + 2
    return {
      start: `${y}${String(m1).padStart(2, '0')}`,
      end: `${y}${String(m2).padStart(2, '0')}`
    }
  }
  return { start: `${year}01`, end: `${year}12` }
}

async function fetchGoal() {
  if (!employeeCode) return
  const { start, end } = getRange(filterType.value, filterYear.value, filterValue.value)
  const res = await api.get('/api/goal/employee', { params: { employeeCode, startDate: start, endDate: end }, headers: { Authorization: `Bearer ${localStorage.getItem('accessToken')}` }, withCredentials: true })
  const allGoals = Array.isArray(res.data) ? res.data : []
  let months = []
  if (filterType.value === 'month') months = [filterValue.value]
  else if (filterType.value === 'quarter') {
    const s = (filterValue.value - 1) * 3 + 1
    months = [s, s + 1, s + 2]
  } else months = Array.from({ length: 12 }, (_, i) => i + 1)
  const sel = allGoals.filter(g => months.includes(parseInt(String(g.targetDate).slice(4, 6), 10)))
  let sum = { rental_product_goal: 0, rental_sales_goal: 0, new_customer_goal: 0 }
  const rateArr = [], satArr = []
  sel.forEach(g => {
    sum.rental_product_goal += g.rentalProductCount || 0
    sum.rental_sales_goal += g.totalRentalAmount || 0
    sum.new_customer_goal += g.newCustomerCount || 0
    if (g.totalRentalCount) rateArr.push((g.rentalRetentionCount / g.totalRentalCount) * 100)
    if (g.customerFeedbackCount) satArr.push(g.customerFeedbackScore / 10 / g.customerFeedbackCount)
  })
  goal.value = {
    rental_product_goal: sum.rental_product_goal,
    rental_sales_goal: sum.rental_sales_goal,
    new_customer_goal: sum.new_customer_goal,
    retention_rate_goal: rateArr.length ? rateArr.reduce((a, b) => a + b, 0) / rateArr.length : 0,
    satisfaction_goal: satArr.length ? satArr.reduce((a, b) => a + b, 0) / satArr.length : 0
  }
}

async function fetchCompare() {
  if (!employeeCode) return
  const curr = kpi.value.total_rental_amount
  const prev = getRange(filterType.value, filterYear.value, filterValue.value - 1)
  const next = getRange(filterType.value, filterYear.value, filterValue.value + 1)
  const [pr, nx] = await Promise.all([
    api.get(`/api/performance/employee/${employeeCode}`, { params: { startDate: prev.start, endDate: prev.end }, headers: { Authorization: `Bearer ${localStorage.getItem('accessToken')}` }, withCredentials: true }),
    api.get(`/api/performance/employee/${employeeCode}`, { params: { startDate: next.start, endDate: next.end }, headers: { Authorization: `Bearer ${localStorage.getItem('accessToken')}` }, withCredentials: true })
  ])
  const pSum = (Array.isArray(pr.data) ? pr.data : []).reduce((s, d) => s + (d.totalRentalAmount || 0), 0)
  const nSum = (Array.isArray(nx.data) ? nx.data : []).reduce((s, d) => s + (d.totalRentalAmount || 0), 0)
  compareKpi.value = [pSum, curr, nSum]
}

const perfList = ref([])
const compareKpi = ref([0, 0, 0])

async function fetchData() {
  if (!employeeCode) return
  const { start, end } = getRange(filterType.value, filterYear.value, filterValue.value)
  const res = await api.get(`/api/performance/employee/${employeeCode}`, {
    params: { startDate: start, endDate: end },
    headers: { Authorization: `Bearer ${localStorage.getItem('accessToken')}` },
    withCredentials: true
  })
  const arr = Array.isArray(res.data) ? res.data : []
  perfList.value = arr.sort((a, b) => a.targetDate - b.targetDate)

  if (filterType.value === 'month') {
    const f = perfList.value[0] || {}
    kpi.value = {
      rental_product_count: f.rentalProductCount || 0,
      rental_retention_count: f.rentalRetentionCount || 0,
      total_rental_count: f.totalRentalCount || 0,
      new_customer_count: f.newCustomerCount || 0,
      total_rental_amount: f.totalRentalAmount || 0,
      customer_feedback_score: f.customerFeedbackScore || 0,
      customer_feedback_count: f.customerFeedbackCount || 0
    }
  } else {
    const sumField = fld => perfList.value.reduce((s, d) => s + (d[fld] || 0), 0)
    kpi.value = {
      rental_product_count: sumField('rentalProductCount'),
      rental_retention_count: sumField('rentalRetentionCount'),
      total_rental_count: sumField('totalRentalCount'),
      new_customer_count: sumField('newCustomerCount'),
      total_rental_amount: sumField('totalRentalAmount'),
      customer_feedback_score: sumField('customerFeedbackScore'),
      customer_feedback_count: sumField('customerFeedbackCount')
    }
  }

  productWeeks.value = splitToWeeks(kpi.value.rental_product_count)
  newCustomerWeeks.value = splitToWeeks(kpi.value.new_customer_count)
  totalCountWeeks.value = splitToWeeks(kpi.value.total_rental_count)
  salesWeeks.value = splitToWeeks(kpi.value.total_rental_amount)
  retentionWeeks.value = totalCountWeeks.value.map((t, i) => t ? Number(((splitToWeeks(kpi.value.rental_retention_count)[i] / t) * 100).toFixed(1)) : 0)

  await fetchGoal()
  await fetchCompare()
}

onMounted(fetchData)
watch([filterYear, filterType, filterValue], fetchData)

const productWeeks = ref([0, 0, 0, 0])
const newCustomerWeeks = ref([0, 0, 0, 0])
const totalCountWeeks = ref([0, 0, 0, 0])
const salesWeeks = ref([0, 0, 0, 0])
const retentionWeeks = ref([0, 0, 0, 0])

const productPercent = computed(() => goal.value.rental_product_goal ? kpi.value.rental_product_count / goal.value.rental_product_goal : 0)
const retentionPercent = computed(() => kpi.value.total_rental_count ? ((kpi.value.rental_retention_count / kpi.value.total_rental_count) * 100).toFixed(1) : '0.0')
const retentionPercentNum = computed(() => parseFloat(retentionPercent.value) / 100)
const newCustomerPercent = computed(() => goal.value.new_customer_goal ? kpi.value.new_customer_count / goal.value.new_customer_goal : 0)
const feedbackScore = computed(() => {
  const s = kpi.value.customer_feedback_score, c = kpi.value.customer_feedback_count
  return c ? (s / c / 10).toFixed(2) : '0.00'
})
const feedbackScoreNum = computed(() => parseFloat(feedbackScore.value))
const feedbackMax = 5
const salesPercent = computed(() => goal.value.rental_sales_goal ? kpi.value.total_rental_amount / goal.value.rental_sales_goal : 0)

function gaugeOption(val, max, col) {
  return {
    series: [{
      type: 'gauge', startAngle: 90, endAngle: -270, min: 0, max, splitNumber: 5,
      axisLine: { lineStyle: { width: 8, color: [[Math.min(val, max) / max, col], [1, '#eee']] } },
      pointer: { show: false }, detail: { show: false },
      axisTick: { show: false }, splitLine: { show: false }, axisLabel: { show: false },
      data: [{ value: Math.min(val, max) }]
    }]
  }
}

const productGaugeOption = computed(() => gaugeOption(productPercent.value, 1, '#f5365c'))
const retentionGaugeOption = computed(() => {
  const gf = goal.value.retention_rate_goal / 100
  return gaugeOption(retentionPercentNum.value, gf, '#5e72e4')
})
const customerGaugeOption = computed(() => gaugeOption(newCustomerPercent.value, 1, '#11cdef'))
const feedbackGaugeOption = computed(() => gaugeOption(feedbackScoreNum.value, goal.value.satisfaction_goal, '#fb6340'))
const salesGaugeOption = computed(() => gaugeOption(salesPercent.value, 1, '#11cdef'))

const mainTitle = computed(() => {
  if (filterType.value === 'month') return `${filterYear.value}년 ${filterValue.value}월 매출 (주차)`
  if (filterType.value === 'quarter') return `${filterYear.value}년 ${filterValue.value}분기 매출`
  return `${filterYear.value}년 전체 매출`
})

const mainOption = computed(() => {
  let xData = [], yData = []
  if (filterType.value === 'month') {
    xData = ['1주차', '2주차', '3주차', '4주차']
    yData = salesWeeks.value
  } else {
    xData = perfList.value.map(d => `${String(d.targetDate).slice(-2)}월`)
    yData = perfList.value.map(d => d.totalRentalAmount)
  }
  return {
    grid: { top: '10%', left: '5%', right: '5%', bottom: '10%', containLabel: true },
    tooltip: { trigger: 'axis' },
    xAxis: { type: 'category', data: xData, axisLine: { lineStyle: { color: '#e9ecef' } }, axisLabel: { color: '#9fa9b1' }, axisTick: { show: false } },
    yAxis: { type: 'value', splitLine: { lineStyle: { color: '#e9ecef' } }, axisLabel: { color: '#9fa9b1' }, axisLine: { show: false } },
    series: [{
      type: 'line', smooth: true, data: yData, symbol: 'circle', symbolSize: 6,
      lineStyle: { width: 2, color: '#55ce63' },
      areaStyle: { color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [{ offset: 0, color: 'rgba(85,206,99,0.4)' }, { offset: 1, color: 'rgba(255,255,255,0.1)' }]) }
    }]
  }
})

const miniStats = computed(() => {
  let xArr = [], pArr = [], nArr = [], rArr = []
  if (filterType.value === 'month') {
    xArr = ['1주차', '2주차', '3주차', '4주차']
    pArr = productWeeks.value; nArr = newCustomerWeeks.value; rArr = retentionWeeks.value
  } else {
    xArr = perfList.value.map(d => `${String(d.targetDate).slice(-2)}월`)
    pArr = perfList.value.map(d => d.rentalProductCount)
    nArr = perfList.value.map(d => d.newCustomerCount)
    rArr = perfList.value.map(d => d.totalRentalCount ? Number(((d.rentalRetentionCount / d.totalRentalCount) * 100).toFixed(1)) : 0)
  }
  return { xArr, product: kpi.value.rental_product_count, productArr: pArr, newCustomer: kpi.value.new_customer_count, newCustomerArr: nArr, retention: retentionPercent.value, retentionArr: rArr }
})

function miniOption(dataArr, xArr) {
  return {
    tooltip: { trigger: 'axis', axisPointer: { type: 'line', lineStyle: { width: 1, color: '#aaa' } } },
    grid: { left: '10%', right: '10%', top: '10%', bottom: '15%', containLabel: true },
    xAxis: { type: 'category', data: xArr, axisLine: { lineStyle: { color: '#e9ecef' } }, axisTick: { show: false }, axisLabel: { color: '#9fa9b1' } },
    yAxis: { type: 'value', splitLine: { lineStyle: { color: '#e9ecef' } }, axisLabel: { color: '#9fa9b1' }, axisLine: { show: false } },
    series: [{ type: 'line', smooth: true, data: dataArr, symbol: 'circle', symbolSize: 4, lineStyle: { width: 2, color: '#5e72e4' }, areaStyle: { color: 'rgba(94,114,228,0.1)' } }]
  }
}

const compareTitle = computed(() => {
  if (filterType.value === 'month') return `${filterYear.value}년 ${filterValue.value}월 전후 비교`
  if (filterType.value === 'quarter') return `${filterYear.value}년 ${filterValue.value}분기 전후 비교`
  return `${filterYear.value}년 전후 비교`
})

const compareOption = computed(() => {
  let labels = []
  if (filterType.value === 'month') {
    labels = [
      getMonthLabel(filterValue.value - 1),
      getMonthLabel(filterValue.value),
      getMonthLabel(filterValue.value + 1)
    ]
  } else if (filterType.value === 'quarter') {
    labels = [
      getQuarterLabel(filterValue.value - 1),
      getQuarterLabel(filterValue.value),
      getQuarterLabel(filterValue.value + 1)
    ]
  } else {
    labels = [
      getYearLabel(filterYear.value - 1),
      getYearLabel(filterYear.value),
      getYearLabel(filterYear.value + 1)
    ]
  }
  return {
    grid: { top: '10%', left: '10%', right: '10%', bottom: '15%', containLabel: true },
    xAxis: { type: 'category', data: labels, axisLine: { lineStyle: { color: '#e9ecef' } }, axisTick: { show: false }, axisLabel: { color: '#9fa9b1' } },
    yAxis: { type: 'value', splitLine: { lineStyle: { color: '#e9ecef' } }, axisLabel: { color: '#9fa9b1' }, axisLine: { show: false } },
    series: [{
      type: 'bar', data: compareKpi.value, barWidth: '40%',
      itemStyle: { color: (p) => ['#ff7f7f', '#ffe066', '#a3e635'][p.dataIndex], borderRadius: [8, 8, 0, 0] },
      label: { show: true, position: 'top', color: '#333',
        formatter: params => formatKoreanAmount(params.value)
      }
    }]
  }
})
</script>

<style scoped>
.dashboard-page {
  padding: 24px;
  background: #f4f5f7;
  min-height: 100vh;
}
.filter-row {
  display: flex;
  gap: 1rem;
  margin-bottom: 1.3rem;
}
.filter-row select {
  padding: .5rem;
  border: 1px solid #ccc;
  border-radius: .25rem;
  background: #fff;
  font-size: 1rem;
}
.icon-stats-row {
  display: flex;
  gap: 1rem;
  margin-bottom: 24px;
}
.icon-card {
  flex: 1;
  background: #fff;
  border-radius: .75rem;
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.1);
  display: flex;
  align-items: center;
  padding: 1.2rem 1.2rem 1.2rem 1.6rem;
  min-width: 0;
  min-height: 130px;
}
.gauge-wrapper {
  position: relative;
  margin-right: 18px;
}
.gauge-over {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-size: .78rem;
  font-weight: bold;
  color: red;
  pointer-events: none;
}
.icon-content .icon-title {
  margin: 0 0 6px 0;
  font-size: .98rem;
  color: #7b8aaf;
  font-weight: 600;
  letter-spacing: -0.5px;
}
.icon-content .icon-value {
  margin: 0 0 4px 0;
  font-size: 1.25rem;
  font-weight: 700;
}
.icon-content .icon-subtitle {
  font-size: .78rem;
  color: #adb5bd;
}
.main-row {
  display: flex;
  gap: 1rem;
  margin-bottom: 24px;
}
.main-chart,
.compare-chart {
  flex: 1;
  min-width: 390px;
  max-width: 500%;
  display: flex;
  flex-direction: column;
}
.card {
  background: #fff;
  border-radius: .75rem;
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.1);
  margin-bottom: 24px;
  min-width: 0;
  display: flex;
  flex-direction: column;
}
.main-header-align {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  padding: 22px 36px 10px 36px;
}
.main-title-block {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}
.main-card-title {
  margin: 0 0 6px 0;
  font-size: 1.15rem;
  color: #7b8aaf;
  font-weight: 600;
  letter-spacing: -0.5px;
}
.main-card-category {
  margin: 0;
  font-size: .98rem;
  color: #adb5bd;
  letter-spacing: -0.5px;
}
.main-total-gauge {
  display: flex;
  align-items: center;
  gap: 18px;
  min-width: 0;
  flex-wrap: nowrap;
}
.main-total-label p {
  margin: 0 0 2px 0;
  font-size: .98rem;
  color: #adb5bd;
}
.main-total-label h3 {
  margin: 0;
  font-size: 1.25rem;
  font-weight: 700;
  color: #222;
}
.card-header {
  padding: 38px 36px 10px 36px;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}
.padded-chart {
  padding: 1rem 0 1.1rem 0;
}
.stats-row {
  display: flex;
  gap: 1rem;
  margin-bottom: 24px;
}
.stat-card {
  flex: 1;
  min-width: 0;
}
.stat-header {
  padding: 18px 22px 12px 22px;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}
.stat-body {
  padding-top: 8px;
}
.stat-header .icon-title {
  margin: 0 0 6px 0;
  font-size: .98rem;
  color: #7b8aaf;
  font-weight: 600;
  letter-spacing: -0.5px;
}
.stat-header .icon-value {
  font-size: 1.25rem;
  font-weight: 700;
  margin: 0 0 4px 0;
  color: #222;
}
.card-header.with-gauge {
  position: relative;
  padding: 38px 36px 10px 36px;
  min-height: 95px;
}
.main-total-gauge {
  position: absolute;
  top: 24px;
  right: 42px;
  display: flex;
  align-items: center;
  gap: 0px;
  min-width: 0;
  flex-wrap: nowrap;
}
.card-header.main-header-align {
  padding: 22px 36px 10px 36px;
  min-height: 60px;
}
</style>

<template>
  <div class="approval-search-fields">
    <!-- 첫 번째 행 - 항상 보임 -->
    <div class="row">
      <div class="field">
        <label>결재 코드</label>
        <input v-model="filters.approvalCode" placeholder="예: A-2501-0001" type="text" />
      </div>
      <div class="field">
        <label>결재 제목</label>
        <input v-model="filters.title" placeholder="예: 홍길동 냉장고 계약 건" type="text" />
      </div>
      <div class="field">
        <label>결재 내용</label>
        <input v-model="filters.content" placeholder="예: 2년 재계약" type="text" />
      </div>
      <div class="field">
        <label>결재 상태</label>
        <select v-model="filters.status">
          <option value="">전체</option>
          <option value="요청">요청</option>
          <option value="승인">승인</option>
          <option value="반려">반려</option>
        </select>
      </div>
    </div>
    
    <!-- 나머지 행들 - expanded가 true일 때만 보임 -->
    <template v-if="expanded">
      <div class="row">
        <div class="field">
          <label>결재 요청일</label>
          <div class="date-inputs">
            <input v-model="filters.requestDateStart" type="date" placeholder="시작일" />
            <span class="date-separator">~</span>
            <input v-model="filters.requestDateEnd" type="date" placeholder="종료일" />
          </div>
        </div>
        <div class="field">
          <label>결재 일자</label>
          <div class="date-inputs">
            <input v-model="filters.approvalDateStart" type="date" placeholder="시작일" />
            <span class="date-separator">~</span>
            <input v-model="filters.approvalDateEnd" type="date" placeholder="종료일" />
          </div>
        </div>
      </div>
      <div class="row">
        <div class="field">
          <label>요청 담당자</label>
          <input v-model="filters.requester" placeholder="예: 홍길동" type="text" />
        </div>
        <div class="field">
          <label>결재 담당자</label>
          <input v-model="filters.approver" placeholder="예: 유관순" type="text" />
        </div>
        <div class="field">
          <label>결재 코멘트</label>
          <input v-model="filters.comment" placeholder="예: 계약서 내용 미흡" type="text" />
        </div>
        <div class="field">
          <label>계약 코드</label>
          <input v-model="filters.contractCode" placeholder="예: C-2501-0001" type="text" />
        </div>
      </div>
    </template>
  </div>
</template>

<script setup>
defineProps({ 
  filters: Object,
  expanded: { type: Boolean, default: false }
})
</script>

<style scoped>
.approval-search-fields {
  padding: 16px 24px;
  border-radius: 8px;
  margin-bottom: 12px;
}

.row {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 20px;
  margin-bottom: 10px;
}

.row:nth-child(2) {
  grid-template-columns: 1fr 1fr;
}

.date-inputs {
  display: grid;
  grid-template-columns: 1fr auto 1fr;
  gap: 5px;
  align-items: center;
}

.date-separator {
  text-align: center;
  font-weight: 600;
  color: #666;
  font-size: 14px;
}

.row:last-child {
  grid-template-columns: repeat(4, 1fr);
}

.field {
  display: flex;
  flex-direction: column;
}

.field label {
  font-size: 13px;
  font-weight: 600;
  margin-bottom: 2px;
  color: #333;
}

.field input,
.field select {
  padding: 4px 8px;
  font-size: 13px;
  border: 1px solid #ccc;
  border-radius: 4px;
  background-color: white;
  appearance: none;
  -webkit-appearance: none;
  -moz-appearance: none;
  vertical-align: middle;
  line-height: 1.5;
  width: 100%;
  box-sizing: border-box;
  min-width: 0;
}

.field input:focus,
.field select:focus {
  outline: none;
  border-color: #007bff;
  box-shadow: none;
}
</style> 
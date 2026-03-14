<script setup lang="ts">
import { computed, ref, watch } from 'vue'

type Rank = string | number

export type AwardRow = {
  name: string
  award: string
  rank: Rank
}

const props = withDefaults(
  defineProps<{
    rows: AwardRow[]
    placeholder?: string
    namePlaceholder?: string
    awardPlaceholder?: string
    rankPlaceholder?: string
    showAwardFilter?: boolean
    showCount?: boolean
    paginate?: boolean
    pageSize?: number
    pageSizeOptions?: number[]
    showPageSize?: boolean
    initialQuery?: string
    initialAward?: string
    emptyText?: string
  }>(),
  {
    rows: () => [],
    placeholder: '筛选（姓名/奖项/排名）',
    namePlaceholder: '按姓名筛选',
    awardPlaceholder: '按奖项筛选',
    rankPlaceholder: '按排名筛选',
    showAwardFilter: true,
    showCount: false,
    paginate: true,
    pageSize: 10,
    pageSizeOptions: () => [10, 20, 50],
    showPageSize: true,
    initialQuery: '',
    initialAward: '全部',
    emptyText: '无匹配结果'
  }
)

const nameQuery = ref(props.initialQuery)
const awardQuery = ref(props.initialAward === '全部' ? '' : props.initialAward)
const rankQuery = ref('')

const page = ref(1)
const pageSize = ref(Math.max(1, props.pageSize))

watch(
  () => props.initialQuery,
  (v) => {
    nameQuery.value = v ?? ''
  }
)

watch(
  () => props.initialAward,
  (v) => {
    const next = v ?? ''
    awardQuery.value = next === '全部' ? '' : next
  }
)

watch(
  () => props.pageSize,
  (v) => {
    pageSize.value = Math.max(1, Number(v ?? 10) || 10)
    page.value = 1
  }
)

const awardOptions = computed(() => {
  const uniq = new Set<string>()
  for (const row of props.rows) {
    const a = (row?.award ?? '').trim()
    if (a) uniq.add(a)
  }
  return Array.from(uniq)
})

function normalize(value: unknown) {
  return String(value ?? '').trim().toLowerCase()
}

const filteredRows = computed(() => {
  const nq = normalize(nameQuery.value)
  const aq = normalize(awardQuery.value)
  const rq = normalize(rankQuery.value)

  const base = props.rows.filter((row) => {
    if (!row) return false
    if (nq && !normalize(row.name).includes(nq)) return false
    if (aq && !normalize(row.award).includes(aq)) return false
    if (rq && !normalize(row.rank).includes(rq)) return false
    return true
  })

  return base
})

watch([nameQuery, awardQuery, rankQuery], () => {
  page.value = 1
})

const totalPages = computed(() => {
  if (!props.paginate) return 1
  return Math.max(1, Math.ceil(filteredRows.value.length / pageSize.value))
})

watch(totalPages, (tp) => {
  if (page.value > tp) page.value = tp
})

const pagedRows = computed(() => {
  if (!props.paginate) return filteredRows.value
  const start = (page.value - 1) * pageSize.value
  return filteredRows.value.slice(start, start + pageSize.value)
})

function goPrev() {
  page.value = Math.max(1, page.value - 1)
}

function goNext() {
  page.value = Math.min(totalPages.value, page.value + 1)
}
</script>

<template>
  <div class="award-table award-table__card">
    <div class="award-table__toolbar">
      <input
        v-model="nameQuery"
        class="award-table__input"
        type="search"
        :placeholder="namePlaceholder || placeholder"
        autocomplete="off"
        spellcheck="false"
      />
      <input
        v-if="showAwardFilter"
        v-model="awardQuery"
        class="award-table__input"
        type="search"
        :placeholder="awardPlaceholder"
        list="award-table__award-options"
        autocomplete="off"
        spellcheck="false"
      />
      <datalist id="award-table__award-options">
        <option v-for="opt in awardOptions" :key="opt" :value="opt" />
      </datalist>
      <input
        v-model="rankQuery"
        class="award-table__input"
        type="search"
        :placeholder="rankPlaceholder"
        autocomplete="off"
        spellcheck="false"
      />
      <span v-if="showCount" class="award-table__count" aria-label="当前显示条目数">
        {{ filteredRows.length }}/{{ rows.length }}
      </span>
    </div>

    <div class="award-table__wrap">
      <table class="award-table__table">
        <thead>
          <tr>
            <th class="award-table__th">姓名</th>
            <th class="award-table__th">奖项</th>
            <th class="award-table__th">排名</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(row, i) in pagedRows" :key="`${row.name}-${row.award}-${row.rank}-${i}`">
            <td>{{ row.name }}</td>
            <td>{{ row.award }}</td>
            <td>{{ row.rank }}</td>
          </tr>
          <tr v-if="pagedRows.length === 0">
            <td class="award-table__empty" colspan="3">
              {{ emptyText }}
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <div v-if="paginate && totalPages > 1" class="award-table__footer">
      <div class="award-table__pager">
        <button class="award-table__btn" type="button" :disabled="page <= 1" @click="goPrev">
          上一页
        </button>
        <span class="award-table__page">
          第 {{ page }} / {{ totalPages }} 页
        </span>
        <button class="award-table__btn" type="button" :disabled="page >= totalPages" @click="goNext">
          下一页
        </button>
      </div>
      <div v-if="showPageSize" class="award-table__pageSize">
        <span class="award-table__pageLabel">每页</span>
        <select v-model.number="pageSize" class="award-table__select" aria-label="每页条数">
          <option v-for="n in pageSizeOptions" :key="n" :value="n">
            {{ n }}
          </option>
        </select>
        <span class="award-table__pageLabel">条</span>
      </div>
    </div>
  </div>
</template>

<style scoped>
.award-table__card {
  border: 1px solid var(--vp-c-divider);
  border-radius: 14px;
  background: var(--vp-c-bg-soft);
  box-shadow:
    0 1px 2px rgba(0, 0, 0, 0.05),
    0 12px 30px rgba(0, 0, 0, 0.06);
  padding: 14px 14px 10px;
}

.award-table__toolbar {
  display: grid;
  grid-template-columns: repeat(3, minmax(140px, 1fr));
  gap: 8px;
  align-items: center;
  margin: 6px 0 12px;
}

.award-table__input {
  border: 1px solid var(--vp-c-divider);
  border-radius: 8px;
  background: var(--vp-c-bg);
  color: var(--vp-c-text-1);
  padding: 6px 10px;
  line-height: 20px;
  transition: border-color 0.15s ease, box-shadow 0.15s ease;
}

.award-table__input:focus {
  outline: none;
  border-color: var(--vp-c-brand-1);
  box-shadow: 0 0 0 3px color-mix(in srgb, var(--vp-c-brand-1) 20%, transparent);
}

.award-table__count {
  grid-column: 1 / -1;
  justify-self: end;
  color: var(--vp-c-text-2);
  font-size: 12px;
}

.award-table__wrap {
  overflow-x: hidden;
  border-radius: 12px;
  background: var(--vp-c-bg);
  border: 1px solid var(--vp-c-divider);
  width: min(1600px, 100%);
  margin: 0 auto;
}

.award-table__table {
  width: 100%;
  border-collapse: collapse;
  table-layout: fixed;
}

.award-table__table th,
.award-table__table td {
  border-bottom: 1px solid var(--vp-c-divider);
  padding: 20px 22px;
  text-align: left;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  font-size: 18px;
  line-height: 30px;
}

.award-table__table thead {
  background: color-mix(in srgb, var(--vp-c-bg) 92%, var(--vp-c-divider));
}

.award-table__th {
  color: var(--vp-c-text-1);
  font-weight: 650;
  font-size: 18px;
  user-select: none;
  position: sticky;
  top: 0;
  background: inherit;
}

.award-table__table tbody tr:hover td {
  background: color-mix(in srgb, var(--vp-c-bg) 86%, var(--vp-c-divider));
}

.award-table__empty {
  color: var(--vp-c-text-2);
  text-align: center;
  padding: 14px 10px;
}

/* Footer / pagination */
.award-table__footer {
  display: flex;
  gap: 10px;
  align-items: center;
  justify-content: space-between;
  padding-top: 10px;
}

.award-table__pager {
  display: inline-flex;
  gap: 10px;
  align-items: center;
}

.award-table__btn {
  border: 1px solid var(--vp-c-divider);
  background: var(--vp-c-bg);
  color: var(--vp-c-text-1);
  border-radius: 10px;
  padding: 6px 10px;
  line-height: 18px;
  transition: background 0.15s ease, border-color 0.15s ease, opacity 0.15s ease;
}

.award-table__btn:hover:enabled {
  border-color: var(--vp-c-brand-1);
  background: color-mix(in srgb, var(--vp-c-bg) 90%, var(--vp-c-brand-1));
}

.award-table__btn:disabled {
  opacity: 0.45;
  cursor: not-allowed;
}

.award-table__page {
  color: var(--vp-c-text-2);
  font-size: 12px;
}

.award-table__pageSize {
  display: inline-flex;
  gap: 6px;
  align-items: center;
}

.award-table__pageLabel {
  color: var(--vp-c-text-2);
  font-size: 12px;
}

.award-table__select {
  border: 1px solid var(--vp-c-divider);
  border-radius: 10px;
  background: var(--vp-c-bg);
  color: var(--vp-c-text-1);
  padding: 6px 10px;
  line-height: 18px;
}

@media (max-width: 640px) {
  .award-table__toolbar {
    grid-template-columns: 1fr;
  }

  .award-table__count {
    justify-self: start;
  }

  .award-table__footer {
    flex-direction: column;
    align-items: flex-start;
  }
}
</style>

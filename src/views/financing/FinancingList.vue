<template>
  <div class="page-content">
    <art-table-bar
      :showTop="false"
      @search="search"
      @reset="resetForm(searchFormRef)"
      @changeColumn="changeColumn"
      :columns="columns">

      <template #top>
        <el-form :model="searchForm" ref="searchFormRef" label-width="82px">
          <el-row :gutter="20">
            <ArtFormSelect label="融资主体" prop="orgId" v-model="searchForm.orgId" :options="orgs" />
            <ArtFormSelect label="金融机构" prop="financialInstitution" v-model="searchForm.financialInstitution" :options="financingInstitution" />
            <ArtFormSelect label="融资方式" prop="fundingMode" v-model="searchForm.fundingMode" :options="fundingMode" />
            <el-col :xs="24" :sm="12" :lg="6">
              <el-form-item label="资本结构" prop="capitalStructure">
                <el-tree-select v-model="searchForm.capitalStructure" clearable :data="capitalStructure" />
              </el-form-item>
            </el-col>
          </el-row>
          <el-row :gutter="20">
            <ArtFormInput label="融资名称" prop="financingName" v-model="searchForm.financingName" />
          </el-row>
        </el-form>
      </template>
      <template #bottom>
        <el-button v-ripple>新增融资</el-button>
      </template>
    </art-table-bar>

    <art-table :columns="columns" :data="debt" :currentPage="1" :pageSize="10" :total="debt.length">
      <template #default>
        <el-table-column 
          label="融资实体" 
          prop="orgId"
          #default="scope"
          v-if="columns[0].show">
          {{ orgs.find(item => item.value === scope.row.orgId)?.label }}
        </el-table-column>
        <el-table-column label="金融机构" prop="financialInstitution" v-if="columns[1].show" />
        <el-table-column label="合同金额（万元）" prop="contractAmount" v-if="columns[2].show" />
        <el-table-column 
          label="到账金额（万元）" 
          prop="actualAmountReceived" 
          v-if="columns[3].show">
          <template #default="scope">
            <el-tag :type="scope.row.contractAmount == scope.row.actualAmountReceived ? 'primary' : 'warning'">
              {{ scope.row.actualAmountReceived }}
            </el-tag>
          </template>
        </el-table-column>
        <el-table-column label="执行利率" prop="appliedInterestRate" v-if="columns[4].show" />
        <el-table-column label="起息日" prop="interestStartDate" v-if="columns[5].show" />
        <el-table-column label="到期日" prop="finalRePaymentDate" v-if="columns[6].show" />
        <el-table-column label="融资期限（月）" prop="financingTerm" v-if="columns[7].show" />
        <el-table-column label="融资方式" prop="fundingMode" v-if="columns[8].show" />
        <el-table-column label="资本结构" prop="capitalStructure" v-if="columns[9].show" />
        <el-table-column label="操作" fixed="right" v-if="columns[10].show">
          <template #default="scope">
            <ArtButtonTable type="more"  />
            <ArtButtonTable type="delete" />
          </template>
        </el-table-column>
      </template>
    </art-table>

  </div>
</template>

<script setup lang="ts">
  import { orgs } from '@/mock/data/orgs'
  import { financingInstitution } from '@/mock/data/financialInstitution'
  import { capitalStructure } from '@/mock/data/capitalStructure'
  import { fundingMode } from '@/mock/data/fundingMode'
  import type { FormInstance } from 'element-plus'
  import {debt} from '@/mock/data/debt'

  const columns = reactive([
    { name: '融资主体', show: true },
    { name: '金融机构', show: true },
    { name: '合同金额', show: true },
    { name: '到账金额', show: true },
    { name: '执行利率', show: true },
    { name: '起息日', show: true },
    { name: '到期日', show: true },
    { name: '融资期限', show: true },
    { name: '融资方式', show: true },
    { name: '资本结构', show: true },
    { name: '操作', show: true }
  ])

  const searchFormRef = ref<FormInstance>()

  const searchForm = ref({
    orgId: '',
    financialInstitution: '',
    fundingMode: '',
    capitalStructure: '',
    financingName: ''
  })

  const search = () => {}

  const changeColumn = () => {}

  const resetForm = (formEl: FormInstance | undefined) => {
    if (!formEl) return
    formEl.resetFields()
  }
</script>

<style lang="scss" scoped>
  .page-content {
    width: 100%;
    height: 100%;
  }
</style>
<template>
  <div class="page-content">
    <el-form>
      <el-row justify="space-between" :gutter="20">
        <el-col :lg="6" :md="8" :sm="14" :xs="16">
          <el-form-item label="融资主体">
            <el-select v-model="entity" filterable clearable>
              <el-option 
                v-for="item in orgs" 
                :key="item.value" 
                :label="item.label" 
                :value="item.value" />
            </el-select>
          </el-form-item>
        </el-col>
        <el-col :lg="6" :md="8" :sm="14" :xs="16">
          <el-form-item label="金融机构">
            <el-select v-model="finName" filterable clearable>
              <el-option 
                v-for="item in bankList" 
                :key="item.id" 
                :label="item.bankName" 
                :value="item.id" />
            </el-select>
          </el-form-item>
        </el-col>
        <el-col :lg="6" :md="8" :sm="14" :xs="16">
          <el-form-item label="融资方式">
            <el-select v-model="finMethod" filterable clearable>
              <el-option 
                v-for="item in financingMethod" 
                :key="item.value" 
                :label="item.label" 
                :value="item.value" />
            </el-select>
          </el-form-item>
        </el-col>
        <el-col :lg="6" :md="8" :sm="14" :xs="16">
          <el-form-item label="融资结构">
            <el-tree-select v-model="finStructure" filterable clearable :data="financingStructure" />
          </el-form-item>
        </el-col>
      </el-row>
      <el-row justify="space-between" :gutter="20">
        <el-col :lg="6" :md="8" :sm="14" :xs="16">
          <el-form-item label="债务名称">
            <el-input v-model="debtName" />
          </el-form-item>
        </el-col>
        <el-col :lg="6" :md="8" :sm="14" :xs="16" style="display: flex; justify-content: end">
          <el-button type="primary">搜索</el-button>
          <el-button type="primary">重置</el-button>
        </el-col>

      </el-row>
    </el-form>
  </div>
</template>

<script setup lang="ts">
import { orgs } from '@/mock/temp/orgs'
import { bankList } from '@/mock/temp/bankList'
import { financingStructure } from '@/mock/temp/financingStructure'
import { financingMethod } from '@/mock/temp/financingMethod'
  

  let entity = ref()
  let finName = ref()
  let debtName = ref()
  let finStructure = ref()
  let finMethod = ref()
</script>

<style lang="scss" scoped>
  .page-content {
    width: 100%;
    height: 100%;
  }
</style>
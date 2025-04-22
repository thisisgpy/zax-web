<template>
  <div class="page-content">
    <div class="header">
      <h3>基础信息</h3>
    </div>
    <el-form>
      <el-form-item label="融资主体">
        <el-tree-select v-model="entity" filterable :data="data" />
      </el-form-item>
    </el-form>
  </div>
</template>

<script setup lang="ts">
  const data = [{
    value: 1,
    label: '成都蜀州城市发展集团有限公司',
    children: [
      {
        value: 2,
        label: '成都市蜀州兴业实业发展有限公司'
      },
      {
        value: 3,
        label: '崇州市蜀州交通发展有限公司',
        children: [
          {
            value: 4,
            label: '崇州市公共交通有限公司'
          }
        ]
      }
    ]
  }]
  let entity = ref('')
</script>

<style lang="scss" scoped>
  .page-content {
    width: 100%;
    height: 100%;
  }
</style>
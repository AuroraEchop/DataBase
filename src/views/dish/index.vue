<script setup lang="ts">
import { reactive, ref } from 'vue'
// 删除了 getCategoryPageListAPI
import { getDishPageListAPI, updateDishStatusAPI, deleteDishesAPI } from '@/api/dish'
import { ElMessage, ElMessageBox, ElTable } from 'element-plus'
import { useRouter } from 'vue-router'
import { Plus } from '@element-plus/icons-vue'

// ------ .d.ts 属性类型接口 ------
// 你的 interface 保持不变
interface dish {
  shipId: number
  shipName: string
  imo: string
  mmsi: string
  status: string
  buildYear: string
}
// 删除了 interface Category

// ------ 数据 ------

// 当前页的菜品列表
const dishList = ref<dish[]>([])
// 删除了 categoryList
// 分页参数
const pageData = reactive({
  shipName: '', // 【修改1】name -> shipName
  // categoryId: '', // 【修改2】删除 categoryId
  status: '',
  page: 1,
  pageSize: 6
})
const total = ref(0)
const options = [
  {
    value: '1',
    label: '起售'
  },
  {
    value: '0',
    label: '停售'
  }
]

// ------ 方法 ------

// 【修改3】删除了 init 函数

// 刷新页面数据
const showPageList = async () => {
  const { data: res } = await getDishPageListAPI(pageData)
  console.log('菜品列表')
  console.log(res.data)
  dishList.value = res.data.list // 【修改4】records -> list
  total.value = res.data.total
}
// 删除了 init()
showPageList() // 页面一开始就要展示分页菜品列表

// 监听翻页和每页显示数量的变化
const handleCurrentChange = (val: number) => {
  pageData.page = val
  showPageList()
}

const handleSizeChange = (val: number) => {
  pageData.pageSize = val
  showPageList()
}

const multiTableRef = ref<InstanceType<typeof ElTable>>()
const multiSelection = ref<dish[]>([])

const handleSelectionChange = (val: dish[]) => {
  multiSelection.value = val
}

// 新增和修改菜品都是同一个页面，不过要根据路径传参的方式来区分
const router = useRouter()
const to_add_update = (row?: any) => {
  console.log('看有没有传过来，来判断要add还是update', row)
  if (row && row.shipId) {
    // 【修改5】ShipId -> shipId
    router.push({
      path: '/dish/add',
      query: { id: row.shipId } // 【修改6】ShipId -> shipId
    })
  } else {
    router.push('/dish/add')
  }
}

// 修改菜品状态
const change_btn = async (row: any) => {
  console.log('要修改的行数据')
  console.log(row)
  await updateDishStatusAPI(row.shipId) // 【修改7】ShipId -> shipId
  // 修改后刷新页面，更新数据
  await showPageList()
  ElMessage({
    type: 'success',
    message: '修改成功'
  })
}

// 删除菜品
const deleteBatch = (row?: any) => {
  console.log('要删除的行数据')
  console.log(row)
  ElMessageBox.confirm('该操作会永久删除菜品，是否继续？', 'Warning', {
    confirmButtonText: 'OK',
    cancelButtonText: 'Cancel',
    type: 'warning'
  })
    .then(async () => {
      // 1. 没传入行数据，批量删除
      if (row == undefined) {
        if (multiSelection.value.length == 0) {
          ElMessage({
            type: 'warning',
            message: '请先选择要删除的菜品'
          })
          return
        }
        let ids: any = []
        multiSelection.value.map((item) => {
          ids.push(item.shipId) // (这里你原来就是对的)
        })
        ids = ids.join(',')
        console.log('ids', ids)
        let res = await deleteDishesAPI(ids)
        if (res.data.code != 0) return
      }
      // 2. 传入行数据，单个删除
      else {
        console.log('id包装成数组，然后调用批量删除接口')
        console.log(row.shipId) // 【修改8】ShipId -> shipId
        let res = await deleteDishesAPI(row.shipId) // 【修改9】ShipId -> shipId
        if (res.data.code != 0) return
      }
      // 删除后刷新页面，更新数据
      await showPageList()
      ElMessage({
        type: 'success',
        message: '删除成功'
      })
    })
    .catch(() => {
      ElMessage({
        type: 'info',
        message: '取消删除'
      })
    })
}
</script>

<template>
  <el-card>
    <div class="horizontal">
      <el-input size="large" class="input" v-model="pageData.shipName" placeholder="请输入船舶名" />
      <el-select
        class="input"
        clearable
        v-model="pageData.status"
        placeholder="选择菜品状态"
        size="large"
      >
        <el-option
          v-for="item in options"
          :key="item.value"
          :label="item.label"
          :value="item.value"
        />
      </el-select>
      <el-button size="large" class="btn" round type="success" @click="showPageList()"
        >查询菜品</el-button
      >
      <el-button size="large" class="btn" round type="danger" @click="deleteBatch()"
        >批量删除</el-button
      >
      <el-button size="large" class="btn" type="primary" @click="to_add_update()">
        <el-icon style="font-size: 15px; margin-right: 10px"> <Plus /> </el-icon>添加菜品
      </el-button>
    </div>
    <el-table
      class="table_box"
      ref="multiTableRef"
      :data="dishList"
      stripe
      @selection-change="handleSelectionChange"
    >
      <el-table-column type="selection" width="55" />

      <el-table-column prop="shipName" label="船舶名称" align="center" />
      <el-table-column prop="imo" label="IMO" align="center" />
      <el-table-column prop="mmsi" label="MMSI" align="center" />
      <el-table-column prop="builtYear" label="建造年份" align="center" />

      <el-table-column prop="status" label="状态" align="center">
        <template #default="scope">
          <el-tag :type="scope.row.status === '1' ? 'success' : 'danger'" round>
            {{ scope.row.status === '1' ? '启售' : '停售' }}
          </el-tag>
        </template>
      </el-table-column>

      <el-table-column label="操作" width="200px" align="center">
        <template #default="scope">
          <el-button @click="to_add_update(scope.row)" type="primary">修改</el-button>
          <el-button
            @click="change_btn(scope.row)"
            plain
            :type="scope.row.status === '1' ? 'danger' : 'primary'"
          >
            {{ scope.row.status === '1' ? '停售' : '起售' }}</el-button
          >
        </template>
      </el-table-column>
      <template #empty>
        <el-empty description=" 没有数据" />
      </template>
    </el-table>

    <el-pagination
      class="page"
      background
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
      :page-sizes="[2, 4, 6, 8]"
      v-model:current-page="pageData.page"
      v-model:page-size="pageData.pageSize"
      @current-change="handleCurrentChange"
      @size-change="handleSizeChange"
    />
  </el-card>
</template>

<style lang="less" scoped>
/* 样式保持不变 */
.el-table {
  width: 90%;
  height: 500px;
  margin: 3rem auto;
  text-align: center;
  border: 1px solid #e4e4e4;
}

:deep(.el-table tr) {
  font-size: 12px;
}

.el-button {
  width: 45px;
  font-size: 12px;
}

.el-pagination {
  justify-content: center;
}

body {
  background-color: #c91c1c;
}

.horizontal {
  display: flex;
  justify-content: space-around;
  align-items: center;
  margin: 0 80px;

  .input {
    width: 160px;
  }

  .btn {
    width: 120px;
  }
}

img {
  width: 50px;
  height: 50px;
  border-radius: 10px;
}

.add_btn {
  width: 100px;
  height: 40px;
  margin-left: 900px;
}
</style>
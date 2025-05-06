<script setup>
import { ref,watch,onMounted } from "vue";
import { queryPageApi,queryAllApi } from "@/api/emp";
//搜索表单对象
const searchEmp = ref({ name: '', gender: '', date: [],begin:'',end:'' });

onMounted(() => {
  search();
})


//查询
const search = async () => {
  const result = await queryPageApi(searchEmp.value.name, searchEmp.value.gender, searchEmp.value.begin,
                                  searchEmp.value.end,currentPage.value,pageSize.value);
  if (result.code) {
    empList.value = result.data.rows;
    total.value = result.data.total;
  }
}


const currentPage = ref(1)
const pageSize = ref(10)
const background = ref(true)
const total = ref(0)

const handleSizeChange = (val) => {
  search();
}
const handleCurrentChange = (val) => {
  search();
}

// 示例数据
const empList = ref([])


const clear = () => {
  // 清空表单
  searchEmp.value = {
    name: '',
    gender: '',
    date: [],
    begin: '',
    end: ''
  }
  search()
}

//侦听searchEmp中的date属性
watch(
  () => searchEmp.value.date,
  (newValue, oldValue) => {
     if(newValue.length == 2){
      searchEmp.value.begin = newValue[0]
      searchEmp.value.end = newValue[1]
     }else {
      searchEmp.value.begin = ''
      searchEmp.value.end = ''
     }
  }
)

</script>

<template>
  <h1>员工管理</h1>

  <!-- 搜索栏 -->
  <div class="container">
    <el-form
      :inline="true"
      :model="searchEmp"
      class="demo-form-inline"
    >
      <el-form-item label="姓名">
        <el-input
          v-model="searchEmp.name"
          placeholder="请输入员工姓名"
          clearable
        />
      </el-form-item>

      <el-form-item label="性别">
        <el-select
          v-model="searchEmp.gender"
          placeholder="请选择"
          clearable
        >
          <el-option label="男" value="1" />
          <el-option label="女" value="2" />
        </el-select>
      </el-form-item>

      <el-form-item label="入职时间">
        <el-date-picker
          v-model="searchEmp.date"
          type="daterange"
          range-separator="到"
          start-placeholder="开始日期"
          end-placeholder="结束日期"
          value-format="YYYY-MM-DD"
        />
      </el-form-item>

      <el-form-item>
        <el-button type="primary" @click="search">查询</el-button>
        <el-button type="info" @click="clear">清空</el-button>
      </el-form-item>
    </el-form>
  </div>

  <!-- 功能按钮 -->
  <div class="container">
    <el-button type="primary" @click="search"> + 新增员工</el-button>
    <el-button type="danger" @click="clear"> - 批量删除</el-button>
  </div>

  <!-- 表单 -->
  <div>
    <el-table :data="empList" border style="width: 100%">
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column prop="name" label="姓名" width="120" align="center"/>
      <el-table-column label="性别" width="120" align="center">
        <template #default="scope">
          {{ scope.row.gender == 1? '男' : '女' }}
        </template>
      </el-table-column>
      <el-table-column label="头像" align="center" width="120">
        <template #default="scope">
          <img :src="image">
        </template>
      </el-table-column>
      <el-table-column prop="deptName" label="所属部门" width="120" align="center"/>

      <el-table-column label="职位" width="120" align="center">
        <template #default="scope">
          <span v-if="scope.row.job == 1">班主任</span>
          <span v-else-if="scope.row.job == 2">讲师</span>
          <span v-else-if="scope.row.job == 3">学工主管</span>
          <span v-else-if="scope.row.job == 4">教研主管</span>
          <span v-else-if="scope.row.job == 5">咨询师</span>
          <span v-else>其他</span>
        </template>
      </el-table-column>

      <el-table-column prop="entryDate" label="入职日期" width="180" align="center"/>
      <el-table-column prop="updateTime" label="最后操作时间" width="180" align="center"/>
      <el-table-column label="操作" align="center">
        <template #default="scope">
          <el-button type="primary" size="small" @click="edit(scope.row.id)" round><el-icon><Edit /></el-icon>编辑</el-button>
          <el-button type="danger" size="small" @click="delById(scope.row.id)" round><el-icon><Delete /></el-icon>删除</el-button>
        </template>
      </el-table-column>
    </el-table>
  </div>
  <!-- 分页 -->
  <div class="container">
    <el-pagination
      v-model:current-page="currentPage"
      v-model:page-size="pageSize"
      :page-sizes="[5, 10, 20, 50]"
      :background="background"
      :total="total"
      layout="total, sizes, prev, pager, next, jumper"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
    />
  </div>

  
</template>

<style scoped>
  .container {
    margin:10px 0px; 
  }
</style>

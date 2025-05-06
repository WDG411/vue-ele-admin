<script setup>
import { ref, onMounted } from 'vue'
import { queryAllApi, addDeptApi } from "@/api/dept";
import { ElMessage } from 'element-plus';

const deptList = ref([])

onMounted(() => {
  search()
})

const search = async () => {
  const result = await queryAllApi();
  if (result.code) {
    deptList.value = result.data;
  }
}

const dialogFormVisible = ref(false);

const dept = ref({ name: '' });

const formTitle = ref('');
const addDept = () => {
  formTitle.value = '新增部门';
  dialogFormVisible.value = true;
  dept.value = { name: '' };
}

const save = async () => {
  const result = await addDeptApi(dept.value);
  if (result.code) {
    ElMessage.success('操作成功');

    dialogFormVisible.value = false;

    search();
  } else {
    ElMessage.error(result.msg());
  }
}

</script>

<template>
  <h1>部门管理</h1>

  <div class="container">
    <el-button type="primary" @click="addDept"> + 新增部门</el-button>
  </div>
  
  <!-- 数据展示表格 -->
  <div class="container">
    <el-table :data="deptList" border style="width: 100%">
      <el-table-column type=index label="序号" width="100" align="center"/>
      <el-table-column prop="name" label="部门名称" width="260" align="center"/>
      <el-table-column prop="updateTime" label="最后操作时间" width="300" align="center"/>

      <el-table-column label="操作"align="center">

        <template #default="scope">
          <el-button type="primary" size="small"><el-icon><Edit /></el-icon>编辑</el-button>
          <el-button type="danger" size="small"><el-icon><Delete /></el-icon>删除</el-button>
        </template>

      </el-table-column>
    </el-table>
  </div>

  <!-- 新增部门的对话框 -->
  <el-dialog v-model="dialogFormVisible" :title=formTitle width="500">
    <el-form :model="dept">
      <el-form-item label="部门名称" :label-width="formLabelWidth">
        <el-input v-model="dept.name" autocomplete="off" />
      </el-form-item>
    </el-form>
    <template #footer>
      <div class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取消</el-button>
        <el-button type="primary" @click="save">
          确定
        </el-button>
      </div>
    </template>
  </el-dialog>
  
</template>

<style scoped>

.container {
  margin: 10px 0px;
}

</style>

<script setup>
import { ref, onMounted,nextTick } from 'vue'
import { queryAllApi, addDeptApi,queryInfoApi,updateDeptApi,deleteDeptApi} from "@/api/dept";
import { ElMessage,ElMessageBox} from 'element-plus';

//声名列表展示数据
const deptList = ref([])

onMounted(() => {
  search()
})

//查询部门
const search = async () => {
  const result = await queryAllApi();
  if (result.code) {
    deptList.value = result.data;
  }
}

//对话框的状态
const dialogFormVisible = ref(false);

const dept = ref({ name: '', obj: '' });

const formTitle = ref('');

//查询回显
const edit = async (id) => {
  formTitle.value = '编辑部门';
  const result = await queryInfoApi(id);

  if (deptFormRef.value) {
    deptFormRef.value.resetFields();
  }
  
  if (result.code) {
    dialogFormVisible.value = true;
    dept.value = result.data;
  }
}

//新增部门
const addDept = () => {
  formTitle.value = '新增部门';
  dialogFormVisible.value = true;
  dept.value = { name: '' };
  if (deptFormRef.value) {
    deptFormRef.value.resetFields();
  }
  
}

//删除
const delById = async (id) => {
  ElMessageBox.confirm(
    '删除部门后将不可撤销',
    '提示',
    {
      confirmButtonText: '确认',
      cancelButtonText: '取消',
      type: 'error',
      draggable: 'true',
      roundButton:'false',
    }
  )
    .then(async () => {
      const result = await deleteDeptApi(id);
      if (result.code) {
        ElMessage.success('删除成功');
        search();
      } else {
        ElMessage.error(result.msg);
      }
      
    })
    // .catch(() => {
    //   ElMessage({
    //     type: 'info',
    //     message: 'Delete canceled',
    //   })
    // })
}

// 提交表单
const save = async () => {

  if (!deptFormRef.value) return;
  let result;
  deptFormRef.value.validate( async (valid) => {
    if (valid) {

      if (dept.value.id) {
        result = await updateDeptApi(dept.value);
      } else {
        result = await addDeptApi(dept.value);
      }
    }

    if (result.code) {
      ElMessage.success('操作成功');
      
      dialogFormVisible.value = false;
    
      search();
    }
    
    else {
      ElMessage.error('表单校验不通过');
    }
  })

}

const deptFormRef = ref();

const rules = ref({
  name: [
    { required: true, message: '请输入部门名称', trigger: 'blur' },
    { min: 2, max: 10, message: '长度在2-10个字符', trigger: 'blur' },
  ],
  obj: [
    { required: true, message: '请输入部门名称', trigger: 'blur' },
    { min: 2, max: 10, message: '长度在2-10个字符', trigger: 'blur' },
  ]
})

// 提交逻辑封装在函数里
// const submitForm = () => {
//   if (!deptFormRef.value) return;
//   deptFormRef.value.validate((valid, fields) => {
//     if (valid) {
//       save();   // 走你已有的保存逻辑
//     } else {
//       // 找到第一个校验失败字段并聚焦
//       const firstInvalid = Object.keys(fields)[0];
//       nextTick(() => {
//         const el = document.querySelector(`[prop="${firstInvalid}"] input`);
//         el && el.focus();
//       });
//       ElMessage.error('请正确填写表单！');
//     }
//   });
// };


const submitForm = () => {
  if (!deptFormRef.value) return

  deptFormRef.value.validate((valid) => {
    if (valid) {
      save()
    } else {
      nextTick(() => {
        // 拿到 el-form 的根节点
        const formEl = deptFormRef.value.$el
        // 找到第一个带 .is-error 的表单项
        const errItem = formEl.querySelector('.el-form-item.is-error')
        if (errItem) {
          // 在其内部找 input 或 textarea
          const focusEl = errItem.querySelector('input,textarea')
          focusEl && focusEl.focus()
        }
      })
      ElMessage.error('请正确填写表单！')
    }
  })
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
          <el-button type="primary" size="small" @click="edit(scope.row.id)" round><el-icon><Edit /></el-icon>编辑</el-button>
          <el-button type="danger" size="small" @click="delById(scope.row.id)" round><el-icon><Delete /></el-icon>删除</el-button>
        </template>

      </el-table-column>
    </el-table>
  </div>

  <!-- 对话框 -->
  <el-dialog v-model="dialogFormVisible" :title=formTitle width="500">
    <el-form :model="dept" ref="deptFormRef" :rules="rules">
      <el-form-item label="部门名称" prop="name" :label-width="formLabelWidth">
        <el-input v-model="dept.name" autocomplete="off"  @keydown.enter.native.prevent="submitForm"/>
      </el-form-item>
      <el-form-item label="部门obj" prop="obj" :label-width="formLabelWidth">
        <el-input v-model="dept.obj" autocomplete="off"  @keydown.enter.native.prevent="submitForm"/>
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

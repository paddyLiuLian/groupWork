<template>
  <div class="dept">
<!--    面包屑-->
   <div>
     <el-breadcrumb separator="/">
       <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
       <el-breadcrumb-item><a href="/">部门管理</a></el-breadcrumb-item>
     </el-breadcrumb>
   </div>
    <!--    操作-->
    <el-card>
      <el-button type="primary" @click="dialogFormVisible = true">添加部门</el-button>
    </el-card>
    <div>
      <!--      添加-->
      <el-dialog title="添加部门" :visible.sync="dialogFormVisible">
        <el-form :model="form">
          <el-form-item label="部门名称">
            <el-input v-model="form.name" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="部门描述">
            <el-input v-model="form.descript" autocomplete="off"></el-input>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="dialogFormVisible = false">取 消</el-button>
          <el-button type="primary" @click="addDept">添加</el-button>
        </div>
      </el-dialog>
    </div>
    <div>
      <el-table
        :data="deptData"
        height="550"
        border
        style="width: 100%">
        <el-table-column
          type="selection"
          width="50">
        </el-table-column>
        <el-table-column
          prop="DeptNo"
          label="部门编号"
          width="180">
        </el-table-column>
        <el-table-column
          prop="DeptName"
          label="部门名称"
          width="180">
        </el-table-column>
        <el-table-column
          prop="DeptDescript"
          label="描述">
        </el-table-column>
        <el-table-column
          label="状态">
          <template slot-scope="scope">
            <span :class="scope.row.Status===1?'statusActive':'statusDel'">{{scope.row.Status===1?'有效':'无效'}}</span>
          </template>
        </el-table-column>
        <el-table-column
          prop="CreateDate"
          :formatter="DateHandler"
          label="创建时间">
        </el-table-column>
        <el-table-column
          label="操作">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" circle @click="edit(scope.row)"></el-button>
            <el-button type="danger" icon="el-icon-delete" circle @click="del(scope.row.DeptNo)"></el-button>
          </template>
        </el-table-column>
      </el-table>
<!--      修改-->
      <el-dialog title="修改部门" :visible.sync="dialogFormEditVisible">
        <el-form :model="editform">
          <el-form-item label="部门名称">
            <el-input v-model="editform.name" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="部门描述">
            <el-input v-model="editform.descript" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="状态" class="selStatus">
            <br/>
            <el-select v-model="editform.status">
              <el-option label="有效" value="1"></el-option>
              <el-option label="无效" value="0"></el-option>
            </el-select>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="dialogFormEditVisible = false">取 消</el-button>
          <el-button type="primary" @click="saveDept">修改</el-button>
        </div>
      </el-dialog>
    </div>
  </div>
</template>

<script>
export default {
  name: 'DeptManage',
  data () {
    return {
      deptData: [],
      dialogFormVisible: false,
      dialogFormEditVisible: false,
      form: {
        name: '',
        descript: ''
      },
      editform: {
        deptNo: '',
        name: '',
        descript: '',
        status: 0
      }
    }
  },
  methods: {
    DateHandler: function (row, column) {
      return new Date(row.CreateDate).toLocaleDateString()
    },
    addDept: function () {
      this.$axios.post('/api/addDept', { deptName: this.form.name, deptDescipt: this.form.descript })
        .then((response) => {
          const result = response.data
          if (result.code === 200) {
            this.$alert(result.message)
            if (result.message === '添加成功') {
              this.getDeptList()
            }
            this.dialogFormVisible = false
          } else {
            this.$alert(result.message)
          }
        }).catch(() => {
          this.$alert('请求出错')
        })
    },
    getDeptList: function () {
      this.$axios.post('/api/deptList').then((response) => {
        const result = response.data
        if (result.code === 200) {
          this.deptData = result.data
        } else {
          this.$alert(result.message())
        }
      }).catch(() => {
      })
    },
    del: function (deptNo) {
      this.$axios.get('/api/delDept', {
        params: {
          deptNo: deptNo
        }
      }).then((response) => {
        const result = response.data;
        (result.message === '删除成功') ? this.getDeptList() : this.$alert(result.message)
      }).catch((err) => {
        console.log(err)
        this.$alert('请求出错，请检查')
      })
    },
    edit: function (obj) {
      this.dialogFormEditVisible = true
      this.editform.name = obj.DeptName
      this.editform.deptNo = obj.DeptNo
      this.editform.descript = obj.DeptDescript
      this.editform.status = obj.Status
    },
    saveDept: function () {
      this.$axios.post('/api/editDept', {
        deptNo: this.editform.deptNo,
        deptName: this.editform.name,
        deptDescript: this.editform.descript,
        status: this.editform.status
      }).then((response) => {
        const result = response.data
        console.log(result)
        if (result.message === '修改成功') {
          this.$alert(result.message)
          this.getDeptList()
          this.dialogFormEditVisible = false
        } else {
          this.$alert(result.message)
        }
      }).catch((err) => {
        console.log(err)
        this.$alert('请求出错')
      })
    }
  },
  mounted () {
    // 向后端服务器去请求数据
    this.getDeptList()
  }
}
</script>

<style scoped>
.dept .el-card {
  margin: 20px 0;
  text-align: left;
}
.statusActive {
  padding: 5px 10px;
  color: white;
  background-color: green;
}
.statusDel {
  padding: 5px 10px;
  color: white;
  background-color: red;
}
.selStatus {
  text-align: left;
}
</style>

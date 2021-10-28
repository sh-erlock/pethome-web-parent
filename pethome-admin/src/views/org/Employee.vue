<template>
  <section>
    <!--工具条-->
    <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
      <el-form :inline="true" :model="query">
        <el-form-item>
          <el-input v-model="query.keyword" placeholder="关键字"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="warning" v-on:click="search">查询</el-button>
        </el-form-item>
        <el-form-item>
          <el-button type="success" @click="handleAdd">新增</el-button>
        </el-form-item>
      </el-form>
    </el-col>

    <!--列表:优秀实践，copy domain修改列表-->
    <el-table :data="pageData.data" highlight-current-row v-loading="listLoading" @selection-change="selsChange" style="width: 100%;">
      <el-table-column type="selection" width="55">
      </el-table-column>
      <el-table-column type="index" label="序号" width="60">
      </el-table-column>
      <el-table-column prop="username" label="用户名" width="100" sortable>
      </el-table-column>
      <el-table-column prop="password" label="密码" width="280" sortable>
      </el-table-column>
      <el-table-column prop="email" label="邮箱" width="200" sortable>
      </el-table-column>
      <el-table-column prop="phone" label="电话" width="160" sortable>
      </el-table-column>
      <el-table-column prop="age" label="年龄" width="120" sortable>
      </el-table-column>
      <!--<el-table-column prop="state" label="状态" width="120" sortable :formatter="stateFormatter">-->
      <el-table-column prop="state" label="状态" width="100" sortable>
        <template scope="scope">
          <span style="color: green" v-if="scope.row.state===1">正常</span>
          <span style="color: red" v-else-if="scope.row.state===0">禁用</span>
          <span style="color:yellow" v-else>未知</span>
        </template>
      </el-table-column>
      <!--private Employee parent;//上级部门-->
      <el-table-column prop="department.name" label="所属部门" width="140" sortable>
      </el-table-column>
      <el-table-column label="操作" min-width="120">
        <template scope="scope">
          <el-button  type="primary" icon="el-icon-edit" size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
          <el-button type="danger" icon="el-icon-delete" size="small" @click="handleDel(scope.$index, scope.row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!--工具条-->
    <el-col :span="24" class="toolbar">
      <el-button type="danger" @click="batchRemove" :disabled="this.sels.length===0">批量删除</el-button>
      <el-pagination layout="prev, pager, next"
                     @current-change="handleCurrentChange"
                     :current-page="query.currentPage"
                     :page-size="query.pageSize"
                     :total="pageData.total"
                     style="float:right;">
      </el-pagination>
    </el-col>

    <!--编辑界面-->
    <el-dialog :title="title"  :visible.sync="employeeFormVisible" :close-on-click-modal="false">
      <!--ref="employeeForm" 指定表单名称，以后可以通过this.$refs.employeeForm-->
      <el-form :model="employee" label-width="80px" :rules="employeeFormRules" ref="employeeForm">
        <el-form-item label="名称" prop="username">
          <el-input v-model="employee.username" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="employee.password" show-password auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="employee.email" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="电话" prop="phone">
          <el-input v-model="employee.phone" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="年龄" prop="age">
          <el-input v-model="employee.age" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="所属部门" prop="department">
          <el-cascader v-model="employee.department"
                       :options="deptTree"
                       :props="{checkStrictly: true,label:'name',value:'id'}" clearable></el-cascader>

        </el-form-item>
        <el-form-item label="状态">
          <el-radio-group v-model="employee.state">
            <el-radio class="radio" :label="1">正常</el-radio>
            <el-radio class="radio" :label="0">禁用</el-radio>
          </el-radio-group>
        </el-form-item>

      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click.native="employeeFormVisible = false">取消</el-button>
        <el-button type="primary" @click.native="save" :loading="employeeLoading">保存</el-button>
      </div>
    </el-dialog>

  </section>
</template>

<script>
export default {
  data() {
    return {
      title:'',
      pageData:{
        total:0,//分页条
        data:[]//数据表格
      },
      query:{
        keyword:'',
        currentPage:1,//分页条
        pageSize:5 //分页条
      },
      listLoading: false,
      sels: [],//列表选中列
      employeeFormVisible: false,//编辑界面是否显示
      employeeLoading: false,
      employeeFormRules: {
        username: [
          { required: true, message: '--请输入用户名--', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '--请输入密码--', trigger: 'blur' }
        ]
      },
      //编辑界面数据
      employee: {
        id: null,
        password:'',
        username: '',
        email: '',
        phone:'',
        state: 1,
        age: '',
        department: {}
      },
      employees:[],
      deptTree:[]
    }
  },

  methods: {
    search(){
      //修改页码
      this.query.currentPage =1;
      this.getEmployees();
    },
    handleCurrentChange(val) {
      this.query.currentPage = val;
      this.getEmployees();
    },

    //获取用户列表
    getEmployees() {
      this.listLoading = true; //加载条
      this.$http.post("/emp",this.query)
          .then(result => {
            /*console.log(result)*/
            this.pageData.data = result.data.list;
            this.pageData.total = result.data.totals;

            this.listLoading = false; //去除加载条
          })
          .catch(result=>{
            alert("报错了！");
          })

    },
    //删除
    handleDel: function (index, row) {
      this.$confirm('确认删除该记录吗?', '提示', {
        type: 'warning'
      }).then(() => {
        this.listLoading = true;
        let url = "/emp/"+row.id;
        this.$http.delete(url)
            .then((result) => {
              // console.log(result)
              result = result.data;
              this.listLoading = false;
              if(result.success){ //成功提示
                this.$message({
                  message: '删除成功',
                  type: 'success'
                });
              }else{
                this.$message({ //错误提示
                  message: result.message,
                  type: 'error'
                });
              }
              //重新加载数据
              this.getEmployees();
            });
      }).catch(() => {
      });
    },
    //显示编辑界面
    handleEdit: function (index, row) {
      //修改对话框标题
      this.title = "修改员工"
      /*this.getEmployees();*/
      this.getDeptTree();
      //弹出对话框
      this.employeeFormVisible = true;
      //回显
      this.employee = Object.assign({}, row);

      if (this.employee.dirPath) {
        let dirPath = this.employee.dirPath;
        let arr = dirPath.split("/");
        let parr = [];
        for (let i = 1; i < arr.length - 1; i++) {
          parr.push(parseInt(arr[i]));
        }
        this.employee.department = parr;
      }


    },
    //显示新增界面
    handleAdd: function () {
      //修改对话框标题
      this.title = "添加员工"
      this.getEmployees();
      this.getDeptTree();
      //弹出对话框
      this.employeeFormVisible = true;
      //清空数据
      this.employee =  {
        id: null,
        password:'',
        username: '',
        email: '',
        phone:'',
        state: 1,
        age: '',
        department: {}
      };

    },

    //保存
    save: function () {
      this.$refs.employeeForm.validate((valid) => {
        if (valid) {
          this.$confirm('确认保存吗？', '提示', {}).then(() => {
            this.employeeLoading = true;
            let para = Object.assign({}, this.employee);
            let parentArr = para.department;
            if (parentArr){
              let parentId = parentArr[parentArr.length-1]
              para.department = {"id":parentId}
            }
            this.$http.put("/emp",para)
                .then((result) => {
                  this.employeeLoading = false;
                  if(result.data.success){
                    this.$message({
                      message: '提交成功',
                      type: 'success'
                    });
                    this.employeeFormVisible = false; //关闭对话框
                    this.$refs['employeeForm'].resetFields();//重置表单
                    this.getEmployees(); //重新加载
                  }else{
                    this.$message({
                      message: result.data.message,
                      type: 'error'
                    });
                  }
                });
          });
        }
      });
    },
    selsChange: function (sels) {
      this.sels = sels;
      // console.log(this.sels)
    },
    //批量删除
    batchRemove: function () {
      //遍历选择记录把id放到一个数组中
      var ids = this.sels.map(item => item.id); //[1,2,3]
      this.$confirm('确认删除选中记录吗？', '温馨提示', {
        type: 'warning'
      }).then(() => {
        this.listLoading = true;
        this.$http.patch("/emp",ids)
            .then((result) => {
              this.listLoading = false;
              result = result.data;
              if(result.success){
                this.$message({
                  message: '删除成功',
                  type: 'success'
                });
              }else{
                this.$message({
                  message:result.message,
                  type: 'error'
                });
              }
              //跳转到第一页
              this.handleCurrentChange(1);
            });
      }).catch(() => {

      });
    },

    getDeptTree(){
      this.$http.get("/dept/tree")
          .then(result => {
            /*console.log(result);*/
            this.deptTree = result.data;
          })
    }
  },
  mounted() {
    this.getEmployees();
  }
}

</script>

<style scoped>

</style>
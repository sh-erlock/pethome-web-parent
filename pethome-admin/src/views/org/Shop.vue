<template>
  <section>
    <!--工具条-->
    <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
      <el-form :inline="true" :model="query">
        <el-form-item>
          <el-input v-model="query.keyword" placeholder="关键字"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" v-on:click="search">查询</el-button>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="handleAdd">新增</el-button>
        </el-form-item>
      </el-form>
    </el-col>
    <!--列表-->
    <el-table align="center" :data="pageData.data" highlight-current-row v-loading="listLoading" @selection-change="selsChange" style="width: 100%;">
      <el-table-column type="selection" width="55">
      </el-table-column>
      <el-table-column type="index" label="序号" width="60" align="center">
      </el-table-column>
      <el-table-column prop="name" label="店铺名" width="100" sortable align="center">
      </el-table-column>
      <el-table-column prop="tel" label="号码" width="90" sortable align="center">
      </el-table-column>
      <el-table-column prop="registerTime" label="创建时间" width="120" sortable align="center">
      </el-table-column>
      <el-table-column prop="address" label="地址" width="100" sortable align="center">
      </el-table-column>
      <el-table-column prop="logo" label="图标" width="90" sortable align="center">
        <template scope="scope">
          <img :src="'http://47.95.117.210:8888' + scope.row.logo" width="30" alt="">
        </template>
      </el-table-column>
      <el-table-column prop="state" label="状态" width="100" sortable align="center">
        <template scope="scope">
          <span style="color: blue" v-if="scope.row.state===0">待审核</span>
          <span style="color: green" v-else-if="scope.row.state===1">审核通过</span>
          <span style="color: red" v-else>驳回</span>
        </template>
      </el-table-column>
      <el-table-column prop="admin.username" label="店铺管理员" min-width="90" sortable align="center">
      </el-table-column>
      <el-table-column label="操作" width="250" align="center">
        <template scope="scope">
          <el-button size="small" @click="handleAudit(scope.$index, scope.row)">审核</el-button>
          <el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
          <el-button type="danger" size="small" @click="handleDel(scope.$index, scope.row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!--批量删除/分页 工具条-->
    <el-col :span="24" class="toolbar">
      <el-button type="danger" @click="batchRemove" :disabled="this.sels.length===0">批量删除</el-button>
      <el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :current-page="query.currentPage" :page-size="query.pageSize" :total="pageData.total" style="float:right;">
      </el-pagination>
    </el-col>

    <!--新增/编辑界面-->
    <el-dialog :title="title" :visible.sync="shopFormVisible" :close-on-click-modal="false">
      <el-form :model="shop" label-width="80px" :rules="shopRules" ref="shopForm">
        <el-form-item label="姓名" prop="username">
          <el-input v-model="shop.username" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="年龄" prop="age">
          <el-input v-model="shop.age" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="号码" prop="phone">
          <el-input v-model="shop.phone" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="shop.email" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="状态">
          <el-radio-group v-model="shop.state">
            <el-radio class="radio" :label="0">正常</el-radio>
            <el-radio class="radio" :label="-1">禁用</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="店铺管理员">
          <el-select v-model="shop.admin" value-key="id" placeholder="请选择">
            <el-option v-for="department in departments" :key="department.id" :label="department.name" :value="department">
              <span style="float: left">{{ department.name }}</span>
              <span style="float: right; color: #8492a6; font-size: 13px">{{ department.sn }}</span>
            </el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click.native="shopFormVisible = false">取消</el-button>
        <el-button type="primary" @click.native="saveSubmit" :loading="shopLoading">保存</el-button>
      </div>
    </el-dialog>

    <!--审核界面-->
    <el-dialog :title="title" :visible.sync="shopAuditFormVisible" :close-on-click-modal="false">
      <el-form :model="shopAuditLog" label-width="80px" :rules="shopRules" ref="shopAuditForm">
        <el-form-item label="店铺名" prop="name">
          <el-input v-model="shopAuditLog.shop.name" auto-complete="off" disabled="disabled"></el-input>
        </el-form-item>
        <el-form-item label="审核备注" prop="node">
          <el-input v-model="shopAuditLog.note" auto-complete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click.native="shopAuditFormVisible = false">取消</el-button>
        <el-button type="danger" @click.native="reject" :loading="shopLoading">驳回</el-button>
        <el-button type="success" @click.native="pass" :loading="shopLoading">通过</el-button>
      </div>
    </el-dialog>

  </section>
</template>

<script>
export default {
  data() {
    return {
      //审核界面数据
      shopAuditLog: {
        shop: {},
        note:''
      },
      //新增/修改弹出框标题
      title: '',
      //分页查询返回的数据
      pageData:{
        total:0,//分页条
        data:[]//数据表格
      },
      //分页查询+高级查询传递到后台的参数
      query:{
        keyword:'',
        currentPage:1,//分页条
        pageSize:5 //分页条
      },
      //加载动画
      listLoading: false,
      sels: [],//列表选中列

      shopFormVisible: false,//编辑界面是否显示
      shopAuditFormVisible: false,//审核界面是否显示
      shopLoading: false,
      //校验规则
      shopRules: {
        username: [
          { required: true, message: '请输入姓名', trigger: 'blur' }
        ],
        age: [
          { required: true, message: '请输入标识', trigger: 'blur' }
        ]
      },
      //新增界面数据
      shop: {
        id: null,
        name: '',
        tel:'',
        registerTime: '',
        address: '',
        state: 0,
        logo: 0,
        admin: {}
      },
      //所有部门
      departments:[],
    }
  },
  methods: {
    getDepartments(){
      this.$http.get("/department").then(res=>{
        this.departments = res.data;
      })
    },
    handleCurrentChange(val) {
      this.query.currentPage = val;
      this.getShops();
    },
    //高级查询按钮触发的方法
    search(){
      //修改页码
      this.query.currentPage = 1;
      //改查询条件=双向绑定输入时自动改变，只需要重新查询一次就ok
      this.getShops();
    },
    //获取用户列表
    getShops() {
      this.listLoading = true;
      this.$http.post("/shop", this.query)
          .then(result => {
            this.pageData.data = result.data.list;
            this.pageData.total = result.data.totals;
            this.listLoading = false;
          })
          .catch(result=>{
            alert("Failure！");
          })
    },
    //删除
    handleDel: function (index, row) {
      this.$confirm('确认删除该记录吗?', '温馨提示', {
        type: 'warning'
      }).then(() => {
        this.listLoading = true;
        let url = "/shop/"+row.id;
        console.log(url)
        this.$http.delete(url)
            .then((res) => {
              this.listLoading = false;
              res = res.data;
              if(res.success){
                this.$message({
                  message: '删除成功',
                  type: 'success'
                });
              }else {
                this.$message({
                  message: res.msg,
                  type: 'error'
                });
              }
              this.handleCurrentChange(1);
            }).catch(() => {
          alert("报错了")
        });
      }).catch(() => {

      });
    },
    //显示编辑界面
    handleEdit: function (index, row) {
      this.shopFormVisible = true;
      this.title = "编辑员工信息";
      this.getDepartments();
      this.shop = Object.assign({}, row);
    },
    //显示审核界面
    handleAudit: function (index, row) {
      this.shopAuditFormVisible = true;
      this.title = "审核信息";
      this.shopAuditLog.shop = Object.assign({}, row);
      //this.shopAuditLog.node = '';
    },
    //显示新增界面
    handleAdd: function () {
      this.shopFormVisible = true;
      this.title = "新增员工";
      this.getDepartments();
      // this.getDeptTree();
      //this.$refs['shopForm'].resetFields();
      this.shop = {
        id: null,
        name: '',
        tel:'',
        registerTime: '',
        address: '',
        state: 0,
        logo: 0,
        admin: {}
      };
    },
    //新增/编辑==》保存功能
    saveSubmit: function () {
      this.$refs.shopForm.validate((valid) => {
        if (valid) {
          this.$confirm('确认提交吗？', '提示', {}).then(() => {
            this.shopLoading = true;
            let para = Object.assign({}, this.shop);
            this.$http.put("/shop",para)
                .then((res) => {
                  this.shopLoading = false;
                  res = res.data;
                  if(res.success){
                    this.$message({
                      message: '保存成功',
                      type: 'success'
                    });
                    this.$refs['shopForm'].resetFields();//重置表单
                    this.shopFormVisible = false;
                    this.getShops();
                  }else{
                    this.$message({
                      message: res.msg,
                      type: 'error'
                    });
                  }
                });
          });
        }
      });
    },
    //驳回
    reject: function () {
      this.$refs.shopAuditForm.validate((valid) => {
        if (valid) {
          this.$confirm('确认提交吗？', '提示', {}).then(() => {
            this.shopLoading = true;
            this.$http.post("/shop/reject",this.shopAuditLog)
                .then((res) => {
                  this.shopLoading = false;
                  res = res.data;
                  if(res.success){
                    this.$message({
                      message: '操作成功',
                      type: 'success'
                    });
                    this.shopAuditFormVisible = false;
                    this.getShops();
                  }else{
                    this.$message({
                      message: res.msg,
                      type: 'error'
                    });
                  }
                });
          });
        }
      });
    },
    //通过
    pass: function () {
      this.$refs.shopAuditForm.validate((valid) => {
        if (valid) {
          this.$confirm('确认提交吗？', '提示', {}).then(() => {
            this.shopLoading = true;
            this.$http.post("/shop/pass",this.shopAuditLog)
                .then((res) => {
                  this.shopLoading = false;
                  res = res.data;
                  if(res.success){
                    this.$message({
                      message: '操作成功',
                      type: 'success'
                    });
                    this.shopAuditFormVisible = false;
                    this.getShops();
                  }else{
                    this.$message({
                      message: res.msg,
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
    },
    //批量删除
    batchRemove: function () {
      var ids = this.sels.map(item => item.id);
      this.$confirm('确认删除选中记录吗？', '提示', {
        type: 'warning'
      }).then(() => {
        this.listLoading = true;
        this.$http.patch("/shop",ids)
            .then((res) => {
              this.listLoading = false;
              res = res.data;
              if(res.success){
                this.$message({
                  message: '删除成功',
                  type: 'success'
                });
              }else {
                this.$message({
                  message: res.msg,
                  type: 'error'
                });
              }
              this.handleCurrentChange(1);
            }).catch(() => {
          alert("报错了")
        });
      }).catch(() => {

      });
    }
  },
  mounted() {
    this.getShops();
  }
}

</script>

<style scoped>

</style>
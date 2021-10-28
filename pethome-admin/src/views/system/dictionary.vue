<template>
  <section>
    <!--查询新增工具条-->
    <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
      <el-form :inline="true" :model="filters">
        <el-form-item>
          <el-input v-model="filters.keyword" placeholder="关键字"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" v-on:click="search">查询</el-button>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="handleAdd">新增</el-button>
        </el-form-item>
      </el-form>
    </el-col>
    <!--字典类型展示-->
    <el-table :data=" dictionaryType" @cell-dblclick="showDetails" highlight-current-row v-loading="listLoading" @selection-change="selsChange" style="width: 100%;">
      <el-table-column type="selection" width="55">
      </el-table-column>
      <el-table-column type="index" width="60">
      </el-table-column>
      <el-table-column prop="sn" label="编号" width="150" sortable>
      </el-table-column>
      <el-table-column prop="name" label="字典名" width="150" sortable>
      </el-table-column>
      <el-table-column label="操作">
        <template scope="scope">
          <el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
          <el-button type="danger" size="small" @click="handleDel(scope.$index, scope.row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!--分页工具条-->
    <el-col :span="24" class="toolbar">
      <el-button type="danger" @click="batchRemove" :disabled="this.sels.length===0">批量删除</el-button>
      <el-pagination layout="prev, pager, next"
                     @current-change="handleCurrentChange"
                     :current-page="currentPage"
                     :page-size="pageSize"
                     :total="total"
                     style="float:right;">
      </el-pagination>
    </el-col>
    <!--字典类型新增页面-->
    <el-dialog :title="title" :visible.sync="dictionaryTypeFormVisible" :close-on-click-modal="false">
      <el-form :model="dictionaryTypeForm" label-width="80px" :rules="dictionaryTypeFormRules" ref="dictionaryTypeForm">
        <el-form-item label="编号" prop="sn">
          <el-input v-model="dictionaryTypeForm.sn" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="字典名" prop="name">
          <el-input v-model="dictionaryTypeForm.name" auto-complete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click.native="dictionaryTypeFormVisible = false">取消</el-button>
        <el-button type="primary" @click.native="departmentSubmit" :loading="dictionaryTypeLoading">提交</el-button>
      </div>
    </el-dialog>
    <!--字典明细展示抽屉-->
    <el-drawer title="数据字典明细" :visible.sync="drawer">
      <!--工具条-->
      <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
        <el-form :inline="true" :model="filters">
          <el-form-item>
            <el-input v-model="filters.keyword" placeholder="关键字"></el-input>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" v-on:click="">查询</el-button>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="detailsAdd">新增</el-button>
          </el-form-item>
        </el-form>
      </el-col>
      <!--列表-->
      <el-table :data="details"  highlight-current-row v-loading="listLoading" @selection-change="selsChange" style="width: 100%;">
        <el-table-column type="selection" width="55">
        </el-table-column>
        <el-table-column prop="id" label="编号" width="120" sortable>
        </el-table-column>
        <el-table-column prop="name" label="明细名" width="120" sortable>
        </el-table-column>
        <el-table-column label="操作">
          <template scope="scope">
            <el-button size="small" @click="detailsEdit(scope.$index, scope.row)">编辑</el-button>
            <el-button type="danger" size="small" @click="detailsDel(scope.$index, scope.row)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>

      <!--工具条-->
      <el-col :span="24" class="toolbar">
        <el-button type="danger" @click="batchRemove" :disabled="this.sels.length===0">批量删除</el-button>
        <el-pagination layout="prev, pager, next"
                       @current-change="handleCurrentChange"
                       :current-page="currentPage"
                       :page-size="pageSize"
                       :total="total"
                       style="float:right;">
        </el-pagination>
      </el-col>
    </el-drawer>
    <!--字典明细新增/编辑页面-->
    <el-dialog :title="title" :visible.sync="detailsFormVisible" :close-on-click-modal="false">
      <el-form :model="detailsForm" label-width="80px" :rules="detailsFormRules" ref="detailsForm">
        <el-form-item label="明细名" prop="name">
          <el-input v-model="detailsForm.name" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="类型" prop="type">
          <el-input v-model="detailsForm.type" auto-complete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click.native="dictionaryTypeFormVisible = false">取消</el-button>
        <el-button type="primary" @click.native="detailSubmit" :loading="dictionaryTypeLoading">提交</el-button>
      </div>
    </el-dialog>
  </section>
</template>

<script>
export default {
  data() {
    return {
      title:"",
      filters: {
        keyword: ''
      },
      dictionaryType:[],//字典类型展示
      total: 0,
      currentPage:1,
      pageSize: 5,
      listLoading: false,
      sels: [],//列表选中行
      details:[],
      dictionaryTypeFormVisible: false,//新增界面是否显示
      detailsFormVisible: false,//字典明细新增页面是否显示
      dictionaryTypeLoading: false,
      drawer:false,//默认关闭抽屉
      dictionaryTypeFormRules: {
        name: [
          { required: true, message: '请输入字典名', trigger: 'blur' }
        ],
        sn: [
          { required: true, message: '请输入编码', trigger: 'blur' }
        ]
      },
      detailsFormRules: {
        name: [
          { required: true, message: '请输入字典名', trigger: 'blur' }
        ],
        sn: [
          { required: true, message: '请输入编码', trigger: 'blur' }
        ]
      },
      //字典类型新增界面数据
      dictionaryTypeForm: {
        id:null,
        sn:'',
        name:''
      },
      //字典明细新增界面数据
      detailsForm: {
        id:null,
        name:'',
        type:'',
      }

    }
  },
  methods: {
    //显示字典明细
    showDetails(row){//把当前行的id传递到后端后端通过id查找到相应的明细列表展示到前端
      this.drawer=true;//展开抽屉
      this.$http.post("/detail/"+row.id)
          .then(result=>{
            this.details = result.data;
          })
    },
    handleCurrentChange(val) {
      this.currentPage = val;
      this.getDictionary();
    },
    search(){
      this.currentPage=1;
      this.getDictionary();
    },


    getDictionary() {
      let para = {//请求参数
        currentPage: this.currentPage,
        pageSize: this.pageSize,
        keyword: this.filters.keyword
      };
      this.listLoading = true;//开启忙等框
      //这里发送axios请求从后端拿数据
      this.$http.post("/type",para)
          .then(result=>{
            this.listLoading = false;//关闭忙等框
            result = result.data;//PageList
            console.log(result);
            this.dictionaryType = result.list;
            this.total = result.totals;

          })
          .catch(result=>{
            this.$message({
              message: '操作失败！',
              type: 'error'
            });
          })

    },
    //获取明细列表
    /*getDetails(){
      this.$http.get("/dicts")
          .then(result=>{
            this.details=result.data;
          })
          .catch(result=>{
            this.$message({
              message: '操作失败！',
              type: 'error'
            });
          })
    },
    */
    //分页查询
    getDetails() {
      let para = {//请求参数
        currentPage: this.currentPage,
        pageSize: this.pageSize,
        keyword: this.filters.keyword
      };
      this.listLoading = true;//开启忙等框
      //这里发送axios请求从后端拿数据
      this.$http.post("/detail",para)
          .then(result=>{
            this.listLoading = false;//关闭忙等框
            result = result.data;//PageList
            console.log(result);
            this.details = result.list;
            this.total = result.totals;

          })
          .catch(result=>{
            this.$message({
              message: '操作失败！',
              type: 'error'
            });
          })

    },
    //删除
    handleDel: function (index, row) {
      this.$confirm('确认删除该记录吗?', '温馨提示', {
        type: 'warning'
      }).then(() => {//点击确定按钮
        this.listLoading = true;//开启忙等框
        this.$http.delete("/type/"+ row.id)
            .then(result=>{
              //3.关闭忙等框
              this.listLoading = false;//关闭忙等框
              result = result.data;//AjaxResult
              //1.要提示后端业务执行到底是：成功还是失败
              if(result.success){
                this.$message({
                  message: '删除成功！',
                  type: 'success'
                });
              }else{
                this.$message({
                  message: result.message,
                  type: 'error'
                });
              }
              //4.定位到第一页 要？ 如果不要会不会有问题？
              this.currentPage = 1;
              //2.刷新页面
              this.getDictionary();

            })
            .catch(result=>{
              console.log(result);
              this.$message({
                message: '操作失败',
                type: 'error'
              });
            });
      }).catch(() => {//点击取消

      });
    },
    //明细删除
    detailsDel(index,row){
      this.$confirm('确认删除该记录吗?', '温馨提示', {
        type: 'warning'
      }).then(() => {//点击确定按钮
        this.listLoading = true;//开启忙等框
        this.$http.delete("/detail/"+ row.id)
            .then(result=>{
              //3.关闭忙等框
              this.listLoading = false;//关闭忙等框
              result = result.data;//AjaxResult
              //1.要提示后端业务执行到底是：成功还是失败
              if(result.success){
                this.$message({
                  message: '删除成功！',
                  type: 'success'
                });
              }else{
                this.$message({
                  message: result.message,
                  type: 'error'
                });
              }
              //4.定位到第一页 要？ 如果不要会不会有问题？
              this.currentPage = 1;
              //2.刷新页面
              this.getDetails();

            })
            .catch(result=>{
              console.log(result);
              this.$message({
                message: '操作失败',
                type: 'error'
              });
            });
      }).catch(() => {//点击取消

      });
    },
    //显示编辑界面
    handleEdit: function (index, row) {
      //获取部门树
      this.title = "编辑";
      this.dictionaryTypeFormVisible = true;//控制新增或者编辑的模态框的显示
      // Object.assign 克隆
      this.dictionaryTypeForm = Object.assign({}, row);

    },
    //显示明细编辑界面
    detailsEdit: function (index, row) {
      this.getDetails();
      this.title = "编辑";
      this.detailsFormVisible = true;//控制新增或者编辑的模态框的显示
      // Object.assign 克隆
      this.detailsForm = Object.assign({}, row);

    },
    //显示字典类型新增界面
    handleAdd: function () {
      this.title = "新增";
      this.dictionaryTypeFormVisible = true;//控制新增或者编辑的模态框的显示
      this.dictionaryTypeForm = {//清空历史遗留数据
        id:null,
        sn:'',
        name:''
      };
    },
    //显示字典明细新增界面
    detailsAdd: function () {
      this.title = "新增";
      this.detailsFormVisible = true;//控制新增或者编辑的模态框的显示
      this.detailsForm = {//清空历史遗留数据
        id:null,
        name:'',
        type:'',
      };
      console.log(this.detailsForm.type_id);

    },

    //新增/编辑
    departmentSubmit: function () {
      this.$refs.dictionaryTypeForm.validate((valid) => {
        if (valid) {//表单校验成功
          this.$confirm('确认提交吗？', '提示', {}).then(() => {
            this.dictionaryTypeLoading = true;//开启提交按钮忙等框
            let param = Object.assign({}, this.dictionaryTypeForm);
            //发送请求
            this.$http.put("/type",param)
                .then(result=>{
                  result = result.data;
                  this.dictionaryTypeLoading = false;//关闭提交按钮忙等框
                  if(result.success){
                    this.dictionaryTypeFormVisible = false;
                    //刷新页面
                    this.getDictionary();
                    this.$message({
                      message: '操作成功！！！',
                      type: 'success'
                    });
                  }else {
                    this.$message({
                      message:result.msg,
                      type: 'error'
                    });
                  }
                }).catch(result=>{
              console.log(result)
              alert("出错了！！！")
            })
          });
        }
      });
    },
    //明细新增/编辑
    detailSubmit: function () {
      this.$refs.detailsForm.validate((valid) => {
        if (valid) {//表单校验成功
          this.$confirm('确认提交吗？', '提示', {}).then(() => {
            this.dictionaryTypeLoading = true;//开启提交按钮忙等框

            let param = Object.assign({}, this.detailsForm)
            //发送请求
            this.$http.put("/detail",param)
                .then(result=>{
                  result = result.data;
                  this.dictionaryTypeLoading = false;//关闭提交按钮忙等框
                  if(result.success){
                    this.  detailsFormVisible = false;
                    //刷新页面
                    this.getDictionary();
                    this.$message({
                      message: '操作成功！！！',
                      type: 'success'
                    });
                  }else {
                    this.$message({
                      message:result.msg,
                      type: 'error'
                    });
                  }
                }).catch(result=>{
              console.log(result)
              alert("出错了！！！")
            })
          });
        }
      });
    },
    selsChange: function (sels) {//每点击一次复选框，会把当前页面上所选中的所有行的数据封装到sels
      this.sels = sels;
    },
    //批量删除
    batchRemove: function () {
      var ids = this.sels.map(item => item.id);//[1,2,3]
      this.$confirm('确认删除选中记录吗？', '温馨提示', {
        type: 'warning'
      }).then(() => {
        this.listLoading = true;
        this.$http.patch("/detail",ids)
            .then(result=>{
              //3.关闭忙等框
              this.listLoading = false;//关闭忙等框
              result = result.data;//AjaxResult
              //1.要提示后端业务执行到底是：成功还是失败
              if(result.success){
                this.$message({
                  message: '删除成功！',
                  type: 'success'
                });
              }else{
                this.$message({
                  message: result.msg,
                  type: 'error'
                });
              }
              //4.定位到第一页 要？ 如果不要会不会有问题？
              this.currentPage = 1;
              //2.刷新页面
              this.getDictionary();
            })
            .catch(result=>{
              console.log(result);
              this.$message({
                message: '操作失败',
                type: 'error'
              });
            });
      }).catch(() => {

      });
    }
  },
  mounted() {
    //获取员工列表
    this.getDictionary();

  }
}

</script>

<style scoped>

</style>
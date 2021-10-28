<template>
	<section>
		<!--工具条-->
		<el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
			<el-form :inline="true" :model="filters">
				<el-form-item>
					<el-input v-model="filters.keyword" placeholder="输入部门编号或名称"></el-input>
				</el-form-item>
				<el-form-item>
					<el-button type="warning" v-on:click="getDepartments">查询</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="success" @click="handleAdd">新增</el-button>
				</el-form-item>
			</el-form>
		</el-col>

		<!--列表-->
		<el-table :data="departments" highlight-current-row v-loading="listLoading" @selection-change="selsChange" style="width: 100%;">
			<el-table-column type="selection" width="55">
			</el-table-column>
			<el-table-column type="index" label="序号" width="60">
			</el-table-column>
			<el-table-column align="center" prop="sn" label="部门编号" width="120" sortable>
			</el-table-column>
      <el-table-column align="center" prop="name" label="部门名称" width="120" sortable>
			</el-table-column>
			<el-table-column align="center" prop="state" label="状态" width="100" sortable>

        <template slot-scope="scope" align="center">
          <span style="color:green" v-if="scope.row.state===0">启用</span>
          <span style="color:red" v-else-if="scope.row.state===-1">禁用</span>
          <span style="color:yellow" v-else>未知</span>
        </template>

			</el-table-column>
			<el-table-column align="center" prop="manager.username" label="部门经理" width="100" sortable>
			</el-table-column>
			<el-table-column align="center" prop="parent.name" label="上级部门" width="120" sortable>
			</el-table-column>

			<el-table-column label="操作" >
				<template scope="scope">
					<el-button size="small" type="primary" icon="el-icon-edit" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
					<el-button type="danger" size="small" icon="el-icon-delete" @click="handleDel(scope.$index, scope.row)">删除</el-button>
				</template>
			</el-table-column>
		</el-table>

		<!--工具条-->
		<el-col :span="24" class="toolbar">
			<el-button type="danger" @click="batchRemove" :disabled="this.sels.length===0">批量删除</el-button>
			<el-pagination layout="prev, pager, next"
                     @current-change="handleCurrentChange"
                     :page-size="pageSize"
                     :total="total"
                     style="float:right;">
			</el-pagination>
		</el-col>

		<!--编辑和添加模态框界面-->
		<el-dialog :title="title" :visible.sync="editFormVisible" :close-on-click-modal="false">
			<el-form :model="editForm" label-width="80px" :rules="editFormRules" ref="editForm">

				<el-form-item label="部门编号" prop="sn">
					<el-input v-model="editForm.sn" auto-complete="off"></el-input>
				</el-form-item>

        <el-form-item label="部门名称" prop="name">
					<el-input v-model="editForm.name" auto-complete="off"></el-input>
				</el-form-item>

				<el-form-item label="状态">
					<el-radio-group v-model="editForm.state">
						<el-radio class="radio" :label="-1">禁用</el-radio>
						<el-radio class="radio" :label="0">启用</el-radio>
					</el-radio-group>
				</el-form-item>

        <el-form-item label="部门经理" prop="manager">
          <!--<el-input v-model="editForm.manager" auto-complete="off"></el-input>-->
          <el-select v-model="editForm.manager" clearable value-key="id" placeholder="--请选择--">
            <!--
                label: 展示的数据
                value: 传到后台的数据
                key: 唯一的标识
             -->
            <el-option
                v-for="emp in employees"
                :key="emp.id"
                :label="emp.username"
                :value="emp">
              <span style="float: left">{{ emp.username }}</span>
              <span style="float: right; color: #8492a6; font-size: 13px">{{ emp.phone }}</span>
            </el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="上级部门" prop="parent">
          <!--<el-input v-model="editForm.parent" auto-complete="off"></el-input>-->
          <!--
                              label: 展示的数据
                              value: 传到后台的数据
                           -->
          <el-cascader v-model="editForm.parent"
                       :options="tree"
                       :props="{
                          checkStrictly: true,
                          label: 'name',
                          value: 'id'
                        }"
                       clearable>
          </el-cascader>
        </el-form-item>

			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button type="danger" @click.native="editFormVisible = false">取消</el-button>
				<el-button type="success" @click.native="editSubmit" :loading="editLoading">提交</el-button>
			</div>
		</el-dialog>

	</section>
</template>

<script>
	import util from '../../common/js/util'
	//import NProgress from 'nprogress'
	import { getDepartmentListPage, removeDepartment, batchRemoveDepartment, editDepartment, addDepartment } from '../../api/api';

	export default {
		data() {
			return {
        tree:[],
				filters: {
					keyword: ''
				},
        title: '',
        employees: [],
				departments: [],
        pageSize: 5,
				total: 0,
        currentPage: 1,
				listLoading: false,
				sels: [],//列表选中列

				editFormVisible: false,//编辑界面是否显示
				editLoading: false,
				editFormRules: {
					name: [
						{ required: true, message: '--请输入部门名称--', trigger: 'blur' }
					],
          sn: [
						{ required: true, message: '--请输入部门编号--', trigger: 'blur' }
					]
				},


				//编辑界面数据
				editForm: {
					id: null,
          sn: '',
					name: '',
					state: 0,
					manager: null,
					parent: null
				},
			}
		},

		methods: {
			//状态显示转换
			/*formatState: function (row, column) {
				return row.state == 1 ? '启用' : row.state == 0 ? '禁用' : '未知';
			},*/
			handleCurrentChange(val) {
				this.currentPage = val;
				this.getDepartments();

			},
			//获取用户列表
			getDepartments() {
				let para = {
          pageSize: this.pageSize,
					currentPage: this.currentPage,
					keyword: this.filters.keyword
				};
				this.listLoading = true; // 开启忙等框
				//NProgress.start();

        // 发送请求
        this.$http.post("/dept",para).then(result => {
          /*console.log(result);*/
          this.departments = result.data.list;
          this.total = result.data.totals;
          this.listLoading = false; // 关闭忙等框
        }).catch(result => {
          alert("Error, Please try again later!!!!")
        })
      },

			//删除
			handleDel: function (index, row) {
				this.$confirm('Are you sure about this???', 'Warning!!!', {
					type: 'warning'
				}).then(() => {
          // 开启忙等框
					this.listLoading = true;

          // 发送删除请求
          this.$http.delete("/dept/" + row.id)  // 走localhost:8080/department/1
              .then(result => {
                // 关闭忙等框
                console.debug(result);
                this.listLoading = false;
                if (result.data.success) { // 删除成功，提示信息
                    this.$message({
                      message: '删除成功！！！',
                      type: 'success'
                    });
                } else { // 删除失败，提示信息
                    this.$message({
                      message: result.data.message,
                      type: 'error'
                    });
                }
                // 删除之后，数据跳回第一页
                this.currentPage = 1;
                // 重新显示数据
                this.getDepartments();

              }).catch(result => { // 发送请求失败的回调函数
                  this.$message({
                    message: "System Failure, Please try again later!!!",
                    type: 'error'
                 });

          })

				}).catch(() => {

				});
			},


			//显示编辑界面
			handleEdit: function (index, row) {
        this.queryTree(); // 显示所有部门
        this.title = '编辑';
				this.editFormVisible = true;
				this.editForm = Object.assign({}, row);  // 回显数据
        if (this.editForm.dirPath) {
            let dirPath = this.editForm.dirPath;
            let arr = dirPath.split("/");
            let parr = [];
            for (let i = 1; i < arr.length - 1; i++) {
              parr.push(parseInt(arr[i]));
            }
            this.editForm.parent = parr;
        }
			},


			//显示新增界面
			handleAdd: function () {
        this.queryTree(); // 显示所有部门
        this.title = '添加';
				this.editFormVisible = true;
				this.editForm = {
          id: null,
          sn: '',
          name: '',
          state: 0,
          manager: null,
          parent: null
				};
			},


			// 编辑
			editSubmit: function () {
				this.$refs.editForm.validate((valid) => {
					if (valid) {
						this.$confirm('确认提交吗？', '提示', {}).then(() => {
							this.editLoading = true;
							// 克隆数据
							let para = Object.assign({}, this.editForm);
              // 处理传输的数据格式
              // 向后台传输数据时para.parent是被作为数组传输的，而后端是用对象来接收，所以要转换为对象
              if (para.parent) {
                let par = para.parent; // 获得parent数组
                let id = par[par.length - 1]; // 获得数组的最后一个元素，因为最后一个值id才是上级部门的id
                para.parent = {"id" : id}; // 转换为对象，方便向后台传值
              }

              this.$http.put("/dept", para)
                  .then(result => {
                    this.editLoading = false; // 关闭忙等框
                    if (result.data.success) {
                      this.$message({
                        message: '提交成功',
                        type: 'success'
                      });
                    } else {
                      this.$message({
                        message: result.data.message,
                        type: 'error'
                      });
                    }
                    this.editFormVisible = false; // 编辑界面是否显示
                    this.getDepartments();

                  }).catch(result => {
                      this.$message({
                        message: "系统故障，正在殴打程序员",
                        type: 'error'
                      });

              })

						});
					}
				});
			},

      selsChange: function (sels) {
        this.sels = sels;
      },

			//批量删除
			batchRemove: function () {
				/*var ids = this.sels.map(item => item.id).toString();*/
        var ids = this.sels.map(item => item.id)
				this.$confirm('确认删除选中记录吗？', '亲情提示', {
					type: 'warning'
				}).then(() => {
					this.listLoading = true;
          this.$http.patch("/dept", ids)
              .then(result => {
                console.log(result);
                this.listLoading = false; // 关闭忙等框
                if (result.data.success) { // 删除成功，提示信息
                  this.$message({
                      message: '删除成功',
                      type: 'success'
                  });
                } else {
                    this.$message({
                      message: result.data.message,
                      type: 'error'
                    });
                }
                this.currentPage=1;
                this.getDepartments();
              }).catch(result => {
                    this.$message({
                      message: 'System Failure ,Please try again later!!!',
                      type: 'error'
                    });
                  this.listLoading = false; // 关闭忙等框
          })
				}).catch(() => {

				});
			},

      // 查询所有员工，为了在修改的时候不能随意输入，而是要选择数据库中存在的数据
      getEmployees(){
        this.$http.get("/emp")
            .then(result => {
              this.employees = result.data;
            }).catch(() => {
        })
      },


      // 查询部门树，为了在修改或添加的时候不能随意输入，而是要选择数据库中存在的数据
      queryTree(){
        this.$http.get("/dept/tree")
            .then(result => {
              this.tree = result.data;
            }).catch(() => {
        })
      }
		},

		mounted() {
			this.getDepartments();
			this.getEmployees();
		}
	}

</script>

<style scoped>

</style>
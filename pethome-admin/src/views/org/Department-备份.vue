<template>
	<section>
		<!--工具条-->
		<el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
			<el-form :inline="true" :model="filters">
				<el-form-item>
					<el-input v-model="filters.keyword" placeholder="输入部门编号或名称"></el-input>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" v-on:click="getDepartments">查询</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="handleAdd">新增</el-button>
				</el-form-item>
			</el-form>
		</el-col>

		<!--列表-->
		<el-table :data="departments" highlight-current-row v-loading="listLoading" @selection-change="selsChange" style="width: 100%;">
			<el-table-column type="selection" width="55">
			</el-table-column>
			<el-table-column type="index" width="60">
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

		<!--编辑界面-->
		<el-dialog title="编辑" :visible.sync="editFormVisible" :close-on-click-modal="false">
			<el-form :model="editForm" label-width="80px" :rules="editFormRules" ref="editForm">
				<el-form-item label="姓名" prop="name">
					<el-input v-model="editForm.name" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="性别">
					<el-radio-group v-model="editForm.sex">
						<el-radio class="radio" :label="1">男</el-radio>
						<el-radio class="radio" :label="0">女</el-radio>
					</el-radio-group>
				</el-form-item>
				<el-form-item label="年龄">
					<el-input-number v-model="editForm.age" :min="0" :max="200"></el-input-number>
				</el-form-item>
				<el-form-item label="生日">
					<el-date-picker type="date" placeholder="选择日期" v-model="editForm.birth"></el-date-picker>
				</el-form-item>
				<el-form-item label="地址">
					<el-input type="textarea" v-model="editForm.addr"></el-input>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click.native="editFormVisible = false">取消</el-button>
				<el-button type="primary" @click.native="editSubmit" :loading="editLoading">提交</el-button>
			</div>
		</el-dialog>

		<!--新增界面-->
		<el-dialog title="新增" :visible.sync="addFormVisible" :close-on-click-modal="false">
			<el-form :model="addForm" label-width="80px" :rules="addFormRules" ref="addForm">
				<el-form-item label="姓名" prop="name">
					<el-input v-model="addForm.name" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="性别">
					<el-radio-group v-model="addForm.sex">
						<el-radio class="radio" :label="1">男</el-radio>
						<el-radio class="radio" :label="0">女</el-radio>
					</el-radio-group>
				</el-form-item>
				<el-form-item label="年龄">
					<el-input-number v-model="addForm.age" :min="0" :max="200"></el-input-number>
				</el-form-item>
				<el-form-item label="生日">
					<el-date-picker type="date" placeholder="选择日期" v-model="addForm.birth"></el-date-picker>
				</el-form-item>
				<el-form-item label="地址">
					<el-input type="textarea" v-model="addForm.addr"></el-input>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click.native="addFormVisible = false">取消</el-button>
				<el-button type="primary" @click.native="addSubmit" :loading="addLoading">提交</el-button>
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
				filters: {
					keyword: ''
				},
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
						{ required: true, message: '请输入姓名', trigger: 'blur' }
					]
				},
				//编辑界面数据
				editForm: {
					id: 0,
					name: '',
					sex: -1,
					age: 0,
					birth: '',
					addr: ''
				},

				addFormVisible: false,//新增界面是否显示
				addLoading: false,
				addFormRules: {
					name: [
						{ required: true, message: '请输入姓名', trigger: 'blur' }
					]
				},
				//新增界面数据
				addForm: {
					name: '',
					sex: -1,
					age: 0,
					birth: '',
					addr: ''
				}

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
				/*getDepartmentListPage(para).then((res) => {
					this.total = res.data.total;
					this.departments = res.data.departments;
					this.listLoading = false;
					//NProgress.done();
				});
			},*/


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



					/*removeDepartment(para).then((res) => {
						this.listLoading = false;
						//NProgress.done();
						this.$message({
							message: '删除成功',
							type: 'success'
						});
						this.getDepartments();
					});*/
				}).catch(() => {

				});
			},
			//显示编辑界面
			handleEdit: function (index, row) {
				this.editFormVisible = true;
				this.editForm = Object.assign({}, row);
			},
			//显示新增界面
			handleAdd: function () {
				this.addFormVisible = true;
				this.addForm = {
					name: '',
					sex: -1,
					age: 0,
					birth: '',
					addr: ''
				};
			},
			//编辑
			editSubmit: function () {
				this.$refs.editForm.validate((valid) => {
					if (valid) {
						this.$confirm('确认提交吗？', '提示', {}).then(() => {
							this.editLoading = true;
							//NProgress.start();
							let para = Object.assign({}, this.editForm);
							para.birth = (!para.birth || para.birth == '') ? '' : util.formatDate.format(new Date(para.birth), 'yyyy-MM-dd');
							editDepartment(para).then((res) => {
								this.editLoading = false;
								//NProgress.done();
								this.$message({
									message: '提交成功',
									type: 'success'
								});
								this.$refs['editForm'].resetFields();
								this.editFormVisible = false;
								this.getDepartments();
							});
						});
					}
				});
			},
			//新增
			addSubmit: function () {
				this.$refs.addForm.validate((valid) => {
					if (valid) {
						this.$confirm('确认提交吗？', '提示', {}).then(() => {
							this.addLoading = true;
							//NProgress.start();
							let para = Object.assign({}, this.addForm);
							para.birth = (!para.birth || para.birth == '') ? '' : util.formatDate.format(new Date(para.birth), 'yyyy-MM-dd');
							addDepartment(para).then((res) => {
								this.addLoading = false;
								//NProgress.done();
								this.$message({
									message: '提交成功',
									type: 'success'
								});
								this.$refs['addForm'].resetFields();
								this.addFormVisible = false;
								this.getDepartments();
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
				var ids = this.sels.map(item => item.id).toString();
				this.$confirm('确认删除选中记录吗？', '提示', {
					type: 'warning'
				}).then(() => {
					this.listLoading = true;
					//NProgress.start();
					let para = { ids: ids };
					batchRemoveDepartment(para).then((res) => {
						this.listLoading = false;
						//NProgress.done();
						this.$message({
							message: '删除成功',
							type: 'success'
						});
						this.getDepartments();
					});
				}).catch(() => {

				});
			}
		},
		mounted() {
			this.getDepartments();
		}
	}

</script>

<style scoped>

</style>
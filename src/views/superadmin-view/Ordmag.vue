<template>
	<section class="container">
		<!--工具条-->
		<el-col :span="24" class="toolbar" style="margin-bottom: 0px;">
			<el-form :inline="true" :model="filters" style="width:100%;" >
				<el-form-item>
					<el-input v-model="filters.search" placeholder="输入关键字查询"></el-input>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" v-on:click="getUsers(1)">查询</el-button>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" @click="handleAdd">新增</el-button>
				</el-form-item>
				<el-form-item style="float:right; margin-right:0px;">
					<el-button  type="primary" @click="handleimport" >Excel批量导入</el-button>
				</el-form-item>
			</el-form>
		</el-col>

		<!--批量导入界面-->	
		<el-dialog title="Excel批量导入" :visible.sync ="fileimportVisible" :close-on-click-modal="false" >
			
				<el-upload  class="upload-demo" :limit=1  ref="upload" method="post" action="http://120.79.12.163/ordfile"  accept=".csv, application/vnd.openxmlformats-officedocument.spreadsheetml.sheet, application/vnd.ms-excel" :on-preview="handlePreview"  :on-remove="handleRemove" :on-success="handleSuccess" :on-error="handleError" :on-progress="handlePro"  :file-list="fileList" :data="uploadform"  :auto-upload="false">
					<el-button slot="trigger" size="small" type="primary">选取文件</el-button>
					<el-button style="margin-left: 10px;" size="small" type="success" @click="submitUpload">上传到服务器</el-button>
					<form method="get" action="http://120.79.12.163/ordmodel" style="position:absolute; margin-top:-32px;right:20px;" >
						<el-button style="float:right" size="small"  type="success" native-type="submit" target="_blank">Excel模板下载</el-button>
					</form>
					<div slot="tip" class="el-upload__tip">只能上传xls/xlsx文件</div>
				</el-upload>
			
			<div slot="footer" class="dialog-footer">
				<el-button @click.native="fileimportVisible = false">取消</el-button>
				<el-button type="primary" @click.native="fileimportVisible = false">完成</el-button>
			</div>
		</el-dialog>

		<!--列表-->
		<el-table :data="users" fit  highlight-current-row :border="border" v-loading="listLoading" @selection-change="selsChange" style="width: 100%;margin-bottom:15px;" class="tablelist">
			<el-table-column fixed="left" type="selection" width="55" align="center" >
			</el-table-column>
			<el-table-column  fixed="left" type="index" :index="tableindex" label="ID" width="60" align="center">
			</el-table-column>
			<el-table-column prop="name" label="姓名" min-width="120" align="center" sortable>
			</el-table-column>
			<el-table-column prop="workid" label="工号" min-width="140" align="center" sortable>
			</el-table-column>
			<el-table-column prop="workplace" label="工作单位" min-width="100" align="center" sortable>
			</el-table-column>
			<el-table-column prop="phone" label="手机号" min-width="120" align="center" sortable>
			</el-table-column>
			<el-table-column label="管理的签到域" min-width="300" max-width="500" align="center" sortable>
				<template slot-scope="scope">
					<el-tag :key="taglist" v-for="taglist in users[scope.$index].signno" >
						{{taglist}}
					</el-tag>
				</template>
			</el-table-column>
			<el-table-column fixed="right" label="操作" width="250" align="center">
				<template slot-scope="scope">
					<el-button class="resetpsd" type="danger" size="small" @click="handleresetpsd(scope.$index, scope.row)">重置密码</el-button>
					<el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
					<el-button type="danger" size="small" @click="handleDel(scope.$index, scope.row)">删除</el-button>
				</template>
			</el-table-column>
		</el-table>

		<!--工具条-->
		<el-col :span="24" class="toolbar">
			<el-button type="danger" @click="batchRemove" :disabled="this.sels.length===0">批量删除</el-button>
			<el-pagination class="pagebtn" :current-page="page" background layout="total, sizes, prev, pager, next, jumper" @size-change="handleSizeChange" @current-change="handleCurrentChange" 
      :page-sizes="[5, 10, 15, 20]" :page-size="10" :total="parseInt(total)"  style="float:right;">
			</el-pagination>
		</el-col>

		<!--编辑界面-->
		<el-dialog title="编辑" :visible.sync ="editFormVisible" :close-on-click-modal="false" >
			<el-form :model="editForm" label-width="80px" :rules="editFormRules" ref="editForm">
				<el-form-item label="姓名" >
					<el-input v-model="editForm.name"  style="width:400px;"></el-input>
				</el-form-item>
				<el-form-item label="工号">
					<el-input v-model="editForm.workid" :disabled='true' style="width:400px;"></el-input>
				</el-form-item>
				<el-form-item label="单位">
					<el-input v-model="editForm.workplace" style="width:400px;"></el-input>
				</el-form-item>
				<el-form-item label="手机号">
					<el-input v-model="editForm.phone" style="width:400px;"></el-input>
				</el-form-item>
				<el-form-item label="签到域" >
					<el-tag :key="tag" v-for="tag in editForm.signno" closable :disable-transitions="false" @close="handleClose(tag)">
						{{tag}}
					</el-tag>
					<el-input class="input-new-tag" v-if="inputVisible" v-model="inputValue" ref="saveTagInput" size="small" @keyup.enter.native="handleInputConfirm" @blur="handleInputConfirm">
					</el-input>
					<el-button v-else class="button-new-tag" size="small" @click="showInput">点击添加新的签到域</el-button>
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
				<el-form-item label="姓名"  style="width:400px;">
					<el-input v-model="addForm.name" auto-complete="off"></el-input>
				</el-form-item>
				<el-form-item label="工号" prop="name" style="width:400px;">
					<el-input  v-model="addForm.workid"></el-input>
				</el-form-item>
				<el-form-item label="单位" style="width:400px;">
					<el-input  v-model="addForm.workplace"></el-input>
				</el-form-item>
				<el-form-item label="手机号" style="width:400px;">
					<el-input  v-model="addForm.phone"></el-input>
				</el-form-item>
				<el-form-item label="签到域" style="width:400px;">
					<el-tag :key="tag2" v-for="tag2 in addForm.signno" closable :disable-transitions="false" @close="handleClose2(tag2)">
						{{tag2}}
					</el-tag>
					<el-input class="input-new-tag" v-if="inputVisible2" v-model="inputValue2" ref="saveTagInput" size="small" @keyup.enter.native="handleInputConfirm2" @blur="handleInputConfirm2">
					</el-input>
					<el-button v-else class="button-new-tag" size="small" @click="showInput2">点击添加新的签到域</el-button>
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
	import axios from 'axios'
	export default {
		data() {
			return {
				//文件上传
				
				fileList: [],
				limit:1,
				uploadform:{

				},
				fileimportVisible:false,
				//密码重置
				//getUsers相关
				filters: {
					search: ''
				},
				users: [],
				total: 0,
				page: 1,
				pagesize:10,
				border:false,
				listLoading: false,
				sels: [],//列表选中列

				//编辑页面相关
				editFormVisible: false,//编辑界面是否显示
				editLoading: false,
				editFormRules: {
					name: [
						{ required: true, message: '请输入学号', trigger: 'blur' }
					]
				},
				//编辑界面数据
				editForm: {
					name: '',
					workid:'',
					workplace:'',
					signno:[],
					phone:''
				},
				//编辑页面tag
				inputVisible: false,
				inputValue: '',

				addFormVisible: false,//新增界面是否显示
				addLoading: false,
				addFormRules: {
					name: [
						{ required: true, message: '请输入学号', trigger: 'blur' }
					]
				},
				//新增界面数据
				addForm: {
					name: '',
					workid:'',
					workplace:'',
					signno:[],
					phone:''
				},
				//新增页面tag
				inputVisible2: false,
				inputValue2: '',

			}
		},
		methods: {
			//批量导入操作
			//显示批量导入页面
			handleimport(){
				this.fileimportVisible = true;
			},
			//excel下载
			exceldownload(){
				let _this = this;
				window.open("http://120.79.12.163/ordmodel");
			},
			submitUpload() {
				this.$refs.upload.submit();
			},
			handleRemove(file, fileList) {
			},
			handlePreview(file) {
			},
			handleSuccess(res, file, fileList){
				if(res.status==0){
					this.$message({
						message: res.message,
						type: 'error',
						duration:2000,
						showClose:true
					});
				}
				if(res.status==1){
					this.$message({
						message: res.message,
						type: 'success',
						duration:2000,
						showClose:true
					});
				}
			},
			handleError(err, file, fileList){
			},
			handlePro(event, file, fileList){
			},

			//性别显示转换
			formatSex: function (row, column) {
				return row.sex == 1 ? '男' : row.sex == 0 ? '女' : '未知';
			},
			

			//分页器
			handleSizeChange(val) {
				console.log(`每页 ${val} 条`);
				this.pagesize = val;
				this.getUsers(0);
			},
			handleCurrentChange(val) {
				console.log(`当前页: ${val}`);
				this.page = val;
				this.getUsers(0);
				return 
			},

			//得到正确的index索引
			tableindex(index){
				index = (this.page-1)*this.pagesize+index+1;
				return index;
			},
			//获取用户列表
			getUsers(x) {
				let _this = this;
				this.listLoading = true;
				if(x==1){
					this.page = 1;
				}
				axios.get('http://120.79.12.163/getord',{//http://120.79.12.163/getord
					params: {
						page: _this.page,//查询参数
						pagesize:_this.pagesize,
						search: _this.filters.search//查询参数
					}
				})
				.then(function (response) {
					_this.total = response.data.count;
					_this.users = response.data.users;
					_this.listLoading = false;
				})
				.catch(function (error) {
					console.log(error);
				});
			},
			
			//密码重置
			handleresetpsd: function(index, row){
				let _this = this;
				//和删除一样的ID 接口位置sturesetpsd
				this.$confirm('确认要重置密码吗?', '提示', {
					type: 'warning'
				}).then(() => {
					this.listLoading = true;
					axios.get('http://120.79.12.163/ordresetpsd',{
						params: {
							workid: '['+'\"'+row.workid+'\"'+']',
						}
					})
					.then(function (response) {
						let d = response.data;
						if(d.status==1)
						_this.$message({
							message: d.message,
							type: 'success'
						});
						if(d.status==0)
						_this.$message({
							message: d.message,
							type: 'error'
						});
						_this.getUsers(0);
						_this.listLoading = false;
					})
					.catch(function (error) {
						console.log(error);
						_this.listLoading = false;
					});
				}).catch(() => {

				});

			},
			//删除
			handleDel: function (index, row) {
				let _this = this;
				this.$confirm('确认删除该记录吗?', '提示', {
					type: 'warning'
				}).then(() => {
					axios.get('http://120.79.12.163/orddelete',{
						params: {
							workid: '['+'\"'+row.workid+'\"'+']',
						}
					})
					.then(function (response) {
						let d = response.data;
						
						if(d.status==1)
						_this.$message({
							message: d.message,
							type: 'success'
						});
						if(d.status==0)
						_this.$message({
							message: d.message,
							type: 'error'
						});
						_this.getUsers(0);
						_this.listLoading = false;
					})
					.catch(function (error) {
						console.log(error);
						_this.listLoading = false;
					});
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
					workid:'',
					workplace:'',
					signno:[],
					phone:''
				};
			},
			//密码重置
			resetpsdSubmit: function (){
				let _this = this;
			},
			//编辑
			editSubmit: function () {
				let _this = this;
				this.$refs.editForm.validate((valid) => {
					if (valid) {
						this.$confirm('确认提交吗？', '提示', {}).then(() => {
							this.editLoading = true;
							//NProgress.start();
							let para = Object.assign({}, this.editForm);
							//para.birth = (!para.birth || para.birth == '') ? '' : util.formatDate.format(new Date(para.birth), 'yyyy-MM-dd');
							for(let i=0; i<para.signno.length; i++){
								para.signno[i] ='\"'+para.signno[i]+'\"';
							}
							para.signno = '['+para.signno.toString()+']';
							para.way = 2;
							//para.signno 
							axios.get('http://120.79.12.163/ordmodify',{params:para})
							.then(function (response) {
								_this.editLoading = false;
								//NProgress.done();
								let d=response.data
								if(d.status==1)
								_this.$message({
									message: d.message,
									type: 'success'
								});
								if(d.status==0)
								_this.$message({
									message: d.message,
									type: 'error'
								});
								_this.$refs['editForm'].resetFields();
								_this.editFormVisible = false;
								_this.getUsers(0);
							})
							.catch(function (error) {
								console.log(error);
							});
						});
					}
				});
			},
			//编辑页面的tag操作
			handleClose(tag) {
				this.editForm.signno.splice(this.editForm.signno.indexOf(tag), 1);
			},

			showInput() {
				this.inputVisible = true;
				this.$nextTick(_ => {
				this.$refs.saveTagInput.$refs.input.focus();
				});
			},

			handleInputConfirm() {
				let inputValue = this.inputValue;
				if (inputValue) {
				this.editForm.signno.push(inputValue);
				}
				this.inputVisible = false;
				this.inputValue = '';
			},

			//新增
			addSubmit: function () {
				let _this = this;
				this.$refs.addForm.validate((valid) => {
					if (valid) {
						this.$confirm('确认提交吗？', '提示', {}).then(() => {
							this.addLoading = true;
							let para = Object.assign({}, this.addForm);
							
							for(let i=0; i<para.signno.length; i++){
								para.signno[i] ='\"'+para.signno[i]+'\"';
							}
							para.signno = '['+para.signno.toString()+']';
							para.way = 1;
							axios.get('http://120.79.12.163/ordmodify',{params:para})
							.then(function (response) {
								let d = response.data;
								_this.addLoading = false;
								//NProgress.done();
								if(d.status==1)
								_this.$message({
									message: d.message,
									type: 'success'
								});
								if(d.status==0)
								_this.$message({
									message: d.message,
									type: 'error'
								});
								_this.$refs['addForm'].resetFields();
								_this.addFormVisible = false;
								_this.getUsers(0);
							})
							.catch(function (error) {
								console.log(error);
							});
						});
					}
				});
			},

			//新增页面的tag操作
			handleClose2(tag2) {
				this.addForm.signno.splice(this.addForm.signno.indexOf(tag2), 1);
			},
			showInput2() {
				this.inputVisible2 = true;
				this.$nextTick(_ => {
				this.$refs.saveTagInput.$refs.input.focus();
				});
			},
			handleInputConfirm2() {
				let inputValue2 = this.inputValue2;
				if (inputValue2) {
				this.addForm.signno.push(inputValue2);
				}
				this.inputVisible2 = false;
				this.inputValue2 = '';
			},

			selsChange: function (sels) {
				this.sels = sels;
			},
			
			
			
			//批量删除
			batchRemove: function () {
				let _this = this;
				let workids = this.sels.map(item => '\"'+item.workid+'\"').toString();
				workids = '['+workids+']';
				this.$confirm('确认删除选中记录吗？', '提示', {
					type: 'warning'
				}).then(() => {
					let para = { workid: workids };
					axios.get('http://120.79.12.163/orddelete',{params:para})
					.then(function (response) {
						console.log(response);
						let d = response.data;
						_this.listLoading = false;
						if(d.status==1)
						_this.$message({
							message: d.message,
							type: 'success'
						});
						if(d.status==0)
						_this.$message({
							message: d.message,
							type: 'error'
						});
						_this.getUsers(0);
					})
					.catch(function (error) {
						console.log(error);
					});
				}).catch(() => {

				});
			}
		},
		mounted() {
			this.getUsers(0);
		}
	}

</script>

<style scoped>
	.container{
		width: 100%;
		height: 100%;
		left: 0;
		position: relative;
		top: 0;
		background-color: #81c1eb;
	}
	.tablelist{
		border-radius: 8px;
	}
	.el-tag + .el-tag {
		margin-left: 10px;
	}
	.button-new-tag {
		margin-left: 10px;
		height: 32px;
		line-height: 30px;
		padding-top: 0;
		padding-bottom: 0;
	}
	.input-new-tag {
		width: 90px;
		margin-left: 10px;
		vertical-align: bottom;
	}
	.toolbar{
		margin-bottom: 15px;
	}
	.pagebtn{
		background: #658aac;
		color:white;
		border-radius: 5px;
		border:2px solid #658aac;
	}
	.el-button--danger {
		color: #fff;
		background-color: #658aac;
		border-color: #658aac;
	}
	.el-button--danger.is-disabled{
		color: #fff;
		background-color: #658aacab;
		border-color: #658aacab;
	}
	/*导入部分*/
	.el-upload__tip{
	}
	
	
</style>


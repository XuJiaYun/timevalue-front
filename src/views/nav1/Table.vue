<template>
	<section>
		<!--工具条-->
		<el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
			<el-form :inline="true" :model="filters">
				<el-form-item>
					<el-input v-model="filters.name" placeholder="标题"></el-input>
				</el-form-item>
				<el-form-item>
					<el-button type="primary" v-on:click="getNewsByTitle">查询</el-button>
				</el-form-item>
			</el-form>
		</el-col>

		<!--列表-->
		<el-table :data="list" highlight-current-row v-loading="listLoading" @selection-change="selsChange" style="width: 100%;">
			<el-table-column type="selection" width="55">
			</el-table-column>
			<el-table-column prop="id" width="100" label="id">
			</el-table-column>
			<el-table-column prop="title" label="标题" width="400" >
			</el-table-column>
			<el-table-column prop="author" label="作者" width="200">
			</el-table-column>
			<el-table-column prop="timeValue" label="时效性(天)" width="120" >
			</el-table-column>
			<el-table-column prop="releaseTime" label="发表日期" width="250" >
			</el-table-column>
			<el-table-column prop="endTime" label="预计失效日期" min-width="180" >
			</el-table-column>
			<el-table-column label="操作" width="150">
				<template scope="scope">
					<el-button size="small" @click="handleShow(scope.$index, scope.row)">查看</el-button>
					<el-button type="danger" size="small" @click="handleDel(scope.row)">删除</el-button>
				</template>
			</el-table-column>
		</el-table>

		<!--工具条-->
		<el-col :span="24" class="toolbar">
			<el-button type="danger" @click="batchRemove" :disabled="this.sels.length===0">批量删除</el-button>
			<el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="20" :total="total" style="float:right;">
			</el-pagination>
		</el-col>

		<!--查看界面-->
		<el-dialog  v-model="showFormVisible" :close-on-click-modal="false" width="30%" max-height="800">
			<h2>{{news.title}}</h2>
			<h3>{{news.author}}</h3>
      <div>
				<h4>{{news.content}}</h4>
			</div>
			<p></p>
			<span><pre>{{news.information}}</pre></span>
      <span slot="footer" class="dialog-footer">
      <el-button type="primary" @click="showFormVisible = false">确 定</el-button>
      </span>
		</el-dialog>

	</section>
</template>

<script>
	import util from '../../common/js/util'
	//import NProgress from 'nprogress'
	import { getUserListPage, removeUser, batchRemoveUser, editUser, addUser } from '../../api/api';
	import qs from 'qs'

	export default {
		data() {
			return {
				filters: {
					name: ''
				},
				list: [],
				total: 0,
				page: 1,
				listLoading: false,
				sels: [],//列表选中列

				showFormVisible: false,//编辑界面是否显示
				editLoading: false,
				editFormRules: {
					name: [
						{ required: true, message: '请输入姓名', trigger: 'blur' }
					]
				},

				//详情数据
				news: {
					content:'',
					title:'',
					author:'',
					releaseTime:'',
					endTime:'',
					information:''
				},

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
			//性别显示转换
			formatSex: function (row, column) {
				return row.sex == 1 ? '男' : row.sex == 0 ? '女' : '未知';
			},
			handleCurrentChange(val) {
				this.page = val;
				this.getNews();
			},
			//删除
			handleDel: function (row) {
				this.$confirm('确认删除该记录吗?', '提示', {
					type: 'warning'
				}).then(() => {
					this.listLoading = true;
					//NProgress.start();
					this.axios.delete("http://localhost:8080/news/"+row.id).then((res) => {
						this.listLoading = false;
						//NProgress.done();
						this.$message({
							message: '删除成功',
							type: 'success'
						});
						this.getNews();
					});
				}).catch(() => {

				});
			},
			//显示内容界面
			handleShow: function (index, row) {
				this.showFormVisible = true;
				this.news.title = row.title;
				this.news.content = row.content;
				this.news.author = row.author;
				this.news.releaseTime = row.releaseTime;
				this.news.endTime = row.endTime;
				this.news.information = row.information;
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
					//NProgress.start();
					console.log(ids)
					this.axios.delete("http://localhost:8080/news/batchRemove",{
						params:{
							ids:ids
						},
						paramsSerializer: params => {
     		 	return qs.stringify(params, { indices: false })
    			}
					}).then((res) => {
						this.listLoading = false;
						//NProgress.done();
						this.$message({
							message: '删除成功',
							type: 'success'
						});
						this.getNews();
					});
				}).catch(() => {
					
				});
			},
			getNews: function(){
				this.axios
        .get("http://localhost:8080/news/list/" + this.page + "/20",{
					params:{
						title: this.filters.name
					}
				})
        .then(res => {
          console.log(res);
          if (res.data.meta.code == 1000) {
            this.list = [];
            for (var i in res.data.data.newsList) {
              this.list.push(res.data.data.newsList[i]);
            }
            this.pageSize = res.data.data.pageSize;
            this.page = res.data.data.pageNum;
            this.total = res.data.data.total;
          }
        });
			},
			getNewsByTitle: function(){
				this.page = 1;
				this.getNews();
			}
		},
		mounted() {
			this.getNews();
		}
	}

</script>

<style scoped>

</style>
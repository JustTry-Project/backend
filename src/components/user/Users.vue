<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item>Home</el-breadcrumb-item>
      <el-breadcrumb-item>User Management</el-breadcrumb-item>
      <el-breadcrumb-item>User List</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <el-row :gutter="20">
        <el-col :span="5">
          <el-input placeholder="Please input name" v-model="queryInfo.query" clearable></el-input>
        </el-col>
        <el-col :span="3">
          <el-button type="primary" icon="el-icon-search" @click="getList">Search</el-button>
        </el-col>
      </el-row>
      <!-- 用户列表区 -->
      <el-table  v-loading="loading" :data="users" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="Name" prop="username"> </el-table-column>
        <el-table-column label="Nickname" prop="nike"> </el-table-column>
        <el-table-column label="Phone" prop="phone"> </el-table-column>
         <el-table-column label="Account balance" prop="acount"> </el-table-column>
        <el-table-column label="Operation">
          <template slot-scope="scope">
            <el-button
              @click="toEdit(scope.row)"
              type="primary"
              icon="el-icon-edit"
              circle
            ></el-button>
            <el-button @click="userDelete(scope.row.uid)"
             type="danger" icon="el-icon-delete" circle ></el-button>
          </template>
        </el-table-column>
      </el-table>

      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10]"
        :page-size="queryInfo.pageSize"
        layout="total, sizes, prev, pager, next"
        :total="total"
      >
      </el-pagination>
    </el-card>
    <el-dialog title="Modify user" :visible.sync="editShow" width="50%" center @close="editClose" >
      <!-- 内容主题区域 -->
      <el-form :model="editUser" :rules="editRules" ref="editRules" label-width="80px">
        <el-form-item label="Nickname" prop="nike">
          <el-input v-model="editUser.nike"></el-input>
        </el-form-item>
        <el-form-item label="Password" prop="password">
          <el-input v-model="editUser.password"></el-input>
        </el-form-item>
        <el-form-item label="Phone" prop="phone">
          <el-input v-model="editUser.phone"></el-input>
        </el-form-item>
         <el-form-item label="Account balance" prop="acount">
          <el-input v-model="editUser.acount"></el-input>
        </el-form-item>
      </el-form>

      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editClose()">cancel</el-button>
        <el-button type="primary" @click="editCommit('editRules')">confirm</el-button>
      </span>
    </el-dialog>

  </div>
</template>
<script>
import axios from 'axios'
export default {
  data() {
    // 定义电话校验规则
    var checkMobie = (rule,value,callback)=>{
      const regMobie = /^((0\d{2,3}-\d{7,8})|(1[3456789]\d{9}))$/
      if(value==''){callback(new Error('Please enter your mobile number'));}
      else if(!regMobie.test(value)){callback(new Error('Please enter a legal mobile phone number'));}
      else callback();
    }

    var checkPassword =  (rule,value,callback)=>{
      const regPa = /^\w{6,16}$/
      if(value=='')callback(new Error('Please input a password'));
      else if(!regPa.test(value))callback(new Error('The password is composed of 6 ~ 16 digits of numbers, letters and underscores'));
      else callback();
    }

    var checkUsername =  (rule,value,callback)=>{
      const regPa = /^\w{4,16}$/
      if(value=='')callback(new Error('Please enter user name'));
      else if(!regPa.test(value)){
        callback(new Error('The user name is composed of 4 ~ 16 digits of numbers, letters and underscores'));
      }else{
        axios.post('/api/ssm/user/checkUserRegist',{'username':value}).then(res=>{
          if(res.data.flag){
            callback(new Error('User name already exists！'));
          }else{
            callback();
          }
        })
      }
    }


    return {
      total: 0,
      loading:true,
      queryInfo: {
        query: '',
        pagenum: 1,
        pageSize: 5,
      },
      users:[],
      editShow:false,
      editUser:{
        uid:'',
        nike:'',
        phone:'',
        password:'',
        acount:''
      },
      //表单的验证规则对象
      editRules: {
        nike: [{required: true,message: 'Please enter nickname', trigger: 'blur',}],
        username: [{required: true,validator:checkUsername, trigger: 'blur',}],
        phone: [{required: true,validator:checkMobie,trigger:'blur'}],
        password:[{required: true,validator:checkPassword,trigger:'blur'}],
      },
    }
  },
  methods: {
    //获取所有的用户信息
    getList() {
      axios.post('/api/dincan/admin/getUsers',this.queryInfo).then(res=>{
        this.loading = false;
        this.total = res.data.count;
        this.users = res.data.list;
      })
    },
    //底部每页大小改变触发
    handleSizeChange(newSize) {
      this.queryInfo.pageSize = newSize
      this.getList()
    },
    //底部每页当前页码改变时触发
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getList()
    },
    toEdit(user){
      this.editUser.uid = user.uid;
      this.editUser.nike = user.nike;
      this.editUser.password = user.password;
      this.editUser.phone = user.phone;
      this.editUser.acount = user.acount;
      
      this.editShow = true;
    },
    editClose(){
      this.editShow = false;
      this.$nextTick(()=>{
        this.$refs.editRules.resetFields();
      });
    },
    editCommit(formName){
      this.$refs[formName].validate(valid => {
        if(valid){
          //请求
          axios.post('/api/dincan/admin/updateUserBase',this.editUser).then(res=>{
            if(res.data==1){
              this.$message.success('Modified successfully！');
              this.getList();
            }else{
              this.$message.error('Modified failed！');
            }
            this.editShow = false;
          })
        
        }else{
          return false;
        }
      })

    },
    async userDelete(id) {
      const re = await this.$confirm('This operation will permanently delete the user. Do you want to continue?', 'Tips', {
          confirmButtonText: 'confirm',
          cancelButtonText: 'cancel',
          type: 'warning'
      }).catch(err=>err)
      // console.log(re)
      if(re !='confirm'){
        this.$message.info('Deletion cancelled！')
      }else{
        axios.get('/api/dincan/admin/deleteUser?uid='+id).then(res=>{
          if(res.data==1){
            this.$message.success('Deleted successfully！')
            this.getList();
          }else{
            this.$message.error('Deleted failed！')
          }
        })
      }
    },
  
  },
  mounted() {
    this.getList();
  }
}
</script>
<style scoped>
  body {
    margin: 0;
  }
</style>
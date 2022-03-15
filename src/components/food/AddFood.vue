<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item>Home</el-breadcrumb-item>
      <el-breadcrumb-item>Food Management</el-breadcrumb-item>
      <el-breadcrumb-item>Food Add</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-button type="primary" @click="toShop">Return to food list</el-button>
      <el-form :model="addFood" :rules="addRules" ref="addRules" label-width="150px">
        <el-form-item label="Food   name" prop="fname">
          <el-input v-model="addFood.fname"></el-input>
        </el-form-item>
        <el-form-item label="Sell  price" prop="fprice">
          <el-input type="number" v-model="addFood.fprice"></el-input>
        </el-form-item>
        <el-form-item label="Description" prop="fdesc">
            <el-input
            type="textarea"
            :autosize="{ minRows: 2, maxRows: 4}"
            placeholder="Please input"
            v-model="addFood.fdesc">
          </el-input>
        </el-form-item>
         <el-form-item label="Food type" prop="tid">
          <el-select v-model="addFood.tid" placeholder="Please select">
            <el-option
              v-for="item in options"
              :key="item.tid"
              :label="item.tname"
              :value="item.tid">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="Food picture">
          <el-upload
            class="upload-demo"
            action="/api/dincan/admin/uploadImg"
            :on-preview="handlePreview"
            :on-remove="handleRemove"
            :on-success="successUpload"
            :on-error="errorUpload"
            :file-list="fileList"
            :multiple="false"
            list-type="picture">
            <el-button size="small" type="primary">Upload</el-button>
            <div slot="tip" class="el-upload__tip">Only JPG / PNG files can be uploaded, and no more than 500kb</div>
          </el-upload>
        </el-form-item>
        <el-form-item label="  ">
           <el-button type="primary" @click="onFood">Add Food</el-button>
        </el-form-item>
      </el-form>
      
    </el-card>
  </div>
</template>
<script>
import axios from 'axios';
export default {
  data() {
     var valiNumberPass1 = (rule, value, callback) => {//正整数
            if (value === '') {
                callback(new Error('Please enter the market price'));
            } else if (isNaN(parseFloat(value))) {
                callback(new Error('Please enter the correct value'));
            } else {
                callback();
            }
     };
    return {
     addFood:{
       fname:'',
       fprice:'',
       tid:'',
       fdesc:'',
       fpicture:''
     },
     fileList:[],
     options:[],
     addRules:{
        fname:[{ required: true, message: 'Please enter the product name', trigger: 'blur' }],
        fprice:[{ required: true, validator:valiNumberPass1, trigger: "blur" }],
        fdesc:[{ required: true, message: 'Please enter the quantity of goods', trigger: 'blur'}],
     },


    }
  },
  methods: {
    toShop(){
      this.$router.push('/food');
    },
    //获取所有食品类型
    getFoodType(){
      axios.get('/api/dincan/admin/getTypes').then(res=>{  
          this.options = res.data;
      })
    },
     handlePreview(file, fileList){
       
     },
     handleRemove(file, fileList){
       axios.get('/api/dincan/admin/deleteUpload?img='+this.addFood.fpicture).then(res=>{
          this.$message.success('Removed successfully！');
       })
     }, 
     successUpload(response, file, fileList){
      if(response.flag){
        this.addFood.fpicture = response.img;
        this.$message.success('Added successfully！');
      }
     },
     errorUpload(err, file, fileList){
        this.$message.console.error('Add failed！');
     },
     onFood(){
       if(this.addFood.fname!=''&&this.addFood.fprice!=null&&this.addFood.fdesc!=''){
         if(this.addFood.tid!=''){
          if(this.addFood.fpicture!=''){
            axios.post('/api/dincan/admin/addFood',this.addFood).then(res=>{
              if(res.data==1){
                this.$message.success("Added successfully！");
                this.$router.push('/food');
              }else{
                this.$message.error("Add failed！")
              }
            })
          }else{
            this.$message("Please upload food pictures");
          }
        }else{
          this.$message("Please select type");
        }
       }else{
         this.$message("Improve basic food information");
       }
     }
  },
  mounted(){
    this.getFoodType();
  }
}
</script>
<style lang="less" scoped>
.el-steps {
  margin: 15px 0;
  padding: 5px;
}
.el-step__title {
  font-size: 12px;
}
.el-tab-pane {
  border-bottom: black;
}
.el-form{
  width: 60%;
  margin-top: 20px;
  // /* border: 1px solid red; */
  // margin: 5px auto;
  // .el-form-item{
  //   margin: 50px 0px;
  // }
}
</style>
<template>
  <el-card class="form-container" shadow="never">
    <el-form :model="menu" :rules="rules" ref="form" label-width="150px">
      <el-form-item label="菜单名称：" prop="name">
        <el-input v-model="menu.name"></el-input>
      </el-form-item>
      <el-form-item label="菜单路径：" prop="href">
        <el-input v-model="menu.href"></el-input>
      </el-form-item>
      <el-form-item label="上级菜单：">
        <el-select v-model="menu.parentId" placeholder="请选择菜单">
          <el-option
            v-for="item in selectMenuList"
            :key="item.id"
            :label="item.title"
            :value="item.id"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="描述：" prop="description">
        <el-input v-model="menu.description"></el-input>
      </el-form-item>
      <el-form-item label="前端图标：" prop="icon">
        <el-input v-model="menu.icon" style="width: 80%"></el-input>
      </el-form-item>
      <el-form-item label="是否显示：">
        <el-radio-group v-model="menu.shown">
          <el-radio :label="0">是</el-radio>
          <el-radio :label="1">否</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="排序：">
        <el-input v-model="menu.orderNum"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="handleSave()">提交</el-button>
      </el-form-item>
    </el-form>
  </el-card>
</template>

<script>
import { axios } from "../../utils";

const menu = {
  description: "",
  parentId: -1,
  name: "",
  icon: "",
  shown: 0,
  orderNum: 0
};

const rules = {
  description: [
    { required: true, message: "请输入菜单描述", trigger: "blur" },
    {
      min: 2,
      max: 140,
      message: "长度在 2 到 140 个字符",
      trigger: "blur"
    }
  ],
  name: [
    { required: true, message: "请输入菜单名称", trigger: "blur" },
    {
      min: 2,
      max: 140,
      message: "长度在 2 到 140 个字符",
      trigger: "blur"
    }
  ],
  icon: [
    { required: true, message: "请输入前端图标", trigger: "blur" },
    {
      min: 2,
      max: 140,
      message: "长度在 2 到 140 个字符",
      trigger: "blur"
    }
  ]
};

export default {
  name: "MenuDetail",
  props: {
    isEdit: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      menu, //菜单对象
      selectMenuList: [], //下拉列表数据
      rules
    };
  },
  created() {
    if(this.isEdit){
      //修改,回显菜单信息 
      const id = this.$route.query.id;
      //获取当前菜单和父菜单信息 
      this.findMenuInfoById(id);
    }
    else{
      //新增 
      this.menu = {}; 
      //获取父类菜单信息 
      this.findMenuInfoById(-1);
    }
  },
  methods: {
    //方法1: 添加或修改 下拉父菜单回显
    findMenuInfoById(id) {
      axios
      .get("/menu/findMenuInfoById",{
        params:{
          id:id,
        }
      })
      .then(res=>{
        console.log(res.data);
        //判断不为null,修改操作需要回显
        //debugger;
        if(res.data.content.menuInfo!=null){
          this.menu=res.data.content.menuInfo;
          this.selectMenuList=res.data.content.parentMenuList.map(item=>{
            return{id:item.id,title:item.name};
          });
        }
        else{
           this.selectMenuList=res.data.content.map(item=>{
            return{id:item.id,title:item.name};
          });
        }
        //获取到父菜单信息,保存到selectMenuList

        //-1 显示 无上级菜单 (unshift向数组的开头添加一个元素)
        this.selectMenuList.unshift({id: -1, title: "无上级菜单"});
      })
      .catch(err=>{
        this.$message("数据获取失败！！！");
      })
    },

    //保存菜单
    handleSave() {
      this.$refs.form.validate(valid => {
        if (!valid) return false;
        axios
        .post("/menu/saveOrUpdateMenu",this.menu)
        .then(res=>{
          this.$router.back();
        })
        .catch(err=>{
          this.$message("数据加载失败！！！");
        })
      });
    }
  }
};
</script>

<style scoped>
</style>

<template>
  <el-card class="form-container" shadow="never">
    <el-tree
      :data="menuTreeList"
      show-checkbox
      default-expand-all
      node-key="id"
      ref="tree"
      highlight-current
      :props="defaultProps"
    ></el-tree>
    <div style="margin-top: 20px" align="center">
      <el-button type="primary" @click="handleSave()">保存</el-button>
      <el-button @click="handleClear()">清空</el-button>
    </div>
  </el-card>
</template>

<script>
import { axios } from "../../utils";

export default {
  name: "allocMenu",
  title: "角色菜单管理",
  data() {
    return {
      menuTreeList: [], //菜单数据
      checkedMenuId: [], //被选中的菜单

      //树形结构子节点设置
      defaultProps: {
        children: "subMenuList",
        label: "name"
      },
      roleId: null
    };
  },
  //钩子函数
  created() {
    //获取路由携带的id
    this.roleId = this.$route.query.roleId;

    //获取菜单列表
    this.treeList();

    //获取角色所拥有的菜单信息
    this.getRoleMenu(this.roleId);
  },
  methods: {
    //方法1: 获取菜单列表,使用树形控件展示
    treeList() {
      this.listLoading=true;
      axios
      .get("/role/findAllMenu")
      .then(res=>{
        console.log(res.data);
        this.menuTreeList = res.data.content.parentMenuList;
        this.listLoading=false;
      })
      .catch(err=>{
        this.$message("数据加载失败！！！");
      })
    },

    //方法2: 获取当前角色所拥有菜单列表id
    getRoleMenu(roleId) {
      axios.get("/role/findMenuByRoleId",{
        params:{
          roleId:roleId,
        }
      })
      .then(res=>{
        this.$refs.tree.setCheckedKeys(res.data.content);
      })
      .catch(err=>{
        this.$message("数据加载成功！！！");
      })
    },

    //方法3: 修改角色所拥有的菜单列表
    handleSave() {
      //获取所有被选中的节点 
      const checkedNodes = this.$refs.tree.getCheckedNodes();
      //定义常量 保存被选中的菜单id 
      const checkedMenuIds = [];
      if (checkedNodes != null && checkedNodes.length > 0) { 
        //遍历获取节点对象 
        for (let i = 0; i < checkedNodes.length; i++) { 
          const checkedNode = checkedNodes[i]; 
          //保存菜单列表id 
          checkedMenuIds.push(checkedNode.id); 
          //判断: 当前节点为子节点 && 其父ID在数组没有出现过,就保存这个父Id 
          if (checkedNode.parentId !== -1 && checkedMenuIds.filter(item => checkedNode.parentId)
          .length === 0 ) {checkedMenuIds.push(checkedNode.parentId); 
          } 
        } 
      }
      this.$confirm("是否分配菜单？", "提示", {
        confirmButtonText: "确定", 
        cancelButtonText: "取消", 
        type: "warning" })
      .then(() => {
        //准备参数 
        const params = { roleId: this.roleId, 
        //角色ID 
        menuIdList: checkedMenuIds //当前角色拥有的菜单权限ID
       };
       //请求后台
       axios.post("/role/RoleContextMenu",params)
       .then(res=>{
         this.$router.back();
       })
       .catch(err=>{
         this.$message("数据加载失败！！！");
       });
      });
    },

    //清空选择
    handleClear() {
      this.$refs.tree.setCheckedKeys([]);
    }
  }
};
</script>

<style scoped>
</style>

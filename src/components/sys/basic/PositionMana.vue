<template>
  <div>
    <div class="positionManaTool">
      <el-input v-model="role.name" placeholder="请输入角色英文名" size="small">
        <template slot="prepend">ROLE_</template>
      </el-input>
      <el-input v-model="role.nameZh" placeholder="请输入角色中文名" size="small" @keydown.enter.native="doAddRole"></el-input>
      <el-button type="primary" icon="el-icon-plus" size="mini" @click="doAddRole">添加角色</el-button>
    </div>
    <!-- 手风琴 -->
    <div class="positionManaMain">
      <el-collapse v-model="activeName" accordion @change="change">
        <el-collapse-item :title="r.nameZh" :name="r.id" v-for="(r,index) in roles" :key="index">
          <el-card class="box-card">
            <div slot="header" class="clearfix">
              <span>可访问资源</span>
              <el-button type="text" icon="el-icon-delete" style="float: right;padding: 3px 0;color: #f41f0a" @click="doDeleteRole(r)">
              </el-button>
            </div>
            <div>
              <el-tree ref="treeRef" show-checkbox :data="allMenus" :props="defaultProps"
                       :default-checked-keys="selectedMenus"
                       node-key="id" :key="index"></el-tree>
              <div style="display: flex;justify-content: flex-end">
                <el-button size="mini" @click="cancelUpdate">取消修改</el-button>
                <el-button size="mini" type="primary" @click="doUpdate(r.id,index)">确认修改</el-button>
              </div>
            </div>
          </el-card>
        </el-collapse-item>
      </el-collapse>
    </div>
  </div>
</template>

<script>
export default {
  name: "PositionMana",
  data() {
    return {
      role: {
        name: '',
        nameZh: ''
      },
      roles: [],
      allMenus: [],
      defaultProps: { // 树形控件
        children: 'children',
        label: 'name' // 绑定数据 :name="r.id"
      },
      selectedMenus: [],
      activeName: -1 // 折叠面板 默认关闭
    }
  },
  mounted() {
    this.initRoles()
    this.initAllMenus()
  },
  methods:{
    doDeleteRole(role){
      this.$confirm('此操作将永久删除[' + role.nameZh + ']角色, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.deleteRequest('/system/basic/permissing/role/' + role.id).then(resp => {
          if (resp) {
            this.initRoles()
          }
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        });
      });

    },
    doAddRole(){
      if(this.role.name && this.role.nameZh){
          this.postRequest('/system/basic/permissing/role',this.role).then(resp=>{
            if(resp){
              this.initRoles();
              this.role.name = '';
              this.role.nameZh = ''
            }
          })
      }else{
        this.$message.error('所有字段不能为空')
      }
    },
    doUpdate(rid, index) {
      let tree = this.$refs.treeRef[index] // 获取引用对象和索引
      let selectedKeys = tree.getCheckedKeys(true) // 获取选中的节点数组,true 仅返回被选中的叶子节点的 keys,如三级分类
      // console.log(selectedKeys)
      let url = '/system/basic/permissing/?rid=' + rid
      selectedKeys.forEach(key => {
        // 循环遍历出数组 id ，拼接在一起
        url += '&mids=' + key
      })
      this.putRequest(url).then(resp => {
        if (resp) {
          this.activeName = -1 // 关闭折叠面板
        }
      })
    },
    // 取消修改
    cancelUpdate() {
      this.activeName = -1 // 关闭折叠面板
    },
    initRoles(){
      this.getRequest('/system/basic/permissing/').then(resp=>{
        if(resp){
          this.roles = resp;
        }
      })
    },
    initAllMenus(){
      this.getRequest('/system/basic/permissing/menus').then(resp=>{
        if(resp){
          this.allMenus = resp
        }
      })
    },
    initSelectedMenus(rid){
      this.getRequest('/system/basic/permissing/mid/'+rid).then(resp=>{
        if(resp){
          this.selectedMenus = resp
        }
      })
    },
    change(rid){
      if(rid){
        this.initAllMenus();
        this.initSelectedMenus(rid)
      }
    },
  },
}
</script>

<style>
.positionManaTool {
  display: flex;
  justify-content: flex-start;
}

.positionManaTool .el-input {
  width: 300px;
  margin-right: 6px;
}

.positionManaMain {
  margin-top: 10px;
  width: 700px;
}
</style>

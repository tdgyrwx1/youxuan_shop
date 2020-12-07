<template>
  <div>
		<!-- 角色列表界面 -->
    <!-- 面包屑导航区域 通过设置 separator-class 可使用相应的 iconfont 作为分隔符，注意这将使 separator 设置失效 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色按钮区域 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addDialogVisible = true">添加角色</el-button>
        </el-col>
      </el-row>

      <!-- 角色列表区域 api 1.5.1 通过:data="rolelist"绑定数据源 -->
      <el-table :data="rolelist" border stripe>
        <!-- 展开列 通过设置type="expand" 和 Scoped slot可以开启展开行功能 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
						<!-- 通过row 和 col组件，并通过 col组件的 span属性我们就可以自由地组合布局。栅格系统分为24列 scope.row是角色的数据对象 children是角色数据下的一级权限的数组 尽量使用api 1.5.1的id作为key值 -->
						<!-- :class="[]" 动态绑定class数组语法 如果索引i1为0则加顶边框bdtop，否则不加''  -->
            <el-row :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']" v-for="(item1, i1) in scope.row.children" :key="item1.id">
              <!-- 渲染一级权限 :span="5"占整个table的5列 -->
              <el-col :span="5">
								<!-- tag标签 el-icon-caret-right icon小黑右箭头 设置closable属性可以定义一个标签是否可移除。close关闭 Tag 时触发的事件 -->
                <el-tag closable @close="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级和三级权限 二级权限占整个table的19列 -->
              <el-col :span="19">
                <!-- 通过 for 循环 嵌套渲染二级权限 循环遍历一级权限下item1.children的二级权限 -->
                <el-row :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']" v-for="(item2, i2) in item1.children" :key="item2.id">
                  <el-col :span="6">
                    <el-tag type="success" closable @close="removeRightById(scope.row, item2.id)">{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
									<!-- 三级权限。 二级权限的19列再分为24列（包含三级权限）其中二级权限占其中的6列，三级权限占18列 -->
                  <el-col :span="18">
                    <el-tag type="warning" v-for="item3 in item2.children" :key="item3.id" closable @close="removeRightById(scope.row, item3.id)">{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>

            <!-- <pre> 美化渲染
              {{scope.row}}
            </pre> -->
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
				<!-- 操作列用到作用域插槽，所以可以不指定prop -->
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row.id)">编辑</el-button>
            <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeRolesById(scope.row.id)">删除</el-button>
            <el-button size="mini" type="warning" icon="el-icon-setting" @click="showSetRightDialog(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
		
		<!-- 添加角色的对话框 dialog对话框组件 visible属性，它接收Boolean，当为true时显示Dialog。 监听关闭事件@close，关闭对话框后重置 -->
		<el-dialog title="添加角色" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
		  <!-- 添加角色内容主体区域 使用form表单组件 -->
		  <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="80px">
		    <el-form-item label="角色名称" prop="roleName">
		      <el-input v-model="addForm.roleName"></el-input>
		    </el-form-item>
		    <el-form-item label="角色描述" prop="roleDesc">
		      <el-input v-model="addForm.roleDesc"></el-input>
		    </el-form-item>
		  </el-form>
		  <!-- 添加角色底部区域 -->
		  <span slot="footer" class="dialog-footer">
		    <el-button @click="addDialogVisible = false">取 消</el-button>
		    <el-button type="primary" @click="addRoles">确 定</el-button>
		  </span>
		</el-dialog>
		
		<!-- 修改角色的对话框 四个对话框 添加 修改 删除 分配角色 -->
		<el-dialog title="修改角色" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
		  <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="80px">
		    <el-form-item label="角色名称">
					<!-- disabled属性为禁用状态 注：prop为校验规则 roleName和 roleDesc是api提供的 -->
		      <el-input v-model="editForm.roleName"></el-input> 
		    </el-form-item>
		    <el-form-item label="角色描述" prop="roleDesc">
		      <el-input v-model="editForm.roleDesc"></el-input>
		    </el-form-item>
		  </el-form>
		  <span slot="footer" class="dialog-footer">
		    <el-button @click="editDialogVisible = false">取 消</el-button>
		    <el-button type="primary" @click="editRolesInfo">确 定</el-button>
		  </span>
		</el-dialog>

    <!-- 分配权限的对话框 -->
    <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%" @close="setRightDialogClosed">
      <!-- Tree树形控件 :data="rightslist"绑定数据源。 :props树形控件的属性绑定对象 show-checkbox可选框属性。node-key 每个树节点用来作为唯一标识的属性。 default-expand-all是否默认展开所有节点。default-checked-keys	默认勾选的节点的 key 的数组 -->
      <el-tree :data="rightslist" :props="treeProps" show-checkbox node-key="id" default-expand-all :default-checked-keys="defKeys" ref="treeRef"></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 所有角色列表数据
      rolelist: [],
			// 控制添加角色对话框的显示与隐藏
			addDialogVisible: false,
			// 添加角色的表单数据
			addForm: {
				roleName: '',
				roleDesc: ''
			},
			// 添加form表单的验证规则对象 required必填项 trigger: 'blur'失去焦点时触发
			addFormRules: {
			  roleName: [
			    { required: true, message: '请输入角色名', trigger: 'blur' },
			    {
			      min: 3,
			      max: 10,
			      message: '角色名的长度在3~10个字符之间',
			      trigger: 'blur'
			    }
			  ]
			},
			// 控制修改角色对话框的显示与隐藏
			editDialogVisible: false,
			// 查询到的角色信息对象
			editForm: {},
			// 修改表单的验证规则对象
			editFormRules: {
			  roleName: [
			    { required: true, message: '请输入角色名称', trigger: 'blur' },
			  ],
			  roleDesc: [
			    { required: true, message: '请输入角色描述', trigger: 'blur' },
			  ]
			},
      // 控制分配权限对话框的显示与隐藏
      setRightDialogVisible: false,
      // 所有权限的数据
      rightslist: [],
      // 树形控件的属性绑定对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认选中的节点Id值数组
      defKeys: [],
      // 当前即将分配权限的角色id
      roleId: ''
    }
  },
  created() { // 在页面创建时获取列表数据
    this.getRolesList()
  },
  methods: {
    // 获取所有角色的列表
    async getRolesList() {
      const { data: res } = await this.$http.get('roles') //api 1.5.1

      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败！')
      }
			// 将角色列表数据 赋值给res.data
      this.rolelist = res.data

      // console.log(this.rolelist)
    },
		// 监听添加角色对话框的关闭事件
		addDialogClosed() {
		  this.$refs.addFormRef.resetFields() //获取addFormRef的引用$refs，并使用Form Methods resetFields方法，对整个表单进行重置，将所有字段值重置为初始值并移除校验结果
		},
		// 点击按钮，添加新角色
		addRoles() {
		  this.$refs.addFormRef.validate(async valid => {//Form Methods validate方法
		    if (!valid) return
		    // 可以发起添加角色的网络请求 api接口 1.5.2
		    const { data: res } = await this.$http.post('roles', this.addForm)//路径roles 参数this.addForm
		
		    if (res.meta.status !== 201) {// 1.3.2成功的状态是201
		      this.$message.error('添加角色失败！')
		    }
		
		    this.$message.success('添加角色成功！')
		    // 添加成功后，隐藏添加角色的对话框
		    this.addDialogVisible = false
		    // 成功添加角色后，重新获取角色列表数据（刷新列表）
		    this.getRolesList()
		  })
		},
		// 展示编辑角色的对话框
		async showEditDialog(id) {
		  console.log(id)
		  const { data: res } = await this.$http.get('roles/' + id) // api 1.5.3
		
		  if (res.meta.status !== 200) {
		    return this.$message.error('查询角色信息失败！')
		  }
		
		  this.editForm = res.data //editForm查询到的角色信息对象 保存到res的数据中
		  this.editDialogVisible = true
		},
		// 监听编辑角色对话框的关闭事件
		editDialogClosed() {
		  this.$refs.editFormRef.resetFields()// 同上添加角色
		},
		// 编辑角色信息并提交 完成提交修改之前的表单预验证
		editRolesInfo() {
		  this.$refs.editFormRef.validate(async valid => {
		    if (!valid) return // 如果valid校验失败 直接返回
		    // 发起编辑角色信息的数据请求
		    const { data: res } = await this.$http.put(// api1.5.4
		      'roles/' + this.editForm.roleId,// roleId 在editForm中获取 this.editForm = res.data
		      {
		        roleName: this.editForm.roleName,
		        roleDesc: this.editForm.roleDesc
		      }
		    )
				console.log(this.editForm);
		    if (res.meta.status !== 200) {
		      return this.$message.error('更新角色信息失败！')// message消息提示组件 Element为 Vue.prototype 添加了全局方法 $message。
		    }
		
		    // 修改成功后关闭对话框 一般修改成功都会有下面三个操作
		    this.editDialogVisible = false
		    // 刷新数据列表
		    this.getRolesList()
		    // 提示修改成功
		    this.$message.success('更新角色信息成功！')
		  })
		},
		// 根据Id删除对应的角色信息
		async removeRolesById(id) {
		  // 弹框询问角色是否删除数据
		  const confirmResult = await this.$confirm(// confirmResult 接收this.$confirm 的返回值
		    '此操作将永久删除该角色, 是否继续?',// 参数 参考elementUI MessageBox写法
		    '提示',
		    {
		      confirmButtonText: '确定',
		      cancelButtonText: '取消',
		      type: 'warning'
		    }
		  ).catch(err => err)// 相当于 err => {return err} 函数体只有一条可以省略{}和return
		  // 如果角色确认删除，则返回值为字符串 confirm
		  // 如果角色取消了删除，则返回值为字符串 cancel
		  console.log(confirmResult)
		  if (confirmResult !== 'confirm') {
		    return this.$message.info('已取消删除')
		  }
			// 发起删除角色的请求 
		  const { data: res } = await this.$http.delete('roles/' + id) // api 1.3.6 删除 请求方法一般用 delete
		
		  if (res.meta.status !== 200) {
		    return this.$message.error('删除角色失败！')
		  }
		
		  this.$message.success('删除角色成功！')
		  this.getRolesList() //重新刷新角色列表
		},
    // 根据Id删除对应的权限
    async removeRightById(role, rightId) {// role接收scope.row中的id rightId接收item3.id
      // 弹框提示角色是否要删除
      const confirmResult = await this.$confirm(// 参考messagebox弹框 $confirm的返回值是promise
        '此操作将永久删除该文件, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err) // catch 捕获取消行为 相当于 err => {return err} 函数体只有一条可以省略{}和return
      // 如果角色确认删除，则返回值为字符串 confirm
      // 如果角色取消了删除，则返回值为字符串 cancel
      console.log(confirmResult)

      if (confirmResult !== 'confirm') {
        return this.$message.info('取消了删除！')
      }

      const { data: res } = await this.$http.delete(// api 1.5.7
        `roles/${role.id}/rights/${rightId}`
      )

      if (res.meta.status !== 200) {
        return this.$message.error('删除权限失败！')
      }

      // this.getRolesList() 不建议完整渲染，重新给data赋值即可
      role.children = res.data
    },
    // 展示分配权限的对话框
    async showSetRightDialog(role) {// role接收scope.row
      this.roleId = role.id
      // 获取所有权限的数据
      const { data: res } = await this.$http.get('rights/tree') //api 1.4.1

      if (res.meta.status !== 200) {
        return this.$message.error('获取权限数据失败！')
      }

      // 把获取到的权限数据保存到 data 中
      this.rightslist = res.data
      // console.log(this.rightslist)

      // 递归获取三级节点的Id 注：树形图经常使用递归 参考pinkjs高级 第三天
      this.getLeafKeys(role, this.defKeys) //defKeys 默认选中的节点Id值数组  role接收scope.row

      this.setRightDialogVisible = true
    },
    // 通过递归的形式，获取角色下所有三级权限的id，并保存到 defKeys 数组中
    getLeafKeys(node, arr) {
      // 如果当前 node 节点不包含 children 属性，则是三级节点
      if (!node.children) {
        return arr.push(node.id)
      }

      node.children.forEach(item => this.getLeafKeys(item, arr))
    },
    // 监听分配权限对话框的关闭事件 在每次打开分配权限时清除上次缓存的数组
    setRightDialogClosed() {
      this.defKeys = []
    },
    // 点击为角色分配权限
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(), //若节点可被选择（即 show-checkbox 为 true），则返回目前被选中的节点的 key 所组成的数组
        ...this.$refs.treeRef.getHalfCheckedKeys() //若节点可被选择（即 show-checkbox 为 true），则返回目前半选中的节点的 key 所组成的数组
      ]
			// console.log(keys);
			
      const idStr = keys.join(',') //join(分隔符) 把数组转换为字符串

      const { data: res } = await this.$http.post( //api 1.5.6
        `roles/${this.roleId}/rights`,
        { rids: idStr } //请求体
      )

      if (res.meta.status !== 200) {
        return this.$message.error('分配权限失败！')
      }

      this.$message.success('分配权限成功！')
      this.getRolesList() //刷新权限列表
      this.setRightDialogVisible = false //隐藏对话框
    }
  }
}
</script>

<style lang="less" scoped>
.el-tag {// 一级权限标签
  margin: 7px;
}

.bdtop {
  border-top: 1px solid #eee;
}

.bdbottom {
  border-bottom: 1px solid #eee;
}

.vcenter {
  display: flex;
	// 側轴居中对齐 默认主轴为x轴，側轴为y轴
  align-items: center;
}
</style>

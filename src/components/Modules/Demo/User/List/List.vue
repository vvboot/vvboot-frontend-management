<template>
    <div class="list">
        <el-col :span="24" class='actions-top'>
            <el-button type='danger' icon='delete'
                       :disabled='batch_flag'
                       @click='onDeleteUser(true)'>删除选中
            </el-button>

            <el-form :inline="true" :model='search_data' class="demo-form-inline">
                <el-form-item>
                    <el-input placeholder="姓名"
                              v-model='search_data.username'
                              clear></el-input>
                </el-form-item>
                <el-form-item>
                    <el-input placeholder="邮箱" v-model='search_data.email'></el-input>
                </el-form-item>
                <el-form-item>
                    <el-button type="default" @click='onSearch'>查询</el-button>
                </el-form-item>
            </el-form>
        </el-col>
        <el-table border style="width: 100%" align='center'
                  :data="user_list"
                  @selection-change='onSelectionChange'>
            <el-table-column
                    label='住址'
                    type="expand">
                <template scope="props">
                    <p>住址: {{ props.row.address }}</p>
                </template>
            </el-table-column>

            <el-table-column
                    type="selection"
                    width="55">
            </el-table-column>
            <el-table-column
                    prop="username"
                    label="姓名"
                    align="center"
                    width="150"
                    :sortable="true">
            </el-table-column>
            <el-table-column
                    prop="email"
                    label="邮箱"
                    align="center"
                    :sortable="true">
            </el-table-column>
            <el-table-column
                    prop="sex"
                    label="性别"
                    align="center"
                    width="130"
                    sortable="true"
                    :formatter="formatterSex"
                    :filters="sex_filters.list"
                    :filter-method="filterSex"
                    :filter-multiple="sex_filters.multiple">
            </el-table-column>
            <el-table-column
                    prop="create_time"
                    label="创建时间"
                    align="center"
                    width="180"
                    :sortable="true">
            </el-table-column>
            <el-table-column
                    prop="status"
                    label="状态"
                    align="center"
                    width="130"
                    :sortable="true"
                    :formatter="formatterStatus"
                    :filters="status_filters.list"
                    :filter-method="filterStatus"
                    :filter-multiple="status_filters.multiple">
            </el-table-column>

            <el-table-column
                    label="操作"
                    :width="$store.state.user.userinfo.pid==0 ? 280 : 260"
                    :context="_self">
                <template scope='scope'>
                    <el-button
                            type="info"
                            icon='view'
                            size="mini"
                            @click='onSelectUser(scope.row)'></el-button>
                    <el-button
                            type="info"
                            icon='edit'
                            size="mini"
                            @click='onEditUser(scope.row)'></el-button>

                    <el-button
                            type="primary"
                            size="mini"
                            @click='onSetAccess(scope.row,scope.$index,user_list)'>设置权限
                    </el-button>


                    <el-button
                            type="danger"
                            icon='delete'
                            size="mini"
                            @click='onDeleteUser(scope.row,scope.$index,user_list)'></el-button>


                    <!-- 不要试图强制打开这个操作，因为服务端做了验证，打开也没用n(*≧▽≦*)n -->
                    <el-button
                            size="mini"
                            :type="scope.row.status==1 ? 'danger' : 'info'"
                            @click='onSetStatusUser(scope.row,scope.$index,user_list)'>{{scope.row.status==1 ? '禁用' :
                        '启用'}}
                    </el-button>
                </template>
            </el-table-column>
        </el-table>

        <el-dialog title="用户信息" v-model="dialog.show" size="tiny">
            <el-form style="margin:20px;width:60%;min-width:100%"
                     label-width="100px"
                     :model="dialog.user_info">
                <el-form-item class='edit-form'
                              label="邮箱"
                              prop='email'>
                    {{dialog.user_info.email}}
                </el-form-item>
                <el-form-item class='edit-form'
                              label="用户名称"
                              prop='username'>
                    {{dialog.user_info.username}}
                </el-form-item>
                <el-form-item label="性别">
                    {{dialog.user_info.sex==1 ? '男' : (dialog.user_info.sex==2 ? '女' : '未知')}}
                </el-form-item>
                <el-form-item label="生日">
                    {{dialog.user_info.birthday}}
                </el-form-item>
                <el-form-item class="edit-form"
                              label='地址'>
                    {{dialog.user_info.address}}
                </el-form-item>
                <el-form-item label="状态">
                    {{dialog.user_info.status==1 ? '启用' : '禁用'}}
                </el-form-item>
                <el-form-item label="创建时间">
                    {{dialog.user_info.create_time}}
                </el-form-item>
                <el-form-item label="最后更新时间">
                    {{dialog.user_info.update_time}}
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="dialog.show = false">取 消</el-button>
                <el-button type="primary" @click="dialog.show = false">确 定</el-button>
            </span>
        </el-dialog>


        <el-dialog title="设置权限" v-model="dialog_access.show" size="small">
            <el-form style="margin:20px;width:60%;min-width:100%"
                     label-width="100px"
                     :model="dialog_access.userinfo">
                <el-form-item class='edit-form'
                              label="邮箱"
                              prop='email'>
                    {{dialog_access.userinfo.email}}
                </el-form-item>
                <el-form-item class='edit-form'
                              label="用户名称"
                              prop='username'>
                    {{dialog_access.userinfo.username}}
                </el-form-item>

                <el-form-item class='edit-form'
                              label="前端页面">
                    <!-- CheckBox选项列表 -->
                    <el-checkbox :indeterminate="isIndeterminate" v-model="checkAll" @change="handleCheckAllChange">全选
                    </el-checkbox>
                    <div style="margin: 15px 0;"></div>
                    <el-checkbox-group v-model="checkedCities" @change="handleCheckedCitiesChange">
                        <el-checkbox v-for="city in cities" :label="city" :key="city">{{city}}</el-checkbox>
                    </el-checkbox-group>
                </el-form-item>

                <el-form-item class='edit-form'>
                    <el-tree
                            class="filter-tree"
                            show-checkbox
                            default-expand-all
                            node-key="path"
                            :data="accesss"
                            :props="defaultProps"
                            :filter-node-method="filterNode"
                            @check-change='checkChange'
                            @current-change='currentChange'
                            @node-click='nodeClick'
                            ref="accesss">
                    </el-tree>
                </el-form-item>

            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="dialog_access.show = false">取 消</el-button>
                <el-button type="primary" @click="dialog_access.show = false">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
  module.exports = {
    name: 'list',
    data() {
      return {
        user_list: [], //用户列表数组

        batch_id: '', //批量删除时这是多个用逗号隔开的id字符串
        batch_flag: true, //符合批量删除为true,否则为false

        //搜索表单信息
        search_data: {
          username: '',
          email: '',
        },

        //详情弹框信息
        dialog: {
          show: false,
          user_info: {}
        },

        dialog_access:{
          show:false,
          userinfo:{},
          web_routers:[],
          api_routers:[]
        },

        //列表过滤性别
        sex_filters: {
          list: [{
            text: '男',
            value: 1
          }, {
            text: '女',
            value: 2
          }, {
            text: '保密',
            value: 3
          }],
          multiple: false
        },

        //列表过滤状态
        status_filters: {
          list: [{
            text: '启用',
            value: 1
          }, {
            text: '禁用',
            value: 2
          }],
          multiple: false
        },



        checkAll: true,
        checkedCities: ['上海', '北京'],
        cities: ['上海', '北京', '广州', '深圳'],
        isIndeterminate: true,

        accesss: [],
        checkeds: [],
        defaultProps: {
          children: 'children',
          label: 'name'
        }

      }
    },
    methods: {
      handleCheckAllChange(event) {
        this.checkedCities = event.target.checked ? this.cities : [];
        this.isIndeterminate = false;
      },
      handleCheckedCitiesChange(value) {
        let checkedCount = value.length;
        this.checkAll = checkedCount === this.cities.length;
        this.isIndeterminate = checkedCount > 0 && checkedCount < this.cities.length;
      },

      filterNode(value, data) {
        if (!value) return true;
        return data.label.indexOf(value) !== -1;
      },

      currentChange(data, node) {
        // console.log(data, node);
      },

      nodeClick(data, node, self) {
        // console.log(node);
      },

      checkChange(data, selfIsChecked, childHasChecked) {
        if (selfIsChecked === true && data.access.split('/').length == 4 && this.checkeds.indexOf(data.access) === -1) {
          this.checkeds.push(data.access);
        } else {
          var index = this.checkeds.indexOf(data.access);
          if (index !== -1) {
            this.checkeds.splice(index, 1);
          }
        }
      },

      setUserAccess() {
        var flag = false;
        for (var i = 0; i < this.checkeds.length; i++) {
          var arr = this.checkeds[i].split('/');

          if (arr.length === 4) {
            flag = true;
            var rootPath = '/' + arr[1],
              twoPath = rootPath + '/' + arr[2];

            if (this.checkeds.indexOf(rootPath) === -1) {
              this.checkeds.push(rootPath);
            }
            if (this.checkeds.indexOf(twoPath) === -1) {
              this.checkeds.push(twoPath);
            }
          }
        }

        //当前所选中的节点
        if (flag === false) {
          this.checkeds = [];
        }

        // console.log(this.checkeds.join(','));
        // console.log(this.user_id.join(','));


        if (this.user_id.length) {
          UserApi.setAccessUser.call(this, {
            user_id: this.user_id.join(','),
            user_accesss: this.checkeds.join(',')
          }, data => {
            this.$message.success('设置成功');
          });
        } else {
          this.$message.error('用户不能为空');
        }
      },


      initRouters(){
        var routes = this.$router.options.routes;
        for (var i = 0; i < routes.length; i++) {
          if (routes[i].hidden !== true && routes[i].children && routes[i].children.length) {
            var tempObj = {},
              module = routes[i],
              menus = module.children;
            tempObj.name = module.name;
            tempObj.path = module.path;
            tempObj.access = module.path;
            tempObj.children = [];
            for (var j = 0; j < menus.length; j++) {
              if (menus[j].hidden !== true && menus[j].children && menus[j].children.length) {
                var tempChildObj = {},
                  menu = menus[j],
                  pages = menu.children;
                tempChildObj.name = menu.name;
                tempChildObj.path = '/' + menu.path;
                tempChildObj.access = tempObj.path + '/' + menu.path;
                tempChildObj.children = [];
                for (var k = 0; k < pages.length; k++) {
                  if (pages[k].hidden !== true) {
                    var tempPageObj = {},
                      page = pages[k];
                    tempPageObj.name = page.name;
                    tempPageObj.path = '/' + page.path;
                    tempPageObj.access = tempObj.path + '/' + menu.path + '/' + page.path;
                    tempChildObj.children.push(tempPageObj);
                  }
                }
                tempObj.children.push(tempChildObj);
              }
            }
            this.accesss.push(tempObj);
          }
        }
      },


      /**
       * 列表性别格式化事件
       * @param  {object} item 当前用户信息
       * @return {string}      根据定义的类型转换文本描述值
       */
      formatterSex(item) {
        return item.sex == 1 ? '男' : (item.sex == 2 ? '女' : '保密');
      },


      /**
       * 列表状态格式化事件
       * @param  {object} item 当前用户信息
       * @return {string}      根据定义的类型转换文本描述值
       */
      formatterStatus(item) {
        return item.status == 1 ? '启用' : '禁用';
      },


      /**
       * 列表过滤性别事件
       * @param  {number} value 状态码
       * @param  {object} item  当前用户信息
       * @return {boolean}       匹配成功为true,否则为false
       */
      filterSex(value, item) {
        return item.sex == value;
      },


      /**
       * 列表过滤状态事件
       * @param  {number} value 状态码
       * @param  {object} item  当前用户信息
       * @return {boolean}       匹配成功为true,否则为false
       */
      filterStatus(value, item) {
        return item.status == value;
      },


      /**
       * 点击搜索按钮事件
       */
      onSearch() {
        // console.log(this.search_data);

        var sd = {};

        var query = this.$route.query;
        for (var p in query) {
          sd[p] = query[p];
        }

        for (var s in this.search_data) {
          sd[s] = this.search_data[s];

          if (!sd[s]) {
            delete sd[s];
          }
        }


        this.$router.push({
          path: this.$route.path,
          query: sd
        });
      },


      /**
       * 表格列表触发CheckBox的事件
       * @param  {array} val 当前选中的用户信息数组，每个元素是用户信息对象
       */
      onSelectionChange(val) {
        // console.log(val);
        if (val.length) {
          this.batch_flag = false;
          var ids = [];
          for (var i = 0; i < val.length; i++) {
            ids.push(val[i].id);
          }
          this.batch_id = ids.join(',');
        } else {
          this.batch_flag = true;
          this.batch_id = '';
        }
      },


      /**
       * 改变页码和当前页时需要拼装的路径方法
       * @param {string} field 参数字段名
       * @param {string} value 参数字段值
       */
      setPath(field, value) {
        var path = this.$route.path,
          query = Object.assign({}, this.$route.query);

        query[field] = value;

        this.$router.push({
          path: path,
          query: query
        });
      },


      /**
       * 改变当前页事件
       * @param  {number} page 当前页码
       */
      onChangeCurrentPage(page) {
        this.setPath('page', page);
      },


      /**
       * 改变每页显示数量事件
       * @param  {number} size 当前每页显示数量
       */
      onChangePageSize(size) {
        this.setPath('page_size', size);
      },


      /**
       * 设置状态
       */
      onSetStatusUser(user, index, list) {
        this.$$api_user_updateUserStatus({
          id: user.id
        }, (data) => {
          user.status = user.status == 1 ? 2 : 1;
        });
      },


      /**
       * 设置权限
       */
      onSetAccess(user,index,list){
        this.$router.push({
          path:'/demo/user/access',
          query:{
            id:user.id
          }
        });

        // this.dialog_access.userinfo=user;
        // this.dialog_access.show=true;
      },


      /**
       * 删除用户事件
       * @param  {object || boolean} user  当前用户信息对象或者为布尔值,为布尔值时，代表是批量删除
       * @param  {number} index 当前用户列表索引
       * @param  {array} list  当前用户列表数组
       */
      onDeleteUser(user, index, list) {
        // console.dir(user);

        if (user === true) {
          var id = this.batch_id;
        } else {
          var id = user.id;
        }
        this.$confirm('你确定删除用户 '+user.username+' 么?', '删除用户', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$$api_user_deleteUser({
            id: id
          }, (data) => {
            if (user === true) {
              this.user_list = this.user_list.filter(function(item, idx) {
                return id.indexOf(item.id) === -1;
              });
            } else {
              list.splice(index, 1);
            }
            this.getList();
          });
        });

      },


      /**
       * 修改用户
       * @param  {object} user 当前用户信息对象
       */
      onEditUser(user) {
        if (user && user.id) {
          this.$router.push('/demo/user/edit?id=' + user.id);
        } else {
          this.$message({
            showClose: true,
            message: 'ID跑哪去了？',
            type: 'error'
          });
        }
      },


      /**
       * 查看用户信息事件
       * @param  {object} user 当前用户信息对象
       */
      onSelectUser(user) {
        this.dialog.show = true;
        this.dialog.user_info = user;
      },


      /**
       * 获取用户信息列表方法
       */
      getList() {
        var data = {};

        var query = this.$route.query;
        for (var k in query) {
          if (this.search_data[k] !== undefined) {
            this.search_data[k] = query[k];
            data[k] = query[k];
          }
        }

        this.$$api_user_selectUser(data, (data) => {
          this.user_list = data.list;
        });
      }
    },

    mounted() {
      this.getList();

      this.initRouters();

      //test dialog

      /* setTimeout(() => {
       this.onSelectUser(this.user_list[0]);
       }, 600);*/
    },
    watch: {
      '$route' (to, from) {
        this.getList();
      }
    }
  }
</script>
<style scoped lang='less'>
    .demo-form-inline {
        display: inline-block;
        float: right;
    }

    .btm-action {
        margin-top: 20px;
        text-align: center;
    }

    .actions-top {
        height: 46px;
    }

    .pagination {
        display: inline-block;
    }
</style>

<template Enabletheming=“false”>
  <a-card :bordered="false" class="card-area">
    <div :class="advanced ? 'search' : null">
      <!-- 搜索区域 -->
      <a-form layout="horizontal">
        <a-row>
          <div :class="advanced ? null: 'fold'">
            <a-col :md="12" :sm="24">
              <a-form-item label="住院号" :labelCol="{span: 4}" :wrapperCol="{span: 18, offset: 2}">
                <a-input v-model="queryParams.username" />
              </a-form-item>
            </a-col>
            <a-col :md="12" :sm="24">
              <a-form-item label="入院时间" :labelCol="{span: 4}" :wrapperCol="{span: 18, offset: 2}">
                <range-date @change="handleDateChange" ref="createTime"></range-date>
              </a-form-item>
            </a-col>
            <template v-if="advanced">
              <a-col :md="12" :sm="24">
                <a-form-item
                  label="其他查询1"
                  :labelCol="{span: 4}"
                  :warpperCol="{span: 18, offset: 2}"
                >
                  <a-input />
                </a-form-item>
              </a-col>
              <a-col :md="12" :sm="24">
                <a-form-item
                  label="其他查询2"
                  :labelCol="{span: 4}"
                  :warpperCol="{span: 18, offset: 2}"
                >
                  <a-input />
                </a-form-item>
              </a-col>
            </template>
          </div>
          <span style="float: right; margin-top: 3px;">
            <a-button type="primary" @click="search">查询</a-button>
            <a-button style="margin-left: 8px" @click="reset">重置</a-button>
            <a @click="toggleAdvanced" style="margin-left: 8px">
              {{advanced ? '收起' : '展开'}}
              <a-icon :type="advanced ? 'up' : 'down'" />
            </a>
          </span>
        </a-row>
      </a-form>
    </div>
    <div>
      <!-- 表格区域 :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"-->
      <a-table
        ref="TableInfo"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="pagination"
        :loading="loading"
        :scroll="{ x: 900 }"
        @change="handleTableChange"
      >
        <!-- <template slot="email" slot-scope="text, record">
          <a-popover placement="topLeft">
            <template slot="content">
              <div>{{text}}</div>
            </template>
            <p style="width: 150px;margin-bottom: 0">{{text}}</p>
          </a-popover>
        </template>
        <template slot="operation" slot-scope="text, record">
          <a-icon v-hasPermission="'user:update'" type="setting" theme="twoTone" twoToneColor="#4a9ff5" @click="edit(record)" title="修改用户"></a-icon>
          &nbsp;
          <a-icon v-hasPermission="'user:view'" type="eye" theme="twoTone" twoToneColor="#42b983" @click="view(record)" title="查看"></a-icon>
          <a-badge v-hasNoPermission="'user:update','user:view'" status="warning" text="无权限"></a-badge>
        </template>-->
        <!-- <a href="javascript:;" slot-scope="text" slot="operation">Publish</a>    -->
        <template slot="operation" slot-scope="text, record">
          <a href="javascript:;"  @click="addReport(record.pid, record.patientName)">Publish</a>
        </template>
        <!-- <router-link to="/infection/report/add" tag="a" slot="operation" slot-scope="text">Publish</router-link> -->

        <!-- slot-scope="text" slot="operation" -->
        <a-table
          slot="expandedRowRender"
          slot-scope="text"
          :columns="innerColumns"
          :dataSource="innerData"
          :pagination="false"
        >
          <span slot="status" slot-scope="text">
            <a-badge status="success" />Finished
          </span>
          <span slot="operation" slot-scope="text" class="table-operation">
            <a href="javascript:;">Pause</a>
            <a href="javascript:;">Stop</a>
            <a-dropdown>
              <a-menu slot="overlay">
                <a-menu-item>Action 1</a-menu-item>
                <a-menu-item>Action 2</a-menu-item>
              </a-menu>
              <a href="javascript:;">
                More
                <a-icon type="down" />
              </a>
            </a-dropdown>
          </span>
        </a-table>
      </a-table>
    </div>
  </a-card>
  <!-- <div id="components-layout-demo-basic">
    <a-layout>
      <a-layout-sider collapsible v-model="collapsed">
        <ul>
          <li>患者号  姓名   性别 ===================</li>
          <li v-for="c in cList" :key="c.pid">
            {{ c.pid }}  {{ c.pname }}  {{ c.gender }}
          </li>
        </ul>
      </a-layout-sider>
      <a-layout>
        <a-layout-header>Header</a-layout-header>
        <a-layout-content>Content</a-layout-content>
        <a-layout-footer>Footer</a-layout-footer>
      </a-layout>
    </a-layout>
  </div>-->
</template>

<script>
/* eslint-disable */
const innerColumns = [
  { title: "生成时间", dataIndex: "date", key: "date" },
  { title: "内容", dataIndex: "name", key: "name" },
  { title: "感染诊断", key: "state", scopedSlots: { customRender: "status" } },
  { title: "标本", dataIndex: "upgradeNum", key: "upgradeNum" },
  {
    title: "Action",
    dataIndex: "operation",
    key: "operation",
    scopedSlots: { customRender: "operation" }
  }
];

const innerData = [];
for (let i = 0; i < 1; ++i) {
  innerData.push({
    key: i,
    date: "2014-12-24 23:12:00",
    name: "上呼吸道感染",
    upgradeNum: "标本"
  });
}
import RangeDate from "@/components/datetime/RangeDate";
export default {
  name: "User",
  components: { RangeDate },
  data() {
    return {
      advanced: false,
      userInfo: {
        visiable: false,
        data: {}
      },
      userAdd: {
        visiable: false
      },
      userEdit: {
        visiable: false
      },
      queryParams: {},
      filteredInfo: null,
      sortedInfo: null,
      paginationInfo: null,
      dataSource: [],
      selectedRowKeys: [],
      loading: false,
      pagination: {
        pageSizeOptions: ["10", "20", "30", "40", "100"],
        defaultCurrent: 1,
        defaultPageSize: 10,
        showQuickJumper: true,
        showSizeChanger: true,
        showTotal: (total, range) =>
          `显示 ${range[0]} ~ ${range[1]} 条记录，共 ${total} 条记录`
      },
      innerColumns,
      innerData
    };
  },
  computed: {
    columns() {
      let { sortedInfo, filteredInfo } = this;
      sortedInfo = sortedInfo || {};
      filteredInfo = filteredInfo || {};
      // [{
      //   title: '用户名',
      //   dataIndex: 'username',
      //   sorter: true,
      //   sortOrder: sortedInfo.columnKey === 'username' && sortedInfo.order
      // }, {
      //   title: '性别',
      //   dataIndex: 'ssex',
      //   customRender: (text, row, index) => {
      //     switch (text) {
      //       case '0':
      //         return '男'
      //       case '1':
      //         return '女'
      //       case '2':
      //         return '保密'
      //       default:
      //         return text
      //     }
      //   },
      //   filters: [
      //     { text: '男', value: '0' },
      //     { text: '女', value: '1' },
      //     { text: '保密', value: '2' }
      //   ],
      //   filterMultiple: false,
      //   filteredValue: filteredInfo.ssex || null,
      //   onFilter: (value, record) => record.ssex.includes(value)
      // }, {
      //   title: '邮箱',
      //   dataIndex: 'email',
      //   scopedSlots: { customRender: 'email' },
      //   width: 100
      // }, {
      //   title: '部门',
      //   dataIndex: 'deptName'
      // }, {
      //   title: '电话',
      //   dataIndex: 'mobile'
      // }, {
      //   title: '状态',
      //   dataIndex: 'status',
      //   customRender: (text, row, index) => {
      //     switch (text) {
      //       case '0':
      //         return <a-tag color="red">锁定</a-tag>
      //       case '1':
      //         return <a-tag color="cyan">有效</a-tag>
      //       default:
      //         return text
      //     }
      //   },
      //   filters: [
      //     { text: '有效', value: '1' },
      //     { text: '锁定', value: '0' }
      //   ],
      //   filterMultiple: false,
      //   filteredValue: filteredInfo.status || null,
      //   onFilter: (value, record) => record.status.includes(value)
      // }, {
      //   title: '创建时间',
      //   dataIndex: 'createTime',
      //   sorter: true,
      //   sortOrder: sortedInfo.columnKey === 'createTime' && sortedInfo.order
      // }, {
      //   title: '操作',
      //   dataIndex: 'operation',
      //   scopedSlots: { customRender: 'operation' }
      // }]
      return [
        {
          title: "住院号",
          dataIndex: "pid",
          sorter: true,
          sortOrder: sortedInfo.columnKey === "pid" && sortedInfo.order
        },
        {
          title: "患者姓名",
          dataIndex: "patientName",
          sorter: true,
          sortOrder: sortedInfo.columnKey === "patientName" && sortedInfo.order
        },
        {
          title: "性别",
          dataIndex: "gender",
          customRender: (text, row, index) => {
            switch (text) {
              case "0":
                return "男";
              case "1":
                return "女";
              case "2":
                return "保密";
              default:
                return text;
            }
          },
          filters: [
            { text: "男", value: "0" },
            { text: "女", value: "1" },
            { text: "保密", value: "2" }
          ]
        },
        {
          title: "年龄",
          dataIndex: "age"
        },
        {
          title: "单位",
          dataIndex: "ageUnit"
        },
        {
          title: "入院时间",
          dataIndex: "admissionDate"
        },
        {
          title: "Action",
          key: "operation",
          scopedSlots: { customRender: "operation" }
        }
      ];
    }
  },
  mounted() {
    this.fetch();
  },
  methods: {
    onSelectChange(selectedRowKeys) {
      this.selectedRowKeys = selectedRowKeys;
    },
    toggleAdvanced() {
      this.advanced = !this.advanced;
      if (!this.advanced) {
        this.queryParams.createTimeFrom = "";
        this.queryParams.createTimeTo = "";
      }
    },
    view(record) {
      this.userInfo.data = record;
      this.userInfo.visiable = true;
    },
    add() {
      this.userAdd.visiable = true;
    },
    handleUserAddClose() {
      this.userAdd.visiable = false;
    },
    handleUserAddSuccess() {
      this.userAdd.visiable = false;
      this.$message.success("新增用户成功，初始密码为1234qwer");
      this.search();
    },
    edit(record) {
      this.$refs.userEdit.setFormValues(record);
      this.userEdit.visiable = true;
    },
    handleUserEditClose() {
      this.userEdit.visiable = false;
    },
    handleUserEditSuccess() {
      this.userEdit.visiable = false;
      this.$message.success("修改用户成功");
      this.search();
    },
    handleUserInfoClose() {
      console.log("userinfo.close");
      this.userInfo.visiable = false;
    },
    handleDeptChange(value) {
      this.queryParams.deptId = value || "";
    },
    handleDateChange(value) {
      if (value) {
        this.queryParams.createTimeFrom = value[0];
        this.queryParams.createTimeTo = value[1];
      }
    },
    batchDelete() {
      if (!this.selectedRowKeys.length) {
        this.$message.warning("请选择需要删除的记录");
        return;
      }
      let that = this;
      this.$confirm({
        title: "确定删除所选中的记录?",
        content: "当您点击确定按钮后，这些记录将会被彻底删除",
        centered: true,
        onOk() {
          let userIds = [];
          for (let key of that.selectedRowKeys) {
            userIds.push(that.dataSource[key].userId);
          }
          that.$delete("user/" + userIds.join(",")).then(() => {
            that.$message.success("删除成功");
            that.selectedRowKeys = [];
            that.search();
          });
        },
        onCancel() {
          that.selectedRowKeys = [];
        }
      });
    },
    resetPassword() {
      if (!this.selectedRowKeys.length) {
        this.$message.warning("请选择需要重置密码的用户");
        return;
      }
      let that = this;
      this.$confirm({
        title: "确定重置选中用户密码?",
        content: "当您点击确定按钮后，这些用户的密码将会重置为1234qwer",
        centered: true,
        onOk() {
          let usernames = [];
          for (let key of that.selectedRowKeys) {
            usernames.push(that.dataSource[key].username);
          }
          that
            .$put("user/password/reset", {
              usernames: usernames.join(",")
            })
            .then(() => {
              that.$message.success("重置用户密码成功");
              that.selectedRowKeys = [];
            });
        },
        onCancel() {
          that.selectedRowKeys = [];
        }
      });
    },
    exportExcel() {
      let { sortedInfo, filteredInfo } = this;
      let sortField, sortOrder;
      // 获取当前列的排序和列的过滤规则
      if (sortedInfo) {
        sortField = sortedInfo.field;
        sortOrder = sortedInfo.order;
      }
      this.$export("user/excel", {
        sortField: sortField,
        sortOrder: sortOrder,
        ...this.queryParams,
        ...filteredInfo
      });
    },
    search() {
      let { sortedInfo, filteredInfo } = this;
      let sortField, sortOrder;
      // 获取当前列的排序和列的过滤规则
      if (sortedInfo) {
        sortField = sortedInfo.field;
        sortOrder = sortedInfo.order;
      }
      this.fetch({
        sortField: sortField,
        sortOrder: sortOrder,
        ...this.queryParams,
        ...filteredInfo
      });
    },
    reset() {
      // 取消选中
      this.selectedRowKeys = [];
      // 重置分页
      this.$refs.TableInfo.pagination.current = this.pagination.defaultCurrent;
      if (this.paginationInfo) {
        this.paginationInfo.current = this.pagination.defaultCurrent;
        this.paginationInfo.pageSize = this.pagination.defaultPageSize;
      }
      // 重置列过滤器规则
      this.filteredInfo = null;
      // 重置列排序规则
      this.sortedInfo = null;
      // 重置查询参数
      this.queryParams = {};
      // 清空部门树选择
      this.$refs.deptTree.reset();
      // 清空时间选择
      if (this.advanced) {
        this.$refs.createTime.reset();
      }
      this.fetch();
    },
    handleTableChange(pagination, filters, sorter) {
      // 将这三个参数赋值给Vue data，用于后续使用
      this.paginationInfo = pagination;
      this.filteredInfo = filters;
      this.sortedInfo = sorter;

      this.userInfo.visiable = false;
      this.fetch({
        sortField: sorter.field,
        sortOrder: sorter.order,
        ...this.queryParams,
        ...filters
      });
    },
    fetch(params = {}) {
      // 显示loading
      this.loading = true;
      if (this.paginationInfo) {
        // 如果分页信息不为空，则设置表格当前第几页，每页条数，并设置查询分页参数
        this.$refs.TableInfo.pagination.current = this.paginationInfo.current;
        this.$refs.TableInfo.pagination.pageSize = this.paginationInfo.pageSize;
        params.pageSize = this.paginationInfo.pageSize;
        params.pageNum = this.paginationInfo.current;
      } else {
        // 如果分页信息为空，则设置为默认值
        params.pageSize = this.pagination.defaultPageSize;
        params.pageNum = this.pagination.defaultCurrent;
      }
      this.$get("infection", {
        ...params
      }).then(r => {
        let data = r.data;
        const pagination = { ...this.pagination };
        pagination.total = data.total;
        this.dataSource = data.rows;
        this.pagination = pagination;

        // 数据加载完毕，关闭loading
        this.loading = false;
      });
    },

    addReport(pid, patientName) {
      console.log(patientName)
      this.$router.push({ path: '/infection/report/add', query: { pid: pid, userName: patientName}})
    },
  }
};
/* eslint-disable */
</script>

<style lang="less" scoped>
@import "../../../static/less/Common";
</style>

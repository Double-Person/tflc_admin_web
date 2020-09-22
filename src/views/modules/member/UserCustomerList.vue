<template>
  <a-card :bordered="false">

    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24">

          <a-col :md="6" :sm="8">
            <a-form-item label="手机号">
              <a-input placeholder="请输入手机号" v-model="queryParam.username"></a-input>
            </a-form-item>
          </a-col>

          <!--</template>-->
          <a-col :md="6" :sm="8">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              <a @click="handleToggleSearch" style="margin-left: 8px">
                {{ toggleSearchStatus ? '收起' : '展开' }}
                <a-icon :type="toggleSearchStatus ? 'up' : 'down'"/>
              </a>
            </span>
          </a-col>

        </a-row>
      </a-form>
    </div>


    <!-- table区域-begin -->
    <div>
      <!--<div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">-->
        <!--<i class="anticon anticon-info-circle ant-alert-icon"></i> 数据列表-->
      <!--</div>-->

      <a-table
        ref="table"
        size="middle"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"

        @change="handleTableChange">
        <!--:rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"-->
        <!--<span slot="action" slot-scope="text, record">-->
          <!--<a @click="handleEdit(record)">编辑</a>-->

          <!--<a-divider type="vertical"/>-->
          <!--<a-dropdown>-->
            <!--<a class="ant-dropdown-link">更多 <a-icon type="down"/></a>-->
            <!--<a-menu slot="overlay">-->
              <!--<a-menu-item>-->
                <!--<a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">-->
                  <!--<a>删除</a>-->
                <!--</a-popconfirm>-->
              <!--</a-menu-item>-->
            <!--</a-menu>-->
          <!--</a-dropdown>-->
        <!--</span>-->

      </a-table>
    </div>
    <!-- table区域-end -->

    <!-- 表单区域 -->
    <userCustomer-modal ref="modalForm" @ok="modalFormOk"></userCustomer-modal>
  </a-card>
</template>

<script>
  import UserCustomerModal from './UserCustomerModal'
  import {JeecgListMixin} from '@/mixins/JeecgListMixin'
  import {userCustomerList} from "@/api/api";

  export default {
    name: "UserCustomerList",
    mixins: [JeecgListMixin],
    components: {
      UserCustomerModal
    },
    data() {
      return {
        description: '客户信息管理页面',
        // 表头
        columns: [
          {
            title: '手机号',
            align: "center",
            dataIndex: 'username'
          },
          {
            title: '注册时间',
            align: "center",
            dataIndex: 'createTime'
          },
          // {
          //   title: '操作',
          //   dataIndex: 'action',
          //   align: "center",
          //   scopedSlots: {customRender: 'action'},
          // }
        ],
        url: {
          list: "/admin/user_customer/list",
          delete: "/bp/userCustomer/delete",
          deleteBatch: "/bp/userCustomer/deleteBatch",
          exportXlsUrl: "bp/userCustomer/exportXls",
          importExcelUrl: "bp/userCustomer/importExcel",
        },
      }
    },
    computed: {
      importExcelUrl: function () {
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
      }
    },
    mounted(){
      this.getUserCustomerList()
    },
    methods: {
      getUserCustomerList(){
        userCustomerList().then(res=>{
          console.log(res);
        })
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>
<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24">
          <a-col :md="6" :sm="8">
            <a-form-item label="标题">
              <a-input placeholder="请输入标题" v-model="queryParam.title"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="类型">
              <a-select v-model="queryParam.typeCode" allowClear placeholder="请选择">
                <a-select-option
                  v-for="(item, index) in type"
                  :value="item.itemValue"
                >{{item.itemText}}</a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
            <a-col :md="6" :sm="8">
              <a-form-item label="有效性">
                <a-select v-model="queryParam.delFlag" allowClear placeholder="请选择">
                  <a-select-option
                    v-for="(item, index) in effective"
                    :value="item.value"
                  >{{item.label}}</a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
            <a-col :md="6" :sm="8">
              <a-form-item label="创建时间">
                <j-date
                  v-model="queryParam.createTime"
                  :showTime="false"
                  style="width:45%"
                  placeholder="请选择开始时间"
                ></j-date>
                <span style="width: 10px;">~</span>
                <j-date
                  v-model="queryParam.updateTime"
                  :showTime="false"
                  style="width:45%"
                  placeholder="请选择结束时间"
                ></j-date>
              </a-form-item>
            </a-col>
          </template>
          <a-col :md="6" :sm="8">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button
                type="primary"
                @click="searchReset"
                icon="reload"
                style="margin-left: 8px"
              >重置</a-button>
              <a @click="handleToggleSearch" style="margin-left: 8px">
                {{ toggleSearchStatus ? '收起' : '展开' }}
                <a-icon :type="toggleSearchStatus ? 'up' : 'down'" />
              </a>
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <!-- <a-button type="primary" icon="download" @click="handleExportXls('广告')">导出</a-button>
      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel"><a-icon type="delete"/>删除</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down" /></a-button>
      </a-dropdown>-->
    </div>

    <!-- table区域-begin -->
    <div>
      <!-- <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div> 
      :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
      -->

      <a-table
        ref="table"
        size="middle"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        @change="handleTableChange"
      >
        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">
              更多
              <a-icon type="down" />
            </a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => delOrResClick(record.id)" v-if="record.delFlag">
                  <a>删除</a>
                </a-popconfirm>
                <a-popconfirm title="确定恢复吗?" @confirm="() => delOrResClick(record.id)" v-else>
                  <a>恢复</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>
      </a-table>
    </div>
    <!-- table区域-end -->

    <!-- 表单区域 -->
    <sysAdvertise-modal ref="modalForm" @ok="modalFormOk"></sysAdvertise-modal>
  </a-card>
</template>

<script>
import SysAdvertiseModal from './SysAdvertiseModal'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import JDate from '@/components/jeecg/JDate'
import * as Constant from '@/utils/constant'
import { getAction, deleteAction } from '@/api/manage'

export default {
  name: 'SysAdvertiseList',
  mixins: [JeecgListMixin],
  components: {
    SysAdvertiseModal,
    JDate
  },
  data() {
    return {
      description: '广告管理页面',
      // 表头
      columns: [
        {
          title: '标题',
          align: 'center',
          dataIndex: 'title'
        },
        {
          title: '类型',
          align: 'center',
          dataIndex: 'typeCode_dictText'
        },
        {
          title: '有效性',
          align: 'center',
          // dataIndex: 'delFlag'
          customRender: (text, record) => {
            return record.delFlag ? '有效' : '无效'
          }
        },
        {
          title: '描述',
          align: 'center',
          dataIndex: 'description'
        },
        {
          title: '添加时间',
          align: 'center',
          dataIndex: 'createTime'
        },
        {
          title: '操作',
          dataIndex: 'action',
          align: 'center',
          scopedSlots: { customRender: 'action' }
        }
      ],
      url: {
        list: '/system/sysAdvertise/list',
        delete: '/system/sysAdvertise/delete',
        deleteBatch: '/system/sysAdvertise/deleteBatch',
        exportXlsUrl: 'system/sysAdvertise/exportXls',
        importExcelUrl: 'system/sysAdvertise/importExcel'
      },
      type: [],
      effective: Constant.EFFECTIVE
    }
  },
  computed: {
    importExcelUrl: function() {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
    }
  },
  created() {
    this.searchType()
  },
  methods: {
    // 查询广告类型
    searchType() {
      const url = '/sys/dictItem/list'
      const params = {}
      params.dictId = 'a3d1b1ee6706c63271b5637bea47dd67'
      params.field = 'id,,itemText,itemValue,action'
      params.pageNo = '1'
      params.pageSize = '50'
      return getAction(url, params).then(res => {
        if (res.success) {
          let result = res.result
          // console.log(result)
          this.type = result.records
        }
      })
    },
    // 删除还是恢复
    delOrResClick(id) {
      const url = '/system/sysAdvertise/delete'
      const params = {}
      params.id = id
      return deleteAction(url, params).then(res => {
        if (res.success) {
          // console.log(res)
          this.searchQuery()
        } else {
          this.$message.error(res.message)
        }
      })
    }
  }
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>
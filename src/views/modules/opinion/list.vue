<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24">
          <a-col :md="6" :sm="8">
            <a-form-item label="反馈编号">
              <a-input placeholder="请输入反馈编号" v-model="queryParam.id"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="联系方式">
              <a-input placeholder="请输入联系方式" v-model="queryParam.phone"></a-input>
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
            <a-col :md="6" :sm="8">
              <a-form-item label="反馈来源">
                <a-select v-model="queryParam.source" allowClear placeholder="请选择">
                  <a-select-option value>全部</a-select-option>
                  <a-select-option value="1">Android</a-select-option>
                  <a-select-option value="2">IOS</a-select-option>
                  <a-select-option value="3">PC</a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
            <a-col :md="6" :sm="8">
              <a-form-item label="处理状态">
                <a-select v-model="queryParam.status" allowClear placeholder="请选择">
                  <a-select-option value>全部</a-select-option>
                  <a-select-option value="0">待处理</a-select-option>
                  <a-select-option value="1">已处理</a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
            <a-col :md="6" :sm="8">
              <a-form-item label="反馈时间">
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

    <!-- table区域-begin -->
    <div>
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
          <span class="view" @click="viewClick(record.id)">查看</span>

          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">
              更多
              <a-icon type="down" />
            </a>
            <a-menu slot="overlay">
              <a-menu-item v-if="record.status===0">
                <!-- <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <span>删除</span>
                </a-popconfirm>-->
                <p class="deal" @click="() => dealClick(record.id)">处理</p>
              </a-menu-item>
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <span>删除</span>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>
      </a-table>
    </div>
    <!-- model区 -->
    <div class="opinion-modal">
      <a-modal title="查看" v-model="viewVisible" class="viewModal">
        <div class="viewDialog-cont">
          <div class="viewDialog-item">
            <div class="viewDialog-tit">反馈人：</div>
            <div class="viewDialog-content">{{viewData.userName}}</div>
          </div>
          <div class="viewDialog-item">
            <div class="viewDialog-tit">反馈来源：</div>
            <div
              class="viewDialog-content"
            >{{viewData.source===1?'Android':viewData.source===2?'IOS':'PC'}}</div>
          </div>
          <div class="viewDialog-item">
            <div class="viewDialog-tit">反馈时间：</div>
            <div class="viewDialog-content">{{viewData.createTime}}</div>
          </div>
          <div class="viewDialog-item">
            <div class="viewDialog-tit">反馈内容：</div>
            <div class="viewDialog-content">{{viewData.content || '无'}}</div>
          </div>
          <div class="viewDialog-item">
            <div class="viewDialog-tit">图片：</div>
            <div class="viewDialog-content" v-if="feedBackImg.length<1">{{'无'}}</div>
            <div class="viewDialog-content viewDialog-img" v-else>
              <ul>
                <li class="img-item" v-for="(item, index) in feedBackImg">
                  <img :src="item" :preview="[item]" />
                </li>
              </ul>
            </div>
          </div>
          <div class="viewDialog-item">
            <div class="viewDialog-tit">处理结果：</div>
            <div class="viewDialog-content">{{viewData.solveContent?viewData.solveContent:'待处理'}}</div>
          </div>
          <div class="viewDialog-item">
            <div class="viewDialog-tit">处理时间：</div>
            <div class="viewDialog-content">{{viewData.solveTime||'无'}}</div>
          </div>
        </div>
      </a-modal>
      <a-modal title="处理" v-model="dealVisible" @ok="dealOk" class="viewModal">
        <div class="viewDialog-cont">
          <div class="viewDialog-item">
            <div class="viewDialog-tit">处理结果：</div>
            <div class="viewDialog-content">
              <a-textarea
                v-model="solveInput"
                placeholder="请输入处理结果"
                :autosize="{ minRows: 2, maxRows: 3 }"
              />
            </div>
          </div>
        </div>
      </a-modal>
    </div>
  </a-card>
</template>

<script>
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import JDate from '@/components/jeecg/JDate'
import * as Constant from '@/utils/constant'
import { getAction, postAction } from '@/api/manage'

export default {
  name: 'SysArticleList',
  mixins: [JeecgListMixin],
  components: {
    JDate
  },
  data() {
    return {
      description: '文章管理页面',
      // 表头
      columns: [
        {
          title: '反馈编号',
          align: 'center',
          dataIndex: 'id'
        },
        {
          title: '反馈内容',
          align: 'center',
          dataIndex: 'content'
        },
        {
          title: '反馈来源',
          align: 'center',
          // dataIndex: 'isRecommend'
          customRender: (text, record) => {
            const source = String(record.source)
            if (source === '1') {
              return 'Android'
            } else if (source === '2') {
              return 'IOS'
            } else if (source === '3') {
              return 'PC'
            } else {
              return ''
            }
          }
        },
        // {
        //   title: '反馈人账号',
        //   align: 'center',
        //   dataIndex: 'username'
        //   // customRender: (text, record) => {
        //   //   return String(record.delFlag) === '0' ? '有效' : '无效'
        //   // }
        // },
        // {
        //   title: '联系方式',
        //   align: 'center',
        //   dataIndex: 'phone'
        // },
        {
          title: '反馈人',
          align: 'center',
          dataIndex: 'userId'
        },
        {
          title: '反馈时间',
          align: 'center',
          dataIndex: 'createTime'
        },
        {
          title: '反馈状态',
          align: 'center',
          // dataIndex: 'status'
          customRender: (text, record) => {
            return String(record.status) === '1' ? '已处理' : '待处理'
          }
        },
        {
          title: '操作',
          dataIndex: 'action',
          align: 'center',
          scopedSlots: { customRender: 'action' }
        }
      ],
      url: {
        list: '/system/sysFeedback/list',
        delete: '/system/sysFeedback/delete',
        deleteBatch: '/system/sysArticle/deleteBatch',
        exportXlsUrl: 'system/sysArticle/exportXls',
        importExcelUrl: 'system/sysArticle/importExcel'
      },
      viewVisible: false,
      dealVisible: false,
      viewData: {},
      feedBackImg: [],
      solveInput: '',
      dealId: ''
    }
  },
  computed: {
    importExcelUrl: function() {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
    }
  },
  methods: {
    // 查看点击
    viewClick(id) {
      this.feedBackImg = []

      let url = '/system/sysFeedback/queryById'
      let params = {
        id: id
      }
      return getAction(url, params).then(res => {
        if (res.success) {
          let result = res.result
          console.log(result)
          this.viewData = result
          this.viewVisible = true
          this.feedBackImg = JSON.parse(result.img)
        }
      })
    },
    // 处理点击
    dealClick(id) {
      this.dealVisible = true
      this.dealId = id
    },
    // 处理ok
    dealOk() {
      if (!this.solveInput) {
        this.dealVisible = false
        return
      }
      let url = '/system/sysFeedback/solve'
      let data = this.$qs.stringify({
        id: this.dealId,
        content: this.solveInput
      })
      return postAction(url, data).then(res => {
        if (res.success) {
          let result = res.result
          // console.log(result)
          this.dealVisible = false
          this.searchQuery()
        }
      })
    }
  }
}
</script>
<style scoped>
@import '~@assets/less/common.less';
@import './_list.scss';
</style>
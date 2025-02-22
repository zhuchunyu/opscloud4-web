<template>
  <d2-container>
    <template>
      <div>
        <h1>{{ title }}</h1>
      </div>
      <el-row style="margin-bottom: 5px" :gutter="24">
        <el-input v-model="queryParam.queryName" placeholder="输入关键字模糊查询"/>
        <el-select
          v-model="queryParam.tagId" filterable clearable remote reserve-keyword
          placeholder="请输入关键词搜索标签" :remote-method="getTag" @change="fetchData">
          <el-option
            v-for="item in tagOptions"
            :key="item.id"
            :label="item.tagKey"
            :value="item.id">
          </el-option>
        </el-select>
        <el-button @click="fetchData" style="margin-left: 5px">查询</el-button>
        <el-button style="margin-left: 5px" @click="handleAdd">新增</el-button>
      </el-row>
      <el-table :data="table.data" style="width: 100%" v-loading="table.loading">
        <el-table-column label="应用" width="200px">
          <template v-slot="scope">
            <el-row>
              <span>{{ scope.row.name }}</span>
              <el-button type="text" v-if="scope.row.document !== null" style="margin-left: 10px"
                         @click="handleDocRead(scope.row)"><i class="fab fa-creative-commons-share"></i>
              </el-button>
            </el-row>
            <el-row v-if="false">
              <el-tag disable-transitions type="primary" plain size="mini">{{ scope.row.applicationKey }}</el-tag>
            </el-row>
            <el-row>
              <b style="color: #9d9fa3">{{ scope.row.comment }}</b>
            </el-row>
            <!--Tags-->
            <business-tags :tags="scope.row.tags"></business-tags>
          </template>
        </el-table-column>
        <el-table-column prop="resourceMap" label="绑定资源" width="400px">
          <template v-slot="scope">
            <div v-for="(value,key) in scope.row.resourceMap" :key="key" :label="key" class="resDiv">
              <el-divider content-position="left"><b style="color: #9d9fa3">{{ key | getAppResText }}</b></el-divider>
              <div v-for="item in value" :key="item.id">
                <el-tooltip effect="dark" :content="item.comment" placement="top-start"
                            :disabled="!item.comment">
                  <el-tag size="mini" style="margin-left: 5px;margin-bottom: 5px">
                    <span v-if="item.instance !== null">{{ item.instance.instanceName }}/</span>
                    {{ item.name }}
                  </el-tag>
                </el-tooltip>
              </div>
            </div>
          </template>
        </el-table-column>
        <el-table-column prop="users" label="授权用户" width="350">
          <template v-slot="scope">
            <users-tag :users="scope.row.users"></users-tag>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="200">
          <template v-slot="scope">
            <el-button type="primary" plain size="mini" @click="handleRowEdit(scope.row)">编辑</el-button>
            <el-button type="primary" plain size="mini" @click="handleRowTagEdit(scope.row)">标签</el-button>
            <el-button type="danger" plain size="mini" @click="handleRowDel(scope.row)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
      <pagination :pagination="table.pagination" @paginationCurrentChange="paginationCurrentChange"
                  @handleSizeChange="handleSizeChange"></pagination>
      <application-editor ref="applicationEditor" :formStatus="formStatus.application"
                          @close="fetchData"></application-editor>
      <business-tag-editor ref="businessTagEditor" :business-type="businessType" :business-id="instance.id"
                           :form-status="formStatus.businessTag" @close="fetchData"></business-tag-editor>
      <business-doc-reader :form-status="formStatus.businessDoc" ref="businessDocReader"></business-doc-reader>
    </template>
  </d2-container>
</template>

<script>

import Pagination from '@/components/opscloud/common/page/Pagination'
import { DELETE_APPLICATION, QUERY_APPLICATION_PAGE } from '@/api/modules/application/application.api'
import ApplicationEditor from '@/components/opscloud/application/ApplicationEditor'
import AppDsInstanceAssetType from '@/components/opscloud/common/enums/application.ds.instance.asset.type'
import UsersTag from '../../components/opscloud/common/tag/UsersTag'
import BusinessType from '@/components/opscloud/common/enums/business.type'
import BusinessTagEditor from '@/components/opscloud/common/tag/BusinessTagEditor'
import BusinessDocReader from '@/components/opscloud/business/BusinessDocReader'
import { QUERY_TAG_PAGE } from '@/api/modules/tag/tag.api'
import BusinessTags from '@/components/opscloud/common/tag/BusinessTags.vue'

export default {
  name: 'ApplicationTable',
  data () {
    return {
      title: '应用管理',
      instance: {
        id: ''
      },
      table: {
        data: [],
        loading: false,
        pagination: {
          currentPage: 1,
          pageSize: 10,
          total: 0
        }
      },
      options: {
        stripe: true
      },
      businessType: BusinessType.APPLICATION,
      queryParam: {
        queryName: '',
        tagId: ''
      },
      formStatus: {
        application: {
          visible: false,
          operationType: true,
          addTitle: '新增应用配置',
          updateTitle: '更新应用配置'
        },
        businessTag: {
          visible: false,
          title: '编辑应用标签'
        },
        businessDoc: {
          visible: false,
          title: '应用文档'
        }
      },
      tagOptions: []
    }
  },
  filters: {
    getAppResText (value) {
      switch (value) {
        case AppDsInstanceAssetType.GITLAB.GITLAB_GROUP:
          return 'GitLab群组'
        case AppDsInstanceAssetType.GITLAB.GITLAB_PROJECT:
          return 'GitLab项目'
        case AppDsInstanceAssetType.KUBERNETES.KUBERNETES_DEPLOYMENT:
          return 'Kubernetes无状态'
        case 'SERVERGROUP':
          return '服务器组'
        case 'SERVER':
          return '服务器'
        default:
          return value
      }
    }
  },
  computed: {},
  mounted () {
    this.getTag('')
    this.fetchData()
  },
  components: {
    Pagination,
    ApplicationEditor,
    UsersTag,
    BusinessTagEditor,
    BusinessDocReader,
    BusinessTags
  },
  methods: {
    paginationCurrentChange (currentPage) {
      this.table.pagination.currentPage = currentPage
      this.fetchData()
    },
    handleSizeChange (size) {
      this.table.pagination.pageSize = size
      this.fetchData()
    },
    handleDocRead (row) {
      this.$refs.businessDocReader.initData(Object.assign({}, row.document))
      this.formStatus.businessDoc.visible = true
    },
    getTag (name) {
      const requestBody = {
        tagKey: name,
        businessType: this.businessType,
        append: true,
        page: 1,
        length: 20
      }
      QUERY_TAG_PAGE(requestBody)
        .then(res => {
          this.tagOptions = res.body.data
        })
    },
    handleAdd () {
      this.formStatus.application.operationType = true
      const application = {
        id: '',
        name: '',
        applicationKey: '',
        applicationType: 0,
        isActive: true,
        comment: ''
      }
      this.$refs.applicationEditor.initData(application)
      this.formStatus.application.visible = true
    },
    handleRowTagEdit (row) {
      this.instance.id = row.id
      const businessTags = {
        tagIds: row.tags !== null ? row.tags.map(e => e.id) : []
      }
      this.$refs.businessTagEditor.initData(businessTags)
      this.formStatus.businessTag.visible = true
    },
    handleRowEdit (row) {
      this.formStatus.application.operationType = false
      this.formStatus.application.visible = true
      this.$refs.applicationEditor.initData(Object.assign({}, row))
    },
    handleRowDel (row) {
      this.$confirm('此操作将删除当前配置?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        DELETE_APPLICATION({ id: row.id })
          .then(res => {
            this.fetchData()
            this.$message.success('删除成功!')
          })
      }).catch(() => {
        this.$message.info('已取消删除')
      })
    },
    fetchData () {
      this.loading = true
      const requestBody = {
        ...this.queryParam,
        extend: 1,
        page: this.table.pagination.currentPage,
        length: this.table.pagination.pageSize
      }
      QUERY_APPLICATION_PAGE(requestBody)
        .then(res => {
          this.table.data = res.body.data
          this.table.pagination.total = res.body.totalNum
          this.table.loading = false
        })
    },
    onCopy (e) {
      this.$message.success('内容已复制到剪切板！')
    },
    onError (e) {
      this.$message.error('抱歉，复制失败！')
    }
  }
}
</script>

<style lang="less" scoped>

.el-input {
  display: inline-block;
  max-width: 200px;
}

.el-select {
  margin-left: 5px;
}

.el-button {
  margin-left: 5px;
}

.nameCopy i {
  display: none;
}

.nameCopy:hover i {
  display: inline;
}

.el-divider--horizontal {
  display: block;
  height: 1px;
  width: 100%;
  margin: 12px 0;
}
</style>

<template>
  <div>
    <el-row :gutter="24" style="margin-bottom: 5px; margin-left: 0px;">
      <el-input v-model="queryParam.queryName" @change="fetchData" placeholder="输入关键字模糊查询"/>
      <el-select v-model="queryParam.envType" clearable filterable
                 remote reserve-keyword placeholder="输入关键词搜索环境" :remote-method="getEnv">
        <el-option
          v-for="item in envOptions"
          :key="item.id"
          :label="item.envName"
          :value="item.envType">
          <select-item :name="item.envName" :comment="item.comment"></select-item>
        </el-option>
      </el-select>
      <el-button @click="fetchData">查询</el-button>
      <el-button @click="handleAdd" class="button">新增</el-button>
      <el-button @click="handleScan" class="button">扫描</el-button>
    </el-row>
    <el-table :data="table.data" style="width: 100%" v-loading="table.loading">
      <el-table-column prop="name" label="名称" width="200"></el-table-column>
      <el-table-column prop="env" label="环境" width="80">
        <template slot-scope="scope">
          <env-tag :env="scope.row.env"></env-tag>
        </template>
      </el-table-column>
      <el-table-column prop="asset" label="关联资产" width="200">
        <template slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.asset === null">未创建</el-tag>
          <el-tag size="mini" v-if="scope.row.asset !== null" type="success">{{ scope.row.asset.assetKey }}</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="template" label="模板" width="250">
        <template slot-scope="scope">
          <span>{{ scope.row.template.name }}</span>
          <div>
            <el-tag size="mini">{{ scope.row.template.templateKey }}</el-tag>
          </div>
        </template>
      </el-table-column>
      <el-table-column prop="vars" label="变量">
        <template slot-scope="scope">
          <my-highlight :code="scope.row.vars" :lang="scope.row.templateType"></my-highlight>
        </template>
      </el-table-column>
      <el-table-column label="操作" width="280">
        <template slot-scope="scope">
          <slot name="operation" :row="scope.row"></slot>
          <el-button :type="scope.row.businessId === 0 ? 'success' : 'warning'" plain size="mini"
                     @click="handleRowCreate(scope.row)"
                     :loading="scope.row.creating"><span v-text="scope.row.businessId === 0 ? '创建' : '更新'"></span>
          </el-button>
          <el-button type="primary" plain size="mini"
                     @click="handleRowEdit(scope.row)">编辑
          </el-button>
          <el-button type="danger" plain size="mini"
                     @click="handleRowDel(scope.row)">删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <pagination :pagination="table.pagination" @paginationCurrentChange="paginationCurrentChange"
                @handleSizeChange="handleSizeChange"></pagination>
    <business-template-editor :formStatus="formStatus.businessTemplate"
                              :instanceTypeOptions="instanceTypeOptions"
                              ref="businessTemplateEditor" @close="fetchData"></business-template-editor>
  </div>
</template>

<script>

import {
  QUERY_BUSINESS_TEMPLATE_PAGE, CREATE_ASSET_BY_BUSINESS_TEMPLATE_ID,
  SCAN_BUSINESS_TEMPLATE_BY_INSTANCE,
  DELETE_BUSINESS_TEMPLATE_BY_ID
} from '@/api/modules/template/business.template.api.js'
import Pagination from '../../common/page/Pagination'
import BusinessType from '@/components/opscloud/common/enums/business.type.js'
import BusinessTemplateEditor from '@/components/opscloud/datasource/template/BusinessTemplateEditor'
import { QUERY_ENV_PAGE } from '@/api/modules/sys/sys.env.api'
import SelectItem from '@/components/opscloud/common/SelectItem'
import EnvTag from '@/components/opscloud/common/tag/EnvTag'
import MyHighlight from '@/components/opscloud/common/MyHighlight'

const instanceTypeOptions = [{
  value: 'KUBERNETES',
  label: 'KUBERNETES'
}]

export default {
  name: 'KubernetesTemplateTable',
  props: {
    instanceUuid: {
      type: String,
      required: true
    }
  },
  data () {
    return {
      table: {
        data: [],
        loading: false,
        pagination: {
          currentPage: 1,
          pageSize: 10,
          total: 0
        }
      },
      formStatus: {
        businessTemplate: {
          visible: false,
          labelWidth: '150px',
          operationType: true,
          addTitle: '新增业务模板配置',
          updateTitle: '更新业务模板配置'
        }
      },
      envOptions: [],
      businessId: '',
      businessType: BusinessType.ASSET,
      instanceTypeOptions: instanceTypeOptions,
      queryParam: {
        envType: '',
        queryName: '',
        extend: true
      }
    }
  },
  computed: {},
  mounted () {
    this.getEnv('')
  },
  components: {
    EnvTag,
    SelectItem,
    BusinessTemplateEditor,
    Pagination,
    MyHighlight
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
    getEnv (name) {
      const requestBody = {
        envName: name,
        page: 1,
        length: 20
      }
      QUERY_ENV_PAGE(requestBody)
        .then(res => {
          this.envOptions = res.body.data
        })
    },
    handleRowCreate (row) {
      this.$confirm('是否从模板配置创建/更新资产?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        for (let i = 0; i < this.table.data.length; i++) {
          if (this.table.data[i].id === row.id) {
            this.table.data[i].creating = true
            break
          }
        }
        row.creating = true
        CREATE_ASSET_BY_BUSINESS_TEMPLATE_ID(row.id).then(() => {
          this.$message.success('创建成功!')
          this.fetchData()
        })
      }).catch(() => {
        this.$message.info('已取消!')
      })
    },
    handleRowDel (row) {
      this.$confirm('此操作将删除当前配置?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        DELETE_BUSINESS_TEMPLATE_BY_ID(row.id).then(() => {
          this.$message.success('删除成功!')
          this.fetchData()
        })
      }).catch(() => {
        this.$message.info('已取消删除!')
      })
    },
    handleAdd () {
      const businessTemplate = {
        id: '',
        name: '',
        instanceUuid: this.instanceUuid,
        businessType: 5, // Asset
        businessId: 0,
        templateId: '',
        envType: this.queryParam.envType === '' ? 0 : this.queryParam.envType,
        content: '',
        vars: '',
        comment: ''
      }
      this.$refs.businessTemplateEditor.initData(businessTemplate, 'KUBERNETES')
      this.formStatus.businessTemplate.operationType = true
      this.formStatus.businessTemplate.visible = true
    },
    handleScan () {
      SCAN_BUSINESS_TEMPLATE_BY_INSTANCE(this.instanceUuid)
        .then(res => {
          this.$message.success('后台执行中!')
        })
    },
    handleRowEdit (row) {
      this.$refs.businessTemplateEditor.initData(row, row.template.instanceType)
      this.formStatus.businessTemplate.operationType = false
      this.formStatus.businessTemplate.visible = true
    },
    fetchData () {
      this.table.loading = true
      const requestBody = {
        instanceUuid: this.instanceUuid,
        ...this.queryParam,
        page: this.table.pagination.currentPage,
        length: this.table.pagination.pageSize
      }
      QUERY_BUSINESS_TEMPLATE_PAGE(requestBody)
        .then(res => {
          this.table.data = res.body.data
          this.table.pagination.total = res.body.totalNum
          this.table.loading = false
        })
    }
  }
}
</script>

<style scoped>
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

</style>

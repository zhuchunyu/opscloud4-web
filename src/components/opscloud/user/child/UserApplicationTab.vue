<template>
  <div>
    <el-row :gutter="24" style="margin-bottom: 5px; margin-left: 0px;">
      <el-select v-model.trim="applicationId" clearable filterable
                 remote reserve-keyword placeholder="选择要授权的应用" :remote-method="getApplication">
        <el-option
          v-for="item in applicationOptions"
          :key="item.id"
          :label="item.name"
          :value="item.id">
        </el-option>
      </el-select>
      <el-button type="primary" plain size="mini" @click="handleGrant()" :disabled="applicationId === ''">授权
      </el-button>
    </el-row>
    <el-divider></el-divider>
    <el-row :gutter="24" style="margin-bottom: 5px; margin-left: 0px;">
      <el-input v-model="queryParam.queryName" placeholder="名称"/>
      <el-button @click="fetchData">查询</el-button>
    </el-row>
    <el-table :data="table.data" style="width: 100%" v-loading="table.loading">
      <el-table-column prop="name" label="名称"></el-table-column>
      <el-table-column prop="userPermission.roleName" label="角色">
        <template v-slot="scope">
          <el-tag size="mini" :type=" scope.row.userPermission.permissionRole === 'admin' ? 'danger' :'info'">
            {{ scope.row.userPermission.permissionRole === 'admin' ? '管理员' : '普通用户' }}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="comment" label="描述"></el-table-column>
      <el-table-column label="操作" width="200">
        <template v-slot="scope">
          <el-button type="primary" plain size="mini" @click="handleRowSet(scope.row)">
            {{ scope.row.userPermission.permissionRole === 'admin' ? '降权' : '提权' }}
          </el-button>
          <el-button type="danger" plain size="mini" @click="handleRowRevoke(scope.row)">解除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <pagination :pagination="table.pagination" @paginationCurrentChange="paginationCurrentChange"
                @handleSizeChange="handleSizeChange"></pagination>
  </div>
</template>

<script>

import {
  QUERY_USER_BUSINESS_PERMISSION,
  GRANT_USER_BUSINESS_PERMISSION,
  REVOKE_USER_BUSINESS_PERMISSION,
  SET_USER_BUSINESS_PERMISSION
} from '@/api/modules/user/user.api.js'
import BusinessType from '@/components/opscloud/common/enums/business.type.js'
import Pagination from '../../common/page/Pagination'

export default {
  name: 'UserApplicationTab',
  props: ['user'],
  data () {
    return {
      labelWidth: '150px',
      businessType: BusinessType,
      table: {
        data: [],
        loading: false,
        pagination: {
          currentPage: 1,
          pageSize: 10,
          total: 0
        }
      },
      queryParam: {
        queryName: '',
        extend: true,
        authorized: true
      },
      applicationId: '',
      applicationOptions: []
    }
  },
  mounted () {
  },
  components: {
    Pagination
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
    init () {
      this.getApplication('')
      this.fetchData()
    },
    getApplication (name) {
      const requestBody = {
        queryName: name,
        businessType: this.businessType.APPLICATION,
        extend: false,
        authorized: false,
        userId: this.user.id,
        page: 1,
        length: 20
      }
      QUERY_USER_BUSINESS_PERMISSION(requestBody)
        .then(res => {
          this.applicationOptions = res.body.data
        })
    },
    handleRowSet (row) {
      SET_USER_BUSINESS_PERMISSION({ id: row.userPermission.id })
        .then(res => {
          this.$message.success('设置角色成功!')
          this.applicationId = ''
          this.fetchData()
        })
    },
    /**
     * 授权
     */
    handleGrant () {
      const requestBody = {
        userId: this.user.id,
        businessType: this.businessType.APPLICATION,
        businessId: this.applicationId
      }
      GRANT_USER_BUSINESS_PERMISSION(requestBody)
        .then(res => {
          this.$message.success('授权成功!')
          this.applicationId = ''
          this.getApplication('')
          this.fetchData()
        })
    },
    /**
     * 解除
     * @param row
     */
    handleRowRevoke (row) {
      const requestBody = {
        userId: this.user.id,
        businessType: this.businessType.APPLICATION,
        businessId: row.id
      }
      REVOKE_USER_BUSINESS_PERMISSION(requestBody)
        .then(res => {
          this.$message.success('解除成功!')
          this.getApplication('')
          this.fetchData()
        })
    },
    fetchData () {
      this.table.loading = true
      const requestBody = {
        ...this.queryParam,
        businessType: this.businessType.APPLICATION,
        userId: this.user.id,
        page: this.table.pagination.currentPage,
        length: this.table.pagination.pageSize
      }
      QUERY_USER_BUSINESS_PERMISSION(requestBody)
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

.el-select {
  display: inline-block;
  width: 250px;
}

.el-input {
  display: inline-block;
  max-width: 200px;
}

.el-button {
  margin-left: 5px;
}

</style>

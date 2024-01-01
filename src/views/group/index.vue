<template>
    <div class="app-container">
        <div class="filter-container">
            <el-input v-model="listQuery.group_name" placeholder="组名" style="width: 200px;" class="filter-item"
                @keyup.enter.native="handleFilter" />
            <el-button class="filter-item" style="margin-left:10px;" type="primary" icon="el-icon-search"
                @click="handleFilter">
                搜索
            </el-button>
            <el-button class="filter-item" style="margin-left: 10px; float: right;" type="primary" icon="el-icon-edit"
                @click="handleCreate">
                添加
            </el-button>
        </div>

        <el-table :key="tableKey" v-loading="listLoading" :data="list" border fit highlight-current-row style="width: 100%;"
            @sort-change="sortChange">
            <el-table-column label="序号" prop="id" sortable="custom" align="center" width="80"
                :class-name="getSortClass('id')">
                <template slot-scope="{row}">
                    <span>{{ row.id }}</span>
                </template>
            </el-table-column>

            <el-table-column label="组名" min-width="50px">
                <template slot-scope="{row}">
                    <span class="link-type" @click="handleUpdate(row)">{{ row.group_name }}</span>
                    <el-tag>{{ row.group_name }}</el-tag>
                </template>
            </el-table-column>

            <el-table-column label="更新时间" width="180px" align="center">
                <template slot-scope="{row}">
                    <span>{{ row.update_time }}</span>
                </template>
            </el-table-column>

            <el-table-column label="更新人" width="110px" align="center">
                <template slot-scope="{row}">
                    <span>{{ row.update_user }}</span>
                </template>
            </el-table-column>
            <el-table-column label="操作" align="center" width="230" class-name="small-padding fixed-width">
                <template slot-scope="{row,$index}">
                    <el-button type="primary" size="mini" @click="handleUpdate(row)">
                        编辑
                    </el-button>
                    <el-button v-if="row.status != 'deleted'" size="mini" type="danger" @click="handleDelete(row, $index)">
                        删除
                    </el-button>
                </template>
            </el-table-column>
        </el-table>

        <pagination v-show="total > 0" :total="total" :page.sync="listQuery.page" :limit.sync="listQuery.limit"
            @pagination="getList" />
    </div>
</template>
  
<script>
import Pagination from '@/components/Pagination' // secondary package based on el-pagination

export default {
    name: 'Group',
    components: { Pagination },
    filters: {
        statusFilter(status) {
            const statusMap = {
                published: 'success',
                draft: 'info',
                deleted: 'danger'
            }
            return statusMap[status]
        }
    },
    data() {
        return {
            tableKey: 0,
            list: [
                {
                    "id":1,
                    "group_name":"测试分组1",
                    "update_time": "2023-12-22 18:18:00",
                    "update_user": "admin"
                },
                {
                    "id": 2,
                    "group_name": "测试分组2",
                    "update_time": "2023-12-23 09:00:00",
                    "update_user": "admin"
                }
            ],
            total: 21,
            listLoading: false,
            listQuery: {
                page: 1,
                limit: 20,
                group_name: null,
                sort: '+id'
            },
            importanceOptions: [1, 2, 3],
            sortOptions: [{ label: 'ID Ascending', key: '+id' }, { label: 'ID Descending', key: '-id' }],
            statusOptions: ['published', 'draft', 'deleted'],
            showReviewer: false,
            temp: {
                id: undefined,
                importance: 1,
                remark: '',
                timestamp: new Date(),
                title: '',
                type: '',
                status: 'published'
            },
            dialogFormVisible: false,
            dialogStatus: '',
            textMap: {
                update: 'Edit',
                create: 'Create'
            },
            dialogPvVisible: false,
            pvData: [],
            rules: {
                type: [{ required: true, message: 'type is required', trigger: 'change' }],
                timestamp: [{ type: 'date', required: true, message: 'timestamp is required', trigger: 'change' }],
            },
            downloadLoading: false
        }
    },
    created() {
    },
    methods: {
        getList() { },
        handleFilter() {
            this.listQuery.page = 1
        },
        handleModifyStatus(row, status) {
            this.$message({
                message: '操作Success',
                type: 'success'
            })
            row.status = status
        },
        sortChange(data) {
            const { prop, order } = data
            if (prop === 'id') {
                this.sortByID(order)
            }
        },
        sortByID(order) {
            if (order === 'ascending') {
                this.listQuery.sort = '+id'
            } else {
                this.listQuery.sort = '-id'
            }
            this.handleFilter()
        },
        resetTemp() {
            this.temp = {
                id: undefined,
                importance: 1,
                remark: '',
                timestamp: new Date(),
                title: '',
                status: 'published',
                type: ''
            }
        },
        handleCreate() {
            this.resetTemp()
            this.dialogStatus = 'create'
            this.dialogFormVisible = true
            this.$nextTick(() => {
                this.$refs['dataForm'].clearValidate()
            })
        },
        createData() {
            this.$refs['dataForm'].validate((valid) => {
                if (valid) {
                    this.temp.id = parseInt(Math.random() * 100) + 1024 // mock a id
                    this.temp.author = 'vue-element-admin'
                    createArticle(this.temp).then(() => {
                        this.list.unshift(this.temp)
                        this.dialogFormVisible = false
                        this.$notify({
                            title: 'Success',
                            message: 'Created Successfully',
                            type: 'success',
                            duration: 2000
                        })
                    })
                }
            })
        },
        updateData() {
            this.$refs['dataForm'].validate((valid) => {
                if (valid) {
                    const tempData = Object.assign({}, this.temp)
                    tempData.timestamp = +new Date(tempData.timestamp) // change Thu Nov 30 2017 16:41:05 GMT+0800 (CST) to 1512031311464
                    updateArticle(tempData).then(() => {
                        const index = this.list.findIndex(v => v.id === this.temp.id)
                        this.list.splice(index, 1, this.temp)
                        this.dialogFormVisible = false
                        this.$notify({
                            title: 'Success',
                            message: 'Update Successfully',
                            type: 'success',
                            duration: 2000
                        })
                    })
                }
            })
        },
        handleUpdate(row, index) {

        },
        handleDelete(row, index) {
            this.$notify({
                title: 'Success',
                message: 'Delete Successfully',
                type: 'success',
                duration: 2000
            })
            this.list.splice(index, 1)
        },
        handleFetchPv(pv) {
            fetchPv(pv).then(response => {
                this.pvData = response.data.pvData
                this.dialogPvVisible = true
            })
        },
        formatJson(filterVal) {
            return this.list.map(v => filterVal.map(j => {
                if (j === 'timestamp') {
                    return parseTime(v[j])
                } else {
                    return v[j]
                }
            }))
        },
        getSortClass: function (key) {
            const sort = this.listQuery.sort
            return sort === `+${key}` ? 'ascending' : 'descending'
        }
    }
}
</script>

<style>
.app-container {
    padding: 20px;
}
.filter-container {
    padding-bottom: 20px;
}
</style>
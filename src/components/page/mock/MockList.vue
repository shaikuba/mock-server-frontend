<template>
    <div>
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item>
                    <i class="el-icon-lx-cascades"></i> Mock List
                </el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <div class="handle-box">
                <el-button
                        type="primary"
                        icon="el-icon-delete"
                        class="handle-del mr10"
                        @click="delAllSelection"
                >Delete
                </el-button>
                <el-select
                        v-model="query.criteria.requestMethod"
                        placeholder="Method"
                        class="handle-select mr10"
                >
                    <el-option label="Select" value=""></el-option>
                    <el-option v-for="(item, index) in requestMethods" :key="index" :label="item"
                               :value="item"></el-option>

                </el-select>
                <el-input
                        v-model="query.criteria.requestUrl"
                        placeholder="Request Url"
                        class="handle-input mr10"
                ></el-input>
                <el-button type="primary" icon="el-icon-search" @click="handleSearch">Search</el-button>
            </div>
            <el-table
                    :data="tableData"
                    border
                    class="table"
                    ref="multipleTable"
                    header-cell-class-name="table-header"
                    @selection-change="handleSelectionChange"
            >
                <el-table-column type="selection" width="55" align="center"></el-table-column>
                <el-table-column prop="id" label="ID" width="55" align="center"></el-table-column>
                <el-table-column prop="requestUrl" label="Url"></el-table-column>
                <el-table-column prop="requestMethod" label="Method"></el-table-column>
                <el-table-column prop="contentType" label="Content-Type" align="center"></el-table-column>
                <el-table-column prop="queryString" label="Query-String"></el-table-column>
                <el-table-column label="Status" align="center">
                    <template slot-scope="scope">
                        <el-tag effect="dark" :type="scope.row.stateType">{{scope.row.stateDesc}}</el-tag>
                    </template>
                </el-table-column>

                <el-table-column label="Operation" width="180" align="center">
                    <template slot-scope="scope">
<!--
                        <el-button
                                v-if="scope.row.state === 1"
                                type="text"
                                icon="el-icon-error"
                                class="red"
                                @click="handleEdit(scope.$index, scope.row)"
                        >Stop
                        </el-button>
                        <el-button
                                v-else
                                type="text"
                                icon="el-icon-success"
                                @click="handleEdit(scope.$index, scope.row)"
                        >Run
                        </el-button>
-->

                        <el-button type="text" icon="el-icon-edit" @click="handleEdit(scope.$index, scope.row, scope)" >
                            Edit
                        </el-button>
                        <el-button type="text" icon="el-icon-delete" class="red" @click="handleDelete(scope.$index, scope.row)" >
                            Delete
                        </el-button>
                    </template>
                </el-table-column>
            </el-table>
            <div class="pagination">
                <el-pagination
                        background
                        layout="total, prev, pager, next"
                        :current-page="query.page.pageNumber + 1"
                        :page-size="query.page.pageSize"
                        :total="pageTotal"
                        @current-change="handlePageChange"
                ></el-pagination>
            </div>
        </div>

        <!-- 编辑弹出框 -->
        <el-dialog title="Mock Edit" :visible.sync="editVisible" width="60%">
            <mock-create :isNew="false" :form="form" @close-dialog="editVisible = false; this.getData()"></mock-create>
        </el-dialog>

        <!--
                <el-dialog title="Edit-deprecated" :visible.sync="editVisible-deprecated" width="30%">
                    <el-form ref="form" :model="form" label-width="70px">
                        <el-form-item label="用户名">
                            <el-input v-model="form.name"></el-input>
                        </el-form-item>
                        <el-form-item label="Type">
                            <el-input v-model="form.type"></el-input>
                        </el-form-item>
                    </el-form>
                    <span slot="footer" class="dialog-footer">
                        <el-button @click="editVisible = false">Cancel</el-button>
                        <el-button type="primary" @click="saveEdit">Confirm</el-button>
                    </span>
                </el-dialog>
        -->

    </div>
</template>

<script>
    import {mockList, deleteMock} from '../../../api/mock';
    import {requestMethods} from '../../../utils/constant';
    import mockCreate from './MockCreate.vue' // For edit page

    export default {
        name: 'basetable',
        data() {
            return {
                requestMethods: requestMethods,
                query: {
                    criteria: {
                        requestMethod: '',
                        requestUrl: ''
                    },
                    page: {
                        pageNumber: 0,
                        pageSize: 10
                    },
                    sort: {
                        sortBy: 'id',
                        direction: 'DESC'
                    }
                },
                tableData: [],
                multipleSelection: [],
                editVisible: false,
                pageTotal: 0,
                form: {},
                idx: -1,
                id: -1
            };
        },
        components: {
            mockCreate
        },
        created() {
            this.getData();
        },
        methods: {
            getData() {
                mockList(JSON.stringify(this.query)).then(res => {
                    console.log(res);
                    this.tableData = res.data.elements.map(mock => {

                        mock.stateType = 'success';
                        mock.stateDesc = 'Active';

                        return mock;
                    });
//                this.query.page.pageNumber = res.pageNumber;
//                this.query.page.pageSize = res.pageSize;
                    this.pageTotal = res.data.dataCount || 10;
                });
            },
            handleSearch() {
                //this.$set(this.query.page, 'pageNumber', 1);
                this.getData();
            },
            handleDelete(index, row) {

                this.$confirm('确定要删除吗？', '提示', {
                    type: 'warning'
                })
                    .then(() => {
                        deleteMock(row.id);
                        this.$message.success('删除成功');
                        this.getData();
                        //this.tableData.splice(index, 1);
                    })
                    .catch(() => {
                    });
            },
            handleSelectionChange(val) {
                console.log(val);
                this.multipleSelection = val;
            },
            delAllSelection() {
                let idList = this.multipleSelection.map(select => select.id).join();
                if (idList.length != 0) {
                    deleteMock(idList);
                    this.$message.error(`删除了${idList}`);
                    this.getData();
                }
                this.multipleSelection = [];
            },
            // 编辑操作
            handleEdit(index, row, scope) {
                console.log(scope);
                this.idx = index;
                this.form = row;
                this.editVisible = true;
            },
            // 保存编辑
            saveEdit() {
                this.editVisible = false;
                this.$message.success(`修改第 ${this.idx + 1} 行成功`);
                this.$set(this.tableData, this.idx, this.form);
            },
            // 分页导航
            handlePageChange(val) {
                this.$set(this.query.page, 'pageNumber', val - 1);
                this.getData();
            }
        }
    };
</script>

<style scoped>
    .handle-box {
        margin-bottom: 20px;
    }

    .handle-select {
        width: 120px;
    }

    .handle-input {
        width: 300px;
        display: inline-block;
    }

    .table {
        width: 100%;
        font-size: 14px;
    }

    .red {
        color: #ff0000;
    }

    .mr10 {
        margin-right: 10px;
    }

    .table-td-thumb {
        display: block;
        margin: auto;
        width: 40px;
        height: 40px;
    }
</style>

<template>
  <el-row style="background-color: #FFFFFF;padding: 10px 0;border-radius: 5px;">
    <el-row style="padding: 10px 0 20px;border-bottom: 1px solid #f1f1f1;margin: 0 10px;">
      <el-row>
        <span class="top-bar">标签名</span>
        <el-input size="small" style="width: 188px;" v-model="tagsQueryDto.name" placeholder="输入处" clearable
                  @clear="handleFilterClear">
        </el-input>
        <el-button size="small" class="customer"
                   style="margin-left: 20px;background-color: rgb(43, 121, 203);border: none;" type="primary"
                   @click="handleFilter">立即查询</el-button>
        <el-button size="small" class="customer reset"
                   style="background-color: #f1f1f1;border: none;color: #909399;border: 1px solid #f1f1f1;" type="info"
                   @click="resetQueryCondition">条件重置</el-button>
        <el-button size="small" style="background-color: rgb(43, 121, 203);border: none;" class="customer"
                   type="info" @click="add()">新增标签</el-button>
        <el-button size="small" class="customer"
                   :style="{ marginLeft: '10px', backgroundColor: selectedRows.length ? '#a7535a' : '#F1F1F1', border: 'none', color: selectedRows.length ? '#FFFFFF' : '#909399' }"
                   type="danger" @click="batchDelete()">批量删除</el-button>
      </el-row>
    </el-row>
    <el-row style="margin: 10px;">
      <el-table row-key="id" @selection-change="handleSelectionChange" :data="tableData" style="width: 100%">
        <el-table-column type="selection" width="55"></el-table-column>
        <el-table-column prop="name" width="188" label="标签名"></el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <span class="text-button" @click="handleEdit(scope.row)">编辑</span>
            <span class="text-button" @click="handleDelete(scope.row)">删除</span>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination style="margin: 20px 0;" @size-change="handleSizeChange" @current-change="handleCurrentChange"
                     :current-page="currentPage" :page-sizes="[5, 7]" :page-size="pageSize"
                     layout="total, sizes, prev, pager, next, jumper" :total="totalItems"></el-pagination>
    </el-row>
    <el-dialog :show-close="false" :visible.sync="dialogUserOperaion" width="24%">
      <div slot="title">
        <p class="dialog-title">{{ !isOperation ? '新增新标签' : '编辑标签信息' }}</p>
      </div>
      <div style="padding:0 20px;">
        <el-row>
          <span class="dialog-hover">标签名</span>
          <input class="dialog-input" v-model="data.name" placeholder="标签名" />
        </el-row>
      </div>
      <span slot="footer" class="dialog-footer">
                <el-button size="small" v-if="!isOperation" style="background-color: rgb(43, 121, 203);border: none;"
                           class="customer" type="info" @click="addOperation">新增</el-button>
                <el-button size="small" v-else style="background-color: rgb(43, 121, 203);border: none;"
                           class="customer" type="info" @click="updateOperation">修改</el-button>
                <el-button class="customer" size="small" style="background-color: rgb(241, 241, 241);border: none;"
                           @click="dialogUserOperaion = false">取消</el-button>
            </span>
    </el-dialog>
  </el-row>
</template>

<script>
export default {
  data() {
    return {
      data: {},
      filterText: '',
      currentPage: 1,
      pageSize: 7,
      totalItems: 0,
      dialogUserOperaion: false, // 开关
      isOperation: false, // 开关-标识新增或修改
      tableData: [],
      searchTime: [],
      selectedRows: [],
      status: null,
      tagsQueryDto: {}, // 搜索条件
      messsageContent: ''
    };
  },
  watch: {
    dialogUserOperaion(v1, v2) {
      if (!v1) {
        this.isOperation = !this.isOperation;
      }
      if (!v1 && v2) {
        this.data = {};
      }
    },
  },
  created() {
    this.fetchFreshData();
  },
  methods: {
    add() {
      this.dialogUserOperaion = true;
    },
    // 多选框选中
    async addOperation() {
      try {
        const response = await this.$axios.post('/tags/save', this.data);
        this.$message[response.data.code === 200 ? 'success' : 'error'](response.data.msg);
        if (response.data.code === 200) {
          this.closeDialog();
          this.fetchFreshData();
          this.clearFormData();
        }
      } catch (error) {
        console.error('提交表单时出错:', error);
        this.$message.error('提交失败，请稍后再试！');
      }
    },
    // 批量删除数据
    async batchDelete() {
      if (!this.selectedRows.length) {
        this.$message(`未选中任何数据`);
        return;
      }
      const confirmed = await this.$swalConfirm({
        title: '删除标签数据',
        text: `删除后不可恢复，是否继续？`,
        icon: 'warning',
      });
      if (confirmed) {
        try {
          let ids = this.selectedRows.map(entity => entity.id);
          const response = await this.$axios.post(`/tags/batchDelete`, ids);
          if (response.data.code === 200) {
            this.$swal.fire({
              title: '删除提示',
              text: response.data.msg,
              icon: 'success',
              showConfirmButton: false,
              timer: 2000,
            });
            this.fetchFreshData();
            return;
          }
        } catch (e) {
          this.$swal.fire({
            title: '错误提示',
            text: e,
            icon: 'error',
            showConfirmButton: false,
            timer: 2000,
          });
          console.error(`标签信息删除异常：`, e);
        }
      }
    },
    clearFormData() {
      this.data = {};
    },
    // 修改信息
    closeDialog() {
      this.dialogUserOperaion = false;
    },
    // 信息新增
    fetchFreshData: async function () {
      try {
        // 请求参数
        const params = {
          current: this.currentPage,
          size: this.pageSize,
          ...this.tagsQueryDto
        };
        const response = await this.$axios.post("/tags/query", params);
        const {data} = response;
        this.tableData = data.data;
        this.totalItems = data.total;
      } catch (error) {
        console.error('查询标签信息异常:', error);
      }
    },
    handleAvatarSuccess(res, file) {
      if (res.code !== 200) {
        this.$message.error(`标签头像上传异常`);
        return;
      }
      this.$message.success(`标签头像上传成功`);
      this.data.userAvatar = res.data;
      console.log(this.data);
    },
    handleCurrentChange(val) {
      this.currentPage = val;
      this.fetchFreshData();
    },
    handleDelete(row) {
      this.selectedRows.push(row);
      this.batchDelete();
    },
    handleEdit(row) {
      this.dialogUserOperaion = true;
      this.isOperation = true;
      row.userPwd = null;
      this.data = {...row}
    },
    handleFilter() {
      this.currentPage = 1;
      this.fetchFreshData();
    },
    handleFilterClear() {
      this.filterText = '';
      this.handleFilter();
    },
    handleSelectionChange(selection) {
      this.selectedRows = selection;
    },
    handleSizeChange(val) {
      this.pageSize = val;
      this.currentPage = 1;
      this.fetchFreshData();
    },
    resetQueryCondition() {
      this.tagsQueryDto = {};
      this.searchTime = [];
      this.fetchFreshData();
    },
    async updateOperation() {
      try {
        const response = await this.$axios.put('/tags/update', this.data);
        this.$swal.fire({
          title: '标签信息修改',
          text: response.data.msg,
          icon: response.data.code === 200 ? 'success' : 'error',
          showConfirmButton: false,
          timer: 1000,
        });
        if (response.data.code === 200) {
          this.closeDialog();
          this.fetchFreshData();
          this.clearFormData();
        }
      } catch (error) {
        console.error('提交表单时出错:', error);
        this.$message.error('提交失败，请稍后再试！');
      }
    }
  },
};
</script>
<style scoped lang="scss">
.tag-tip {
  display: inline-block;
  padding: 5px 10px;
  border-radius: 5px;
  background-color: rgb(245, 245, 245);
  color: rgb(104, 118, 130);
}

.input-def {
  height: 40px;
  line-height: 40px;
  outline: none;
  border: none;
  font-size: 20px;
  color: rgb(102, 102, 102);
  font-weight: 900;
  width: 100%;
}

.dialog-footer {
  /* 使按钮水平居中 */
  display: flex;
  justify-content: center;
  align-items: center;
}

/* 如果需要调整按钮之间的间距 */
.customer {
  margin: 0 8px;
  /* 根据需要调整间距 */
}
</style>
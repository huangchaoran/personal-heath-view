<template>
  <el-row style="background-color: #FFFFFF;padding: 10px 0;border-radius: 5px;">
    <el-row style="padding: 10px 0 20px;border-bottom: 1px solid #f1f1f1;margin: 0 10px;">
      <el-row>
        <span class="top-bar">咨询标题</span>
        <el-input size="small" style="width: 188px;" v-model="newsQueryDto.name" placeholder="输入处" clearable
                  @clear="handleFilterClear">
        </el-input>
        <span class="top-bar" >分类</span>
        <el-select size="small" v-model="newsQueryDto.tagId" placeholder="请选择">
          <el-option
              v-for="tag in tagsList"
              :key="tag.id"
              :label="tag.name"
              :value="tag.id">
          </el-option>
        </el-select>
        <span class="top-bar">注册时间</span>
        <el-date-picker size="small" style="margin-left: 10px;width: 220px;" v-model="searchTime"
                        type="daterange" range-separator="至" start-placeholder="开始时间" end-placeholder="结束时间">
        </el-date-picker>
        <span class="top-bar">是否推荐</span>
        <el-switch
            v-model="newsQueryDto.isTop"
            active-color="#13ce66"
            inactive-color="#ff4949">
        </el-switch>
        <el-button size="small" class="customer"
                   style="margin-left: 20px;background-color: rgb(43, 121, 203);border: none;" type="primary"
                   @click="handleFilter">立即查询</el-button>
        <el-button size="small" class="customer reset"
                   style="background-color: #f1f1f1;border: none;color: #909399;border: 1px solid #f1f1f1;" type="info"
                   @click="resetQueryCondition">条件重置</el-button>
        <el-button size="small" style="background-color: rgb(43, 121, 203);border: none;" class="customer"
                   type="info" @click="add()">新增咨询</el-button>
        <el-button size="small" class="customer"
                   :style="{ marginLeft: '10px', backgroundColor: selectedRows.length ? '#a7535a' : '#F1F1F1', border: 'none', color: selectedRows.length ? '#FFFFFF' : '#909399' }"
                   type="danger" @click="batchDelete()">批量删除</el-button>
      </el-row>
    </el-row>
    <el-row style="margin: 10px;">
      <el-table row-key="id" @selection-change="handleSelectionChange" :data="tableData" style="width: 100%">
        <el-table-column type="selection" width="55"></el-table-column>
        <el-table-column prop="name" width="188" label="咨询名"></el-table-column>
        <el-table-column prop="cover" width="138" label="封面">
          <template slot-scope="scope">
            <img :src="scope.row.cover" style="width: 88px;height: 54px;border-radius: 5px;"/>
          </template>
        </el-table-column>
        <el-table-column prop="name" width="218" label="资讯名"></el-table-column>
        <el-table-column prop="tagName" width="88" label="标签"></el-table-column>
        <el-table-column prop="isTop" width="148" label="是否推荐">
          <template slot-scope="scope">
            <span>{{ scope.row.isTop ? '已经推荐' : '未推荐' }}</span>
          </template>
        </el-table-column>
        <el-table-column prop="createTime" width="168" label="发布时间"></el-table-column>
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
    <el-dialog :show-close="false" :visible.sync="dialogUserOperaion" width="50%">
      <div slot="title">
        <p class="dialog-title">{{ !isOperation ? '新增新咨询' : '编辑咨询信息' }}</p>
      </div>
      <div style="padding:0 20px;">
        <el-row style="margin: 12px 0px">
          <span class="dialog-hover">标题</span>
          <input class="dialog-input" v-model="data.name" placeholder="标题名" />
        </el-row>
        <el-row>
          <el-upload class="avatar-uploader" action="/api/personal-heath/v1.0/file/upload"
                     :show-file-list="false" :on-success="handleAvatarSuccess">
            <img v-if="data.cover" :src="data.cover" class="dialog-avatar">
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
          </el-upload>
        </el-row>
        <el-row style="margin: 12px 0px">
          <el-row>
            <span class="dialog-hover">分类</span>
          </el-row>
          <el-radio-group v-model="data.tagId">
            <el-radio :key="index" :label="tag.id" v-for="(tag,index) in tagsList">{{tag.name}}</el-radio>
          </el-radio-group>
        </el-row>

        <el-row style="margin: 12px 0px">
          <el-row > <span class="dialog-hover">是否推荐</span></el-row>
          <el-switch
              v-model="data.isTop"
              active-color="#13ce66"
              inactive-color="#ff4949">
          </el-switch>
        </el-row>
        <el-row>
          <Editor :receive-content="data.content" @on-receive="onReceiveContent" />
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
import Editor from "@/components/Editor";

export default {
  components: { Editor },
  data() {
    return {
      data: { cover:''},
      currentPage: 1,
      pageSize: 7,
      totalItems: 0,
      dialogUserOperaion: false, // 开关
      isOperation: false, // 开关-标识新增或修改
      tableData: [],
      searchTime: [],
      value: true,
      selectedRows: [],
      status: null,
      newsQueryDto: {}, // 搜索条件
      messsageContent: '',
      tagsList: []


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
    this.loadAllTags();
  },
  methods: {
    onReceiveContent(html) {
      this.data.content=html;
    },
    loadAllTags(){
      this.$axios.post('/tags/query',{}).then(response => {
        const { data } = response;
        if (data.code === 200) {
          this.tagsList = data.data;
        }
      })
    },
    handleAvatarSuccess(res, file) {
      if (res.code !== 200) {
        this.$message.error(`资讯封面上传异常`);
        return;
      }
      this.$message.success(`资讯封面上传成功`);
      this.data.cover = res.data;
    },
    // 多选框选中
    handleSelectionChange(selection) {
      this.selectedRows = selection;
    },
    // 批量删除数据
    async batchDelete() {
      if (!this.selectedRows.length) {
        this.$message(`未选中任何数据`);
        return;
      }
      const confirmed = await this.$swalConfirm({
        title: '删除咨询数据',
        text: `删除后不可恢复，是否继续？`,
        icon: 'warning',
      });
      if (confirmed) {
        try {
          let ids = this.selectedRows.map(entity => entity.id);
          const response = await this.$axios.post(`/news/batchDelete`, ids);
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
          console.error(`咨询信息删除异常：`, e);
        }
      }
    },
    resetQueryCondition() {
      this.newsQueryDto = {};
      this.searchTime = [];
      this.fetchFreshData();
    },
    // 修改信息
    async updateOperation() {
      try {
        const response = await this.$axios.put('/news/update', this.data);
        this.$swal.fire({
          title: '咨询信息修改',
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
    },
    // 信息新增
    async addOperation() {
      try {
        const response = await this.$axios.post('/news/save', this.data);
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
    closeDialog() {
      this.dialogUserOperaion = false;
    },
    clearFormData() {
      this.data = {};
    },
    async fetchFreshData() {
      try {
        this.tableData = [];
        let startTime = null;
        let endTime = null;
        if (this.searchTime != null && this.searchTime.length === 2) {
          const [startDate, endDate] = await Promise.all(this.searchTime.map(date => date.toISOString()));
          startTime = `${startDate.split('T')[0]}T00:00:00`;
          endTime = `${endDate.split('T')[0]}T23:59:59`;
        }
        // 请求参数
        const params = {
          current: this.currentPage,
          size: this.pageSize,
          startTime: startTime,
          endTime: endTime,
          ...this.newsQueryDto
        };
        const response = await this.$axios.post('/news/query', params);
        const { data } = response;
        this.tableData = data.data;
        this.totalItems = data.total;
      } catch (error) {
        console.error('查询咨询信息异常:', error);
      }
    },
    add() {
      this.dialogUserOperaion = true;
    },
    handleFilter() {
      this.currentPage = 1;
      this.fetchFreshData();
    },
    handleFilterClear() {
      this.filterText = '';
      this.handleFilter();
    },
    handleSizeChange(val) {
      this.pageSize = val;
      this.currentPage = 1;
      this.fetchFreshData();
    },
    handleCurrentChange(val) {
      this.currentPage = val;
      this.fetchFreshData();
    },
    handleEdit(row) {
      this.dialogUserOperaion = true;
      this.isOperation = true;
      row.userPwd = null;
      this.data = { ...row}
    },
    handleDelete(row) {
      this.selectedRows.push(row);
      this.batchDelete();
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
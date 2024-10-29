<template>
  <!-- 主容器，设置背景颜色、内边距和圆角 -->
  <el-row style="background-color: #FFFFFF;padding: 10px 0;border-radius: 5px;">
    <!-- 顶部搜索栏，设置内边距、下边框和外边距 -->
    <el-row style="padding: 10px 0 20px;border-bottom: 1px solid #f1f1f1;margin: 0 10px;">
      <el-row>
        <!-- 评论内容标签 -->
        <span class="top-bar">评论内容</span>
        <!-- 输入框，绑定评论内容，设置宽度、占位符和清除按钮，支持回车搜索 -->
        <el-input size="small" style="width: 190px;" v-model="evalustionsQueryDto.content" placeholder="输入处"
                  clearable @keyup.enter.native="keySearch">
        </el-input>
        <!-- 评论时间标签 -->
        <span style="display: inline-block;margin-left: 10px;font-size: 14px;color: #909399;">评论时间</span>
        <!-- 日期选择器，绑定搜索时间，设置宽度、分隔符和占位符 -->
        <el-date-picker size="small" style="margin-left: 10px;width: 220px;" v-model="searchTime"
                        type="daterange" range-separator="至" start-placeholder="开始时间" end-placeholder="结束时间">
        </el-date-picker>
        <!-- 立即查询按钮，设置内边距、背景颜色和边框，点击事件触发查询 -->
        <el-button size="small" class="customer"
                   style="margin-left: 10px;background-color: rgb(43, 121, 203);border: none;" type="primary"
                   @click="handleFilter">立即查询</el-button>
        <!-- 条件重置按钮，设置背景颜色、边框和文字颜色，点击事件重置查询条件 -->
        <el-button size="small" class="customer reset"
                   style="background-color: #f1f1f1;border: none;color: #909399;border: 1px solid #f1f1f1;" type="info"
                   @click="resetQueryCondition">条件重置</el-button>
        <!-- 批量删除按钮，根据选中行数动态设置背景颜色和文字颜色，点击事件触发批量删除 -->
        <el-button size="small" class="customer" icon="el-icon-delete"
                   :style="{ backgroundColor: selectedRows.length ? '#a7535a' : '#F1F1F1', border: 'none', color: selectedRows.length ? '#FFFFFF' : '#909399' }"
                   type="danger" @click="batchDelete()">批量删除</el-button>
      </el-row>
    </el-row>
    <!-- 评论层级说明，设置内边距和外边距 -->
    <el-row style="padding: 10px 0 0 10px;">
            <span style="margin: 8px 10px;font-size: 12px;">
                <!-- 父级评论说明 -->
                <span>
                    <i
                        style="display: inline-block;width: 10px;height: 10px;border-radius: 5px;background-color: #ee3f4d;"></i>
                    <span style="color: rgb(102, 102, 102);margin-left: 6px;">父级评论</span>
                </span>
              <!-- 子级评论说明 -->
                <span style="margin-left: 10px;">
                    <i
                        style="display: inline-block;width: 10px;height: 10px;border-radius: 5px;background-color: #bec936"></i>
                    <span style="color: rgb(102, 102, 102);margin-left: 6px;">子级评论</span>
                </span>
            </span>
    </el-row>
    <!-- 表格展示区域，设置外边距 -->
    <el-row style="margin: 10px;">
      <!-- 表格组件，绑定数据，设置行键和选择变化事件 -->
      <el-table row-key="id" @selection-change="handleSelectionChange" :data="tableData" style="width: 100%">
        <!-- 选择列 -->
        <el-table-column type="selection" width="55"></el-table-column>
        <!-- 层级列，根据parentId显示不同的图标和文字 -->
        <el-table-column prop="parentId" width="100" label="层级">
          <template slot-scope="scope">
                        <span v-if="!scope.row.parentId">
                            <i
                                style="display: inline-block;width: 10px;height: 10px;border-radius: 5px;background-color: #ee3f4d;"></i>
                            父级评论
                        </span>
            <span v-else>
                            <i
                                style="display: inline-block;width: 10px;height: 10px;border-radius: 5px;background-color: #bec936"></i>
                            子级评论
                        </span>
          </template>
        </el-table-column>
        <!-- 挂载源列，显示contentType -->
        <el-table-column prop="contentType" width="70" label="挂载源">
          <template slot-scope="scope">
            <span>{{ scope.row.contentType }}</span>
          </template>
        </el-table-column>
        <!-- 评论于列，显示createTime -->
        <el-table-column prop="createTime" width="168" label="评论于">
          <template slot-scope="scope">
            <span>{{ scope.row.createTime }}</span>
          </template>
        </el-table-column>
        <!-- 评论者列，显示userName并高亮关键词 -->
        <el-table-column prop="userName" width="150" label="评论者">
          <template slot-scope="scope">
            <span v-html="highlightKeyword(scope.row.userName)"></span>
          </template>
        </el-table-column>
        <!-- 被评论者列，显示replierName并高亮关键词 -->
        <el-table-column prop="replierName" width="150" label="被评论者">
          <template slot-scope="scope">
            <span v-html="highlightKeyword(scope.row.replierName)"></span>
          </template>
        </el-table-column>
        <!-- 点赞列，显示upvoteList的长度 -->
        <el-table-column prop="upvoteList" width="60" label="点赞">
          <template slot-scope="scope">
                        <span v-if="scope.row.upvoteList !== null" style="font-size: 16px;font-weight: bolder;">{{
                            scope.row.upvoteList.split(',').length }}</span>
            <span v-else style="font-size: 16px;font-weight: bolder;">0</span>
          </template>
        </el-table-column>
        <!-- 评论内容列，显示content -->
        <el-table-column prop="content" width="218" label="评论内容">
          <template slot-scope="scope">
            <span>{{ scope.row.content }}</span>
          </template>
        </el-table-column>
        <!-- 数据操作列，显示删除按钮 -->
        <el-table-column label="数据操作">
          <template slot-scope="scope">
            <span class="text-button" @click="handleDelete(scope.row)">删除</span>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页组件，设置当前页、每页大小、总条数和布局 -->
      <el-pagination style="margin: 10px 0;" @size-change="handleSizeChange" @current-change="handleCurrentChange"
                     :current-page="currentPage" :page-sizes="[20, 50]" :page-size="pageSize"
                     layout="total, sizes, prev, pager, next, jumper" :total="totalItems"></el-pagination>
    </el-row>
    <!-- 举报面板，设置标题、关闭按钮和可见性 -->
    <el-dialog title="" :show-close="false" :visible.sync="reportDialog" width="35%">
      <!-- 自定义标题区域，设置内边距 -->
      <div slot="title" style="padding: 25px 0 0 20px;">
        <span style="font-size: 18px;font-weight: 800;">举报详情</span>
      </div>
      <!-- 举报详情内容区域，设置内边距 -->
      <el-row style="padding: 10px 20px 20px 20px;">
        <!-- 左侧饼图区域 -->
        <el-col :span="12">
          <PieChart :types="types" :values="values" />
        </el-col>
        <!-- 右侧数据列表区域 -->
        <el-col :span="12">
          <el-row class="main">
            <!-- 如果没有数据，显示暂无数据 -->
            <div v-if="!reportsDate.length">
              <span class="count">暂无数据</span>
            </div>
            <!-- 遍历reportsDate数据，显示每项的计数和名称 -->
            <el-col :span="6" class="item" v-for="(entity, index) in reportsDate" :key="index">
              <div class="count" style="margin-top: 20px;">
                                <span>
                                    {{ entity.count }}
                                </span>
              </div>
              <div>{{ entity.name }}</div>
            </el-col>
          </el-row>
        </el-col>
      </el-row>
      <!-- 底部按钮区域 -->
      <span slot="footer" class="dialog-footer">
                <!-- 取消按钮，点击关闭举报面板 -->
                <el-button class="customer" size="small" style="background-color: rgb(241, 241, 241);border: none;"
                           @click="reportDialog = false">取 消</el-button>
            </span>
    </el-dialog>
  </el-row>
</template>

<script>
import Editor from "@/components/Editor"; // 引入编辑器组件
import PieChart from "@/components/PieChart"; // 引入饼图组件

export default {
  components: {
    Editor,
    PieChart
  },
  data() {
    return {
      data: { cover: '' }, // 数据对象，包含封面图
      reportsDate: [], // 举报数据列表
      filterText: '', // 过滤文本
      tableData: [], // 表格数据
      currentPage: 1, // 当前页码
      pageSize: 20, // 每页大小
      totalItems: 0, // 总条数
      dialogOperaion: false, // 操作对话框可见性
      isOperation: false, // 是否为操作状态
      searchTime: [], // 搜索时间范围
      selectedRows: [], // 选中的行
      html: {}, // HTML内容
      fileList: [], // 文件列表
      dynamicTags: ['健康', '养生'], // 动态标签
      inputVisible: false, // 输入框可见性
      inputValue: '', // 输入框值
      coverDialog: false, // 封面图对话框可见性
      commentDialog: false, // 评论对话框可见性
      evaluationsList: [], // 评论列表
      selectedRows: [], // 选中的行
      count: 0, // 计数
      evaluationsDelete: false, // 评论删除状态
      idToDelete: null, // 待删除的ID
      contentDialog: false, // 内容对话框可见性
      reportDialog: false, // 举报对话框可见性
      types: [], // 举报类型
      values: [], // 举报值
      evalustionsQueryDto: {}, // 评论查询DTO
    };
  },
  created() {
    this.fetchFreshData(); // 组件创建时获取最新数据
  },
  methods: {
    // 回车搜索
    keySearch() {
      this.fetchFreshData();
    },
    // 打开举报面板并加载举报数据
    reportsList(row) {
      this.reportDialog = true;
      this.reportList(row.id);
    },
    // 获取举报数据
    async reportList(id) {
      const response = await this.$axios(`/evaluations-reports/reportCount/${id}`);
      const { data } = response;
      this.reportsDate = data.data;
      this.types = this.reportsDate.map(entity => entity.name);
      this.values = this.reportsDate.map(entity => entity.count);
    },
    // 显示内容
    showContent(news) {
      this.data = news;
      this.contentDialog = true;
    },
    // 删除评论
    async removeEvaluations(id) {
      this.evaluationsDelete = true;
      this.idToDelete = id;
    },
    // 确认删除
    confirmDelete() {
      this.$axios.delete(`/evaluations/delete/${this.idToDelete}`).then((response) => {
        this.$message[response.data.code === 200 ? 'success' : 'error'](response.data.msg);
        if (response.data.code === 200) {
          this.evaluationsDelete = false;
          this.loadEvaluationsList();
        }
      }).catch((error) => {
        console.log(error);
      });
    },
    // 关闭评论对话框
    handleEvaluationsClose() {
      this.commentDialog = false;
    },
    // 查看评论列表
    commentList(news) {
      this.data = news;
      this.commentDialog = true;
      this.loadEvaluationsList();
    },
    // 加载评论数据列表
    async loadEvaluationsList() {
      try {
        const response = await this.$axios.get(`/evaluations/list/${this.data.id}/evaluations`);
        this.evaluationsList = response.data.data.data;
        this.count = response.data.data.count;
      } catch (error) {
        console.error(`加载评论列表异常：`, error);
      }
    },
    // 查看封面图
    showPic(news) {
      this.data = news;
      this.coverDialog = true;
    },
    // 关闭标签
    handleClose(tag) {
      this.dynamicTags.splice(this.dynamicTags.indexOf(tag), 1);
    },
    // 显示输入框
    showInput() {
      this.inputVisible = true;
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus();
      });
    },
    // 确认输入
    handleInputConfirm() {
      let inputValue = this.inputValue;
      if (inputValue) {
        this.dynamicTags.push(inputValue);
      }
      this.inputVisible = false;
      this.inputValue = '';
    },
    // 返回
    goBack() {
      this.data = {};
      this.dialogOperaion = false;
    },
    // 返回页面
    goBackPage() {
      this.data = {};
      this.commentDialog = false;
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
        title: '删除评论数据',
        text: `删除后不可恢复，是否继续？`,
        icon: 'warning',
      });
      if (confirmed) {
        try {
          let ids = this.selectedRows.map(entity => entity.id);
          const response = await this.$axios.post(`/evaluations/batchDelete`, ids);
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
          console.error(`评论信息删除异常：`, e);
        }
      }
    },
    // 关键词高亮显示
    highlightKeyword(text) {
      if (text == null) {
        return;
      }
      if (this.filterText != '') {
        const regex = new RegExp(this.filterText, 'gi');
        return text.replace(regex, '<span class="highlight" style="background-color: #bec936;color:#373834;">$&</span>');
      }
      return text;
    },
    // 重置查询条件
    resetQueryCondition() {
      this.filterText = '';
      this.searchTime = '';
      this.fetchFreshData();
    },
    // 更新操作
    async updateOperation() {
      try {
        const response = await this.$axios.put('/evaluations/update', this.data);
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
    // 添加操作
    async addOperation() {
      this.data.tag = this.dynamicTags.join(',');
      try {
        const response = await this.$axios.post('/evaluations/save', this.data);
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
    // 关闭表单
    closeDialog() {
      this.dialogOperaion = false;
    },
    // 清除表单数据
    clearFormData() {
      this.data = {};
      this.html = '';
      this.fileList = [];
    },
    // 获取最新数据
    async fetchFreshData() {
      try {
        let startTime = '';
        let endTime = '';
        if (this.searchTime !== null && this.searchTime.length === 2) {
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
          ...this.evalustionsQueryDto,
        };
        // 使用await等待请求完成
        let response = await this.$axios.post('/evaluations/query', params);
        const { data } = response;
        this.tableData = data.data;
        this.totalItems = data.total;
      } catch (error) {
        // 错误处理
        console.error('Fetch data failed:', error);
      }
    },
    // 添加
    add() {
      this.dialogOperaion = true;
    },
    // 处理过滤
    handleFilter() {
      this.currentPage = 1;
      this.fetchFreshData();
    },
    // 处理每页大小变化
    handleSizeChange(val) {
      this.pageSize = val;
      this.currentPage = 1;
      this.fetchFreshData();
    },
    // 处理当前页变化
    handleCurrentChange(val) {
      this.currentPage = val;
      this.fetchFreshData();
    },
    // 处理编辑
    handleEdit(row) {
      this.dialogOperaion = true;
      this.isOperation = true;
      this.data = row;
      console.log(this.data);
    },
    // 处理删除
    async handleDelete(row) {
      this.selectedRows.push(row);
      this.batchDelete();
    }
  },
};
</script>

<style scoped lang="scss">
.main {
  padding: 50px 0;
  border-radius: 3px;
  display: flex;
  justify-content: center;

  .item:last-child {
    border-right: none;
  }

  .count,
  .tag {
    text-align: center;
  }

  .count {
    height: 30px;
    line-height: 30px;
    font-size: 26px;
    font-weight: bold;
    color: rgb(120 102 102);
  }

  .tag {
    height: 30px;
    line-height: 30px;
    font-size: 12px;
    color: rgb(120 102 102);
  }
}

.tag {
  color: #1E80FF;
  background-color: #EAF2FF;
  padding: 3px 4px;
  border-radius: 3px;
  font-size: 10px;
  margin-right: 5px;
}

.tag1 {
  color: #1c1c1c;
  background-color: #d7d8b8;
  padding: 3px 4px;
  border-radius: 3px;
  font-size: 10px;
  margin-right: 5px;
}
</style>
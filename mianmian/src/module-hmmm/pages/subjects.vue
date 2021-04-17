<template>
  <div class="pages-subjects">
    <div class="el-card__body">
      <!-- 顶部搜索模块 -->
      <div class="el-row">
        <div class="el-col-18">
          <el-form :inline="true" :model="queryInfo" class="demo-form-inline">
            <el-form-item label="学科名称">
              <el-input v-model="queryInfo.subjectName"></el-input>
            </el-form-item>

            <!-- 清除 -->
            <el-form-item>
              <el-button @click="onRemove">清除</el-button>
            </el-form-item>

            <!-- 查询 -->
            <el-form-item>
              <el-button type="primary" @click="getList">查询</el-button>
            </el-form-item>
          </el-form>
        </div>

        <!-- 新增标签 -->
        <div class="el-col-6">
          <!-- 新增按钮的插槽 -->
          <el-button
            type="success"
            icon="el-icon-edit"
            size="small"
            @click="addSubject"
            >新增标签</el-button
          >
        </div>
        <!-- /新增标签 -->
      </div>
      <!-- /顶部搜索模块 -->

      <!-- 消息提示 -->
      <div class="el-alert">
        <el-alert type="info" :closable="false">
          <!-- show-icon -->
          <i class="el-alert__icon el-icon-info"></i>
          <span class="el-alert__title">数据一共{{ total.counts }} 条</span>
        </el-alert>
      </div>
      <!-- /消息提示 -->

      <!-- 标签内容 -->
      <div class="el-table">
        <el-table :data="getSubList" style="width: 100%">
          <el-table-column type="index" prop="date" label="序号" width="48">
          </el-table-column>
          <el-table-column prop="subjectName" label="学科名称" width="90">
          </el-table-column>
          <el-table-column prop="username" label="创建者" width="110">
          </el-table-column>
          <el-table-column label="创建日期" width="210">
            <!-- 日期插槽 -->
            <template slot-scope="scope">
              {{ scope.row.addDate | parseTime }}
            </template>
            <!-- /日期插槽 -->
          </el-table-column>
          <el-table-column label="是否前台显示" width="110">
            <template slot-scope="scope">
              <span v-if="scope.row.isFrontDisplay == '1'">
                是
              </span>
              <span v-else>否</span>
            </template>
          </el-table-column>
          <el-table-column
            prop="twoLevelDirectory"
            label="二级目录"
            width="110"
          >
          </el-table-column>
          <el-table-column prop="tags" label="标签" width="120">
          </el-table-column>
          <el-table-column prop="totals" label="题目数量" width="140">
          </el-table-column>
          <el-table-column label="操作" width="230">
            <!-- 插槽 -->
            <template slot-scope="scope">
              <el-button
                type="text"
                @click="$router.push('/subjects/directorys')"
                >学科分类</el-button
              >
              <el-button type="text" @click="$router.push('/subjects/tags')"
                >学科标签</el-button
              >

              <!-- 修改 -->
              <el-button type="text" @click="editor(scope.row)">修改</el-button>

              <!-- 删除 -->
              <el-button type="text" @click="deleteSubjects(scope.row.id)"
                >删除</el-button
              >
            </template>
            <!-- /插槽 -->
          </el-table-column>
        </el-table>
      </div>
      <!-- /标签内容 -->

      <!-- 修改的弹出层 -->
      <el-dialog title="修改学科" :visible.sync="editSubjectShow" width="30%">
        <!-- 学科名称 -->
        <el-form
          :model="subjectFrom"
          :rules="rules"
          ref="subjectFromRef"
          label-width="100px"
          class="demo-subjectFrom"
        >
          <el-form-item label="学科名称" prop="subjectName">
            <el-input v-model="subjectFrom.subjectName"></el-input>
          </el-form-item>
        </el-form>

        <!-- 是否显示的switch开关 -->
        <el-form ref="form" :model="form" label-width="100px">
          <el-form-item label="是否显示">
            <el-switch
              v-model="subjectFrom.isFrontDisplay"
              active-color="#13ce66"
              inactive-color="#ff4949"
              active-value="1"
              inactive-value="0"
            >
            </el-switch>
          </el-form-item>
        </el-form>

        <div slot="footer" class="dialog-footer">
          <el-button @click="editSubjectShow = false">取 消</el-button>
          <el-button type="primary" @click="editSubFrom">确 定</el-button>
        </div>
      </el-dialog>
      <!-- /修改的弹出层 -->

      <!-- 分页功能 -->
      <div class="block">
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="queryInfo.page"
          :page-sizes="[5, 10, 20, 25]"
          background
          :page-size="queryInfo.pagesize"
          layout="prev, pager, next, sizes,jumper"
          :total="total.counts"
        >
        </el-pagination>
      </div>
      <!-- /分页功能 -->

      <!-- 新增标签 -->
      <el-dialog title="新增学科" :visible.sync="addSubjectShow" width="30%">
        <!-- 学科名称 -->
        <el-form
          :model="addSubjectFrom"
          :rules="rules"
          ref="addSubjectFrom"
          label-width="100px"
          class="demo-addSubjectFrom"
        >
          <el-form-item label="学科名称" prop="subjectName">
            <el-input v-model="addSubjectFrom.subjectName"></el-input>
          </el-form-item>
        </el-form>

        <!-- switch开关 -->
        <el-form ref="form" :model="form" label-width="100px">
          <el-form-item label="是否显示">
            <el-switch
              v-model="addSubjectFrom.isFrontDisplay"
              active-color="#13ce66"
              inactive-color="#ff4949"
              active-value="1"
              inactive-value="0"
            >
            </el-switch>
          </el-form-item>
        </el-form>

        <div slot="footer" class="dialog-footer">
          <el-button @click="addSubjectShow">取 消</el-button>
          <el-button type="primary" @click="onCheck">确 定</el-button>
        </div>
      </el-dialog>
      <!-- /新增标签 -->
    </div>
  </div>
</template>

<script>
import { list as getSubjects } from "@/api/hmmm/subjects";
import { update as updateSub } from "@/api/hmmm/subjects";
import { remove as removeSub } from "@/api/hmmm/subjects";
import { add as addSub } from "@/api/hmmm/subjects";

export default {
  name: "PagesSubjects",
  components: {},
  props: {},
  data() {
    return {
      // 获取学科数据
      getSubList: [], // 所有学科数据
      queryInfo: {
        page: 1,
        pagesize: 5,
        subjectName: null
      },
      // 修改学科
      formSubjects: {
        subjectName: ""
      },
      // 修改学科的弹出框
      editSubjectShow: false, //关闭修改学科的弹出框
      form: {},
      formLabelWidth: "90px",
      // 学科名称 验证规则
      rules: {
        subjectName: [
          { required: true, message: "请输入学科名称", trigger: "blur" }
          // { min: 3, max: 5, message: "长度在 3 到 5 个字符", trigger: "blur" }
        ]
      },
      // 修改的默认表单值
      subjectFrom: {
        subjectName: "",
        isFrontDisplay: "",
        id: ""
      },
      total: {}, // 数据总数
      // 新增标签
      dialogFormVisible: false,
      formLabelWidth: "120px",
      addSubjectShow: false, // 默认关闭新增标签的弹窗
      // 新增标签的表单
      addSubjectFrom: {
        subjectName: "",
        isFrontDisplay: "1"
      },
      // 新增学科的验证规则
      rules: {
        name: [{ required: true, message: "请输入活动名称", trigger: "blur" }]
      }
    };
  },
  computed: {},
  watch: {},
  created() {
    this.getList(); // 渲染整个页面
  },
  mounted() {},
  methods: {
    // 获取学科列表
    async getList() {
      try {
        const { data } = await getSubjects({
          page: this.queryInfo.page,
          pagesize: this.queryInfo.pagesize,
          subjectName: this.queryInfo.subjectName
        });
        // console.log(data);
        // 获取总条数
        this.total = data;
        this.getSubList = data.items;
        // this.$message.success("小主牛批，获取学科列表成功");
      } catch (error) {
        this.$message.error("憨憨小主，获取学科列表失败");
      }
    },

    // 展示修改学科 的弹窗
    async editor(row) {
      // console.log(row);
      this.subjectFrom = {
        subjectName: row.subjectName,
        isFrontDisplay: row.isFrontDisplay.toString(),
        id: row.id
      };
      this.editSubjectShow = true; //点击'修改'开启弹窗
    },

    // 修改学科
    async editSubFrom() {
      try {
        // 获取数据
        await updateSub(this.subjectFrom);
        // 重新渲染数据
        this.getList();
        // 关闭弹窗
        this.editSubjectShow = false;
      } catch (error) {
        this.$message.error("小主真笨，修改学科失败");
      }
    },

    // 删除学科
    async deleteSubjects(id) {
      this.$confirm("此操作将永久删除该文件, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(async () => {
          const data = await removeSub({
            id
          });

          // 重新渲染列表
          this.getList();

          this.$message({
            type: "success",
            message: "删除成功!"
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
      // 获取数据
    },

    // 清除
    onRemove() {
      this.queryInfo.subjectName = "";
      this.getList();
    },

    // 分页
    // 每页多少个
    handleSizeChange(val) {
      this.queryInfo.pagesize = val;
      // if (this.queryInfo.page === 1) {
      this.getList();
      // }
    },
    // 进入某一页
    handleCurrentChange(val) {
      this.queryInfo.page = val;
      this.getList();
    },

    // 新增标签
    // 展示新增标签的弹窗
    addSubject() {
      this.addSubjectShow = true;
    },

    // 新增标签
    async onCheck() {
      try {
        // 获取数据
        const { data } = await addSub(this.addSubjectFrom);
        console.log(data);
        // 请空表单
        this.addSubjectFrom.subjectName = "";
        // 关闭弹窗
        this.addSubjectShow = false;
        this.getList();
      } catch (error) {
        this.$message.error("笨蛋小主，新增失败");
      }
    }
  }
};
</script>

<style scoped lang="scss">
.pages-subjects {
  padding: 10px;
  width: 100%;
  .el-card__body {
    padding: 20px;
    border: 1px solid #ebeef5;
    background-color: #fff;
    // 顶部搜索模块
    .el-row {
      display: flex;
      justify-content: space-between;
      .el-col-18 {
        width: 75%;
        margin-left: 20px;
      }
      .el-col-6 {
        width: 25%;
        text-align: right;
      }
    }
    // /顶部搜索模块

    // <!-- 消息提示 -->
    .el-alert {
      height: 35px;
      line-height: 35px;
      padding: 0;
    }
    //  /消息提示

    // /标签内容
    .el-table {
      margin-top: 15px;
      .el-alert__icon {
        font-size: 20px;
        width: 16px;
      }
      .el-icon-info {
        content: "\e7a1";
      }
    }
    // /标签内容
  }
  .block {
    text-align: right;
  }
}
</style>

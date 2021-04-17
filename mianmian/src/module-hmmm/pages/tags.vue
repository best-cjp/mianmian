<template>
  <div class="pages-tags">
    <div class="el-card">
      <!-- 顶部搜索 -->
      <div class="el-row">
        <!-- 标签名称 -->
        <div class="el-col-18">
          <el-form
            :inline="true"
            :model="tagsFrom"
            class="demo-form-inline"
            style="padding-left: 20px;"
          >
            <el-form-item label="标签名称">
              <el-input v-model="tagsFrom.tagName" size="small"></el-input>
            </el-form-item>
            <el-form-item label="状态" style="margin-left: 40px;">
              <el-select
                v-model="tagsFrom.status"
                placeholder="请选择"
                size="small"
              >
                <el-option
                  v-for="item in status"
                  :key="item.value"
                  :label="item.label"
                  :value="item.value"
                ></el-option>
              </el-select>
            </el-form-item>
            <!-- 清除 -->
            <el-form-item>
              <el-button @click="onRemove" size="small">清除</el-button>
            </el-form-item>
            <!-- 查询 -->
            <el-form-item>
              <el-button type="primary" @click="getList" size="small"
                >查询</el-button
              >
            </el-form-item>
          </el-form>
        </div>

        <!-- 新增标签 -->
        <div class="el-col-6">
          <el-button type="success" icon="el-icon-edit" @click="addShowTsgs"
            >新增标签</el-button
          >
        </div>
      </div>
      <!-- /顶部搜索 -->

      <!-- 消息提示 -->
      <div>
        <el-alert
          title="数据一共 {{}} 条"
          type="info"
          show-icon
          :closable="false"
        ></el-alert>
      </div>
      <!--/消息提示 -->

      <!-- tab表格 -->
      <div class="el-alert">
        <el-table :data="tagsList" style="width: 100%">
          <el-table-column type="index" prop="date" label="序号" width="60">
          </el-table-column>

          <el-table-column prop="subjectName" label="所属学科" width="180">
          </el-table-column>

          <el-table-column prop="tagName" label="标签名称" width="180">
          </el-table-column>
          <el-table-column prop="username" label="创建者" width="190">
          </el-table-column>

          <el-table-column label="创建日期" width="200">
            <!-- 日期插槽 -->
            <template slot-scope="scope">
              {{ scope.row.addDate | parseTime }}
            </template>
          </el-table-column>

          <el-table-column prop="state" label="状态" width="180">
            <!-- 状态插槽 -->
            <template slot-scope="scope">
              <span>{{ scope.row.state === 1 ? "已启用" : "已禁用" }}</span>
            </template>
          </el-table-column>

          <el-table-column label="操作" width="180">
            <!-- 操作插槽 -->
            <template slot-scope="scope">
              <el-button type="text" @click="stopState(scope.row)">{{
                scope.row.state === 1 ? "禁用" : "启用"
              }}</el-button>

              <el-button
                type="text"
                @click="editState(scope.row)"
                :disabled="scope.row.state === 0"
                >修改</el-button
              >

              <el-button
                type="text"
                @click="removeState(scope.row)"
                :disabled="scope.row.state === 0"
                >删除</el-button
              >
            </template>
            <!-- /操作插槽 -->
          </el-table-column>
        </el-table>
      </div>
      <!-- tab表格 -->

      <!-- 新增标签内容 -->
      <el-dialog title="新增目录" :visible.sync="addShow" width="28%">
        <!-- 所属学科 -->
        <el-form :model="form" label-width="80px">
          <el-form-item label="所属学科">
            <el-select
              v-model="addTagForm.subjectID"
              placeholder="请选择"
              style="width: 100%;"
            >
              <el-option
                v-for="item in simpleSubjectList"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              ></el-option>
            </el-select>
          </el-form-item>
        </el-form>

        <!-- 目录名称 -->
        <el-form
          :model="form"
          :rules="rules"
          ref="addShow"
          label-width="80px"
          class="demo-ruleForm"
        >
          <el-form-item label="目录名称" prop="tagName">
            <el-input
              v-model="form.tagName"
              placeholder="请输入目录名称"
            ></el-input>
          </el-form-item>
        </el-form>

        <!-- 取消/关注 -->
        <div slot="footer" class="dialog-footer">
          <el-button @click="addShow = false">取 消</el-button>
          <el-button type="primary" @click="addShowTrue">确 定</el-button>
        </div>
      </el-dialog>
      <!-- /新增标签内容 -->
    </div>
  </div>
</template>

<script>
import { list, changeState, add } from "@/api/hmmm/tags";
import { status } from "@/api/hmmm/constants";
import { simple } from "@/api/hmmm/subjects";

export default {
  name: "",
  components: {},
  props: {},
  data() {
    return {
      page: 1,
      pagesize: 5,
      // 标签名称
      tagsFrom: {
        tagName: "",
        state: null
      },
      // 新增标签的所属学科
      addTagForm: {
        subjectID: ""
      },
      // 获取tab表单数据
      tagsList: [], // 所有标签列表数据
      simpleSubjectList: [], //新增标签的学科列表
      status: status,
      // 新增标签内容
      addShow: false, // 控制新增标签的弹窗
      // 新增表单
      form: {
        name: "",
        region: ""
      },
      formLabelWidth: "120px",
      // 目录名称的验证规则
      rules: {
        name: [
          {
            type: "tagName",
            required: true,
            message: "请输入标签名称",
            trigger: "blur"
          },
          { min: 3, max: 5, message: "长度在 3 到 5 个字符", trigger: "blur" }
        ]
      }
    };
  },
  computed: {},
  watch: {},
  created() {
    this.getList();
  },
  mounted() {},
  methods: {
    // 获取标签列表
    async getList() {
      try {
        const { data } = await list({
          page: this.page,
          pagesize: this.pagesize,
          // ...this.tagsFrom
          tagName: this.tagsFrom.tagName
        });
        this.tagsList = data.items;
        // console.log(data);
        // this.$message.success("憨憨牛批，获取标签列表成功");
      } catch (error) {
        this.$message.error("憨憨小主，获取标签列表失败");
      }
    },

    // 清除
    onRemove() {
      this.tagsFrom.tagName = "";
      this.tagsFrom.status = "";
    },

    // 禁用启用状态
    async stopState(row) {
      try {
        await changeState({
          id: row.id,
          state: row.state === 0 ? 1 : 0
        });
        this.getList();
        this.$message.success("恭喜憨憨，改变按钮状态成功");
      } catch (error) {
        this.$message.error("你个憨憨，改变按钮状态失败");
      }
    },

    // 加载学科简单列表
    async locaList() {
      try {
        const { data } = await simple();
        this.simpleSubjectList = data;
        // console.log(data, 111);
        // this.$message.success("加载新增标签里的学科列表成功");
      } catch (err) {
        this.$message.err("加载新增标签里的学科列表失败");
      }
    },

    // 新增标签内容
    // 点击弹窗
    addShowTsgs() {
      this.locaList();
      this.addShow = true;
    },

    // 点击确定按钮
    async addShowTrue() {
      try {
        const { data } = await add({
          tagName: this.tagsFrom.tagName,
          subjectID: this.addTagForm.subjectID
        });
        console.log(data, 222);
        this.getList();
        this.addShow = false;
      } catch (err) {
        this.$message.err("加载新增标签里的学科列表失败");
      }
    },

    // 修改内容
    // 点击修改按钮

    async editState() {
      try {
      } catch (err) {
        this.$message.err("修改内容失败");
      }
    }
  }
};
</script>

<style scoped lang="scss">
.pages-tags {
  padding: 10px;
  .el-card {
    padding: 20px;
    background-color: #fff;
    .el-row {
      display: flex;
      justify-content: space-between;
      .el-col-18 {
        width: 75%;
      }
      .el-col-6 {
        width: 25%;
        text-align: right;
      }
    }
    .el-alert {
      padding: 0;
      margin-top: 15px;
    }
  }
}
</style>

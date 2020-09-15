<template>
  <div class="app-container">
    <div class="filter-container">
      <el-input
        v-model="selectPhone"
        placeholder="手机号"
        style="width: 200px;"
        class="filter-item"
        @keyup.enter.native="selectFun"
      />
      <el-button
        v-waves
        class="filter-item one_btn"
        type="primary"
        icon="el-icon-search"
        @click="selectFun"
      >查询</el-button>
      <el-button
        class="filter-item"
        style="margin-left: 10px;"
        type="primary"
        icon="el-icon-edit"
        @click="newMsg"
      >添加</el-button>
      <el-button
        v-waves
        :loading="downloadLoading"
        class="filter-item"
        type="primary"
        icon="el-icon-download"
        @click="handleDownload"
      >导出</el-button>
    </div>
    <el-table :data="list" border fit highlight-current-row style="width: 100%;">
      <el-table-column label="id" prop="id"></el-table-column>
      <el-table-column label="门店名" prop="name"></el-table-column>
      <el-table-column label="手机号" prop="phone"></el-table-column>
      <el-table-column label="快递公司">
        <template slot-scope="{row}">
          <el-tag>{{ row.kdgs }}</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="公司地址" min-width="150px">
        <template slot-scope="{row}">
          <span class="link-type">{{ row.addres }}</span>
        </template>
      </el-table-column>
      <el-table-column label="邀请码" prop="code"></el-table-column>
      <el-table-column label="状态">
        <template slot-scope="{row}">
          <el-tag :type="row.status | statusFilter">{{ row.status }}</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="审核图片">
        <template slot-scope="{row}">
          <el-link :underline="false" @click="seeImg(row)" type="primary">查看</el-link>
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center" width="230" class-name="small-padding fixed-width">
        <template slot-scope="{row,$index}">
          <el-button type="primary" size="mini" @click="handleUpdate(row,$index)">修改</el-button>
          <el-button
            v-if="row.status!='正常'"
            size="mini"
            type="success"
            @click="handleModifyStatus(row,'正常')"
          >正常</el-button>
          <el-button v-if="row.status!='禁用'" size="mini" @click="handleModifyStatus(row,'禁用')">禁用</el-button>
          <el-button
            v-if="row.status!='deleted'"
            size="mini"
            type="danger"
            @click="handleDelete(row,$index)"
          >删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-dialog title="修改" :visible.sync="editDialog">
      <el-form :model="edit" label-width="70px" style="width: 400px; margin-left:50px;">
        <el-form-item label="快递公司">
          <el-input v-model="edit.kdgs" :placeholder="edit.kdgs"></el-input>
        </el-form-item>
        <el-form-item label="公司地址">
          <el-input v-model="edit.addres" :placeholder="edit.addres"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="editDialog = false">取 消</el-button>
        <el-button type="primary" @click="listUpdata">提交</el-button>
      </div>
    </el-dialog>
    <el-dialog title="添加" :visible.sync="newEditDialog">
      <el-form
        ref="newEditref"
        :model="newEdit"
        label-width="120px"
        style="width: 400px; margin-left:50px;"
      >
        <el-form-item label="姓名" prop="name" :rules="[{ required: true, message: '姓名不能为空'}]">
          <el-input v-model="newEdit.name" :placeholder="newEdit.kdgs"></el-input>
        </el-form-item>
        <el-form-item
          label="手机号"
          prop="phone"
          :rules="[{ required: true, message: '手机号不能为空'},{ type: 'number', message: '手机号只能是数字'}]"
        >
          <el-input
            maxlength="11"
            minlength="11"
            v-model="newEdit.phone"
            :placeholder="newEdit.addres"
            @input="numFun"
          ></el-input>
        </el-form-item>
        <el-form-item label="快递公司" prop="kdgs" :rules="[{required:true,message:'所属公司不能为空'}]">
          <el-input v-model="newEdit.kdgs" :placeholder="newEdit.kdgs"></el-input>
        </el-form-item>
        <el-form-item label="公司地址" prop="addres" :rules="[{required:true,message:'公司地址不能为空'}]">
          <el-input v-model="newEdit.addres" :placeholder="newEdit.addres"></el-input>
        </el-form-item>
        <el-form-item label="状态">
          <el-select
            v-model="newEdit.state"
            placeholder="请选择状态"
            prop="state"
            @change="test"
            :rules="[{required:true,message:'状态不能为空'}]"
          >
            <el-option
              v-for="item in newEditSele"
              :key="item.id"
              :label="item.val"
              :value="item.id"
            ></el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="newEditDialog = false">取 消</el-button>
        <el-button type="primary" @click="newEditFun">提交</el-button>
      </div>
    </el-dialog>
    <el-dialog title="查看图片" :visible.sync="imgDialog" :fullscreen="false" top="3vh" close-on-press-escape='false'	>
      <el-row :gutter="10">
        <el-col :span="12">
          <p class="imgtit">图片1</p>
          <el-image :src="imgVal.img1" :preview-src-list="[imgVal.img1]" placeholder="图片错误" error="加载失败"></el-image>
        </el-col>
        <el-col :span="12">
          <p class="imgtit">图片2</p>
          <el-image :src="imgVal.img2" :preview-src-list="[imgVal.img2]" placeholder="图片错误" error="加载失败"></el-image>
        </el-col>
      </el-row>
      <el-row :gutter="10">
        <el-col :span="12">
          <p class="imgtit">图片3</p>
          <el-image :src="imgVal.img3" :preview-src-list="[imgVal.img3]" placeholder="图片错误" error="加载失败"></el-image>
        </el-col>
        <el-col :span="12">
          <p class="imgtit">图片4</p>
          <el-image :src="imgVal.img4" :preview-src-list="[imgVal.img4]" placeholder="图片错误" error="加载失败"></el-image>
        </el-col>
      </el-row>
      <div slot="footer" class="dialog-footer">
        <el-button @click="imgDialog = false">关 闭</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
import waves from "@/directive/waves"; // waves directive

export default {
  directives: { waves },
  filters: {
    statusFilter(status) {
      if (status === "正常") {
        return "success";
      } else {
        return "info";
      }
    },
  },
  data() {
    return {
      selectPhone: undefined,

      list: [
        {
          id: 1,
          name: "大蛤蟆",
          phone: "15215315222",
          addres: "山东省临沂市兰山区IEC国际企业中心",
          code: "3rg5d",
          kdgs: "申通快递",
          status: "正常",
          img1:
            "https://fuss10.elemecdn.com/e/5d/4a731a90594a4af544c0c25941171jpeg.jpeg",
          img2:
            "https://cube.elemecdn.com/6/94/4d3ea53c084bad6931a56d5158a48jpeg.jpeg",
          img3:
            "https://fuss10.elemecdn.com/a/3f/3302e58f9a181d2509f3dc0fa68b0jpeg.jpeg",
          img4:
            "https://fuss10.elemecdn.com/1/34/19aa98b1fcb2781c4fba33d850549jpeg.jpeg",
        },
        {
          id: 2,
          name: "大蛤蟆",
          phone: "15215315222",
          addres: "山东省临沂市兰山区IEC国际企业中心",
          code: "3rg5d",
          kdgs: "申通快递",
          status: "正常",
        },
      ],
      edit: {},
      newEdit: {
        name: "",
        phone: undefined,
        kdgs: "",
        addres: "",
        state: "",
      },
      newEditSele: [
        {
          id: 0,
          val: "禁用",
        },
        {
          id: 1,
          val: "正常",
        },
      ],
      imgVal: {},
      newEditDialog: false,
      editDialog: false,
      downloadLoading: false,
      imgDialog: false,
    };
  },
  methods: {
    seeImg(row) {
      this.imgDialog = true;
      this.imgVal.img1 = row.img1;
      this.imgVal.img2 = row.img2;
      this.imgVal.img3 = row.img3;
      this.imgVal.img4 = row.img4;
    },
    numFun(val) {
      this.newEdit.phone = Number(val);
    },
    test(val) {
      console.log(val);
    },
    selectFun() {
      console.log(this.selectPhone);
    },
    newMsg() {
      this.newEditDialog = true;
    },
    newEditFun() {
      this.$refs.newEditref.validate((vaild) => {
        if (vaild) {
          console.log(vaild);
        } else {
          console.table(vaild);
          console.log(this.newEdit);
        }
      });
    },
    handleUpdate(row, index) {
      this.edit = Object.assign({}, row);
      this.editDialog = true;
    },
    listUpdata() {
      const index = this.list.findIndex((v) => v.id === this.edit.id);
      this.list.splice(index, 1, this.edit);
      this.editDialog = false;
    },
    handleModifyStatus(row, status) {
      this.$message({
        message: "状态修改成功",
        type: "success",
      });
      row.status = status;
    },
    handleDelete(row, index) {
      this.$confirm("确认删除该商家吗?", "删除", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.list.splice(index, 1);
          this.$message({
            type: "success",
            message: "删除成功!",
          });
        })
        .catch(() => {});
    },
    handleDownload() {
      this.downloadLoading = true;
      import("@/vendor/Export2Excel").then((excel) => {
        const tHeader = [
          "id",
          "姓名",
          "手机号",
          "所属快递公司",
          "公司地址",
          "邀请码",
          "状态",
        ];
        const filterVal = [
          "id",
          "name",
          "phone",
          "kdgs",
          "addres",
          "code",
          "status",
        ];
        const data = this.formatJson(filterVal);
        excel.export_json_to_excel({
          header: tHeader,
          data,
          filename: "商家表",
        });
        this.downloadLoading = false;
      });
    },
    formatJson(filterVal) {
      return this.list.map((v) =>
        filterVal.map((j) => {
          if (j === "timestamp") {
            return parseTime(v[j]);
          } else {
            return v[j];
          }
        })
      );
    },
  },
};
</script>

<style scoped>
.one_btn {
  margin-left: 60px;
}
.imgtit {
  text-align: center;
  font-size: 20px;
}
</style>
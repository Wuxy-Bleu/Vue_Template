<template>
  <div class="app-container wuxy">
    <el-table
      v-loading="listLoading"
      :data="list"
      border
      fit
      highlight-current-row
      stripe
      style="width: 100%"
    >
      <el-table-column align="center" label="song" width="80" prop="songName">
        <!-- <span>{{ songName }}</span> -->
        <!-- <template slot-scope="{ row }">
          <span @click="testClick(row)">{{ row.songName }}</span>
        </template> -->
      </el-table-column>

      <el-table-column
        width="180px"
        align="center"
        label="singer"
        prop="singerName"
      >
        <!-- <template slot-scope="{ row }">
          <i class="el-icon-time"></i>
          <span>{{ row.singerName | parseTime("{y}-{m}-{d} {h}:{i}") }}</span>
        </template> -->
      </el-table-column>

      <el-table-column
        width="120px"
        align="center"
        label="album"
        prop="albumName"
      >
        <!-- <template slot-scope="{ row }">
          <span>{{ row.albumName }}</span>
        </template> -->
      </el-table-column>

      <!-- <el-table-column width="100px" label="Importance">
        <template slot-scope="{ row }">
          <svg-icon
            v-for="n in +row.importance"
            :key="n"
            icon-class="star"
            class="meta-item__icon"
          />
        </template>
      </el-table-column> -->

      <!-- <el-table-column class-name="status-col" label="Status" width="110">
        <template slot-scope="{ row }">
          <el-tag :type="row.status | statusFilter">
            {{ row.status }}
          </el-tag>
        </template>
      </el-table-column> -->

      <el-table-column min-width="300px" label="desc" prop="desc">
        <!-- <template slot-scope="{ row }">
          <template v-if="row.edit">
            <el-input v-model="row.title" class="edit-input" size="small" />
            <el-button
              class="cancel-btn"
              size="small"
              icon="el-icon-refresh"
              type="warning"
              @click="cancelEdit(row)"
            >
              cancel
            </el-button>
          </template>
          <span v-else>{{ row.title }}</span>
        </template> -->
      </el-table-column>

      <el-table-column align="center" label="upload cover" width="400px">
        <!-- <el-button type="primary" icon="el-icon-edit" circle></el-button> -->
        <el-upload
          class="upload-demo"
          drag
          action="https://jsonplaceholder.typicode.com/posts/"
          multiple
          show-file-list
          :before-upload="handleBeforeUpload"
          accept="image/jpeg,image/gif,image/png"
          :http-request="Upload"
        >
          <i class="el-icon-upload"></i>
          <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
          <!-- <div class="el-upload__tip" slot="tip">
            只能上传jpg/png文件，且不超过500kb
          </div> -->
        </el-upload>
        <!-- <template slot-scope="{ row }">
          <el-button
            v-if="row.edit"
            type="success"
            size="small"
            icon="el-icon-circle-check-outline"
            @click="confirmEdit(row)"
          >
            Ok
          </el-button>
          <el-button
            v-else
            type="primary"
            size="small"
            icon="el-icon-edit"
            @click="row.edit = !row.edit"
          >
            upload
          </el-button>
        </template> -->
      </el-table-column>

      <el-table-column
        width="180px"
        align="center"
        label="cover"
        prop="coverURL"
      >
      </el-table-column>
    </el-table>

    <el-button
      type="primary"
      @click="test02"
      style="margin-left: 10px; margin-top: 10px; margin-bottom: 10px;"
      >upload<i class="el-icon-upload el-icon--right"></i>
    </el-button>

    <el-dialog title="ADD" :visible.sync="dialogFormVisible">
      <el-form
        :model="ruleForm"
        status-icon
        :rules="rules"
        ref="ruleForm"
        label-width="100px"
        class="demo-ruleForm"
      >
        <el-form-item label="歌曲" prop="pass">
          <el-input
            v-model="ruleForm.pass"
            autocomplete="off"
          ></el-input>
        </el-form-item>
        <el-form-item label="歌手" prop="checkPass">
          <el-input v-model="ruleForm.checkPass" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="描述" prop="age">
          <el-input v-model.number="ruleForm.age"></el-input>
        </el-form-item>
        <el-form-item label="专辑">
          <el-select
            v-model="value"
            multiple
            filterable
            allow-create
            default-first-option
            placeholder="请选择专辑名"
          >
            <el-option
              v-for="item in options"
              :key="item.value"
              :label="item.label"
              :value="item.value"
            >
            </el-option>
          </el-select>
        </el-form-item>
      </el-form>

      <el-button type="primary" @click="getDialogData">test</el-button>

      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="dialogFormVisible = false"
          >确 定</el-button
        >
      </div>
    </el-dialog>

    <el-upload
      class="upload-demo"
      action="http://localhost"
      :on-preview="handlePreview"
      :on-remove="handleRemove"
      :before-remove="beforeRemove"
      multiple
      :limit="3"
      :on-exceed="handleExceed"
      :file-list="fileList"
      :before-upload="handleBeforeUpload"
    >
      <el-button size="small" type="primary">点击上传</el-button>
      <div slot="tip" class="el-upload__tip">
        只能上传jpg/png文件，且不超过500kb
      </div>
    </el-upload>
  </div>
</template>

<script>
import { fetchList } from "@/api/article";
import axios from "axios";
import request from "@/utils/request";
import OSS from "ali-oss";

/**
 * 生成随机文件名称
 * 规则八位随机字符，加下划线连接时间戳
 */
const getFileNameUUID = () => {
  function rx() {
    return (((1 + Math.random()) * 0x10000) | 0).toString(16).substring(1);
  }
  return `${+new Date()}_${rx()}${rx()}`;
};

export default {
  name: "InlineEditTable",
  filters: {
    statusFilter(status) {
      const statusMap = {
        published: "success",
        draft: "info",
        deleted: "danger"
      };
      return statusMap[status];
    }
  },
  data() {
    var checkAge = (rule, value, callback) => {
      if (!value) {
        return callback(new Error("年龄不能为空"));
      }
      setTimeout(() => {
        if (!Number.isInteger(value)) {
          callback(new Error("请输入数字值"));
        } else {
          if (value < 18) {
            callback(new Error("必须年满18岁"));
          } else {
            callback();
          }
        }
      }, 1000);
    };
    var validatePass = (rule, value, callback) => {
      if (value === "") {
        callback(new Error("请输入密码"));
      } else {
        if (this.ruleForm.checkPass !== "") {
          this.$refs.ruleForm.validateField("checkPass");
        }
        callback();
      }
    };
    var validatePass2 = (rule, value, callback) => {
      if (value === "") {
        callback(new Error("请再次输入密码"));
      } else if (value !== this.ruleForm.pass) {
        callback(new Error("两次输入密码不一致!"));
      } else {
        callback();
      }
    };
    return {
      list: null,
      listLoading: false,
      listQuery: {
        page: 1,
        limit: 2
      },
      fileList: [
        {
          name: "food.jpeg",
          url:
            "https://fuss10.elemecdn.com/3/63/4e7f3a15429bfda99bce42a18cdd1jpeg.jpeg?imageMogr2/thumbnail/360x360/format/webp/quality/100"
        },
        {
          name: "food2.jpeg",
          url:
            "https://fuss10.elemecdn.com/3/63/4e7f3a15429bfda99bce42a18cdd1jpeg.jpeg?imageMogr2/thumbnail/360x360/format/webp/quality/100"
        }
      ],
      ossSignature: null,
      dialogFormVisible: false,
      form: {
        name: "",
        region: "",
        date1: "",
        date2: "",
        delivery: false,
        type: [],
        resource: "",
        desc: ""
      },
      formLabelWidth: "120px",
      ruleForm: {
        pass: "",
        checkPass: "",
        age: ""
      },
      rules: {
        pass: [{ validator: validatePass, trigger: "blur" }],
        checkPass: [{ validator: validatePass2, trigger: "blur" }],
        age: [{ validator: checkAge, trigger: "blur" }]
      },
      options: [
        {
          value: "HTML",
          label: "HTML"
        },
        {
          value: "CSS",
          label: "CSS"
        },
        {
          value: "JavaScript",
          label: "JavaScript"
        }
      ],
      value: []
    };
  },
  created() {
    // this.getList();

    axios
      .get("/wuxy/musics")
      .then(response => {
        console.log(Array.from(response.data));
        console.log(Array.from(response.data).length);
        console.log(JSON.stringify(response.data));
        console.log(JSON.parse(JSON.stringify(response.data)));

        this.list = JSON.parse(JSON.stringify(response.data));
        // console.log(this)
      })
      .catch(function(error) {
        console.log(error);
      });

    // console.log('+++++++++++++++++++++++++++++++++++++++++');
    // const { data } = await fetchList(this.listQuery);

    // console.log(data.items)
    // console.log('+++++++++++++++++++++++++++++++++++++++++')

    axios
      .get("/upload/test")
      .then(response => {
        // console.log(response);
        this.ossSignature = response.data;
        console.log(this.ossSignature);
      })
      .catch(function(error) {
        console.log(error);
      });
  },
  methods: {
    getDialogData() {
      console.log(this.ruleForm);
    },
    // async getList() {
    //   this.listLoading = true;
    //   const { data } = await fetchList(this.listQuery);
    //   console.log(data.items);
    //   const items = data.items;
    //   this.list = items.map(v => {
    //     this.$set(v, "edit", false); // https://vuejs.org/v2/guide/reactivity.html
    //     v.originalTitle = v.title; //  will be used when user click the cancel botton
    //     return v;
    //   });
    //   // console.log(this.list);
    //   this.listLoading = false;
    // },
    cancelEdit(row) {
      console.log(typeof row);
      console.log("=====================");

      row.title = row.originalTitle;
      row.edit = false;
      this.$message({
        message: "The title has been restored to the original value",
        type: "warning"
      });
    },
    confirmEdit(row) {
      row.edit = false;
      row.originalTitle = row.title;
      this.$message({
        message: "The title has been edited",
        type: "success"
      });
    },
    testClick() {
      console.log(row);
    },
    test02() {
      this.dialogFormVisible = true;
    },

    handleRemove(file, fileList) {
      console.log(file, fileList);
    },
    handlePreview(file) {
      console.log(file);
    },
    handleExceed(files, fileList) {
      // this.$message.warning(
      //   `当前限制选择 3 个文件，本次选择了 ${
      //     files.length
      //   } 个文件，共选择了 ${files.length + fileList.length} 个文件`
      // );
    },
    beforeRemove(file, fileList) {
      return this.$confirm(`确定移除 ${file.name}？`);
    },
    handleBeforeUpload(file) {
      // console.log("before upload");
      // console.log(file);
    },
    Upload(file) {
      console.log("这个是最关键的了，点击上传会做什么");
      // console.log(file.file)

      // let client = new OSS({
      //   region: "oss-cn-shanghai",
      //   accessKeyId: "LTAI4G1KczRx4Aeju4eShCD5",
      //   accessKeySecret: "d4XFPpeR7p8QkLDKfUCJ61UE8jTpX6",
      //   bucket: "bleuweb-music",
      //   secure: true
      // });

      // let temporary = file.file.name.lastIndexOf(".");
      // let fileNameLength = file.file.name.length;
      // let fileFormat = file.file.name.substring(temporary + 1, fileNameLength);
      // let fileName = getFileNameUUID() + "." + fileFormat;
      // console.log(`image/${fileName}`);
      // // console.log(file.file)

      // client
      //   .multipartUpload(`image/${fileName}`, file.file)
      //   .then(result => {
      //     console.log(result);
      //   })
      //   .catch(err => {
      //     console.log("err:", err);
      //   });

      // console.log(file)
      // console.log(file.file)

      let temporary = file.file.name.lastIndexOf(".");
      let fileNameLength = file.file.name.length;
      let fileFormat = file.file.name.substring(temporary + 1, fileNameLength);
      let fileName = getFileNameUUID() + "." + fileFormat;

      //注意formData里append添加的键的大小写
      // console.log("+++++++++++++++++++++++++++++++++++");
      // console.log(this.ossSignature.accessid);
      // console.log("+++++++++++++++++++++++++++++++++++");

      let formData = new FormData();
      formData.append("key", this.ossSignature.dir + fileName); //存储在oss的文件路径
      formData.append("OSSAccessKeyId", this.ossSignature.accessid); //accessKeyId
      formData.append("policy", this.ossSignature.policy); //policy
      formData.append("Signature", this.ossSignature.signature); //签名
      // formData.append("callback", ossToken.callback); //回调
      formData.append("success_action_status", 200); //成功后返回的操作码
      //如果是base64文件，那么直接把base64字符串转成blob对象进行上传就可以了
      formData.append("file", file.file);
      // console.log("+++++++++++++++++++++++++++++++++++");
      // console.log(formData.getAll('OSSAccessKeyId'));
      // console.log("+++++++++++++++++++++++++++++++++++");
      axios
        .put(this.ossSignature.host, formData, {
          "Content-Type": "multipart/form-data"
        })
        .then(res => {
          console.log(res);
        });
    }
  }
};
</script>

<style scoped>
.edit-input {
  padding-right: 100px;
}
.cancel-btn {
  position: absolute;
  right: 15px;
  top: 10px;
}

.wuxy {
  background-color: chartreuse;
}
</style>

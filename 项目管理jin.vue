<template>
  <div class="editForm">
    <box-headr :title="title" :editType="editType" :pdfName="pdfName">
      <div>
        <p class="formTitle">一、 基础信息</p>
        <div class="line"></div>
        <el-form class="form" ref="form" label-width="141px">
          <el-form-item
            v-for="(item,index) in formList"
            :key="index"
            :label="item.label"
            :required="item.required"
            :class="[item.class]"
          >
            <el-select
              v-if="item.type == 'select'"
              :disabled="item.disabled"
              size="small"
              v-model="item.value"
              placeholder="请选择"
            >
              <el-option
                v-for="(opt,index) in item.options"
                :key="index"
                :value="opt.value"
                :label="opt.label"
              ></el-option>
            </el-select>
            <el-input
              v-else-if="item.type=='input'"
              :disabled="item.disabled"
              size="small"
              v-model="item.value"
            ></el-input>
            <el-input
              v-else
              :disabled="item.disabled"
              :type="item.type"
              :autosize="{ minRows: 2, maxRows: 5}"
              size="small"
              v-model="item.value"
            ></el-input>
          </el-form-item>
        </el-form>
      </div>
      <div v-for="(item,index) in itemList" :key="index">
        <p class="formTitle">{{item.title}}</p>
        <div class="line"></div>
        <file-edit
          :type="item.type"
          :id="recordId"
          :recordType="recordType"
          :files="fileList"
          ref="newFile"
        ></file-edit>
        <div v-if="item.type == 'check'" class="node">
          <p>
            <span style="color:#F56C6C;margin-right:4px;">*</span>审批意见:
          </p>
          <el-input
            size="small"
            type="textarea"
            :autosize="{ minRows: 3, maxRows: 6}"
            placeholder=""
            v-model="checkMessage"
          ></el-input>
        </div>
        <el-table
          v-else-if="item.type == 'table'"
          :data="item.tableData"
          border=""
          class="checkTable"
        >
          <el-table-column prop="auditUserName" label="" width="150">
            <template slot="header" slot-scope="scope">
              <p class="tableHeadr">
                <img src="../../assets/form_icon/shenpiren.png" class="headrIcon">审批人
              </p>
            </template>
          </el-table-column>
          <el-table-column prop="auditOpion" label="" width="">
            <template slot="header" slot-scope="scope">
              <p class="tableHeadr">
                <img src="../../assets/form_icon/yijian.png" class="headrIcon">审批意见
              </p>
            </template>
          </el-table-column>
          <el-table-column prop="auditDate" label="" width="200">
            <template slot="header" slot-scope="scope">
              <p class="tableHeadr">
                <img src="../../assets/form_icon/date.png" class="headrIcon">审批时间
              </p>
            </template>
            <template slot-scope="scope">
              <span>{{scope.row.auditDate.split(".")[0].replace("T"," ")}}</span>
            </template>
          </el-table-column>
        </el-table>
        <div v-else-if="item.type == 'node'" class="node">下一节点处理人:
          <el-select size="small" style="width:100px" :disabled="item.disabled" v-model="nodePerson" placeholder="请选择">
            <el-option
              v-for="(opt,index) in personList"
              :key="index"
              :value="opt.userName"
              :label="opt.userName"
            ></el-option>
          </el-select>
        </div>
        <el-form v-else-if="item.type == 'stop'" class="form" ref="form" label-width="120px">
          <el-form-item :label="item.label+ '申请人:'" class="rightItem">
            <el-input disabled size="small" v-model="item.form.person"></el-input>
          </el-form-item>
          <el-form-item :label="item.label+ '申请日期:'" class="leftItem">
            <el-input disabled size="small" v-model="item.form.date"></el-input>
          </el-form-item>
          <el-form-item :label="item.label+ '原因:'" required>
            <el-input
              :disabled="item.form.disabled"
              type="textarea"
              :autosize="{ minRows: 2, maxRows: 5}"
              size="small"
              v-model="item.form.reason"
            ></el-input>
          </el-form-item>
        </el-form>
        <el-form v-else-if="item.type == 'restart'" class="form" ref="form" label-width="120px">
          <el-form-item :label="item.label+ '申请人:'" class="rightItem">
            <el-input disabled size="small" v-model="item.form.person"></el-input>
          </el-form-item>
          <el-form-item :label="item.label+ '申请日期:'" class="leftItem">
            <el-input disabled size="small" v-model="item.form.date"></el-input>
          </el-form-item>
          <el-form-item :label="item.label+ '原因:'" required>
            <el-input
              :disabled="item.form.disabled"
              type="textarea"
              :autosize="{ minRows: 2, maxRows: 5}"
              size="small"
              v-model="item.form.reason"
            ></el-input>
          </el-form-item>
        </el-form>
      </div>
      <div
        v-show="editType == 'add'||editType == 'edit'"
        style="padding: 25px 0 15px 0;text-align: center"
      >
        <el-button type="primary" size="small" @click="commitForm(302)">保存为草稿</el-button>
        <span style="padding: 0 25px"></span>
        <el-button type="success" size="small" @click="commitForm(301)">提&emsp;交</el-button>
      </div>
      <div v-show="editType == 'check'" style="padding: 25px 0 15px 0;text-align: center">
        <el-button type="primary" size="small" @click="commitCheck(0)">审批不通过</el-button>
        <span style="padding: 0 25px"></span>
        <el-button type="success" size="small" @click="commitCheck(1)">审批通过</el-button>
      </div>
    </box-headr>
  </div>
</template>

<script>
import moment from "moment";
import jlProjectApi from "../../request/JLProjectApi";
import { mapActions, mapState, mapMutations } from "vuex";
require("moment/locale/zh-cn");
moment.locale("zh-cn");
import fileEdit from "../../components/fileEdit.vue";
import boxHeadr from "../../components/boxHeader.vue";

export default {
  name: "Form",
  data() {
    return {
      nodePerson:"",
      title: "",
      editType: "",
      recordType: "104",
      checkMessage: "",
      reason: "",
      recordId: "",
      formList: [
        {
          label: "工程名称:",
          value: "",
          options: [],
          required: true,
          disabled: false,
          type: "input",
          class: ""
        },
        {
          label: "报名时间:",
          value: "",
          options: [],
          required: true,
          disabled: false,
          type: "input",
          class: "rightItem"
        },
        {
          label: "公告时间:",
          value: "",
          options: [],
          required: true,
          disabled: false,
          type: "select",
          class: "leftItem"
        },
        {
          label: "开评标时间:",
          value: " ",
          options: [],
          required: true,
          disabled: false,
          type: "select",
          class: "rightItem"
        },
        {
          label: "资审时间:",
          value: " ",
          options: [],
          required: false,
          disabled: false,
          type: "select",
          class: "leftItem"
        },
        {
          label: "中标单位:",
          value: "",
          options: [],
          required: true,
          disabled: false,
          type: "input",
          class: "rightItem"
        },
        {
          label: "中标金额(元):",
          value: "",
          options: [],
          required: true,
          disabled: false,
          type: "input",
          class: "leftItem"
        },
        {
          label: "所属区域:",
          value: "",
          options: [],
          required: true,
          disabled: false,
          type: "select",
          class: "rightItem"
        },
        {
          label: "招标人:",
          value: "",
          options: [],
          required: true,
          disabled: false,
          type: "input",
          class: "leftItem"
        },
        {
          label: "支付方式:",
          value: "",
          options: [],
          required: false,
          disabled: false,
          type: "input",
          class: "rightItem"
        },
        {
          label: "施工单位联系方式:",
          value: "",
          options: [],
          required: false,
          disabled: false,
          type: "input",
          class: "leftItem"
        },
        {
          label: "设计单位:",
          value: "",
          options: [],
          required: true,
          disabled: false,
          type: "input",
          class: ""
        },
        {
          label: "设计单位联系人:",
          value: "",
          options: [],
          required: false,
          disabled: false,
          type: "input",
          class: "rightItem"
        },
        {
          label: "设计单位联系方式:",
          value: "",
          options: [],
          required: false,
          disabled: false,
          type: "input",
          class: "leftItem"
        },
        {
          label: "填报日期:",
          value: JSON.parse(sessionStorage.getItem("userData")).userName,
          options: [],
          required: false,
          disabled: true,
          type: "input",
          class: "rightItem"
        },
        {
          label: "填报人:",
          value: moment().format("YYYY年MM月DD日"),
          options: [],
          required: false,
          disabled: true,
          type: "input",
          class: "leftItem"
        }
      ],
      itemList: [],
      btnList: [],
      loading: false,
      fileList: [],
      personList:[]
    };
  },
  computed: {
    ...mapState(["supervisorDeptList", "cityList"])
  },
  components: {
    fileEdit,boxHeadr
  },
  created() {
    Object.keys(this.cityList).forEach(key => {
      let item = { label: this.cityList[key], value: key };
      this.formList[4].options.push(item);
    });
    this.formList[5].options = this.supervisorDeptList;
    this.formList.forEach(item => {
      item.disabled = true;
    });
    let path = this.$route.path;
    let data = this.$route.query;
    data.path = path;
    this.recordId = data.recordId ? data.recordId : "";
    // this.reason = data.recordReason;
    // this.formList[5].value = data.depName;
    // this.formList[4].value = data.areaName;
    if (path.indexOf("add") > -1) {
      this.recordId = "";
      let params = {
        projectName: data.projectName
      };
      jlProjectApi.getProjectDetail(params).then(resp => {
        // var data = Object.assign(
        // {},
        // resp.data.pmProjectBase,
        // resp.data.recordPromotes[0]
        // );
        // data.path = path;
        // data.recordPromotes = resp.data.recordPromotes;
        data.checkList = [];
        data.recordPause = resp.data.recordPauses[0];
        // data.recordRestart = resp.data.recordRestarts[0];
        // data.recordSettles = resp.data.recordSettles;
        this.initPage(data);
      });
      return;
    }

    let params = {
      pmProjectBase: { projectName: data.projectName },
      recordRestart: {
        recordId: data.recordId,
        recordType: this.recordType
      },
      userId:JSON.parse(sessionStorage.getItem("userData")).id
    };
    jlProjectApi.recordDetail(params).then(resp => {
      var data = Object.assign(
        {},
        resp.data.records[0].pmProjectBase,
        resp.data.records[0].recordPromote
      );
      data.path = path;
      data.checkList = resp.data.records[0].auditRecord;
      data.recordPause = resp.data.records[0].recordPause;
      data.recordRestart = resp.data.records[0].recordRestart;
      // this.reason = data.recordRestart.recordReason;
      resp.data.records[0].files.forEach(f => {
        f.name = f.fileName;
      });
      this.fileList = resp.data.records[0].files;
      this.initPage(data);
    });
  },
  methods: {
    ...mapActions(["setBreadcrumData"]),
    initPage(data) {
      let title = "项目启动";
      this.pdfName = data.projectName + title + "详情";
      if (data.path.indexOf("add") > -1) {
        this.title = "新增" + title + "申请";
        this.itemList = [
          {
            title: "二、 附件信息",
            type: "upload"
          },
          {
            title: "三、 暂停信息",
            type: "stop",
            label: "暂停",
            form: {
              person: data.recordPause.userName,
              date: data.recordPause.recordDate.split("T")[0],
              reason: data.recordPause.recordReason,
              disabled: true
            }
          },
          {
            title: "四、 启动信息",
            type: "restart",
            label: "启动",
            form: {
              person: JSON.parse(sessionStorage.getItem('userData')).userName,
              date: moment().format("YYYY-MM-DD"),
              reason: "",
              disabled: false
            }
          },
          {
            title: "五、 节点信息",
            type: "node",
            nodePerson: "王吉利"
          }
        ];

        this.editType = "add";
      } else if (data.path.indexOf("edit") > -1) {
        this.title = "修改" + title + "申请";
        this.itemList = [
          {
            title: "二、 附件信息",
            type: "upload"
          },
          {
            title: "三、 暂停信息",
            type: "stop",
            label: "暂停",
            form: {
              person: data.recordPause.userName,
              date: data.recordPause.recordDate.split("T")[0],
              reason: data.recordPause.recordReason,
              disabled: true
            }
          },
          {
            title: "四、 启动信息",
            type: "stop",
            label: "启动",
            form: {
              person: data.recordRestart.userName,
              date: data.recordRestart.recordDate.split("T")[0],
              reason: data.recordRestart.recordReason,
              disabled: false
            }
          },
          {
            title: "五、 节点信息",
            type: "node",
            nodePerson: "王吉利"
          }
        ];
        this.editType = "edit";
      } else if (data.path.indexOf("check") > -1) {
        this.title = "审批" + title + "申请";
        this.itemList = [
          {
            title: "二、 附件信息",
            type: "download"
          },
          {
            title: "三、 暂停信息",
            type: "stop",
            label: "暂停",
            form: {
              person: data.recordPause.userName,
              date: data.recordPause.recordDate.split("T")[0],
              reason: data.recordPause.recordReason,
              disabled: true
            }
          },
          {
            title: "四、 启动信息",
            type: "stop",
            label: "启动",
            form: {
              person: data.recordRestart.userName,
              date: data.recordRestart.recordDate.split("T")[0],
              reason: data.recordRestart.recordReason,
              disabled: true
            }
          },
          {
            title: "五、 审批信息",
            type: "check"
          }
        ];
        this.editType = "check";
      } else if (data.path.indexOf("detail") > -1) {
        this.title = title + "";
        this.itemList = [
          {
            title: "二、 附件信息",
            type: "download"
          },
          {
            title: "三、 暂停信息",
            type: "stop",
            label: "暂停",
            form: {
              person: data.recordPause.userName,
              date: data.recordPause.recordDate.split("T")[0],
              reason: data.recordPause.recordReason,
              disabled: true
            }
          },
          {
            title: "四、 启动信息",
            type: "stop",
            label: "启动",
            form: {
              person: data.recordRestart.userName,
              date: data.recordRestart.recordDate.split("T")[0],
              reason: data.recordRestart.recordReason,
              disabled: true
            }
          },
          {
            title: "五、 审批信息",
            type: "table",
            tableData: data.checkList
          }
          // {
          //   title:'四、 节点信息',
          //   type:'node',
          //   nodePerson:'王吉利',
          // },
        ];
        if (data.recordRestart.auditStatus !== "402") {
          this.itemList.push({
            title: "六、 节点信息",
            type: "node",
            disabled: true,
            nodePerson: "王吉利"
          });
        }
        this.editType = "edtail";
      }
      if (data.path.indexOf("check") == -1) {
        let userId = JSON.parse(sessionStorage.getItem("userData")).id;
        jlProjectApi
          .getRecordReviewerByUserId({ userId: userId })
          .then(resp => {
            if (resp.data && resp.data.length) {
              this.personList = resp.data;
              if(data.recordRestart && data.recordRestart.reviewer && data.recordRestart.reviewer !== "") {
                this.nodePerson = data.recordRestart.reviewer;
              }else {
                this.nodePerson = resp.data[0].userName;
              }
            }
          });
      }
      this.formList[0].value = data.projectName;
      this.formList[1].value = data.projectPeriod;
      this.formList[2].value = data.projectAmount;
      this.formList[3].value = data.projectSituation;
      this.formList[4].value = data.projectArea;
      this.formList[5].value = data.supervisionDepartment;

      this.formList[6].value = data.ownerUnit;
      this.formList[7].value = data.ownerUnitContacter;
      this.formList[8].value = data.ownerUnitPhone;

      this.formList[9].value = data.constructUnit;
      this.formList[10].value = data.constructUnitContacter;
      this.formList[11].value = data.constructUnitPhone;

      this.formList[12].value = data.designUnit;
      this.formList[13].value = data.designUnitContacter;
      this.formList[14].value = data.designUnitPhone;
      this.formList[15].value = data.userName;
      this.formList[16].value = data.recordDate.split("T")[0];
    },
    commitForm(val) {
      let params = {
        pmProjectBase: {
          projectName: this.formList[0].value
        },
        recordRestart: {
          recordId: this.recordId,
          userId: JSON.parse(sessionStorage.getItem("userData")).id,
          recordReason: this.itemList[2].form.reason,
          sendStatus: val,
          reviewer: this.nodePerson
        }
      };
      jlProjectApi.projectRestart(params).then(resp => {
        if (parseInt(resp.status) == 200) {
          this.$message({
            type: "success",
            message: "保存成功!"
          });
          this.recordId = resp.data;
          this.$refs.newFile[0].updateFile(val);
        }
      });
    },
    commitCheck(val) {
      if (!this.checkMessage || this.checkMessage == '') {
          this.$message({
            type: "warning",
            message: "请填写审批意见!"
          });
          return;
      }
      let params = {
        auditRecord: [
          {
            projectName: this.formList[0].value,
            recordId: this.recordId,
            recordType: this.recordType,
            auditUserId: JSON.parse(sessionStorage.getItem("userData")).id,
            auditUserName: JSON.parse(sessionStorage.getItem("userData")).userName,
            auditOpion: this.checkMessage,
            auditPass: val
          }
        ]
      };
      jlProjectApi.projectCheck(params).then(resp => {
        // debugger;
        if (resp.status == 200) {
          this.$message({
            type: "success",
            message: "审批信息提交成功!"
          });
          var routers = this.$store.state.breadcrumbData;
          var index = routers.length - 2;
          var data = [];
          var router = routers[index];

          for (var i = 0; i < index; i++) {
            data.push(routers[i]);
          }
          this.setBreadcrumData(data);
          this.$router.push({ path: router.path });
        }
      });
    }
  }
};
</script>




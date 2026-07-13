<template>
  <div id="app">
    <!-- <div class="title">
      <a
        href="https://github.com/zyqwst/json-schema-editor-vue"
        target="_blank"
      >json-schema-editor-vue</a>
      <span class="version">version：{{ version }}</span>
    </div>-->
    <div class="desc">
      <div>
        <a @click="visible = true">import json</a>
      </div>
    </div>
    <div class="container">
      <!-- <codemirror class="code" v-model="jsonStr" :readOnly="false" /> -->
      <!-- 原 -->
      <json-schema-editor
        class="schema"
        :value="tree"
        disabledType
        lang="zh_CN"
        custom
        :extra="extraSetting"
        :showSensitive="showSensitive"
        @sensitive-click="handleSensitive"
      />
    </div>
    <a-modal
      v-model="visible"
      title="import json"
      width="800px"
      height="600x"
      @ok="handleImportJson"
    >
      <div class="code-container">
        <codemirror class="code" v-model="importJson" :readOnly="false" />
      </div>
    </a-modal>
  </div>
</template>

<script>
var app = require("../package.json");
import Codemirror from "./components/Codemirror.vue";
import GenerateSchema from "generate-schema";
export default {
  name: "App",
  components: { Codemirror },
  provide() {
    return {
      handleSensitiveGlobal: this.handleSensitive
    };
  },
  computed: {
    jsonStr: {
      get: function() {
        return JSON.stringify(this.tree, null, 2);
      },
      set: function(newVal) {
        this.tree = JSON.parse(newVal);
      }
    }
  },
  data() {
    return {
      version: app.version,
      importJson: "",
      visible: false,
      extraSetting: {
        // integer: {
        //   default: {
        //     name: "默认值",
        //     type: "integer"
        //   }
        // },
        // string: {
        //   default: {
        //     name: "默认值",
        //     type: "integer"
        //   }
        // }
      },
      tree: {
        root: {
          type: "object",
          title: "条件",
          remark: "备注",
          properties: {
            name: {
              type: "string",
              title: "名称",
              remark: "备注",
              maxLength: 10,
              minLength: 2
            },
            appId: {
              type: "integer",
              title: "应用ID",
              remark: "备注",

              default: 3
            },
            credate: {
              type: "string",
              title: "创建日期",
              remark: "备注",
              format: "date"
            }
          },
          required: ["name", "appId", "credate"]
        }
      },
      showSensitive: true
    };
  },
  methods: {
    // 原
    // handleImportJson() {
    // console.log(this.importJson,'importJson')
    // const t = GenerateSchema.json(JSON.parse(this.importJson));
    // delete t.$schema;
    // this.tree.root = t;
    // this.visible = false;
    // },
    // 新
handleImportJson() {
  console.log(this.importJson,'importJson')
  const rawData = JSON.parse(this.importJson)
  const t = GenerateSchema.json(rawData)
  delete t.$schema

  function injectDefault(schema, data) {
    // 对象类型：标记禁用，不赋值default
    if (schema.type === 'object' && data && typeof data === 'object') {
      schema.disabledInput = true // 自定义禁用标记
      for (const key in schema.properties) {
        injectDefault(schema.properties[key], data[key])
      }
    } 
    // 数组类型：标记禁用，不赋值default
    else if (schema.type === 'array' && Array.isArray(data)) {
      schema.disabledInput = true // 自定义禁用标记
      if (schema.items) {
        data.forEach(item => injectDefault(schema.items, item))
      }
    } 
    // 基础类型：正常赋值默认值，不禁用
    else {
      schema.default = data
      schema.disabledInput = false
    }
  }

  injectDefault(t, rawData)
  this.tree.root = t
  this.visible = false
},
    // 敏感信息方法
    handleSensitive(value) {
      console.log(value, "schema, parent11");
      console.log('测试git')
      // 示例：弹出对话框，修改当前节点的 title 和 remark
      // this.$dialog.confirm({
      //   title: `编辑敏感字段：${value.key}`,
      //   content: `
      //     <input v-model="tempTitle" placeholder="title" />
      //     <input v-model="tempRemark" placeholder="remark" />
      //   `,
      //   onConfirm: () => {
      //     value.updateSchema({
      //       title: this.tempTitle,
      //       remark: this.tempRemark
      //     });
      //   }
      // });
    }
  }
};
</script>
<style>
* {
  padding: 0;
  margin: 0;
  box-sizing: border-box;
}
.title {
  text-align: center;
  font-size: 40px;
  font-weight: bold;
  height: 100px;
  line-height: 100px;
}
.version {
  font-size: 16px;
}
.desc {
  padding: 20px;
  width: 80vw;
  min-width: 800px;
  margin: auto;
  padding: 0 3em;
  font-size: 1.2em;
}
.container {
  /* display: flex;
  padding: 20px;
  width: 80vw;
  min-width: 800px;
  justify-content: center;
  height: calc(100vh - 150px);
  margin: auto; */
  height: calc(100vh - 150px);

}
.code-container {
  max-height: 600px;
  overflow: auto;
}
 .schema {
  margin-left: 20px;
  width: 80%;
  height: 100%;
  overflow-y: auto;
  overflow-x: hidden;
  border: 1px solid rgba(0, 0, 0, 0.1);
  border-radius: 8px;
  padding: 12px;
}

.CodeMirror {
  height: 100% !important;
}
.vue-codemirror {
  flex: 1;
  margin: 0 24px;
  border: 1px solid rgba(0, 0, 0, 0.1);
  min-height: 300px;
  overflow: auto;
  border-radius: 6px;
}
</style>

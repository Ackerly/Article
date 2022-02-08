# 10个Element-UI技巧
## el-scrollbar 滚动条
``` 
<el-scrollbar>
  <div class="box">
    <p v-for="item in 15" :key="item">欢迎使用 el-scrollbar {{item}}</p>
  </div>
</el-scrollbar>

<style scoped>
.el-scrollbar {
  border: 1px solid #ddd;
  height: 200px;
}
.el-scrollbar ::v-deep  .el-scrollbar__wrap {
    overflow-y: scroll;
    overflow-x: hidden;
  }
</style>
```
## el-upload 模拟点击
想用 el-upload 的上传功能，但又不想用 el-upload 的样式，如何实现呢？隐藏 el-upload，然后再模拟点击就可以了。
``` 
<button @click="handleUpload">上传文件</button>
<el-upload
  v-show="false"
  class="upload-resource"
  multiple
  action=""
  :http-request="clickUploadFile"
  ref="upload"
  :on-success="uploadSuccess"
>
    上传本地文件
</el-upload>

<script>
export default {
    methods: {
        // 模拟点击
        handleUpload() {
            document.querySelector(".upload-resource .el-upload").click()
        },
        // 上传文件
        async clickUploadFile(file) {
            const formData = new FormData()
            formData.append('file', file.file)
            const res = await api.post(`xxx`, formData)
        }
        // 上传成功后，清空组件自带的文件列表
        uploadSuccess() {
            this.$refs.upload.clearFiles()
        }
    }
}
</script>
```
## el-select 下拉框选项过长  
很多时候下拉框的内容是不可控的，如果下拉框选项内容过长，势必会导致页面非常不协调，解决办法就是，单行省略加文字提示。  
``` 
<el-select popper-class="popper-class" :popper-append-to-body="false" v-model="value" placeholder="请选择">
    <el-option
      v-for="item in options"
      :key="item.value"
      :label="item.label"
      :value="item.value"
    >
        <el-tooltip
          placement="top"
          :disabled="item.label.length<17"
        >
            <div slot="content">
                <span>{{item.label}}</span>
            </div>
            <div class="iclass-text-ellipsis">{{ item.label }}</div>
        </el-tooltip>
    </el-option>
</el-select>

<script>
  export default {
    data() {
      return {
        options: [{
          value: '选项1',
          label: '黄金糕黄金糕黄金糕黄金糕黄金糕黄金糕黄金糕黄金糕黄金糕'
        }, {
          value: '选项2',
          label: '双皮奶双皮奶双皮奶双皮奶双皮奶双皮奶双皮奶双皮奶双皮奶'
        }, {
          value: '选项3',
          label: '蚵仔煎蚵仔煎蚵仔煎蚵仔煎蚵仔煎蚵仔煎蚵仔煎蚵仔煎蚵仔煎'
        }],
        value: ''
      }
    }
  }
</script>

<style scoped>
.el-select {
  width: 300px;
}
.el-select ::v-deep .popper-class {
  width: 300px;
}
.iclass-text-ellipsis {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
</style>
```
## el-input 首尾不能为空格
``` 
<el-form :rules="rules" :model="form" label-width="80px">
    <el-form-item label="活动名称" prop="name">
        <el-input v-model="form.name"></el-input>
    </el-form-item>
</el-form>

<script>
export default {
    data() {
        return {
            form: {
                name: ''
            },
            rules: {
                name: [
                        { required: true, message: '请输入活动名称', trigger: 'blur'},
                        { pattern: /^(?!\s+).*(?<!\s)$/,  message: '首尾不能为空格', trigger: 'blur' }
                ]
            }
        }
    }
}
</script>
```
## el-input type=number 输入中文，焦点上移
当 el-input 设置 type="number" 时，输入中文，虽然中文不会显示出来，但焦点会上移。
``` 
<style scoped>
::v-deep .el-input__inner {
    line-height: 1px !important;
}
</style>
```
## el-input type=number 去除聚焦时的上下箭头
``` 
<el-input class="clear-number-input" type="number"></el-input>

<style scoped>
.clear-number-input ::v-deep input[type="number"]::-webkit-outer-spin-button,
.clear-number-input ::v-deep input[type="number"]::-webkit-inner-spin-button {
    -webkit-appearance: none !important;
}
</style>
```
## el-form 只校验表单其中一个字段
``` 
this.$refs.form.validateField('name', valid => {
    if (valid) { 
        console.log('send!'); 
    } else { 
        console.log('error send!'); 
        return false; 
    }
})
```
## el-dialog 重新打开弹窗，清除表单信息
``` 
<el-dialog @closed="resetForm">
    <el-form ref="form">
    </el-form>
</el-dialog>

<script>
export default {
    methods: {
        resetForm() {
            this.$refs.form.resetFields()
        }
    }
}
</script>
```
## el-dialog 的 destroy-on-close 属性设置无效
``` 
<el-dialog :visible.sync="visible" v-if="visible" destroy-on-close>
</el-dialog>
```
## el-table 表格内容超出省略
``` 
<el-table-column
  prop="address"
  label="地址"
  width="180"
  show-overflow-tooltip
>
</el-table-column>
```

参考：  
[Element-UI 10个奇淫技巧，你知道几个？](https://juejin.cn/post/7017957779176423454?utm_source=gold_browser_extension)

<template>
  <div id="app">
    <el-collapse v-model="activeName" accordion>
      <el-collapse-item title="GED Computation" name="1">
        <el-divider>File Upload</el-divider>
        <el-row>
          <el-upload
            class="upload-demo"
            ref="upload"
            name="files"
            action=""
            multiple
            :limit="2"
            :on-exceed="handleExceed"
            :on-change="handleChange"
            :http-request="uploadFile"
            :file-list="fileList"
            :auto-upload="false"
            :drag="true">
            <el-button slot="trigger" type="success">Choose File</el-button>
            <el-button type="success" @click="submitUpload">Upload<i class="el-icon-upload el-icon--right"></i></el-button>
            <div slot="tip" class="el-upload__tip">Receive only files with the suffix GXL or GEXF.</div>
          </el-upload>
        </el-row>
        <el-divider>Preview</el-divider>
        <img :src="processImgSrc1" alt="Graph1" />
        <img :src="processImgSrc2" alt="Graph2" />
        <el-divider>Method</el-divider>
        <el-row>
          <el-col :span="16">
          <el-cascader-panel :options="options" v-model="value" :clearable="true" @change="changeDisable"></el-cascader-panel>
          </el-col>
          <el-col :span="4" :offset="2">
            <div class="sub-title">Beam Size Input</div>
            <el-input v-model="beamSize" placeholder="Please input beamsize." clearable></el-input>
          </el-col>
        </el-row>
        <el-divider>Action</el-divider>
        <el-row>
          <el-popconfirm
            confirm-button-text='OK'
            cancel-button-text='Quit'
            icon="el-icon-info"
            icon-color="red"
            title="Are you sure to clear your selection？"
            @confirm="refresh"
          >
            <el-button type="danger" slot="reference" icon="el-icon-delete">Clear</el-button>
          </el-popconfirm>
<!--          <el-button type="primary" @click="sendQuery">Confirm</el-button>-->
          <el-button type="primary" :disabled="isAble" @click="sendQuery">Confirm</el-button>
        </el-row>
        <el-divider>Result</el-divider>
        <el-row>
          <el-col :span="16" :offset="4">
            <el-card :body-style="{ padding: '0px' }">
              <img :src="editPath" alt="EditPath" />
              <div style="padding: 14px;">
                <span :key="editCost">{{'GED is ' + editCost }}</span>
                <div class="bottom clearfix">
                  <time class="time">{{ currentDate }}</time>
                </div>
              </div>
            </el-card>
          </el-col>
        </el-row>
      </el-collapse-item>
      <el-collapse-item title="Model Training" name="2">
        <div>控制反馈：通过界面样式和交互动效让用户可以清晰的感知自己的操作；</div>
        <div>页面反馈：操作后，通过页面元素的变化清晰地展现当前状态。</div>
      </el-collapse-item>
    </el-collapse>
  </div>
</template>

<script>
import Axios from 'axios'
export default {
  data () {
    return {
      activeName: '1',
      value: [],
      fileList: [],
      processImgSrc1: '',
      processImgSrc2: '',
      isAble: true,
      beamSize: 100,
      options: [{
        value: 'A* Algorithm',
        label: 'A* Algorithm',
        children: [{
          value: 'Beam',
          label: 'Beam',
          children: [{
            value: 'BFS',
            label: 'BFS'
          }, {
            value: 'Label Set',
            label: 'Label Set'
          }, {
            value: 'Star Match',
            label: 'Star Match'
          }, {
            value: 'Branch Match',
            label: 'Branch Match'
          }, {
            value: 'Noah',
            label: 'Noah'
          }]
        }, {
          value: 'Normal',
          label: 'Normal',
          children: [{
            value: 'BFS',
            label: 'BFS'
          }, {
            value: 'Label Set',
            label: 'Label Set'
          }, {
            value: 'Star Match',
            label: 'Star Match'
          }, {
            value: 'Branch Match',
            label: 'Branch Match'
          }, {
            value: 'Noah',
            label: 'Noah'
          }]
        }]
      }, {
        value: 'Bipartite Graph Algorithm',
        label: 'Bipartite Graph Algorithm',
        children: [{
          value: 'Hungarian',
          label: 'Hungarian'
        }, {
          value: 'VJ',
          label: 'VJ'
        }]
      }, {
        value: 'End-to-End Neural Networks',
        label: 'End-to-End Neural Networks',
        children: [{
          value: 'SimGNN',
          label: 'SimGNN',
          children: [
            {
              value: 'Histogram',
              label: 'Histogram'
            },
            {
              value: 'None',
              label: 'None'
            }
          ]
        }, {
          value: 'GMN',
          label: 'GMN'
        }, {
          value: 'Customize',
          label: 'Customize'
        }]
      }],
      currentDate: new Date(),
      editPath: '',
      editCost: ''
    }
  },
  methods: {
    refresh () {
      Object.assign(this.$data, this.$options.data())
    },
    handleChange (file, fileList) {
      this.fileList = fileList
    },
    submitUpload (file) {
      this.$refs.upload.submit()
      let formData = new FormData()
      formData.append('username', 'Lei Yang')
      this.fileList.forEach(file => {
        formData.append('files', file.raw)
        formData.append('filename', file.name)
      })
      console.log(formData)
      Axios({
        method: 'post',
        url: '/upload',
        headers: {'Content-Type': 'multipart/form-data'},
        data: formData
      })
        .then(response => {
          if (response.data) {
            // console.log(response.data)
            let base64Str = this.clear(response.data)
            if (base64Str.length === 2) {
              this.processImgSrc1 = 'data:image/png;base64,' + base64Str[0]
              this.processImgSrc2 = 'data:image/png;base64,' + base64Str[1]
            } else {
              if (this.processImgSrc1.length === 0) {
                this.processImgSrc1 = 'data:image/png;base64,' + base64Str
              } else {
                this.processImgSrc2 = 'data:image/png;base64,' + base64Str
              }
            }
            if (this.value.length !== 0) {
              this.isAble = false
            }
            // this.image_preview = this.dataURItoBlob(response.data)
            // this.processImgSrc = 'data:image/png;base64,' + btoa(new Uint8Array(response.data).reduce((data, byte) => data + String.fromCharCode(byte), ''))
          }
        }).catch(err => {
          // alert('请求失败')
          console.log(err.data)
        })
    },
    handleExceed (files) {
      this.$message.warning(`The number of files are limited to 2!`)
    },
    uploadFile (file) {
      console.log(file)
    },
    clear (buffer) {
      buffer = buffer.replace(/\n/g, '')
      if (buffer.indexOf(',') !== -1) {
        var str = buffer.split(/,/)
        return str
      }
      return buffer
    },
    changeDisable (value) {
      if (value.length === 0) {
        this.isAble = true
      } else {
        if (this.processImgSrc1.length !== 0 & this.processImgSrc2.length !== 0) {
          this.isAble = false
          console.log(this.isAble)
        }
      }
      this.value = value
      console.log(this.value)
    },
    sendQuery () {
      var data = {}
      data['alg'] = this.value[0]
      data['method'] = this.value[1]
      if (this.value.length === 3) {
        data['function'] = this.value[2]
      }
      if (this.value[1] === 'Beam') {
        data['beamsize'] = this.beamSize
      }
      Axios({
        method: 'post',
        url: '/query',
        headers: {'Content-Type': 'application/json;charset=utf-8'},
        data: data
      })
        .then(response => {
          if (response.data) {
            // console.log(response.data)
            let base64Str = this.clear(response.data)
            // var newData = {cost: base64Str[1], method: this.value, path: '0'}
            // this.resultData.append(newData)
            // console.log(this.resultData)
            this.editCost = base64Str[1]
            this.editPath = 'data:image/png;base64,' + base64Str[0]
          }
        }).catch(err => {
          // alert('请求失败')
          console.log(err.data)
        })
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>

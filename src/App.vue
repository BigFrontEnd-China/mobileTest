<template>
  <div id="app">
    <el-container style="height:100%">
      <el-header>
        <span class="icon"></span>
      </el-header>
      <el-main>
        <div class="images">
          <el-image :src="imageUrl">
              <div slot="error" style="margin-top:130px;">
                暂无图片
              </div>
          </el-image>
        </div>
        <div class="text-container">
          <span v-show="text">识别结果:&nbsp;&nbsp;</span>
          <span style="font-weight: 600;">{{text}}</span>
        </div>
      </el-main>
      <el-footer>
        <div tabindex="0" class="el-upload el-upload--text">
          <i class="el-icon-plus"></i>
          <input type="file" name="file" class="el-upload__input" capture="camera" accept="image/png,image/jpeg" @change="upload($event)">
        </div>
      </el-footer>
    </el-container>
  </div>
</template>

<script>
export default {
  name: 'app', 
  data() {
      return {
        imageUrl: '',
        text: ''
      };
    },
  methods: {
    showImage(res, file) {
      this.imageUrl = URL.createObjectURL(file.raw);
    },
    upload(e) {  
      let _this = this
      _this.text = ''
      let file = e.target.files[0];
      const isLt2M = file.size / 1024 / 1024 < 4;
      if (file.type !== 'image/jpeg' && file.type !== 'image/png') {
        this.$message.error('上传图片只能是 JPG 格式!');
        return
      }
      if (!isLt2M) {
        this.$message.error('上传图片大小不能超过 4MB!');
        return
      }
      let reader = new FileReader()
      reader.readAsDataURL(file); // 读出 base64
      reader.onloadend = function () {
        // 图片的 base64 格式, 可以直接当成 img 的 src 属性值        
        _this.imageUrl = reader.result
        if (_this.imageUrl) {
          let params = {
            imgBase64: _this.imageUrl.replace('data:image/jpeg;base64,','')
          }
          let xhr = new XMLHttpRequest()
          xhr.open('post', '/MiddlewareServer/EmoR/img', true)
          xhr.setRequestHeader('Content-Type', 'application/json')
          xhr.setRequestHeader('X-Auth-Token', 'kdxf_REG62S6BDFT_53425TFDH')
          xhr.send(JSON.stringify(params))   
          xhr.onreadystatechange = function(){
            // 通信成功时，状态值为4
            if (xhr.readyState === 4){
              if (xhr.status === 200){
                let data = JSON.parse(xhr.response)
                let code = data.result.code
                if (code == '1') {
                  if (data.result.details && data.result.details.length >0) {
                    _this.text = data.result.details[0].emotionDesc
                  }
                } else {
                  if (code == '-4') {
                    _this.text = '未知'
                  } else {
                    _this.text = '识别失败'
                  }
                }
              } else {
                _this.$message.error(xhr.statusText)
              }
            }
          }
          xhr.onerror = function () {
            _this.$message.error(xhr.statusText);
          }
        }
      }
    }
  },
  mounted() {}
}
</script>

<style>
html, body {
  width: 100%;
  height: 100%;
  background: #409eff;
}
html, body, div {
  margin: 0px;
  padding: 0px;
}
#app {
  width: 100%;
  height: 100%;
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #fff;
}
.el-main {
  padding: 0px;
  text-align: center;
}
.images {
  margin-top: 4rem;
  width: 100%;
  height: 20rem;
  padding: 0px 4rem;
  box-sizing: border-box;
}
.el-image {
  width: 100%; 
  height: 100%; 
  background-color:#fff; 
  color:#999;
  border-radius: 4px;
  box-shadow: 1px 1px 5px #666;
}
.text-container {
  height: 4rem;
  line-height: 4rem;
  font-size: 1.2rem;
}
.el-upload {
  width: 40px;
  height: 40px;
  line-height: 40px;
  background-color: #eee;
  border-radius: 50%;
}
.el-header {
  background-color: #3a8ee6;
}
.el-header .icon {
  display: inline-block;
  width: 155px;
  height: 40px;
  margin: 10px 0;
  background: url('../src/assets/logo.png') no-repeat;
}
.el-header, .el-footer {
  height: 4rem;
  line-height: 4rem;
  text-align: center;
}
.el-icon-plus {
  color: #3a8ee6;
}
.el-upload__input {
  position: relative;
  top: -40px;
  display: block;
  height: 40px;
  width: 40px;
  opacity: 0;
}
</style>

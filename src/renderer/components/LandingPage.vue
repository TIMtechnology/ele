<template>
  <div id="wrapper">
    <main>
      <div >
         <el-form ref="form" :model="form" label-width="80px">
          <el-form-item label="用户名">
            <el-input v-model="form.username"></el-input>
          </el-form-item>
          <el-form-item label="密码">
            <el-input v-model="form.password" show-password></el-input>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="onSubmit">立即登陆</el-button>
            <el-button @click="startDwonLoad">开始批量下载</el-button>
          </el-form-item>
          
        </el-form>


        <el-select v-model="printername" placeholder="请选择打印机"> 
          <el-option v-for="item in PrinterList" >
            {{ item }}
          </el-option>
        </el-select>
        <el-select v-model="printmode" placeholder="请选择打印模式"> 
          <el-option v-for="item in ModeList" >
            {{ item }}
          </el-option>
        </el-select>
      
        <el-button @click="startPrint">开始打印</el-button>
         
      </div>

    </main>
  </div>
</template>

<script>
export default {
  name: "landing-page",
  data() {
    return {
      PrinterList: {},
      printername:'',
      printmode:'正面',
      ModeList:['正面','反面','逆序反面'],
      form: {
        username: "",
        password: "",
      },
      fileList: [],
    };
  },
  methods: {
    open(link) {
      this.$electron.shell.openExternal(link);
    },
    onSubmit() {
      this.$http
        .post("http://synu.yingxin.timkj.cn/admin.php/index/login_api", {
          ...this.form,
        })
        .then((s) => {
          console.log(s);
          var data = s.data;
          if (data.msg) {
            alert(data.msg);
          } else {
            this.fileList = data;
          }
        });
    },
    change(e){
      console.log(e)
      this.printername =  e
    }
    ,
    startDwonLoad() {
      console.log(this.fileList);
      alert("共计需要下载:"+this.fileList.length+"个文件，请等待文件下载完成后再执行打印任务。");
      //调用接口进行下载

      // this.$http({
      //   url: "http://127.0.0.1:12345/local/download",
      //   data: { file_urls: this.fileList },
      //   methods: "POST",
      //   headers: {
      //     "Content-Type": "application/json",
      //   },
      // }).then((e) => {
      //   console.log(e);
      // });

      this.$http
        .post(
          "http://127.0.0.1:12345/local/download",
          {file_urls: this.fileList},
          // { emulateJSON: true }
        //    { headers: {
        //   "Content-Type": "application/json",
        // },}
        )
        .then((e) => {
          console.log(e);
          var data = e.data
          if(data.result === true){
            alert('下载完成')
          }else{
            alert("下载出现异常")
          }
        }).catch((e)=>{
          alert("发起下载任务失败")
        });
    },
    startPrint(){
      var files = this.fileList

      files.forEach((element,index) => {
        console.log()
        var i = element.split("/")[2];
        var name = i.split("_")[0]+"_"+i.split("_")[1]+".pdf";
        files[index] = "PDF/"+name
      });

      console.log(files);
       this.$http
        .post(
          "http://127.0.0.1:12345/local/commit",
          {"files": [files[0],files[1]],'printer_name':this.printername,'mode':this.printmode},
        // {"printer_name" : "打印机A", "mode":"正面"/"反面"/"逆序反面","files":["D:/1.pdf","D:/2.pdf"]}
        )
        .then((e) => {
          console.log(e);
          var data = e.data 
           if (data.result === true) {
            alert("创建打印任务成功")
          } else {
            alert(data.error_message);
          }
          
        }).catch((e)=>{
          alert("发起打印任务失败")
        });
    },
    GetPrinterList() {
      this.$http.get("http://127.0.0.1:12345/local/search").then(
        (s) => {
          console.log(s);
          var data = s.data;
          if (data.result === true) {
            this.PrinterList = data.service_list;
          } else {
            alert("获取打印机列表失败,JAVA服务启动失败");
          }
        },
        (e) => {
          alert("获取打印机列表失败");
        }
      );
    },
  },
  created: function () {
    // `this` 指向 vm 实例x
    this.GetPrinterList();
  },
};
</script>

<style>
@import url("https://fonts.googleapis.com/css?family=Source+Sans+Pro");

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: "Source Sans Pro", sans-serif;
}

#wrapper {
  background: radial-gradient(
    ellipse at top left,
    rgba(255, 255, 255, 1) 40%,
    rgba(229, 229, 229, 0.9) 100%
  );
  height: 100vh;
  padding: 60px 80px;
  width: 100vw;
}

#logo {
  height: auto;
  margin-bottom: 20px;
  width: 420px;
}

main {
  display: flex;
  justify-content: space-between;
}

main > div {
  flex-basis: 50%;
}

.left-side {
  display: flex;
  flex-direction: column;
}

.welcome {
  color: #555;
  font-size: 23px;
  margin-bottom: 10px;
}

.title {
  color: #2c3e50;
  font-size: 20px;
  font-weight: bold;
  margin-bottom: 6px;
}

.title.alt {
  font-size: 18px;
  margin-bottom: 10px;
}

.doc p {
  color: black;
  margin-bottom: 10px;
}

.doc button {
  font-size: 0.8em;
  cursor: pointer;
  outline: none;
  padding: 0.75em 2em;
  border-radius: 2em;
  display: inline-block;
  color: #fff;
  background-color: #4fc08d;
  transition: all 0.15s ease;
  box-sizing: border-box;
  border: 1px solid #4fc08d;
}

.doc button.alt {
  color: #42b983;
  background-color: transparent;
}
</style>

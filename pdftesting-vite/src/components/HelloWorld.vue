<script setup>
import { ref, reactive } from 'vue'
import axios from 'axios';
import { ElNotification } from 'element-plus'
// defineProps({ })
const questionsArr = reactive([]);
const errorStatus = ref(0);
const editToggle = reactive([]);
reset();
function reset(){
  axios.get("http://localhost:1029/questions").then(res=>{
    errorStatus.value = 0;
    questionsArr.splice(0);
    questionsArr.push(...res.data)
  }).catch(()=>{
    errorStatus.value = 1;
  });

}

function forceclose(){
  alert("請稍候關閉，勿重複操作");
  axios.post("http://localhost:1029/forceclose").then(()=>{
    window.close();
  })
}

function savedone(){
  alert("傳送中，傳送完成視窗將自動關閉")
  axios.post("http://localhost:1029/done", questionsArr).then(()=>{
    window.close();
  })
}

function saveTmp(){

  ElNotification({
    title: '送出中',
    message: "請勿大量重複傳送。",
    type:"info"
  })
  axios.post("http://localhost:1029/questions", questionsArr).then(()=>{
    ElNotification({
    title: '暫存成功！',
    message: "暫存已成功，請勿關閉伺服器以免暫存資料遺失",
    type:"success"
  }) 
  })

}



</script>

<template>
<nav style="position:fixed; top:0; left:0">
  <el-button style="height:50px; width:50px" circle type="primary" @click="saveTmp()"><el-icon style="font-size: 25px;"><Promotion /></el-icon></el-button>

</nav>
  <h1>單字題目索引</h1>
  <h2 style="color:darkred" v-if="errorStatus == 1">與後端連接錯誤，請按下方「全部重來」重試</h2>
  <div v-for="(item, index) in questionsArr" :key="index" style="float:left; margin:0 auto; width: 100%">
    <el-input
      v-if="editToggle[index]==true"
      autosize
      type="textarea"
      v-model="questionsArr[index].question">
    </el-input>
    <p v-else>{{ item.question }}</p>
    <el-button style="height:30px; width:30px;" type="primary" @click="editToggle[index] = !editToggle[index]">
      <el-icon><Edit/></el-icon>
    </el-button>
    <el-button style="margin-left:1em" :class="questionsArr[index].correct == 'A'?'selected':undefined"  @click="questionsArr[index].correct ='A'">{{ questionsArr[index].answer[0] }}</el-button>
    <el-button  :class="questionsArr[index].correct == 'B'?'selected':undefined"  @click="questionsArr[index].correct ='B'">{{ questionsArr[index].answer[1] }}</el-button>
    <el-button  :class="questionsArr[index].correct == 'C'?'selected':undefined"  @click="questionsArr[index].correct ='C'">{{ questionsArr[index].answer[2] }}</el-button>
    <el-button  :class="questionsArr[index].correct == 'D'?'selected':undefined"  @click="questionsArr[index].correct ='D'">{{ questionsArr[index].answer[3] }}</el-button>
    <el-input hint="畫線字" v-model="questionsArr[index].highlighted" style="margin:20px;max-width:100px"></el-input>
    <br/>
  </div>

  <div class="card" style="margin-top:10%">
    <el-popconfirm title="所有尚未暫存內容將會遺失" @confirm="reset()">
    <template #reference>
        <el-button type="danger" class="button-padding">全部重來</el-button>
      </template>
      </el-popconfirm>
      <el-popconfirm title="所有內容將會遺失" @confirm="forceclose()">
    <template #reference>
      <el-button type="danger"  class="button-padding" >不儲存關伺服</el-button>
    </template>
    </el-popconfirm>
    <el-button type="success" class="button-padding" @click="savedone()" >完成送出</el-button>
  </div>
</template>

<style scoped>


.selected{
  background-color:cadetblue;
  color:white;
}
.el-input__wrapper{
  margin-bottom:10px
}
</style>

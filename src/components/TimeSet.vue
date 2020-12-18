<template>
  <div class="hello">
    <section class="hero is-primary">
      <div class="hero-body">
        <div class="container">
          <h1 class="title">
            お目覚め
          </h1>
          <h2 class="subtitle">
            設定時刻の確認、変更が可能です
          </h2>
        </div>
      </div>
    </section>
    <nav class="level" id="setTime">
      <div class="level-item has-text-centered">
        <div>
          <p class="heading">設定中の時刻(HH:mm:ss)</p>
          <p class="title" v-if="setTime">{{setTime}}</p>
        </div>
      </div>
      <div class="level-item has-text-centered">
        <div>
          <p class="heading">ステータス</p>
          <p class="title" v-if="setStatus">{{setStatus}}</p>
        </div>
      </div>
    </nav>
    <section class="hero">
      <div class="hero-body">
        <div class="container">
          <h1 class="title is-4">
            時刻を変更する
          </h1>
          <div class="control has-icons-left has-icons-right">
            <input class="input is-primary is-rounded is-large" v-model="timeInput" @input="timeVerify" type="text" placeholder="設定時刻を入力(例: 12:34, 11:45:14)">
            <span class="icon is-small is-left">
              <i class="fas fa-clock"></i>
            </span>
            <span v-show="checkIcon" class="icon is-small is-right" >
              <i class="fas fa-check"></i>
            </span>
          </div>
          <div class="has-text-centered" id="updateBtn">
            <button v-show="checkIcon" v-on:click="post" class="button is-primary is-rounded is-large is-fullwidth" >
              <span class="icon is-small">
                <i class="fas fa-check fa-fw"></i>
              </span>
              <span>
                変更
              </span>
            </button>
          </div>
        </div>
      </div>
    </section>

  </div>
</template>

<script>
import axios from 'axios';
const moment = require('moment');

const apiUrl = "https://kazuki-alarm-65535.herokuapp.com/alarm";
export default {
  name: 'TimeSet',
  data: function (){
    return{
      timeInput: null,
      setTime: null,
      checkIcon: false,
      setStatus: null
    }
  },
  methods:{
    timeVerify: function() {
      console.log('変更');
      if(this.timeInput !== null && this.timeInput !== ""){
        if(this.verify(this.timeInput)){
          this.checkIcon = true;
        }else{
          this.checkIcon = false;
        }
      }else{
        this.checkIcon = false;
      }
    },
    verify(text){
      if(moment(text,'HH:mm').isValid || moment(text, 'HH:mm:ss').isValid){
        return true;
      }
      return false;
    },

    post: function() {
      console.log("送信");
      const data = {"time": this.timeInput};
      this.updateTime(data);
    },

    updateTime(replaceTime){
      const then = this;
      axios.post(apiUrl+'/update', replaceTime).then(function(Response){
        console.log(Response.status);
        if(Response.data.res === "success"){
          then.updateSettingTime();
        }else{
          return false;
        }
      })
    },

    updateSettingTime(){
      // 時刻の取得を行う
      const then = this;
      axios.get(apiUrl+'/echo').then(Response => {
        console.log(Response.status);
        console.log(Response.data);
        then.setTime = Response.data.setTime;
        const that = this;
        if(Response.data.stopFlg === true){
          that.setStatus = "無効";
        }else{
          that.setStatus = "有効";
        }
      })
    }
  },
  created: function() {
    // 時刻の取得を行う
    axios.get(apiUrl+'/echo').then(Response => {
      console.log(Response.status);
      console.log(Response.data);
      console.log(Response.data.stopFlg);
      this.setTime = Response.data.setTime;
      const that = this;
      if(Response.data.stopFlg === true){
        that.setStatus = "無効";
      }else{
        that.setStatus = "有効";
      }
    })
  }

}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}

#setTime {
  margin-top: 1em;
}
#updateBtn{
  margin-top: 1em;
}
</style>

  <script lang="ts" setup>
import router from '@/router';
import { ref } from 'vue';
const baseurl = "https://b.sakurasen.cn/api/myapi/v1"
const imgsrc = ref('/imgs/m1.gif')
function sel(num: number) {
  if (num == 0) {
    imgsrc.value = ('/imgs/wx.png')
    return
  }
  if (num == 1) {
    imgsrc.value = ('/imgs/zfb.png')
    return
  }
}
const pay = ref(false)
const list = ref({
  "name": "",
  "avat": "",
  "msg": "",
  "url": "",
  "wx": true,
  "value": "",
  "email": "",
  "code": NaN
})
function payed() {
  pay.value = false
  alert("请联系QQ:2402147211")
}
function upload() {

  list.value.wx = (document.getElementsByName("paytype").item(0) as HTMLInputElement).checked
  const emailcheck = new RegExp('\\S+[@]+\\S+[.]+\\S')
  if (!emailcheck.test(list.value.email)) {
    tips.value = ""
    tips.value = "邮箱格式错误"
    return
  }
  if (typeof list.value.code !== 'number') {
    tips.value = ""
    tips.value = "验证码格式错误"
    return
  }
  fetch(baseurl + "/home/sponsor", {
    method: "POST",
    headers: {
      "Content-type": "application/json"
    },
    body: JSON.stringify(list.value)
  }).then((d) => d.json()).then((d) => {
    if (d.code !== 200) {
      tips.value = d.msg
      setTimeout(() => {
        tips.value = ""
      }, 3000);
    } else {
      tips.value = "提交成功！"
      setTimeout(() => {
        router.push('/')
      }, 3000);
      list.value = {
        "name": "",
        "avat": "",
        "msg": "",
        "url": "",
        "wx": true,
        "value": "",
        "email": "",
        "code": NaN
      }
    }
  })
}
const sended = ref(false)
const sendok = ref(false)
function SendEmail() {
  vali.value = "发送中..."
  if (sended.value) {
    return
  }
  sended.value = true
  const emailcheck = new RegExp('\\S+[@]+\\S+[.]+\\S')
  if (!emailcheck.test(list.value.email)) {
    tips.value = "邮箱格式错误"
    sended.value = false
    vali.value = "获取验证码"
    setTimeout(() => {
      tips.value = ""
    }, 3000);
    return
  }
  fetch(baseurl + "/vali/email?email=" + list.value.email)
    .then((d) => d.json()).then((d) => {
      if (d.code !== 200) {
        tips.value = d.msg
        vali.value = "错误"

        setTimeout(() => {
          sended.value = false
          vali.value = "获取验证码"
        }, 3000);
      } else {
        vali.value = d.msg

        setTimeout(() => {
          sendok.value = true
        }, 1000);
        const timer = setInterval(() => {
          if (time.value <= 0) {
            clearInterval(timer)
            sendok.value = false
            vali.value = "重新发送"
            return
          }
          time.value--
        }, 1000);
      }
    })
}
const time = ref(300)
const tips = ref('这并不是必填的，你可以无名支持')
const vali = ref('获取验证码')
</script>
<template>
  <div id="zcn">

    <div id="topay" v-if="!pay">
      <h2>你可能无法获得任何收益...</h2>
      <h1>你真的要这么做吗？</h1>
      <img :src="imgsrc" alt="">
      <div class="sel"><span class="wechat" @click="sel(0)">微信</span><span @click="router.push('/')">取消</span><span
          @click="payed">我已付款</span><span class="alipay" @click="sel(1)">支付宝</span>
      </div>

    </div>
    <div class="payed" v-if="pay">
      <h1>感谢您的支持</h1>
      <p>{{ tips }}</p>
      <span>您的昵称</span>
      <input type="text" id="Iname" v-model="list.name">
      <span>您的头像地址（可选）</span>
      <input type="text" id="Iavat" v-model="list.avat">
      <span>您想说的话（可选）</span>
      <input type="text" id="Imsg" v-model="list.msg">
      <span>您的网址（可选）</span>
      <input type="text" id="Itype" v-model="list.url">
      <span>支付方式</span>
      <div style="display: flex; align-items: center;">
        <input type="radio" checked="true" name="paytype" id="wx"><label for="wx"><span class="wechat">微信</span></label>
        <input type="radio" name="paytype" id="zfb"><label for="zfb"><span class="alipay">支付宝</span></label>
      </div>
      <span>付款金额</span>
      <input type="text" id="Ivalue" v-model="list.value">
      <span>邮箱</span>
      <input type="text" id="Iemail" v-model="list.email">
      <span>验证码</span>
      <div id="c_code"><input type="number" id="Icode" v-model="list.code"><span v-if="!sendok" style="cursor: pointer;"
          class="vali" @click="SendEmail">{{ vali
          }}</span>
        <span v-if="sendok">{{ time }}</span>
      </div>
      <div class="bt"><span @click="upload" id="cancel">完毕</span>
        <span @click="pay = false" id="over">取消</span>
      </div>
    </div>
  </div>
</template>
<style scoped>
.vali {
  position: relative;
  left: 0;
}



#c_code {
  display: flex;
  align-items: center;
  gap: 1rem;

  input::-webkit-inner-spin-button {
    appearance: none !important;
  }

  input {
    appearance: textfield !important;
  }

  input {
    width: 8rem;
  }


  position: relative;
}

* {
  --wx: #1eff00;
  --zfb: #1f41ff
}

#topay {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

#zcn {
  .payed {
    display: flex;
    flex-direction: column;

    .bt {
      margin: 2rem 0;

      width: fit-content;

      gap: 2rem;
      display: flex;

      span {
        padding: 0.1rem 0.5rem;
        font-weight: 500;
        font-size: 1.2rem;
        color: #000000;
        border-radius: 2px;
        cursor: pointer;
        box-shadow: #fff 2px 2px, #fff -2px -2px;
      }

      #over {
        background-color: #5cff46;
      }

      #cancel {
        background-color: #ff3e37;
      }
    }

    p {
      text-align: center;
      color: #ff1e1e;
    }

    span {
      color: #ffffff;
      background-color: #000000;
    }

    input {
      background-color: #000000;

      color: #ffffff;
      height: 2rem;
      padding: 1rem;
      box-sizing: border-box;
      outline: none;
      border: #fff solid 2px;
    }

    .wechat {
      color: var(--wx);
    }

    .alipay {
      color: var(--zfb);
    }
  }

  width: 100dvw;
  height: 100dvh;
  position: absolute;
  top: 0;
  left: 0;
  background-color: #000;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;

  h1 {
    color: #ffffff;

  }

  h2 {
    color: #ff1e1e;
    margin: 0;
  }

  .sel {
    display: flex;
    gap: 2rem;
    margin: 2rem;

    span {
      color: #ff0000;
      cursor: pointer;
    }

    .wechat {
      color: var(--wx);
    }

    .alipay {
      color: var(--zfb);
    }
  }

  span {
    color: #ff3c3c;
  }

  img {
    width: 256px;
  }
}

@media (height < 814px) {
  #zcn {
    justify-content: start;

    height: 110dvh !important;

    #payed {
      margin-top: 5rem;
      top: 0;
    }

    #topay {
      top: 4rem;
      position: relative;
    }
  }
}
</style>

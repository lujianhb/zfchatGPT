<script setup>
import { ref } from "vue";
import { Cloud } from "laf-client-sdk";
// 将marked 引入
import { marked } from "marked";

const cloud = new Cloud({
  baseUrl: "https://l5hrw3.laf.dev",
  getAccessToken: () => "",
  timeout: 60000,
});

const list = ref([]);
const question = ref("");
const parentMessageId = ref("");
const loading = ref(false);

async function send() {
  if (loading.value) return;

  list.value.push({
    text: question.value,
    avatar: "/avatar.png",
  });

  setScreen();

  const message = question.value;
  question.value = "";
  loading.value = true;
  let res;
  if (message == "") {
    loading.value = false;
    list.value.push({
      text: "问题不能为空！",
      avatar: "/log.png",
    });
    setScreen();
    return;
  }
  try {
    if (!parentMessageId.value) {
      res = await cloud.invoke("send", { message });
    } else {
      res = await cloud.invoke("send", {
        message,
        parentMessageId: parentMessageId.value,
      });
    }
  } catch (error) {
    console.log(error);
    loading.value = false;
    list.value.push({
      text: "出错了，请重试！",
      avatar: "/log.png",
    });
    setScreen();
    return;
  }

  // console.log(res)

  parentMessageId.value = res.id;

  // res.text = res.text.replace(/\n/g, "<br>");
  res.text = marked.parse(res.text);

  list.value.push({
    text: res.text,
    avatar: "/log.png",
  });

  loading.value = false;
  setScreen();
}

function setScreen() {
  setTimeout(() => {
    window.scrollTo(0, document.body.scrollHeight);
  }, 0);
}
</script>

<template>
  <div class="page">
    <div class="begintitle">
      <h1 v-show="!list.length" @click="send">时维的ChatGPT</h1>
    </div>

    <div id="myList">
      <div :class="item.type === 0 ? 'problemList' : 'answerList'" v-for="item in list">
        <img class="listImg" :src="item.avatar" alt="" />
        <div v-highlight v-html="item.text" class="listText"></div>
      </div>

      <div v-show="loading" class="answerList">
        <img class="listImg" src="/log.png" alt="" />
        <img class="addin" src="/loading.gif" alt="" />
      </div>
    </div>

    <div class="steppingstone"></div>

    <div class="inputbox">
      <input
        v-bind:readonly="loading"
        @keyup.enter="send"
        tabindex="0"
        data-id="root"
        rows="1"
        v-model="question"
        type="text"
        id="message"
        placeholder="输入你的指令"
        maxlength="100"
      />
      <div class="btn-send" id="submit-btn" @click="send">
        <div class="send-view" style="display: flex">
          <svg
            stroke="currentColor"
            fill="none"
            stroke-width="2"
            viewBox="0 0 24 24"
            stroke-linecap="round"
            stroke-linejoin="round"
            class="h-4 w-4 mr-1"
            height="1.5em"
            width="1.5em"
            xmlns="http://www.w3.org/2000/svg"
          >
            <line x1="22" y1="2" x2="11" y2="13"></line>
            <polygon points="22 2 15 22 11 13 2 9 22 2"></polygon>
          </svg>
        </div>
        <div class="send-loading" style="display: none">
          <div></div>
          <div></div>
          <div></div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.page {
  position: relative;
  height: 100vh;
}

.defbut {
  position: fixed;
  right: 2px;
  bottom: 152px;
}
.btn-send {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 48px;
  height: 32px;
  border-radius: 6px;
  color: rgba(0, 0, 0, 0.6);
  background: rgba(0, 0, 0, 0.1);
}
.btn-send:hover {
  cursor: pointer;
  opacity: 0.85;
}
.text {
  position: absolute;
  top: 50px;
  border: 1px solid #e5e5e5;
  height: 60px;
  padding: 10px;
  width: 90%;
}

#myList {
  max-width: 1000px;
  margin: 0 auto;
  overflow-x: hidden;
  overflow-y: auto;
}

.problemList {
  display: flex;
  padding: 0px 200px;
}

.answerList {
  position: relative;
  padding: 20px 18px;
  font-size: 15px;
  display: flex;
  overflow-x: auto;
  white-space: pre-wrap;
  border-top: 1px solid #e5e5e5;
  border-bottom: 1px solid #e5e5e5;
}

.listImg {
  margin-top: 5px;
  width: 40px;
  height: 40px;
  border-radius: 50%;
}

.listText {
  margin-left: 20px;
  padding-top: 10px;
  width: 100%;
  white-space: pre-wrap;
}

.inputbox {
  position: fixed;
  bottom: 30px;
  left: 0;
  right: 0;
  margin: auto;
  width: 90%;
  max-width: 1000px;
  margin: 0 auto;
  display: flex;
  align-items: center;
  border: 1px solid rgba(0, 0, 0, 0.3);
  padding-right: 12px;
  background: #fff;
  border-radius: 8px;
}

.inputbox input {
  flex-grow: 1;
  height: 44px;
  max-height: 100px;
  border: 0;
  outline: none;
  padding: 12px 15px;
  background: transparent;
  font-size: 16px;
  width: 100%;
  font-weight: bold;
  color: rgba(0, 0, 0, 0.7);
}

.inputbox button {
  margin-left: 15px;
  width: 56px;
  height: 82%;
  border-radius: 6px;
  border: 0;
  background: silver;
  color: #333;
  font-size: 14px;
  outline: none;
}

.inputbox button:hover {
  cursor: pointer;
  opacity: 0.8;
}

.addin {
  margin: 10px 20px;
  width: 30px;
  height: 30px;
}

.steppingstone {
  width: 100%;
  height: 160px;
}

.begintitle {
  width: 100%;
  /* padding: 50px 50px 30px 50px; */
}
.begintitle h1 {
  padding: 50px;
  font-size: 28px;
  font-weight: bold;
  text-align: center;
}

.exhibition {
  width: 80%;
  margin: auto;
  display: flex;
  justify-content: space-around;
}

.witem p {
  margin: auto;
  padding: 10px;
  margin-top: 15px;
  font-size: 16px;
  border-radius: 5px;
  text-align: center;
}

.witem h3 {
  padding: 15px;
  font-size: 20px;
  color: #606266;
  text-align: center;
}

textarea {
  border: none;
  resize: none;
  cursor: pointer;
  outline: none;
  overflow-y: hidden;
}

@media screen and (max-width: 600px) {
  .text {
    position: absolute;
    top: 30px;
    border: 1px solid #e5e5e5;
    height: 45px;
    padding: 10px;
    width: 80%;
  }
}
</style>

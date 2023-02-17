<template>
  <div class="mobile-chat-wrapper">
    <div class="chat-history">
      <div class="chat-content" ref="chatContent">
        <div class="chat-wrapper" v-for="(item, index) in chatList" :key="item.id">
          <div class="chat-friend" v-if="item.uid !== '1001'">
            <div class="info-time">
              <img :src="item.headImg" alt=""/>
              <span>{{ item.name }}</span>
              <span>{{ item.time }}</span>
            </div>
            <div class="chat-text" v-if="item.chatType == 0">
              <template v-if="isSend && index === chatList.length - 1">
                <span class="flash_cursor"></span>
              </template>
              <template v-else>
                <pre>{{ item.msg }}</pre>
              </template>
            </div>
            <div class="chat-img" v-if="item.chatType == 1">
              <img :src="item.msg" alt="表情" v-if="item.extend.imgType == 1" style="width: 100px; height: 100px"/>
              <el-image :src="item.msg" :preview-src-list="srcImgList" v-else>
              </el-image>
            </div>
            <div class="chat-img" v-if="item.chatType == 2">
              <div class="word-file">
                <FileCard :fileType="item.extend.fileType" :file="item.msg"></FileCard>
              </div>
            </div>
          </div>
          <div class="chat-me" v-else>
            <div class="info-time">
              <span>{{ item.name }}</span>
              <span>{{ item.time }}</span>
              <img :src="item.headImg" alt=""/>
            </div>
            <div class="chat-text" v-if="item.chatType == 0">
              {{ item.msg }}
            </div>
            <div class="chat-img" v-if="item.chatType == 1">
              <img :src="item.msg" alt="表情" v-if="item.extend.imgType == 1" style="width: 100px; height: 100px"/>
              <el-image style="max-width: 300px; border-radius: 10px" :src="item.msg" :preview-src-list="srcImgList"
                        v-else>
              </el-image>
            </div>
            <div class="chat-img" v-if="item.chatType == 2">
              <div class="word-file">
                <FileCard :fileType="item.extend.fileType" :file="item.msg"></FileCard>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="chat-input-wrapper">
      <el-input v-model="inputMsg" @change="sendText" autofocus/>
      <img class="send-icon" src="@/assets/img/emoji/rocket.png" alt="" @click="sendText"/>
    </div>
  </div>
</template>

<script>
import {chatgpt} from "@/api/getData";
import {animation} from "@/util/util";

export default {
  name: "MobileChat",
  data() {
    return {
      chatList: [
        {
          "headImg": "img/head_portrait1.84f92874.jpg",
          "name": "ChatGPT",
          "time": new Date().toLocaleTimeString(),
          "msg": " ChatGPT为您服务",
          "chatType": 0,
          "uid": "1002"
        }
      ],
      inputMsg: '',
      isSend: false
    }
  },
  methods: {
    sendText() {
      if (this.inputMsg) {
        let chatMsg = {
          headImg: require("@/assets/img/head_portrait.jpg"),
          name: "卧龙",
          time: new Date().toLocaleTimeString(),
          msg: this.inputMsg,
          chatType: 0, //信息类型，0文字，1图片
          uid: "1001", //uid
        };
        this.sendMsg(chatMsg);
        this.inputMsg = "";
        let data = {
          prompt: chatMsg.msg,
          temperature: 1,
          top_p: 1,
          model: 'text-davinci-003',
          max_tokens: 2048,
          frequency_penalty: 0,
          presence_penalty: 0,
          stop: ["Human:", "AI:"]
        }
        this.loading = true
        this.isSend = true;
        let chatGPT = {
          headImg: require("@/assets/img/head_portrait1.jpg"),
          name: "ChatGPT",
          time: new Date().toLocaleTimeString(),
          msg: "",
          chatType: 0, //信息类型，0文字，1图片
          uid: "1002", //uid
        };
        this.sendMsg(chatGPT);
        chatgpt(data).then((res) => {
          this.isSend = false;
          this.chatList[this.chatList.length-1].msg = res.choices[0].text;
        });
      } else {
        this.$message({
          message: "消息不能为空哦~",
          type: "warning",
        });
      }
    },
    //发送信息
    sendMsg(msgList) {
      this.chatList.push(msgList);
      this.scrollBottom();
    },
    //获取窗口高度并滚动至最底层
    scrollBottom() {
      this.$nextTick(() => {
        const scrollDom = this.$refs.chatContent;
        animation(scrollDom, scrollDom.scrollHeight - scrollDom.offsetHeight);
      });
    },
  }
}
</script>

<style lang="scss">
.mobile-chat-wrapper {
  display: flex;
  flex-direction: column;
  overflow: hidden;
  height: 100vh;
  position: relative;
  background-color: #272a37;

  .chat-history {
    flex: 1 1 0;
    overflow-y: auto;
  }

  .chat-input-wrapper {
    display: flex;
    align-items: center;
    padding: 8px 16px 8px 8px;
    position: absolute;
    left: 0;
    right: 0;
    bottom: 0;

    .send-icon {
      height: 40px;
      margin-left: 16px;
    }
  }

  .chat-content {
    width: 100%;
    height: 85%;
    overflow-y: scroll;
    padding: 20px;
    box-sizing: border-box;

    &::-webkit-scrollbar {
      width: 0;
      /* Safari,Chrome 隐藏滚动条 */
      height: 0;
      /* Safari,Chrome 隐藏滚动条 */
      display: none;
      /* 移动端、pad 上Safari，Chrome，隐藏滚动条 */
    }

    .chat-wrapper {
      position: relative;
      word-break: break-all;

      .chat-friend {
        width: 100%;
        float: left;
        margin-bottom: 20px;
        display: flex;
        flex-direction: column;
        justify-content: flex-start;
        align-items: flex-start;

        .chat-text {
          max-width: 90%;
          padding: 20px;
          border-radius: 20px 20px 20px 5px;
          background-color: rgb(56, 60, 75);
          color: #fff;

          &:hover {
            background-color: rgb(39, 42, 55);
          }

          pre {
            white-space: break-spaces;
          }
        }

        .chat-img {
          img {
            width: 100px;
            height: 100px;
          }
        }

        .info-time {
          margin: 10px 0;
          color: #fff;
          font-size: 14px;

          img {
            width: 30px;
            height: 30px;
            border-radius: 50%;
            vertical-align: middle;
            margin-right: 10px;
          }

          span:last-child {
            color: rgb(101, 104, 115);
            margin-left: 10px;
            vertical-align: middle;
          }
        }
      }

      .chat-me {
        width: 100%;
        float: right;
        margin-bottom: 20px;
        position: relative;
        display: flex;
        flex-direction: column;
        justify-content: flex-end;
        align-items: flex-end;

        .chat-text {
          float: right;
          max-width: 90%;
          padding: 20px;
          border-radius: 20px 20px 5px 20px;
          background-color: rgb(29, 144, 245);
          color: #fff;

          &:hover {
            background-color: rgb(26, 129, 219);
          }
        }

        .chat-img {
          img {
            max-width: 300px;
            max-height: 200px;
            border-radius: 10px;
          }
        }

        .info-time {
          margin: 10px 0;
          color: #fff;
          font-size: 14px;
          display: flex;
          justify-content: flex-end;

          img {
            width: 30px;
            height: 30px;
            border-radius: 50%;
            vertical-align: middle;
            margin-left: 10px;
          }

          span {
            line-height: 30px;
          }

          span:first-child {
            color: rgb(101, 104, 115);
            margin-right: 10px;
            vertical-align: middle;
          }
        }
      }
    }
  }
  .flash_cursor {
    width: 20px;
    height: 30px;
    display: inline-block;
    background: #d6e3f5;
    opacity: 1;
    animation: glow 800ms ease-out infinite alternate;
  }
  @keyframes glow {
    0% {
      opacity: 1;
    }

    25% {
      opacity: .5;
    }

    50% {
      opacity: 0;
    }

    75% {
      opacity: .5;
    }

    100% {
      opacity: 1;
    }
  }
}
</style>

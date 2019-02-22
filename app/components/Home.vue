<template>
  <Page class="page">
    <ActionBar class="action-bar">
      <Label class="action-bar-title" text="Workout rest timer"></Label>
    </ActionBar>
    <StackLayout>
      <FlexboxLayout class="center">
        <Button @tap="pickFile" text="Select audio"></Button>
        <Button @tap="count_down(3)" text="3 sec"></Button>
        <Button @tap="count_down(60)" text="1 min"></Button>
        <Button @tap="reset" text="reset"></Button>
      </FlexboxLayout>
      <Label :text="display_num" class="display"></Label>
      <Label :text="'play:'  + audioPath"></Label>
      <Button v-if="showStopbtn" @tap="clickStop" text="Stop"></Button>
    </StackLayout>
  </Page>
</template>

<script>
const { Mediafilepicker } = require("nativescript-mediafilepicker");
import { TNSPlayer } from "nativescript-audio";
export default {
  data() {
    return {
      display_num: 0,
      c_id: null,
      audioPath: null,
      player: null,
      showStopbtn: false
    };
  },
  methods: {
    pickFile() {
      let options = {
        android: {
          maxNumberFiles: 1, // Only pick 1 file a time. 只接受挑選一個檔案 
          extensions: ["mp3"] // // Only pick 1 file a time. 只接受 *.mp3
        }
      };
      let mpicker = new Mediafilepicker();
      mpicker.openFilePicker(options);
      mpicker.on("getFiles", res => {
        let results = res.object.get("results");
        // the file path is results[0].file, set it to `this.audioPath`
        // 選到的檔案路徑為 results[0].file，設定到 `this.audioPath` 裡
        this.audioPath = results[0].file;
        console.log("Set path: ", this.audioPath);
      });
      mpicker.on("error", res => {
        let msg = res.object.get("msg");
        console.log(msg);
      });
      mpicker.on("cancel", res => {
        let msg = res.object.get("msg");
        console.log(msg);
      });
    },
    count_down(num) {
      if (this.c_id) clearInterval(this.c_id);
      this.display_num = num;
      this.showStopbtn = true //When start counting, shows Stop button 顯示「Stop」按鈕
      this.c_id = setInterval(() => {
        this.display_num = this.display_num - 1;
        if (this.display_num === 0) {
          this.display_num = `Time's up`;
          this.play(); //play music  播放音樂
          clearInterval(this.c_id); //Count down finished, stop counting. 把計時器清空
        }
      }, 1000);
    },
    play() {
      if (!this.audioPath) return;
      this.player = new TNSPlayer();
      const playerOptions = {
         // Set the music file path (pickupFile from Step 4.1) to `audioFile`
        // 把 4.1 選到的 audioPath 指定給 audioFile 參數
        audioFile: this.audioPath,
        loop: false,
        completeCallback: function() {
          console.log("finished playing");
        },
        errorCallback: function(errorObject) {
          console.log(JSON.stringify(errorObject));
        },
        infoCallback: function(args) {
          console.log(JSON.stringify(args));
        }
      };
      this.player
        .playFromUrl(playerOptions)
        .then(res => {
          console.log(res);
        })
        .catch(err => {
          console.log("something went wrong...", err);
        });
    },
    clickStop() {
      if (this.c_id) clearInterval(this.c_id);
      if (this.player) this.player.pause();
      this.showStopbtn = false;
      this.display_num = 0;
    },
    reset() {
      if (this.c_id) clearInterval(this.c_id);
      this.display_num = 0;
      this.audioPath = null;
      if (this.player) this.player.pause();
      this.showStopbtn = false;
    }
  }
};
</script>
<style scoped lang="scss">
.display {
  font-size: 85;
  text-align: center;
}
.center {
  justify-content: center;
  align-items: center;
}
</style>
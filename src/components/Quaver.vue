<style scoped>
canvas {
  padding-left: 0;
  padding-right: 0;
  margin-top: 3%;
  margin-left: auto;
  margin-right: auto;
  display: block;
  /*background-color: #e6e6e6;*/
  background-color: #f3f3f3;
  border: none;
}

.bottom-menu {
  width: 800px;
  padding-left: 0;
  padding-right: 0;
  margin-top: 0;
  margin-left: auto;
  margin-right: auto;
  display: block;
  background-color: #333333;
  border: none;
  outline: 0;
}

.menu {
  padding-top: 20px;
  padding-left: 0;
  padding-right: 0;
  margin-left: auto;
  margin-right: auto;
  outline: 0;
  /*display: block;*/
  /*new */
  border: none;
  display: flex;
}

.time {
  position: absolute;
  background-color: #333333;
  color: #f3f3f3;
  width: 135px;
  margin-left: 59%;
  margin-top: 1%;
  margin-bottom: 10px;
  padding-left: 10px;
  font-size: 20px;
}

.menuButton {
  background-color: #bcbcbc;
  -moz-border-radius: 5px;
  -webkit-border-radius: 5px;
  border-radius: 5px;
  border: 0px solid #c2c2c2;
  justify-content: space-around;
  cursor: pointer;
  color: #333333;
  font-family: Trebuchet MS;
  font-size: 15px;
  font-weight: bold;
  padding: 8px 15px;
  margin: 20px;
  text-decoration: none;
}

.menuButton:hover {
  background-color: #666666;
  /* background-color:#d3d3d3; */
}

.menuButton:active {
  position: relative;
  top: 2px;
}

.menuButton:focus {
  outline: 0;
}

.sensitivity {
  margin-top: 12px;
  justify-content: space-between;
  color: #bbbbbb;
  margin-left: auto;
  margin-right: auto;
  outline: 0;
}



#sensitivity-control {
  margin-top: 12px;
  justify-content: space-around;
  outline: 0;
}

#sensitivity-control:focus {
  outline: 0;
}
</style>

<template>
<div id="quaver">
  <div class='time'>
    Time: {{this.timeSpend/1000}}
  </div>
  <canvas id="myCanvas" width="800" height="500">
        alternate content
  </canvas>
  <div class='bottom-menu'>
    <div class='menu'>
      <button class='menuButton' v-on:click="mute">Mute</button>
      <span class='sensitivity'>
        <span>Voice Sensitivity</span>
      <input id="sensitivity-control" type="range" min="1000" max="2000" value="this.sensitivity" step="10"></input>
      </span>
      <button class='menuButton' v-on:click='handleBack'>Back</button>
    </div>
  </div>
</div>
</template>
<script>
// let globalMute = false;
// if (globalMute) {
//
// } else {


// }

import * as StageData from '../utils/StageData';
import AudioAPI from '../API/audioAPI';

export default {
  name: 'quaver',
  data() {
    return {
      ctx: null,
      stage: null,
      rects: [],
      // soundMute: (globalMute ? globalMute : false),
      quaver: {
        jumpState: null,
        fallState: null,
        walkState: null
      },
      stageLevel: null,
      flag: null,
      pose: null,
      quaver_width: 58,
      quaver_height: 86,
      analyser: null,
      voiceSize: 0,
      sensitivity: 1500,
      timer: '',
      startTime: new Date(),
      currentTime: new Date(),
      timeSpend: this.currentTime - this.startTime,
      gameStop: false,
    }
  },
  computed: {
    quaver_rect: function() {
      return new createjs.Rectangle(this.quaver.x, this.quaver.y, this.quaver_width, this.quaver_height);
    },
    // voice() {
    //   return parseInt(this.voiceSize / this.sensitivity)
    // },
  },
  watch: {},
  methods: {
    mute: function() {
      createjs.Sound.muted = !createjs.Sound.muted;
      // globalMute = true;
    },
    handleBack: function() {
      window.location = '#'
    },
    tick: function(event) {

      // quaver.image = new createjs.Bitmap("../../static/img/" + Math.floor(pose + 1) + ".png").image;
      //   quaver.x = 0;
      //   // stage.fall();
      // }
      // quaver.fallState = true;

      this.game();

      // if (rect.x < -400) {
      //   //   rect.x = 500;
      // }
      // if (quaver.y = 800) {
      //   quaver.velocity = 0;
      // }
      this.stage.update();
    },

    handleKeyDown: function(e) {
      e = e || window.event;
      if (e.keyCode == '38') {
        // up arrow
        this.quaver.jumpState = true;
        // console.log("up");
      } else if (e.keyCode == '40') {
        // down arrow
        // console.log("down");
      } else if (e.keyCode == '37') {
        // left arrow
      } else if (e.keyCode == '39') {
        // right arrow

        this.quaver.walkState = true;
        // console.log("right");
      }
    },

    handleKeyUp: function(e) {
      e = e || window.event;
      // console.log(e);
      // console.log(this);

      if (e.keyCode == '38') {
        // up arrow
        this.quaver.jumpState = false;
        // console.log("up keyup");

      } else if (e.keyCode == '40') {
        // down arrow
        // this.quaver.fallState = false;
        // console.log("down keyup");
      } else if (e.keyCode == '37') {
        // left arrow
        // console.log("left keyup");
      } else if (e.keyCode == '39') {
        this.quaver.walkState = false;
        // console.log("right keyup");
      }
    },

    game: function() {
      // let rect = this.rects[0];
      if (this.gameStop) {
        this.timeSpend = 0;
        return;
      }
      this.currentTime = new Date();
      this.timeSpend = this.currentTime - this.startTime;
      let pose = this.pose;
      let quaver = this.quaver;

      if (this.analyser) {
        this.startGetVoiceSize();
      }

      if (this.isOnGround()) {
        this.quaver.fallState = false;
      } else {
        if (!this.quaver.jumpState)
          this.quaver.fallState = true;
      }

      if (quaver.fallState) {
        this.fall();
        if (quaver.outState) {
          quaver.velocity = 0;
        }
      }

      if (quaver.jumpState) {
        this.jump();
      }
      if (quaver.walkState) {
        this.walk();
      }
      if (quaver.y >= 600) {
        this.gameOver();
      }
      if (this.quaver_rect.contains(this.flag.x + 25, this.flag.y + 25)) {
        this.gameWin();
      }
    },

    fall: function() {
      let quaver = this.quaver;
      let rects = this.rects;
      quaver.velocity += 0.29 * quaver.gravity;
      for (let i = 0; i < rects.length; i++) {
        let rect = rects[i];
        let q_rect = new createjs.Rectangle(quaver.x, quaver.y + quaver.velocity, this.quaver_width, this.quaver_height);
        let rect_rect = new createjs.Rectangle(rect.rectangle.x, rect.rectangle.y, rect.rectangle.width, rect.rectangle.height);

        if (rect_rect.intersects(q_rect)) {
          quaver.y = rect.rectangle.y - 86;
          this.fallState = false;
          return;
        }
      }
      quaver.y += quaver.velocity;


    },

    walk: function() {
      let stage = this.stage;
      let quaver = this.quaver;
      let rects = this.rects;
      if (!this.quaver.jumpState && !this.quaver.fallState) {
        this.pose = (this.pose + 0.4) % 2;
        this.quaver.image = new createjs.Bitmap("static/img/" + Math.floor(this.pose + 2) + ".png").image;
      }

      let movement = 5;
      for (let i = 0; i < rects.length; i++) {
        let rect = rects[i];
        let q_rect = new createjs.Rectangle(quaver.x + movement, quaver.y, this.quaver_width, this.quaver_height);
        let rect_rect = new createjs.Rectangle(rect.rectangle.x, rect.rectangle.y, rect.rectangle.width, rect.rectangle.height);

        if (rect_rect.intersects(q_rect)) {
          if (rect.rectangle.x > quaver.x + this.quaver_width) {
            movement = 0;
            break;
          }
        }
      }

      quaver.x += movement;
      stage.x -= movement;
    },

    jump: function() {
      let quaver = this.quaver;
      quaver.fallState = false;
      quaver.velocity -= quaver.gravity;
      this.pose = (this.pose + 0.4) % 2;
      this.quaver.image = new createjs.Bitmap("static/img/" + Math.floor(this.pose + 4) + ".png").image;
      if (quaver.velocity < 0) {
        quaver.velocity = -quaver.velocity;
        quaver.jumpState = false;
        quaver.fallState = true;
        quaver.velocity = 25;
      }
      if (quaver.y < 0) {
        quaver.jumpState = false;
        quaver.fallState = true;
        quaver.velocity = 25;
      }
      quaver.y -= quaver.velocity;
    },

    resetPosition: function() {
      //looks super weird here!!!!
      this.quaver.y = 250;
    },

    isOnGround: function() {
      let quaver = this.quaver;
      let rects = this.rects;
      for (let i = 0; i < rects.length; i++) {
        let rect = rects[i];
        let q_rect = new createjs.Rectangle(quaver.x, quaver.y, this.quaver_width, this.quaver_height);
        let rect_rect = new createjs.Rectangle(rect.rectangle.x, rect.rectangle.y, rect.rectangle.width, rect.rectangle.height);
        if (rect_rect.intersects(q_rect)) {
          return true;
        }
      }
      return false;
    },
    gameOver: function() {
      this.gameStop = true;
      createjs.Sound.muted = true;

      let closing = new createjs.Shape();

      let r = new createjs.Shape();

      let x = this.quaver.x - 100

      // console.log('cnt');
      let stage = this.stage;
      let cnt = 0;
      let goInterval = setInterval(function() {
        if (!cnt) {
          // closing.graphics.beginFill('#333333').drawRect(x+800, 0, 1000, 500);
          closing.graphics.beginFill('#333333').drawRect(x + 800, 0, 1000, 500);
          stage.addChild(closing);
        }
        cnt += 1;
        // console.log(closing.graphics)
        stage.x -= 40;
        // console.log(cnt);
        if (cnt > 20) {
          clearInterval(goInterval);
          let text = new createjs.Text("GAME OVER", "40px Arial", "#f3f3f3");
          text.x = x + 1115;
          text.y = 200;
          text.textBaseline = "alphabetic";
          stage.addChild(text);
          return;
        }
      }, 50);
    },
    gameWin: function() {
      this.gameStop = true;
      createjs.Sound.muted = true;

      let closing = new createjs.Shape();

      let r = new createjs.Shape();

      let x = this.quaver.x - 100

      // console.log('cnt');
      let stage = this.stage;
      let cnt = 0;
      let goInterval = setInterval(function() {
        if (!cnt) {
          // closing.graphics.beginFill('#333333').drawRect(x+800, 0, 1000, 500);
          closing.graphics.beginFill('#333333').drawRect(x + 800, 0, 1000, 500);
          stage.addChild(closing);
        }
        cnt += 1;
        // console.log(closing.graphics)
        stage.x -= 40;
        console.log(cnt);
        if (cnt > 20) {
          console.log('if');
          clearInterval(goInterval);
          let text = new createjs.Text("YOU WIN!", "40px Arial", "#f3f3f3");
          text.x = x + 1115;
          text.y = 200;
          text.textBaseline = "alphabetic";
          stage.addChild(text);
          return;
        }
      }, 50);
    },
    startGetVoiceSize() {
      this.timer = setTimeout(() => {
        // console.log(this.analyser);
        const voiceSize = AudioAPI.getVoiceSize(this.analyser)
        this.voiceSize = voiceSize
        this.voiceChanged()
        this.startGetVoiceSize()
      }, 100)
    },
    voiceChanged() {
      // console.log(this.voice);
      let voice = parseInt(this.voiceSize / this.sensitivity)
      if (voice > 15) {
        this.quaver.walkState = true;
        if (voice > 50) {
          this.quaver.jumpState = true;
        }
      } else
      if (voice < 5) {
        this.quaver.walkState = false;
      }
    },
    adjustSensitivity(newVal) {
      this.sensitivity = newVal;
    }
  },
  mounted: function() {
    //Create a stage by getting a reference to the canvas
    this.stage = new createjs.Stage('myCanvas');
    var c = document.getElementById("myCanvas");
    this.ctx = c.getContext("2d");
    this.pose = 1;
    this.quaver = new createjs.Bitmap("static/img/" + this.pose + ".png");
    this.flag = new createjs.Bitmap("static/img/victory.png");
    //Create a Shape DisplayObject.
    this.rects = StageData.getObstacleData(this.stageLevel);

    //if it has been set it should stay at the set value????
    this.adjustSensitivity = this.adjustSensitivity.bind(this);
    let sensitivity = document.getElementById("sensitivity-control");
    let that = this;
    sensitivity.addEventListener("input", function() {
      // console.log(this.value);
      const newVal = 3000 - this.value;
      // console.log(newVal);
      that.adjustSensitivity(newVal * 1);
      // console.log(that);
    });
    // console.log(this.sensitivity);

    let flag = this.flag;
    let rects = this.rects;
    let quaver = this.quaver;

    quaver.x = 100;
    quaver.y = 164;
    quaver.gravity = 2;
    quaver.velocity = 25;
    quaver.fallState = false;
    quaver.jumpState = false;
    // quaver.jumpState = true;
    //Add Shape instance to stage display list

    let flag_pos = StageData.getFlagData(this.stageLevel)
    flag.x = flag_pos[0];
    flag.y = flag_pos[1];
    flag.scaleX = 0.2;
    flag.scaleY = 0.2;

    this.stage.addChild(quaver);
    rects.forEach((rect) => {
      this.stage.addChild(rect);
    })
    this.stage.addChild(flag);

    createjs.Ticker.addEventListener("tick", this.tick);
    createjs.Ticker.setFPS(30);
    // console.log(document);
    // let that = this;
    window.onkeydown = this.handleKeyDown.bind(this);
    window.onkeyup = this.handleKeyUp.bind(this);
    // this.startGetVoiceSize();
  },
  beforeCreate() {

  },
  created: function() {

    this.stageLevel = this.$route.query.s;
    if (AudioAPI.isSupport) {
      if (!this.analyser) {
        AudioAPI.start().then(a => {
          this.analyser = a;
        });

      }
      // console.log(this);
      // this.loadHandler = this.loadHandler.bind(this)
      createjs.Sound.on("fileload", loadHandler, this);
      createjs.Sound.registerSound("static/bgm/theme.mp3", "theme");
      createjs.Sound.muted = false;

      function loadHandler(event) {
        // console.log("here");
        // This is fired for each sound that is registered.
        var instance = createjs.Sound.play("theme"); // play using id.  Could also use full source path or event.src.
        // instance.on("complete", handleComplete, this);
        instance.volume = 0.2;
      }
    }

  }
}
</script>

<template>
  <div class="music-wrap" id="home">
    <div class="setting">
      <span class="setting-icon">
      </span>
      <span @click="minWin" class="minWin-icon">
      </span>
      <span @click="closeWin" class="closeWin-icon">
      </span>
    </div>
    <div class="music-top">
     <span @click="getSongs(singer)">
     </span>
     <input type="text" placeholder="请输入要搜索的歌手/歌曲" v-model="singer" @keydown.enter="getSongs(singer)">
     <span @click="showLogin = true">登陆</span>
    </div>
    <div class="btn-wrap">
      <ul class="btn-list">
        <li class="btn-item" v-for="(song, index) in songs" :key="song.name" @click="select(song)">
          {{index}} . {{song.name}}---{{song.fsinger}}
        </li>
      </ul>
    </div>
    <audio src="" ref="audio"></audio>
    <div class="music-control ">
      <div class="img-wrap" :class="{'add-anim':isPlay, pause: isPause}" ref='musicIcon'>
        <img :src="imgSrc" alt="">
      </div>
      <div class="title-wrap">
        {{title}}
        <span class="yin-liang">
        </span>
        <span class="add" @click="addYL()">
      </span>
        <div class="yl-wrap">
          <div class="yl-inner" ref="yljindu" style="width: 35px">

          </div>
        </div>
        <span class="sub" @click="subYL()">

         </span>

      </div>
      <div style="font-size: 12px; flex: .5; color: white;display: flex;align-items: center;justify-content: space-between;">
        <a href="" download="music" ref="aaa" class="download-btn" style="margin-right: 10px" >
          
        </a>
        {{currentTime}}/{{duration}}
      </div>
      <div class="control-btn">
        <div class="play-btn" v-show="!isShowBtn" @click="playMusic"></div>
        <div class="stop-btn" @click="stopMusic" v-show="isShowBtn" >
          <div></div>
          <div></div>
        </div>
      </div>
    </div>
    <transition name="fade">
      <div class="login-wrap" v-show="showLogin">
        <transition name="fade">
          <Login v-show="showLogin" @closeLogin='closeLogin'></Login>
        </transition>
      </div>
    </transition>
    <canvas id="canvas" class="canvas"></canvas>
  </div>
</template>

<script>
  import axios from 'axios'
  import Login from './login/login.vue'
  import {ipcRenderer} from 'electron'
  export default {
    name: 'landing-page',
    data () {
      return {
        songs: [],
        imgSrc: 'http://imgcache.qq.com/music/photo/album_300/20/300_albumpic_140820_0.jpg',
        title: 'qq音乐',
        isPlay: false,
        isPause: false,
        isShowBtn: false,
        singer: '',
        showLogin: false,
        duration: `00:00`,
        currentTime: `00:00`

      }
    },
    mounted () {
      this.getSongs('长安忆');
      var canvas = document.querySelector('canvas'),
      ctx = canvas.getContext('2d')
      canvas.width = window.innerWidth;
      canvas.height = window.innerHeight;
      ctx.lineWidth = .3;
      ctx.strokeStyle = (new Color(150)).style;

      // var mousePosition = {
      // 	x: 30 * canvas.width / 100,
      // 	y: 30 * canvas.height / 100
      // };
      var mousePosition = {
        x:  canvas.width - 100,
        y:  canvas.height - 60
      };

      var dots = {
        nb: 30,
        distance: 100,
        d_radius: 150,
        array: []
      };

      function colorValue(min) {
        return Math.floor(Math.random() * 255 + min);
      }

      function createColorStyle(r,g,b) {
        return 'rgba(' + r + ',' + g + ',' + b + ', 0.8)';
      }

      function mixComponents(comp1, weight1, comp2, weight2) {
        return (comp1 * weight1 + comp2 * weight2) / (weight1 + weight2);
      }

      function averageColorStyles(dot1, dot2) {
        var color1 = dot1.color,
          color2 = dot2.color;

        var r = mixComponents(color1.r, dot1.radius, color2.r, dot2.radius),
          g = mixComponents(color1.g, dot1.radius, color2.g, dot2.radius),
          b = mixComponents(color1.b, dot1.radius, color2.b, dot2.radius);
        return createColorStyle(Math.floor(r), Math.floor(g), Math.floor(b));
      }

      function Color(min) {
        min = min || 0;
        this.r = colorValue(min);
        this.g = colorValue(min);
        this.b = colorValue(min);
        this.style = createColorStyle(this.r, this.g, this.b);
      }

      function Dot(){
        this.x = Math.random() * canvas.width;
        this.y = Math.random() * canvas.height;

        this.vx = -.5 + Math.random();
        this.vy = -.5 + Math.random();

        this.radius = Math.random() * 2;

        this.color = new Color();
      }

      Dot.prototype = {
        draw: function(){
          ctx.beginPath();
          ctx.fillStyle = this.color.style;
          ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2, false);
          ctx.fill();
        }
      };

      function createDots(){
        for(var i = 0; i < dots.nb; i++){
          dots.array.push(new Dot());
        }
      }

      function moveDots() {
        for(var i = 0; i < dots.nb; i++){

          var dot = dots.array[i];

          if(dot.y < 0 || dot.y > canvas.height){
            dot.vx = dot.vx;
            dot.vy = - dot.vy;
          }
          else if(dot.x < 0 || dot.x > canvas.width){
            dot.vx = - dot.vx;
            dot.vy = dot.vy;
          }
          dot.x += dot.vx;
          dot.y += dot.vy;
        }
      }

      function connectDots() {
        for(var i = 0; i < dots.nb; i++){
          for(var j = 0; j < dots.nb; j++){
            var i_dot = dots.array[i];
            var j_dot = dots.array[j];

            if((i_dot.x - j_dot.x) < dots.distance && (i_dot.y - j_dot.y) < dots.distance && (i_dot.x - j_dot.x) > - dots.distance && (i_dot.y - j_dot.y) > - dots.distance){
              if((i_dot.x - mousePosition.x) < dots.d_radius && (i_dot.y - mousePosition.y) < dots.d_radius && (i_dot.x - mousePosition.x) > - dots.d_radius && (i_dot.y - mousePosition.y) > - dots.d_radius){
                ctx.beginPath();
                ctx.strokeStyle = averageColorStyles(i_dot, j_dot);
                ctx.moveTo(i_dot.x, i_dot.y);
                ctx.lineTo(j_dot.x, j_dot.y);
                ctx.stroke();
                ctx.closePath();
              }
            }
          }
        }
      }

      function drawDots() {
        for(var i = 0; i < dots.nb; i++){
          var dot = dots.array[i];
          dot.draw();
        }
      }

      function animateDots() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        moveDots();
        connectDots();
        drawDots();

        requestAnimationFrame(animateDots);
      }

      //----------------------跟着鼠标动--------------------
      document.getElementById('home').addEventListener('mousemove', function(e){
        mousePosition.x = e.pageX;
        mousePosition.y = e.pageY;
      });

      document.getElementById('home').addEventListener('mouseleave', function(e){
        mousePosition.x = canvas.width / 2;
        mousePosition.y = canvas.height / 2;
      });
      //----------------------跟着鼠标动--------------------

      createDots();
      requestAnimationFrame(animateDots);
      
    },
    computed: {

    },
    methods: {
      select (song) {
        if (time) {
          time.clearInterval()
        }
        this.$refs.audio.src = `http://ws.stream.qqmusic.qq.com/${song.playUrl}.m4a?fromtag=46`
        this.title = song.name
        this.imgSrc = song.musicIcon
        this.$refs.audio.play()
        this.isPlay = true
        this.isShowBtn = true
        this.getLyric(song.lyricId)
        this.$refs.aaa.href = `http://ws.stream.qqmusic.qq.com/${song.playUrl}.m4a?fromtag=46`
        setTimeout(() => {
          let s = parseInt(this.$refs.audio.duration%60)
          if(s < 10){
            s = `0${s}`
          }
          this.duration = `${parseInt(this.$refs.audio.duration/60)}:${s}`
        }, 500)
        let time = setInterval( () => {
          let s = parseInt(this.$refs.audio.currentTime%60)
          if(s < 10){
            s = `0${s}`
          }
          this.currentTime = `${parseInt(this.$refs.audio.currentTime/60)}:${s}`
        }, 1000)

      },
      playMusic () {
        this.$refs.audio.play()
        this.isShowBtn = true
        this.isPause = false
        
      },
      stopMusic () {
        this.$refs.audio.pause()
        this.isPause = true
        this.isShowBtn = false
      },
      getSongs (singer) {
        this.songs = []
        let url = `/api/fcgi-bin/music_search_new_platform?t=0&n=10&aggr=1&cr=1&loginUin=0&format=json&inCharset=GB2312&outCharset=utf-8&notice=0&platform=jqminiframe.json&needNewCode=0&p=1&catZhida=0&remoteplace=sizer.newclient.next_song&w=${singer}`
        axios.get(url)
        .then(res => {
          res.data['data']['song']['list'].forEach(e => {
            let name = e['f'].split('|')[1]
            name = name.replace(/\s[\x00-\xff]*/g, '')
            let img = e['f'].split('|')[22]
            let lyricId = e['f'].split('|')[0]
            let song = {
              name,
              lyricId,
              fsinger: e.fsinger,
              pubTime: e['f'].split('|')[6],
              playUrl: e['f'].split('|')[0],
              musicIcon : 'http://imgcache.qq.com/music/photo/mid_album_90/'+ img.charAt(img. length-2)+'/'+img.charAt(img.length-1)+'/'+img+'.jpg'  
            }
            this.songs.push(song)
            
          });
          let song = this.songs[0]
          this.$refs.audio.src = `http://ws.stream.qqmusic.qq.com/${song.playUrl}.m4a?fromtag=46`
          this.title = song.name
          this.imgSrc = song.musicIcon 
          this.$refs.aaa.href = `http://ws.stream.qqmusic.qq.com/${song.playUrl}.m4a?fromtag=46`
        })
      },
      closeLogin() {
        this.showLogin = false
      },
      addYL () {
        let yL = this.$refs.audio.volume
        if (yL === 1) {
          return
        }
        yL += 0.2
        this.$refs.audio.volume = yL
        this.$refs.yljindu.style.width = 35 * yL + 'px'
      },
      subYL () {
        let yL = this.$refs.audio.volume
        yL -= 0.2
        if (yL < 0) {
          return
        }
        this.$refs.yljindu.style.width = 35 * yL + 'px'
        this.$refs.audio.volume = yL
      },
      closeWin () {
        ipcRenderer.send('close-main-window')
      },
      minWin () {
        ipcRenderer.send('hide-window')
      },
      getLyric (lyricId) {
        let id = lyricId
        let url = `/sss/miniportal/static/lyric/${id%100}/${id}.xml`
        axios({
          method: 'get',
          url: url,
          responsetype: 'text',
        }).then((res) => {
          console.log(res.data)
        })

      }
    },
    components: {
      Login,
    }
  }
</script>

<style>
  * {  
    box-sizing: border-box;
    margin: 0;
    padding: 0;
    list-style: none;
  }

  #home{

  }
  .canvas {
    position: fixed;
    top: 0;
    left: 0px;
    z-index: -1;
  }
  .icon {
    width: 100%; height: 100%;
    vertical-align: -0.15em;
    fill: currentColor;
    overflow: hidden;
  }
  body { font-family: 'Source Sans Pro', sans-serif; }
  .fade-enter-active {
    animation: bounce-in .5s linear;
  }
  .fade-leave-active {
    animation: bounce-in .5s reverse linear;
  }
  @keyframes bounce-in {
    0% {
      transform: translateX(40%);
      opacity: 0;
    }
    50% {
      transform: translateX(20%);
      opacity: 0.5;
    }
    100% {
      transform: translateX(0);
      opacity: 1;
    }
  }
  .setting {
    display: flex;
    height: 20px;
    padding-top: 5px;
    justify-content: flex-end;
    align-items: center;
    background-color: dodgerblue;
  }
  .setting span {
    width: 20px;
    height: 20px;
    color: white;
    margin-right: 10px;
    cursor: pointer;
  }
  .setting-icon {
    background: url("./set.png") no-repeat center;
    background-size: 15px 15px;
  }
  .minWin-icon {
    background: url("./minus.png") no-repeat center;
    background-size: 15px 15px;
  }
  .closeWin-icon {
    background: url("./close.png") no-repeat center;
    background-size: 15px 15px;
  }
  .music-wrap {
    width: 100vw;
  }
  .music-top {
    display: flex;
    align-items: center;
    height: 50px;
    margin: 0px auto 0px auto;
    background-color: dodgerblue;

  }
  .music-top span:first-child{
    margin-left: 10px;
    display: inline-block;  
    height: 32px;
    width: 31px;
    color: #fff;
    background: url("./搜索 (1).png") no-repeat center;
    background-size: 30px 30px;
    
  }
  .music-top span:last-child {
    color: white;
    margin-left: 10px;
    cursor: pointer;
  }
  .music-top input {
    border: none;
    outline: none;
    height: 30px;
    border-radius: 5px;
    width: 250px;
    margin-left: 10px;
  }
  .music-song {
    padding-left: 10px; 
  }
  .music-song-list {
    float: left;
    margin-right: 10px;
    margin-left: 10px;
    margin-bottom: 20px;
    width: 12px;
    height: 12px;
    border-radius: 50%;
  }
  .title {
    color: #ffffff;
    text-align: right;
    padding-right: 50px;

  }
  .btn-wrap {
    padding-top: 10px
  }
  .btn-list {
    width: 100%;
    margin: 0px auto 0px auto;
    padding-left: 10px;
  }
  .btn-item {
    margin-right: 10px;
    margin-left: 10px;
    cursor: pointer;
    border-bottom: 1px solid #BDBDBD;
    height: 30px;
    line-height: 30px;
  }
  .music-control{
    position: fixed;
    bottom: 0;
    width: 100%;
    height: 50px;
    background-color: dodgerblue;
    display: flex;
  }
  .img-wrap{
    display: inline-block;
    width: 50px;
    height: 50px;
    border-radius: 50%;
    background-color: yellow;
    margin-left: 5px;
    
  }
  .img-wrap img{
    width: 50px;
    height: 50px;
    border-radius: 50%;
  }
  .title-wrap{
    vertical-align: top;
    display: inline-block;
    text-align: center;
    height: 50px;
    line-height: 50px;
    color: #fff;
    flex: 2.9;
  }
  .control-btn{
    flex: .6;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .play-btn{
    width: 20px;
    height: 20px;
    border: 15px solid #fff;
    border-color:  transparent transparent transparent #fff;
    transform: translateX(10px)
  }
  .stop-btn{
    width: 20px;
    height: 30px;
    display: flex;
    justify-content: space-between;
  }
  .stop-btn div{
    width: 6px;
    height: 30px;
    background-color: #fff;
  }
  .add-anim {
    animation: myfirst 3s infinite linear reverse;
  }
  .pause {
    animation-play-state: paused;
  }
  /* 登陆 */
  .login-wrap {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(189, 189, 189, .5);
    z-index: 999;
    
  }
  .download-btn {
    text-decoration: none;
    margin-top: 10px;
    height: 32px;
    width: 15px;
    color: #fff;
    float: right;
    cursor: pointer;
    margin-bottom: 10px;
    background: url("./download.png") no-repeat center;
    background-size: 15px 15px;
  }
  .yin-liang {
    margin-left: 0px;
    background: url("./音量.png") no-repeat center;
    background-size: 20px 20px;
  }
  .add, .sub, .yin-liang {
    display: inline-block;
    width: 20px;
    height: 20px;
    cursor: pointer;
  }
  .sub{
    background: url("./减.png") no-repeat center;
    background-size: 15px 15px;
  }
  .add{
    background: url("./add.png") no-repeat center;
    background-size: 15px 15px;
  }
  .yl-wrap {
    display: inline-block;
    width: 30px;
    height: 15px;
    vertical-align: middle;
    margin-right: 0px;
  }
  .yl-inner {
    width: 35px;
    height: 3px;
    margin-top: 3px;
    background-color: gainsboro;
  }
  @keyframes myfirst
    {
    from {
      transform: rotate(360deg)
    }
    
    to {
      transform: rotate(0deg)
    }
    }
</style>

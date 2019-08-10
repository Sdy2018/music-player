<template>
	<div id="app" class="interface">
		<el-popover
		ref="popover"
		placement="top"
		width="160"
		v-model="visible"
		>
			<p>这是一段内容这是一段内容确定删除吗？</p>
			<div style="text-align: right; margin: 0 auto ">
			  <el-button size="mini" type="text" @click="visible = false">取消</el-button>
			  <el-button type="primary" size="mini" @click="confirm">确定</el-button>
			</div>
		</el-popover>
		<el-container>
			<el-header>音乐播放器</el-header>
			<!--    <router-view></router-view> -->
			<el-main>
				<ul class="group" ref="group">
					<li class="group-item" v-for="(item,index) in musicList" :key=item.src>
						<div>
							<el-button  @click="play(item.src,index)">
								<i class="fa fa-play"></i>
							</el-button>
						</div>
						<p>{{item.songName+"/"+item.author}}</p>
						<div>
							<el-button   >
								<a :href="require('../../public/music/'+item.src)" :download="item.src">
									<i class="fa fa-download"></i>
								</a>
							</el-button>
							<el-button  @click="deleteMusic(index)">
								<i class="fa fa-trash-o"></i>
							</el-button>
						</div>
					</li>
				</ul>
				<musicLyric ref="musicLric" class="music-lyric"></musicLyric>
			</el-main>
			<el-footer >
				<div>
					<el-button  @click="backward()">
						<i class="fa fa-step-backward"></i>
					</el-button>
					<el-button  @click="mainPlay()">
						<i ref="play" class="fa fa-play"></i>
					</el-button>
					<el-button  @click="forward()">
						<i class="fa fa-step-forward"></i>
					</el-button>
				</div>
				<p class="music-info">{{songName}}{{author}}</p>
				<div class="progress-bg" ref="progressBg" @click="setMusicSchedule">
					<div ref="progressBar" :style="{'width':musicSchedule+'%'}"  class="progress-bar">
						<div class="progress-dot" id="dot" ref="progressDot"></div>
					</div>
				</div>
				<span ref="time">{{currentTime+"/"+time}}</span>
				<div style="display: flex;align-items:center">
					<i class="fa fa-volume-up" style="margin-right:5px"></i> 			
					<div class="voice-bg" ref="voiceBg" @click="setMusicVoice">
					<div class="voice-bar" ref="voiceBar" :style="{'width':voiceSchedule*100+'%'}">
						<div class="voice-dot" id="voiceDot" ref="voiceDot"></div>
					</div>
					</div>
				</div>
			</el-footer>
		</el-container>
	</div>
</template>

<script>
import $ from 'jquery'
var audio=new Audio
var lastMusic=null
var currentIndex=null
var lastLi=null
import musicLyric from "../components/music-lyric.vue"
export default{
	data(){
		return{
				//动态加载需要使用require,否则会将数据转换成字符串类型
				musicList:[],
				musciStatus:true,
				audio:audio,
				currentTime:'00:00',
				duration:null,
				time:'00:00',
				musicSchedule:0,
				progressBg:null,
				progressDot:null,
				body:null,
				songName:null,
				author:null,
				visible:false,
				confirmDelete:null,
				voiceBg:null,
				voiceBar:null,
				voiceDot:null,
				voiceSchedule:0.5
			}
		},
		methods:{
			//获取歌曲信息
			getMusicListInfo(){
				this.axios.get("http://localhost:8080/music/music.json")
				.then(res => {
					if(res.data.status==0){
						this.musicList=res.data.result
					}
				})
			},
			//获取歌词信息
			getMusicLrcInfo(index){
				this.axios.get("http://localhost:8080/music/lrc/"+this.musicList[index].lrc).then(res=>{
					this.$refs.musicLric.getLyrics(res,parseInt(audio.currentTime),this.musicList[index].img)
				})
			},
			//播放点击事件 
			play(src,index){
				var li=this.$refs.group.children[index]
				var item=this.$refs.group.children[index].getElementsByTagName("i")[0]
				// $(lastMusic).toggleClass("fa-pause fa-play");
				if(lastMusic===null){
					//获取歌词
					this.getMusicLrcInfo(index)
					this.playMusic(src,item)
					$(li).addClass("border")
				}else{
					//判断点击的是不是同一首音乐
					if(lastMusic===item){
						$(lastMusic).toggleClass("fa-pause fa-play")
						//判断当前音乐是否属于暂停状态
						audio.paused?this.getMusicLrcInfo(index):this.$refs.musicLric.clearTimer()
						audio.paused?audio.play():audio.pause()
						audio.paused?$(li).removeClass("border"):$(li).addClass("border")
					}else{
						this.getMusicLrcInfo(index)
						this.playMusic(src,item)
						//将上一首的类改变
						$(lastMusic).hasClass("fa-pause")?$(lastMusic).toggleClass("fa-pause fa-play"):null
						$(lastLi).removeClass("border")
						$(li).addClass("border")
					}
				}
				//把当前的item赋值给上一首
				lastMusic=item
				//获取当前index
				currentIndex=index
				//获取播放状态
				if(audio.paused!=this.musciStatus){
					$(this.$refs.play).toggleClass("fa-pause fa-play")
					this.musciStatus=audio.paused
				}
				//设置上一次点击的li
				lastLi=li
				//获取音乐时长
				this.audio.oncanplay = () => {
					// console.log(audio.volume=0.5)
					audio.volume=this.voiceSchedule
					this.duration=this.audio.duration
					this.time=parseInt( "0"+this.duration/60).toString().padStart(2,'0')+":"+parseInt(this.duration%60).toString().padStart(2,'0')
				}
				//获取歌手名称
				this.songName=this.musicList[index].songName+"/"
				this.author=this.musicList[index].author
			},
			//播放音乐
			playMusic(src,element){
				$(element).toggleClass("fa-pause fa-play")
				audio.src=require("../../public/music/"+src)
				//更改src后需要重新加载音乐
				audio.load()
				audio.play()
			},
			//当前音乐
			currentMusic(index){
				var src=this.musicList[index].src
				this.play(src,index)
			}
			,
			//上一首
			backward(){
				this.backwardOrForwar(-1,0,this.musicList.length-1)
			},
			//下一首
			forward(){
				this.backwardOrForwar(+1,this.musicList.length-1,0)
			},
			backwardOrForwar(num,critical,jump){
				if(currentIndex===null){
					this.currentMusic(0)
				}else if(currentIndex===critical){
					this.currentMusic(jump)
				}
				else{
					this.currentMusic(currentIndex+num)
				} 
			},
			//主播放按钮
			mainPlay(){
				if(lastMusic===null){
					this.play(this.musicList[0].src,0)
				}else{
					this.play(this.musicList[currentIndex].src,currentIndex)
				}
			},
			//监听音乐播放
			listenMusciPlay(){
				//监听音乐时间事件
				audio.ontimeupdate=(()=>{
					// console.log(audio.currentTime)
					if(this.body.onmousemove===null){
						this.musicSchedule=audio.currentTime/audio.duration*100
					}
					this.currentTime=parseInt(this.audio.currentTime/60).toString().padStart(2,'0')+":"+parseInt(this.audio.currentTime%60).toString().padStart(2,'0')
					if(this.audio.ended){
						this.forward()
					}
				})
			},
			//设置音乐进度
			setMusicSchedule(){
				var mouseX = event.clientX+document.body.scrollLeft;//鼠标x位置
				var progressStartLocation=this.progressBg.offsetLeft//控件x位置
				var left=mouseX-progressStartLocation
				this.musicSchedule=left/this.progressBg.offsetWidth*100//设置音乐进度
				audio.currentTime=audio.duration*this.musicSchedule/100
				// console.log(currentIndex+"--------")
				this.getMusicLrcInfo(currentIndex)
			},
			//滑动小圆点
			dragProgressDot(){
				//鼠标按下
				this.progressDot.onmousedown=()=>{
					//鼠标移动
					this.body.onmousemove=(event)=>{
						var mouseX = event.clientX+document.body.scrollLeft;//鼠标x位置
						var botRelativeLeft=mouseX-this.progressBg.offsetLeft//小点的相对位置
						if(botRelativeLeft<this.progressBg.offsetWidth){
							this.musicSchedule=botRelativeLeft/this.progressBg.offsetWidth*100
						}
					}
					//鼠标抬起
					this.body.onmouseup=()=>{
						this.body.onmousemove=null
						audio.currentTime=audio.duration*this.musicSchedule/100
						this.getMusicLrcInfo(currentIndex)
					}
				}


				this.voiceDot.onmousedown=()=>{
					//鼠标移动
					this.body.onmousemove=(event)=>{
						var mouseX = event.clientX+document.body.scrollLeft;//鼠标x位置
						var botRelativeLeft=mouseX-this.voiceBg.offsetLeft//小点的相对位置
						if(botRelativeLeft<this.voiceBg.offsetWidth){
							this.voiceSchedule=botRelativeLeft/this.voiceBg.offsetWidth
							if(this.voiceSchedule>=0){
								audio.volume=this.voiceSchedule
							}
						}
					}
					//鼠标抬起
					this.body.onmouseup=()=>{
						this.body.onmousemove=null
					}
				}
			},
			loadData(){
				this.progressDot=this.$refs.progressDot
				this.progressBg=this.$refs.progressBg
				this.voiceDot=this.$refs.voiceDot
				this.voiceBg=this.$refs.voiceBg
				this.body=$("body")[0]
			},
			deleteMusic(index){
				this.visible=true
				this.confirmDelete=index
			},
			confirm(){
				this.musicList.splice(this.confirmDelete,1)
				this.visible=false
			},
			setMusicVoice(){
				var mouseX = event.clientX+document.body.scrollLeft;//鼠标x位置
				var progressStartLocation=this.voiceBg.offsetLeft//控件x位置
				var left=mouseX-progressStartLocation
				this.voiceSchedule=left/this.voiceBg.offsetWidth//设置音乐进度
				this.audio.volume=this.voiceSchedule
			}
		},
		created(){
			this.getMusicListInfo()
		},
		mounted(){
			this.loadData()
			this.listenMusciPlay()
			this.dragProgressDot()
		},
		components:{
			musicLyric
		}
	}
	</script>

	<style lang="scss" >
	html,body{
		margin: 0px;
		padding: 0px;
		height: 100%;
		background-color: rgb(169, 113, 235);
	}
	.interface,.el-container{
		height: 100%;
	}
	.border{
		border:1px solid #f40
	}
	element.style{
		height: 40px;
	}
	.el-popover{
		// left:50% !important;
		// margin-left: -80px;
		// display: block;
		left:50% !important;
		margin-left:-93px !important;
		top:25% !important;
	}
	.el-container{
		padding: 0px;
		margin: 0px 10%;
		background-color: #fff;
		.el-main {
			background-color: #E9EEF3;
			color: #333;
			text-align: center;
			line-height: 20px;
			display: flex;
			margin: 0px;
			padding: 0px;
			.group{
				flex:5;
				padding: 0px;
				width: 100%;
				list-style: none;
				padding-inline-start: 0px;
				.group-item{
					display: flex;
					justify-content:space-between;
					align-items:center;
					width: 100%;
					height: 60px;
					background-color: rgba(154, 131, 249,0.5);
					box-sizing:border-box;
					.el-button{
						i{
							color: #30BFCD;
							font-size: 15px;
						}
					}
				}
			}
			.music-lyric{
				flex:2;
			}
		}
		.el-header{
			height: 5% !important;
			border:1px solid rgb(238, 68, 13);
			text-align: center;
		}
		.el-footer {
			height: 90px !important;
			background-color: #B3C0D1;
			color: #333;
			text-align: center;
			line-height: 60px;
			display: flex;
			justify-content:space-between;
			align-items:center;
			.music-info{
				width: 100px;
				line-height: 20px;
			}
			.progress-bg{
				margin: 0px;
				height: 6px;
				width: 50%;
				background-color: #fff;
				border-radius: 5px;
				.progress-bar{
					height: 100%;
					width: 0%;
					background-color: #f40;
					border-radius: 5px;
					position: relative;
					.progress-dot{
						position: absolute;
						height: 12px;
						width: 12px;
						border-radius: 50%;
						background-color: #666;
						z-index: 998;
						top: -2px;
						left:100%;
					}
				}
			}
			.voice-bg{
				margin: 0px;
				height: 6px;
				width:120px;
				border-radius: 5px;
				background-color: #fff;
				.voice-bar{
					height: 100%;
					width: 50%;
					background-color: #f40;
					border-radius: 5px;
					position: relative;
					.voice-dot{
						position: absolute;
						height: 12px;
						width: 12px;
						border-radius: 50%;
						background-color: #666;
						z-index: 998;
						top: -2px;
						left: 100%;
					}
				}
			}
		}
	}
	</style>

<template>
	<div class="music-lyric-container">
		<img ref="img"  alt="">
		<!-- <img src="../../public/music/pic/回忆的沙漏.jpg" alt=""> -->
		<ul  class="group">
			<li   class="group-item" v-for=" (item,index) in currentLrc" :key="index">
				<p :style="index===currentRow&&currentRow!==null?{color:'red'}:{color:'green'}">{{item}}</p>
			</li>
		</ul>
	</div>  
</template>

<script >
	export default{
		data(){
			return {
				lrc:"",
				lrcTime:[],
				lrcContent:[],
				currentTime:0,
				currentIndex:0,
				index:0,
				currentLrc:[],
				currentRow:null,
				timer:null,
				imgSrc:null
			}
		},
		methods:{
			getLyrics(res,time,img){
				this.lrcTime=[]
				this.lrcContent=[]
				this.currentLrc=[]
				// this.imgSrc=img
				this.$refs.img.src=require("../../public/music/pic/"+img)  
				window.clearInterval(this.timer)
				this.currentRow=null
				this.currentTime=time
				this.currentIndex=0
				this.lrc=res.data
				//把歌词分割成数组,以换行分割
				var lrcList = this.lrc.split('\r\n');
				// console.log(lrcList)
				// console.log(lrcList.length)
				lrcList.forEach((item,index)=>{
					var item =item.split("]")
					if(item[1]){
						var timeList=item[0].replace("[","").split(":")
						//获取分钟
						var minute=parseInt(timeList[0])
						//获取秒钟
						var second=parseInt(timeList[1])
						//歌词事件数组
						this.lrcTime.push(minute*60+second)
						//歌词内容数组
						this.lrcContent.push(item[1])
						// console.log(item[1])

						// this.currentLrc=this.lrcContent.slice(0,10)
					}
				})
				// console.log(this.lrcTime)
				// console.log(this.lrcContent)
				this.setTimeout()
			},
			setTimeout(){
				if(this.currentTime>this.lrcTime[this.lrcTime.length-1]){
						this.currentRow=9
						this.currentLrc=this.lrcContent.slice(this.lrcContent.length-10,this.lrcContent.length)
						return
				}
				//获取当前时间对应的index
				this.lrcTime.some( (element,index)=> {
					// statements
					if(element>this.currentTime){
						this.currentLrc=this.lrcContent.slice(this.currentIndex,this.currentIndex+10)
						return true;
					}
					if(element<=this.currentTime&&this.lrcTime[index+1]>this.currentTime){
						this.currentIndex=index
						if(this.currentIndex<=this.lrcContent.length-10){
							this.currentRow=0
							this.currentLrc=this.lrcContent.slice(this.currentIndex,this.currentIndex+10)
						}else{
							this.currentLrc=this.lrcContent.slice(this.lrcContent.length-10,this.lrcContent.length)
							this.currentRow=10-(this.lrcContent.length-this.currentIndex)
						}
						return true;
					}
				})
				// 设置定时器
				this.timer=setInterval(()=>{
					// console.log(this.currentTime)
					// console.log(this.lrcTime[this.currentIndex])

					// console.log(this.currentIndex)
					// console.log(this.lrcContent.length-1)
					//手动设置的时间
					if(this.currentTime>this.lrcTime[this.currentIndex]){
						this.currentIndex++
					}
					if(this.currentTime==this.lrcTime[this.currentIndex]){
						if(this.currentIndex===0){
							this.currentRow=0
						}
						if(this.currentIndex>0&&this.currentIndex<=this.lrcContent.length-10){
							this.currentRow=0
							this.currentLrc.shift() 
							this.currentLrc.push(this.lrcContent[this.currentIndex+9])
						}
						if(this.currentIndex<this.lrcContent.length&&this.currentIndex>this.lrcContent.length-10){
							this.currentRow=10-(this.lrcContent.length-this.currentIndex)
						}
						this.currentIndex++
					}
					this.currentTime++
				},1000)
			},
			clearTimer(){
				window.clearInterval(this.timer)
			}
		},
		created(){
			// this.getLyrics()
		},
		mounted(){
			// this.getLyrics()
		}
	}
</script>

<style lang="scss">
.music-lyric-container{
	width: 100%;
	height: 100%;
	box-sizing:border-box;
	border:1px solid black;
	img{
		width: 160px;
		height: 160px;
		margin-top: 20px;
	}
	.group{
		list-style: none;
		position: relative;
		width: 100%;
		.group-item{
			font-size: 14px;
			height: 50px !important;
			display: flex;
			justify-content:center;
			border-bottom: 1px solid #f40;
			p{
				margin: 0 auto;
			}
		}
	}
}

</style>
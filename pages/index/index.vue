<template>
	<view class="uni-margin-wrap">
		<swiper class="swiper" circular :indicator-dots="true" :autoplay="true" :interval="4000" :duration="800">
			<swiper-item class="bginfo" v-for="item in background" >
				<!-- <image :src="item" mode="aspectFit" ></image> -->
				<uni-card :cover="item" border="true"></uni-card>
			</swiper-item>
		</swiper>
	</view>
	<Weather/>
    <view class="uni-padding-wrap">
		<uni-list  v-for="(item,index) in postList" :key="index">
			<uni-card :title="item.title" :sub-title="`${item.year}-${item.month}-${item.day}`" :extra="item.category" thumbnail="/static/love.svg" @click="onClick(item.title,item.text)">
				<text class="mainFont">{{getPostText(item.text)}}</text>
			</uni-card>
		</uni-list>
    </view>
</template>
<script>
	import uniList from "@/components/uni-list/uni-list.vue"
	import uniListItem from "@/components/uni-list-item/uni-list-item.vue"
	import Weather from "@/components/weather/weather.vue"
	const innerAudioContext = uni.createInnerAudioContext();
	const mainApi = "";//这里填你自己的typecho服务器
    export default {
        components: {uniList,uniListItem,Weather},
        data() {
            return {
                postList: [],
				lastRecord: '',
				background: [],
				postText: '',
            };
        },
        onLoad() {
            this.getList();
			this.getMain();
        },
		onPullDownRefresh() {
			console.log("触发了下拉刷新"),
			setTimeout(()=>{
			this.getList();
			uni.stopPullDownRefresh();
			},2000)
		},
        methods: {
            async getList() {
				let data = {
					page: 1,
					pageSize: 100,
				}
                uni.request({
                    url: mainApi+"posts", 
                    method: 'get',
                    dataType: 'json',
					data: data,
                    success: (res) => {
                        console.log(res.data.data);
                        this.postList = res.data.data;
                    },
                });
				uni.showLoading({
					title: '加载中'
				});
				
				setTimeout(function () {
					uni.hideLoading();
				}, 2000);
            },
			onClick(title,text){
				uni.navigateTo({
				    url: '/pages/post/post?title='+title+'&text='+text
				});
			},
			async getMain() {
				uni.request({
					url: mainApi+"post?cid=",//这里的cid要填你创建的独立页面的cid
					method: 'get',
					dataType: 'json',
					success: (res) => {
						var main = res.data.data.text;
						var mainList = main.split(',');
						var flag = parseInt(mainList[0]);
						var musicUrl = mainList[1];
						this.background[0]=mainList[2];
						this.background[1]=mainList[3];
						this.background[2]=mainList[4];
						var tips = mainList[5];
						this.putInfo(flag,tips);
						this.playVoice(musicUrl);
					},
				})
			},
			getPostText(text){
				let main = text.split(';');
				let postText = "";
				if(main[1]==null){
					if(!main[0]){
						postText = "暂时没有简介了啦(●'◡'●)";
					}else{
						postText = main[0];
					}
				}else{
					postText = main[1];
				}
				return postText;
			},
			putInfo(flag,tips){
				switch(flag){
					case 1:
						uni.showModal({
							title: 'Tips',
							content: tips,
							showCancel: false,
							confirmText: 'OK！'
						})
						break;
					case 2:
						uni.showModal({
							title: '有更新啦！',
							content: tips,
							showCancel: '取消',
							confirmText: '前往更新',
							success: function(res){
								if (res.confirm) {
									plus.runtime.openURL('');//这里填你的更新链接
								} else if (res.cancel) {
									console.log('用户点击取消');
								}
							}
						})
				}
			},
			playVoice(url) { // url即为音频路径
				if (url) {
					// 判断路径是否已经被赋值，以及音频播放状态 !innerAudioContext.paused为正在播放
					if (this.lastRecord == url && !innerAudioContext.paused) {
						innerAudioContext.stop();  // 停止
						return;
					}
					this.lastRecord = url; 			// 将路径赋值给定义的变量好做判断
					innerAudioContext.src = url;  	// 配置音频播放路径
					innerAudioContext.play();		// 播放
					innerAudioContext.loop = true	// 是否循环播放
				}
			}
		}
	}
</script>
<style>
	.uni-margin-wrap {
		width:690rpx;
		width: 100%;
	}
	.swiper {
		height: 550rpx;
	}
	.swiper-item {
		display: block;
		height: 300rpx;
		line-height: 300rpx;
		text-align: center;
	}
	
	.swiper-list {
		margin-top: 40rpx;
		margin-bottom: 0;
	}
	
	.uni-common-mt{
		margin-top:60rpx;
		position:relative;
	}
	
	.info {
		position: absolute;
		right:20rpx;
	}
	
	.uni-padding-wrap {
	    width:100%;
	}
</style>
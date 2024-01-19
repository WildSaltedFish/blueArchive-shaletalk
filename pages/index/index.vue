<template>
	<view>
		<cu-custom bgColor="bg-orange">
			<block slot="content">{{studentName}}</block>
		</cu-custom>
		<view class="cu-chat">
			<view v-for="(item,index) in textInfo">
				<!-- 老师 -->
				<view class="cu-item self" v-if="item.side === 'teacher' && item.info !== '不要显示'"
					@click="modify(index)">
					<view class="main" style="max-width: calc(100% - 175upx);">
						<view class="action" v-if="onChangeNum === index">
							<text class="cuIcon-locationfill" style="font-size: 40rpx;"></text>
						</view>
						<view class="content shadow" v-if="item.type ==='text'">
							<text class='teacherInfoBox'>{{item.info}}</text>
						</view>
						<view class="content shadow" v-if="item.type ==='pic'"
							style="background-color:  #FFFFFF;border-radius: 5px;max-width: 100%;">
							<image :src='item.url' mode="aspectFit" style="max-width: 400rpx;"></image>
						</view>
					</view>
				</view>
				<!-- 旁白 -->
				<view class='backText' v-if="item.side === 'narration'" @click="modify(index)">
					<view class="action" v-if="onChangeNum === index">
						<text class="cuIcon-locationfill" style="font-size: 40rpx;"></text>
					</view>
					<view v-if="item.type ==='text'">
						{{item.info}}
					</view>
					<image :src='item.url' mode="aspectFit" v-if="item.type ==='pic'" style="max-width: 400rpx;"></image>
					<image :src='item.url' mode="aspectFit" v-if="item.type ==='picLove'" style="min-width: 400rpx;"></image>			
				</view>
				<!-- 学生 -->
				<view class="cu-item" v-if="item.side === 'student'" @click="modify(index)">
					<image class="cu-avatar radius" :src="item.pic"></image>
					<view class="main" style="display: grid;">
						<view style="color: #3B424B;font-size: 15px;font-weight: 400;"
							v-if="textInfo[index - 1].side || textInfo[index - 1].side !=='student'">
							<text>{{item.name}}</text>
						</view>
						<view class="content shadow"
							style="background-color: #3B424B;color: white;font-size: 17px;font-weight: 500;"
							v-if="item.type ==='text'">
							<text
								style="max-width: 500rpx;word-break: break-all;white-space: pre-line;">{{item.info}}</text>
						</view>
						<view class="content shadow"
							style="background-color: #FFFFFF;color: white;font-size: 17px;font-weight: 500;"
							v-if="item.type === 'pic'">
							<image :src='item.url' mode="aspectFit" style="max-width: 400rpx;"></image>
						</view>
					</view>
					<view class="action" v-if="onChangeNum === index">
						<text class="cuIcon-locationfill"
							style="font-size: 40rpx;margin-left: -50rpx;margin-top: 35rpx;"
							v-if="textInfo[index - 1].side ||textInfo[index - 1].side !=='student'"></text>
						<text class="cuIcon-locationfill" style="font-size: 40rpx;margin-left: -50rpx;margin-top: 5rpx;"
							v-else></text>
					</view>
				</view>
			</view>
		</view>
		<view class="cu-bar foot input" :style="[{bottom:InputBottom+'px'}]">
			<input v-if="textInfo[this.onChangeNum].type === 'pic'" class="solid-bottom" :adjust-position="false" :focus="false" maxlength="300" cursor-spacing="10"
				@focus="InputFocus" @blur="InputBlur" @input="changeWorld()" disabled></input>
			<input v-else class="solid-bottom" :adjust-position="false" :focus="false" maxlength="300" cursor-spacing="10"
				@focus="InputFocus" @blur="InputBlur" v-model="modifyInput" @input="changeWorld()" :disabled="textInfo[this.onChangeNum].type === 'pic'"></input>
			<view class="action">
				<button class="cu-btn round lines-blue shadow" @tap="modalName = 'bottomModal'"
					data-target="RadioModal">编辑</button>
			</view>
			<button class="cu-btn round lines-blue shadow" @tap="modalName = 'RadioModal'"
				data-target="RadioModal">新增</button>
			<!-- 弹窗部分 -->
			<!-- 新增弹窗 -->
			<view class="cu-modal" :class="modalName=='RadioModal'?'show':''" @tap="hideModal">
				<view class="cu-dialog" @tap.stop="">
					<radio-group class="block">
						<view class="cu-list menu text-left">
							<!-- 老师 -->
							<view class="cu-item">
								<label class="flex justify-between align-center flex-sub">
									<view class="flex-sub">老师</view>
								</label>
								<view class="cu-form-group">
									<button class="cu-btn round Editbutton" @tap="createText(1)">文字</button>
									<button class="cu-btn round Editbutton" @tap="createPic(1)">图片</button>
								</view>
							</view>
							<!-- 学生 -->
							<view class="cu-item">
								<label class="flex justify-between align-center flex-sub">
									<view class="flex-sub">学生</view>
								</label>
								<view class="cu-form-group">
									<button class="cu-btn round Editbutton" @tap="createText(2)">文字</button>
									<button class="cu-btn round Editbutton" @tap="createPic(2)">图片</button>
								</view>
							</view>
							<view class="cu-item">
								<label class="flex justify-between align-center flex-sub">
									<view class="flex-sub">旁白</view>
								</label>
								<view class="cu-form-group">
									<button class="cu-btn round Editbutton" @tap="createText(3)">文字</button>
									<button class="cu-btn round Editbutton" @tap="createPic(3)">图片</button>
									<button class="cu-btn round Editbutton" @tap="createPic(4)">羁绊</button>
								</view>
							</view>
						</view>
					</radio-group>
				</view>
			</view>
			<!-- 编辑弹窗 -->
			<view class="cu-modal bottom-modal" :class="modalName=='bottomModal'?'show':''" style="margin-top: 150rpx;">
				<view class="cu-dialog">
					<view class="cu-bar bg-white">
						<view class="action" @tap="showFastListMe">快捷选择:on</view>
						<view class="action" @tap="delText">删除</view>
						<view class="action" @tap="hideModal">关闭</view>
					</view>
					<view class="padding-xl">
						<view style="margin-bottom: 50rpx;">
							角色：
							<button class="cu-btn round Editbutton"
								:class="textInfo[this.onChangeNum].side === 'teacher' ? 'bg-orange': ''"
								@tap="changeRole(1)">老师</button>
							<button class="cu-btn round Editbutton"
								:class="textInfo[this.onChangeNum].side === 'student' ? 'bg-orange': ''"
								@tap="changeRole(2)">学生</button>
							<button class="cu-btn round Editbutton"
								:class="textInfo[this.onChangeNum].side === 'narration' ? 'bg-orange': ''"
								@tap="changeRole(3)">旁白</button>
						</view>
						<view v-if="textInfo[this.onChangeNum].side === 'student'" style="margin-top: 10rpx;">
							<!-- //自定义模式 -->
							<view v-if="showFastList === false">
										<view class="cu-form-group" style="width: 100%;">
								<view class="title">名字</view>
								<input name="input" v-model="textInfo[onChangeNum].name"></input>
							</view>
							<view class="cu-bar bg-white margin-top">
								<view class="action">
									头像
								</view>
							</view>
							<view class="cu-form-group" v-if="textInfo[onChangeNum].type === 'pic'">
								<view class="grid col-4 grid-square flex-sub">
									<image style="width: 280rpx;height: 280rpx" :src="textInfo[onChangeNum].pic"
										mode="aspectFill" @tap="ChooseImage(1)"></image>
								</view>
							</view>
							</view>
							<!-- 极速模式 -->
							<view v-else style="width: 100%;height:190px;background-color: #FFFFFF;padding: 10px;display: flex;flex-wrap: wrap;">
								<view v-for="item in fastStudent" style="margin-left: 10px;">
								<image :src="item.pic" style="border-radius: 200px;width: 100rpx;height: 100rpx"></image>
								<br>	<span>{{item.name}}</span>
								</view>
							</view>
						</view>
						<view class="cu-bar bg-white margin-top" v-if="textInfo[onChangeNum].type === 'pic'">
							<view class="action">
								更换图片
							</view>

						</view>
						<view class="cu-form-group" v-if="textInfo[this.onChangeNum].side === 'student'">
							<view class="grid col-4 grid-square flex-sub">
								<image style="max-width: 300px;" :src="textInfo[onChangeNum].url" mode="aspectFill"
									@tap="ChooseImage(2)"></image>
							</view>
							<button class="cu-btn round Editbutton"
								:class="textInfo[this.onChangeNum].side === 'teacher' ? 'bg-orange': ''"
								@tap="ChooseImage(2)" v-if="textInfo[onChangeNum].url === ''" >
								添加</button>
						</view>


					</view>
				</view>
			</view>
		</view>

	</view>
</template>

<script>
	export default {
		data() {
			return {
				InputBottom: 0,
				studentName: '沙勒特供Wechat',
				textInfo: [{
						side: 'teacher',
						info: '不要显示',
						type: 'text'
					},
					{
						side: 'teacher',
						info: '这是一段老师文字',
						type: 'text',
					},
					{
						side: 'student',
						name: '响',
						pic: '/static/YOU.png',
						type: 'pic',
						url: '/static/student.jpg'
					},
					{
						side: 'narration',
						info: '这是一段旁白文字',
						type: 'text',
					},
				],
				modifyInput: '这是一段老师文字',
				onChangeNum: 1,
				modalName: '',
				createSide: 0,
				editRow: {},
				imgList: [],
				showFastList: true,
				fastStudent: [{
					side: 'student',
					name: '响',
					pic: '/static/fastImg/YOU.png',
					type: 'pic',
					url: '/static/student.jpg'
				},{
					side: 'student',
					name: '老八',
					pic: '/static/fastImg/laoba.jpg',
					type: 'pic',
					url: '/static/student.jpg'
				},{
					side: 'student',
					name: '真白',
					pic: '/static/fastImg/zhenbai.jpg',
					type: 'pic',
					url: '/static/student.jpg'
				},{
					side: 'student',
					name: '千夏',
					pic: '/static/fastImg/qianxia.jpg',
					type: 'pic',
					url: '/static/student.jpg'
				},{
					side: 'student',
					name: '优香',
					pic: '/static/fastImg/youxiang.png',
					type: 'pic',
					url: '/static/student.jpg'
				},{
					side: 'student',
					name: '梓',
					pic: '/static/fastImg/zi.png',
					type: 'pic',
					url: '/static/student.jpg'
				},{
					side: 'student',
					name: '未花',
					pic: '/static/fastImg/mika.png',
					type: 'pic',
					url: '/static/student.jpg'
				},{
					side: 'student',
					name: '星野',
					pic: '/static/fastImg/xingye.png',
					type: 'pic',
					url: '/static/student.jpg'
				},{
					side: 'student',
					name: '兄弟',
					pic: '/static/fastImg/xiongdi.png',
					type: 'pic',
					url: '/static/student.jpg'
				},{
					side: 'student',
					name: '小春',
					pic: '/static/fastImg/xiaochun.png',
					type: 'pic',
					url: '/static/student.jpg'
				},]
			};
		},
		methods: {
			InputFocus(e) {
				this.InputBottom = e.detail.height
			},
			InputBlur(e) {
				this.InputBottom = 0
			},
			modify(num) {
				console.log(num)
				this.modifyInput = this.textInfo[num].info
				console.log(this.textInfo[num].type)
				this.editRow = this.textInfo[num]
				this.onChangeNum = num
			},
			changeWorld() {
				this.textInfo[this.onChangeNum].info = this.modifyInput
			},
			// 创建新的文本条
			createText(num) {
				console.log(num)
				this.createSide = num
				if (num === 1) {
					this.textInfo.push({
						side: 'teacher',
						info: '这是一段老师文字',
						type: 'text'
					})
				} else if (num === 2) {
					this.textInfo.push({
						side: 'student',
						info: '这是一段学生文字',
						name: '响',
						pic: '/static/fastImg/YOU.png',
						type: 'text'
					})
				} else {
					this.textInfo.push({
						side: 'narration',
						info: '这是一段旁白文字',
						type: 'text'
					}, )
				}
				this.modalName = ''
			},
			// 创建新的图片条
			createPic(num) {
				console.log(num)
				this.createSide = num
				if (num === 1) {
					this.textInfo.push({
						side: 'teacher',
						type: 'pic',
						url: '/static/teacher.jpg'
					})
				} else if (num === 2) {
					this.textInfo.push({
						side: 'student',
						name: '响',
						pic: '/static/fastImg/YOU.png',
						type: 'pic',
						url: '/static/student.jpg'
					})
				} else if(num === 3) {
					this.textInfo.push({
						side: 'narration',
						type: 'pic',
						url: '/static/normal.jpeg'
					})
				}else{
					this.textInfo.push({
						side: 'narration',
						type: 'picLove',
						url: '/static/love.png'
					})
				}
				this.modalName = ''
			},
			hideModal(e) {
				this.modalName = null
			},
			delText() {
				console.log(this.onChangeNum)
				console.log(this.textInfo.length)
				console.log(this.onChangeNum === this.textInfo.length - 1)
				this.textInfo.splice(this.onChangeNum, 1)	
				this.onChangeNum = this.onChangeNum - 1
				
				this.hideModal()
			},
			changeRole(num) {
				this.$nextTick(() => {
					if (num === 1) {
						this.textInfo[this.onChangeNum].side = 'teacher'
					} else if (num === 2) {
						this.textInfo[this.onChangeNum].side = 'student'
						this.textInfo[this.onChangeNum].name = '响'
						this.textInfo[this.onChangeNum].pic = '/static/fastImg/YOU.png'
					} else {
						this.textInfo[this.onChangeNum].side = 'narration'
					}
				})
			},
			ChooseImage(e) {
				uni.chooseImage({
					count: 1, //默认9
					sizeType: ['original', 'compressed'], //可以指定是原图还是压缩图，默认二者都有
					sourceType: ['album'], //从相册选择
					success: (res) => {
						this.$nextTick(() => {
							if (e === 1) {
								this.textInfo[this.onChangeNum].pic = res.tempFilePaths[0]
								this.$forceUpdate();
							} else {
								this.textInfo[this.onChangeNum].url = res.tempFilePaths[0]
								this.$forceUpdate();
							}
						})
					}
				})
			},
			DelImg(e) {
				this.$nextTick(() => {
					this.textInfo[this.onChangeNum].pic = ''
				})
			},	
			//开启快捷选人（学生）
			showFastListMe(e) {
				this.$nextTick(() => {
					this.showFastList = true
				})
			},
		}
	}
</script>

<style>
	page {
		padding-bottom: 100upx;
		background-color: #fcf6e6;
	}

	.Editbutton {
		margin-left: 20rpx;
	}

	.teacherInfoBox {
		max-width: 900rpx;
		word-break: break-all;
		white-space: pre-line;
	}

	.backText {
		width: 90%;
		text-align: center;
		font-size: 13px;
		font-weight: 400;
		word-break: break-all;
		white-space: pre-line;
		margin-left: 5%;
		margin-top: 20rpx;
	}
</style>
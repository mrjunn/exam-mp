<template>
	<view class="warp" style="height: 100%;">
		<!-- #ifdef APP-PLUS -->
		<status-bar />
		<!-- #endif -->
		
		<uni-section title="答题中" style="margin: 0;" type="line"></uni-section>
		
		<div style="height:calc(100% - 99px);overflow-y: auto;">
			<div v-for="(item, index) in questions" :key="index" class="ques-item">
				<!-- 题目 -->
				<div class="subject">
					<uni-icons v-if="status !== 'answering'" :type="answers[index].value === item.answer?'checkmarkempty':'closeempty'" size="24"></uni-icons>
					{{index+1}}. {{item.subject}}
				</div>
				<!-- 选项 -->
				<uni-data-checkbox class="option" v-model="answers[index].value" :localdata="item.options2"></uni-data-checkbox>
				
				<div v-if="status !== 'answering' && answers[index].value !== item.answer" class="right-answer">
					正确答案：<strong>{{tag[item.answer]}}</strong>
				</div>
			</div>
		</div>
		
		<div style="position: fixed;bottom: 0;width: 100%;background-color: #fff;">
			<button type="primary" style="margin: 5px 15px;" @click="status = 'answered'">提交试卷</button>
		</div>
		
	</view>
</template>

<script>
	import statusBar from "@/uni_modules/uni-nav-bar/components/uni-nav-bar/uni-status-bar";
	export default {
		components: {
			statusBar
		},
		data() {
			return {
				questions: [],
				answers: [],
				tag: ['A', 'B', 'C', 'D', 'E'],
				status: 'answering',
				hasLogin: false
			}
		},
		onShow() {
			this.hasLogin = uniCloud.getCurrentUserInfo().tokenExpired > Date.now()
		},
		onLoad(opt) {
			let d = JSON.parse(opt.d)
			uni.setNavigationBarTitle({
				title: d.title
			});
			this.getQuestions(d.id)
		},
		methods: {
			getQuestions(id){
				console.log('id', id);
				uni.showLoading({
					title: '',
					mask:true
				});
				// 保存到云数据库
				const db = uniCloud.database() //代码块为cdb
				db.collection('exam-questions')
				  .where(`exam_id=="${id}"`)
					.get()
				  .then((res)=>{
				    // res 为数据库查询结果
					console.log('res', res)
					if(res.result.errCode===0 && res.result.data.length) {
						let questions = res.result.data;
						questions.forEach((item, index)=>{
							let options2 = []
							item.options.forEach((optItem, optIndex)=>{
								options2.push({
									value:optIndex,
									text: `${this.tag[optIndex]}. ${optItem}`
								})
							})
							questions[index]['options2'] = options2
							this.answers.push({value:-1})
						})
						this.questions = questions
					}
					uni.hideLoading()
				  }).catch((err)=>{
				    console.log(err.code); // 打印错误码
					console.log(err.message); // 打印错误内容
					uni.showToast({
						title: "考题获取失败",
						icon: "error",
						duration: 2000
					});
				  })
			},
			goToExam(examId){
				uni.navigateTo({
					url: 'exam/paper?id='+examId
				});
			}
		}
	}
</script>

<style>
	page {
		display: flex;
		flex-direction: column;
		box-sizing: border-box;
		background-color: #f3f3f3;
		height: 100%;
	}
	view {
		font-size: 14px;
		line-height: inherit;
	}
	
	.ques-item{
		padding: 15px;
		background: #fff;
		margin: 20px 12px;
		border-radius: 10px;
	}
	
	.ques-item .subject{
		font-size: 15px;
		font-weight: 500;
		margin-bottom: 10px;
	}
	
	/* #ifndef APP-NVUE || VUE3*/
	::v-deep
	/* #endif */
	.ques-item .checklist-group{
		display: block!important;
	}
	/* #ifndef APP-NVUE || VUE3*/
	::v-deep
	/* #endif */
	.checklist-box{
		margin-top: 15px!important;
	}
	/* #ifndef APP-NVUE || VUE3*/
	::v-deep
	/* #endif */
	.checklist-text{
		font-size: 15px!important;
	}
	
	.subject .uni-icons{
		margin-right: 10px;
	}
	.ques-item .uniui-checkmarkempty{
		color: #18bc37!important;
	}
	.ques-item .uniui-closeempty{
		color: #e43d33!important;
	}
	
	.right-answer{
		color: #e43d33!important;
		border-top: 1px dashed lightgray;
		padding-top: 12px;
		margin-top: 20px;
	}
</style>

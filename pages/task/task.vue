<template>
	<view class="page">
		<view class="" style="height: 60rpx;"></view>
		<view class="slectebox">
			<uni-data-select
			  style="width: 40%;"
			  :value="value"
			  v-model="value"
			  :localdata="range"
			  :clear="false"
			  @change="selectChange"
			></uni-data-select>
			<view class="" style="margin-left: 40rpx;width: 400rpx;display: flex;justify-content: flex-end;">
				<button class="mini-btn" type="primary" size="mini" style="margin-left: auto;margin-right: 4rpx;" @click="operateFn" >operate</button>
				<button class="mini-btn" type="primary" size="mini" style="margin-right: 0;" @click="addCate()" >add cate</button>
			</view>
		</view>
	
		<uni-section title="Task" style="margin: 20rpx;background-color: transparent;position: relative;" type="line" >
				<view class="" style="display: flex;position: absolute;right: 20rpx;top: 12rpx;">
					
					<button class="mini-btn" type="primary" size="mini" style="margin-left: auto;" @click="addTask" >add task</button>
				</view>
		</uni-section>
		
		<view v-if="subTaskList.length" class="list">
			<view class="item" v-for="(item, index) in subTaskList" :key="index" @click="openOne(item, index)">
				<view class="" style="display: flex;align-items: center;justify-content: space-between;">
					<text style="font-weight: bold;">{{item.taskName}}</text>
					<uni-tag :text="item.cate" type="success" />
				</view>
				<uni-rate :readonly="true" :value="item.rate" />
			</view>
		</view>
		
		<view class="nodata" v-if="!subTaskList.length">
			<image src="/static/nodata.png" mode=""></image>
			<text>no data</text>
		</view>
		
		
		<uni-popup ref="operatePopup" :is-mask-click="false"  >
			<view class="operatebox">
				<view class="bottomoperate">
					<button class="mini-btn" type="default" size="mini" @click="cancelFn4()">cancel</button>
					<button class="mini-btn" type="primary" size="mini" style="background-color: red;" @click="confirmFn4()">confirm delete</button>
				</view>
				<select-tree @choose='choose' v-model='selectList' ref='selectTree'></select-tree>
			</view>
		</uni-popup>

		<uni-popup ref="addTaskPopup" :is-mask-click="false"  >
				<view class="popup-content">
					
					<view class="popupitem">
						<text style="color: red;margin-right: 4rpx;">*</text>
						<text class="left">task:</text>
						<input style="flex: 1;" class="uni-input" placeholder="Type a name for this task" v-model="operateItem.taskName" />
					</view>
					
					<view class="popupitem">
						<text class="left">score:</text>
						<uni-rate :max="5" v-model="operateItem.rate" :value="operateItem.rate" @change="(event) => {changeRate(operateItem, event)}"/>
					</view>
					
					<view class="popupitem">
						<text class="left">details:</text>
						<textarea v-model="operateItem.detail" placeholder="Please enter details" auto-height />
					</view>
					
					<view class="popupitem">

						<text style="color: red;margin-right: 4rpx;">*</text>
						<text class="left">category:</text>		
						<uni-data-select
						  style="flex: 1;"
						  v-model="operateItem.cate"
						  :localdata="subTaskCateList"
						  :clear="false"
						  @change="selectChange2"
						></uni-data-select>
						<button type="primary" 
						style="margin-left: 10rpx;height: 70rpx;font-size: 32rpx; padding: 0 20rpx ;display: flex;align-items: center;" 
						plain="true" @click="addSubCate">add subCate</button>
					</view>
					
					<view class="bottomoperate">
						<button class="mini-btn" type="default" size="mini" @click="cancelFn()">cancel</button>
						<button class="mini-btn" type="primary" size="mini" @click="confirmFn()">confirm</button>
					</view>
				</view>
		</uni-popup>
		
		<uni-popup ref="subCatePopup" :is-mask-click="false"  >
			<view class="addcatebox">
				<view class="ipt">
					<input style="flex: 1;" class="uni-input" placeholder="Type a subcate for this task" v-model="subcate" />
				</view>
				<view class="bottomoperate">
					<button class="mini-btn" type="default" size="mini" @click="cancelFn2">cancel</button>
					<button class="mini-btn" type="primary" size="mini" @click="confirmFn2">confirm</button>
				</view>
			</view>
		</uni-popup>
		
		<uni-popup ref="catePopup" :is-mask-click="false"  >
			<view class="addcatebox">
				<view class="ipt">
					<input style="flex: 1;" class="uni-input" placeholder="Type a cate for this task" v-model="cate" />
				</view>
				<view class="bottomoperate">
					<button class="mini-btn" type="default" size="mini" @click="cateCancelFn()">cancel1</button>
					<button class="mini-btn" type="primary" size="mini" @click="cateConfirmFn()">confirm2</button>
				</view>
			</view>
		</uni-popup>


	</view>
</template>

<script>
	import selectTree from "@/components/DLHTX-select_tree/select-tree"
	export default {
		components: {
			selectTree,
		},
		data() {
			return {
				value: '', 
				range: [ ],
				subTaskList: [],
				isEdit: false, 
				editIndex: 0,
				tempObj: {}, 
				subTaskCateList: [], 
				operateItem: { 
					  id: Date.now(),
					  rate: 0,
					  taskName: '',
					  detail: '',
					  cate: ''
				},
				selectList:  [
					{
						name: 'fruit',
						checked: false,
						show: false,
						childrenList: [
							{
								checked: false,
								name: 'watermelon',
							}, {
								checked: false,
								name: 'peach'
							}
						]
					},
					{
						name: 'tools',
						checked: false,
						show: false,
						childrenList: [
							{
								checked: false,
								name: 'knife'
							}, {
								checked: false,
								name: 'fork'
							}
						]

					}
				],
				subcate: '',
				cate: '',				  
			}
		},
		onTabItemTap() {
			console.log('onTabItemTap task',)
		},
		created() {
			console.log('created')
	
			this.getStoreFn()
		},
		methods: {
			choose(e){
				console.log(e)
			
			},
			cancelFn4() {
				this.$refs.selectTree.cancelAll()
				this.$refs.operatePopup.close()
			},
			confirmFn4() {
				console.log('confirmFn4')
				console.log(JSON.stringify(this.selectList))
				// this.$refs.selectTree.cancelAll()
				this.$refs.operatePopup.close()
			},
			operateFn() {
				console.log('operateFn')

				let showList = []
				let tempObj = JSON.parse(JSON.stringify(this.tempObj))
				console.log('tempObj', tempObj)
				let range = tempObj.range

				range.forEach(firstItem => {
					let showObj = {
						name: firstItem.value,
						checked: false,
						show: (firstItem.subTaskList && firstItem.subTaskList.length > 0) ? true : false,
						childrenList: []
					}
					if(firstItem.subTaskList) {
						firstItem.subTaskList.forEach(subItem => {
							showObj.childrenList.push({
								checked: false,
								name: `${subItem.cate} ${subItem.taskName ? '('+subItem.taskName+')' : ''}`
							})
						})
						
					}
					
					showList.push(showObj)
					
					
				})
				
				
				this.selectList = showList || []
				this.$refs.operatePopup.open('center')
			},
			firstInit() {
				let tempObj = {
						range: [
							{
								value: 'Mathematics',
								text: 'Mathematics',
								subTaskList: [
									{
										  taskName: 'Statistics',
										  rate: 3,
										  id: Date.now(),
										  detail: 'details',
										  cate: 'Statistics'
									},
									{
										  taskName: 'Algebra',
										  rate: 2,
										  id: Date.now(),
										  detail: 'details',
										  cate: 'Algebra'
									},
									{
										  taskName: 'Geometry',
										  rate: 4,
										  id: Date.now(),
										detail: 'details',
										cate: 'Geometry'
									},
								],
								subTaskCateList: [
									{ value: "Statistics", text: "Statistics" },
									{ value: "Algebra", text: "Algebra" },
									{ value: "Geometry", text: "Geometry" },
								]
							}
						]
					}
					this.tempObj = JSON.parse(JSON.stringify(tempObj))
				this.range = tempObj.range
	
				this.tempObj = tempObj
				let temp = tempObj.range[0] || {}
				this.value = temp.text
				this.subTaskList =  temp.subTaskList
				this.subTaskCateList = temp.subTaskCateList
			},
			hasValueInit(obj = {}) {
				this.tempObj = obj
				let temp = obj.range[0] || {}
				this.value = temp.text
				this.range = obj.range
				console.log('hasValueInit',obj, this.value, temp, this.range)
				
				this.subTaskList = temp.subTaskList
				this.subTaskCateList = temp.subTaskCateList
			},
			getStoreFn() { 
				try {
				    var value = uni.getStorageSync('key');
				    if (value) {
				        console.log('获取到值getStoreFn value', value); 
						this.hasValueInit(JSON.parse(value) || {})
				    } else {
						console.log('没有值')
						this.firstInit()
					}
				} catch (e) {
					console.log('异常')
			
					this.firstInit()
				    console.error(e);
				}
			},
			saveFn(type) { 
			console.log('saveFn', type)
				if(type == 'task') { 
					try {
						let temp = JSON.parse(JSON.stringify(this.tempObj))
						let index = this.range.findIndex(item => item.text == this.value)
						if(index > -1) {

							temp.range[index].subTaskList = this.subTaskList
							uni.setStorageSync('key', JSON.stringify(temp));
							this.tempObj = temp
							console.log('success');
						} else {
							uni.showToast({
								icon:'none',
								title: 'ex1'
							})
						}
					} catch (e) {
	
						uni.showToast({
							icon:'none',
							title: 'ex2'
						})
					    console.error(e);
					}
				} else if(type == 'cate') { 
					try {
						let temp = JSON.parse(JSON.stringify(this.tempObj))
						console.log('temp', temp, this.cate)
						temp.range.push({
							value: this.cate,
							text: this.cate,
							subTaskList: [],
							subTaskCateList: []
						})
						uni.setStorageSync('key', JSON.stringify(temp));
						this.tempObj = temp
						console.log('temp key', temp)
						
						
					} catch (e) {
			
						uni.showToast({
							icon:'none',
							title: 'save ex'
						})
					    console.error(e);
					}
				} else if(type == 'subcate') { 
					try {
						let temp = JSON.parse(JSON.stringify(this.tempObj))
						
						
						let index = this.range.findIndex(item => item.text == this.value)
						if(index > -1) {
							temp.range[index].subTaskCateList = this.subTaskCateList
							uni.setStorageSync('key', JSON.stringify(temp));
							this.tempObj = temp
						} else {
							uni.showToast({
								title: 'ex1'
							})
						}
						
					
					} catch (e) {
	
						uni.showToast({
							icon:'none',
							title: 'ex'
						})
					    console.error(e);
					}
				}
				

			},
			selectChange(value) {
			  console.log("e:", value);

			  this.value = value
			  let index = this.tempObj.range.findIndex(item => item.text == value)
			  if(index > -1) {
				 this.cate = ''
				 this.subcate = ''
				 let temp = this.tempObj.range[index]
				
				 this.range = this.tempObj.range
				 
				 this.subTaskList = temp.subTaskList
				 this.subTaskCateList = temp.subTaskCateList
				  
			  }
			  
			},
			addTask() { 
				this.isEdit = false
				this.operateItem = {
					id: '',
					rate:  0,
					taskName: '',
					detail: '',
					cate: ''
				}
				this.$refs.addTaskPopup.open('center')
			},
			openOne(item, index) {
				console.log('openOne')
				this.isEdit = true
				this.editIndex = index
				this.operateItem = {
					id: item.id || Date.now(),
					rate: item.rate || 0,
					taskName: item.taskName || '',
					detail: item.detail || '',
					cate: item.cate
				}
				this.$refs.addTaskPopup.open('center')
				
				
			},
			
			cancelFn() {
				this.$refs.addTaskPopup.close('center')
			},
			confirmFn() {
				console.log('confirmFn')
				if(!this.operateItem.taskName) {
					uni.showToast({
						icon:'none',
						title: 'Task cannot be empty'
					})
					return
				}
				console.log('isEdit', this.isEdit)
				if(!this.isEdit) {

					let taskNameIndex = this.subTaskList.findIndex((item) => item.taskName == this.operateItem.taskName)
					if(taskNameIndex > -1) {
						uni.showToast({
							icon:'none',
							title: 'Task already exists'
						})
						return
					}
					let cateIndex = this.subTaskList.findIndex((item) => item.cate == this.operateItem.cate)
					if(cateIndex > -1) {
						uni.showToast({
							icon:'none',
							title: 'This subcategory has been selected'
						})
						return
					}
					if(!this.operateItem.cate) {
						uni.showToast({
							icon:'none',
							title: 'Please select subcategory'
						})
						return
					}
					let obj = {
						  taskName: this.operateItem.taskName || '',
						  rate: this.operateItem.rate || 0,
						  id: this.isEdit ? this.operateItem.id : Date.now(),
						  detail: this.operateItem.detail || '',
						  cate: this.operateItem.cate || ''
					}
					console.log('obj', obj)
					this.subTaskList.push(obj)
				} else {

					let cateIndex = this.subTaskList.findIndex((item) => item.cate == this.operateItem.cate)
					if(cateIndex > -1) {
						if(this.editIndex != cateIndex) {
							uni.showToast({
								icon:'none',
								title: 'This subcategory has been selected'
							})
							return
						}
			
					}
					
					let obj = {
						  taskName: this.operateItem.taskName || '',
						  rate: this.operateItem.rate || 0,
						  id: this.isEdit ? this.operateItem.id : Date.now(),
						  detail: this.operateItem.detail || '',
						  cate: this.operateItem.cate || ''
					}
					console.log('obj', obj)
					this.subTaskList.splice(this.editIndex, 1, obj)
				}
				this.saveFn('task')

				this.$refs.addTaskPopup.close('center')
			},
			selectChange2() {
				
			},
			changeRate(obj, event) {
				obj.rate = event.value
			},

			
			addCate() {
				console.log('addCate')
				this.cate = ''
				this.$refs.catePopup.open('center')
			},
			cateCancelFn() {
				this.$refs.catePopup.close('center')
			},
			cateConfirmFn() {
				console.log('confirmFn3')
				let index = this.range.findIndex(item => item.value == this.cate)
				if(index > -1) {
					uni.showToast({
						icon:'none',
						title: 'Already exists'
					})
					return;
				}
				let obj = { value: this.cate, text: this.cate }
				console.log('obj', obj)
				this.range.push(obj)
				console.log('this.range', this.range)
				this.saveFn('cate')
				this.$refs.catePopup.close('center')
			},

			addSubCate() {
				console.log()
				this.$refs.subCatePopup.open('center')
			},
			cancelFn2() {
				this.$refs.subCatePopup.close('center')
			},
			confirmFn2() {
				
				
				let index = this.subTaskCateList.findIndex(item => item.value == this.subcate)
				if(index > -1) {
					uni.showToast({
						icon:'none',
						title: 'Already exists'
					})
					return;
				}
				let obj = { value: this.subcate, text: this.subcate }
				console.log('obj', obj)
				this.subTaskCateList.push(obj)
				console.log('this.subTaskCateList', this.subTaskCateList)
				this.saveFn('subcate')
				
				this.$refs.subCatePopup.close('center')
			},
	
		}
	}
</script>

<style lang="less" scoped>
	page {
		height: 100%;
		background-color: rgb(239,243,246);
	}
	.page {
		height: 100%;
		position: relative;
		display: flex;
		flex-direction: column;
		
	}
	.slectebox {
		padding: 40rpx;

		display: flex;
		align-items: center;
		margin-bottom: 20rpx;
	}
	.list {
		padding: 20rpx;
		flex: 1;
		overflow-y: auto;
		.item {
			padding: 20rpx;
			background-color: #fff;
			border-radius: 30rpx;
			margin-bottom: 30rpx;
		}
	}
	
	.nodata {
		width: 85vw;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		margin-top: 60rpx;
		image {
			width: 318rpx;
			height: 200rpx;
			margin-bottom: 20rpx;
		}
		text {
			font-size: 38rpx;
			color: #ccc;
		}
	}
	
	
	.operatebox {
		background-color: #fff;
		border-radius: 10rpx;
		width: 85vw;
		margin: 0 auto;
		.bottomoperate {
			display: flex;
			justify-content: center;
			padding: 20rpx;
			button {
				margin: 0 20rpx;
			}
		}
	}
	
	
	.addcatebox {
		background-color: #fff;
		border-radius: 10rpx;
		width: 85vw;
		margin: 0 auto;
		.ipt {
			padding: 20rpx;
		}
		.bottomoperate {
			display: flex;
			justify-content: center;
			padding: 20rpx;
			button {
				margin: 0 20rpx;
			}
		}
	}
	
	
	.popup-content {
		background-color: #fff;
		border-radius: 10rpx;
		width: 85%;
		margin: 0 auto;
		.popupitem {
			display: flex;
			align-items: center;
			padding: 20rpx;

			.left {
				width: 200rpx;
			}
			
			textarea {
				flex: 1;
			}
		}
		.bottomoperate {
			display: flex;
			justify-content: center;
			padding: 20rpx;
			button {
				margin: 0 20rpx;
			}
		}
	}
	
	

</style>

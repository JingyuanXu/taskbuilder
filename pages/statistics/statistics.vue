<template>
	<view class="page">
		<view class="" style="height: 60rpx;">
			
		</view>
		<view class="slectebox">
			<uni-data-select
			  style="width: 40%;"
			  v-model="value"
			  :localdata="range"
			  :clear="false"
			  @change="selectChange"
			  placeholder="please select"
			  emptyTips="no data"
			></uni-data-select>
			<!-- Total score-->
			<text style="margin-left: 40rpx;width: 400rpx">Total score: {{allScore ? allScore.toFixed(0) : 0}}%</text>
		</view>
		<view class="nodata" v-if="!range.length">
			<image src="/static/nodata.png" mode=""></image>
			<text>no data</text>
		</view>
		<view class="charts-box" v-else>
		    <qiun-data-charts type="column" :chartData="chartData" :opts="chartOptions"   />
		  </view>
		  
<!-- 		  <div>read:{{readTxt}}</div>
		  
		  <div @click="openBokeHuiyiFolder()">open folder</div> -->
		  
		  <!-- <uni-link :href="`mailto:${'test@qq.com'}?subject=${encodeURIComponent('task')}&body=${encodeURIComponent(this.showObj)}`" text="sent" font-size="30"></uni-link> -->
		<!--  <div style="display: flex;justify-content: center;">
			  <uni-link :href="`mailto:${'test@qq.com'}?subject=${encodeURIComponent('task xml')}&body=${getResult()}`" text="send email" font-size="30"></uni-link>
		  </div> -->
		  <!-- <div><uni-link href="mailto:no-reply@xyz.org" text="send to" font-size="30"></uni-link></div> -->
		  <button @tap="sendEmail">sendEmail</button>
		  <!-- <uni-link href="https://uniapp.dcloud.io/" text="https://uniapp.dcloud.io/"></uni-link> -->
		  <div href="#" @click="exportFn()" class="preview-btn" id="previewBtn">
		  	<!-- export xml -->
			Export XML
		  </div>
	</view>
</template>

<script>
	
	function convertToXML(data) {
	  let xml = '<?xml version="1.0" encoding="UTF-8"?>\n';
	  xml += '<task>\n';
	  
	  
	  xml += `  <result>${escapeXML(data.result)}</result>\n`;
	  xml += `  <taskName>${escapeXML(data.taskName)}</taskName>\n`;
	  
	 
	  if (data.secordTask && data.secordTask.length) {
	    xml += '  <secordTask>\n';
	    data.secordTask.forEach(task => {
	      xml += '    <item>\n';
	      xml += `      <taskName>${escapeXML(task.taskName)}</taskName>\n`;
	      xml += `      <result>${escapeXML(task.result.toString())}</result>\n`;
	      xml += '    </item>\n';
	    });
	    xml += '  </secordTask>\n';
	  }
	  
	  xml += '</task>';
	  return xml;
	}
	
	
	function escapeXML(str) {
	  return str
	    .replace(/&/g, '&amp;')
	    .replace(/</g, '&lt;')
	    .replace(/>/g, '&gt;')
	    .replace(/"/g, '&quot;')
	    .replace(/'/g, '&apos;');
	}
	
	export default {
		data() {
			return {
				chartOptions: {  				  yAxis:{
					  data: [{  min: 0, max: 5, splitNumber: 1,}]
				  }
				},
				chartData: {},
				  range: [
				  ],
				  value: '',
				  allScore: 0,
				  tempObj: {}, 
				  
				  
				  readTxt: '',
				  showObj: ''
			}
		},
		created() {
			
			this.getStoreFn()
			
			plus.android.requestPermissions([
			'android.permission.WRITE_EXTERNAL_STORAGE',
			'android.permission.READ_EXTERNAL_STORAGE',
			'android.permission.INTERNET',
			'android.permission.ACCESS_WIFI_STATE'
			], e => {
			
		
			}, v => {
			 uni.showToast({
				title:'failed'
			 })
			})
			
		},
		onTabItemTap() {
			console.log('onTabItemTap statistics',)
			this.getStoreFn()
		},
		mounted() {
			
			

		},
		methods: {
			getResult() {
				const showObj = {
				  result: this.allScore ? this.allScore.toFixed(0) : 0,
				  taskName: '',
				  secordTask: [
				  ]
				};
				
				let scoreList = []
				let nameList = []
				let obj = this.tempObj 
				let index = obj.range.findIndex(item => item.value == this.value)
				if(index > -1) {
					showObj.taskName = obj.range[index].text
					let temp = obj.range[index] || {}
					temp.subTaskCateList.forEach((item, ind2) => {
						let score = 0
					
						let ind = temp.subTaskList.findIndex(itm => itm.cate == item.value)
						if(ind > -1) {
							score = temp.subTaskList[ind].rate
						}
						showObj.secordTask.push({
							taskName: item.value,
							 result: score
						})							
					})
				}
				console.log('showObj', showObj)
				const xmlString = convertToXML(showObj)
				console.log('xmlString', xmlString)
				console.log('zwwq', encodeURIComponent(xmlString))
				return encodeURIComponent(xmlString)
			},
			sendEmail() {
				const showObj = {
				  result: this.allScore ? this.allScore.toFixed(0) : 0,
				  taskName: '',
				  secordTask: [

				  ]
				};
				
				let scoreList = []
				let nameList = []
				let obj = this.tempObj 
				let index = obj.range.findIndex(item => item.value == this.value)
				if(index > -1) {
					showObj.taskName = obj.range[index].text
					let temp = obj.range[index] || {}
					temp.subTaskCateList.forEach((item, ind2) => {
						let score = 0
						// cate rate 
						let ind = temp.subTaskList.findIndex(itm => itm.cate == item.value)
						if(ind > -1) {
							score = temp.subTaskList[ind].rate
						}
						showObj.secordTask.push({
							taskName: item.value,
							 result: score
						})							
					})
				}
				console.log('showObj', showObj)
				const xmlString = convertToXML(showObj)
				console.log('xmlString', xmlString)
				console.log('zwwq', encodeURIComponent(xmlString))
				var mail = plus.messaging.createMessage(plus.messaging.TYPE_EMAIL);
						mail.to = ["no-reply@xyz.org"];
						mail.subject = "xml";
						mail.body = xmlString;
						mail.bodyType = "text/html";
				
						plus.messaging.sendMessage(mail, function() {
							console.log("test");
						}, function(e) {
							console.log(e);
						});
				

			},
			firstInit() {
				let tempObj = {
						range: [
							{
								value: 'test',
								text: 'test',
								subTaskList: [
									{
										  taskName: 'test',
										  rate: 3,
										  id: Date.now(),
										  detail: 'test',
										  cate: 'test'
									},
									{
										  taskName: 'test2',
										  rate: 2,
										  id: Date.now(),
										  detail: 'test2',
										  cate: 'test2'
									},
									{
										  taskName: 'test3',
										  rate: 4,
										  id: Date.now(),
										detail: 'test3',
										cate: 'test3'
									},
								],
								subTaskCateList: [
									{ value: "test", text: "test" },
									{ value: "test2", text: "test2" },
									{ value: "test3", text: "test3" },
								]
							}
						]
					}
					this.tempObj = JSON.parse(JSON.stringify(tempObj))
	
				this.range = tempObj.range
		
				let temp = tempObj.range[0] || {}
				this.value = temp.text
				let nameList = ['test', 'test2', 'test3']
				let scoreList = [3,2,4]
				
				this.getServerData(nameList, scoreList)
				console.log('2222')
				this.allScore = eval(scoreList.join('+')) / (scoreList.length * 5) * 100

			},
			hasValueInit(obj = {}) {
				this.tempObj = obj
				
				
				this.range = obj.range
				console.log('hasValueInit',obj, this.value,  this.range)
				// let index = obj.range.findIndex(item => item.value == obj.range[0].value)
				let index = 0
				console.log('index', index)
				if(index > -1) {
					
					let temp = obj.range[index] || {}
					this.value = temp.text
					let tempList = []
					let scoreList = []
					let nameList = []
					console.log('1111111')
					temp.subTaskCateList.forEach(item => {
						let score = 0
				
						let ind = temp.subTaskList.findIndex(itm => itm.cate == item.value)
						if(ind > -1) {
							score = temp.subTaskList[ind].rate
						}
						scoreList.push(score)
						nameList.push(item.value)

						
					})
					console.log('aaaaaa', nameList, scoreList)
					this.getServerData(nameList, scoreList)
					console.log('2222')
					this.allScore = eval(scoreList.join('+')) / (scoreList.length * 5) * 100
				}
				

			},
			getStoreFn() {
				try {
				    var value = uni.getStorageSync('key');
				    if (value) {
						this.hasValueInit(JSON.parse(value) || {})
				    } else {
						this.firstInit()
					}
				} catch (e) {
					this.firstInit()
				    console.error(e);
				}
			},
			selectChange(value) {
			  console.log("e:", value);
			  
			  
			  
			  let obj = this.tempObj 
			  
			  
			  this.range = obj.range
			  console.log('hasValueInit',obj, this.value,  this.range)
			  let index = obj.range.findIndex(item => item.value == value)
			  // let index = 0
			  console.log('index', index)
			  if(index > -1) {
			  	
			  	let temp = obj.range[index] || {}
			  	this.value = temp.text
			  	let tempList = []
			  	let scoreList = []
			  	let nameList = []
			  	console.log('1111111')
			  	temp.subTaskCateList.forEach(item => {
			  		let score = 0
			  		let ind = temp.subTaskList.findIndex(itm => itm.cate == item.value)
			  		if(ind > -1) {
			  			score = temp.subTaskList[ind].rate
			  		}
			  		scoreList.push(score)
			  		nameList.push(item.value)

			  		
			  	})
			  	console.log('aaaaaa', nameList, scoreList)
			  	this.getServerData(nameList, scoreList)
			  	console.log('2222')
			  	this.allScore = eval(scoreList.join('+')) / (scoreList.length * 5) * 100
			  }
			  

			},
			getServerData(categoriesList = [], seriesList = []) {
			      setTimeout(() => {
			        let res = {
			            categories: categoriesList,
			            series: [
			              {
			                name: "Secondary Category", // Secondary Category 二级类别
			                data: seriesList,
							// yAxis: [{min:0,max:5, interval: 1}]
							// yAxis: {
							// 	max: 5,     
							// 	min: 0,     
							// 	interval: 1 
							// }
			              },
			            ]
			          };
			        this.chartData = JSON.parse(JSON.stringify(res));
			      }, 500);
			    },
				
				generateFn() {
					
					const obj = {
					  result: '60%',
					  taskName: 'test',
					  secordTask: [
					    { taskName: 'test', result: 3 },
					    { taskName: 'test2', result: 2 },
					    { taskName: 'test3', result: 4 }
					  ]
					};
					const xmlString = convertToXML(obj)
					console.log('xmlString', xmlString)
				},
				
				
				exportFn() {
					console.log('exportFn')	
					
					const showObj = {
					  result: this.allScore ? this.allScore.toFixed(0) : 0,
					  taskName: '',
					  secordTask: [
					    // { taskName: 'test', result: 3 },
					    // { taskName: 'test2', result: 2 },
					    // { taskName: 'test3', result: 4 }
					  ]
					};
					
					let scoreList = []
					let nameList = []
					let obj = this.tempObj 
					let index = obj.range.findIndex(item => item.value == this.value)
					if(index > -1) {
						showObj.taskName = obj.range[index].text
						let temp = obj.range[index] || {}
						temp.subTaskCateList.forEach((item, ind2) => {
							let score = 0
							// cate rate 
							let ind = temp.subTaskList.findIndex(itm => itm.cate == item.value)
							if(ind > -1) {
								score = temp.subTaskList[ind].rate
							}
							showObj.secordTask.push({
								taskName: item.value,
								 result: score
							})							
						})
					}
					console.log('showObj', showObj)
					const xmlString = convertToXML(showObj)
					this.showObj = JSON.stringify(showObj)
					console.log('xmlString', xmlString)
				},
				
				readFN2() {
					let that = this
					let pathUrl = '/index.txt';    
					let environment = plus.android.importClass("android.os.Environment");  
					var sdRoot = environment.getExternalStorageDirectory(); 
					var File = plus.android.importClass("java.io.File");  
					var BufferedReader = plus.android.importClass("java.io.BufferedReader");  
					var FileReader = plus.android.importClass("java.io.FileReader");  
					var FileWriter = plus.android.importClass("java.io.FileWriter");  
		
					let readFr = new File(sdRoot + pathUrl);  
					let txt = '';  
					try {  
						var reader = new BufferedReader(new FileReader(readFr))  
						
						let arr = [];  
						let txt;  
						while ((txt = reader.readLine()) != null) {  
							arr.push(txt)  
						}  
						that.readTxt = arr.join('')
											} catch (e) {  
		
					}
				},
				openBokeHuiyiFolder() {
				    this.generateFn()
					
				    let that = this;
				    let main = plus.android.runtimeMainActivity();
				    
				    let Intent = plus.android.importClass('android.content.Intent');
				    let Activity = plus.android.importClass('android.app.Activity');
				    let Build = plus.android.importClass('android.os.Build');
				    let DocumentsContract = plus.android.importClass('android.provider.DocumentsContract');
				    let Environment = plus.android.importClass('android.os.Environment');
				    let Uri = plus.android.importClass('android.net.Uri');
				    let File = plus.android.importClass('java.io.File');
				    
				   
				    let rootPath = Environment.getExternalStorageDirectory();
				    let targetPath = new File(rootPath, "boke", "huiyi");
					
				   
				    if (!targetPath.exists()) {
				        targetPath.mkdirs();
				    }
				    
				    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
				        try {
				            let intent = new Intent(Intent.ACTION_OPEN_DOCUMENT_TREE);
				            
				            
				            let targetUri = getTreeUriForDirectory(targetPath);
				            
				            
				            intent.putExtra(DocumentsContract.EXTRA_INITIAL_URI, targetUri);
				            
				            main.startActivityForResult(intent, 203);
				            
				            main.onActivityResult = function(requestCode, resultCode, data) {
				                if (requestCode === 203 && resultCode === Activity.RESULT_OK) {

				                    that.$emit('folder-accessed', targetPath.getAbsolutePath());
				                } else {
				                }
				            };
				        } catch (e) {
				            openParentFolder();
				        }
				    } else {
				        openParentFolder();
				    }
				    
				    function getTreeUriForDirectory(targetDir) {
				        try {
				            let MediaStore = plus.android.importClass('android.provider.MediaStore');
				            let ContentUris = plus.android.importClass('android.content.ContentUris');
				         
				            let externalUri = MediaStore.Files.getContentUri("external");
				          
				            let projection = ["_id"];
				            let selection = MediaStore.MediaColumns.DATA + "=?";
				            let selectionArgs = [targetDir.getAbsolutePath()];
				            
				            let cursor = main.getContentResolver().query(
				                externalUri, 
				                projection, 
				                selection, 
				                selectionArgs, 
				                null
				            );
				            
				            if (cursor && cursor.moveToFirst()) {
				                let id = cursor.getLong(0);
				                return ContentUris.withAppendedId(externalUri, id);
				            }
				            
				            let authority = "com.android.externalstorage.documents";
				            
				            let treePath = targetDir.getAbsolutePath()
				                .replace(Environment.getExternalStorageDirectory().getAbsolutePath(), "")
				                .replace(File.separator, ":");
				                
				            if (treePath.startsWith(":")) treePath = treePath.substring(1);
				                
				            return Uri.parse("content://" + authority + "/tree/primary:" + treePath);
				        } catch (e) {
				            return Uri.fromFile(targetDir);
				        }
				    }
				    
				    function openParentFolder() {
				        try {
				            let intent = new Intent(Intent.ACTION_OPEN_DOCUMENT_TREE);
				          
				            let parentDir = new File(rootPath, "boke");
				            let parentUri = getTreeUriForDirectory(parentDir);
				            
				            intent.putExtra(DocumentsContract.EXTRA_INITIAL_URI, parentUri);
				            main.startActivityForResult(intent, 204);
				        } catch (e) {
				        }
				    }
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
	}
	
	.slectebox {
		padding: 40rpx;
		/* padding-left: 40rpx; */
		display: flex;
		align-items: center;
		margin-bottom: 20rpx;
	}
  .charts-box {
    width: 100%;
    height: 300px;
  }
  
  .nodata {
  	width: 100%;
  	display: flex;
  	flex-direction: column;
  	justify-content: center;
  	align-items: center;
	margin-top: 80rpx;
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
  
  .preview-btn:hover {
  	            transform: translateY(-5px) scale(1.05);
  	            box-shadow: 0 12px 30px rgba(255, 111, 0, 0.7);
  	        }
  	        
  	        .preview-btn i {
  	            font-size: 1.5rem;
  	            margin-left: 10px;
  	        }
  	
  	        .preview-btn {
  	            position: fixed;
  	            /* bottom: 10%; */
  		/* 		width: 13.2%;
  				height: 13.%; */
				width: 60px;
				height: 60px;
				border-radius: 50%;
				right: 40px;
				bottom: 80px;
  				/* left: 50%; */
  				/* margin-left: -3.3%; */
  	            // right: 40px;
  	            // background: linear-gradient(135deg, #ff6f00, #ff9800);
  				background-color: rgb(238,130,57);
  	            color: white;
  				text-align: center;
  	            // padding: 18px 30px;
  	            /* border-radius: 10%; */
  	            font-size: 12px;
  	            font-weight: 600;
  	            display: flex;
  	            align-items: center;
  	            text-decoration: none;
  	            box-shadow: 0 8px 25px rgba(255, 111, 0, 0.5);
  	            transition: all 0.3s;
  				display: flex;
  				justify-content: center;
  	            z-index: 100;
  	        }
</style>

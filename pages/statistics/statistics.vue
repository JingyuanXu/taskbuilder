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
			></uni-data-select>
			<text style="margin-left: 40rpx;width: 400rpx">Total score: {{allScore.toFixed(0)}}%</text>
		</view>
		<view class="nodata" v-if="!range.length">
			<image src="/static/nodata.png" mode=""></image>
			<text>no data</text>
		</view>
		<view class="charts-box" v-else>
		    <qiun-data-charts type="column" :chartData="chartData" :opts="chartOptions"   />
		  </view>
		  

		  
		  <div href="#" @click="exportFn()" class="preview-btn" id="previewBtn">
		  	<!-- xml -->
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
				chartOptions: {  
				  yAxis:{
					  data: [{  min: 0, max: 5, splitNumber: 1,}]
				  }
				},
				chartData: {},
				  range: [

				  ],
				  value: '',
				  allScore: 0,
				  tempObj: {}, 
				  
				  
				  readTxt: ''
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
										  detail: '',
										  cate: 'Algebra'
									},
									{
										  taskName: 'Geometry',
										  rate: 4,
										  id: Date.now(),
										detail: '',
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

				let temp = tempObj.range[0] || {}
				this.value = temp.text
				let nameList = ['Statistics', 'Algebra', 'Geometry']
				let scoreList = [3,2,4]
				
				this.getServerData(nameList, scoreList)
				console.log('2222')
				this.allScore = eval(scoreList.join('+')) / (scoreList.length * 5) * 100

			},
			hasValueInit(obj = {}) {
				this.tempObj = obj
				
				
				this.range = obj.range
				console.log('hasValueInit',obj, this.value,  this.range)
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
				        console.log('getStoreFn value', value); 
						this.hasValueInit(JSON.parse(value) || {})
				    } else {
						console.log('no value')
						this.firstInit()
					}
				} catch (e) {
					console.log('exceptions')
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
			                name: "Secondary Category", 
			                data: seriesList,

			              },
			            ]
			          };
			        this.chartData = JSON.parse(JSON.stringify(res));
			      }, 500);
			    },
				
				generateFn() {
					const obj = {
					  result: '60%',
					  taskName: 'Mathematics',
					  secordTask: [
					    { taskName: 'Statistics', result: 3 },
					    { taskName: 'Algebra', result: 2 },
					    { taskName: 'Geometry', result: 4 }
					  ]
					};
					const xmlString = convertToXML(obj)
					console.log('xmlString', xmlString)
				},
				
				exportFn() {
					
					const obj = {
					  result: '60%',
					  taskName: 'Mathematics',
					  secordTask: [
					    { taskName: 'Statistics', result: 3 },
					    { taskName: 'Algebra', result: 2 },
					    { taskName: 'Geometry', result: 4 }
					  ]
					};
					const xmlString = convertToXML(obj)
					
					let environment = plus.android.importClass("android.os.Environment");
					var sdRoot = environment.getExternalStorageDirectory(); //文件夹根目录  
					
					let path = sdRoot + '/task.xml'
					
					const File = plus.android.importClass('java.io.File');
						const FileOutputStream = plus.android.importClass('java.io.FileOutputStream');
						const OutputStreamWriter = plus.android.importClass('java.io.OutputStreamWriter');
					 
						 const isFileExist = function(path = ''){
							const File = plus.android.importClass('java.io.File');
							return new File(path).exists()
						 }
					 
						let outputStreamWriter;
						let file = new File(path);
						try {
							if (!file.exists()) {
								file.createNewFile();
							}
							outputStreamWriter = new OutputStreamWriter(new FileOutputStream(path, false), 'utf-8');
							outputStreamWriter.write(xmlString);
							outputStreamWriter.close();
						} catch (e) {
							if (null != outputStreamWriter) {
								outputStreamWriter.close();
							}
							return false;
						}

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
				            
				            console.log("navigate: " + targetPath.getAbsolutePath());
				            main.startActivityForResult(intent, 203);
				            
				            main.onActivityResult = function(requestCode, resultCode, data) {
				                if (requestCode === 203 && resultCode === Activity.RESULT_OK) {
				                    console.log("success:", targetPath.getAbsolutePath());
				                    that.$emit('folder-accessed', targetPath.getAbsolutePath());
				                } else {
				                    console.log("cancel");
				                }
				            };
				        } catch (e) {
				            console.error("error:", e);
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
				            console.error("form URI failed:", e);
				            
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
				            console.error("open file path failed:", e);
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
				width: 60px;
				height: 60px;
				border-radius: 50%;
				right: 40px;
				bottom: 80px;
  				background-color: rgb(238,130,57);
  	            color: white;
  				text-align: center;
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

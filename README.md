呼吸灯美化
int brightness=0;
int lightPlus=3;
void setup()
{
  pinMode(9,OUTPUT);
  pinMode(10,OUTPUT);
  pinMode(11,OUTPUT);

}
void loop()
{
  do{
  analogWrite(9,brightness);
  brightness = brightness + lightPlus;
  if(brightness==255 || brightness==0)
  lightPlus = -lightPlus;
  analogWrite(9,brightness);
  delay(10);
  }
  while(brightness>0);

  do{
  analogWrite(10,brightness);
  brightness = brightness + lightPlus;
  if(brightness==255 || brightness==0)
  lightPlus = -lightPlus;
  analogWrite(10,brightness);
  delay(10);
  }
  while(brightness>0);
  do{
  analogWrite(11,brightness);
  brightness = brightness + lightPlus;
  if(brightness==255 || brightness==0)
  lightPlus = -lightPlus;
  analogWrite(11,brightness);
  delay(10);
  }
  while(brightness>0);

}

wxss
.list{
font-size:35rpx;
font-style:oblique;
background:white;
margin-top:19rpx;
padding-top:15rpx;
border-left-width:10rpx;
border-right-width:10rpx;
text-align:left;
size:2000rpxdefault;
width:500%;
top:0%;
opacity:200%;
border-style:solid;
border-color:#ff0000#0000ff;
}
晏得鑫：温度感受器
var myCharts = require(“../../../utils/wxcharts.js”)//引入一个绘图的插件

const devicesId = “562141708” // 填写在OneNet上获得的devicesId 形式就是一串数字 例子:9939133
const api_key = “i2ab=rQ7M1D0R1G5TZ7=iyOm1mg=” // 填写在OneNet上的 api-key 例子: VeFI0HZ44Qn5dZO14AuLbWSlSlI=

Page({
data: {},

/**
* @description 页面下拉刷新事件
*/
onPullDownRefresh: function () {
wx.showLoading({
title: “正在获取”
})
this.getDatapoints().then(datapoints => {
this.update(datapoints)
wx.hideLoading()
}).catch((error) => {
wx.hideLoading()
console.error(error)
})
},

/**
* @description 页面加载生命周期
*/
onLoad: function () {
console.log(`your deviceId: ${devicesId}, apiKey: ${api_key}`)

//每隔6s自动获取一次数据进行更新
const timer = setInterval(() => {
this.getDatapoints().then(datapoints => {
this.update(datapoints)
})
}, 6000)

wx.showLoading({
title: ‘加载中’
})

this.getDatapoints().then((datapoints) => {
wx.hideLoading()
this.firstDraw(datapoints)
}).catch((err) => {
wx.hideLoading()
console.error(err)
clearInterval(timer) //首次渲染发生错误时禁止自动刷新
})
},

/**
* 向OneNet请求当前设备的数据点
* @returns Promise
*/
getDatapoints: function () {
returnnewPromise((resolve, reject) => {
wx.request({
url: `https://api.heclouds.com/devices/${devicesId}/datapoints?datastream_id=Light,Temperature,Humidity&limit=20`,
/**
* 添加HTTP报文的请求头,
* 其中api-key为OneNet的api文档要求我们添加的鉴权秘钥
* Content-Type的作用是标识请求体的格式, 从api文档中我们读到请求体是json格式的
* 故content-type属性应设置为application/json
*/
header: {
‘content-type’: ‘application/json’,
‘api-key’: api_key
},
success: (res) => {
const status = res.statusCode
const response = res.data
if (status !== 200) { // 返回状态码不为200时将Promise置为reject状态
reject(res.data)
return ;
}
if (response.errno !== 0) { //errno不为零说明可能参数有误, 将Promise置为reject
reject(response.error)
return ;
}

if (response.data.datastreams.length === 0) {
reject(“当前设备无数据, 请先运行硬件实验”)
}

//程序可以运行到这里说明请求成功, 将Promise置为resolve状态
resolve({
temperature: response.data.datastreams[0].datapoints.reverse(),
light: response.data.datastreams[1].datapoints.reverse(),
humidity: response.data.datastreams[2].datapoints.reverse()
})
},
fail: (err) => {
reject(err)
}
})
})
},

/**
* @param {Object[]} datapoints 从OneNet云平台上获取到的数据点
* 传入获取到的数据点, 函数自动更新图标
*/
update: function (datapoints) {
const wheatherData = this.convert(datapoints);

this.lineChart_hum.updateData({
categories: wheatherData.categories,
series: [{
name: ‘humidity’,
data: wheatherData.humidity,
format: (val, name) => val.toFixed(2)
}],
})

this.lineChart_light.updateData({
categories: wheatherData.categories,
series: [{
name: ‘light’,
data: wheatherData.light,
format: (val, name) => val.toFixed(2)
}],
})

this.lineChart_tempe.updateData({
categories: wheatherData.categories,
series: [{
name: ‘tempe’,
data: wheatherData.tempe,
format: (val, name) => val.toFixed(2)
}],
})

},

/**
*
* @param {Object[]} datapoints 从OneNet云平台上获取到的数据点
* 传入数据点, 返回使用于图表的数据格式
*/
convert: function (datapoints) {
var categories = [];
var humidity = [];
var light = [];
var tempe = [];

var length = datapoints.humidity.length
for (var i = 0; i < length; i++) {
categories.push(datapoints.humidity[i].at.slice(5, 19));
humidity.push(datapoints.humidity[i].value);
light.push(datapoints.light[i].value);
tempe.push(datapoints.temperature[i].value);
}
return {
categories: categories,
humidity: humidity,
light: light,
tempe: tempe
}
},

/**
*
* @param {Object[]} datapoints 从OneNet云平台上获取到的数据点
* 传入数据点, 函数将进行图表的初始化渲染
*/
firstDraw: function (datapoints) {

//得到屏幕宽度
var windowWidth = 320;
try {
var res = wx.getSystemInfoSync();
windowWidth = res.windowWidth;
} catch (e) {
console.error(‘getSystemInfoSync failed!’);
}

var wheatherData = this.convert(datapoints);

//新建湿度图表
this.lineChart_hum = new myCharts({
canvasId: ‘humidity’,
type: ‘line’,
categories: wheatherData.categories,
animation: false,
background: ‘#f5f5f5’,
series: [{
name: ‘humidity’,
data: wheatherData.humidity,
format: function (val, name) {
return val.toFixed(2);
}
}],
xAxis: {
disableGrid: true
},
yAxis: {
title: ‘humidity (%)’,
format: function (val) {
return val.toFixed(2);
}
},
width: windowWidth,
height: 200,
dataLabel: false,
dataPointShape: true,
extra: {
lineStyle: ‘curve’
}
});
// 新建光照强度图表
this.lineChart_light = new myCharts({
canvasId: ‘light’,
type: ‘line’,
categories: wheatherData.categories,
animation: false,
background: ‘#f5f5f5’,
series: [{
name: ‘light’,
data: wheatherData.light,
format: function (val, name) {
return val.toFixed(2);
}
}],
xAxis: {
disableGrid: true
},
yAxis: {
title: ‘light (lux)’,
format: function (val) {
return val.toFixed(2);
}
},
width: windowWidth,
height: 200,
dataLabel: false,
dataPointShape: true,
extra: {
lineStyle: ‘curve’
}
});

//新建温度图表
this.lineChart_tempe = new myCharts({
canvasId: ‘tempe’,
type: ‘line’,
categories: wheatherData.categories,
animation: false,
background: ‘#f5f5f5’,
series: [{
name: ‘temperature’,
data: wheatherData.tempe,
format: function (val, name) {
return val.toFixed(2);
}
}],
xAxis: {
disableGrid: true
},
yAxis: {
title: ‘temperature (摄氏度)’,
format: function (val) {
return val.toFixed(2);
}
},
width: windowWidth,
height: 200,
dataLabel: false,
dataPointShape: true,
extra: {
lineStyle: ‘curve’
}
});
},
})
邹学泽：小程序端播放音乐代码
Page({
onReady: function (e) {
// 使用 wx.createAudioContext 获取 audio 上下文 context
this.audioCtx = wx.createAudioContext(‘myAudio’)
},
data: {
poster: ‘http://y.gtimg.cn/music/photo_new/T002R300x300M000003rsKF44GyaSk.jpg?max_age=2592000’,
name: ‘此时此刻’,
author: ‘许巍’,
src: ‘http://ws.stream.qqmusic.qq.com/M500001VfvsJ21xFqb.mp3?guid=ffffffff82def4af4b12b3cd9337d5e7&uin=346897220&vkey=6292F51E1E384E06DCBDC9AB7C49FD713D632D313AC4858BACB8DDD29067D3C601481D36E62053BF8DFEAF74C0A5CCFADD6471160CAF3E6A&fromtag=46’,
},
audioPlay: function () {
this.audioCtx.play()
},
audioPause: function () {
this.audioCtx.pause()
},
audio60: function () {
this.audioCtx.seek(60)
},
audioStart: function () {
this.audioCtx.seek(0)
}
})​


：wxml代码
<viewclass=”a”>最爱的G.E.M邓紫棋</view>
<image src=”/images/dengziqi.jpg”></image>
<view class=”PlayPageFragement”>
<scroll-viewenable-back-to-to=”true”scroll-top=”true”scroll-y=”true”class=’listArea’>
<button wx:for-items=”{{music_list}}” wx:key=”musickey” bindtap=’chooseMusic’ data-id=”{{item.name}}” wx:if=’true’ class=”{{music_list[playId].name==item.name&&canPlay?’playing’:’list’}}”>
{{index+1}}:{{item.name}}
</button>
</scroll-view>
<!– <audio wx:if=”true” bindtimeupdate=”timeChange” style=”opacity:0″ src=”{}”></audio> –>
<view class=”play_area”>
<!– <slider activeColor=’green’ id=’paly_progress’ step=’1′ max=”{{music_list[playId].length}}” min=”0″ bindchange=’dragedProgress’ class=”slider” value=”{{playPrecent}}”></slider> –>
<slider value='{{vol}}’ max=”40″ min=’10’ step=’1′ activeColor=’green’ bindchange=’vol_change’/>
<progress percent=”{{playPrecent}}” class=”slider”/>
<view class=’buttons_class’ >
<image src=”{{previous}}” class=”previous_button” bindtap=’PreviousMusic’></image>
<image src=”{{play}}” class=”play_button” bindtap=’PlayOrPause’></image>
<image src=”{{next}}” class=”next_button” bindtap=’NextMusic’></image>

</view>
</view>
</view>
wxss代码
.playing{
background-color:#E0E0E0;
font-family:”Times New Roman”,Georgia,Sx;
font-style:italic;
color:blue;
size:default;
width:100%;
top:0%;
opacity:100%;
margin-top:3rpx;
font-weight:10000;
padding-top:3rpx;
border-left-width:5rpx;
border-right-width:5rpx;
text-align:left;
border-style:solid;
border-color:red#0000ff;
}
.list{
font-size:40rpx;
font-style:italic;
margin-top:30rpx;
padding-top:3rpx;
border-left-width:5rpx;
border-right-width:5rpx;
text-align:left;
size:default;
width:100%;
top:0%;
opacity:100%;
border-style:solid;
border-color:#ff0000#0000ff;
border-bottom-style:dotted;
border-top-style:groove;
background-image: ;
}
.listArea{
flex-direction:column;
display:flex;
top:0;
height:200%;
width:100%;
opacity:62%;

}
.play_area{
background:white;
/* position:fixed; */
bottom:50px;
/* align-items:flex-start; */
width:200%;
height:210rpx;
opacity:62%;
left:0rpx;
}
.previous_button{
width:110rpx;
height:110rpx;
left:5rpx;
color:green;
position:fixed;
opacity:62%;
bottom:10rpx;
}
.play_button{
width:120rpx;
height:120rpx;
left:40%;
position:fixed;
justify-content:center;
opacity:40%;
bottom:0rpx;
}
.next_button{
background-color:black;
width:130rpx;
height:130rpx;
position:fixed;
right:0%;
opacity:62%;
bottom:0rpx;

}
.slider{
width:100%;
height:20rpx;
position:fixed;
/* padding-bottom: 30%; */
/* opacity: 62%; */
background-color:white;
left:0rpx;
 
bottom:130rpx;
}
.buttons_class{
flex-direction:row;
position:fixed;
width:100%;
height:130rpx;
bottom:0rpx;
background-color:white;
opacity:62%;
}
.a{
font-display:fixed;
font-family:” Fantasy”,Georgia,Serif;
text-align:center;
font-size: 60rpx;
font-weight:900;
}

mcookie

#include “./audio.h”
#include <SoftwareSerial.h>
#include <Wire.h>
#include “U8glib.h”//使用oled
U8GLIB_SSD1306_128X64 u8g(U8G_I2C_OPT_NONE);
#include <ESP8266.h>//网页
#ifdef ESP32
#error “Error”
#endif
#if defined(__AVR_ATmega32U4__) || defined(__AVR_ATmega1284P__) || defined (__AVR_ATmega644P__) || defined(__AVR_ATmega128RFA1__)
#define EspSerial Serial1
#define UARTSPEED  115200//波特率
#endif
#if defined (__AVR_ATmega168__) || defined (__AVR_ATmega328__) || defined (__AVR_ATmega328P__)
#define EspSerial mySerial
#define UARTSPEED  9600
#endif
//Hotpots
#define SSID        F(“griffin”)
#define PASSWORD    F(“14567890”)//自己的热点和密码
#define HOST_NAME   F(“api.heclouds.com”)
#define HOST_PORT   (80)
static const byte  GETDATA[]  PROGMEM = {
 “GET https://api.heclouds.com/devices/22834365/datapoints?datastream_id=id,status,vol&limit=1 HTTP/1.1\r\napi-key: zXUxQaupAumEyFfR8rEX7l9c9Yg=\r\nHost: api.heclouds.com\r\nConnection: close\r\n\r\n”
};
ESP8266 wifi(&EspSerial);
int rollCount=0,rollRoomMembers=0;
bool volChange=false,statusChange=false,idChange=false;
int music_status=0,temp_music_status=0;
int music_vol=10,temp_music_vol=10;
int music_num_MAX=9;
int current_music=1,temp_current_music=1;
char* names[]={“BUPT Theme Song-AiYue “,”Glad You Come-Boyce Avenue  “,”Penumatic Tokyo-Env “,”Luv Letter-DJ “,”Only My Railgun-None “,”Sugar-Kidz Bop Kids “,”Unity-The Fat Rat “,”Star Sky-Two Steps From Hell “,”Schnappi-Joy Gruttmann “};
bool isConnected=false;
bool bottomBar=true;
bool canPlay=false;
void drawTitle(){
 u8g.setPrintPos(4,48);
 for(int i=0;i<strlen(names[current_music]);i++){
  if(rollCount+i<strlen(names[current_music])){
   u8g.print(*(names[current_music]+rollCount+i));
  }
  else
  {
   u8g.print(*(names[current_music]+rollCount+i-strlen(names[current_music])));
  }
 }
  u8g.print(“…”);
}
void drawNotConnected(){
 u8g.drawTriangle(64,4,32,48,96,48);
 u8g.drawTriangle(64,60,32,16,96,16);
}
bool networkHandle() {
 canPlay=true;
 uint8_t buffer[415]={0};
 uint32_t len = wifi.recv(buffer, sizeof(buffer), 2000);
 if (len > 0) {
  for (uint32_t i = 0; i < len; i++) {
   Serial.print((char)buffer[i]);
  }
 }
 temp_music_vol=((int)buffer[272]-48)*10+((int)buffer[273]-48)-10;
 temp_current_music=(int)buffer[344]-48;
 temp_music_status=(int)buffer[414]-48;
 Serial.println(temp_music_vol);
 Serial.println(temp_current_music);
 Serial.println(temp_music_status);  
        free(buffer);
 wifi.releaseTCP();
}
void audioHandle(){
 if(canPlay){
 if(current_music!=temp_current_music){
  idChange=true;
  current_music=temp_current_music;
 }
 if(music_vol!=temp_music_vol){
  volChange=true;
  music_vol=temp_music_vol;
 }
 if(music_status!=temp_music_status){
  statusChange=true;
  music_status=temp_music_status;
 }
 if(statusChange){
  if(music_status==1){
   audio_play();
  }else{
   audio_pause();
  }
 }
 if(volChange){
  audio_vol(music_vol);
 }
 if(idChange){
  audio_choose(current_music+1);
  audio_play();
 }
 volChange=false;
 idChange=false;
 statusChange=false;
}
}
void setup(void)
{
Serial.begin(115200);
  while (!Serial); // wait for Leonardo enumeration, others continue immediately
  Serial.println(F(“WIFI start”));
  delay(100);
  WifiInit(EspSerial, UARTSPEED);
  wifi.setOprToStationSoftAP();
  if (wifi.joinAP(SSID, PASSWORD)) {
   Serial.println(F(“WIFI Connect!”));
  isConnected=true;
  } else {
   isConnected=false;
   Serial.println(F(“NO WIFI”));
  }
  wifi.disableMUX();
  audio_init(DEVICE_TF,MODE_One_END,music_vol);  
}
void drawPlay(){
 u8g.drawBox(4,4,8,16);
 u8g.drawBox(20,4,8,16);
 u8g.setPrintPos(32,16);
 u8g.print(“Playing”);
}
void drawPause(){
 u8g.drawTriangle(4,4,30,8,4,16);
 u8g.setPrintPos(32,16);
 u8g.print(“Paused”);
}
void drawVol(){
 u8g.drawLine(4,35,4+30*4,35);
 u8g.drawLine(4+30*4,35,4+30*4,35-30*0.6);
 u8g.drawLine(4,35,4+30*4,35-30*0.6);
 u8g.drawTriangle(4,35,4+music_vol*4,35,4+music_vol*4,35-music_vol*0.6);
}
void drawBottom(){
  u8g.setPrintPos(4, 16 * 4);
  switch(rollRoomMembers){
   case 0:
   u8g.print(“****ROOM 324****”);
   break;
   case 1:
   u8g.print(“Zhao: 2017210672”);
   break;
   case 2:
   u8g.print(“Ran:   2017210676”);
   break;
   case 3:
   u8g.print(“Cheng:2017210679”);
   break;
 }
}
void drawAll(){
  u8g.setFont(u8g_font_7x13);
  if(music_status==0){
   drawPause();
  }else{
   drawPlay();
  }
  drawVol();
  drawTitle();
  drawBottom();
}
void loop(void)
{
 if(isConnected){
  if (wifi.createTCP(HOST_NAME, HOST_PORT)) {
   Serial.print(F(“TCP\n”));
   isConnected=true;
  } else {
   Serial.print(F(“No TCP\n”));
   isConnected=true;
  }
  wifi.sendFromFlash(GETDATA, sizeof(GETDATA)); 
  networkHandle();
  audioHandle();
  u8g.firstPage();
  do {
   drawAll();
  } while (u8g.nextPage());
 }
 else
 {
  u8g.firstPage();
  do{
   drawNotConnected();
  }while(u8g.nextPage());
  delay(5000);
  setup();
 }
 rollCount++;
 rollCount=rollCount%strlen(names[current_music]);
 if(bottomBar){
  bottomBar=false;
  rollRoomMembers=rollRoomMembers+1;
  rollRoomMembers=rollRoomMembers%4;
 }else{
  bottomBar=true;
 }
}



：js
Page({
data: {

music_list: [
{
name: “光年之外”,
length: 231
},
{
name: “你不是真正的快乐”,
length: 197
},
{
name: “喜欢你”,
length: 228
},
{
name: “画”,
length: 270
},
{
name: “泡沫”,
length: 257
},
{
name: “你把我灌醉”,
length: 235
},
{
name: “来自天堂的魔鬼”,
length: 249
},
{
name: “红蔷薇白玫瑰”,
length: 334
},
{
name: “后会无期”,
length: 132
}],
play: “https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1512218011639&di=47d229f8166e09fbcb360285db64db8a&imgtype=0&src=http%3A%2F%2Fpic.58pic.com%2F58pic%2F14%2F64%2F28%2F05T58PICd5S_1024.jpg”,
next: “https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1512210880414&di=ae93550e8f8c5e0169361061673f5e24&imgtype=0&src=http%3A%2F%2Fimgsrc.baidu.com%2Fimgad%2Fpic%2Fitem%2F0e2442a7d933c895ac5bcf50da1373f0820200f7.jpg”,
previous: “https://ss1.bdstatic.com/70cFuXSh_Q1YnxGkpoWK1HF6hhy/it/u=3626158946,5519961&fm=27&gp=0.jpg”,
playId: 0,
//0:pause,1:play
playButtonStatus: 0,
playPrecent: 0,
canPlay: false,
vol: 20
},

onLoad: function (options) {

},

onReady: function () {

},

onShow: function () {
 
this.autoPlus()
},
onHide: function () {

},

 
onUnload: function () {

},

 
onPullDownRefresh: function () {

},

 
onReachBottom: function () {

},

onShareAppMessage: function () {

},
 
autoPlus: function () {
if (this.data.playButtonStatus == 1 && this.data.canPlay) {
if (this.data.playPrecent < this.data.music_list[this.data.playId].length) {
var interval = 100 / this.data.music_list[this.data.playId].length;
this.setData({
playPrecent: this.data.playPrecent + interval
})
}
else {
this.setData({
playPrecent: 0
})
this.NextMusic()
}
}
setTimeout(this.autoPlus, 1000)
},
sendRequest: function (obj) {
wx.request(obj);
},
makeObj: function (i, sta, pre, msg) {
var obj = {
url: “http://api.heclouds.com/devices/22834365/datapoints?type=3”,
header: {
“api-key”: “zXUxQaupAumEyFfR8rEX7l9c9Yg=”,
“Content-Type”: “application/json”
},
method: “post”,
data: {
//msuci id,playing status,playing precent
“id”: i,
“status”: sta,
“vol”: this.data.vol
//”precent”:pre
},
success: function (res) {
if (msg != “”) {
wx.showToast({
title: msg,
duration: 1000
})
//console.log(i);
}
}
}
return obj;
},
 
chooseMusic: function (event) {
var temp = 0;
var musicName = event.currentTarget.dataset.id
for (var i = 0; i < this.data.music_list.length; i++) {
if (musicName == this.data.music_list[i].name) {
temp = i;
this.sendRequest(this.makeObj(i, 1, 0, “成功播放”))
break
}
}
this.setData({
playPrecent: 0,
playId: temp,
canPlay: true,
playingMusic: this.data.music_list[this.data.playId],
play: “https://ss0.bdstatic.com/94oJfD_bAAcT8t7mm9GUKT-xh_/timg?image&quality=100&size=b4000_4000&sec=1512287432&di=0d1c4453ce9a687ec14264e9e24e55bf&src=http://pic.58pic.com/58pic/14/80/56/85Y58PICdcV_1024.jpg”,
playButtonStatus: 1
})
var sta = this.data.playButtonStatus
this.sendRequest(this.makeObj(this.data.playId, sta, 0, “”))
// this.ProgressPlus()
},
PreviousMusic: function () {
var currentId = this.data.playId;
var finalId = 0;
if (currentId >= 1) {
finalId = currentId – 1
}
else {
finalId = this.data.music_list.length – 1
}
var musicName = this.data.music_list[this.data.playId].name
this.sendRequest(this.makeObj(this.data.playId, 1, 0, musicName + “播放成功”))
this.setData({
playId: finalId,
playPrecent: 0,
playingMusic: this.data.music_list[this.data.playId]
})
},
NextMusic: function () {
var currentId = this.data.playId;
var finalId = 0;
if (currentId < this.data.music_list.length – 1) {
finalId = currentId + 1;
}
else {
finalId = 0;
}
var musicName = this.data.music_list[this.data.playId].name
this.sendRequest(this.makeObj(this.data.playId, 1, 0, musicName + “播放成功”))
this.setData({
playId: finalId,
playPrecent: 0,
playingMusic: this.data.music_list[this.data.playId]
})
//this.ProgressPlus()
//console.log(this.data.playId);
},
PlayOrPause: function () {
var pre = this.data.playPrecent
if (this.data.playButtonStatus == 0) {
this.data.playButtonStatus = 1;
if (!this.data.canPlay) {
this.setData({
canPlay: true
})
}
this.setData({
play: “https://ss0.bdstatic.com/94oJfD_bAAcT8t7mm9GUKT-xh_/timg?image&quality=100&size=b4000_4000&sec=1512287432&di=0d1c4453ce9a687ec14264e9e24e55bf&src=http://pic.58pic.com/58pic/14/80/56/85Y58PICdcV_1024.jpg”
})
}
else {
this.data.playButtonStatus = 0;
this.setData({
play: “https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1512218011639&di=47d229f8166e09fbcb360285db64db8a&imgtype=0&src=http%3A%2F%2Fpic.58pic.com%2F58pic%2F14%2F64%2F28%2F05T58PICd5S_1024.jpg”
})

}
var sta = this.data.playButtonStatus
this.sendRequest(this.makeObj(this.data.playId, sta, pre, “”))

},
 
dragedProgress: function (e) {

},
vol_change: function (e) {
this.setData({
vol: e.detail.value
});
var sta = this.data.playButtonStatus;
var pre = this.data.precent;
this.sendRequest(this.makeObj(this.data.playId, sta, pre, “”));

}
})


手表代码
const unsigned char PROGMEM ri[] = {
0x00,0x00,0x00,0x3F,0xFF,0xC0,0x3F,0xFF,0xE0,0x38,0x00,0xE0,0x38,0x00,0xE0,0x3F,
0xFF,0xE0,0x3F,0xFF,0xE0,0x38,0x00,0xE0,0x38,0x00,0xE0,0x3F,0xFF,0xE0,0x3F,0xFF,
0xE0,0x00,0x70,0x00,0x00,0x70,0x00,0x7F,0xFF,0xF0,0xFF,0xFF,0xF8,0x00,0x70,0x00,
0x00,0x70,0x00,0x00,0x70,0x00,0x00,0x70,0x00,0x00,0x70,0x00
};
const unsigned char PROGMEM ri2[] = {
0x00,0x00,0x00,0x00,0x00,0x00,0x7E,0x1F,0xE0,0x7F,0xFF,0xC0,0xFE,0x1E,0x00,0xFF,
0xFF,0xF0,0xFF,0xFF,0xF0,0xFE,0xFF,0xC0,0xFE,0xFF,0xC0,0xFE,0xFF,0xC0,0xFF,0xFF,
0xF8,0xFF,0xFF,0xF0,0xFE,0xFF,0xC0,0xFE,0xFF,0xC0,0xFF,0xFF,0xF0,0xFF,0xFF,0xF0,
0xFE,0x1E,0x00,0x7E,0x1E,0x00,0x7F,0xFF,0xF8,0x03,0xFF,0xF0
};
const unsigned char PROGMEM ri3[] = {
0x00,0x00,0x00,0x3F,0xFF,0xC0,0x3F,0xFF,0xE0,0x38,0x00,0xE0,0x38,0x00,0xE0,0x3F,
0xFF,0xE0,0x3F,0xFF,0xE0,0x38,0x00,0xE0,0x38,0x00,0xE0,0x3F,0xFF,0xE0,0x3F,0xFF,
0xE0,0x00,0x70,0x00,0x00,0x70,0x00,0x7F,0xFF,0xF0,0xFF,0xFF,0xF8,0x00,0x70,0x00,
0x00,0x70,0x00,0x00,0x70,0x00,0x00,0x70,0x00,0x00,0x70,0x00
};
const unsigned char PROGMEM ri4[] = {
0x00,0x00,0x00,0x06,0x00,0x00,0x0F,0x00,0x00,0x3F,0xFF,0xE0,0x7F,0xFF,0xF0,0x0F,
0x00,0xF0,0x0F,0x00,0xF0,0x0F,0x00,0xF0,0xFF,0xFF,0xF0,0x0F,0x3F,0xE0,0x3F,0x38,
0x00,0x3F,0x38,0x00,0x3F,0xF8,0x70,0x3F,0xF8,0x70,0x3F,0x38,0x70,0x7F,0x3F,0xF0,
0x7F,0x1F,0xE0,0x7F,0x00,0x00,0x77,0xFF,0xF8,0xF1,0xFF,0xF8
};
const unsigned char PROGMEM ri5[] = {
0x00,0x00,0x00,0x00,0xF0,0x00,0x00,0xE0,0x00,0x1F,0xFF,0x80,0x3F,0xFF,0xC0,0x3C,
0x03,0xC0,0x3F,0xFF,0xC0,0x3F,0xFF,0xF8,0x3C,0x03,0xF8,0x3F,0xFF,0xF8,0x3F,0xFF,
0xF0,0x3C,0x03,0xE0,0x3C,0x03,0xE0,0x7F,0xFF,0xC0,0xFF,0xFF,0xC0,0x00,0x3F,0xC0,
0x00,0xFF,0xC0,0x07,0xE3,0xC0,0x7F,0x9F,0xC0,0x78,0x3F,0xC0
};
const unsigned char PROGMEM ri6[] = {
0x00,0x00,0x00,0x0E,0x1C,0x00,0x1E,0x3C,0x00,0x1E,0x3C,0x00,0x1C,0x3C,0x00,0x1F,
0xFF,0xF8,0x3C,0x3C,0x00,0x3C,0x3E,0x00,0x7C,0x7E,0x00,0x7C,0x7F,0x00,0xFC,0xFF,
0x80,0xFD,0xFF,0x80,0x3D,0xFF,0xC0,0x3F,0xFD,0xE0,0x3F,0xFF,0xF0,0x3F,0xFF,0xF8,
0x3C,0x3C,0x10,0x3C,0x3C,0x00,0x3C,0x3C,0x00,0x3C,0x3C,0x00
};
const unsigned char PROGMEM ri7[] = {
0x00,0x00,0x00,0x1C,0x00,0x00,0x1C,0xFF,0xE0,0x1C,0xFF,0xE0,0xFF,0xC1,0xE0,0x7F,
0xC3,0xC0,0x3D,0xC7,0x80,0x3B,0xCF,0x00,0x3B,0xCF,0x00,0x7B,0x87,0x00,0x7B,0xFF,
0xF8,0x77,0xFF,0xF8,0x7F,0x83,0x80,0x3F,0x03,0xC0,0x1F,0x03,0xC0,0x1F,0x03,0xC0,
0x1F,0x83,0xC0,0x7F,0xE3,0x80,0xF8,0xFF,0x80,0xE0,0x3E,0x00
};
#include <Microduino_RTC.h>
 
RTC rtc;
#include <U8glib.h>
#define INTERVAL_LCD             20             //定义OLED刷新时间间隔  
int RECV_PIN = 10;                               //D10接红外接收传感器
unsigned long lcd_time = millis();                 //OLED刷新时间计时器
U8GLIB_SSD1306_128X64 u8g(U8G_I2C_OPT_NONE);     //设置OLED型号  
//——-字体设置，大、中、小
#define setFont_L u8g.setFont(u8g_font_7x13)
#define setFont_M u8g.setFont(u8g_font_fixed_v0r)
#define setFont_S u8g.setFont(u8g_font_fixed_v0r)
#define setFont_SS u8g.setFont(u8g_font_fub25n)
/* 设置RTC启动时间
 * 年, 月, 星期, 日, 时, 分, 秒 */
DateTime dateTime = {2019, 12, 5, 20, 22, 50, 00};
uint16_t tYear;
uint8_t tMonth, tWeekday, tDay, tHour, tMinute, tSecond;
void setup()
{
  Serial.begin(9600);
  //清除所有寄存器
  rtc.begin();
  rtc.clearAll();
  //设置启动时间
  rtc.setDateTime(dateTime);
}
void loop()
{
  //通过getDataTime获取时间
  rtc.getDateTime(&dateTime);
   u8g.firstPage();//u8glib规定写法
    do {
        setFont_L;
        u8g.drawBitmapP(0, 0, 3, 20, ri);
        u8g.drawBitmapP(24, 0, 3, 20, ri2);
        u8g.drawBitmapP(48, 0, 3, 20, ri3);
        u8g.drawBitmapP(72, 0, 3, 20, ri4);
        u8g.setPrintPos(10, 36);//选择位置
        u8g.print(dateTime.hour);//写入字符串
       u8g.setPrintPos(24, 36);
       u8g.print(“:”);
        u8g.setPrintPos(36, 36);
        u8g.print(dateTime.minute);
         u8g.setPrintPos(56, 36);
          u8g.print(“:”);
          u8g.setPrintPos(76, 36);
        u8g.print(dateTime.second);
         u8g.drawBitmapP(0, 40, 3, 20, ri5);
          u8g.drawBitmapP(24, 40, 3, 20, ri6);
           u8g.drawBitmapP(48, 40, 3, 20, ri7);
    }while( u8g.nextPage() );
 
  Serial.print(dateTime.hour);
  Serial.print(“:”);
  Serial.print(dateTime.minute);
  Serial.print(“:”);
  Serial.print(dateTime.second);
  Serial.print(“\r\n”);
 
  delay(1000);
  Serial.print(“\r\n”);
}

--- 
layout: post 
title: "Html5 实现摇一摇功能" 
date: 2014-12-11 
categories: 前端
tags: html5 javascript
--- 

### 原理

* 在HTML5中，DeviceOrientation特性所提供的DeviceMotion事件封装了设备的运动传感器时间，通过改时间可以获取设备的运动状态、加速度等数据（另还有deviceOrientation事件提供了设备角度、朝向等信息）。
* 而通过DeviceMotion对设备运动状态的判断，则可以帮助我们在网页上就实现“摇一摇”的交互效果。


### 实现

* 把监听事件绑定给 deviceMotionHandler

<pre>
if (window.DeviceMotionEvent) {
	window.addEventListener('devicemotion', deviceMotionHandler, false);
} else {
	alert('本设备不支持devicemotion事件');
}
</pre>

* 获取设备加速度信息 accelerationIncludingGravity

<pre>
function deviceMotionHandler(eventData) {
	var acceleration = eventData.accelerationIncludingGravity,
	x, y, z;
	x = acceleration.x;
	y = acceleration.y;
	z = acceleration.z;
	//将加速度信息打印置页面，通过演示地址可以看到随着设备的移动，屏幕上数字的变化。
	document.getElementById("status").innerHTML = "x:"+x+"y:"+y+"z:"+z;
}
</pre>

* “摇一摇”的动作既“一定时间内设备了一定距离”，因此通过监听上一步获取到的x, y, z,值在一定时间范围内的变化率，即可进行设备是否有进行晃动的判断。而为了防止正常移动的误判，需要给该变化率设置一个合适的临界值。

<pre>
var SHAKE_THRESHOLD = 9999;
var last_update = 0;
var x, y, z, last_x, last_y, last_z;

if (window.DeviceMotionEvent) {

    window.addEventListener('devicemotion', function (eventData) {

        var acceleration = eventData.accelerationIncludingGravity;

        var curTime = new Date().getTime();

        if ((curTime - last_update) > 100) {

            var diffTime = curTime - last_update;

            last_update = curTime;

            x = acceleration.x;
            y = acceleration.y;
            z = acceleration.z;

            var speed = Math.abs(x + y + z - last_x - last_y - last_z) / diffTime * 10000;

            if (speed > SHAKE_THRESHOLD) {
                alert("shaked!");
            }

            last_x = x;
            last_y = y;
            last_z = z;
        }

    }, false);
}
</pre>	
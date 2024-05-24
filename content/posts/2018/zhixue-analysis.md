+++
title = "智学网HTTP协议分析记录"
date = 2018-10-17T01:29:43+08:00
tags = ["experiment"]
+++

> 这篇文章很差，可读性很烂，但我保留它，因为这是我分析研究的第一篇文章
>
> 我不会去修改下面的任何内容，这些内容时刻提醒自己，我还有很大的提升空间

2018.11.23，智学网APP，Android，Charles抓包

# 登录

## 接口API

```
POST
http://www.zhixue.com/container/app/weakCheckLogin?

password=08c185208d0e57efc854&loginName=38268068&description=%7B%27encrypt%27%3A%5B%27password%27%5D%7D&deviceMac=50%3A8F%3A4C%3AFA%3AAA%3A3B&deviceId=866146033672761
```

POST数据解析

| password    | 08c185208d0e57efc854     |
| ----------- | ------------------------ |
| loginName   | 38268068                 |
| description | {'encrypt':['password']} |
| deviceMac   | 50:8F:4C:FA:AA:3B        |
| deviceId    | 866146033672761          |

**password**:加密后的密码，算法不知道，估计难拆，不过用作查询不需要拆

**description**:给服务端表明这份密码是加密过的

**deviceMac**:设备的mac地址，用作查询不需要改

**deviceld**:这个不知道..为什么不是id而是ld..

## 返回json

```json
{
	"errorCode": 0,
	"errorInfo": "操作成功",
	"result": {
		"accountType": "userCode",
		"clazzInfo": {
			"code": "5423495a-22f6-4be8-97de-3d19206f4be7",
			"gradeCode": "11",
			"graduated": false,
			"id": "4444000020000342148",
			"name": "高二年级4班",
			"order": 4,
			"phase": "05",
			"phaseName": "高中",
			"schoolId": "2244000020000000338",
			"year": 2017
		},
		"createdBy": "zx",
		"currentloginType": "zx",
		"customInfo": {
			"simpleAndNormalVip": false,
			"simpleVip": false,
			"viewSummerHomework": false
		},
		"hasOrg": true,
		"id": "4444000020022642541",
		"isAdmin": false,
		"isInitialPwd": false,
		"name": "林黄骁",
		"preLoginDate": 1542987371000,
		"role": "student",
		"roles": ["student"],
		"serverInfo": {
			"curServerTime": 1542987504466
		},
		"token": "63db0c7d-b13f-4300-a643-16bad50cd221",
		"userInfo": {
			"birthday": 1503244800000,
			"code": "38268068",
			"email": "",
			"gender": "1",
			"id": "4444000020022642541",
			"im": "",
			"loginName": "zx38268068",
			"mobile": "13950383863",
			"name": "林黄骁",
			"school": {
				"areaId": "1267",
				"cityId": "1263",
				"countryId": "1",
				"districtId": "1267",
				"phase": "05",
				"provinceId": "1262",
				"schoolCode": "1003",
				"schoolId": "2244000020000000338",
				"schoolName": "福建省福州八中"
			}
		},
		"vipInfo": {
			"hidden": false,
			"vipBeginTime": 1509206400000,
			"vipEndTime": 1540828799000,
			"vipLevel": 0
		},
		"virtualTagInfo": {
			"areaName": "",
			"cityId": "",
			"cityName": "",
			"districtId": "",
			"districtName": "",
			"needAreaPop": false,
			"needGradePop": false,
			"nickName": "",
			"provinceId": "",
			"provinceName": "",
			"roleTag": ""
		}
	}
}
```

此处的token=63db0c7d-b13f-4300-a643-16bad50cd221会发现基本所有的接口都需要这个token作为一个验证参数

# 查询

## 接口API

```


GET

http://app.zhixue.com/study/social/getGuessScore?&examId=f899e9d4-db72-4fad-96ea-307b15d4896b&guessUserId=4444000020022642815&token=f867f8c8-fb39-4de6-8188-e65ccd1565bc&childrenId=4444000020022642541

http://app.zhixue.com/study/social/getGuessScore?&examId=911973c8-750d-4bd2-9037-32eb60904012&guessUserId=4444000020022642389&token=e5295e82-6684-44e5-a2f9-7f25c64ae0c1&childrenId=4444000020022642541

http://app.zhixue.com/study/social/getGuessScore?&examId={exam_id}&guessUserId={userid}&token={token}&childrenId={userid_myself}

```

## 参数分析

**examId**：考试ID，已经可以抓到

**guessUserId**：伴学的ID，不需要添加为好友

**token**：智学网APP账户钥匙，与登录的保持一致

**childrenId**：自己的智学网ID

## 缺陷

需要VIP账号，不然有次数限制

## 返回json

```json
{
	"errorCode": 0,
	"errorInfo": "操作成功",
	"result": {
		"customForbidDTO": {
			"user_Ratio_To_Level": false,
			"user_Score_To_Level": false,
			"forbid_Class_Rank": false,
			"forbid_Grade_Rank": false,
			"forbid_Union_Rank": false,
			"forbid_Class_AvgScore": true,
			"forbid_Grade_AvgScore": true,
			"forbid_Union_AvgScore": false,
			"forbid_Class_HighScore": true,
			"forbid_Grade_HighScore": true,
			"forbid_Union_HighScore": false,
			"forbid_Origin_Paper": false,
			"forbid_Score_PK": false,
			"show_Intelligent_Correct": false,
			"show_UserLevel": true,
			"show_Score_Distribution": true,
			"show_WrongTopic_Training": true
		},
		"studentPKDTOs": [{
			"userId": "4444000020022642541",
			"avatar": null,
			"name": "林黄骁",
			"subjectList": [{
				"subjectCode": "",
				"subjectName": "总分",
				"score": 215.0,
				"classPercent": 0,
				"gradePercent": 0
			}, {
				"subjectCode": "14",
				"subjectName": "地理",
				"score": 76.5,
				"classPercent": 0,
				"gradePercent": 0
			}, {
				"subjectCode": "12",
				"subjectName": "历史",
				"score": 69.5,
				"classPercent": 0,
				"gradePercent": 0
			}, {
				"subjectCode": "27",
				"subjectName": "政治",
				"score": 69.0,
				"classPercent": 0,
				"gradePercent": 0
			}],
			"isFriend": false,
			"isSelf": true,
			"isVip": false
		}, {
			"userId": "4444000020022642535",
			"avatar": null,
			"name": "杨宸杰",
			"subjectList": [{
				"subjectCode": "",
				"subjectName": "总分",
				"score": 171.5,
				"classPercent": 0,
				"gradePercent": 0
			}, {
				"subjectCode": "14",
				"subjectName": "地理",
				"score": 64.0,
				"classPercent": 0,
				"gradePercent": 0
			}, {
				"subjectCode": "12",
				"subjectName": "历史",
				"score": 63.5,
				"classPercent": 0,
				"gradePercent": 0
			}, {
				"subjectCode": "27",
				"subjectName": "政治",
				"score": 44.0,
				"classPercent": 0,
				"gradePercent": 0
			}],
			"isFriend": true,
			"isSelf": false,
			"isVip": true
		}]
	}
}
```

# 2018.12.23

## 前言

大概是一周前，应该是我生日前一天，智学网对上文提到的接口api添加了协议头验证，扒出来是加了这个验证也花了挺多时间，然后顺便也解决了我三个月前的一个web端疑问，web端有几个api总是提示参数缺失，而提交的数据和cookie都没有什么问题。

## 协议头HTTP HEADER

首先扒出来了他的协议头：

```html
POST /container/app/weakCheckLogin? HTTP/1.1
deviceId: 866146033672761
browserVersion: Android_1.8.1595
channelId: Android_11010005
deviceMac: 50:8F:4C:FA:AA:3B
resolution: 1920*1080
authbizcode: 0001
authguid: 4ff59b11-eb56-4614-b6f5-2bfa889d4d40
authtimestamp: 1545492288739
authtoken: 8e59161b459405c5b7c48006c0d8b426
Content-Length: 168
Content-Type: application/x-www-form-urlencoded
Host: www.zhixue.com
Cookie: tlsysSessionId=bc887c7f-6626-46a7-906a-9dab53f17211; __DAYU_PP=fE7YI3jmfneyaq6NMmma43a9dc4cbb04; JSESSIONID=0EF8C11939B7BD9E5C459E2DD17F4B59
Cookie2: $Version=1
Accept-Encoding: gzip
Connection: keep-alive
```

经过一波分析和实验，得出智学网其实只是验证了这三个参数

```
authguid: 4ff59b11-eb56-4614-b6f5-2bfa889d4d40
authtimestamp: 1545492288739 //13位时间戳，精确到毫秒
authtoken: 8e59161b459405c5b7c48006c0d8b426
```

很头大，中间花了一个多小时找这三个参数来源，最后发现，这个应该是实时生成的，每一次请求这三个都不一样

但是手机端又不会给出加密算法，只能靠电脑端了

还好这个加密算法被封进了app.js

## app.js

从google dev-tools里面扒拉出来的，不知道为什么charles没有扒拉出这个..很玄吧

### AuthGUID

关于对guid生成的算法：

```javascript
for (var t = [], e = 0; e < 36; e++) //一共取36位
t[e] = "0123456789abcdef".substr(Math.floor(16 * Math.random()), 1); 
//随机0-1，相乘，取不大于这个数的最大整数

return t[14] = "4", //14位强行替换为4，参照结果来看这是一个事实
t[19] = "0123456789abcdef".substr(3 & t[19] | 8, 1), //第19位进行与或
//从代码上看我怎么感觉只能取出8,9,10,11位的数..一定是错觉
t[8] = t[13] = t[18] = t[23] = "-", //这里进行替换，强行改成-
t.join("")
```

```
结果：
012345678901234567890123456789012345 //位数对照
f0e39247-054b-4042-9382-38511c3b2060
4ff59b11-eb56-4614-b6f5-2bfa889d4d40
```

这里已经算出来guid了



# AuthToken

这个算法很奇怪，这是里面的一部分，或者就是全部的代码吧？

```javascript
webpackJsonp([1], {
    "+BTi": function(t, e) {},
    "+ECs": function(t, e) {},
    "0RHc": function(t, e) {},
    "150l": function(t, e) {},
    "1zUs": function(t, e) {},
    "2aH0": function(t, e) {
            t.exports = "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAB0AAAAgCAMAAADZqYNOAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAAyRpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDUuNS1jMDE0IDc5LjE1MTQ4MSwgMjAxMy8wMy8xMy0xMjowOToxNSAgICAgICAgIj4gPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4gPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIgeG1sbnM6eG1wTU09Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9tbS8iIHhtbG5zOnN0UmVmPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvc1R5cGUvUmVzb3VyY2VSZWYjIiB4bWxuczp4bXA9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC8iIHhtcE1NOkRvY3VtZW50SUQ9InhtcC5kaWQ6OTMwMjYyMUZGM0FFMTFFODg3OTFGNzlFNjAwQ0ZENjkiIHhtcE1NOkluc3RhbmNlSUQ9InhtcC5paWQ6OTMwMjYyMUVGM0FFMTFFODg3OTFGNzlFNjAwQ0ZENjkiIHhtcDpDcmVhdG9yVG9vbD0iQWRvYmUgUGhvdG9zaG9wIENTNiAoTWFjaW50b3NoKSI+IDx4bXBNTTpEZXJpdmVkRnJvbSBzdFJlZjppbnN0YW5jZUlEPSJ4bXAuaWlkOjIyN0FGNEQ3NTI1ODExRTZBMzg2RjlDRjg2RkI0NUY1IiBzdFJlZjpkb2N1bWVudElEPSJ4bXAuZGlkOjIyN0FGNEQ4NTI1ODExRTZBMzg2RjlDRjg2RkI0NUY1Ii8+IDwvcmRmOkRlc2NyaXB0aW9uPiA8L3JkZjpSREY+IDwveDp4bXBtZXRhPiA8P3hwYWNrZXQgZW5kPSJyIj8+NRPJrwAAAJ9QTFRFrOrkiuLZm+bf4vj1et7Up+niHse2B8GvkeTbXdfLieLZFMWz0/Txte3nMcy82PXyuu7ptu3n1vXy6vr4n+fg9f385vn3t+3o8Pv63fb0tOznHce20PPwSNLE8fz7yPHtyfLusezmX9fLH8i3oejhvu/qwO/rCMKvXtfL7/v6guDWXNfK8/z73Pb0revk5/n3qerj7fv5BsGuBcGu////VYK8ngAAADV0Uk5T/////////////////////////////////////////////////////////////////////wB8tdAKAAAAtklEQVR42qzT2Q7CIBAF0IFSqGutW9W27vuuw/9/mzFVHqAlJXofOcllQgaQtkAVXQrUI/ZKd2hGKEU0Wt9HH80ymxYk15gbN/JYKS8YiCstGaiqdgmZTkhfBmQ2JqSnqY/o1bAtG1hvITY19QAGc+jIBYRDgJGmF0rPG3qVa5p6lD6cmu1T2e+1N6eMHe4skEe2DRm7Ob3Vf9X/7lWZPk+25nyff9GivUqURomBq8jll5XkJcAA89Ooa6nFvYgAAAAASUVORK5CYII="
    },
    "3JRO": function(t, e) {
            t.exports = "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAB0AAAAgCAMAAADZqYNOAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAAyRpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDUuNS1jMDE0IDc5LjE1MTQ4MSwgMjAxMy8wMy8xMy0xMjowOToxNSAgICAgICAgIj4gPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4gPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIgeG1sbnM6eG1wTU09Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9tbS8iIHhtbG5zOnN0UmVmPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvc1R5cGUvUmVzb3VyY2VSZWYjIiB4bWxuczp4bXA9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC8iIHhtcE1NOkRvY3VtZW50SUQ9InhtcC5kaWQ6OTMwMjYyMjNGM0FFMTFFODg3OTFGNzlFNjAwQ0ZENjkiIHhtcE1NOkluc3RhbmNlSUQ9InhtcC5paWQ6OTMwMjYyMjJGM0FFMTFFODg3OTFGNzlFNjAwQ0ZENjkiIHhtcDpDcmVhdG9yVG9vbD0iQWRvYmUgUGhvdG9zaG9wIENTNiAoTWFjaW50b3NoKSI+IDx4bXBNTTpEZXJpdmVkRnJvbSBzdFJlZjppbnN0YW5jZUlEPSJ4bXAuaWlkOjIyN0FGNEQ3NTI1ODExRTZBMzg2RjlDRjg2RkI0NUY1IiBzdFJlZjpkb2N1bWVudElEPSJ4bXAuZGlkOjIyN0FGNEQ4NTI1ODExRTZBMzg2RjlDRjg2RkI0NUY1Ii8+IDwvcmRmOkRlc2NyaXB0aW9uPiA8L3JkZjpSREY+IDwveDp4bXBtZXRhPiA8P3hwYWNrZXQgZW5kPSJyIj8+5kDDrQAAAYBQTFRFVdXIQdDBPc/AK8q6G8a1FsWz8Pv6+/7+BsGuM8y90/Tx+P39yfLuqOnj7Pr5RNHCCMKvw/DsPs/ALcu7UNPGD8OxiuLZGMa0EMSxte3net7UT9PGF8W0EcSys+zm+v79DMOw6vr4uO3of9/VbdvQOs6/wO/rW9bKeN3T+f79Lsu70fTwDcOxtO3n9v38CcKv/f/+rOrkWNbJv+/qwfDrHse2ZdnN8/z7LMu7/P7+8vz77vv67/v6dd3Sq+rk5Pj29/38b9vQbtvQJ8q51/Xy4Pf1YNjMH8i3l+Xddt3T2fbzhuHYN82+c9zSp+niC8KwP8/BXNfKi+La5fj2zfPvxvHt9Pz8gN/W5/n37fv5nufgTNPFIsi4vO7p8fz7m+bfRtHDIci3EsSyMcy8yPHtsezmSdLEfd/VguDWcNzRB8GvJMm42vbzRdHDOc6/oOfg6fr4zPLu3Pb0Msy9e97UTtPGft/VieLZWdbJ3fb01vXyO86/luXdiOLZBcGu////avyEswAAAIB0Uk5T/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////wA4BUtnAAABlklEQVR42qyTV1cCMRCFUWCX6tJ7l6KAKB0Eexd77x1773Xy10123UM8lhedl71zv5Pck81Egn4ryZ9oymqy6NW0p9ZbnNaIQIGUz+DwNJB+2eMw+IijEKk2qDwmvfHOaVRgoVP2agFEij9MoJGzkSVmrjHAYIOi/J7IqwHQeHnZQFP5zXhLu7igXD2KrtFUirdsNXQRGQvpcBOiKRodlrhARaQdZI+bT59yu7FomysTORsjbjdNZVxlnhVzm/WHdX00HcJRO6YlIp9XycGiNGX3KmdxMde8b7kq0rQNi8Q6f8iBsPDLKXoiee2vWWymqUBTF47KDvp5ayIixx2RzL1A8/YOpULIJVfmmtaiZLJ5q168BYT8nZe8LDRlWN7Iua9rtJYr1ANcoB9pVZFCX2mLilca83bpG2pq7cgj1JXVhdE3tDQG2U5mF26RSDUslcu8AFjh3PtB3QA2bjGw8UHVzBSA+1Sc53TOCPwg9uBBPJA6UHsdrNBvwe8ZMUyCUPUILcx8eSkJ/ZvTFw+mi//1yt4FGAAD3X9yTjfCbwAAAABJRU5ErkJggg=="
    },
    "6atI": function(t, e) {},
    "7XQ6": function(t, e) {},
    "8XQW": function(t, e) {},
    "9RkD": function(t, e, s) {
            t.exports = s.p + "static/img/composition-banner.24153ed.png"
    },
    CCWw: function(t, e) {},
    EyqX: function(t, e) {},
    FGfq: function(t, e) {},
    HQok: function(t, e) {},
    I4nB: function(t, e) {},
    "It+H": function(t, e) {},
    JGNI: function(t, e) {},
    KTGR: function(t, e) {
            t.exports = "data:image/png;base64,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"
    },
    "KqW/": function(t, e) {
            t.exports = "data:image/png;base64,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"
    },
    Lrnc: function(t, e) {},
    NHnr: function(t, e, s) {
            "use strict";
            Object.defineProperty(e, "__esModule", {
                    value: !0
            });
            var a = s("7+uW"),
            i = {
                    render: function() {
                            var t = this.$createElement,
                            e = this._self._c || t;
                            return e("div", {
                                    staticClass: "app",
                                    attrs: {
                                            id: "app"
                                    }
                            },
                            [e("router-view")], 1)
                    },
                    staticRenderFns: []
            };
            var n = s("VU/8")({
                    name: "App",
                    data: function() {
                            return {
                                    aaa: 111
                            }
                    },
                    created: function() {},
                    methods: {}
            },
            i, !1,
            function(t) {
                    s("rueH")
            },
            null, null).exports,
            r = s("/ocq"),
            o = (s("Yq4J"), s("+BTi"), s("qubY")),
            l = s.n(o),
            c = new a.default,
            h = (s("I4nB"), s("STLj")),
            u = s.n(h),
            d = (s("cDSy"), s("e0Bm")),
            f = s.n(d),
            p = s("//Fk"),
            v = s.n(p),
            m = s("mtWM"),
            g = s.n(m),
            b = s("Zrlr"),
            w = s.n(b),
            S = s("wxAW"),
            C = s.n(S),
            x = s("NC6I"),
            y = s.n(x),
            A = function() {
                    function t(e, s, a) {
                            w()(this, t),
                            this.bicode = e,
                            this.password = s,
                            this.timeDifference = a
                    }
                    return C()(t, [{
                            key: "guid",
                            value: function() {
                                    for (var t = [], e = 0; e < 36; e++) t[e] = "0123456789abcdef".substr(Math.floor(16 * Math.random()), 1);
                                    return t[14] = "4",
                                    t[19] = "0123456789abcdef".substr(3 & t[19] | 8, 1),
                                    t[8] = t[13] = t[18] = t[23] = "-",
                                    t.join("")
                            }
                    },
                    {
                            key: "getDateString",
                            value: function() {
                                    return (new Date).getTime()
                            }
                    },
                    {
                            key: "make",
                            value: function() {
                                    var t = this.guid(),
                                    e = this.getDateString() - this.timeDifference,
                                    s = t + e + this.password,
                                    a = y()(s);
                                    return {
                                            authguid: t,
                                            authtimestamp: e,
                                            authbizcode: this.bicode,
                                            authtokenOrigin: s,
                                            authtoken: a
                                    }
                            }
                    }]),
                    t
            } (),
```


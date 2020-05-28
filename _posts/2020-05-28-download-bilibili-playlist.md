---
layout: post
title:  如何下载bilibili的视频列表
date: 2020-05-20 23:25:00.000000000 +08:00
---

我们使用 `youtube-dl` 下载油管视频的视频的时候，也可以下载一个视频列表中的视频，这种视频列表在bilibili中也是存在的，`youtube-dl` 还不能支持这种视频列表，具体原因不明。

未能能够批量的下载视频，我们还是能够从一些规律中找到办法。

下面的办法不是最完美的，因为并没有能够获取一个视频列表中不同p中的不同标题，这是因为B站的不同P公用同一个页面标题。这个应该说是B站的一个bug。

我先介绍下载分P的思路：

1. 不同的页面的区别，只是页面的p的值不同。比如第一个视频p=1，第二个是p=2.所以顺序下载视频不难。
2. 由于我们希望能够保存各个视频的顺序，所以需要加上p的数字。
3. 希望能够可以控制开始和结束的数字。

下面的小shell脚本就用来做这个事情

推荐线创建一个子目录，专门存放这些下载下来的文件。

Shell脚本如下：

使用方法： bash a.sh p_start p_end page_base_url 

如果有自己的服务器，不想干等，可以加上nohup 

eg：nohup bash a.sh 1 131 https://www.bilibili.com/video/BV1CJ411m7gg?p= >/dev/null 2>&1 &

```

for ((i=$1; i<=$2; i++))
do
    echo "start download"$3$i
    youtube-dl -o './tmp/%(title)s.LessonNumbers.%(ext)s' $3$i
    oldname=$(ls ./tmp)
    newName=$(ls ./tmp | sed "s/LessonNumbers/Lesson$i/g")
    mv "./tmp/$oldname" "./$newName"
done

```

由于youtube-dl可以获取到的的bilibili的字段可以用以下方法找到：

youtube-dl --print-json https://www.bilibili.com/video/BV1CJ411m7gg?p=1

然后修改`'./tmp/%(title)s.LessonNumbers.%(ext)s'`中的%(字段)s.

比如我们想要在文件名中加上`uploader_id`，那就改为：`'./tmp/%(title)s.LessonNumbers%(uploader_id)s.%(ext)s'`

```
{
  "id": "1CJ411m7gg",
  "title": "2019求知讲堂零基础Java入门编程视频教程 高口碑 无废话 无尿点",
  "formats": [
    {
      "format_id": "0",
      "format": "0 - unknown",
      "ext": "flv",
      "filesize": 24699446,
      "protocol": "http",
      "url": "http://upos-hz-mirrorakam.akamaized.net/upgcxcode/52/14/130391452/130391452-1-64.flv?e=ig8euxZM2rNcNbRM7zUVhoM17wuBhwdEto8g5X10ugNcXBlqNxHxNEVE5XREto8KqJZHUa6m5J0SqE85tZvEuENvNC8xNEVE9EKE9IMvXBvE2ENvNCImNEVEK9GVqJIwqa80WXIekXRE9IMvXBvEuENvNCImNEVEua6m2jIxux0CkF6s2JZv5x0DQJZY2F8SkXKE9IB5QK==&deadline=1590608526&gen=playurl&nbs=1&oi=1753236870&os=akam&platform=pc&trid=98f44a0fc90f487c9e8971b15df91183&uipk=5&upsig=1c997f8817d90c45fc4bb7cf615ef3cf&uparams=e,deadline,gen,nbs,oi,os,platform,trid,uipk&hdnts=exp=1590608526~hmac=3ab8f1f604dba72e6d21ef1fc9ba298ed538cf5f8a422f630f1097cc719e1799&mid=0",
      "http_headers": {
        "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
        "Accept-Encoding": "gzip, deflate",
        "Accept-Language": "en-us,en;q=0.5",
        "Referer": "https://www.bilibili.com/video/BV1CJ411m7gg?p=1",
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3599.1 Safari/537.36",
        "Accept-Charset": "ISO-8859-1,utf-8;q=0.7,*;q=0.7"
      }
    }
  ],
  "extractor": "BiliBili",
  "webpage_url_basename": "BV1CJ411m7gg",
  "format_id": "0",
  "duration": 414.032,
  "format": "0 - unknown",
  "ext": "flv",
  "thumbnail": "http://i1.hdslb.com/bfs/archive/9a24d91389ca595f3eed5d870bcc29f62b4b414e.jpg",
  "webpage_url": "https://www.bilibili.com/video/BV1CJ411m7gg?p=1",
  "thumbnails": [
    {
      "url": "http://i1.hdslb.com/bfs/archive/9a24d91389ca595f3eed5d870bcc29f62b4b414e.jpg",
      "id": "0"
    }
  ],
  "description": "我们这套Java视频教程全网唯一高口碑 全程 无废话 无尿点的课程 讲解同样的内容只需别套课程一半时间 大大减少学员们时间 适合绝对零基础的学员观看，该Java视频教程中讲解了Java开发环境搭建、Java的基础语法、Java的面向对象。每一个知识点都讲解的非常细腻，由浅入深。适合非计算机专业，想转行做Java开发的朋友，或者您想让Java基础更扎实观看",
  "timestamp": 1574163947,
  "filesize": 24699446,
  "_filename": "2019求知讲堂零基础Java入门编程视频教程 高口碑 无废话 无尿点-1CJ411m7gg.flv",
  "url": "http://upos-hz-mirrorakam.akamaized.net/upgcxcode/52/14/130391452/130391452-1-64.flv?e=ig8euxZM2rNcNbRM7zUVhoM17wuBhwdEto8g5X10ugNcXBlqNxHxNEVE5XREto8KqJZHUa6m5J0SqE85tZvEuENvNC8xNEVE9EKE9IMvXBvE2ENvNCImNEVEK9GVqJIwqa80WXIekXRE9IMvXBvEuENvNCImNEVEua6m2jIxux0CkF6s2JZv5x0DQJZY2F8SkXKE9IB5QK==&deadline=1590608526&gen=playurl&nbs=1&oi=1753236870&os=akam&platform=pc&trid=98f44a0fc90f487c9e8971b15df91183&uipk=5&upsig=1c997f8817d90c45fc4bb7cf615ef3cf&uparams=e,deadline,gen,nbs,oi,os,platform,trid,uipk&hdnts=exp=1590608526~hmac=3ab8f1f604dba72e6d21ef1fc9ba298ed538cf5f8a422f630f1097cc719e1799&mid=0",
  "extractor_key": "BiliBili",
  "requested_subtitles": null,
  "display_id": "1CJ411m7gg",
  "uploader": "求知讲堂小龙",
  "playlist_index": null,
  "playlist": null,
  "fulltitle": "2019求知讲堂零基础Java入门编程视频教程 高口碑 无废话 无尿点",
  "uploader_id": "448947071",
  "protocol": "http",
  "upload_date": "20191119",
  "http_headers": {
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
    "Accept-Encoding": "gzip, deflate",
    "Accept-Language": "en-us,en;q=0.5",
    "Referer": "https://www.bilibili.com/video/BV1CJ411m7gg?p=1",
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3599.1 Safari/537.36",
    "Accept-Charset": "ISO-8859-1,utf-8;q=0.7,*;q=0.7"
  }
}

```



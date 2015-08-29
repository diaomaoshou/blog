title: 坑爹的BOM头
date: 2015-08-29 22:34:46
tags: [BOM头,PHP小知识]
categories: 涨姿势
---

当你在一张网页完成时，本应该是激动万分，见证成果的时候。打开网页一看，咦~网页头部怎么会有诡异白条。难道是中邪了？别急~再往下一看，哇擦！我写的神鬼莫测的如满天繁星的大学生交作业的华丽效果为什么变了呢？

那么恭喜你，你被**BOM**头坑了。

### 那么，什么是BOM头？

在utf-8编码文件中**BOM**在文件头部，占用三个字节，用来标示该文件属于utf-8编码。但是坑爹的PHP并不能识别**BOM**头。使用记事本等部分工具在写php文件，如果文件中使用到session、header等php内置的函数时，要求这些语句之前必须不能有空格、回车、echo输出语句、html标记等，否则页面回给出警告，并且提示这些语句之前有输出~

但是当你检查代码后发现这些语句前没有上述我们提及到的一些空格、回车、echo输出语句、html标记等，这是怎么回事呢？这就是文件使用utf-8编码保存的时候，有**BOM**标记！

 ## 所以，怎么去除BOM头呢？

 ### 简单的去除**BOM**头的方法就是利用好编辑器。通过编辑器附带的功能去除。

* DW（DreamWeaver）。Ctrl+J调出页面属性，选择左侧，Title/Encoding-->Include Unicode Signature(**BOM**)，如果打勾，将其取消即可。

* 汉化的打开Dreamweaver->选择编辑->首选参数->新建文档标签->右边->"包括Unicode 签名(**BOM**)" 前面的对钩去掉即可

* notepad++可以直接将文件直接转换为UTF-8（无**BOM**编码）。

>**注：**其他编辑器也基本都附带类似功能。

### 下面是通过代码实现

``` <?php
   /**
    * Created by PhpStorm.
    * User: ds12
    * Date: 2015/8/25
    * Time: 14:26
    */
   if (isset($_GET['dir'])){ //config the basedir
       $basedir=$_GET['dir'];
   }else{
       $basedir = '.';
   }

   $auto = 1;

   checkdir($basedir);

   function checkdir($basedir){
       if ($dh = opendir($basedir)) {
           while (($file = readdir($dh)) !== false) {
               if ($file != '.' && $file != '..'){
                   if (!is_dir($basedir."/".$file)) {
                       echo "filename
    $basedir/$file ".checkBOM("$basedir/$file")." <br>";
                   }else{
                       $dirname = $basedir."/".$file;
                       checkdir($dirname);
                   }
               }
           }
           closedir($dh);
       }
   }

   function checkBOM ($filename) {
       global $auto;
       $contents = file_get_contents($filename);
       $charset[1] = substr($contents, 0, 1);
       $charset[2] = substr($contents, 1, 1);
       $charset[3] = substr($contents, 2, 1);
       if (ord($charset[1]) == 239 && ord($charset[2]) == 187 && ord($charset[3]) == 191) {
           if ($auto == 1) {
               $rest = substr($contents, 3);
               rewrite ($filename, $rest);
               return ("<font color=red>BOM found, automatically removed.</font>");
           } else {
               return ("<font color=red>BOM found.</font>");
           }
       }
       else return ("BOM Not Found.");
   }

   function rewrite ($filename, $data) {
       $filenum = fopen($filename, "w");
       flock($filenum, LOCK_EX);
       fwrite($filenum, $data);
       fclose($filenum);
   }
```

>**注：**上面代码放在项目根目录下执行一次即可

还有体验效果好一点的也贴出来

``` <?php
    $s=0;//统计成功数
    $f=0;//统计失败数
    //遍历所有文件
    function
    find_allfile(){
    $i="*";
    while($file=glob($i)){
    foreach($file as
    $s){
    if(!is_dir($s))$allfile[]=$s;
    }
    $i.="\*";
    }
    return
    $allfile;
    }
    //清除BOM标记
    function del_bom(){
    global
    $s,$f;
    $file=find_allfile();
    foreach($file as
    $fname){
    $fname=dirname(__FILE__)."\\".$fname;
    $filecont=@file_get_contents($fname);
    $bom=substr($filecont,0,3);
    $bom=bin2hex($bom);
    if($bom=="efbbbf"){
    //判断文件中的前3个字节是否为BOM标记值
    $filecont=substr($filecont,3);
    $result=@file_put_contents($fname,$filecont,LOCK_EX);
    if($result){
    echo
    "<div id="down"><a id="load" title="下载链接" href="#button_file"><i class="icon-down"></i>下载地址</a><div class="clear"></div></div> $fname --- --- <em style=\"color:green\">清除成功</em><br
    />";$s++;
    }else{
    echo "<div id="down"><a id="load" title="下载链接" href="#button_file"><i class="icon-down"></i>下载地址</a><div class="clear"></div></div> $fname --- --- <em
    style=\"color:red\">清除失败</em>（文件只读或者被占用）<br
    />";$f++;
    }
    }
    }
    }
    del_bom();
    if($s==0 &&
    $f==0){
    echo "<p>所有文件正常，没有发现BOM标记。</p>";
    }else{
    echo
    "<p>统计结果：清除成功($s) | 清除失败($f)</p>";
    }
    ?>
  ```

 **BOM**头
有时候真的是急死个人。







---
layout: post
title: 基于apache的subversion与bugzilla组合的中文问题
categories: SCM
---

这个折腾了半天时间，记录下来

## 起因 ##

打算将svn的log提交到相应的bugzilla上作为注释，post-commit用的是perl WWW::Bugzilla，但总是出现形如?\228?\184?\173?\230?\150?\135（“中文”二字的UTF-8，十进制），不知为何？

## 解决  ##
首先从ubuntu本身来搞，基本找不到这样的问题，难得有个是windows的，设置LANG=zh_CN.UTF-8，APR_ICONV_PATH，居然就搞定了。于是参考。将/etc/default/locale也改成zh_CN.utf8——linux的这个未免太不统一了。

重启apache，失败……


## 硬性解决 ##
<code>
    sub combine3  
    {  
        my ($in) = @_;  
        my $result;  
        if ($in =~ /\?\\(\d+)\?\\(\d+)\?\\(\d+)/)  
        {  
            return pack("C*", $1,$2,$3);  
        }  
        else  
        {  
            print STDERR "Error occupied! call zhangxiaojie\@gozone-mobi.com\n";  
            exit(1);  
        }  
    }  
      
    sub check_log  
    {  
        my ($log) = @_;  
        my $magic="<abcdefghijk123>";  
        while ($log =~ s/((\?\\\d{1,3}){3})/$magic/)  
        {  
            #print "1 $1\n";  
            my $rep = combine3($1);  
            $log =~ s/$magic/$rep/;  
            #print $log, "\n";  
            #print "----\n";  
        }  
        return $log;  
    }  
</code>

## 注意事项 ##
WWW::Bugzilla的addition_comment接受UTF8，放到bugzilla上却是乱码，需要decode('utf-8', xxx)才算完事。

不同系统间的编码整合真烦！UTF8何苦为难UTF8呢……

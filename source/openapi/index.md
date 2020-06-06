---
title:
  '[object Object]': null
date:
  '[object Object]': null
url: 1145.html
id: 1145
comments: false
---

OpenAPI测试页面
-----------

*   [python3-hello](https://api.12ms.xyz/function/hello-python3)
*   [cow](https://api.12ms.xyz/function/cows)
*   [randnum](https://api.12ms.xyz/function/randnum)
*   [md5sum](https://api.12ms.xyz/function/md5sum)

* * *

  

Go

Go

* * *

$ = jQuery.noConflict(); $.get("https://famousmotto.codeday.me/api/getRandom").complete(function(data, status, headers) { console.log(data.responseJSON); $("#whataword").html(data.responseJSON) //unescape(data.replace(/\\\u/g, '%u')) }) // $('#test').html('test') var jqxhr = $.get( "https://api.12ms.xyz/function/cows", function() { }).done(function(data) { $('#cows').html(data) }).fail(function() { console.log( "API req error" ); }) var jqxhr = $.get( "https://api.12ms.xyz/function/randnum", function() { }).done(function(data) { $('#randnum').html(data) }).fail(function() { console.log( "API req error" ); }) $("#rand\_btn").on("click", function(){ if ($('#rand\_txt').val() != "") { data = JSON.stringify($('#rand\_txt').val().split(',').map(x => parseInt(x))) }else { data = "" } $.post( "https://api.12ms.xyz/function/randnum", data ).done(function(data) { $('#randnum').html(data) }).fail(function() { console.log( "API req error" ); }) }) $("#md5\_btn").on("click", function(){ $.post( "https://api.12ms.xyz/function/md5sum", $('#md5_txt').val()).done(function(data) { $('#md5sum').html(data) }).fail(function() { console.log( "API req error" ); }) }) $.post( "https://api.12ms.xyz/function/md5sum", 'test').done(function(data) { $('#md5sum').html(data) }).fail(function() { console.log( "API req error" ); }) $.post( "https://api.12ms.xyz/function/reqtest", 'this is a echo test').done(function(data) { $('#reqtest').html(data) }).fail(function() { console.log( "API req error" ); })
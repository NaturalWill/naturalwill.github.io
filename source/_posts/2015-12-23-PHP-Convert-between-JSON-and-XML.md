---
layout: post
title: PHP JSON与XML间的相互转换
tags: 
  - PHP
date: 2015/12/23
---



由于某些原因，我需要将json转换为xml，在网上想找个将json转成xml的便捷方法，还是挺多的，但不能完全实现我的需求，最后还是在 [海底苍鹰](http://blog.51yip.com/php/660.html) 所写的基础上完善了一下。以下是我所写的方法。

[下载源文件](https://raw.githubusercontent.com/NaturalWill/jphptools/master/json2xml.php)

## json转换成xml

```php
function json_to_xml($source, $charset = 'UTF-8')
{
    if (empty($source)) {
        return false;
    }
    $array = json_decode($source); //php5，以及以上，如果是更早版本，请下载JSON.php
    $xml   = '<?xml version="1.0" encoding="' . $charset . '"?>';
    $xml .= _json_to_xml($array);
    return $xml;
}

function _json_to_xml($source, $pkey = '')
{
    $string = '';
    foreach ($source as $k => $v) {
        if (is_array($v)) { //判断是否是数组
            $string .= _json_to_xml($v, $k);
            
        } else {
			$key = is_numeric($k) ? $pkey : $k; //判断Key是否是数字，如果是，则使用上一级的Key
            $string .= empty($key) ? '' : '<' . $key . '>';
            if (is_object($v)) { //判断是否是对像
                $string .= _json_to_xml($v, $k);
            } else {
                $string .= $v; //取得标签数据
            }
            $string .= empty($key) ? '' : '</' . $key . '>';
        }
    }
    return $string;
}
```
**注意**：此方法支持`<name>aaaa</name>`，不支持`<name type='test'>aaaaa</name>`

## xml转换成json

```php
function xml_to_json($source)
{
	//判断source是file，还是string
    if (is_file($source)) {
        $xml = simplexml_load_file($source);
    } elseif (is_string($source)) {
        $xml = simplexml_load_string($source);
    } else {
		return false;
	}
    $json = json_encode(array($xml->getName() => $xml)); //php5，以及以上，如果是更早版本，请下载JSON.php
    return $json;
}
```

## 例子

```json
{
    "update": {
        "Android": {
            "version": "3", 
            "name": "Naturalwill's APP", 
            "note": "一些细节的优化"
        }, 
        "iOS": {
            "version": "1", 
            "name": "Naturalwill's APP", 
            "note": "正式上线"
        }, 
        "Oldversion": {
            "Android": [
                {
                    "version": "1", 
                    "name": "Naturalwill's APP", 
                    "note": "正式上线"
                }, 
                {
                    "version": "2", 
                    "name": "Naturalwill's APP", 
                    "note": "一些细节的优化"
                }
            ]
        }
    }
}
```

`json_to_xml`  ↓↑  `xml_to_json`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<update>
	<Android>
		<version>3</version>
		<name>Naturalwill's APP</name>
		<note>一些细节的优化</note>
	</Android>
	<iOS>
		<version>1</version>
		<name>Naturalwill's APP</name>
		<note>正式上线</note>
	</iOS>
	<Oldversion>
		<Android>
			<version>1</version>
			<name>Naturalwill's APP</name>
			<note>正式上线</note>
		</Android>
		<Android>
			<version>2</version>
			<name>Naturalwill's APP</name>
			<note>一些细节的优化</note>
		</Android>
	</Oldversion>
</update>
```

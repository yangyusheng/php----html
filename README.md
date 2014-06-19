php----html---------
====================

php截取保留html标签的完整性的实现

```
/*
 * 保留html标签完整性的字符串截取功能
 */
function keepHtmlTagsSubStr($string = '', $length = 0, $more = '...', $encoding = 'utf-8'){
    if(!$string || !is_numeric($length) || $length <= 0){
        return $string;
    }

    $htmlTagRegular = '/(\<[^\>]+?\>)/simU'; //html标签的匹配正则
    $procStringResult = preg_split($htmlTagRegular, $string, -1, PREG_SPLIT_NO_EMPTY|PREG_SPLIT_DELIM_CAPTURE); //把需要处理的$string根据html标签的正则匹配打散成数组

    $contentArr = array(); //记录除html标签外的$string内容的结果集
    $htmlTagsArr = array(); //记录html标签的结果集
    $htmlCloseTagsNameArr = array(); //记录html闭合标签的标签名的结果集
    $needArr = array(); //记录最后截取字符串后的结果集
    $startHtmlTagsArr = array(); //记录html闭合标签的开始的结果集
    $startHtmlTagsNameArr = array(); //记录html闭合标签的开始的标签名的结果集
    $endHtmlTagsArr = array(); //记录html闭合标签的结束的结果集
    $endHtmlTagsNameArr = array(); //记录html闭合标签的结束的标签名的结果集
    $missingTagsNameArr = array(); //记录截取后缺失的标签的标签名的结果集
    $position = null; //记录截取字符串最后所在$procStringResult结果集中的位置
    $lastConentLength = 0; //记录截取字符串最后所在$procStringResult结果集中的位置的需要的内容的长度

    //获取待处理字符串中的html标签结果集与内容结果集
    if(count($procStringResult)){
        foreach($procStringResult as $key=>$val){
            if(preg_match('/\<[^\>]+?\>/simU',$val)){
                $htmlTagsArr[$key] = $val;
            }else{
                $contentArr[$key] = $val;
            }
        }

    }

    //获取html标签结果集中闭合标签的标签名的结果集
    if(count($htmlTagsArr)){
        foreach($htmlTagsArr as $key=>$tagVal){
            if(preg_match('/\<\/(\w+)\s*\>/ismU',$tagVal,$match)){
                if(count($match) && isset($match[1])){
                    $htmlCloseTagsNameArr[$key] = trim($match[1]);
                }
            }
        }
    }

    //截取处理
    if(count($contentArr)){
        $tmpStringLength = 0;
        foreach($contentArr as $key=>$content){
            $tmpStringLength +=(int)mb_strlen($content,$encoding);

            if($tmpStringLength >= $length){
                $position = $key;
                $lastConentLength = $length + (int)mb_strlen($content,$encoding) - $tmpStringLength;
                break;
            }
        }

    }

    //记录截取的最后位置以及需要的内容的结果集
    $position && ($needArr = array_slice($procStringResult, 0, $position + 1));

    //处理需要的内容的结果集中的html标签不完整问题
    if(count($needArr)){
        $needArr[$position] = mb_substr($needArr[$position], 0, $lastConentLength, $encoding);//处理截取所在$procStringResult结果集中的最后位置的需要的内容长度

        foreach($needArr as $key=>$needVal){
            if(preg_match('/\<[^\/\>]+?\>/simU',$needVal)){//截取内容中含有的html开始的标签
                $startHtmlTagsArr[$key] = $needVal;
            }elseif(preg_match('/\<\/[^\>]+?\>/simU',$needVal)){//截取内容中含有的html结束的标签
                $endHtmlTagsArr[$key] = $needVal;
            }
        }
    }

    //截取中缺失相应的html标签的处理(获取相应的标签名)
    if(count($startHtmlTagsArr) != count($endHtmlTagsArr)){
        foreach($startHtmlTagsArr as $startTagVal){
            if(preg_match('/\<(\w+)(\s+.*|\s*)\>/ismU',$startTagVal,$startMatch)){
                if(count($startMatch) && isset($startMatch[1])){
                    $startHtmlTagsNameArr[] = trim($startMatch[1]);
                }
            }

        }

        foreach($endHtmlTagsArr as $endTagVal){
            if(preg_match('/\<\/(\w+)\s*\>/ismU',$endTagVal,$endMatch)){
                if(count($endMatch) && isset($endMatch[1])){
                    $endHtmlTagsNameArr[] = trim($endMatch[1]);
                }
            }
        }
    }

    //获取截取中缺失的闭合的html标签的标签名
    if(count($startHtmlTagsNameArr)){
        foreach($endHtmlTagsNameArr as $key=>$endTagName){
            if(in_array($endTagName, $startHtmlTagsNameArr)){
                unset($endHtmlTagsNameArr[$key]);
            }

            foreach($startHtmlTagsNameArr as $key=>$startTagName){
                if($startTagName == $endTagName){
                    unset($startHtmlTagsNameArr[$key]);
                    break;
                }
            }
        }

        $missingTagsNameArr = $startHtmlTagsNameArr;
    }

    if(count($missingTagsNameArr) && count($htmlCloseTagsNameArr)){
        foreach($htmlCloseTagsNameArr as $key=>$tagName){
            if(in_array($tagName,$missingTagsNameArr)){
                $needArr[] = $htmlTagsArr[$key];
                unset($htmlCloseTagsNameArr[$key]);

                foreach($missingTagsNameArr as $key=>$missingVal){
                    if($tagName == $missingVal){
                        unset($missingTagsNameArr[$key]);
                        break;
                    }
                }
            }

        }

    }

    if(count($needArr)){
        return implode('',$needArr) . $more;
    }

    return $string;

}
```



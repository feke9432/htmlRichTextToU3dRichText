# html富文本转成u3d引擎可用的富文本

## 代码

```javascript
// 多行的html源码
let data = `<p><strong>我是加粗的<em>倾斜的</em><span style='color:rgb(256,1,256)'>有颜色的<span style='font-size:24px'>大</span></span>文字</strong></p>
<p><span style=\"color: rgb(84, 141, 212); font-size: 20px;\"><strong>关切</strong></span>评<em>估人加<span style=\"font-size: 20px;\">热片区</span></em></p>`;

    function createElementFromHTML(htmlString) {
        let div = document.createElement('div');
        div.innerHTML = htmlString.trim();
        return div
    }

    function parse(ele) {
        let text = "";
        for (let e of ele.childNodes) {
            if (e instanceof HTMLElement) {
                text += getTagName(e, true) + parse(e) + getTagName(e, false);
                continue
            }
            text += e.textContent;
        }

        return text;
    }

    function getTagName(htmlElement,isHead) {
        switch (htmlElement.nodeName.toLocaleLowerCase()) {
            case "strong":
                if (isHead) {
                    return "<b>";
                }
                return "</b>";
            case "em":
                if (isHead) {
                    return "<i>";
                }
                return "</i>";
            case "p":
                if (isHead) {
                    return "";
                }
                return "\r\n";
            case "span":
                let style = htmlElement.style.cssText;
                let colorReg = /color:\s?rgb\((\d{1,3}),\s?(\d{1,3}),\s?(\d{1,3})\);?/;
                let sizeReg = /font-size:\s?(\d{1,3})px?;?/;

                let colorText = "";
                let sizeText = "";

                let colors = colorReg.exec(style);
                if (colors != null && colors.length > 1) {
                    if (isHead) {
                        colorText = "<color=#"+getColorHex(colors[1])+getColorHex(colors[2])+getColorHex(colors[3])+">";
                    } else {
                        colorText = "</color>";
                    }
                }

                let sizes = sizeReg.exec(style);
                if (sizes != null && sizes.length > 1) {
                    if (isHead) {
                        sizeText = "<size="+sizes[1]+">";
                    } else {
                        sizeText = "</size>";
                    }
                }

                if (isHead) {
                    return colorText+sizeText
                }

                return sizeText+colorText;
            default:
                return ""
        }
    }

    // 将rgb的颜色值转为16进制的颜色值
    function getColorHex(v) {
        let num = parseInt(v);
        let text = num.toString(16);
        if (num <= 0xf) {
            text = "0"+text
        }

        return text
    }
```

## 如何使用

执行`parse(createElementFromHTML(data));`得到以下文本：
```
<b>我是加粗的<i>倾斜的</i><color=#ff01ff>有颜色的<size=24>大</size></color>文字</b>
<color=#548dd4><size=20><b>关切</b></size></color>评<i>估人加<size=20>热片区</size></i>
```
将转换后的文本及html源码文本一起保存到服务器，服务器将转换后的文本发送到游戏客户端，html源码文本用于修改时在浏览器上显示，这样减少了把转换后的文本又转回html源码的麻烦


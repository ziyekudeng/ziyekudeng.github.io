---
layout: post
title: 自定义控件-jquery 限制文本框只能输入数字
category: jquery
tags: [jquery]
---
/**
 * @description 用于验证客户输入是否是数字和字母
 * @author gao wei
 * @date
 * @使用方法:默认在head文件引入,使用时在input框的class中写入对应属性
 * 限制使用了onlyNum类样式的控件只能输入数字
 * 限制使用了onlyAlpha类样式的控件只能输入字母
 * 限制使用了onlyNumAlpha类样式的控件只能输入数字和字母
 */
 
// ----------------------------------------------------------------------
// 限制只能输入数字
// ----------------------------------------------------------------------

        $.fn.onlyNum = function () {
            $(this).css("ime-mode", "disabled");
            this.bind("keypress",function(e) {
                var code = (e.keyCode ? e.keyCode : e.which);  //兼容火狐 IE
                if(!$.browser.msie&&(e.keyCode==0x8))  //火狐下不能使用退格键
                {
                    return ;
                }
                return code >= 48 && code<= 57;
            });
            this.bind("blur", function() {
                if (this.value.lastIndexOf(".") == (this.value.length - 1)) {
                    this.value = this.value.substr(0, this.value.length - 1);
                } else if (isNaN(this.value)) {
                    this.value = "";
                }
            });
            this.bind("paste", function() {
                var s = window.clipboardData.getData('text');
                if (!/\D/.test(s.replace(/\s+/g,"")));
                value = s.replace(/^0*/, '');
                return false;
            });
            this.bind("dragenter", function() {
                return false;
            });
            this.bind("keyup", function() {
                if (/(^0+)/.test(this.value.replace(/\s+/g,""))) {
                    this.value = this.value.replace(/^0*/, '');
                }
            });
    
    };
    
// ----------------------------------------------------------------------
// 限制只能输入字母 /^[a-zA-Z]+$/
// ----------------------------------------------------------------------

        $.fn.onlyAlpha = function () {
            $(this).css("ime-mode", "disabled");
            this.bind("keypress",function(e) {
                var code = (e.keyCode ? e.keyCode : e.which);  //兼容火狐 IE
                if(!$.browser.msie&&(e.keyCode==0x8))  //火狐下不能使用退格键
                {
                    return ;
                }
                return code >= 64 && code<= 90;
            });
            //失去焦点触发
            this.bind("blur", function() {
                 if (!/^[a-zA-Z]+$/.test(this.value.replace(/\s+/g,""))) {
                    this.value = "";
                }
            });
            //粘贴时触发
            this.bind("paste", function() {
                var s = window.clipboardData.getData('text');
                if (!/^[a-zA-Z]+$/.test(s.replace(/\s+/g,""))) {
                    this.value = "";
                }
                return false;
            });
            this.bind("dragenter", function() {
                return false;
            });
            this.bind("keyup", function() {
                if (!/^[a-zA-Z]+$/.test(this.value.replace(/\s+/g,""))) {
                    this.value = "";
                }
            });
        };
        
// ----------------------------------------------------------------------
// 限制只能输入数字和字母 /^(\d|[a-zA-Z])+$/
// ----------------------------------------------------------------------

    $.fn.onlyNumAlpha = function () {
        $(this).css("ime-mode", "disabled");
        this.bind("keypress",function(e) {
            var code = (e.keyCode ? e.keyCode : e.which);  //兼容火狐 IE
            if(!$.browser.msie&&(e.keyCode==0x8))  //火狐下不能使用退格键
            {
                return ;
            }
            return (code >= 48 && code<= 57) || (code >= 65 && code<= 90) || (code >= 97 && code<= 122);
        });
        //失去焦点触发
        this.bind("blur", function() {
            if (!/^(\d|[a-zA-Z])+$/.test(this.value.replace(/\s+/g,""))) {
                this.value = "";
            }
        });
        //粘贴时触发
        this.bind("paste", function() {
            var s = window.clipboardData.getData('text');
            if (!/^(\d|[a-zA-Z])+$/.test(s.replace(/\s+/g,""))) {
                this.value = "";
            }
            return false;
        });
        this.bind("dragenter", function() {
            return false;
        });
        this.bind("keyup", function() {
            if (!/^(\d|[a-zA-Z])+$/.test(this.value.replace(/\s+/g,""))) {
                this.value = "";
            }
        });
    };

$(function () {
// 限制使用了onlyNum类样式的控件只能输入数字
    $(".onlyNum").onlyNum();
//限制使用了onlyAlpha类样式的控件只能输入字母
    $(".onlyAlpha").onlyAlpha();
// 限制使用了onlyNumAlpha类样式的控件只能输入数字和字母
    $(".onlyNumAlpha").onlyNumAlpha();
});

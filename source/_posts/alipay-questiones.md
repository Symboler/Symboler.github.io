---
title: 阿里2018面试题总结
date: 2018-04-17 20:18:46
tags:
---
## 使用css实现一个持续的动画效果
 主要考察animation的用法

` animation:mymove 5s infinite;
    
    @keyframe mymove {
    from {top:0;}
    to{top:200px;}
  }
`
## 使用js实现一个持续的动画效果
使用requestAnimationFrame
`   
    //兼容性处理
    
    window.requestAnimFrame = (function(){
      return window.requestAnimationFrame
        window.webkitRequestAnimationFrame
        window.mozRequestAnimationFrame
        function(callback){
          window.setTimeout(callback,1000/60)
        }
    })()
    var e = document.getElementById("e")
    var flag = true;
    var left = 0;
    function render(){
      left ==0?flag=true:left==100?flag=false:'';
      flag?e.style.left=`${left++}px`;
        e.style.left =  `${left--}px`;
    }
    (function animloop(){
      render();
      requestAnimFrame(animloop);
    })()
  `
  浏览器可以优化并行的动画动作，更合理的重新排列动作序列，并把能够合并的动作放在一个渲染周期内完成，从而呈现出更流畅的动画效果
  解决毫秒的不准确性
  避免过度渲染（渲染频率太高、tab不可见暂停等等）
  注：requestFrame和定时器一样也有一个类似的清除方法cancelAnimationFrame
# Changelog

This is the changelog page of this project. More details please check the github repo.

<script async src="//busuanzi.ibruce.info/busuanzi/2.3/busuanzi.pure.mini.js"></script>
<script>
    $(document).ready(function () {

        var int = setInterval(fixCount, 50);  // 50ms周期检测函数
        var pvcountOffset = 80000;  // 初始化首次数据


        function fixCount() {
            if (document.getElementById("busuanzi_container_site_pv").style.display != "none") {
                $("#busuanzi_value_site_pv").html(parseInt($("#busuanzi_value_site_pv").html()) + pvcountOffset);
                clearInterval(int);
            }
            if ($("#busuanzi_container_site_pv").css("display") != "none") {
                $("#busuanzi_value_site_uv").html(parseInt($("#busuanzi_value_site_uv").html()) + uvcountOffset); // 加上初始数据 
                clearInterval(int); // 停止检测
            }
        }
    });
</script>

<span id="busuanzi_container_site_pv">本站总访问量<span id="busuanzi_value_site_pv"></span>次</span>

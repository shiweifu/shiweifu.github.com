终于下决心开始学习shell了，虽然基本shell的所有功能都能通过Python来完成，但其强大的地方在于起到一种胶水的作用，将unix/linux粘在一起，成为一个有机体。
从学习linux的角度来讲是十分有必要的。
以前切割字符串多用Python，代码这么写：
<pre language="python">
line="nobody:*:-2:-2:Unprivileged User:/var/empty:/usr/bin/false"
line.split(":")
</pre>


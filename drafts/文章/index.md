# 

昨天逛 v 站看到一个帖子(go websocket server 开启压缩内存占用高的问题)[https://www.v2ex.com/t/939692]，文章中提到了 websocket 开启压缩后内存占用高的问题，问题的核心直指 compress/flate。为此，我在 goalng 的 github issues 中搜索相关问题，看到了相似的 issues [goalng issues](https://github.com/golang/go/issues/32371)。

[](https://github.com/ethereum/go-ethereum/issues/22567)

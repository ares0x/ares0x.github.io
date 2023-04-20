# 短 url 设计


短链接是一种能将原始网址缩短为更短的链接的技术。通常用于减少 URL 的长度，便于在短信或社交媒体等渠道中传播链接，同时可以进行统计分析。
• 原理简介：短链接的原理是通过一个短的地址来代替原始地址，将目标网址与短地址进行映射，当访问短链接时，服务器会将客户端重定向到目标网址。

## 几个问题
1. 短 url 的长度
2. 跳转方式 301 OR 302
3. 转换算法


## 核心逻辑

```go
//  Encode 生成短链接并存储
func Encode(originalUrl string) string {
    uuid := time.Now().UnixNano() // uuid 可以通过多种方式生成，如，redis incr，独立的发号器等
	var shortUrl string
	for uuid > 0 {
		shortUrl = string(constvar.Seed[uuid%62]) + shortUrl
		uuid = uuid / 62
	}
    // TODO 将 uuid，originalUrl（原始 url）和 shortUrl 组成的三元组存储
    return shortUrl
}

// Decode 根据短链接获取原始链接
func Decode(shortUrl string) (string, error) {
	// 根据短链接获取唯一id
	var id int64
	n := len(shortUrl)
	for i := 0; i < n; i++ {
		pos := strings.IndexByte(constvar.Seed, shortUrl[i])
		id = id*62 + int64(pos)
	}
	urlQuery := data.New(nil)
	url, err := urlQuery.GetById(id)
	if err != nil {
        return "", err
	}
	return url.OriginalUrl, nil
}

```

## 优化
* uuid 的生成方式。在分布式环境下建议通过第三方组件来生成递增的唯一 id，以保证生成的 shortUrl 不重复
* 可以通过 redis 做缓存来加速访问

代码地址：
https://github.com/ares0x/system-design/tree/main/short-url

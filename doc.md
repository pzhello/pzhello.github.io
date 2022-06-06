#### 对接说明

##### 1. 相互说明
接口网页端已经封装好了。双方通过postMessage交互：

- 游戏方：

1. 发送nftList事件请求指定钱包地址的nft列表，
  事例：
   { 
      source: 'game', // 发起方
      type: 'nftList',    // 请求类型
      data: {wallet: '0xabc0k....' }// 钱包地址
    }
    web端会同时监听这个事件，收到后，发送列表事件
2. 接收web端发送的nftList列表
   {
      source: 'web', // 发起方
      type: 'nftList',    // 请求类型
      data: [{nft record}] // nft记录列表，详见之前发的对接文档
   }    
3. 建议本地缓存，建议本地缓存一下，不用每次都请求，操作比较耗时。

##### 2. 对接代码实现

```
// H5与浏览器通信 iframe to parent window
// 事例1
function SendMsgToParentPage(message) {
    parent.postMessage(message, "*");
}
// H5监听message事件，并过滤消息类型，发送数据结构双方约定好
window.addEventListener("message", function(event) {
    // 过滤域 event.origin === 'http://localhost:8080'
    // 过滤消息类型 event.data && "type" in event.data
    // event.data数据结构如下：
    // 1. nft数据
    // {
    //     type: "nftList",
    //     source: "web",
    //     data: [{"img":"...", "type":"eth"},{}]
    // }
    if (event.data && "type" in event.data && "source" in event.data && event.type === "message") {
        const data = event.data
        console.log('[App] data ', data)
        // 向父窗口发送消息
        SendMsgToParentPage(data)
        try {
            // 这里写处理方法
            // 例如：如果是nft消息，通过nft处理方法来处理
        } catch (e) {
            console.log('[App] exception is ', e)
        }
    }
}, false)


// 浏览器与iframe嵌入的 h5 通信，from explore to h5
// 事例
function sendMessage(message, callback)
{
    // 获取通信的子iframe
    const iframe = window.frames[0];

    // 向目标iframe发送message,"*"可以通过指定地址,限制发送的目标
    iframe.postMessage(JSON.stringify(message), "*");
    if (callback) {
        // 100毫秒后用户可再次点击控制按钮发送请求
        setTimeout(callback,100);
    }
}
```

##### 3. web唤起游戏参数定义

```
# 其他说明
// 游戏接收url拼接字符串，拿到钱包的基础信息
open https://gameurl?roomId=1000&wallet=0xabc...&owned=true

```

字段名称 | 类型 | 字段必须 | 备注
--- | --- | --- | ---
roomId| Integer | required | 用户进入房间ID
wallet| String | required | 钱包地址
owned | Boolean | required | 是否为房间所有者 false：非所有者，true:所有者

##### 4. NFT数据结构

- 结构事例

```
[{"token_id":String,"nft_name":String,"image":String},...]

事例：
[
    {
        "token_id": "28712083738263997533335551499943991043390627901407616538526294326653471563439",
        "nft_name": "iamxmm.eth",
        "image":"https://metadata.ens.domains/mainnet/0x57f1887a8bf19b14fc0df6fd9b2acc9af147ea85/0x3f7a76a8029d31e439cc8d9f27af185cd490dbfb000fce0878e264f177d21eaf/image"
    },
    {
        "token_id": "147",
        "nft_name": "Door #147",
        "image":"https://ipfs.io/ipfs/Qmb14XGzJHbPnt4jN5zLFqzBiHFNfUxUULq2pV9jjEpPQv/147.svg"
    },
    {
        "token_id": "1028",
        "nft_name": "COVIDPunk #1028",
        "image":"https://gateway.pinata.cloud/ipfs/QmWpEGsht6aYi5myYAyUpjdLB44eytD9U8uUeeyppSw6DZ"
    }
]
```

- 原文件

```
[
    {
        "token_id": "28712083738263997533335551499943991043390627901407616538526294326653471563439",
        "nft_name": "iamxmm.eth",
        "nft_json": "{\"is_normalized\":true,\"name\":\"iamxmm.eth\",\"description\":\"iamxmm.eth, an ENS name.\",\"attributes\":[{\"trait_type\":\"Created Date\",\"display_type\":\"date\",\"value\":null},{\"trait_type\":\"Length\",\"display_type\":\"number\",\"value\":6},{\"trait_type\":\"Registration Date\",\"display_type\":\"date\",\"value\":1645094337000},{\"trait_type\":\"Expiration Date\",\"display_type\":\"date\",\"value\":1802879097000}],\"name_length\":6,\"url\":\"https://app.ens.domains/name/iamxmm.eth\",\"version\":0,\"background_image\":\"https://metadata.ens.domains/mainnet/avatar/iamxmm.eth\",\"image_url\":\"https://metadata.ens.domains/mainnet/0x57f1887a8bf19b14fc0df6fd9b2acc9af147ea85/0x3f7a76a8029d31e439cc8d9f27af185cd490dbfb000fce0878e264f177d21eaf/image\"}"
    },
    {
        "token_id": "147",
        "nft_name": "Door #147",
        "nft_json": "{\"description\":\"The inaugural launch of Meta User Dungeon (Cycle 0 - Enter the Void) consists of a series of 4179 programatically generated & animated SVG tokens. When one door opens, another door opens.\",\"external_url\":\"https://metadungeon.quest/doors\",\"image\":\"ipfs://Qmb14XGzJHbPnt4jN5zLFqzBiHFNfUxUULq2pV9jjEpPQv/147.svg\",\"name\":\"Door #147\",\"attributes\":[{\"trait_type\":\"Ring\",\"value\":\"???\"},{\"trait_type\":\"Element\",\"value\":\"???\"},{\"trait_type\":\"Suit\",\"value\":\"???\"},{\"trait_type\":\"Theme\",\"value\":\"???\"},{\"trait_type\":\"Animation\",\"value\":\"???\"},{\"trait_type\":\"Quest\",\"value\":\"???\"}]}"
    },
    {
        "token_id": "1028",
        "nft_name": "COVIDPunk #1028",
        "nft_json": "{\"image\":\"https://gateway.pinata.cloud/ipfs/QmWpEGsht6aYi5myYAyUpjdLB44eytD9U8uUeeyppSw6DZ\",\"external_url\":\"https://www.covidpunks.com\",\"tokenId\":1028,\"name\":\"COVIDPunk #1028\",\"description\":\"One of 10,000 computer-generated punks who politely comply with local personal protective gear regulations. Way to go, champ!\",\"attributes\":[{\"value\":\"Mohawk Thin\",\"trait_type\":\"Hair\"},{\"value\":\"Big Shades\",\"trait_type\":\"Eyes\"},{\"value\":\"Male\",\"trait_type\":\"Sex\"},{\"value\":\"Pink\",\"trait_type\":\"Mask\"},{\"value\":\"Vaccinated\",\"trait_type\":\"Vaccine Status\"},{\"value\":\"Pfazer\",\"trait_type\":\"Vaccine\"},{\"value\":\"Healthy\",\"trait_type\":\"COVID Status\"},{\"value\":\"Common\",\"trait_type\":\"Rarity\"}]}"
    }
]
```
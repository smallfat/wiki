---
title: ScryInfo 协议层（简化版）SDK接口文档 v1.0
tags: 
grammar_cjkRuby: true
---



### Scry协议层业务流程

![Diagram](./attachments/1552098289109.drawio.html)

### 接口
##### Sdk 
Package: sdk

![表格](./attachments/1552275406062.table.html)

##### ScryClient - Scry客户端操作
Package: sdk/scryclient

![表格](./attachments/1552112711362.table.html)

##### Contract - 合约操作
Package: sdk/scryclient/chaininterfacewrapper

![表格](./attachments/1552116179330.table.html)

##### Key Manager - 密钥管理接口
Package: sdk/util/accounts

![表格](./attachments/1552269049817.table.html)

##### Event - 合约事件
所有的Event都以JSON格式进行传输。

![表格](./attachments/1552267409890.table.html)

### 例程

##### 调用SDK的第一步：初始化SDK
``` golang
func main() {	
	wd, _ := os.Getwd()
	err := sdk.Init(
	    "http://192.168.1.12:8545/",
	    "192.168.1.6:48080",
	    protocolContractAddr,
	    tokenContractAddr,
	    0,
	    "/ip4/192.168.1.6/tcp/5001",
	    wd+"/testconsole.log",
	    "scryapp1")
		
	......
}
```

##### 第二步：创建用户实例
系统预先生成了用户账户，该账户内包含ETH和token，到时候会分配给各团队
用户需要根据分配的账户地址生成用户实例。

``` golang
seller := scryclient.NewScryClient(address)
buyer := scryclient.NewScryClient(address)

//订购事件
seller.SubscribeEvent("DataPublish", onPublish)
seller.SubscribeEvent("Buy", onPurchase)
seller.SubscribeEvent("TransactionCreate", onTransactionCreate)
seller.SubscribeEvent("TransactionClose", onClose)

//买方若收到onApprovalBuyerTransfer通知，表明approve成功
buyer.SubscribeEvent("Approval", onApprovalBuyerTransfer)
buyer.SubscribeEvent("TransactionCreate", onTransactionCreate)
buyer.SubscribeEvent("ReadyForDownload", onReadyForDownload)
buyer.SubscribeEvent("TransactionClose", onClose)
```

##### 下面可以开始交易了
###### 卖方发布数据
``` golang
func SellerPublishData(supportVerify bool) {


	//待发布的元数据
	metaData := []byte("magic meta data test")
	//一些元数据片段: 用以证明元数据的真实性
	proofData := [][]byte{{'4', '5', '6', '3'}, {'2', '2', '1'}}
	//元数据的描述数据
	despData := []byte{'7', '8', '9', '3'}

	txParam := chainoperations.TransactParams{
	    From: common.HexToAddress(seller.Account.Address),
	    Password: keyPassword,
    }

	cif.Publish(
	    &txParam,
	    big.NewInt(1000),
	    metaData,
	    proofData,
	    2,
	    despData,
	)
}
```
###### 买方收到数据发布通知，取到数据的描述数据ID，据此买方可以从IPFS上下载数据描述信息
``` golang
func onPublish(event events.Event) bool {
   //数据的发布ID
	publishId = event.Data.Get("publishId").(string)
	//描述数据ID
	despDataId := event.Data.Get("despDataId").(string)
	//价格
	price := event.Data.Get("price").(*big.Int)
	return true
}
```

###### 看到数据描述信息后，买方对该数据很感兴趣，于是设定待支付押金额度，允许协议层智能合约从其账户转移Token用作押金
``` golang
func BuyerApproveTransfer() {   

    //buyerPassword: 买方账户密码
    txParam := chainoperations.TransactParams{
        From: common.HexToAddress(buyer.Account.Address),
        Password: buyerPassword,
    }
	
	//protocolContractAddr是协议层合约地址
	//押金数额要大于等于数据价格,否则购买操作会失败	
	err := cif.ApproveTransfer(&txParam,
	    common.HexToAddress(protocolContractAddr),
	    big.NewInt(1000))
		
	if err != nil {
		fmt.Println("BuyerApproveTransfer:", err)
	}
}
```


###### 买方准备购买，智能合约从买方账户扣除token押金

``` golang
func PrepareToBuy(publishId string) {
    txParam := chainoperations.TransactParams{
        From: common.HexToAddress(buyer.Account.Address),
        Password: buyPassword,
    }
	
	err := cif.PrepareToBuy(&txParam, publishId)
	if err != nil {
		fmt.Println("failed to prepareToBuy, error:", err)
	}
}
```

###### 卖方得到transaction id
``` golang
func onTransactionCreate(event events.Event) bool {
    fmt.Println("seller: onTransactionCreated:", event)
	
	//本次交易ID
    txId = event.Data.Get("transactionId").(*big.Int)

    return true
}
```

###### 买方得到数据证明ID，凭借这些ID，买方可以从IPFS上下载这些证明，判断数据是不是自己想要的

``` golang

func onTransactionCreate(event events.Event) bool {
    //本次交易ID
	txId = event.Data.Get("transactionId").(*big.Int)
	
	//数据证明IDs
    proofIDs := event.Data.Get("proofIds").([][32]byte)

	return true
}

```


###### 买方决定正式购买数据

``` golang
func Buy(txId *big.Int) {
    //卖方需要订购Buy事件，这样才能收到买方的购买消息
	seller.SubscribeEvent("Buy", onPurchase)

    //buyerPassword是买方账户密码
    txParam := chainoperations.TransactParams{
        From: common.HexToAddress(buyer.Account.Address),
        Password: buyerPassword,
    }
	
	//txId为本次交易ID
	err := cif.BuyData(&txParam, txId)
	
	if err != nil {
		fmt.Println("failed to buyData, error:", err)
	}
}
```

###### 卖方收到购买数据的通知，生成使用买方公钥加密的meta data id，发给合约

``` golang
func onPurchase(event events.Event) bool {
	fmt.Println("onPurchase:", event)
	
	//使用卖方公钥加密的meta data id
	metaDataIdEncWithSeller = event.Data.Get("metaDataIdEncSeller").([]byte)
	//买方地址
	buyerAddr := event.Data.Get("buyer").(common.Address)

	var err error
	//metaDataIdEncWithBuyer： 使用买方公钥加密的meta data id
	//metaDataIdEncWithSeller: 用卖方公钥加密的meta data id
	//sellerPassword: 卖家密码
    metaDataIdEncWithBuyer, err = accounts.GetAMInstance().ReEncrypt(
        metaDataIdEncWithSeller,
        seller.Account.Address,
        buyerAddr.String(),
        clientPassword,
    )

    if err != nil {
        fmt.Println("failed to ReEncrypt meta data id with buyer's public key")
        return false
    }

    SubmitMetaDataIdEncWithBuyer(txId)
	return true
}


func SubmitMetaDataIdEncWithBuyer(txId *big.Int) {
    //买方需要监听ReadyForDownload事件
    txParam := chainoperations.TransactParams{
        From: common.HexToAddress(seller.Account.Address),
        Password: sellerPassword,
    }
	
	err := cif.SubmitMetaDataIdEncWithBuyer(
	    &txParam,
	    txId,
	    metaDataIdEncWithBuyer)
	if err != nil {
		fmt.Println("failed to SubmitMetaDataIdEncWithBuyer, error:", err)
	}
}
```

###### 买方拿到meta data id，终于可以从IPFS上下载完整数据

``` golang
func onReadyForDownload(event events.Event) bool {
	metaDataIdEncWithBuyer = event.Data.Get("metaDataIdEncBuyer").([]byte)

    //买方解密metaDataIdEncWithBuyer，得到原始meta data id，根据这个id从IPFS上下载真正的数据
	metaDataId, err := accounts.GetAMInstance().Decrypt(
	    metaDataIdEncWithBuyer,
	    buyer.Account.Address,
	    buyerPassword)

    if err != nil {
        fmt.Println("failed to decrypt meta data id with buyer's private key", err)
        return false
    }
    
	return true
}
```

###### 买方看完数据，反馈数据的真实性

``` golang
func ConfirmDataTruth(txId *big.Int) {
	buyer.SubscribeEvent("TransactionClose", onClose)

    txParam := chainoperations.TransactParams{
        From: common.HexToAddress(buyer.Account.Address),
        Password: buyerPassword,
    }
	
	//txId: 本次交易ID
	err := cif.ConfirmDataTruth(
	    &txParam,
	    txId,
	    true)
	if err != nil {
		fmt.Println("failed to ConfirmDataTruth, error:", err)
	}
}
```

###### 交易关闭通知
``` golang
func onClose(event events.Event) bool {
	fmt.Println("onClose:", event)
	return true
}
```


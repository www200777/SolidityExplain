# 二、Solidity特别类型 - 地址类型
## 1. 地址是什么
在以太坊中，一个账户对应着一个地址，不管是外部账户还是合约账户。外部账户在创建账户键入密码后，以太坊通过一系列的算法，生成出密钥对，公钥经过转换得到外部账户的地址。合约在被部署到区块链上时，会根据创建者地址和nonce值等，共同生成合约地址，其他账户可以通过合约地址来调用合约中的功能。

在Solidity中，地址类型address是一个值类型，长度为20个字节。地址类型也有成员，是所有合约的基础。
***
## 2. 地址类型的成员
* `<address>.balance(uint256)` 用于返回某一地址的余额，单位为Wei
* `<address>.send(uint256 amount) returns (bool)` 给address发送以太币，单位为Wei
* `<address>.transfer(uint256 amount)` 给address发送以太币，单位为Wei
* `<address>.call(bytes memory) returns (bool)`
* `<address>.callcode(bytes memory) returns (bool)`
* `<address>.delegatecall(bytes memory) returns (bool)`
### 2.1 send()和transfer()函数
`send`和`transfer`都用于给地址发送以太币，如果地址是合约地址，合约的回退函数（`fallback`函数）会随`send/transfer`调用一起执行（EVM的特性）。
两者的不同点：
* 如果交易执行失败，`transfer`会返回exception，使合约调用终止，并返还以太币；`send`会返回false，合约调用不会终止。
* `send`调用存在一定的风险，如果调用栈深度超过1024或者接收方用完了gas（因为会默认执行接收方的`fallback`函数），交易会失败。为了保证以太币转移安全性，可以检查send调用的返回值/使用`transfer`/接收方合约设置了`withdraw`模式

### 2.2 call(),callcode()和delegatecall()函数
* 可用`call()`和非ABI协议的合约进行交互，用来向另一个合约发送原始数据。
* `delegatecall()`和`call()`类似，区别在于只使用给定地址合约的代码，其他方面（存储、余额...）取自当前合约
* `callcode()`在未来版本中将要被移除

>  `call()`可以调用标识了`payable`的函数来向合约发送以太币，具体写法是：`<address>.call.value(amount)(bytes4(keccak256("函数名称(入参类型)"))`，同理如果只想调用该函数就把`.value()`去掉。

***
## 3. 另外一些全局变量
```javascript
//访问区块信息
blockhash(uint blockNumber)
block.coinbase //address
block.difficulty //uint
block.gaslimit //uint
block.number //uint
block.timestamp //uint
gasleft() returns (uint256) //剩余gas

//访问调用者信息
msg.data //bytes 完整的calldata信息
msg.sender //address 调用者地址
msg.sig //bytes4 calldata前四个字节
msg.value //uint

now //uint 当前区块时间戳

//访问交易信息
tx.gasprice //uint
tx.origin //address 交易发起人地址
```
***
## 4. 合约示例
以下示例用于展示`send/transfer`用法
```javascript
contract Addr {
	event logdata(bytes data);

	function () payable {
		logdata(msg.data);		
	}

	function getBalance () returns(uint) {
		return this.balance;
	}		
}
contract Test {
	function Test () payable{	
	}	

	event logSendEvent(address to, uint value);

	function transferEther (address to) payable {
		to.transfer(10);
		logSendEvent(to, 10);
	}
	function sendEther (address to) payable {
		require(to.send(10));
		logSendEvent(to, 10);
	}
	
	function getBalance () returns(uint) {
		return this.balance;
	}		
}
```
调用`sendEther()`和`transferEther()`会触发两个`event`事件，首先触发Addr合约中的`logdata()`事件，然后再触发Test合约中的`logSendEvent()`事件。事件会被记录在区块链上交易收据的log字段中，以下为事件触发记录的形式：
```json
[
	{
		"topic": "34920d37f800163b15fbdb7561da8052378b409e0fadd3b08a9aecaf3e7ef6c0",
		"event": "logdata",
		"args": [
			"0x"
		]
	},
	{
		"topic": "843274b1501cdcbdd49af3e4cfd3c0332ea2a92bdf82e9d97f2ca51937ef8558",
		"event": "logSendEvent",
		"args": [
			"0x11f0c46dc617619d51aa60e02daca7608dc714e7",
			"10"
		]
	}
]
```
示例中涉及到`payable`、`event`和`fallback`回退函数等将在后文中解释其用法。
***
下一篇文章将讲述一些在编写合约中常用的关键字及其用法。


# 出块奖励返回

## 简述
CITA 中存在两种经济模型：`Quota` 和 `Charge` 模型，默认经济模型 `Quota` ，没有余额概念。
在具有余额的 `Charge` 经济模型中，出块奖励默认返还给矿工，但运营方可以通过配置，将出块奖励返还给自己。

矿工手续费 = quotaUsed * quotaPrice, 其中 `quotaPrice` 默认为 1，`quotaUsed` 在交易回执中可以获取。 

### 操作示例

首先配置链的时候要注意三点(以下均是默认值)：
- 配置经济模型 `--contract_arguments "SysConfig.economicalModel=1" 默认为1`
- 设置奖励返回开关 `--contract_arguments "SysConfig.checkFeeBackPlatform=true" 默认false`
- 设置运营方地址 `--contract_arguments "SysConfig.chainOwner=0x36a60d575b0dee0423abb6a57dbc6ca60bf47545" 默认全0`

```bash
$./env.sh ./scripts/create_cita_config.py create --nodes "127.0.0.1:4000,127.0.0.1:4001,127.0.0.1:4002,127.0.0.1:4003" --contract_arguments "SysConfig.checkFeeBackPlatform=true" --contract_arguments "SysConfig.chainOwner=0x36a60d575b0dee0423abb6a57dbc6ca60bf47545"  --contract_arguments "SysConfig.economicalModel=1"
```

*接下来的测试，用 [cita-cli](https://github.com/cryptape/cita-cli) 交互模式进行演示*。

查看管理员和运营方地址余额
```bash
$rpc getBalance --address "0x4b5ae4567ad5d9fb92bc9afd6a657e6fa13a2523"
```
输出：
```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "result": "0xffffffffffffffffffffffffff"
}
```

```bash
$rpc getBalance --address "0x36a60d575b0dee0423abb6a57dbc6ca60bf47545"
```
输出：
```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "result": "0x0"
}
```
让我们来发一笔交易并获取回执，来看看余额的变化吧。
```bash
$rpc sendRawTransaction --code "0x606060405260008055341561001357600080fd5b60f2806100216000396000f3006060604052600436106053576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff1680634f2be91f1460585780636d4ce63c14606a578063d826f88f146090575b600080fd5b3415606257600080fd5b606860a2565b005b3415607457600080fd5b607a60b4565b6040518082815260200191505060405180910390f35b3415609a57600080fd5b60a060bd565b005b60016000808282540192505081905550565b60008054905090565b600080819055505600a165627a7a72305820906dc3fa7444ee6bea2e59c94fe33064e84166909760c82401f65dfecbd307d50029" --private-key "0x5f0258a4778057a8a7d97809bd209055b2fbafa654ce7d31ec7191066b9225e6" --address "0x4b5ae4567ad5d9fb92bc9afd6a657e6fa13a2523"
```
```bash
$rpc getTransactionReceipt --hash "0x39c4cd332892fb5db11c250275b9a130bf3c087ebdf47b6504d65347ec349406"
```
输出：
```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "result": {
    "blockHash": "0x418784f82250fb7d91dbe79955a366d73adb260fa73476c23484227f04eb21fa",
    "blockNumber": "0x1a",
    "contractAddress": null,
    "cumulativeGasUsed": "0x64",
    "errorMessage": null,
    "gasUsed": "0x64",
    "logs": [
    ],
    "logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
    "root": null,
    "transactionHash": "0x39c4cd332892fb5db11c250275b9a130bf3c087ebdf47b6504d65347ec349406",
    "transactionIndex": "0x0"
  }
}
```
再来查一下余额：
```bash
$rpc getBalance --address "0x4b5ae4567ad5d9fb92bc9afd6a657e6fa13a2523"
```
输出：
```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "result": "0xffffffffffffffffffffffff9b"
}
```

```bash
$rpc getBalance --address "0x36a60d575b0dee0423abb6a57dbc6ca60bf47545"
```
输出：
```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "result": "0x64"
}
```
可以看到余额发生了变化，运营方的账户余额从0变成了64， 查看交易回执中的 (QuotaUsed)gasUsed 同样为64。

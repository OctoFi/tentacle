# Tentacle

```Tentacle is currently being refactored.```

```The next release will be Tentacle 0.5.```

```Current functionality will not be affected / All current contracts will operate as normal, but new ones will be deployed that are hopefully better and do more.```

```What's coming? Code will be cleaner, tests will be at the core, Ganache and mainnet forks will be documented, and a newer Solidity compiler will be used. ```


### Homemade high-reliability oracle aggregator for Ethereum-based DeFi apps that need on and off-chain data, with a bit of streamlined execution functionality in there, as well.


![Tentacle Logo](https://octo.fi/images/projects/project-tentacle.jpg)


Website: [octo.fi](https://www.octo.fi)

## Demo soon

![Try out](https://octo.fi/images/projects/project-tentacle.jpg)

## Getting Started

At the top of your smart contract or in a referenced file in your dApp project, include this interface.

If you're using Solidity 0.5.0+:

```javascript
pragma experimental ABIEncoderV2;

interface TentacleInterface {
  function getExchangeRate ( string calldata fromSymbol, string calldata  toSymbol, string calldata venue, uint256 amount ) external view returns ( uint256 );
  function getTokenDecimalCount ( address tokenAddress ) external view returns ( uint256 );
  function getTokenAddress ( string calldata  symbol ) external view returns ( address );
  function getSynthBytes32 ( string calldata  symbol ) external view returns ( bytes32 );
  function getForexAddress ( string calldata symbol ) external view returns ( address );
  function arb(address  fundsReturnToAddress,  address liquidityProviderContractAddress, string[] calldata   tokens,  uint256 amount, string[] calldata  exchanges) external payable returns (bool);
}
```

Under Solidity 0.5.0:


```javascript
interface TentacleInterface {
  function getExchangeRate ( string fromSymbol, string toSymbol, string venue, uint256 amount ) external view returns ( uint256 );
  function getTokenDecimalCount ( address tokenAddress ) external view returns ( uint256 );
  function getTokenAddress ( string symbol ) external view returns ( address );
  function getSynthBytes32 ( string symbol ) external view returns ( bytes32 );
  function getForexAddress ( string symbol ) external view returns ( address );
  function requestAsyncEvent(string eventName, string source)  external returns(string);
  function getAsyncEventResult(string eventName, string source, string referenceId) external view returns (string);
  function arb(address fundsReturnToAddress, address liquidityProviderContractAddress, string[] tokens,  uint256 amount, string[] exchanges) payable returns (bool);
}
```



To Initialize Tentacle on mainnet, simply include this code:

```javascript
TentacleInterface tentacle= TentacleInterface(0x);

```

Ganache and Truffle documentation coming soon.


One of the best things about Tentacle is that Tentacle automatically detects which kind of asset you are looking for (though the data can come from different providers), as the parameter of "venue" when making the getExchangeRate call. For example, you can get the price for ETH/USD the same way you get the price for JPY/ETH. The 3rd parameter is the venue. Use blank ('') for default oracle. In the future, you can reference several venues/providers to get their data and throw out any that deviate too far from the average.

```javascript
uint jpyusdPrice = tentacle.getExchangeRate("JPY", "USD", "DEFAULT", 100000);
// returns 920 (or $920.00)
```

Note: Replace "DEFAULT" with the oracle provider you would like data from. For example, if you want to know Uniswap's price on the buy side, use "BUY-UNISWAP-EXCHANGE". If you want Kyber's sell side data for the same, you can use "SELL-KYBER-EXCHANGE". Due to the way Bancor works with swaps/liquidity paths, you can simply use "BANCOR" when querying Bancor. Because ERC-20s have many, many integers, when getting prices from token to token, be sure to use very large amounts.... 1000000000 DAI is less than one penny, for example, due to divisibility at 18.

More examples:

```javascript
uint price = tentacle.getExchangeRate("ETH", "USDC", "UNISWAPBYSYMBOLV1", 100000000000000);
```

Supports Uniswap v.2 as follows:

```javascript
uint price = tentacle.getExchangeRate("ETH", "USDC", "UNISWAPBYSYMBOLV2", 100000000000000);
```


```javascript
uint price = tentacle.getExchangeRate("BTC", "DAI", "SELL-UNISWAP-EXCHANGE", 100);
```

```javascript
uint price = tentacle.getExchangeRate("ETH", "DAI", "BANCOR", 1000000000000000);
```

```javascript
uint price = tentacle.getExchangeRate("MKR", "EUR", "", 100000000000000);
```


Experimental:


```javascript
uint price = tentacle.getExchangeRate("AAPL", "USD", "PROVIDER1", 1);
```


Additionally, you can do hacky things like retrieve a "safe" gas price that prevents front-running within your dApp by querying Synthetix's `gasPriceLimit`.


```javascript
uint gasLimit = tentacle.getExchangeRate("skip", "skip", "synthetix-gas-price-limit", 0);
```


## Data And Event Oracles

In addition to pricing and numerical data, string data can also be retrieved using the get `getEventResult` method. You can get string data from registered Tentacle oracles (who can optionally leave notes about how their oracles work and other details). This can be used for sporting events, documents, and notes that one might want to store permanently/temprarily with an expiration for when aliens come and want data on what the human were up to.
```javascript
string memory info = tentacle.getEventResult("skip", "satoshi-first-block");
```
Returns: The Times 03/Jan/2009 Chancellor on brink of second bailout for banks



## RESTful API

You can access the getExchangeRate functionality via RESTful API calls. e.g.
```javascript
https://api.octo.fi/getExchangeRate?fromSymbol=JPY&toSymbol=USD&venue=DEFAULT&amount=10000000000000000  (Soon Live)
```

More of Tentacle's smart contract functionality will be added to RESTful calls soon. You can find the source code for the Node.js API app in /nodeJSAppExamples/Tentacleapi



## Providing Data As An Oracle Provider

You can register a provider name and connect it to your custom oracle contract (DNS-style) via the Tentacle Oracle Registry by calling the registerOracle function.
Additionally, you can tranfer the oracle name, provide contact details in case you are considering selling it, and discover other oracle providers via the smart contract.
An example of an oracle smart contract that will be compatible with the Tentacle proxy contract is available in /contracts/examples/ProvideDataExamples/userGeneratedOracleExample.sol (very simple example that either returns 500 or 2)
Once you deploy your contract and register it to the registry (paying a small amount of ETH to prevent spamming of names), you can check/verify your registration by calling the getOracleAddress function.

As more reputable, as well as trustless, oracle smart contracts register within the Tentacle registry, we will update a new list as a reference.


## Source and Asset Examples (Currently on Main-net)


| Asset       | Example Provider (Venue)           | Type  |
| ------------- |:-------------:| -----:|
| ETH      | DEFAULT | Cryptocurrency |
| BTC      | DEFAULT | Cryptocurrency |
| DAI      | KYBERBYSYMBOLV1      |   Token |
| USDC | UNISWAPBYSYMBOLV1   |    Token |
| LINK | UNISWAPBYSYMBOLV2   |    Token |
| MKR      | BANCOR | Token |
| KNC      | DEFAULT      |   Token |
| ZRX | DEFAULT    |    Token |
| TUSD | DEFAULT    |    Token |
| SNX | DEFAULT    |    Token |
| CUSDC | DEFAULT    |    Token |
| BAT | DEFAULT    |    Token |
| OMG | DEFAULT    |    Token |
| SAI | DEFAULT    |    Token |
| JPY | DEFAULT    |    Forex |
| EUR | DEFAULT    |    Forex |
| CHF | DEFAULT    |    Forex |
| USD | DEFAULT    |    Forex |
| GBP | DEFAULT    |    Forex |
| AAPL | PROVIDER1    |    Equity |
| MSFT | PROVIDER1    |    Equity |
| GOOGL | PROVIDER1    |    Equity |
| NFLX | PROVIDER1    |    Equity |
| BRK.A | PROVIDER1    |    Equity |
| FB | PROVIDER1    |    Equity |
| BABA | PROVIDER1    |    Equity |
| V | PROVIDER1    |    Equity |
| JNJ | PROVIDER1    |    Equity |
| TSLA | PROVIDER1    |    Equity |
| JPM | PROVIDER1    |    Equity |
| DIS | PROVIDER1    |    Equity |
| SPX | PROVIDER1    |    ETF |
| VOO | PROVIDER1    |    ETF |
| QQQ | PROVIDER1    |    ETF |
| GLD | PROVIDER1    |    ETF |
| VXX | PROVIDER1    |    ETF |



The top 20 ERC-20 tokens are available.

contracts/pegTokenExample.sol contains a template code and live contract reference for a token using Tentacle data that is pegged to the value of an off-chain asset (Alibaba Stock in the example). We are looking forward to less primitive examples that leverage DAOs, advanced collateralization techniques, etc. Also, contracts/levFacility.sol is in very early stages and is the begining of creating a token that has a built-in leveraged short/long credit facility for margin trading of futures settled by Tentacle data (very early).


## Examples

The contracts/examples folder contains contracts for both writing data as an oracle provider and for consuming data as an oracle consumer.

The /nodeJSAppExamples folder contains Node.js apps that interface with smart contracts that either read or write oracle data



## Getting Data From [Chainlink](https://chain.link/) via Tentacle

You can retrieve data from a website (off-chain) asynchronously via the Chainlink integration. (coming soon)


Now you are ready!

```javascript
string status = tentacle.requestAsyncEvent("https://min-api.cryptocompare.com/data/price?fsym=ETH&tsyms=USD", "CHAINLINK");
```

After 1 to 3 blocks, Chainlink will send the website data to Tentacle and you can access that data without making a transaction (synchronously). Additionally, you can access data from websites that others have already paid for by inputting their the URL.

```javascript
string result = tentacle.getAsyncEventResult("https://min-api.cryptocompare.com/data/price?fsym=ETH&tsyms=USD", "CHAINLINK", "");
```

Similar integrations with Augur, Provable and Band Protocol are coming soon.

Once your transaction has been confirmed on the blockchain, Chainlink then waits 1-3 blocks and sends the response from their smart contract.

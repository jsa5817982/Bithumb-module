# Bithumb-module
Bithumb module for import exchange on gekko-trading-bot 

Node Bithumb API
This project is designed to help you make your own projects that interact with the Bithumb API. You can stream candlestick chart data, market depth, or use other advanced features such as setting stop losses and iceberg orders. This project seeks to have complete API coverage including WebSockets.


Installation


npm install node-bithumb-api --save


Getting started


const bithumb = require('node-bithumb-api')().options({
  APIKEY: '<key>',
  APISECRET: '<secret>',
  useServerTime: true // If you get timestamp errors, synchronize to server time at startup
});
  
  
Getting latest price of a symbol


bithumb.prices('KRW', (error, ticker) => {
  console.log("Price of KRW: ", ticker.KRW);
});


Getting latest price of all symbols
bithumb.prices((error, ticker) => {
  console.log("prices()", ticker);
  console.log("Price of BTC: ", ticker.BTCUSDT);
});


View Response
Getting list of current balances


bithumb.balance((error, balances) => {
  if ( error ) return console.error(error);
  console.log("balances()", balances);
  console.log("ETH balance: ", balances.ETH.available);
});


// If you have problems with this function,
// see Troubleshooting at the bottom of this page.


View Response
Getting bid/ask prices for a symbol


bithumb.bookTickers('KRW', (error, ticker) => {
  console.log("bookTickers", ticker);
});


View Response
Getting bid/ask prices for all symbols


bithumb.bookTickers((error, ticker) => {
  console.log("bookTickers()", ticker);
  console.log("Price of KRW: ", ticker.KRW);
});


View Response
Get all bid/ask prices


bithumb.bookTickers((error, ticker) => {
  console.log("bookTickers", ticker);
});


View Response
Get market depth for a symbol


bithumb.depth("KRW", (error, depth, symbol) => {
  console.log(symbol+" market depth", depth);
});


View Response
Placing a LIMIT order


var quantity = 1, price = 0.069;
bithumb.buy("ETHBTC", quantity, price);
bithumb.sell("ETHBTC", quantity, price);


Placing a MARKET order
// These orders will be executed at current market price.

var quantity = 1;
bithumb.marketBuy("KRW", quantity);
bithumb.marketSell("ETHBTC", quantity);


LIMIT order with callback


var quantity = 5, price = 0.00402030;
bithumb.buy("KRWETH", quantity, price, {type:'LIMIT'}, (error, response) => {
  console.log("Limit Buy response", response);
  console.log("order id: " + response.orderId);
});


View Response
Chaining orders together


var quantity = 1;
bithumb.marketBuy("KRW", quantity, (error, response) => {
  console.log("Market Buy response", response);
  console.log("order id: " + response.orderId);
  // Now you can limit sell with a stop loss, etc.
});


View Response
Placing a STOP LOSS order


// When the stop is reached, a stop order becomes a market order
// Note: You must also pass one of these type parameters:
// STOP_LOSS, STOP_LOSS_LIMIT, TAKE_PROFIT, TAKE_PROFIT_LIMIT
let type = "STOP_LOSS";
let quantity = 1;
let price = 0.069;
let stopPrice = 0.068;
bithumb.sell("ETHBTC", quantity, price, {stopPrice: stopPrice, type: type});


Placing an ICEBERG order
// Iceberg orders are intended to conceal the order quantity.
var quantity = 1;
var price = 0.069;
bithumb.sell("ETHBTC", quantity, price, {icebergQty: 10});


Cancel an order

bithumb.cancel("ETHBTC", orderid, (error, response, symbol) => {
  console.log(symbol+" cancel response:", response);
});


Cancel all open orders


bithumb.cancelOrders("XMRBTC", (error, response, symbol) => {
  console.log(symbol+" cancel response:", response);
});


Get open orders for a symbol


bithumb.openOrders("ETHBTC", (error, openOrders, symbol) => {
  console.log("openOrders("+symbol+")", openOrders);
});
Get list of all open orders
bithumb.openOrders(false, (error, openOrders) => {
  console.log("openOrders()", openOrders);
});


Check an order's status


let orderid = "7610385";
bithumb.orderStatus("ETHBTC", orderid, (error, orderStatus, symbol) => {
  console.log(symbol+" order status:", orderStatus);
});


Trade history


bithumb.trades("SNMBTC", (error, trades, symbol) => {
  console.log(symbol+" trade history", trades);
});


View Response
Get all account orders; active, canceled, or filled.


bithumb.allOrders("ETHBTC", (error, orders, symbol) => {
  console.log(symbol+" orders:", orders);
});


Get 24hr ticker price change statistics for all symbols


bithumb.prevDay(false, (error, prevDay) => {
  // console.log(prevDay); // view all data
  for ( let obj of prevDay ) {
    let symbol = obj.symbol;
    console.log(symbol+" volume:"+obj.volume+" change: "+obj.priceChangePercent+"%");
  }
});


Get 24hr ticker price change statistics for a symbol


bithumb.prevDay("KRW", (error, prevDay, symbol) => {
  console.log(symbol+" previous day:", prevDay);
  console.log("KRW change since yesterday: "+prevDay.priceChangePercent+"%")
});


Get Kline/candlestick data for a symbol
You can use the optional API parameters for getting historical candlesticks, these are useful if you want to import data from earlier back in time. Optional parameters: limit (max/default 500), startTime, endTime.


// Intervals: 1m,3m,5m,15m,30m,1h,2h,4h,6h,8h,12h,1d,3d,1w,1M
bithumb.candlesticks("KRW", "5m", (error, ticks, symbol) => {
  console.log("candlesticks()", ticks);
  let last_tick = ticks[ticks.length - 1];
  let [time, open, high, low, close, volume, closeTime, assetVolume, trades, buyBaseVolume, buyAssetVolume, ignored] = last_tick;
  console.log(symbol+" last close: "+close);
}, {limit: 500, endTime: 1514764800000});


WebSockets Implementation
Get Complete WebSocket Chart Cache
This function pulls existing chart data before connecting to the WebSocket, and provides you realtime synchronized chart information including the most recent 500 candles.


bithumb.websockets.chart("KRW", "1m", (symbol, interval, chart) => {
  let tick = bithumb.last(chart);
  const last = chart[tick].close;
  console.log(chart);
  // Optionally convert 'chart' object to array:
  // let ohlc = bithumb.ohlc(chart);
  // console.log(symbol, ohlc);
  console.log(symbol+" last price: "+last)
});


View Response
Get Candlestick Updates via WebSocket


// Periods: 1m,3m,5m,15m,30m,1h,2h,4h,6h,8h,12h,1d,3d,1w,1M
bithumb.websockets.candlesticks(['KRW'], "1m", (candlesticks) => {
  let { e:eventType, E:eventTime, s:symbol, k:ticks } = candlesticks;
  let { o:open, h:high, l:low, c:close, v:volume, n:trades, i:interval, x:isFinal, q:quoteVolume, V:buyVolume, Q:quoteBuyVolume } = ticks;
  console.log(symbol+" "+interval+" candlestick update");
  console.log("open: "+open);
  console.log("high: "+high);
  console.log("low: "+low);
  console.log("close: "+close);
  console.log("volume: "+volume);
  console.log("isFinal: "+isFinal);
});


Get Trade Updates via WebSocket


bithumb.websockets.trades(['KRW', 'ETHBTC'], (trades) => {
  let {e:eventType, E:eventTime, s:symbol, p:price, q:quantity, m:maker, a:tradeId} = trades;
  console.log(symbol+" trade update. price: "+price+", quantity: "+quantity+", maker: "+maker);
});


Get miniTicker via WebSocket


bithumb.websockets.miniTicker(markets => {
  console.log(markets);
});


View Response
Get 24hr Price Change Statistics via WebSocket


// For all symbols:
bithumb.websockets.prevDay(false, (error, response) => {
  console.log(response);
});

// For a specific symbol:
bithumb.websockets.prevDay('KRW', (error, response) => {
  console.log(response);
});


View Response
Get Market Depth via WebSocket


bithumb.websockets.depth(['KRW'], (depth) => {
  let {e:eventType, E:eventTime, s:symbol, u:updateId, b:bidDepth, a:askDepth} = depth;
  console.log(symbol+" market depth update");
  console.log(bidDepth, askDepth);
});


Maintain Market Depth Cache Locally via WebSocket


bithumb.websockets.depthCache(['KRW'], (symbol, depth) => {
  let bids = bithumb.sortBids(depth.bids);
  let asks = bithumb.sortAsks(depth.asks);
  console.log(symbol+" depth cache update");
  console.log("bids", bids);
  console.log("asks", asks);
  console.log("best bid: "+bithumb.first(bids));
  console.log("best ask: "+bithumb.first(asks));
});


View Response
Deposit & Withdraw
Get Deposit Address


bithumb.depositAddress("XMR", (error, response) => {
  console.log(response);
});


Get All Deposit History


bithumb.depositHistory((error, response) => {
  console.log(response);
});


Get Deposit History for a specific symbol
bithumb.depositHistory((error, response) => {
  console.log(response);
}, "VEN");
Get All Withdraw History
bithumb.withdrawHistory((error, response) => {
  console.log(response);
});


Get Withdraw History for a specific symbol


bithumb.withdrawHistory((error, response) => {
  console.log(response);
}, "BTC");


Withdraw with AddressTag

// Required for coins like XMR, XRP, etc.
let address = "44tLjmXrQNrWJ5NBsEj2R77ZBEgDa3fEe9GLpSf2FRmhexPvfYDUAB7EXX1Hdb3aMQ9FLqdJ56yaAhiXoRsceGJCRS3Jxkn";
let addressTag = "0e5e38a01058dbf64e53a4333a5acf98e0d5feb8e523d32e3186c664a9c762c1";
let amount = 0.1;
bithumb.withdraw("XMR", address, amount, addressTag);
Withdraw with Callback
bithumb.withdraw("ETH", "0x1d2034348c851ea29c7d03731c7968a5bcc91564", 1, false, (error, response) => {
  console.log(response);
});


Withdraw


bithumb.withdraw("BTC", "1C5gqLRs96Xq4V2ZZAR1347yUCpHie7sa", 0.2);


Advanced Examples

exchangeInfo: Pull minimum order size, quantity, etc
Clamp order quantities to required amounts via minQty, minNotional, stepSize when placing orders
Show API Rate limits
Connect to all WebSockets at once
Get last order for a symbol
newOrderRespType example


Recent Trades (historicalTrades, recentTrades, aggTrades functions)
Terminate WebSocket connections
User Data: Account Balance Updates, Trade Updates, New Orders, Filled Orders, Cancelled Orders via WebSocket
Proxy Support
For the standard REST API the https_proxy or socks_proxy variable is honoured NOTE proxy package has no dns name support, please use proxy IP address


Linux

export https_proxy=http://ip:port
#export socks_proxy=socks://ip:port
# run your app
Windows
set https_proxy=http://ip:port
#set socks_proxy=socks://ip:port
# run your app
For web sockets currently only the socks method is functional at this time


linux
export socks_proxy=socks://ip:port
# run your app


windows
set socks_proxy=socks://ip:port
# run your app


Troubleshooting

Verify that your system time is correct. If you have any suggestions don't hestitate to file an issue.
Having problems? Try adding useServerTime to your options or setting recvWindow:

bithumb.options({
  APIKEY: 'xxx',
  APISECRET: 'xxx',
  useServerTime: true,
  recvWindow: 60000, // Set a higher recvWindow to increase response timeout
  verbose: true, // Add extra output when subscribing to WebSockets, etc
  log: log => {
    console.log(log); // You can create your own logger here, or disable console output
  }
});


Problems getting your balance? Wrap the entry point of your application in useServerTime:
bithumb.useServerTime(function() {
	bithumb.balance((error, balances) => {
		if ( error ) return console.error(error);
		console.log("balances()", balances);
		console.log("KRW balance: ", balances.KRW.available);
	});
});
 
Thank you to all contributors and others!


# coinflex-api
coinflex api
# 根路径 未带有@Headers，默认为({"User-Agent:android", "Content-Type:application/json", "Accept:application/json"})  
```
//    String RootUrl = "https://webapi.coinflex.com/";//正式环境  
    String RootUrl = "https://stgwebapi.coinflex.com/";//测试环境  
```
> ## 1.取临时最新价以及最高最低价，买卖一价。实时最新价根据websocket的 "tonce":"orderMatched" 的 json串中的 <b>last</b> 值  
```
//                      "last": 66970000,//当前最新价  
//                      "bid": 66240000,//当前买一价  
//                      "ask": 66290000,//当前卖一价  
//                      "low": 61600000,//24小时内最低价  
//                      "high": 67370000,//24小时内最高价  
//                      "volume": 868834,//24小时交易量  
TikerBean：{
                    "base": 63488,
                    "counter": 65283,
                    "name": "XBT/USDT",
                    "spot_name": "XBT/USDT",
                    "last": 66970000,
                    "bid": 66240000,
                    "ask": 66290000,
                    "low": 61600000,
                    "high": 67370000,
                    "volume": 868834,
                    "time": 1585823609052186013
                }  
@GET(RootUrl+"tickers/")  
Call<List<TickerBean>> getTickers();  
```
> ## 2.获取market中的交易对信息  
```
//tick：下订单的最小刻度  假如tick=2500；而<3.Assets接口>中scale的是10000，那么下限价单报价就必须是0.25的整数倍  
MarketBean:{
                 "base": 63488,
                 "counter": 65283,
                 "name": "XBT/USDT",
                 "spot_name": "XBT/USDT",
                 "tick": 10000 
             }  
//取名称  base  counter  
@GET(RootUrl+"markets/")  
Call<List<MarketBean>> getMarkets();  
```
> ## 3.Assets接口  
```
//如果缺少spot_id、spot_name就是现货，带有这2个字段的是合约  
AssetsBean:{
                   "id": 51202,
                   "name": "XBTMAR",
                   "spot_id": 63488,
                   "spot_name": "XBT",
                   "scale": 10000
               }  
//取scale  缩放比例  
@GET(RootUrl+"assets/")  
Call<List<AssetsBean>> getScale();  
```
> ## 4.获取某交易对24小时前这一刻的价格  
```
//这个接口是第三方的。相对于我们平台，注意价格的值是缩放10000后的  
After24HPairBean:{
                                     "base": 63488,
                                     "counter": 65283,
                                     "price": 8834.8
                                 }  
//获取24小时前的价格 路径不再是RootUrl开头  
@GET("https://marketapi.coinflex.com/buckets/change")  
Call<BaseBean<List<After24HPairBean>>> getPriceAfter_24h();  
```
#以上1、2、3、4 这4个接口组合得到market的 涨跌幅、最新价、交易量  

> ## 5.深度表接口 即 买卖盘的数据  
```
DepthBean:{
              "bids": [
                  [
                      57000000,
                      1315
                  ],
                  [
                      55000000,
                      74
                  ]
              ],
              "asks": [
                  [
                      72000000,
                      5000
                  ],
                  [
                      74000000,
                      1193
                  ]
              ]
          }
//由于我们正式线上平台深度表数据不准确   可以考虑轮训请求这个接口,  
@GET(RootUrl+"depth/{base}:{counter}")  
Call<DepthBean> getDepth(@Path("base") int base, @Path("counter") int counter);  
```
 #跟用户相关的接口  凡是@Header("authorization")String str就是传 "Basic "+Base64.encodeToString(core_id/api_key:priv_key，Base64.NO_WRAP) Base64必须是Base64.NO_WRAP(去掉加盐后字符串中的\n)  
> ## 6.获取历史订单记录  
```
//此接口没有按交易对查询的方式  只有时间戳、排序方式、条目  
//https://webapi.coinflex.com/trades/?since=1580129123000000&until=1582980323000000&sort=desc&limit=50  
@GET(RootUrl+"trades/")  
Call<List<OrderHistoryBean>> getOrdersHistory(@Header("authorization")String str);  
```
> ## 7.获取余额  
```
//id是资产id  
BalanceBean:{
        id : 51907,
        available : 100000000000000000000,
        reserved : 10000000000000000000
     }  
@GET(RootUrl+"balances/{id}")  
Call<BalanceBean> getBalances(@Header("authorization")String str,@Path("id") String base);  
@GET(RootUrl+"balances/")  
Call<List<BalanceBean>> getAllBalances(@Header("authorization")String str);  
```
> ## 8.获取抵押品列表  
```
CollateralBean:{
         asset_id : 65285,
         available : 3781533926,
         total : 3781533926
        }  
@GET(RootUrl+"borrower/collateral/")  
Call<List<CollateralBean>> getCollateral(@Header("authorization")String str);  
```
> ## 9.获取借杠杆的详情  
```
//根据asset_id将principal的值累加起来就是   这asset_id总借的钱  
//initial_margin 起始保证金率(value/scale) scale是<3.Assets接口>的scale，例：250/10000  
//maintenance_margin 维持保证金率 同上  
LoanBean:{
                 "id": 3710,
                 "offer_id": 2690,
                 "initiated": 1585825783660721,
                 "asset_id": 10053,
                 "principal": 1.487069E10,
                 "term": 7430393,
                 "initial_margin": 250,
                 "maintenance_margin": 200,
                 "apr": 0
             }  
@GET(RootUrl+"borrower/loans/")  
Call<List<LoanBean>> getAllLoans(@Header("authorization")String str);  
```
> ## 10.获取持仓数详情  
```
//asset字段 对应<3.Assets接口>的id  
PositionBean:{
                 asset : 63488,
                 position : -5472811,
                 through : 1585058026335900
             }  
@GET(RootUrl+"positions/")  
Call<List<PositionBean>> getPositions(@Header("authorization")String str);  
```
> ## 11.提供杠杆的详情  
```
//amount 暂时未知作用  
//min_amount 平台要求最小借多少，借杠杆传比这个值更小的时候，请求不会成功  
//max_amount 平台提供最多可借，借杠杆传比这个值大的时候，请求不会成功  
//initial_margin 同借杠杆详情接口的字段  
//maintenance_margin 同借杠杆详情接口的字段  
LeverageBean:{
                     "id": 2701,
                     "created": 1585828807742968,
                     "asset_id": 10029,
                     "amount": 10000000000,
                     "min_amount": 100,
                     "max_amount": 100000000,
                     "term": 7343993,
                     "initial_margin": 250,
                     "maintenance_margin": 200,
                     "apr": 0
                 }  
@GET(RootUrl+"borrower/offers/")  
Call<List<LeverageBean>> getLeverageInfo(@Header("authorization")String str);  
```
> ## 12.借杠杆  （注意Content-Type 以及 参数提交方式，id为 <11.提供杠杆的详情接口> 中id值  
```
LeverageBean:同<11.提供杠杆的详情接口>  
@FormUrlEncoded()  
@POST(RootUrl+"borrower/loans/")  
@Headers({"User-Agent:android", "Content-Type:application/x-www-form-urlencoded", "Accept:application/json"})  
Call<LeverageBean> postAddLeverage(@Header("authorization")String str,@Field("offer_id") Integer id,@Field("amount") Long amount);  
```
> ## 13.还杠杆  （注意Content-Type 以及 参数提交方式，id为 <9.获取借杠杆的详情接口> 中id值, 请求成功无任何数据返回  
```
@FormUrlEncoded  
@POST(RootUrl+"borrower/loans/{id}")  
@Headers({"User-Agent:android", "Content-Type:application/x-www-form-urlencoded", "Accept:application/json"})  
//这里有坑就是服务器什么都没有返回  不能是string  不能是object  
Call<ResponseBody> postRepayLeverage(@Header("authorization")String str, @Path("id") int loan_id, @Field("amount") long amount);  
```
> ## 14.还清杠杆  （注意请求为Delete方式，Content-Type 以及 参数提交方式，id为 <9.获取借杠杆的详情接口> 中id值。注意Delete方式是不带Body的 请求成功无任何数据返回  
```
@DELETE(RootUrl+"borrower/loans/{id}")  
@Headers({"User-Agent:android", "Content-Type:application/x-www-form-urlencoded", "Accept:application/json"})  
//这里有坑就是服务器什么都没有返回  不能是string  不能是object  
Call<ResponseBody> deleteLeverage(@Header("authorization")String str,@Path("id") int loan_id);  
```
> ## 15.转换记录 现货->合约   合约->现货  总的  
```
ConvertedBean:{
                      "asset_from": 63488,
                      "asset_to": 51200,
                      "total": 3710000,
                      "total_from": 3710000,
                      "total_to": 3710000
                  }                  
@GET(RootUrl+"borrower/converted_totals/")  
Call<List<CollateralBean>> getConvertedTotals(@Header("authorization")String str);  
```
# 合约相关的数据3、7、8、9、10、11、15接口组合   按<3.Assets接口>的id匹配  

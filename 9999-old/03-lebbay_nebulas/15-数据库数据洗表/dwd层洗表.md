[toc]

> 订单：https://wiki.lebbay.com/pages/viewpage.action?pageId=5734611



## 01, 订单表分析所有字段并清洗不需要邮箱数据

```mysql
select 
-- 订单id，重要
oi.order_id,
-- az订单均为一个值, 这里不需要
-- oi.party_id,
-- 订单sn，重要
oi.order_sn,
-- 用户ID，重要
oi.user_id, 
-- 订单时间，重要
oi.order_time, 
-- 订单状态，0未付款，1付款中，2已付款，3待退款，4已退款，重要
oi.order_status, 
-- 运输状态
oi.shipping_status,
-- 支付状态，0未确认，1已确认，2已取消，重要
oi.pay_status,
-- 收货人
oi.consignee, 
-- 性别，enum('male','female','unknown') 
oi.gender,
-- 签收地址国家id
oi.country,
-- 签收地址省id
oi.province,
-- 签收地址省
oi.province_text,
-- 签收地城市id，这个好像大部分都是0，有几个是4253，这个字段应该无用
oi.city,
-- 签收地城市
oi.city_text,
-- 这个全部是0，无用字段，县/区；暂时无用
-- oi.district,
-- 这个字段很多都没填，但是有几个有，不知道有没有用， 县/区；暂时无用
-- oi.district_text,
-- 详细地址
oi.address,
-- 邮编
oi.zipcode,
-- 电话
oi.tel,
-- 手机, 这个字段大多数没有填写
oi.mobile,
-- 邮箱, 重要
oi.email,
-- 这个字段没有
-- oi.best_time,
-- 这个后续肯定用不到了
-- oi.sign_building,
-- 订单附言, 后续用不到
-- oi.postscript,
-- 这个估计也用不上
-- oi.important_day,
-- shipping method id，运输方法id，目前az都为2
oi.sm_id,
-- 运输方式id，目前az都为91
oi.shipping_id,
oi.shipping_name,
-- 支付方式id
oi.payment_id,
-- 支付方式
oi.payment_name,


-- ---------以下暂时不知道做什么的，先问一下---------
oi.how_oos,
oi.how_surplus,
oi.pack_name,
oi.card_name,
oi.card_message,
oi.inv_payee,
oi.inv_content,
-- 发票寄送地址
oi.inv_address,
-- 发票寄送地址邮编
oi.inv_zipcode,
-- 发票到的电话
oi.inv_phone,
-- ---------以上先不管---------


-- 商品总价(按基础货币base_currency_id)
oi.goods_amount,
-- 商品转换后的数额，商品总价(按订单支付货币order_currency_id)
oi.goods_amount_exchange,
-- 运费(按基础货币base_currency_id)
oi.shipping_fee,
-- 税金金额，税费(按基础货币base_currency_id)
oi.duty_fee,
-- 运费(按订单支付货币order_currency_id)
oi.shipping_fee_exchange,
-- 税金交易金额，税费(按订单支付货币order_currency_id)
oi.duty_fee_exchange,


-- ---------------以下----------------------
oi.insure_fee,
oi.shipping_proxy_fee,
oi.payment_fee,
oi.pack_fee,
oi.card_fee,
oi.money_paid,
oi.surplus,
-- 已经抵用欧币
oi.integral,
oi.integral_money,
-- ---------------以上----------------------


-- 红包抵扣(按基础货币base_currency_id)
oi.bonus,
-- 红包抵扣(按订单支付货币order_currency_id)
oi.bonus_exchange,
-- 订单总额(按基础货币base_currency_id)
oi.order_amount,

-- 基础货币id, 1:美元, 3:加拿大元
oi.base_currency_id,
-- 该订单支付货币id,1:美元, 3:加拿大元
oi.order_currency_id,
-- 该订单支付货币符号 US$ HK$
oi.order_currency_symbol,
-- 汇率(order_currency/base_currency)
oi.rate,
-- 订单总额(按订单支付货币order_currency_id) 用户当地币种下订单对应的总额
oi.order_amount_exchange,


-- ---------------以下----------------------
-- from_ad
oi.from_ad,
oi.referer,
-- 这个全部是0，应该无用
oi.confirm_time,
-- ---------------以上----------------------



oi.pay_time,
-- 下面物流相关的估计也用不到
oi.shipping_time,



-- ---------------以下----------------------
-- 预计发货日期
oi.shipping_date_estimate,
oi.shipping_carrier,
-- 货运单号
oi.shipping_tracking_number,
-- 全部是0，问一下是否用了c
oi.pack_id,
oi.card_id,
-- ---------------以上----------------------



-- 优惠券代码
oi.coupon_code,



-- ---------------以下----------------------
oi.invoice_no,
oi.extension_code,
oi.extension_id,
oi.to_buyer,
oi.pay_note,
oi.invoice_status,
oi.carrier_bill_id,
oi.receiving_time,
oi.biaoju_store_id,
oi.parent_order_id,
-- ---------------以上----------------------




oi.track_id, 
-- 这个是获取通道信息的，这里先保留
oi.ga_track_id,


-- ---------------以下----------------------
oi.real_paid,
oi.real_shipping_fee,
oi.is_shipping_fee_clear,
oi.is_order_amount_clear,
oi.is_ship_emailed,
-- 代收货款费用
oi.proxy_amount,
oi.pay_method,
-- 是否从快递公司追回已发送货物
oi.is_back, 
-- 财务是否清算
oi.is_finance_clear,
-- 财务清算类型
oi.finance_clear_type,
-- 处理时间，该时间点前的订单不显示在待配货页面
oi.handle_time,
-- 起始快递时间，小时
oi.start_shipping_time,
-- 结束快递时间，小时
oi.end_shipping_time,
-- 缺货状态：0有货；1暂缺货；2请等待；3已到货；4取消
oi.shortage_status,
-- 缺货等待
oi.is_shortage_await,
oi.order_type_id,
-- 特殊类型定义
oi.special_type_id,
-- 是否显示给客服
oi.is_display,
-- 
oi.misc_fee,
-- -h订单还需要增加的费用
oi.additional_amount,
-- 分销商
oi.distributor_id,
-- 
oi.taobao_order_sn,
-- 分销采购单号
oi.distribution_purchase_order_sn,
-- 是否需要发票
oi.need_invoice,
-- 仓库id
oi.facility_id,
oi.language_id,
oi.coupon_cat_id,
-- @see ok_coupon_config
oi.coupon_config_value,
-- @see ok_coupon_config
oi.coupon_config_coupon_type,
-- 数据是否已提交给google adwords
oi.is_conversion,
-- ---------------以上----------------------



-- 订单来源
oi.from_domain,
oi.project_name,


-- ---------------以下----------------------
-- 下单时的 user agent
oi.user_agent_id,
-- 下单时显示的货币
oi.display_currency_id,
-- 下单时显示的货币汇率
oi.display_currency_rate,
-- 显示用运费
oi.display_shipping_fee_exchange,
-- 税金显示金额
oi.display_duty_fee_exchange,
-- 显示用订单总金额
oi.display_order_amount_exchange,
-- 显示用商品总金额
oi.display_goods_amount_exchange,
-- 显示用coupon费
oi.display_bonus_exchange,
-- paypal express checkout token标记
oi.token,
-- paypal payer id
oi.payer_id,
-- ---------------以上----------------------

-- 订单渠道追踪
oi.ga_track_time,
oi.channel,
-- 报告日期
oi.report_date

from ods_order_info_with_channel
where 1=1
and oi.report_date = '2020-05-01'
and oi.email not like '%@tetx.com'
and oi.email not like '%@i9i8.com'
and oi.email not like '%@azazie.com' 
```



## 02, 订单表选取所需要的字段并清洗不需要邮箱数据

> 删除大部分不怎么用到的字段，字段参考：
>
> https://wiki.lebbay.com/pages/viewpage.action?pageId=5734611

```mysql
select 
-- 订单id，重要
oi.order_id,
-- az订单均为一个值, 这里不需要
-- oi.party_id,
-- 订单sn，重要
oi.order_sn,
-- 用户ID，重要
oi.user_id, 
-- 订单时间，重要
oi.order_time, 
-- 订单状态，0未付款，1付款中，2已付款，3待退款，4已退款，重要
oi.order_status, 
-- 运输状态
oi.shipping_status,
-- 支付状态，0未确认，1已确认，2已取消，重要
oi.pay_status,
-- 收货人
oi.consignee, 
-- 性别，enum('male','female','unknown') 
oi.gender,
-- 签收地址国家id
oi.country,
-- 签收地址省id
oi.province,
-- 签收地址省
oi.province_text,
-- 签收地城市id，这个好像大部分都是0，有几个是4253，这个字段应该无用
oi.city,
-- 签收地城市
oi.city_text,
-- 这个全部是0，无用字段，县/区；暂时无用
-- oi.district,
-- 这个字段很多都没填，但是有几个有，不知道有没有用， 县/区；暂时无用
-- oi.district_text,
-- 详细地址
oi.address,
-- 邮编
oi.zipcode,
-- 电话
oi.tel,
-- 手机, 这个字段大多数没有填写
oi.mobile,
-- 邮箱, 重要
oi.email,
-- shipping method id
oi.sm_id,
-- 运输方式id，目前az都为91
oi.shipping_id,
oi.shipping_name,
-- 支付方式id
oi.payment_id,
-- 支付方式
oi.payment_name,


-- 商品总价(按基础货币base_currency_id)
oi.goods_amount,
-- 商品转换后的数额，商品总价(按订单支付货币order_currency_id)
oi.goods_amount_exchange,
-- 运费(按基础货币base_currency_id)
oi.shipping_fee,
-- 税金金额，税费(按基础货币base_currency_id)
oi.duty_fee,
-- 运费(按订单支付货币order_currency_id)
oi.shipping_fee_exchange,
-- 税金交易金额，税费(按订单支付货币order_currency_id)
oi.duty_fee_exchange,


-- 红包抵扣(按基础货币base_currency_id)
oi.bonus,
-- 红包抵扣(按订单支付货币order_currency_id)
oi.bonus_exchange,
-- 订单总额(按基础货币base_currency_id)
oi.order_amount,

-- 基础货币id, 1:美元, 3:加拿大元
oi.base_currency_id,
-- 该订单支付货币id,1:美元, 3:加拿大元
oi.order_currency_id,
-- 该订单支付货币符号 US$ HK$
oi.order_currency_symbol,
-- 汇率(order_currency/base_currency)
oi.rate,
-- 订单总额(按订单支付货币order_currency_id) 用户当地币种下订单对应的总额
oi.order_amount_exchange,

oi.pay_time,
-- 下面物流相关的估计也用不到
oi.shipping_time,

-- 优惠券代码
oi.coupon_code,

oi.track_id, 
-- 这个是获取通道信息的，这里先保留
oi.ga_track_id,

-- 订单来源
oi.from_domain,
oi.project_name,


-- 订单渠道追踪
oi.ga_track_time,
oi.channel,
-- 报告日期
oi.report_date

from ods_order_info_with_channel as oi
where  1=1
and oi.report_date = '2020-05-01'
and oi.email not like '%@tetx.com'
and oi.email not like '%@i9i8.com'
and oi.email not like '%@azazie.com' 
```





## 03, 订单商品表分析所有字段

```mysql
select 
-- 这个是自增id，重要
og.rec_id,
-- 订单id，重要，因为要和订单关联，这里取一个即可
-- og.order_id,
-- @see goods_style.goods_style_id
og.goods_style_id,
-- @see goods_style.sku
og.sku, 
-- @see goods_sku.sku_id
og.sku_id,
og.goods_id, 
-- 商品名不取这个，取goods_languages里面的
-- og.goods_name,
og.goods_number,
og.market_price,
og.shop_price,
-- 价格转换后的数额
og.shop_price_exchange,
-- 总额的转换后数值
og.shop_price_amount_exchange,
-- 该商品的折扣，负值；可能来自分类折扣或该商品折扣
og.bonus,

---------------以下-----------------
-- 优惠券代码, 这里用不到，在订单层面用
og.coupon_code,
-- 一共3个值，没用
og.goods_attr,
-- 全部是0吗，没用
og.send_number,
og.is_real,
og.extension_code,
og.parent_id,
og.is_gift,
og.goods_status,
og.action_amt,
og.action_reason_cat,
og.action_note,
og.carrier_bill_id,
og.provider_id,
og.invoice_num,
og.return_points,
og.return_bonus,
og.biaoju_store_goods_id,
og.subtitle,
og.addtional_shipping_fee,
og.style_id,
-- 表示移动定制机信息
og.customized,
-- 商品新旧状态
og.status_id,
-- 税率， 现在无
og.added_fee,
-- 自定义尺寸的费用
og.custom_fee,
og.custom_fee_exchange,
-- 大尺码加钱
og.plussize_fee,
og.plussize_fee_exchange,
--------------以上------------------

-- rush order fee，加急费
og.rush_order_fee, 
-- rush order fee，加急费
og.rush_order_fee_exchange,

---------------以下-----------------
og.coupon_goods_id,
og.coupon_cat_id,
-- @see ok_coupon_config
og.coupon_config_value,
-- @see ok_coupon_config
og.coupon_config_coupon_type,
--------------以上------------------



-- 用户选择或输入的各种样式，json；记录备查
og.styles,



----------------以下----------------
-- 默认图类型
og.img_type,
-- 下单时产品显示图
og.goods_gallery,
-- 商品未添加任何附加费的价格
og.goods_price_original,
-- 商品未促销的价格
og.no_deal_price,
-- 披肩美元价格
og.wrap_price,
-- 当前下单使用币种转换后的披肩价格
og.wrap_price_exchange,
-- 显示用商品单价
og.display_shop_price_exchange,
-- 显示用商品总金额费
og.display_shop_price_amount_exchange,
-- 显示用自定义尺寸费
og.display_custom_fee_exchange,
-- 显示用加大码费
og.display_plussize_fee_exchange,
-- 显示用加急费
og.display_rush_order_fee_exchange,
-- 显示用披肩费
og.display_wrap_price_exchange,
-- 鞋跟定制费用
og.heel_type_price,
-- 鞋跟定制支付币种费用
og.heel_type_price_exchange,
-- 鞋跟定制支付币种显示费用
og.display_heel_type_price_exchange,
--------------以上------------------
-- 订单中商品的id编号
og.oiid
from 
ods_order_goods as og
where 1=1
and report_date = '2020-05-01'
```



## 04, 订单商品表选取所需要的字段

```mysql
select 
-- 这个是自增id，重要
og.rec_id,
-- 订单id，重要，因为要和订单关联，这里取一个即可
-- og.order_id,
-- @see goods_style.goods_style_id
og.goods_style_id,
-- @see goods_style.sku
og.sku, 
-- @see goods_sku.sku_id
og.sku_id,
og.goods_id, 
-- 商品名不取这个，取goods_languages里面的
-- og.goods_name,
og.goods_number,
-- 商品的市场价格
og.market_price,
-- 这个是商品的实际价格
og.shop_price,
-- 价格转换后的数额
og.shop_price_exchange,
-- 总额的转换后数值
og.shop_price_amount_exchange,
-- 该商品的折扣，负值；可能来自分类折扣或该商品折扣
og.bonus,

-- rush order fee，加急费
og.rush_order_fee, 
-- rush order fee，加急费
og.rush_order_fee_exchange,

-- 用户选择或输入的各种样式，json；记录备查
og.styles,

-- 订单中商品的id编号
og.oiid
from 
ods_order_goods as og
where 1=1
and report_date = '2020-05-01'
```












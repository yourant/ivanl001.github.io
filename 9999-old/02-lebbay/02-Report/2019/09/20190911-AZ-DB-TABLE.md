# AZ DB TABLE DESCRIPTION

## users 用户表
```mysql
CREATE TABLE `users` (
  `user_id` int(10) unsigned NOT NULL AUTO_INCREMENT, # Primary Key
  `userId` char(32) NOT NULL,                         # uuid 暂无用
  `email` varchar(60) NOT NULL DEFAULT '',            # 登录账户
  `inform_email` varchar(60) NOT NULL DEFAULT '',     #  
  `user_name` varchar(60) NOT NULL DEFAULT '',        # 
  `password` varchar(32) NOT NULL DEFAULT '',  
  `question` varchar(255) NOT NULL DEFAULT '',        # ？
  `answer` varchar(255) NOT NULL DEFAULT '',          # ？
  `gender` enum('male','female') DEFAULT 'male',      # 
  `birthday` date NOT NULL DEFAULT '0000-00-00',  
  `address_id` int(10) unsigned NOT NULL DEFAULT '0', # 应该是关联地址表的ID
  `reg_time` datetime NOT NULL DEFAULT '0000-00-00 00:00:00', # 注册时间
  `last_time` datetime NOT NULL DEFAULT '0000-00-00 00:00:00', # 上次登录时间
  `last_ip` varchar(64) NOT NULL DEFAULT '' COMMENT '最后登录IP地址', # 上次登录地址IP
  `visit_count` smallint(5) unsigned NOT NULL DEFAULT '0',     #  ？
  `user_rank` tinyint(3) unsigned NOT NULL DEFAULT '0',        #   ？
  `is_special` tinyint(3) unsigned NOT NULL DEFAULT '0',     #？
  `salt` varchar(10) NOT NULL DEFAULT '0',                   # 看着像密码加密方面的salt，应该无用
  `user_realname` varchar(60) NOT NULL COMMENT '真是姓名',   #
  `user_mobile` varchar(20) NOT NULL COMMENT '手机',         #
  `country` smallint(5) NOT NULL DEFAULT '0',               # 国籍
  `province` smallint(5) NOT NULL DEFAULT '0',              # 户籍
  `city` smallint(5) NOT NULL DEFAULT '0',                  # 户籍城市
  `district` smallint(5) NOT NULL DEFAULT '0',            # ？
  `zipcode` varchar(60) NOT NULL,                          # ？
  `user_address` varchar(255) NOT NULL COMMENT '联系地址',  # 
  `track_id` char(32) NOT NULL,                            # ？
  `reg_source` int(10) unsigned NOT NULL COMMENT '注册来源',
  `reg_province` int(10) unsigned NOT NULL COMMENT '注册来源地',
  `reg_recommender` varchar(50) NOT NULL DEFAULT 'azazie' COMMENT '注册推荐人',
  `user_tel` varchar(20) DEFAULT NULL,             # ？
  `email_valid` tinyint(1) unsigned NOT NULL DEFAULT '0' COMMENT '是否通过邮件验证',
  `email_validate_point_id` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '通过邮件验证时所发放的积分ID',
  `unsubscribe` tinyint(1) unsigned NOT NULL DEFAULT '0' COMMENT '是否取消邮件订阅',
  `is_delete` tinyint(4) NOT NULL DEFAULT '0' COMMENT '帐号删除',
  `language_id` mediumint(9) NOT NULL DEFAULT '0',           # 使用语言
  `reg_site_name` varchar(50) NOT NULL DEFAULT '' COMMENT '注册来源网站',
  `reg_site_host` varchar(255) NOT NULL DEFAULT 'www.azazie.com',
  `wedding_date` date NOT NULL DEFAULT '0000-00-00',
  `last_update_wedding_date` date NOT NULL DEFAULT '0000-00-00',
  `last_get_notification_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '最后访问app消息中心时间',
  PRIMARY KEY (`user_id`),
  UNIQUE KEY `userId` (`userId`),
  UNIQUE KEY `reg_source` (`email`,`reg_recommender`,`reg_site_name`),
  KEY `reg_time` (`reg_time`),
  KEY `user_name` (`user_name`),
  KEY `unsubscribe` (`unsubscribe`),
  KEY `password` (`password`)
) ENGINE=InnoDB AUTO_INCREMENT=870 DEFAULT CHARSET=utf8;
```

## category 分类表
```mysql
CREATE TABLE `category` (
  `cat_id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `cat_name` varchar(90) NOT NULL DEFAULT '',            # 分类名
  `cat_goods_name` varchar(90) NOT NULL DEFAULT '' COMMENT '商品名称中显示的分类名称',
  `depth` int(10) unsigned NOT NULL DEFAULT '0',
  `keywords` varchar(255) NOT NULL DEFAULT '',           # 用于索引
  `cat_desc` varchar(2000) NOT NULL DEFAULT '',          # 描述
  `parent_id` int(10) unsigned NOT NULL DEFAULT '0',     # 父级ID
  `sort_order` int(10) unsigned NOT NULL DEFAULT '0',    # 排序
  `is_show` tinyint(1) NOT NULL DEFAULT '1',
  `party_id` int(10) unsigned NOT NULL DEFAULT '1' COMMENT '分离id',
  `config` text NOT NULL COMMENT '配置信息，制作时间 6 - 8 等',
  `erp_cat_id` int(11) NOT NULL DEFAULT '0' COMMENT 'erp 分类id',
  `erp_top_cat_id` int(11) NOT NULL DEFAULT '0' COMMENT 'erp 父分类id',
  `pk_cat_id` int(11) NOT NULL DEFAULT '0' COMMENT 'see category_erp.pk_cat_id',
  `is_accessory` tinyint(1) NOT NULL DEFAULT '0' COMMENT '是否是配件',
  `last_update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '最后更新时间',
  PRIMARY KEY (`cat_id`),
  KEY `parent_id` (`parent_id`),
  KEY `last_update_time` (`last_update_time`),
  KEY `last_update_time_2` (`last_update_time`)
) ENGINE=InnoDB AUTO_INCREMENT=255 DEFAULT CHARSET=utf8 COMMENT='商品类目信息，cat_id是主键，parent_id是父类目的主键';
```

## goods 商品表
```mysql
CREATE TABLE `goods` (
  `goods_id` int(10) unsigned NOT NULL AUTO_INCREMENT,   # Primary Key
  `goods_party_id` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '分隔不同用户', # 不同来源的商品
  `cat_id` smallint(5) unsigned NOT NULL DEFAULT '0',      # `category`.`cat_id`
  `goods_sn` varchar(60) NOT NULL DEFAULT '',              # ?
  `sku` varchar(60) NOT NULL,                              # ?
  `goods_name` varchar(255) NOT NULL DEFAULT '',           # 商品名，但是以goods_languages表数据为准
  `goods_url_name` varchar(255) NOT NULL DEFAULT '' COMMENT 'url 重写', # 商品地址，同上
  `brand_id` smallint(5) unsigned NOT NULL DEFAULT '0',      # ？
  `goods_number` smallint(255) unsigned NOT NULL DEFAULT '0',  # ？
  `goods_weight` int(11) unsigned NOT NULL DEFAULT '0' COMMENT '商品重量，克(g)',  # ？
  `goods_weight_bak` int(11) NOT NULL DEFAULT '0' COMMENT 'goods_weight的备份20110420-201800', # ？
  `market_price` decimal(10,2) unsigned NOT NULL DEFAULT '0.00',         # 市场价
  `shop_price` decimal(10,2) unsigned NOT NULL DEFAULT '0.00',           # 售价
  `no_deal_price` decimal(10,2) unsigned NOT NULL DEFAULT '0.00',        # 划线价
  `wrap_price` decimal(10,2) unsigned NOT NULL DEFAULT '0.00' COMMENT '披肩价格',    # 配件价格
  `fitting_price` decimal(10,2) unsigned NOT NULL DEFAULT '0.00',           
  `promote_price` decimal(10,2) unsigned NOT NULL DEFAULT '0.00',
  `promote_start` date NOT NULL DEFAULT '0000-00-00',
  `promote_end` date NOT NULL DEFAULT '0000-00-00',
  `warn_number` tinyint(3) unsigned NOT NULL DEFAULT '1',
  `keywords` varchar(255) NOT NULL DEFAULT '',                    # 用以做索引
  `goods_brief` varchar(1024) NOT NULL,         
  `goods_desc` text NOT NULL,                                     # 商品描述
  `goods_thumb` varchar(255) NOT NULL DEFAULT '',                 # 商品默认切图
  `goods_img` varchar(255) NOT NULL DEFAULT '',                   # 商品图（详细图应该）
  `original_img` varchar(255) NOT NULL DEFAULT '',                # 原始图片
  `is_on_sale` tinyint(1) unsigned NOT NULL DEFAULT '1',          # 是否上架
  `add_time` int(10) unsigned NOT NULL DEFAULT '0',              # 添加时间
  `is_delete` tinyint(1) unsigned NOT NULL DEFAULT '0',           # 是否删除
  `is_promote` tinyint(1) unsigned NOT NULL DEFAULT '0', 
  `last_update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '最后更新时间',
  `goods_type` smallint(5) unsigned NOT NULL DEFAULT '0' COMMENT '区分录入商品与定制生成商品',
  `goods_details` text,                      # 商品详情
  `is_on_sale_pending` tinyint(1) NOT NULL,  # 上架状态
  `top_cat_id` smallint(5) NOT NULL DEFAULT '0',
  `sale_status` enum('tosale','presale','shortage','normal','withdrawn','booking') NOT NULL DEFAULT 'normal',
  `is_display` tinyint(4) NOT NULL DEFAULT '1',  # 是否显示
  `is_complete` tinyint(1) unsigned NOT NULL DEFAULT '0' COMMENT '编辑是否完成',
  `comment_count` int(11) NOT NULL DEFAULT '0' COMMENT 'comment count',
  `question_count` int(11) NOT NULL DEFAULT '0' COMMENT '商品提问数量',
  `is_new` tinyint(1) unsigned NOT NULL DEFAULT '0' COMMENT '是否新品',
  `fb_like_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT 'facebook like',
  `model_card` varchar(100) NOT NULL DEFAULT '' COMMENT '模特卡',
  `old_goods_id` int(11) NOT NULL DEFAULT '0' COMMENT '原记录ID',
  `on_sale_time` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
  `cp_goods_id` int(11) NOT NULL DEFAULT '0' COMMENT '复制商品用',
  PRIMARY KEY (`goods_id`),
  UNIQUE KEY `sku` (`sku`),
  KEY `goods_sn` (`goods_sn`),
  KEY `cat_id` (`cat_id`),
  KEY `goods_weight` (`goods_weight`),
  KEY `goods_number` (`goods_number`),
  KEY `top_cat_id` (`top_cat_id`),
  KEY `is_new` (`is_new`),
  KEY `fb_like_count` (`fb_like_count`),
  KEY `is_on_sale` (`is_on_sale`,`is_delete`,`is_display`),
  KEY `goods_thumb` (`goods_thumb`),
  KEY `last_update_time` (`last_update_time`),
  KEY `last_update_time_2` (`last_update_time`),
  KEY `on_sale_time` (`on_sale_time`),
  KEY `cp_goods_id` (`cp_goods_id`)
) ENGINE=InnoDB AUTO_INCREMENT=7001663 DEFAULT CHARSET=utf8;
```

## attribute 属性表 
```mysql
CREATE TABLE `attribute` (
  `attr_id` smallint(5) unsigned NOT NULL AUTO_INCREMENT,  
  `parent_id` smallint(5) unsigned NOT NULL DEFAULT '0',     # 父级ID
  `cat_id` smallint(5) unsigned NOT NULL DEFAULT '0',        # `category`.`cat_id`
  `attr_name` varchar(60) NOT NULL DEFAULT '',               # 属性名称
  `attr_type` enum('radio','checkbox','text') NOT NULL DEFAULT 'text' COMMENT '属性模版输入类型',
  `attr_values` varchar(1024) NOT NULL,                      # 属性值
  `sort_order` tinyint(3) unsigned NOT NULL DEFAULT '0',     # 排序（好像被遗弃）
  `default_value` text NOT NULL, 
  `attr_description` text NOT NULL,
  `is_delete` tinyint(1) unsigned NOT NULL DEFAULT '0',
  `is_show` tinyint(1) unsigned NOT NULL DEFAULT '1',
  `display_order` int(11) DEFAULT '0' COMMENT '属性排序',     # 【排序
  `last_update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '最后更新时间',
  `display_filter` tinyint(1) unsigned DEFAULT '7',
  PRIMARY KEY (`attr_id`),
  UNIQUE KEY `attr_name` (`attr_name`,`parent_id`,`attr_id`),
  KEY `cat_id` (`cat_id`),
  KEY `attr_values_index` (`attr_values`(255)) USING BTREE,
  KEY `last_update_time` (`last_update_time`)
) ENGINE=InnoDB AUTO_INCREMENT=29047 DEFAULT CHARSET=utf8 COMMENT='商品属性表。';
```

## goods_attr 商品属性关联表
```mysql
CREATE TABLE `goods_attr` (
  `goods_attr_id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `goods_id` int(10) unsigned NOT NULL DEFAULT '0',
  `attr_id` int(10) unsigned NOT NULL DEFAULT '0',
  `attr_value_x` text NOT NULL,
  `is_delete` tinyint(1) unsigned NOT NULL DEFAULT '0',
  `is_show` tinyint(1) unsigned NOT NULL DEFAULT '1',
  `last_update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '最后更新时间',
  `cp_goods_attr_id` int(11) NOT NULL DEFAULT '0' COMMENT '复制商品的属性用',
  PRIMARY KEY (`goods_attr_id`),
  UNIQUE KEY `goods_id` (`goods_id`,`attr_id`),
  KEY `attr_id` (`attr_id`),
  KEY `attr_id_2` (`attr_id`,`is_delete`,`is_show`,`goods_id`),
  KEY `last_update_time` (`last_update_time`),
  KEY `last_update_time_2` (`last_update_time`),
  KEY `cp_goods_attr_id` (`cp_goods_attr_id`)
) ENGINE=InnoDB AUTO_INCREMENT=113077 DEFAULT CHARSET=utf8;
```





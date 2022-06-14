# it_blog

```mysql
CREATE TABLE `it_blog` (
  `id` bigint(11) NOT NULL AUTO_INCREMENT,
  `classify_id` bigint(20) NOT NULL DEFAULT '0' COMMENT '分类id',
  `title` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '博客标题',
  `url` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '博客地址',
  `post_time` timestamp NOT NULL COMMENT '发布时间',
  `tag_id` bigint(20) NOT NULL DEFAULT '0' COMMENT '标签id',
  `read_status` tinyint(2) NOT NULL DEFAULT '0' COMMENT '阅读状态',
  `is_deleted` tinyint(2) NOT NULL DEFAULT '0' COMMENT '逻辑删除',
  `server_status` tinyint(2) NOT NULL DEFAULT '0' COMMENT '服务状态',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  `user_id` bigint(20) NOT NULL DEFAULT '0' COMMENT '用户id',
  `mark` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL DEFAULT '' COMMENT '标注',
  `read_time` datetime DEFAULT NULL COMMENT '阅读时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_url` (`url`) USING BTREE,
  KEY `idx_classify` (`classify_id`) USING BTREE,
  KEY `idx_user_tag` (`user_id`,`tag_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1779 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
```



# it_blog_classify

```mysql
CREATE TABLE `it_blog_classify` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `title` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '博客标题',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  `is_deleted` tinyint(2) NOT NULL DEFAULT '0' COMMENT '逻辑删除',
  `user_id` bigint(20) NOT NULL DEFAULT '0' COMMENT '用户id',
  `parent_id` bigint(20) NOT NULL DEFAULT '0' COMMENT '父id',
  PRIMARY KEY (`id`),
  KEY `idx_user` (`user_id`)
) ENGINE=InnoDB AUTO_INCREMENT=11 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
```


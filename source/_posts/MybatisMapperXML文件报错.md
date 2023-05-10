---
title: Mybatis:MapperXML文件报错(Unable to resolve table 'limit')
date: 2023-05-10 14:33:43
categories:
- Mybatis
tags:
- bug
---

# Mybatis:MapperXML文件报错(Unable to resolve table 'limit')

在使用<where><if></if></where>时报错

```javascript
<select id="selectBySearch" parameterType="life.mhe.community.dto.QuestionQueryDTO" resultMap="BaseResultMap">
        select * from question
        <where>
            <if test="search!=null">
                 title regexp #{search}
            </if>
        </where>
        order by gmt_create desc limit #{page},#{size}
    </select>
```

## 解决办法

起初将where和if交换位置依旧报错

类似问题：

[]: https://stackoverflow.com/questions/60575770/

最终将标签<where>改为where关键字解决

```javascript
where
            <if test="search!=null">
                 title regexp #{search}
            </if>
        order by gmt_create desc limit #{page},#{size}
```


---
title: debug-skill
date: 2023-05-12 12:35:32
categories:
- 工具
- idea
- debug
tags:
- debug
- idea
---

# idea中利用条件判断快速定位列表元素

例如在发布问题默认用户时，假设首先通过页面控制台得到question.id==3

我们此时找到QuestionService中 list方法，

```java
for (Question question : questions) {
            User user = userMapper.selectByPrimaryKey(question.getCreator());
            QuestionDTO questionDTO = new QuestionDTO();
            BeanUtils.copyProperties(question, questionDTO);
            questionDTO.setUser(user);
            questionDTOList.add(questionDTO);
        }
```

在第二句中打上断点，并加入condition如下图

![image-20230512124224225](/images/image-20230512124224225.png)

即可直接在id为3时调试bug


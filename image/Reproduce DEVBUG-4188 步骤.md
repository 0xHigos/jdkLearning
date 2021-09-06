Reproduce `DEVBUG-4188` 步骤

### 创建步骤：

+ #### 创建一个新的tenant

  ![image-20210907020758238](/Users/xujie/Library/Application Support/typora-user-images/image-20210907020758238.png)



+ #### 创建 `EXPENSIVE_COMPUTE` 的featureTag

  ![image-20210907020944565](/Users/xujie/Library/Application Support/typora-user-images/image-20210907020944565.png)

+ #### 创建4个`event_attribute`并更新对应的featurTag

  + amount_MARS_648

    ![image-20210907021412977](/Users/xujie/Library/Application Support/typora-user-images/image-20210907021412977.png)

  + phone_MARS_648

    ![image-20210907021504869](/Users/xujie/Library/Application Support/typora-user-images/image-20210907021504869.png)

    

    ![image-20210907022316518](/Users/xujie/Library/Application Support/typora-user-images/image-20210907022316518.png)

  + user_id_MARS_648

  ![image-20210907021545259](/Users/xujie/Library/Application Support/typora-user-images/image-20210907021545259.png)

  ![image-20210907022503618](/Users/xujie/Library/Application Support/typora-user-images/image-20210907022503618.png)

  + payment_time_MARS_648

  ![image-20210907021617796](/Users/xujie/Library/Application Support/typora-user-images/image-20210907021617796.png)

  ![image-20210907022630050](/Users/xujie/Library/Application Support/typora-user-images/image-20210907022630050.png)

  

+ #### 创建一个`TIME_FORMAT` 并更新对应的featureTag

  ![image-20210907021938073](/Users/xujie/Library/Application Support/typora-user-images/image-20210907021938073.png)

![image-20210907022801141](/Users/xujie/Library/Application Support/typora-user-images/image-20210907022801141.png)



+ #### 创建需要计算的feature `getRegistrationPlaceByCode_MARS_64` 并更新featureTag

  格式如下：

  ```json
  {
      "name": "getRegistrationPlaceByCode_MARS_648",
      "script": "String phone = $phone_MARS_648;\nString place = \"Not detected\";\nLOG.error(\"---------------------------------------------------------- start compute feature getRegistrationPlaceByCode_MARS_648 ----------------------------------------------------------\");\nif (phone.contains(\"HUAWEI\")) {\n    place = \"China\";\n} else if (phone.contains(\"SAMSUNG\")) {\n    place = \"Korea\";\n} else if (phone.contains(\"IPHONE\")) {\n    place = \"America\";\n}\nLOG.error(String.format(\"Current phone is: %s, the registration place is: %s\", phone, place));\nLOG.error(\"---------------------------------------------------------- ending compute feature getRegistrationPlaceByCode_MARS_648 ----------------------------------------------------------\");\nreturn place;",
      "returnType": "String",
       "content": "{\"type\":\"coding\",\"function\":{\"script\":\"String phone = $phone_MARS_648;\\nString place = \\\"Not detected\\\";\\nLOG.error(\\\"---------------------------------------------------------- start compute feature getRegistrationPlaceByCode_MARS_648 ----------------------------------------------------------\\\");\\nif (phone.contains(\\\"HUAWEI\\\")) {\\n    place = \\\"China\\\";\\n} else if (phone.contains(\\\"SAMSUNG\\\")) {\\n    place = \\\"Korea\\\";\\n} else if (phone.contains(\\\"IPHONE\\\")) {\\n    place = \\\"America\\\";\\n}\\nLOG.error(String.format(\\\"Current phone is: %s, the registration place is: %s\\\", phone, place));\\nLOG.error(\\\"---------------------------------------------------------- ending compute feature getRegistrationPlaceByCode_MARS_648 ----------------------------------------------------------\\\");\\nreturn place;\"},\"return_type\":\"String\",\"name\":\"getRegistrationPlaceByCode_MARS_648\",\"description\":{\"en\":\"\"}}"
  }
  ```

  

![image-20210907023444078](/Users/xujie/Library/Application Support/typora-user-images/image-20210907023444078.png)

![image-20210907023641216](/Users/xujie/Library/Application Support/typora-user-images/image-20210907023641216.png)



+ #### 创建需要计算的Rule

  格式如下：

  ```json
  {
      "name": "MARS_648_rule_06",
      "script": "if ((((numEQ(@amount_MARS_648, 6000)) && (strEquals(@getRegistrationPlaceByCode_MARS_648, \"Korea\"))))) {result.addAction(\"AUTO_DECISION\", \"{\\\"REVIEW RESULT\\\":\\\"Good\\\",\\\"REVIEW COMMENT\\\":\\\"MARS_648_rule_06\\\"}\");} \n",
      "status": "DRAFT",
      "rawRule": "{\"actions\":{\"AUTO_DECISION\":\"{\\\"REVIEW RESULT\\\":\\\"Good\\\",\\\"REVIEW COMMENT\\\":\\\"MARS_648_rule_06\\\"}\"},\"condition\":{\"and\":[{\"and\":[{\"operands\":[\"amount_MARS_648\",\"6000\"],\"operator\":\"INT_EQ\"},{\"operands\":[\"getRegistrationPlaceByCode_MARS_648\",\"Korea\"],\"operator\":\"STR_EQ\"}]}]}}",
      "type": "MANUAL"
  
  }
  ```

  ![image-20210907024406378](/Users/xujie/Library/Application Support/typora-user-images/image-20210907024406378.png)



+ #### 创建需要使用的rule-set

  ![image-20210907024533542](/Users/xujie/Library/Application Support/typora-user-images/image-20210907024533542.png)



### 验证：

#### 数据一(不满足rule条件)：

```json
{
    "phone_MARS_648": "HUAWEI-P50",
    "user_id_MARS_648": 1,
    "amount_MARS_648": 5000,
    "payment_time_MARS_648": "2021-08-26 20:10:29"
}
```

![image-20210907024819439](/Users/xujie/Library/Application Support/typora-user-images/image-20210907024819439.png)

![image-20210907024853959](/Users/xujie/Library/Application Support/typora-user-images/image-20210907024853959.png)

没有对应的 

 `start/end compute feature getRegistrationPlaceByCode_MARS_648`

日志



#### 数据二(满足rule条件)：

```json
{
    "phone_MARS_648": "SAMSUNG-S22",
    "user_id_MARS_648": 2,
    "amount_MARS_648": 6000,
    "payment_time_MARS_648": "2021-08-26 20:10:32"
}
```

![image-20210907025339174](/Users/xujie/Library/Application Support/typora-user-images/image-20210907025339174.png)

有对应的 

 `start/end compute feature getRegistrationPlaceByCode_MARS_648`

日志



### 数据三(不满足rule条件)：

```
{
    "phone_MARS_648": "IPHONE-13",
    "user_id_MARS_648": 23,
    "amount_MARS_648": 7000,
    "payment_time_MARS_648": "2021-08-26 20:10:39"
}
```

![image-20210907025543283](/Users/xujie/Library/Application Support/typora-user-images/image-20210907025543283.png)



验证的结果，与我们代码逻辑预期是一样的...
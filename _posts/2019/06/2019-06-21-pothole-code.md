---
layout: post
title: 史上最坑爹的代码！个个让人崩溃！
category: life
tags: [life]
---
 


_https://gitee.com/oschina/bullshit-codes_

Java 就是这么一门神奇的语言，任何水平的人都能写出可以运行的代码，但是一看代码便知水平高低。作为一个程序员，你看过哪些坑爹代码，你又写过多少坑爹代码，还有多少你不知道的坑爹代码？

有意思的是码云上建了一个代码仓库：**bullshit-codes**，仓库的目的就是为了收集这些坑爹代码，可以让别人不掉坑或者少掉坑，可以避免自己掉坑，或许哈哈一乐！

上边汇聚了各种编程语言的，仓库地址如下：

https://gitee.com/oschina/bullshit-codes

本文小编给大家整理了几个比较坑爹的代码，整理了几个之后，实在整理不下去了，仅供大家参考，看看能不能崩溃掉！

**一、几个坑爹代码的目录**

1、这样使用 StringBuffer 的方法有什么坑？

2、你写过的最长的一行代码有多长？？？

3、循环+条件判断，你最多能嵌套几层？

4、为了后期优化查询速度 ~ 颇有商业头脑！

5、你是如何被异常玩然后变成玩异常的？

6、Stream 玩得最 6 的代码，看过的人都惊呆了！

**二、坑爹代码 | 这样使用 StringBuffer 的方法有什么坑？**
    
    你是否曾经这样使用过 Java 的 StringBuffer 类？
    
    
    /**
     * Create Time 2019/5/24
     * StringBuffer追加 如痴如醉的写法
     * @author cailong
     **/
    public class Append {
        public static void main(String[] ares){
    
            StringBuffer sb = new StringBuffer();
            //这里都能理解
            sb.append("<?xml version=\"1.0\" encoding=\"UTF-8\"?><ROOT>");
            for (int i = 0; i < 10; i++) {
                //为什么到这里就要这样写？？？既然都用StringBuffer了    （这里省略集合遍历用i代替 意思能懂就行）
                sb.append("<NSRXX>" +
                        "<NSRSBH>"+i+"</NSRSBH>" +
                        "<NSRMC>"+i+"</NSRMC>" +
                        "<DJXH>"+i+"</DJXH>" +
                        "<ZGSWJ_DM>"+i+"</ZGSWJ_DM>" +
                        "<ZGSWJ_MC>"+i+"</ZGSWJ_MC>" +
                        "<SJLY>sjzs</SJLY>" +
                        "<YWSX_DM>"+i+"</YWSX_DM>" +
                        "</NSRXX>");
            }
            sb.append("</ROOT>");
            System.out.println(sb.toString());
        }
    }

**三、坑爹代码 | 你写过的最长的一行代码有多长？****？****？**

你写过的最长的一行代码有多长吗？你为什么要写这么长？是看着帅呢，还是想减少代码行数？
    
    List<OperationPurchaseInfo> purchaseInfoList
                    = sheet.getPurchaseInfoList()
                    .stream()
                    .filter(purchaseInfo -> purchaseInfo.getExteriorOperation()
                            .getExteriorPart()
                            .getExteriorOperationList()
                            .stream()
                            .filter(exteriorOperation -> exteriorOperation
                                    .getProcessState()
                                    .equals(ExteriorOperation.ProcessState.PROCESSING))
                            .count() != 0
                            // 订单明细中工序对应的工件下的其他工序存在加工中，
                            // 且已发给供应商且供应商不是当前订单供应商时，需要判断
                            && (purchaseInfo.getExteriorOperation()
                            .getExteriorPart()
                            .getTeamwork() == null || !purchaseInfo.getExteriorOperation()
                            .getExteriorPart().getTeamwork().equals(sheet.getTeamwork()))
                    ).collect(Collectors.toList());

上面这段代码虽然被拆开多行显示，但本质上是一行，一个极其复杂的赋值语句！

这种代码是不是为了让别人看不懂来彰显自己的编码水平呢？

小编觉得 Java Stream API 以及各种函数式编程方法，以及各种语法糖在某种程度让这种糟糕代码越来越多！

那么一起来批判一下这个代码，或者你有什么好的解决方案呢？

**四、坑爹代码 | 循环+条件判断，你最多能嵌套几层？**

for 循环和 if 条件判断语句，必不可少吧。但是你见过最多嵌套的循环和条件判断有几层呢？或者说，你最多能容忍多少层的嵌套呢？

![](https://ziyekudeng.github.io/assets/images/2019/0621/1.webp) 


我们还是先来看看极端的坑爹代码吧：

    
    // 这个无限循环嵌套，只是总循环的一部分。。。我已经绕晕在黄桷湾立交
            if (recordList.size() > start) {
                for (int i = start; i < end; i++) {
                    Map<String, Object> map = recordList.get(i);
                    Map<String, Object> field11 = (Map<String, Object>) map.get("field"); //name -> code
                    Map<String, Object> record11 = (Map<String, Object>) map.get("record"); // code -> value
                    String catagory1 = map.get("categoryId").toString();
                    //  查询第一种类型对应的其他类型
                    SalaryDataVo ss = JSON.parseObject(JSON.toJSONString(map), SalaryDataVo.class);
                    Page page3 = salaryManagerService.getAllRecordsByCondition(ss);
                    if (page3.getRecords().size() > 0) {
                        List<Map<String, Object>> salaryDataVos = page3.getRecords();
                        salaryDataVos = this.reSetMap(salaryDataVos, null, null);
                        for (Map<String, Object> map2 : salaryDataVos) {
                            Map<String, Object> field2 = (Map<String, Object>) map2.get("field");
                            Map<String, Object> record2 = (Map<String, Object>) map2.get("record");
                            String catagory2 = map2.get("categoryId").toString();
                            List<SalaryGroupVO> groupList2 = salaryGroupService.getSalaryGroupsItems(this.getUserCorpId(), catagory2);
                            for (SalaryGroupVO cc : groupList2) {
                                cc.setCode(cc.getParentId() + cc.getCode());
                            }
                            //计算
                            for (Map.Entry<String, Object> entity : field2.entrySet()) {
                                String keyName = entity.getKey();
                                for (SalaryGroupVO s2 : groupList2) {
                                    if ("bigDecimal".equals(s2.getItemType()) && s2.getCode().equals(field2.get(keyName).toString()) && ("部门" != keyName) && ("姓名" != keyName) && StringUtils.isNotEmpty(s2.getItemType())) {
                                        if (field11.containsKey(keyName)) {
                                            if (field11.containsKey(keyName)) {
                                                String code1 = field11.get(keyName).toString();
                                                Double newValue = 0.0;
                                                Double oldValue = 0.0;
                                                if (!record11.get(code1).toString().matches("^[0-9]*$")) {
                                                    oldValue = Double.parseDouble(record11.get(code1).toString());
                                                    if (record2.containsKey(entity.getValue().toString()) && (!record2.get(entity.getValue().toString()).toString().matches("^[0-9]*$"))) {
                                                        String value2 = record2.get(entity.getValue().toString()).toString();
                                                        newValue = Double.parseDouble(value2);
                                                    }
                                                    record11.remove(field11.get(keyName).toString());
                                                }
                                                if (code1.startsWith(catagory1) || code1.startsWith(catagory2)) {
                                                    String co = code1.replace(catagory1, "hz");
                                                    field11.put(keyName, co);
                                                    record11.put(co, oldValue + newValue);
                                                }
                                            }
                                        } else {
                                            String code = entity.getValue().toString();
                                            String str = entity.getValue().toString();
                                            Object value2 = record2.get(entity.getValue().toString());
                                            if (str.startsWith(catagory1) && str.replace(catagory1, "").startsWith("hz")) {
                                                code = str.replace(catagory1, "");
                                            } else if (str.startsWith(catagory2) && str.replace(catagory2, "").startsWith("hz")) {
                                                code = str.replace(catagory2, "");
                                            }
                                            field11.put(keyName, code);
                                            record11.put(code, value2);
                                        }
                                    }
                                }
                            }
                        }
                    }
                    List<SalaryGroupVO> sList = salaryGroupService.getSalaryGroupItemsByParentId(catagory1);
                    for (SalaryGroupVO s : sList) {
                        if (field11.containsKey(s.getName()) && s.getCode().startsWith("hz")) {
                            String k = field11.get(s.getName()).toString();
    
                            field11.put(s.getName(), s.getCode());
                            String value = null;
                            if (record11.containsKey(k)) {
                                value = record11.get(k).toString();
                            }
                            record11.put(s.getCode(), value);
                        }
                    }
                    resultList.add(map);
                    pageInfo.setRecords(resultList);
                }
            }

我数了数，一共有 11 层的嵌套！！！ 

吐槽归吐槽，这样的代码逻辑有什么重构的好方法吗？

**五、坑爹代码 | 为了后期优化查询速度 ~ 颇有商业头脑！**

什么样的程序员是一个好程序员呢？当我们在给客户开发系统时候，为了后期的优化，预留一些埋点。

通过对这些埋点的优化，可以让客户瞬间感觉系统在运行速度上有质的飞跃，让公司顺利的签署二期开发合同，收取巨额开发费用。

从公司角度来看，这样的程序员就是一个好程序员。—— 这句话不是红薯说的！

比如：

我想说的是：凶碟，你下手能否不那么狠啊，建议对代码进行优化，改成 Thread.sleep(1000);  —— 这句话也不是红薯说的。

或者你有什么更好的建议呢？不要觉得骇人听闻，真有这样的人，这样的代码！！！

![](https://ziyekudeng.github.io/assets/images/2019/0621/2.webp) 

**六、坑爹代码 | 你是如何被异常玩然后变成玩异常的？**

玩 Java 的人，刚开始会被各种异常虐，空指针应该是最常见的。多玩两年就开始玩异常，各种奇葩异常玩法层出不穷。

你觉得下面这种异常的定义妥吗？

    
    /**
     * 处理业务的异常
     * 居然有一堆静态异常，准备好了随时可以抛？？
     * 错误码是字符串
     */
    public class CommandException extends BaseException {
    
      private static final long serialVersionUID = -6354513454371927970L;
      
      public static CommandException PARAM_NULL= new CommandException("Command_assemble_01", "Parameter is Needed But Empty");
      public static CommandException DEVID_NULL = new CommandException("Command_assemble_02", "DevId Cannot Be Null");
      public static CommandException MDCODE_NULL = new CommandException("Command_assemble_03", "Model Code Cannot Be Empty");
      public static CommandException ORDER_NULL = new CommandException("Command_assemble_04", "Order Cannot Be Empty");
      public static CommandException TYPE_NULL = new CommandException("Command_assemble_05", "Upstream / Downstream Type Cannot Be Empty");
      public static CommandException MENUID_NULL = new CommandException("Command_assemble_06", "Menu Id Cannot Be Null");
      public static CommandException CTRLTYPE_NOT_RANGE= new CommandException("Command_assemble_07", "Ctrltype Cannot Be Recognized, Which is not in Range");
      public static CommandException CMD_NULL = new CommandException("Command_analyze_01", "CMD Cannot Be Null");
      public static CommandException PAYLOAD_NULL = new CommandException("Command_analyze_02", "Payload Cannot Be Null");
      public static CommandException FRAMEWORK_FAILED= new CommandException("Command_analyze_03", "Framework Failed to be Checked");
      public static CommandException CHECK_FAILED= new CommandException("Command_analyze_04", "Command Failed to be Checked");
      public static CommandException CONFIGURE_NOT_EXIST = new CommandException("Command_error_1001", "Configure is not Exist");
      public static CommandException REDIS_ERROR = new CommandException("Command_error_1002", "Cache Command in Redis Error", true);
      
      //省略构造函数、get/set方法
    }

如果不妥，有什么问题呢？ 

**七、坑爹代码 | Stream 玩得最 6 的代码，看过的人都惊呆了！**

Stream 作为 Java 8 的一大亮点，它与 java.io 包里的 InputStream 和 OutputStream 是完全不同的概念。Java 8 中的 Stream 是对集合（Collection）对象功能的增强，它专注于对集合对象进行各种非常便利、高效的聚合操作（aggregate operation），或者大批量数据操作 (bulk data operation)。Stream API 借助于同样新出现的 Lambda 表达式，极大的提高编程效率和程序可读性。

那么 Java 8 的 Stream 到底是王者，还是个巨坑，这完全取决于你是怎么玩的？

我不得不说，下面代码是我从业 20 年（说谁呢，谁从业 20 年，我今年 19  岁！！！）看过最牛逼的 Stream 的用法：
    
    
    //Stream 用的66的
    final EventAction eventAction = redisObj(
                    EventActionKey + distributionEventId,
                    () -> Optional
                            .of(distributionEventId)
                            .map(eventId -> {
                                final EventActionExample example = new EventActionExample();
                                example.createCriteria()
                                        .andEventIdEqualTo(eventId)
                                        .andTriggerTypeEqualTo(EnumEventTriggerType.DISTRIBUTION_PURCHASE.getCode().byteValue());
                                return example;
                            })
                            .map(eventActionMapper::selectByExample)
                            .filter(StringUtil::isNotEmpty)
                            .map(e -> e.get(0)).orElseThrow(() -> ExceptionUtil.createParamException("事件触发信息不存在"))
                    , EventAction.class);
            final AwardConfig awardConfig = redisObj(EventConfigKey + eventAction.getId(),
                    () -> Optional.ofNullable(eventAction.getId())
                            .map(actionId -> {
                                final AwardConfigExample example = new AwardConfigExample();
                                example.createCriteria()
                                        .andActionIdEqualTo(actionId);
                                return example;
                            })
                            .map(awardConfigMapper::selectByExample)
                            .filter(StringUtil::isNotEmpty)
                            .map(e -> e.get(0)).orElseThrow(() -> ExceptionUtil.createParamException("xxx")),
                    AwardConfig.class
            );
            Optional.of(req)
                    .map(e -> e.clueUid)
                    .map(id -> {
                        final ClueExample example = new ClueExample();
                        example.createCriteria()
                                .andClueUidEqualTo(id)
                                .andDeletedEqualTo(false)
                                .andReceivedEqualTo(false)
                                .andCreateTimeGreaterThan(now - cluetime);
                        example.setOrderByClause("create_time asc");
                        return example;
                    })  // 获取该被邀请人所有未过期且未被领取的线索的线索
                    .map(clueMapper::selectByExample)
                    .filter(StringUtil::isNotEmpty)
                    .ifPresent(clues -> {
                                final ClueResp clueResp = Optional.of(req)
                                        .filter(c -> {
                                            c.count = clues.size();
                                            return true;
                                        })
                                        .map(this::awardValue)
                                        .orElseThrow(() -> ExceptionUtil.createParamException("参数错误"));
                                final Integer specialId = req.getIsHead()
                                        ? clues.get(0).getId()
                                        : clues.get(clues.size() - 1).getId();
                                clues.stream()
                                        .peek(clue -> {
                                            final AwardConfig awardConfigclone = Optional.of(awardConfig)
                                                    .map(JSONUtil::obj2Json)
                                                    .map(json -> JSONUtil.json2Obj(json, AwardConfig.class))
                                                    .orElseGet(AwardConfig::new);
                                            awardConfigclone.setMoney(
                                                    Optional.of(clue.getId())
                                                            .filter(specialId::equals)
                                                            .map(e -> clueResp.specialReward.longValue())
                                                            .orElse(clueResp.otherAverageReward.longValue())
                                            );
                                            eventActionService.assembleAward(
                                                    awardConfigclone,
                                                    clue.getAdviserUid(),
                                                    clue.getAdviserUid(),
                                                    clue.getClueUid(),
                                                    eventAction,
                                                    new PasMessageParam(),
                                                    clue.getId(),
                                                    AwardRelationType.Clud.code()
                                            );
                                        })
                                        .forEach(clue -> {
                                            clue.setOrderNo(req.orderNo);
                                            clue.setCommodityName(req.commodityName);
                                            clue.setOrderAmount(req.orderAmount);
                                            clue.setReceived(true);
                                            clue.setModifyTime(now);
                                            clueMapper.updateByPrimaryKeySelective(clue);
                                        });
                            }
                    );

Java 就是这么一门神奇的语言，任何水平的人都能写出可以运行的代码，但是一看代码便知水平高低。但是这样的 Stream 代码你一定一口老谈不吐不快！

**八、项目源码地址**

大家有兴趣的可以去：

https://gitee.com/oschina/bullshit-codes


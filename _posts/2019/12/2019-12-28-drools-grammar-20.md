---
layout: post
title: drools规则流的使用
category: drools
tags: [drools]
---


### 什么是规则流，规则流什么用？

规则流能够控制，规则中的复杂流程，在复杂业务中，很多时候并不需要触发所有的规则，很多时候需要触发的规则也需要，像程序一样，符合某些逻辑，如，当X对象X 属性等于 A 时，触发 规则A 中的规则，当等于B时，触发规则B中的规则，这时候用规则流就能够很好的处理，这类问题。

### drools版本

Drools7.5.0

### 规则流怎么使用

如何使用规则流？那我们就来写个例子来尝试下吧！我们还用老例子，输入一个分数，我们判断一个水平，如：60分 及格，70-90分优秀 ，然后我们根据判断结果打印出对应的语句。

1.  项目结构
    ![项目结构](https://ziyekudeng.github.io/assets/images/2019/1228/drools-20/1.png)
2.  创建实体类

        package com.sample;
    
        public class ScoreRule {
            //分数
            private Integer score;
            //注释
            private String desc;
    
            public Integer getScore() {
                return score;
            }
    
            public void setScore(Integer score) {
                this.score = score;
            }
    
            public String getDesc() {
                return desc;
            }
    
            public void setDesc(String desc) {
                this.desc = desc;
            }
    
        }
        
3.  创建规则文件，直接贴代码

        package com.sample
    
        import com.sample.ScoreRule;
    
        //执行之后不重复执行
        no-loop
    
        //规则名
        rule "no-pass"//没过
            //规则组名
            ruleflow-group "score"
            when
                s: ScoreRule( score<60 )
            then
                s.setDesc( "no-pass" );
                update( s );
        end
    
        rule "pass"//及格
            ruleflow-group "score"
            when
                s: ScoreRule( score>=60&&score<70 )
            then
                s.setDesc( "pass" );
                update( s );
        end
    
        rule "good"//良好
            ruleflow-group "score"
            when
                s: ScoreRule( score>=70&&score<90 )
            then
                s.setDesc( "good" );
                update( s );
        end
    
        rule "excellent"//优秀
            ruleflow-group "score"
            when
                s: ScoreRule( score>=90&&score<100 )
            then
                s.setDesc( "excellent" );
                update( s );
        end
    
        rule "perfect"//完美
            ruleflow-group "score"
            when
                s: ScoreRule( score==100 )
            then
                s.setDesc( "perfect" );
                update( s );
        end
4.  修改kmodule.xml 文件

    
    <kmodule xmlns="http://jboss.org/kie/6.0.0/kmodule">
        <kbase name="process" packages="process">
            <ksession name="ksession-process"/>
        </kbase>
    </kmodule> 
5.  新建规则流文件 sample.bpmn
    ![规则](https://ziyekudeng.github.io/assets/images/2019/1228/drools-20/2.png)

    指定规则文件相关属性：
    ![bpmn描述](https://ziyekudeng.github.io/assets/images/2019/1228/drools-20/3.png)

    流程中指定，分数校验模块的规则组：
    ![指定规则组](https://ziyekudeng.github.io/assets/images/2019/1228/drools-20/4.png)

    设置网关的判断条件：
    ![网关判断条件](https://ziyekudeng.github.io/assets/images/2019/1228/drools-20/5.png)

    编辑各条网关连接线的判断依据：
    ![判断依据](https://ziyekudeng.github.io/assets/images/2019/1228/drools-20/6.png)

    编辑分支流程继续执行的任务，在这，仅执行一个打印任务：
    ![分支执行任务](https://ziyekudeng.github.io/assets/images/2019/1228/drools-20/7.png)

6.  好了，我们可以编写测试代码了，代码如下：

        package com.sample;
    
        import org.kie.api.KieServices;
        import org.kie.api.runtime.KieContainer;
        import org.kie.api.runtime.KieSession;
    
        /**
         * This is a sample file to launch a process.
         */
        public class ProcessTest {
    
            public static final void main(String[] args) {
                try {
                    KieServices ks = KieServices.Factory.get();
                    KieContainer kContainer = ks.getKieClasspathContainer();
                    KieSession kSession = kContainer.newKieSession("ksession-process");
                    //编写fact对象
                    ScoreRule rule=new ScoreRule();
                    rule.setScore(75);
                    kSession.insert(rule);
                    //执行所要执行的流程
                    kSession.startProcess("com.sample.bpmn.scorerule");
                    kSession.fireAllRules();  
                } catch (Throwable t) {
                    t.printStackTrace();
                }
            }
    
        } 
7.  好，我们来看看结果吧

    ![结果](https://ziyekudeng.github.io/assets/images/2019/1228/drools-20/8.png)

没错，正确了执行了规则流，当然这个规则流非常简单，大家可以继续尝试更复杂的组件哦，最后，贴下，整个bpmn的代码，供大家参考

     
    <definitions id="Definition"
                 targetNamespace="http://www.jboss.org/drools"
                 typeLanguage="http://www.java.com/javaTypes"
                 expressionLanguage="http://www.mvel.org/2.0"
                 xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL"
                 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                 xsi:schemaLocation="http://www.omg.org/spec/BPMN/20100524/MODEL BPMN20.xsd"
                 xmlns:g="http://www.jboss.org/drools/flow/gpd"
                 xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI"
                 xmlns:dc="http://www.omg.org/spec/DD/20100524/DC"
                 xmlns:di="http://www.omg.org/spec/DD/20100524/DI"
                 xmlns:tns="http://www.jboss.org/drools">
    
      <process processType="Private" isExecutable="true" id="com.sample.bpmn.scorerule" name="ScoreRule" tns:packageName="com.sample" >
    
        <!-- nodes -->
        <startEvent id="_1"  isInterrupting="true">
        </startEvent>
        <endEvent id="_3" >
            <terminateEventDefinition />
        </endEvent>
        <businessRuleTask id="bbb899bd-5f36-4cae-89b6-0b89853a8192" name="分数校验" g:ruleFlowGroup="score" 
     implementation="http://www.jboss.org/drools/rule" >
          <ioSpecification>
            <inputSet>
            </inputSet>
            <outputSet>
            </outputSet>
          </ioSpecification>
        </businessRuleTask>
        <exclusiveGateway id="91cef05a-2603-4018-88ab-d6922ec1f808" name="Gateway" gatewayDirection="Diverging" >
        </exclusiveGateway>
        <scriptTask id="5f6b8b61-685e-4c20-a9b8-bd3e65cc8a4e" name="Script" scriptFormat="http://www.java.com/java" >
          <script>System.out.println("恭喜你已经及格");</script>
        </scriptTask>
        <scriptTask id="084929f2-2d04-45cb-ba3d-ea2755ac402c" name="Script" >
          <script>System.out.println("您的成绩是良好");</script>
        </scriptTask>
        <scriptTask id="54b211e0-c635-4fca-a7b8-4025a05cb3d3" name="Script" >
          <script>System.out.println("优秀优秀！！！！");</script>
        </scriptTask>
        <scriptTask id="840beb7c-37a2-462b-a216-ae80b277ed21" name="Script" >
          <script>System.out.println("恭喜您，满分！");</script>
        </scriptTask>
        <scriptTask id="6d8d6cc7-a04f-4158-8abb-ea85dc0d4c29" name="Script" scriptFormat="http://www.java.com/java" >
          <script>System.out.println("你没及格！");</script>
        </scriptTask>
        <endEvent id="f0ed00b1-39a9-4b27-9234-db4e57beecf9" name="End" >
            <terminateEventDefinition />
        </endEvent>
        <endEvent id="067b5f8a-684d-49fc-9ca2-7c5e125528de" name="End" >
            <terminateEventDefinition />
        </endEvent>
        <endEvent id="d09b3d2e-0483-484f-83be-fd1dedb58b6d" name="End" >
            <terminateEventDefinition />
        </endEvent>
        <endEvent id="b10e26da-6f85-44e1-b49c-7eab1913a1d3" name="End" >
            <terminateEventDefinition />
        </endEvent>
    
        <!-- connections -->
        <sequenceFlow id="6d8d6cc7-a04f-4158-8abb-ea85dc0d4c29-_3" sourceRef="6d8d6cc7-a04f-4158-8abb-ea85dc0d4c29" targetRef="_3" />
        <sequenceFlow id="_1-bbb899bd-5f36-4cae-89b6-0b89853a8192" sourceRef="_1" targetRef="bbb899bd-5f36-4cae-89b6-0b89853a8192" />
        <sequenceFlow id="bbb899bd-5f36-4cae-89b6-0b89853a8192-91cef05a-2603-4018-88ab-d6922ec1f808" sourceRef="bbb899bd-5f36-4cae-89b6-0b89853a8192" targetRef="91cef05a-2603-4018-88ab-d6922ec1f808" />
        <sequenceFlow id="91cef05a-2603-4018-88ab-d6922ec1f808-5f6b8b61-685e-4c20-a9b8-bd3e65cc8a4e" sourceRef="91cef05a-2603-4018-88ab-d6922ec1f808" targetRef="5f6b8b61-685e-4c20-a9b8-bd3e65cc8a4e" name="及格" tns:priority="1" >
          <conditionExpression xsi:type="tFormalExpression" language="http://www.jboss.org/drools/rule" >s: ScoreRule( desc=="pass" )</conditionExpression>
        </sequenceFlow>
        <sequenceFlow id="91cef05a-2603-4018-88ab-d6922ec1f808-084929f2-2d04-45cb-ba3d-ea2755ac402c" sourceRef="91cef05a-2603-4018-88ab-d6922ec1f808" targetRef="084929f2-2d04-45cb-ba3d-ea2755ac402c" name="良好" tns:priority="1" >
          <conditionExpression xsi:type="tFormalExpression" language="http://www.jboss.org/drools/rule" >s: ScoreRule( desc=="good" )</conditionExpression>
        </sequenceFlow>
        <sequenceFlow id="91cef05a-2603-4018-88ab-d6922ec1f808-54b211e0-c635-4fca-a7b8-4025a05cb3d3" sourceRef="91cef05a-2603-4018-88ab-d6922ec1f808" targetRef="54b211e0-c635-4fca-a7b8-4025a05cb3d3" name="优秀" tns:priority="1" >
          <conditionExpression xsi:type="tFormalExpression" language="http://www.jboss.org/drools/rule" >s: ScoreRule( desc=="excellent" )</conditionExpression>
        </sequenceFlow>
        <sequenceFlow id="91cef05a-2603-4018-88ab-d6922ec1f808-840beb7c-37a2-462b-a216-ae80b277ed21" sourceRef="91cef05a-2603-4018-88ab-d6922ec1f808" targetRef="840beb7c-37a2-462b-a216-ae80b277ed21" name="完美" tns:priority="1" >
          <conditionExpression xsi:type="tFormalExpression" language="http://www.jboss.org/drools/rule" >s: ScoreRule( desc=="perfect" )</conditionExpression>
        </sequenceFlow>
        <sequenceFlow id="91cef05a-2603-4018-88ab-d6922ec1f808-6d8d6cc7-a04f-4158-8abb-ea85dc0d4c29" sourceRef="91cef05a-2603-4018-88ab-d6922ec1f808" targetRef="6d8d6cc7-a04f-4158-8abb-ea85dc0d4c29" name="不及格" tns:priority="1" >
          <conditionExpression xsi:type="tFormalExpression" language="http://www.jboss.org/drools/rule" >s: ScoreRule( desc=="no-pass" )</conditionExpression>
        </sequenceFlow>
        <sequenceFlow id="5f6b8b61-685e-4c20-a9b8-bd3e65cc8a4e-f0ed00b1-39a9-4b27-9234-db4e57beecf9" sourceRef="5f6b8b61-685e-4c20-a9b8-bd3e65cc8a4e" targetRef="f0ed00b1-39a9-4b27-9234-db4e57beecf9" />
        <sequenceFlow id="084929f2-2d04-45cb-ba3d-ea2755ac402c-067b5f8a-684d-49fc-9ca2-7c5e125528de" sourceRef="084929f2-2d04-45cb-ba3d-ea2755ac402c" targetRef="067b5f8a-684d-49fc-9ca2-7c5e125528de" />
        <sequenceFlow id="54b211e0-c635-4fca-a7b8-4025a05cb3d3-d09b3d2e-0483-484f-83be-fd1dedb58b6d" sourceRef="54b211e0-c635-4fca-a7b8-4025a05cb3d3" targetRef="d09b3d2e-0483-484f-83be-fd1dedb58b6d" />
        <sequenceFlow id="840beb7c-37a2-462b-a216-ae80b277ed21-b10e26da-6f85-44e1-b49c-7eab1913a1d3" sourceRef="840beb7c-37a2-462b-a216-ae80b277ed21" targetRef="b10e26da-6f85-44e1-b49c-7eab1913a1d3" />
    
      </process>
    
      <bpmndi:BPMNDiagram>
        <bpmndi:BPMNPlane bpmnElement="com.sample.bpmn.scorerule" >
          <bpmndi:BPMNShape bpmnElement="_1" >
            <dc:Bounds x="29" y="173" width="48" height="48" />
          </bpmndi:BPMNShape>
          <bpmndi:BPMNShape bpmnElement="_3" >
            <dc:Bounds x="665" y="19" width="48" height="48" />
          </bpmndi:BPMNShape>
          <bpmndi:BPMNShape bpmnElement="bbb899bd-5f36-4cae-89b6-0b89853a8192" >
            <dc:Bounds x="170" y="173" width="80" height="48" />
          </bpmndi:BPMNShape>
          <bpmndi:BPMNShape bpmnElement="91cef05a-2603-4018-88ab-d6922ec1f808" >
            <dc:Bounds x="354" y="172" width="48" height="48" />
          </bpmndi:BPMNShape>
          <bpmndi:BPMNShape bpmnElement="5f6b8b61-685e-4c20-a9b8-bd3e65cc8a4e" >
            <dc:Bounds x="491" y="98" width="80" height="48" />
          </bpmndi:BPMNShape>
          <bpmndi:BPMNShape bpmnElement="084929f2-2d04-45cb-ba3d-ea2755ac402c" >
            <dc:Bounds x="493" y="170" width="80" height="48" />
          </bpmndi:BPMNShape>
          <bpmndi:BPMNShape bpmnElement="54b211e0-c635-4fca-a7b8-4025a05cb3d3" >
            <dc:Bounds x="496" y="240" width="80" height="48" />
          </bpmndi:BPMNShape>
          <bpmndi:BPMNShape bpmnElement="840beb7c-37a2-462b-a216-ae80b277ed21" >
            <dc:Bounds x="496" y="313" width="80" height="48" />
          </bpmndi:BPMNShape>
          <bpmndi:BPMNShape bpmnElement="6d8d6cc7-a04f-4158-8abb-ea85dc0d4c29" >
            <dc:Bounds x="492" y="22" width="80" height="48" />
          </bpmndi:BPMNShape>
          <bpmndi:BPMNShape bpmnElement="f0ed00b1-39a9-4b27-9234-db4e57beecf9" >
            <dc:Bounds x="670" y="101" width="48" height="48" />
          </bpmndi:BPMNShape>
          <bpmndi:BPMNShape bpmnElement="067b5f8a-684d-49fc-9ca2-7c5e125528de" >
            <dc:Bounds x="671" y="175" width="48" height="48" />
          </bpmndi:BPMNShape>
          <bpmndi:BPMNShape bpmnElement="d09b3d2e-0483-484f-83be-fd1dedb58b6d" >
            <dc:Bounds x="675" y="255" width="48" height="48" />
          </bpmndi:BPMNShape>
          <bpmndi:BPMNShape bpmnElement="b10e26da-6f85-44e1-b49c-7eab1913a1d3" >
            <dc:Bounds x="681" y="326" width="48" height="48" />
          </bpmndi:BPMNShape>
          <bpmndi:BPMNEdge bpmnElement="6d8d6cc7-a04f-4158-8abb-ea85dc0d4c29-_3" >
            <di:waypoint x="532" y="46" />
            <di:waypoint x="689" y="43" />
          </bpmndi:BPMNEdge>
          <bpmndi:BPMNEdge bpmnElement="_1-bbb899bd-5f36-4cae-89b6-0b89853a8192" >
            <di:waypoint x="53" y="197" />
            <di:waypoint x="210" y="197" />
          </bpmndi:BPMNEdge>
          <bpmndi:BPMNEdge bpmnElement="bbb899bd-5f36-4cae-89b6-0b89853a8192-91cef05a-2603-4018-88ab-d6922ec1f808" >
            <di:waypoint x="210" y="197" />
            <di:waypoint x="378" y="196" />
          </bpmndi:BPMNEdge>
          <bpmndi:BPMNEdge bpmnElement="91cef05a-2603-4018-88ab-d6922ec1f808-5f6b8b61-685e-4c20-a9b8-bd3e65cc8a4e" >
            <di:waypoint x="378" y="196" />
            <di:waypoint x="531" y="122" />
          </bpmndi:BPMNEdge>
          <bpmndi:BPMNEdge bpmnElement="91cef05a-2603-4018-88ab-d6922ec1f808-084929f2-2d04-45cb-ba3d-ea2755ac402c" >
            <di:waypoint x="378" y="196" />
            <di:waypoint x="533" y="194" />
          </bpmndi:BPMNEdge>
          <bpmndi:BPMNEdge bpmnElement="91cef05a-2603-4018-88ab-d6922ec1f808-54b211e0-c635-4fca-a7b8-4025a05cb3d3" >
            <di:waypoint x="378" y="196" />
            <di:waypoint x="536" y="264" />
          </bpmndi:BPMNEdge>
          <bpmndi:BPMNEdge bpmnElement="91cef05a-2603-4018-88ab-d6922ec1f808-840beb7c-37a2-462b-a216-ae80b277ed21" >
            <di:waypoint x="378" y="196" />
            <di:waypoint x="536" y="337" />
          </bpmndi:BPMNEdge>
          <bpmndi:BPMNEdge bpmnElement="91cef05a-2603-4018-88ab-d6922ec1f808-6d8d6cc7-a04f-4158-8abb-ea85dc0d4c29" >
            <di:waypoint x="378" y="196" />
            <di:waypoint x="532" y="46" />
          </bpmndi:BPMNEdge>
          <bpmndi:BPMNEdge bpmnElement="5f6b8b61-685e-4c20-a9b8-bd3e65cc8a4e-f0ed00b1-39a9-4b27-9234-db4e57beecf9" >
            <di:waypoint x="531" y="122" />
            <di:waypoint x="694" y="125" />
          </bpmndi:BPMNEdge>
          <bpmndi:BPMNEdge bpmnElement="084929f2-2d04-45cb-ba3d-ea2755ac402c-067b5f8a-684d-49fc-9ca2-7c5e125528de" >
            <di:waypoint x="533" y="194" />
            <di:waypoint x="695" y="199" />
          </bpmndi:BPMNEdge>
          <bpmndi:BPMNEdge bpmnElement="54b211e0-c635-4fca-a7b8-4025a05cb3d3-d09b3d2e-0483-484f-83be-fd1dedb58b6d" >
            <di:waypoint x="536" y="264" />
            <di:waypoint x="699" y="279" />
          </bpmndi:BPMNEdge>
          <bpmndi:BPMNEdge bpmnElement="840beb7c-37a2-462b-a216-ae80b277ed21-b10e26da-6f85-44e1-b49c-7eab1913a1d3" >
            <di:waypoint x="536" y="337" />
            <di:waypoint x="705" y="350" />
          </bpmndi:BPMNEdge>
        </bpmndi:BPMNPlane>
      </bpmndi:BPMNDiagram>
    
    </definitions>


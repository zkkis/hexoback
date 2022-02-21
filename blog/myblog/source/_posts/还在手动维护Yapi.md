---
title: 还在手动维护Yapi?
date: 2022-01-06 16:56:46
tags: Yapi
---

# 【置顶】 留言请点击友链留言室或者在文章底部留言


因前后端人员通过接口定义字段，返回值等对接时非常苦恼，没有一个很好的平台维护，后端每次迭代都要写开发文档，需求变化，多系统联调等，给前后端联调造成阻塞。

# 1、后端开发文档编写规范
1）文档模板统一使用：xxx系统开发文档-模板.docx 
2）后端每次迭代需要编写开发文档，并进行技术评审（前端、后端、测试、产品参与）
3）文档的命名方式：《产品名称-V版本号-设计开发文档-编写人》
4) 文档统一保存
5）后端必须在编写业务代码前，优先设计API（Swagger），并提供给测试和前端，最大化并行迭代。
6）允许特殊情况延期提供API，如：需求变化、多系统联调等。
# Swagger使用规范
1）后端开发人员必须使用Swagger
2）API接口定义参数时，要明确备注信息和是否必须，样例如下：
public class SupplierReq {
   @NotEmpty(message = "姓名必填")
   @ApiModelProperty(value = "员工姓名", required = true)
   private String name;

   @Size(min = 6, max = 64, message = "手机号或工号不能为空，长度介于6~64之间")
   @ApiModelProperty(value = "手机号", required = true)
   private String telephone;
}
3）API定义业务方法时(Controller层)，使用Swagger注解@ApiOperation明确业务方法信息，样例如下：
@PostMapping("/updateCatRateLimit")
@ApiOperation("设置监控项流控QPS")
public CommonResponse<String> updateCatRateLimit(@RequestBody Req req) {
    return super.visit(() -> mcenterRateLimitService.updateCatRateLimit(req));
}
具体swagger详细接口请参考：https://swagger.io/docs/
# 后端接口输出太慢？
![流程](https://zkk-1300025204.cos.ap-nanjing.myqcloud.com/image.png)
当接到新需求，进行产品需求评审-->技术文档编写-->数据库设计-->业务梳理-->测试用例评审等等流程下来，很难在短时间内给出接口，但即便如此，我也没听说过谁会因为后端给不出接口耽误前端开发进度
为了更快速的输出接口等响应，推荐api管理工具APIpost,swagger,Yapi等~
![apipost](https://zkk-1300025204.cos.ap-nanjing.myqcloud.com/apipost.png)

# Swagger遇见Yapi
### 一 YapiUpload
1、在IDEA->Preferences->Plugins中 输入YapiUpload插件，点击Install，重启IDEA后可以使用。
![流程](https://zkk-1300025204.cos.ap-nanjing.myqcloud.com/a4a92829-a7d3-4758-ad34-6b715fd4bd05.png)
2、配置项目
在我们已经从git同步的项目，打开.idea文件夹下的misc.xml，添加如下配置。
```java
<component name="yapi">
  <option name="projectToken">yapi中获取项目token</option>
  <option name="projectId">项目ID</option>
  <option name="yapiUrl">http://mock.b2bcsx.com</option>
  <option name="projectType">api</option>
</component>
```
完整配置如下所示:
![流程](https://zkk-1300025204.cos.ap-nanjing.myqcloud.com/%E5%AE%8C%E6%95%B4%E9%85%8D%E7%BD%AE.png)
 获取配置信息:
![流程](https://zkk-1300025204.cos.ap-nanjing.myqcloud.com/%E8%8E%B7%E5%8F%96%E9%85%8D%E7%BD%AE.png)
3、接口上传 
这一步是我们日常工作经常使用的操作，在我们编写好的接口类文件中，我们只需选中类名或者选中要上传接口的方法名右键选择“UploadToYapi”，两者的区别是，选择类名会上传此类中的全部接口，选择方法名仅上传单个接口，按需选择即可。参数非空需要加入 @NotNull或@NotEmpty。如果需要将接口传入指定目录下，需要在类上添加注释

![流程](https://zkk-1300025204.cos.ap-nanjing.myqcloud.com/c6418ea5-b7e2-41de-9582-3e82e717a8a9.png)
### 批量上传
将本地或者服务器环境生成的swagger.doc/swagger.json地址复制
![上传](https://zkk-1300025204.cos.ap-nanjing.myqcloud.com/import.png)
即可批量上传至Yapi~
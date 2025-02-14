# Specification of setup.sas

本文件规定了 `setup.sas` 文件的一般规范。

`setup.sas` 文件用于初始化项目的一些变量，它只会在 SAS 会话启动时运行一次。
`setup.sas` 文件不会被递交至监管机构。

## 重置运行环境

需要重置的窗口如下：

- LOG 窗口
- OUTPUT 窗口
- ODSRESULT 窗口

```sas
dm 'log; clear; output; clear; odsresult; clear;';
```

## 重置系统自动宏变量

```sas
%let SYSCC = 0;
```

`SYSCC` 会被宏 `%ERROR` 引用，所以必须进行重置。

## 设置路径

动态获取 `setup.sas` 文件的路径

```sas
%let PATH_CURRENT = %sysget(SAS_EXECFILEPATH);
```

设置常用文件夹路径

```sas
%let PATH_PROJECT     = %substr(&PATH_CURRENT, 1, %eval(%index(&PATH_CURRENT, %scan(&PATH_CURRENT, -3, \/)) -2));
%let PATH_STAT        = &PATH_PROJECT\04 统计分析;
%let PATH_RAWDATA     = &PATH_STAT\02 原始数据;
%let PATH_ADAM        = &PATH_STAT\03 分析数据;
%let PATH_TFL         = &PATH_STAT\06 TFL;
%let PATH_TFL_TABLE   = &PATH_TFL\01 table;
%let PATH_TFL_FIGURE  = &PATH_TFL\02 figure;
%let PATH_TFL_LISTING = &PATH_TFL\03 listing;
%let PATH_TFL_OTHER   = &PATH_TFL\04 other;
%let PATH_MACRO       = &PATH_STAT\09 Macro;
%let PATH_INITIAL     = &PATH_STAT\10 Initial;
```

> [!NOTE]
>
> 可根据需要，定义更多常用路径，但不能与上述定义的变量名称冲突。

## 建立逻辑库

```sas
libname rawdata "&PATH_RAWDATA" compress = yes access = readonly;
libname adam    "&PATH_ADAM"    compress = yes;
```

> [!IMPORTANT]
> 建立 `rawdata` 逻辑库使用的 `access = readonly` 是必要的，这可以防止意外更改原始数据库中的数据。

## 调用通用宏程序

1. QC 相关的宏程序

   ```sas
   %include "&PATH_MACRO\02 QC\Transcode.sas";
   %include "&PATH_MACRO\02 QC\ReadRTF.sas";
   %include "&PATH_MACRO\02 QC\CompareRTFWithDataset.sas";
   ```

2. 日志相关的宏程序

   ```sas
   %include "&PATH_MACRO\03 Misc\SM_LOG_V2.sas";
   %include "&PATH_MACRO\03 Misc\ERROR.sas";
   ```

3. RTF 处理相关的宏程序

   ```sas
   %include "&PATH_MACRO\03 Misc\MergeRTF.sas";
   ```

> [!NOTE]
>
> 可根据需要，添加更多宏程序。

## 加载模板

```sas
%include "&PATH_INITIAL\template\template_rtf_threelines.sas";
%include "&PATH_INITIAL\template\template_graph_regression.sas";
%include "&PATH_INITIAL\template\template_graph_baplot.sas";
%include "&PATH_INITIAL\template\template_graph_waterfall.sas";
```

> [!NOTE]
> 可根据需要，添加更多模板。

## 定义 ODS 输出相关变量

1. 定义 RTF 输出目录

```sas
%let RTF_PATH_T = &PATH_TFL_TABLE;
%let RTF_PATH_F = &PATH_TFL_FIGURE;
%let RTF_PATH_L = &PATH_TFL_LISTING;
%let RTF_PATH_O = &PATH_TFL_OTHER;
```

2. 定义 ODS 转义字符

```sas
%let ODS_ESCAPECHAR = @;
```

3. 定义 RTF 页眉

```sas
%let TITLE = \pard\plain\ql\fs21{试验医疗器械：xxxxxxxx};
```

4. 定义 RTF 页脚

```sas
%let FOOTNOTE_LEFT  = Confidential Information;
%let FOOTNOTE_RIGHT = &ODS_ESCAPECHAR{pageof};
```

> [!TIP]
>
> 也可以通过 `%sysfunc(inputc(&SYSODSESCAPECHAR, $hex.))` 获得当前环境设置的 `ODS ESCAPECHAR` 的值。

## 示例

```sas
/*STEP0 : Reset runtime environment*/
dm 'log; clear; output; clear; odsresult; clear;';


/*Rest system automatic macro variable*/
%let syscc = 0;


/*STEP1 : Set the path*/
%let PATH_CURRENT = %sysget(SAS_EXECFILEPATH);

/*设置常用文件夹路径*/
%let PATH_PROJECT     = %substr(&PATH_CURRENT, 1, %eval(%index(&PATH_CURRENT, %scan(&PATH_CURRENT, -3, \/)) -2));
%let PATH_STAT        = &PATH_PROJECT\04 统计分析;
%let PATH_RAWDATA     = &PATH_STAT\02 原始数据;
%let PATH_ADAM        = &PATH_STAT\03 分析数据;
%let PATH_TFL         = &PATH_STAT\06 TFL;
%let PATH_TFL_TABLE   = &PATH_TFL\01 table;
%let PATH_TFL_FIGURE  = &PATH_TFL\02 figure;
%let PATH_TFL_LISTING = &PATH_TFL\03 listing;
%let PATH_TFL_OTHER   = &PATH_TFL\04 other;
%let PATH_MACRO       = &PATH_STAT\09 Macro;
%let PATH_INITIAL     = &PATH_STAT\10 Initial;


/*STEP2 : Establish Logical Library*/
libname rawdata "&PATH_RAWDATA" compress = yes access = readonly;
libname adam    "&PATH_ADAM"    compress = yes;


/*STEP3 : Call Macro*/
/*QC相关的宏程序*/
%include "&PATH_MACRO\02 QC\Transcode.sas";
%include "&PATH_MACRO\02 QC\ReadRTF.sas";
%include "&PATH_MACRO\02 QC\CompareRTFWithDataset.sas";

/*日志相关的宏程序*/
%include "&PATH_MACRO\03 Misc\SM_LOG_V2.sas";
%include "&PATH_MACRO\03 Misc\ERROR.sas";

/*其他宏程序*/
%include "&PATH_MACRO\03 Misc\MergeRTF.sas";


/*STEP4 : Call Style*/
%include "&PATH_INITIAL\template\tagsets_template.sas";


/*STEP5 : Configure RTF Output*/
/*定义 RTF 输出目录*/
%let RTF_PATH_T = &PATH_TFL_TABLE;
%let RTF_PATH_F = &PATH_TFL_FIGURE;
%let RTF_PATH_L = &PATH_TFL_LISTING;
%let RTF_PATH_O = &PATH_TFL_OTHER;

/*定义 ODS 转义字符*/
%let ODS_ESCAPECHAR = @;

/*定义 RTF 页眉*/
%let RTF_TITLE = \pard\plain\ql\fs21{试验医疗器械：X射线计算机体层摄影设备};

/*定义 RTF 页脚*/
%let RTF_FOOTER_L =  Confidential Information; /*页脚-左*/
%let RTF_FOOTER_R = &ODS_ESCAPECHAR{pageof}; /*页脚-右*/

/*STEP7 : Detect Error and Warning*/
%ERROR;
```

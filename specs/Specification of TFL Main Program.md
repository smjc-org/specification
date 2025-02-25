# Specification of TFL Main Program

本文规定了 TFL 主程序文件的一般规范。

## 程序头部

程序头部是一段头部注释（_head comment_），请参考 [头部注释规范](./Specification%20of%20header%20comment.md) 。

## 前置准备

包含一些初始化操作，例如：

### 重置系统自动宏变量

```sas
%let syscc=0;
```

### 清空 WORK 逻辑库中的数据集

```sas
proc datasets lib=work kill memtype=data nolist;
quit;
```

### 清空日志、输出结果、ODS 结果

```sas
dm ' log; clear; output; clear; odsresult; clear; ';
```

### 导入宏程序

```sas
%include "&DIR_MACRO\01 Main\qualify.sas";
%include "&DIR_MACRO\01 Main\quantify.sas";
```

## 程序主体

程序主体包含生成分析结果的程序代码，这些代码会被递交至监管机构。

对于生成表格的程序文件，程序主体的最终输出应是一个数据集，它可以直接被传送至 ODS 处理，进一步输出为外部 RTF 文件。

对于生成图形的程序文件，程序主体的最终输出应当直接指向外部 RTF 文件。

> [!IMPORTANT]
>
> 程序主体应使用注释 `/* SUBMIT BEGIN */` 和 `/* SUBMIT END */` 包围，便于自动化提取应当被递交至监管机构的代码片段。
> 请参考自动化提取递交代码的 Python 工具 [submit](https://github.com/smjc-org/py-submit)。

> [!CAUTION]
>
> 程序主体应当是一个可独立运行的代码片段，程序主体中不应当存在以下行为：
>
> - 操作额外的逻辑库
> - 操作额外的文件或文件引用
> - 使用未在程序主体中定义的自定义输入和输出格式
> - 使用 `setup.sas` 文件中定义的宏变量

在程序主体中，一些常用的固定步骤如下：

### 创建输出格式

由于 `setup.sas` 文件不会被递交，因此，必须在每个 TFL 主程序的主体部分单独定义输出格式，可参考 [常用输出格式](https://github.com/smjc-org/cheatlist/blob/main/src/format.md)。

### 复制数据集

有选择地创建 adam 逻辑库中部分数据地副本，防止意外操作 adam 逻辑库中地数据集。

推荐先复制 `adam.adsl`，然后再复制其他可能使用到地数据集。

```sas
proc sql noprint;
    create table adsl as select * from adam.adsl where fasfl = "Y";
    create table adae as select * from adam.adae where usubjid in (select usubjid from adsl);

    select count(usubjid) into :g1 trimmed from adsl where arm = "试验组";
    select count(usubjid) into :g2 trimmed from adsl where arm = "对照组";
quit;
```

### 创建输出数据集（Figure 不适用）

整理合并程序运行结果，为 ODS 输出 RTF 文件做准备。

```sas
data t_6_1_6;
    length item value $200;
    set t_6_1_6_part1 - t_6_1_6_part7;
run;
```

## 程序尾部

程序尾部包括：

1. 日志输出

   ```sas
   %SM_LOG;
   ```

2. RTF 输出

   ```sas
   data rptdata;
      set t_6_1_6;
   run;

   /*判断数据集是否为空（无需更改此段程序）*/
   proc sql noprint;
      select ifc(nvar = 0 or nobs = 0, "Y", "N") into : is_empty from dictionary.tables where libname = "WORK" and memname = "RPTDATA";
   quit;

   %if &is_empty = Y %then %do;
   data emptydata;
      length describ $200.;
      describ = "未发生";
   run;
   %end;

   /*设置 RTF 输出路径*/
   %let rtf_name       = 表 6.1.6 人口学及基线特征 全分析集;
   %let rtf_path       = &RTF_DIR_T\&rtf_name..rtf; /*注意根据图标类型更改目录为 &RTF_DIR_T, &RTF_DIR_F, &RTF_DIR_L*/

   /*设置 RTF 标题和脚注*/
   %let pretext = \outlinelevel0{&rtf_name}\par\pard;
   %let posttext = 注：百分比计算基于全分析集人数。;

   /*启动 ODS（无需更改此段程序）*/
   options nodate nonumber orientation = landscape;
   ods html close;
   ods rtf file = "&rtf_path" style = styles.threelines;
   ods escapechar = "@";

   /*设置页眉页脚*/
   title    j = l "&logo" j = r "&title";
   footnote j = l "&footnote_left" j = r "&footnote_right";

   /*======空集输出(无需更改此段程序)======*/
   %if &is_empty = Y %then %do;
      proc report data = emptydata split = "~" noheader
                  style(report)={protectspecialchars=on  asis=on  just=c outputwidth=100%
                                 fontfamily="Times New Roman" fontsize=10.5pt pretext="&pretext"
                                 posttext="\ql\fs21"}
                  style(header)={protectspecialchars=on asis=on just=c}
                  style(column)={protectspecialchars=on asis=on just=c}
                  style(lines) ={protectspecialchars=on asis=on};
         column describ;
         define describ / display " " style(column) = {just = c};
      run;
   %end;


   /*======非空集输出（示例，根据实际输出格式进行更改）======*/
   %if &is_empty ^= Y %then %do;
      proc report data = rptdata split = "~" missing
                  style(report)={protectspecialchars=on  asis=on  just=c outputwidth=100%
                                 fontfamily="Times New Roman" fontsize=10.5pt pretext="&pretext"
                                 posttext="\ql\fs21&posttext"}
                  style(header)={protectspecialchars=on asis=on just=c verticalalign=middle}
                  style(column)={protectspecialchars=on asis=on just=c}
                  style(lines) ={protectspecialchars=on asis=on};
         column item value;
         define item  / display "指标~    统计量"
                        style(header) = {protectspecialchars=off just=l}
                        style(column) = {protectspecialchars=off just=l};
         define value / display "合计~N=&n";
      run;
   %end;

   ods rtf close;
   ods html;
   title;
   footnote;
   ```

> [!IMPORTANT]
> 如果你的程序主体和 RTF 输出部分在同一个宏程序中定义，请在宏程序内部使用注释 `/* SUBMIT START */` 和 `/* SUBMIT END */` 定义需要被递交（或不递交）至监管机构的代码片段。

3. 错误检测

   ```sas
   %ERROR;
   ```

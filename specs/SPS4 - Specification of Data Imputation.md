# Specification of Data Imputation

- 创建日期：2025-03-21
- 更新日期：2025-05-15

本文件规定了缺失数据处理的一般规范。

## 日期缺失

典型例子：不良事件、合并用药的开始日期

需要借助一个非缺失的 _干预日期_（治疗开始日期、器械使用事件等）对开始日期进行填补。

填补规则如下：

| 缺失值   | 填补规则                                                                                                                   |
| -------- | -------------------------------------------------------------------------------------------------------------------------- |
| 日       | 如果年和月与 _干预日期_ 的年和月相同，并且结束日期在 _干预日期_ 之后或者缺失，则使用 _干预日期_ 进行填补；否则填补为 1。   |
| 日/月    | 如果年与 _干预日期_ 的年份相同，并且结束日期在 _干预日期_ 之后或者缺失，则使用 _干预日期_ 进行填补；否则填补为 1 月 1 日。 |
| 日/月/年 | 如果结束日期在 _干预日期_ 之后或者缺失，则使用 _干预日期_ 进行填补；否则不进行填补。                                       |

## 评价指标缺失

典型例子：评价指标在多个访视均有收集，但其中一个访视作为主要评价的时间点。

使用多重填补，根据数据缺失的模式（单调缺失、随机缺失）选择具体的填补方法。

主要填补步骤如下：

1. 使用 `PROC MI` 过程填补成多个完整的数据集；
2. 对多个完整的数据集分别建模，得到一组效应量（包含效应量和效应量的标准误）；
3. 使用 `PROC MIANALYZE` 过程合并效应量，得到最终的结果。

例如：在一个平行阳性对照、非劣效试验中，主要指标在基线、治疗后 1 个月、治疗后 3 个月、6 个月均有收集，其中 6 个月较基线变化值为主要指标的评价时间点，数据缺失模式为随机缺失。

参考代码如下：

```sas
/*将访视数据横向排列*/
proc sql noprint;
    create table analysis as
        select
            a.usubjid,
            a.site,
            a.arm,
            a.armn,
            b0.base as base,
            b1.aval as aval1,
            b3.aval as aval3,
            b6.aval as aval6,
        from adsl as a left join adrs(where = (ablfl = "Y"))           as b0 on a.usubjid = b0.usubjid
                       left join adrs(where = (avisit = "治疗后1个月")) as b1 on a.usubjid = b1.usubjid
                       left join adrs(where = (avisit = "治疗后3个月")) as b3 on a.usubjid = b3.usubjid
                       left join adrs(where = (avisit = "治疗后6个月")) as b6 on a.usubjid = b6.usubjid
                       ;
quit;


/*FCS 方法多重填补*/
proc mi data = analysis out = mi_out nimpute = 5 minimum = . 0 0 0 0 maximum = . 100 100 100 100 round = .  1 1 1 1;
    class arm site;
    var arm site base aval1 aval3 aval6;
    fcs reg(aval6 = arm site base aval1 aval3);
run;

data mi_out;
    set mi_out;
    chg = aval6 - base;
run;


/*用填补数据建模*/
ods output LSMeans = LSMeans Estimates = Estimates;
proc glm data = mi_out plots=none;
    class arm site;
    model chg = arm site;
    lsmeans arm /cl stderr;
    estimate "试验组 vs 对照组" arm -1 1;
    by _Imputation_;
quit;


/*合并分析结果*/
proc sort data = LSMeans;
    by arm;
run;

ods output ParameterEstimates = LSMeansPE;
proc mianalyze data = LSMeans;
    modeleffects LSMean;
    stderr stderr;
    by arm;
run;

ods output ParameterEstimates = EstimatesPE;
proc mianalyze data = Estimates;
    modeleffects Estimate;
    stderr StdErr;
run;
```

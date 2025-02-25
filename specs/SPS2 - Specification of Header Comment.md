# Specification of Header Comment

- 创建日期：2025-02-25
- 更新日期：2025-02-25

本文件规定了程序文件头部注释的一般规范。

## 字段解释

| 字段名称         | 含义                     |
| ---------------- | ------------------------ |
| Program Name     | 程序文件名称（不含路径） |
| Path             | 程序文件路径<sup>1</sup> |
| Program Language | 程序语言                 |
| Purpose          | 程序目的                 |
| Macro Calls      | 程序调用的宏程序         |
| Input            | 程序读取的数据集         |
| Output           | 程序输出的数据集         |
| Version          | 版本号                   |
| Date             | 版本日期                 |
| Author           | 作者                     |
| Description      | 描述信息                 |

> [!NOTE]
>
> 1. 建议以项目根目录为起点，使用相对路径。例如：
>    - 实际路径：`D:\OneDrive\统计部\项目\MD\2023\04 X射线计算机体层摄影设备\04 统计分析\04 ADS程序\01 主程序\adsl.sas`
>    - 起点路径：`D:\OneDrive\统计部\项目\MD\2023\04 X射线计算机体层摄影设备\`
>    - 相对路径：`04 统计分析\04 ADS程序\01 主程序\adsl.sas`

## 示例

```sas
/*
 * Program Name          : adsl.sas
 * Path                  : 04 统计分析\04 ADS程序\01 主程序\adsl.sas
 * Program Language      : SAS V9.4
 * ==========================================================
 * Purpose               : create adsl.sas7bdat
 * Macro Calls           : %sm_log, %error, %ads_import_excel
 * Input                 : info, ie, ct, iqe, ds
 * Output                : adsl.sas7bdat
 * ==========================================================
 * Version          Date          Author          Description
 * 1.0              2025-02-17    xxxx            创建新程序
*/
```

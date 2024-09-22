# 亚稳态：Metastability
亚稳态指的是：在某一段时间周期之内，信号处于0和1的不定态。在多时钟源设计内，亚稳态不能被消除，只可能被避免。   
![image](https://github.com/user-attachments/assets/d16814e9-884d-435f-bf01-5a50a7d3ee7b)
图1： 同步失败的例子。  BCLK采样的时候，adata正在变化。采样到的信号是亚稳态的，它应该在下一个posedge bclk拉高但是没有。  

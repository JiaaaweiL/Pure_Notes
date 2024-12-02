# RTL LOGIC DESIGN
### Pipeline 相关内容，有那些hazard，pipeline有什么好处，坏处，为什么不用超级多层pipeline？
Hazard 相关：Data Hazard，Control Hazard, Structural Hazard     

- Data Hazard(AKA Data dependency). There are three kinds of Data dependency: Read after write, Write after write, Write after read. RAW is TRUE Dependency, WAR and WAW are false dependencies (can be solved by register renaming).   
- Control Hazard(AKA Branch Hazard). When there is a branch miss predict, we need to flush the pipeline and execute the correct instruction.  
- Structural Hazard: Many instructions trying to access the same hardware resource at the same clock cycle.  

Pipeline的好处：  
- boost the throughput, and increase the utility of the hardware resource. 
- The critical path will be shorter. Thus, the pipeline can boost the frequency (and performance).

Pipeline的坏处： 
- A deeper pipeline will increase the penalty when there is a branch miss-predict.
- A deeper pipeline will increase the power and area (more stage means more Dff), and the design complexity increase.

### Branch predictor/BTB 设计
- 最简单的 bimodal predictor

用pc的部分值去index一个saturating counter 最简单的做法，但是会被aliasing 困扰（两个不同的pc index 到同一个saturating counter，但是branch的方向不同，造成永远不饱和，叫destructive aliasing） 
变体之一是用哈希代替直接从PC中间取值 可以减少这种情况。  

- 基于局部历史的分支预测 local branch history   

上述的方法无法处理TNTNTNT,改进方法是是用一个branch history register hold每一个pc历史的分支数据，然后用这个历史状态来index saturating counter     
改进1是不可能为每一个分支指令配2一个BHR和PHT，因此是用PC的一部分来寻址PHT，相当于一部分PC要共用一个BRH。可以是pc哈希+pc部分位来index， 也可也是BHR+PC的位拼接    

- 基于全局历史的分支预测global branch history

解决的问题是： 一些分支指令的结果依赖于前面的分支指令，然后就用GHR global history register index PHTs 的entry。具体选择哪一个PHT，用PC做哈希决定。     
如果这种方法有很多的saturating counter, warm-up time(training time)过长，存储的bit过多，一个简单的方法是用一个PHT，然后用GHR+PC hash的位拼接。   
其实跟Local branch history很像。 

- BTB: 一个cache，一般是direct mapping或者少way的set associative BTB。将发生跳转的分支指令对应的目标指令放在BTB中。对pc-relative的指令，BTB的目标地址预测是准确的，但是对于间接分支，跳转不准确。  

 



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

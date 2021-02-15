# OpermBot
`Open + Permission + Bot = OpermBot`  
Largely inspried by [Shizuku](https://github.com/RikkaApps/Shizuku).  
`Shizuku`是一个安卓的超级权限管理器,通过开放系统的Java API使得普通应用不需要`Root`权限也能执行某些操作.

受到启发, `OpermBot`就此诞生.  
`OpermBot`意在解决直接给Bot赋予管理员权限带来的一系列问题, 如意外滥权, 或者管理员人数过多超过限制等等.  
使用`OpermBot`, 群内只需拥有一个机器人管理员, 其他所有需要管理权限的Bot通过与`OpermBot API`交互完成所需操作, 群主无需再为其余任何Bot赋予管理员权限.  

通过`OpermBot`的权限管理, 群主只需赋予机器人必要的权限, 而不用给予完整的管理员权限, 极大的优化了机器人的权限结构, 大幅降低了意外操作的可能性, 特别是在群内有多个不同功能的Bot时效果显著.  
例如, 某个Bot需要修改群名称以实现Tag功能, 通过`OpermBot API`, 这个Bot不再需要完整的管理员权限, 仅需要赋予修改群信息的权限即可, 这个Bot除了能修改群名片以外, 权限都与普通群成员一致.  

`OpermBot API`基于`HTTP`或`Websocket`实现,保证其易用性和简易性.

---------
### 题外话
目前来说, 这个项目只是一个构想, 但这很明显, 对于Bot特别多的群是一种刚需.  
集中并粒度化的管理权限, 弥补了QQ本身权限机制的不足.  
即使没有很多机器人,对于某些强迫症来说也是福音.  
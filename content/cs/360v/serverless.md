---
draft: true
title: 

params: 
    desc: 
    author: Andrew Nguyen 
---



Serverless functions are on-demand. This gives it a leg up over other solutions like containers because if traffic is irregular and bursty, charges will only realistically incur during those periods. They also functionally automatically scale and are stateless. 

Any function can be made serverless, but should it? Such a function should be quick and fundamentally unreliant on state. Then save the environment in a container image. Now, when the serverless function is triggered, the environment is started up. 

Reinitializing every time is slow, though. Therefore, after the first initialization, snapshot the environment into a checkpoint. Instead of reinitializing the environment, load the checkpoint.
Wasn't able to test it out, but it seems that it's purpose is to save on memory in situation:
- where we spawn more than one actor
- and lots of the variables are the same in all instances of actor, so redundancy is huge
- those variables are meant as init values
- they most probably won't change their value in runtime
Creating copies of these variables for each actor instance is just a waste of memory at this moment. 

Documentation: 
https://dev.epicgames.com/documentation/en-us/unreal-engine/sparse-class-data-in-unreal-engine

For some game features it is a good idea to use asynchronous tracing. There might be no benefit to stopping whole frame to fire over 9000 traces for interaction system to analyze. Such stuff could and probably should be done on another thread asynchronously.

### Traces: 

`World->QueryTraceData(Handle, TraceDatum);`  
`World->AsyncLineTraceByChannel();`

### Overlaps:

`World->QueryOverlapData(Handle, TraceDatum);`  
`World->AsyncOverlapByChannel();`

### Two ways of using it
We can either use `World->AsyncLineTraceByChannel(...);`/ `World->AsyncOverlapByChannel(...);` :
 - and use `FTraceDelegate` that will be called once this trace finishes processing on it's thread
 - or use `FTraceHandle` that we receive from these functions and in next frame query the World via `World->QueryTraceData` / `World->QueryOverlapData` and see if the results are ready to be used


### Network consideration:
- if the game runs as a single player game - yes we can use it easily
- if the game runs in multiplayer with `ListenServer`:
	- if Async traces/overlaps are connected to things that each `Client` is processing locally - no problem there as well
	- if it is `ListenServer` that is running this async trace processing, makes deduction and sends results of deduction to other `Clients` - we need to be aware that `Game Host` machine will be burdened with more processing than other `Clients` 
- if it is `Dedicated Server` based game 
	- it makes sense to use Async Traces for local `Client` processing needs i.e. cosmetics or local tracing that we would send results of to `Dedicated Server` 
	- it doesn't makes sense to use Async Traces on `Dedicated Server` itself - as usually it runs on a single thread on a machine in the cloud. It is being done that way to reduce cost for renting machines in the cloud via launching couple of `Dedicated Server` instances on singular machine - each instance running on it's own singular thread on singular machine.
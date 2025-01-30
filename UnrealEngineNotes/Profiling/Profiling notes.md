All of my personal notes about profiling. They are either covering things that are not in documentation or my personal thoughts about use cases or useful links to remind myself of etc.

# Unreal Insights

- To ensure that we have more smooth frame generation on development builds we can use these commands:
	- `-nosound`  - no sound processing impacting frame
	- `-noailogging`  - when we're not profiling *AI* - logging of *AI* won't stall our frame
	- `-noverifygc` - when we're not profiling *Garbage Collection* - make sure that process of *GC* verification won't trigger and hitch our frames
	- `t.maxfps 0`  - make sure we don't restrict maximum FPS
	- `r.vsync 0` - make sure that *Vsync* doesn't impact frame generation so we can look at actual problems instead of everything slowing down, because of *Vsync* being on
- to make sure our exe starts with specific commands that are supposed to be triggered when launching build to profile - add this to exe shortcut:
	- `-execcmds="r.vsync 0"`
- The more `-trace=` params we use, the bigger the trace file! Be mindful of what are you profiling. Not everything is needed at once usually.
- `stat namedevents` - even more details about events when tracing (increases trace file size as well)
- `trace.bookmark <name>` - typed in console of a game, while tracing - emits bookmark with custom name. I.E. `trace.bookmark BossKilled` might be useful to mark specific situation if we're wanna see what happens after killing boss
- `trace.screenshot <name>`  - typed in console of a game, while tracing - creates screenshot of given situation - helpful when you wanna see what was actually happening on the screen at painful performance hitching moment
- `r.screenpercentage 10` - can reduce amount of graphics processing if we wanna concentrate on CPU or see if some graphical issue is screen percentage dependant
- `pause` - pauses the game - might be useful in specific edge cases


### Setting up code for Insights profiling

- `TRACE_BOOKMARK(TEXT("Load started - %s"), *SomeActor->GetName());` - example of how to emit trace bookmark from code if we wanna mark some specific event
- `SCOPED_NAMED_EVENT(Name, Color)` - this will make given scope appear in profiling window. If used at the beginning of given function - it will take scope of the function for consideration. If you wanna profile very specific part of given code, we can use `{ }` to mark scope we wanna see in trace file
- `SCOPED_NAMED_EVENT_FSTRING(Name, Color)` - same as above, but uses FString
- `DECLARE_SCOPE_CYCLE_COUNTER(DisplayName, StatName, StatGroup)`
- 


# In Game profiling commands
- `viewmode` - when typed in build console let's you changes view modes, just like clicking them in the editor windows does
	- `viewmode lit` - final result of our scene
	- `viewmode unlit` - no lightning
	- `viewmode wireframe` 
		- cyan color - objects with *Mobility* set to *Static*
		- violet color - objects with *Mobility* set to *Movable*
		- green color - objects with simulated physics
	- `viewmode lightcomplexity` - shows the number of non-static lights affecting your geometry
		- black - no light affects given surface
		- going from green to red - 1 or more than 5 light sources are affecting given surface
	- `viewmode shadercomplexity` - quickly establish which parts of the scene are most expensive shader wise
	- `viewmode CollisionPawn` - shows us which objects collides with *Pawn*/*Character*
	- `viewmode CollisionVisibility` - useful for checking which Actors are blocking *Visibility*
	- `viewmode LODColoration` - shows which LOD is being used now
	- `viewmode hlodcoloration` - displays the Hierarchial LOD Cluster index of a primitive
- `abtest` - awesome if we want to test and measure performance differences between two console variables values and see witch value is faster or preetier etc.
	- i.e. `abtest r.nanite.MaxPixelsPerEdge 1 4`
	- to make it stop - `abtest stop`
- `stat`
	- `stat detailed` - calls `stat fps`, `stat unit`, `stat unitgraph`
	- `stat uobjects`
	- `stat scenerendering`
	- `stat game`
	- `stat drawcount`
	- `stat gpu`
	- `stat anim`
	- `stat dumphitches` - dumps slow frames to log with callstack of the problematic functions
	- `stat slow -ms=1.0 -maxdepth=6` - creates a hierarchy of slow functions in the frame. We can specify what we consider slow and how deep should the hierarchy go
- `showlog` - opens log in a seperate window


# CPU


# GPU

### Useful console commands
- `profilegpu` 
	- in editor - launches separate debug window for frame profiling
	- in cooked build - dumps current frame to log
- `profilegpuhitches` - triggers `profilegpu`  each time hitch happens
- `rhi.GPUHitchThreshold 100` - establishes threshold for GPU hitches at 100 milliseconds
- `r.ProfileGPU.Pattern *AmbientOcclusion*` - allows for filtering only specific Pattern i.e. AmbientOcclusion


# Garbage Collection

-  There is no sense in profiling GC operations in development builds for GC Hitches
	- GC stages and operations run unoptimized versions of themselves in development builds
	- Their timings will be completely different than in Shipping or Test builds
- We can log which objects are creating most garbage, when via using interface `FUObject:Array::FUObjectDeleteListener` on `GameInstance` 
	- it is a good practise to put inheritence from this in `#if !UE_BUILD_SHIPPING` macro - to make sure that none of this logging goes into shipping build
	- the same way all the overriden functions should be put in `!UE_BUILD_SHIPPING` macro
	- we need to make sure that we initialize our Listener:
	  `void UMyGameInstance::Init() 
	  `{
		  `Super::Init();
	  `#if !UE_BUILD_SHIPPING 
		  `GUObjectArray.AddUObjectDeleteListener(this); 
	  `#endif 
	  `}`
	- we also need to remove listener on game instance shutdown:
		- `void UMyGameInstance::Shutdown()` 
		  `{`
		   `#if !UE_BUILD_SHIPPING` 
			   `GUObjectArray.RemoveUObjectDeleteListener(this);` 
		   `#endif` 
		   `}`
	- than we can override `UMyGameInstance::NotifyUObjectDeleted(const UObjectBase* Object, int32 IdxInGUObjectArray)` and start doing stuff with info
		- we should `const UObject* GarbageObject = (UObject*)Object;`
		- we can check for flags whether something is `EInternalObjectFlags::Garbage`, `UE::GC::GUnreachableObjectFlag`, `UE::GC::GMaybeUnreachableObjectFlag` via `GarbageObj->HasAnyInternalFlags` 
		- we can cast `GarbageObject` to any other Component, Actor or UObject and get more information about collected object
		- we can use `UE_LOG` with `LogGarbage` log channel and `VeryVerbose` so it will not spam log all the time
		- if our code doesn't compile make sure that you have empty implementation of `void UmyGameInstance::OnUObjectArrayShutdown() {}`
		- `log loggarbage veryverbose` - will log garbage channel with very verbose
		- the log will be spamming during play so it is a good idea to use filter for our `UE_LOG` debug string that we used
		- good idea is to log out names of assets: Materials, Anims, Object names, Sounds etc. to know what to search for in Reference Viewer
# Other usefule exe params
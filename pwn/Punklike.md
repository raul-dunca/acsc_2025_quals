This challenge was a cool game, in which I had some cards I could use. To get the flag it seems that I had to call `flag_door_func(int key)` with key=1337. Looking at the available cards. I could `pop_rdi` (right), `pop_rsi` (left), `add rsi to rdi` (tape), `multiply rdi with rsi` (fuse) and I had different int values (like 16 (pistol) 9 (knife), etc.). Thus, I started playing and tried to make `rdi` somehow equal to 1337. When calling a function (e.g.: `flag_door_func`) the parameter (key) is taken from `rdi` (since this was a 64-bit executable). The problem when playing the game was that each action had a cost, so I couldn't get to 1337 using only addition. My approach was to `pop_rdi` then nuke (put 1024 into `rdi` because 1024 will be at the top of the stack) then `pop_rsi` and use machine gun (put 64 in `rsi`) then use tape to add them => `rdi`=1080. So I tried to add until 1337, but as I said, I couldn't because of the cost. Next I was thinking that I had to use `attack_func`, this functions basically saves what is in `edi` (the lower 32 bits of `rdi`) into the variable `damage_pts` and `combo_func` adds what is in `edi` to the variable `damage_pts` and then somehow load the value of `damage_pts` into `rdi`. Later I realized I could calculate 1337 as ((64+16)*16+16+16+16+9) and by fighting against "Gang Grunt" which allows a total cost of 100 I was able to get the flag. This is the sequence of the cards played and the total cost was 94:

```txt
 right                              
 machine gun                    #rdi=64            
 left                               
 pistol                         #rsi=16    
 tape                           #rdi=rdi+rsi=80
 left                               
 pistol                         #rsi=16    
 fuse                           #rdi=rdi*rsi=80*16= 1280
 left                               
 pistol                         #rsi=16    
 tape                           #rdi=1280+16=1296    
 left                               
 pistol                         #rsi=16    
 tape                           #rdi=1296+16=1312  
 left 
 pistol                         #rsi=16    
 tape               	        #rdi=1312+16=1328                
 left                               
 knife                          #rsi=9   
 tape                           #rdi=1328+9=1337  
 secret                         #run the flag_door_func with key=1337
```

Note: I used pwngdp for debugging and I put a breakpoint at `flag_door_func` to check the value of `rdi` after a run. Also, I had to reput the values of `rsi` again to 16 because the `run_turn` functions resets it after each turn.


`dach2025{Ar3_y0u_r3ad7_4_n3w_g4m3+_h11l96jxxxon35ma}`

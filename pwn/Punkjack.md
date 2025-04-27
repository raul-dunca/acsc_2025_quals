After analyzing the source code and interacting with the application a bit, I notice something strange. If the first thing I do is to select the `delete data` option, my username becomes gibberish (a sequence of random bytes) and my coins are set to 5. It seems that the user (the instance) is freed but still used (use after free vulnerability). My guess is that the heap reuses the same memory for other things. This is confirmed because if now I play a game and no matter if I win or lose it, my balance gets around 10.000.000, so the value gets "updated"/actually reused. 

This amount of coins is still not enough to buy the flag, but there is another trick. The `kidney_count` variable is not reset when selecting `delete data`, thus it is possible to buy kidneys and repeat the first step. Essentially, it is possible to generate an infinite number of kidneys, then sell them to be able to afford the flag.

`dach2025{1_h0p3_y0u_d1dnt_f0rg3t_t0_buy_b4ck_y0ur_k1dn3y_6ednuwv7n7kvo7j0}`

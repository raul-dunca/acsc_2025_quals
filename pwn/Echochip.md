In this challenge, a format string vulnerability was needed to be exploited. The vulnerability was due to: `printf(echo);` where user input is directly used in `printf` as the first parameter (the format string parameter). At first, I thought that I must write the address of the flag on the stack and then use a `%s` specifier to read the string from that address. However, the script that led to the solution was:

```bash
 for i in $(seq 1 300); do
  echo "trying %${i}\$s"
  echo "AAA %${i}\$s" | nc <ip> <port>
done > a.txt

```

This scrip basically prints different position from the stack as strings (`%n$s` means: use the n-th argument as a string and print it) and in the output I was able to see that various environment variables were leaked, including the `FLAG` environment variable (at `%150$s`).

`dach2025{d0es_th1s_m4ke_my_v01ce_s0und_deep3r_zdogk7vjs4qqolyg}`

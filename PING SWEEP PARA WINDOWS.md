
Host discovery via ping para Windows CMD

```
for /L %a in (1,1,254) do @start /b ping 10.10.10.%a -w 100 -n 2 >nul

arp -a
```

# Performance testing

Was looking for ways to benchmark diferent zfs configuration, came across [this](https://www.youtube.com/watch?v=fo3EGuCszpE) video

## Write testing with DD

Simple script:

```bash
#!/bin/bash
#sync; dd if=/dev/zero of=/root/tempfile bs=5M count=1024; sync
time sh -c "dd if=/dev/zero of=/root/testfile bs=1900k count=1k && sync"
rm /root/testfile
```

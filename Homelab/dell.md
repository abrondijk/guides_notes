# Dell tips
## Fix high fan speed when using non-dell pci cards
 **Note: non-dell also includes NVIDIA**
 
```
ipmitool raw 0x30 0xce 0x00 0x16 0x05 0x00 0x00 0x00 0x05 0x00 0x01 0x00 0x00
```

## STL files
### Dell R7xx caddie
 - 2.5\" 
   - [https://www.thingiverse.com/thing:3711727]()

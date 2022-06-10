
# Changing LVM size

```
* parted /dev/sde
* print (fix if needed)
* resizepart 3 # <- this depends
* <input size> (eg `100%`)
* quit
* pvresize /dev/sda3 # <- this depends
* lvresize -l +100%FREE $(sudo lvdisplay -c | awk -F':' '{print $1}' | tr -d ' ')
* resize2fs $(sudo lvdisplay -c | awk -F':' '{print $1}' | tr -d ' ')
* lvextend -r -l100%free /dev/mapper/ubuntu--vg-ubuntu--lv
```

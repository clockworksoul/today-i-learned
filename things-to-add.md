# Stuff I Want To Add (So I Don't Forget)

* [Creating units in systemd](https://www.freedesktop.org/software/systemd/man/systemd.unit.html) [also this](https://medium.com/@benmorel/creating-a-linux-service-with-systemd-611b5c8b91d6) (May 8) 
* [The Best Chocolate Chip Cookie Recipe Ever](https://joyfoodsunshine.com/the-most-amazing-chocolate-chip-cookies/) (May 9)
* Bash one-liner to calculate the md5sum of a path in Linux (May 7)
```bash
    tar --exclude='.*' -cf - "$SOME_DIR" 2> /dev/null | md5sum | awk '{print $1}'
```

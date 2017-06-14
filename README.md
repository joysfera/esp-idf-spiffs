SPIFFS for ESP-IDF
==================

Just found some bits and put them together successfully:

spiffs files are from https://github.com/whitecatboard/Lua-RTOS-ESP32/tree/master/components/spiffs

spiffs_vfs is from https://github.com/nkolban/esp32-snippets/tree/master/vfs/spiffs

My changes
----------
* made buildable as an esp-idf component
* fixed ulink() -> unlink()
* fixed return value of unlink()
* implemented lseek(), stat() and fstat()
* removed leading '/' from all filenames

Usage
-----

1. place all files in spiffs folder under components folder in your project
2. mount spiffs filesystem
3. register spiffs VFS at a mount point
4. enjoy using standard file calls

Example
-------
```
    // mount SPIFFS under /data/
    spiffs_registerVFS("/data", &fs);

    // create a file and write some text into it
    FILE *file = fopen("/data/my_file", "w");
    if (file != NULL) {
        const char text[] = "some text";
        fwrite(text, strlen(text), 1, file);
        fclose(file);
    }
```

Author
------
petr@pstehlik.cz

spiffs for esp-idf
==================

spiffs is from https://github.com/whitecatboard/Lua-RTOS-ESP32/tree/master/components/spiffs

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

1. place the spiffs folder under components folder
2. mount spiffs filesystem
3. register spiffs VFS at a mount point
4. enjoy using standard stdio and unistd file calls 

Example
-------
```
    spiffs_registerVFS("/data", &fs);
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

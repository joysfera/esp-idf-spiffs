SPIFFS for ESP-IDF
==================

Reinventing the wheel
---------------------
Loboris just contacted me to tell me about his project that does the same
thing but it's about three weeks older than my trial, and is more advanced:

https://github.com/loboris/ESP32_spiffs_example

https://www.esp32.com/viewtopic.php?f=18&t=1901 (Forum)

It makes sense to abandon my project and switch to his, if it indeed works better.

Original README below
---------------------

Just found some bits and put them together successfully:

spiffs files are from https://github.com/whitecatboard/Lua-RTOS-ESP32/tree/master/components/spiffs

spiffs_vfs is from https://github.com/nkolban/esp32-snippets/tree/master/vfs/spiffs

mounting code is from Neil Kolban's ESP32 book.

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
#include "stdio.h"
#include "fcntl.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"
#include "spiffs.h"
#include "esp_spiffs.h"
#include "spiffs_vfs.h"

spiffs fs;
int mount()
{
    #define LOG_PAGE_SIZE 256
    static uint8_t spiffs_work_buf[LOG_PAGE_SIZE*2];
    static uint8_t spiffs_fds[32*sizeof(uint32_t)];
    static uint8_t spiffs_cache_buf[(LOG_PAGE_SIZE+32)*4];
    spiffs_config cfg;
    cfg.phys_size = 512*1024;                      // use 512K
    cfg.phys_addr = 2*1024*1024 - cfg.phys_size;   // start spiffs at 2MB - 512K
    cfg.phys_erase_block = 65536;                  // according to datasheet
    cfg.log_block_size = 65536;                    // let us not complicate things
    cfg.log_page_size = LOG_PAGE_SIZE;             // as we said
    cfg.hal_read_f = esp32_spi_flash_read;
    cfg.hal_write_f = esp32_spi_flash_write;
    cfg.hal_erase_f = esp32_spi_flash_erase;
    int res = SPIFFS_mount(&fs, &cfg, spiffs_work_buf, spiffs_fds, sizeof(spiffs_fds), spiffs_cache_buf, sizeof(spiffs_cache_buf), 0);
    return res;
}

int init_fs()
{
    int res = mount();
    if (res == SPIFFS_ERR_NOT_A_FS) {
        printf("Formatting...\n");
        int f = SPIFFS_format(&fs);
        printf("Result of formatting: %d\n", f);
        res = mount();
    }
    return res;
}

int app_main()
{
    // mount filesystem
    init_fs();

    // mount SPIFFS under /data/
    spiffs_registerVFS("/data", &fs);

    // create a file and write some text into it
    FILE *file = fopen("/data/my_file", "w");
    if (file != NULL) {
        const char text[] = "some text";
        fwrite(text, strlen(text), 1, file);
        fclose(file);
    }
    return 0;
}
```

Author
------
petr@pstehlik.cz

0. What is zlog?
-------------

zlog is a reliable, high-performance, thread safe, flexible, clear-model, pure C logging library.

  Actually, in the C world there was NO good logging library for applications like logback in java or log4cxx in c++. Using printf can work, but can not be redirected or reformatted easily. syslog is slow and is designed for system use. 
  So I wrote zlog. 
  It is faster, safer and more powerful than log4c. So it can be widely used. 

1. Install
-------------

Downloads: https://github.com/HardySimpson/zlog/releases

    $ tar -zxvf zlog-latest-stable.tar.gz
    $ cd zlog-latest-stable/
    $ make 
    $ sudo make install
    
or

    $ make PREFIX=/usr/local/
    $ sudo make PREFIX=/usr/local/ install

PREFIX indicates the installation destination for zlog. After installation, refresh your dynamic linker to make sure your program can find zlog library.  

    $ sudo vi /etc/ld.so.conf
    /usr/local/lib
    $ sudo ldconfig

Before running a real program, make sure libzlog.so is in the directory where the system's dynamic lib loader can find it. The command metioned above are for linux. Other systems will need a similar set of actions.


2. Introduce configure file
-------------

There are 3 important concepts in zlog: categories, formats and rules.

Categories specify different kinds of log entries. In the zlog source code, category is a (zlog_cateogory_t *) variable. In your program, different categories for the log entries will distinguish them from each other.

Formats describe log patterns, such as: with or without time stamp, source file, source line.

Rules consist of category, level, output file (or other channel) and format. In brief, if the category string in a rule in the configuration file equals the name of a category variable in the source, then they match. Still there is complex match range of category. Rule decouples variable conditions. For example, log4j must specify a level for each logger(or inherit from father logger). That's not convenient when each grade of logger has its own level for output(child logger output at the level of debug, when father logger output at the level of error)

Now create a configuration file. The function zlog_init takes the files path as its only argument.
    $ cat /etc/zlog.conf

    [formats]
    simple = "%m%n"
    [rules]
    my_cat.DEBUG    >stdout; simple

In the configuration file log messages in the category "my_cat" and a level of DEBUG or higher are output to standard output, with the format of simple(%m - usermessage %n - newline). If you want to direct out to a file and limit the files maximum size, use this configuration

    my_cat.DEBUG            "/var/log/aa.log", 1M; simple

3. Using zlog API in C source file
-------------
	$ vi test_hello.c

    #include <stdio.h> 

    #include "zlog.h"

    int main(int argc, char** argv)
    {
    	int rc;
    	zlog_category_t *c;

    	rc = zlog_init("/etc/zlog.conf");
    	if (rc) {
    		printf("init failed\n");
    		return -1;
    	}

    	c = zlog_get_category("my_cat");
    	if (!c) {
    		printf("get cat fail\n");
    		zlog_fini();
    		return -2;
    	}

    	zlog_info(c, "hello, zlog");

    	zlog_fini();

    	return 0;
    } 

4. Compile, and run it!
-------------
    $ cc -c -o test_hello.o test_hello.c -I/usr/local/include
    $ cc -o test_hello test_hello.o -L/usr/local/lib -lzlog -lpthread
    $ ./test_hello
    hello, zlog

5. Advanced Usage
-------------
 *  syslog model, better than log4j model
 *  log format customization
 *  multiple output destinations including static file path, dynamic file path, stdout, stderr, syslog, user-defined output
 *  runtime manually or automatically refresh configure(safely)
 *  high-performance, 250'000 logs/second on my laptop, about 1000 times faster than syslog(3) with rsyslogd
 *  user-defined log level
 *  thread-safe and process-safe log file rotation
 *  microsecond accuracy
 *  dzlog, a default category log API for easy use
 *  MDC, a log4j style key-value map
 *  self debuggable, can output zlog's self debug&error log at runtime
 *  No external dependencies, just based on a POSIX system and a C99 compliant vsnprintf.

6.Links:
-------------
 * Homepage: http://hardysimpson.github.com/zlog
 * Downloads: https://github.com/HardySimpson/zlog/releases
 * Author's Email: HardySimpson1984@gmail.com
 * auto tools version: https://github.com/bmanojlovic/zlog
 * cmake verion: https://github.com/lisongmin/zlog
 * windows version: https://github.com/lopsd07/WinZlog

7.CPP Stream Output and colorful stdout

example:
````c++
#include <iostream>
#include "zlog.h"

using namespace std;

int main()
{
    if(zlog_init("zlog.conf") < 0)
    {
        cout << "init zlog failed" << endl;
        return -1;
    }

    zlog_category_t *tag = zlog_get_category("test");

    zlog_debug(tag, "debug msg");
    zlog_info(tag, "info msg");
    zlog_warn(tag, "warn msg");
    zlog_error(tag, "error msg");

    const string stag("LOG");
    ZLOGD(stag) << "debug msg: " << stag;
    ZLOGI(stag) << "info msg: " << stag;
    ZLOGN(stag) << "notice msg: " << stag;
    ZLOGW(stag) << "warn msg: " << stag;
    ZLOGE(stag) << "error msg: " << stag;
    ZLOGF(stag) << "fatal msg: " << stag;
    zlog_fini();
    return 0;
}
````
output:
![output](output.png)


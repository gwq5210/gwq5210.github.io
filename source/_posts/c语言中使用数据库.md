title: c语言中使用数据库
date: 2016-01-23 20:21:40
tags: [数据库, c语言, mysql]
categories: [编程, mysql]
---

## 安装
首先，要安装mysql数据库，安装好后，可能还不能使用mysql的c语言库，还要安装mysql的库，命令如下：sudo apt-get install libmysql++-dev。
安装好后，就可以包含<mysql/mysql.h>头文件，然后使用mysql的c语言api了。

## 数据结构
1）MYSQL
这个结构体代表一个数据库连接的句柄，它被大多数的mysql函数使用。不要尝试拷贝MYSQL结构体，不保证这样的拷贝是可以使用的。
MYSQL的定义如下：

<!--more-->

```
typedef struct st_mysql
{
  NET       net;            /* Communication parameters */
  unsigned char *connector_fd;      /* ConnectorFd for SSL */
  char      *host,*user,*passwd,*unix_socket,*server_version,*host_info;
  char          *info, *db;
  struct charset_info_st *charset;
  MYSQL_FIELD   *fields;
  MEM_ROOT  field_alloc;
  my_ulonglong affected_rows;
  my_ulonglong insert_id;       /* id if insert on table with NEXTNR */
  my_ulonglong extra_info;      /* Not used */
  unsigned long thread_id;      /* Id for connection in server */
  unsigned long packet_length;
  unsigned int  port;
  unsigned long client_flag,server_capabilities;
  unsigned int  protocol_version;
  unsigned int  field_count;
  unsigned int  server_status;
  unsigned int  server_language;
  unsigned int  warning_count;
  struct st_mysql_options options;
  enum mysql_status status;
  my_bool   free_me;        /* If free in mysql_close */
  my_bool   reconnect;      /* set to 1 if automatic reconnect */

  /* session-wide random string */
  char          scramble[SCRAMBLE_LENGTH+1];
  my_bool unused1;
  void *unused2, *unused3, *unused4, *unused5;

  LIST  *stmts;                     /* list of all statements */
  const struct st_mysql_methods *methods;
  void *thd;
  /*
    Points to boolean flag in MYSQL_RES  or MYSQL_STMT. We set this flag 
    from mysql_stmt_close if close had to cancel result set of this object.
  */
  my_bool *unbuffered_fetch_owner;
  /* needed for embedded server - no net buffer to store the 'info' */
  char *info_buffer;
  void *extension;
} MYSQL;
```

2）MYSQL_RES
这个结构体代表查询结果的所有列（SELECT，SHOW，DESCRIBE，EXPLAIN）。查询返回的结果叫做数据集，可以在后边使用。
MYSQL_RES的定义如下：
```
typedef struct st_mysql_res {
  my_ulonglong  row_count;
  MYSQL_FIELD   *fields;
  MYSQL_DATA    *data;
  MYSQL_ROWS    *data_cursor;
  unsigned long *lengths;       /* column lengths of current row */
  MYSQL     *handle;        /* for unbuffered reads */
  const struct st_mysql_methods *methods;
  MYSQL_ROW row;            /* If unbuffered read */
  MYSQL_ROW current_row;        /* buffer to current row */
  MEM_ROOT  field_alloc;
  unsigned int  field_count, current_field;
  my_bool   eof;            /* Used by mysql_fetch_row */
  /* mysql_stmt_close() had to cancel this result */
  my_bool       unbuffered_fetch_cancelled;  
  void *extension;
} MYSQL_RES;
```

3，MYSQL_ROW
用来表示查询出来的一行，可以通过mysql_fetch_row()函数获得。
MYSQL_ROW的定义如下：
```
typedef char **MYSQL_ROW;       /* return data as array of strings */
```

4，MYSQL_FIELD
用来表示数据库中的字段。包含的信息有字段名，类型，大小等。可以通过mysql_fetch_field()函数获得。
MYSQL_FIELD的定义如下：
```
typedef struct st_mysql_field {
  char *name;                 /* Name of column */
  char *org_name;             /* Original column name, if an alias */
  char *table;                /* Table of column if column was a field */
  char *org_table;            /* Org table name, if table was an alias */
  char *db;                   /* Database for table */
  char *catalog;          /* Catalog for table */
  char *def;                  /* Default value (set by mysql_list_fields) */
  unsigned long length;       /* Width of column (create length) */
  unsigned long max_length;   /* Max width for selected set */
  unsigned int name_length;
  unsigned int org_name_length;
  unsigned int table_length;
  unsigned int org_table_length;
  unsigned int db_length;
  unsigned int catalog_length;
  unsigned int def_length;
  unsigned int flags;         /* Div flags */
  unsigned int decimals;      /* Number of decimals in field */
  unsigned int charsetnr;     /* Character set */
  enum enum_field_types type; /* Type of field. See mysql_com.h for types */
  void *extension;
} MYSQL_FIELD;
```

## 常用函数
1）mysql_init
头文件：
```
#include <mysql/mysql.h>
```
函数原型为：
```
MYSQL *mysql_init(MYSQL *mysql);
```
在连接数据库之前，必须调用mysql_init函数来初始化一个MYSQL结构体。这个结构体可以使用mysql_real_connect()函数连接数据库。如果mysql是NULL，那么函数初始化一个新的结构体，分配新的内存空间，并返回内存空间的地址，如果不是NULL，则初始化mysql指向的内存，返回传进来的地址。函数执行成功返回MYSQL结构体的指针，否则返回NULL。

2）mysql_real_connect
连接数据库需要设置相关的参数，可以使用这个函数来完成，并进行实际的连接。原型如下：
```
MYSQL *mysql_real_connect(MYSQL *mysql, const char *host,
            const char *user,
            const char *passwd,
            const char *db,
            unsigned int port,
            const char *unix_socket,
            unsigned long clientflag);
```

需要包含头文件：
```
#include <mysql/mysql.h>
```

来说明参数的含义:
 - mysql是使用mysql_init()函数初始化的结构体；
 - host是mysql服务器所在的服务器计算机名或者ip地址，如果host是NULL或者是字符串"localhost"，那么就连接到本地计算机；
 - user和passwd是数据库登陆时的用户名和密码，如果user是NULL或者空字符串""，那么假设用户名为当前登陆的用户ID，如果passwd是NULL，则只可以访问这个服务器中不需要口令的数据，口令在传递到网络以前是加密的；
 - db表示数据库的名字，如果db不是NULL，那么连接将这个数据库设置为默认的数据库；
 - 如果port非零，那么这个值用来当做TCP/IP连接的端口，一般来说port取0；
 - 如果unix_socket不是NULL，那么这个字符串指定的socket或者命名管道会使用，这与host参数有关；一般来unix_socket取NULL；
 - clientflag通常为0，在很特殊的情况下可以是一些标志的组合，下面列出一些，详细请见文档。
  1. CLIENT_FOUND_ROWS，返回找到的（匹配的）行数，而不是收到影响的行数。
  2. CLIENT_NO_SCHEMA，不允许db_name.tbl_name.col_name语法，这是为了ODBC，如果使用该语法，将导致语法分析器产生一个错误，它在一些有bugs的ODBC程序中是有用的。
  3. CLIENT_COMPRESS，使用压缩协议。

mysql_real_connect函数成功返回一个只想MYSQL结构体的指针，返回值与第一个参数值相同，如果连接失败返回NULL，可以使用mysql_error()函数来查看错误原因。

3）mysql_close
完成连接任务后，可以用mysql_close()来关闭连接，原型如下：
```
#include <mysql/mysql.h>

void mysql_close(MYSQL *sock);
```

关闭连接后，sock会被清空，指针也变的无效而不能再使用。

4）mysql_options
这个函数与连接函数关系紧密，它只能在mysql_init()函数和mysql_real_connect函数之间调用。原型如下：
```
#include <mysql/mysql.h>

int mysql_options(MYSQL *mysql,enum mysql_option option,  
         const void *arg);
```

这个函数用来设置额外的连接选项，并影响连接的行为。可以多次调用该函数来设置数个选项。option是打算设置的选项，arg是打算设置的值。如果选项是整数，那么arg应该指向整数的值，下面列出一些选项，详细请见文档。
 - MYSQL_INIT_COMMAND，参数类型为char *，当连接到mysql服务器时执行的命令，再次执行连接时会自动的调用。
 - MYSQL_OPT_CONNECT_TIMEOUT，参数类型为unsigned int *，以秒为单位的连接超时。
 - MYSQL_OPT_PROTOCOL，参数类型为unsigned int *，要使用的协议类型，应该是mysql.h中定义的mysql_protocal_type的枚举类型之一。
 - MYSQL_OPT_COMPRESS，参数没有使用，在网络连接中使用压缩。

mysql_options()函数成功调用返回0，否则返回非零值。

5）mysql_errno
函数原型为：
```
#include <mysql/mysql.h>

unsigned int mysql_errno(MYSQL *mysql);
```

这个函数用来获取制定的连接最近调用函数的错误代码。这个函数调用可能成功，也可能失败。返回0表示未出现错误。在MYSQL errmsg.h头文件中列出了错误信息的代码。

6）mysql_error
函数原型为：
```
#include <mysql/mysql.h>

const char *mysql_error(MYSQL *mysql);
```

对于mysql指定的连接，mysql_error()函数返回最近调用函数失败原因的字符串表示，字符串以'\0'作为终结字符，如果没有错误，则返回的可能时上一个错误，或者返回空串。如果返回空串，说明没有产生错误。

7）mysql_query，mysql_real_query
函数原型如下：
```
#include <mysql/mysql.h>

int mysql_query(MYSQL *mysql, const char *q);
int mysql_real_query(MYSQL *mysql, const char *q,
                    unsigned long length);
```

对于制定mysql连接，执行q指向的字符串的SQL语句，与命令行不同，它不需要包含表示终止的分号。如果运行成功，函数返回0，否则返回非零。mysql_query()函数不能用来执行包含二进制数据的语句操作，这时，就必须使用mysql_real_query()函数，length代表字符串的长度，此外，mysql_real_query不需要调用strlen来，所以更快。你可以使用mysql_field_count()函数来检查语句是否返回结果集。
错误信息可能是如下情况：
 - CR_COMMANDS_OUT_OF_SYNC，以不恰当的顺序执行了命令。
 - CR_SERVER_GONE_ERROR，mysql服务器不可用。
 - CR_SERVER_LOST，查询过程中，与服务器的连接丢失了。
 - CR_UNKNOWN_ERROR，未知的错误。

8）mysql_affected_rows
函数原型为：
```
#include <mysql/mysql.h>

my_ulonglong mysql_affected_rows(MYSQL *mysql);
```

这个函数用在不返回数据的SQL语句中，也就是UPDATA，DELETE和INSERT语句。这个函数返回上次UPDATE更改的行数，上次DELETE删除的行数或者上次INSERT插入的行数，或SELECT返回的行数。可以在执行过mysql_query()函数或者mysql_real_query()函数后调用。对于SELECT语句这个函数类似mysql_num_rows()。
返回值大于零表示影响的行数。零表示对于UPDATE没有数据被更新，在执行中没有行匹配WHERE子句或者没有语句被执行。-1表示出错，或者对于SELECT语句，mysql_affected_rows()函数的调用先于mysql_store_result()。因为这个函数返回无符号的值，所以，可以使用如下的方式检查-1，(my_ulonglong)-1或(my_ulonglong)~0，这两个方式是等价的。

通常，从mysql数据库中检索数据有四个步骤。
 - 发出查询。
 - 检索数据。
 - 处理数据。
 - 整理所需要的数据。

用mysql_query或mysql_real_query发出查询。检索数据可以用mysql_store_result或mysql_use_result，取决于怎样检索数据，接着是调用mysql_fetch_row来处理数据，最后必须调用mysql_free_result来让mysql进行必要的整理工作。

9）mysql_store_result
函数原型为：
```
#include <mysql/mysql.h>

MYSQL_RES *mysql_store_result(MYSQL *mysql);
```

这个函数必须在调用mysql_query或者mysql_real_query函数后才能使用，用来在结果集中存储数据。可以产生数据的语句有SELECT，SHOW，DESCRIBE，EXPLAIN，CHECK TABLE等等。在对结果集操作完成后，你必须调用mysql_free_result。
这个函数返回一个结果集的指针，如果失败，则返回NULL。MYSQL_RES结构体定义如下：
```
typedef struct st_mysql_res {
  my_ulonglong  row_count;
  MYSQL_FIELD   *fields;
  MYSQL_DATA    *data;
  MYSQL_ROWS    *data_cursor;
  unsigned long *lengths;       /* column lengths of current row */
  MYSQL     *handle;        /* for unbuffered reads */
  const struct st_mysql_methods *methods;
  MYSQL_ROW row;            /* If unbuffered read */
  MYSQL_ROW current_row;        /* buffer to current row */
  MEM_ROOT  field_alloc;
  unsigned int  field_count, current_field;
  my_bool   eof;            /* Used by mysql_fetch_row */
  /* mysql_stmt_close() had to cancel this result */
  my_bool       unbuffered_fetch_cancelled;  
  void *extension;
} MYSQL_RES;
```

10）mysql_num_rows
函数原型为：
```
#include <mysql/mysql.h>

my_ulonglong STDCALL mysql_num_rows(MYSQL_RES *res);
```

如果调用mysql_store_result成功，则可以使用mysql_num_rows函数来检索实际返回的行数，返回结果当然可能为0。如果mysql_store_result成功，这个函数也会成功。
一旦调用mysql_store_result成功，那么所有查询的数据都将存储在客户端，不需要冒着网络出错的危险。如果查询的数据量很大，最好还是按照需要检索数据。
检索到数据后，需要对数据进行处理，可以使用这些函数：mysql_fetch_row,mysql_data_seek,mysql_row_tell,mysql_row_seek，将在下面介绍。

11）mysql_fetch_row
函数原型为：
```
#include <mysql/mysql.h>

MYSQL_ROW mysql_fetch_row(MYSQL_RES *result);
```

这个函数返回结果集中的下一行数据。如果没有更多数据或出错，则返回NULL。

12）mysql_data_seek
函数原型为：
```
#include <mysql/mysql.h>

void mysql_data_seek(MYSQL_RES *result,  
                    my_ulonglong offset);
```

13）mysql_row_tell
函数原型为：
```
#include <mysql/mysql.h>

MYSQL_ROW_OFFSET mysql_row_tell(MYSQL_RES *res);
```

这个函数返回一个偏移值，它表示结果集的当前位置，它不是行号，不能用于mysql_data_seek。但是可以将它用于mysql_row_seek。

14）mysql_row_seek
函数原型为：
```
#include <mysql/mysql.h>

MYSQL_ROW_OFFSET mysql_row_seek(MYSQL_RES *result,
                        MYSQL_ROW_OFFSET offset);
```

这个函数移动结果集的位置，并且返回当前位置。这在已知的点之间跳转是很有用的。

15）mysql_free_result
函数原型为：
```
#include <mysql/mysql.h>

void mysql_free_result(MYSQL_RES *result);
```
当完成对一个结果集的操作后，必须调用这个函数来释放内存空间。不要试图在调用此函数后，再次使用结果集。

16）mysql_use_result
函数原型为：
```
#include <mysql/mysql.h>

MYSQL_RES *mysql_use_result(MYSQL *mysql);
```

这个函数也和mysql_store_result一样，返回一个结果集的指针，不过，实际上它并没有检索到任何数据，仅仅初始化来接收数据，为了获得数据必须反复调用mysql_fetch_row，知道检索完数据。出错返回NULL。这中情况下，将不能使用函数mysql_num_rows，mysql_data_seek，mysql_row_seek，mysql_rows_tell，准确的说，mysql_num_rows可以被调用，不过在mysql_fetch检索完之前是不可能获取到可用的行数的。使用这个降低了网络通信量。
仅仅检索数据是没有用的，应该能够做进一步工作，返回的数据一般有两种：
 - 检索到的实际数据
 - 关于数据的数据，即元数据

17）mysql_field_count
函数原型为：
```
#include <mysql/mysql.h>

unsigned int mysql_field_count(MYSQL *mysql);
```

可以使用mysql_field_count函数返回最近一次执行查询中字段的数目。当mysql_store_result返回NULL的时候，可以调用这个函数来确定mysql_store_result为什么调用失败。一般来说，mysql_store_result失败的原因有：
 - malloc调用失败，例如，结果集太大了。
 - 不能读数据，如连接出现了错误。
 - 查询没有返回数据，如查询语句是INSERT，UPDATE和DELETE。

总是可以用这个函数来检查查询是否返回了空结果集。

18）mysql_fetch_field,mysql_fetch_fields
函数原型为：
```
#include <mysql/mysql.h>

MYSQL_FIELD *mysql_fetch_field(MYSQL_RES *result);
MYSQL_FIELD *mysql_fetch_fields(MYSQL_RES *res);
```

这个函数返回一个MYSQL_FIELD结构体或数组，是结果集中的列定义。重复调用这个函数返回所有关于列的信息，当没有更多列时，返回NULL。
其中MYSQL_FIELD结构体的定义如下：
```
typedef struct st_mysql_field {
  char *name;                 /* Name of column */
  char *org_name;             /* Original column name, if an alias */
  char *table;                /* Table of column if column was a field */
  char *org_table;            /* Org table name, if table was an alias */
  char *db;                   /* Database for table */
  char *catalog;          /* Catalog for table */
  char *def;                  /* Default value (set by mysql_list_fields) */
  unsigned long length;       /* Width of column (create length) */
  unsigned long max_length;   /* Max width for selected set */
  unsigned int name_length;
  unsigned int org_name_length;
  unsigned int table_length;
  unsigned int org_table_length;
  unsigned int db_length;
  unsigned int catalog_length;
  unsigned int def_length;
  unsigned int flags;         /* Div flags */
  unsigned int decimals;      /* Number of decimals in field */
  unsigned int charsetnr;     /* Character set */
  enum enum_field_types type; /* Type of field. See mysql_com.h for types */
  void *extension;
} MYSQL_FIELD;
```

19）IS_NUM
这是一个宏定义，用来检测字段是否是数字形式的，如果是，返回真。参数为MYSQL_FIELD结构体中的枚举类型type。

20）mysql_field_seek
函数原型为：
```
#include <mysql/mysql.h>

MYSQL_FIELD_OFFSET mysql_field_seek(MYSQL_RES *result,
                       MYSQL_FIELD_OFFSET offset);
```

这个函数将字段光标移动到给定的偏移量，下一调用mysql_fetch_field将返回该偏移量指定的列。返回当前的偏移量。

21）mysql_field_tell
函数原型为；
```
#include <mysql/mysql.h>

MYSQL_FIELD_OFFSET mysql_field_tell(MYSQL_RES *res);
```

这个函数用来设置当前连接的字符编码。csname指向一个有效的字符编码名字。函数作用与SET NAMES类似。这个函数会影响mysql_real_escape_string()的行为。
执行成功返回0，否则返回非0。一般来说，要显式的指定编码。
一个例子：
```
MYSQL mysql;

mysql_init(&mysql);
if (!mysql_real_connect(&mysql,"host","user","passwd","database",0,NULL,0))
{
    fprintf(stderr, "Failed to connect to database: Error: %s\n",
          mysql_error(&mysql));
}

if (!mysql_set_character_set(&mysql, "utf8"))
{
    printf("New client character set: %s\n",
           mysql_character_set_name(&mysql));
}
```

23）mysql_autocommit
函数原型为：
```
#include <mysql/mysql.h>

my_bool mysql_autocommit(MYSQL * mysql, my_bool auto_mode);
```

mysql的默认提交操作是自动提交（autocommit），除非显式的开始一个事务，否则，每个查询被当做一个单独的事务自动执行，可以通过这个函数来开始一个事务。
MySQL默认的存储引擎是MyISAM，MyISAM存储引擎不支持事务处理，所以改变autocommit没有什么作用，InnoDB存储引擎支持事务处理。InnoDB表引擎下关闭mysql自动事物提交可以大大提高数据插入的效率，这是因为如果需要插入1000条数据，mysql会自动发起（提交）1000次的数据写入请求，如果把autocommit关闭掉，通过程序来控制，只要一次commit就可以搞定。
auto_mode参数为0或1。执行成功返回0，否则返回非0。

24）mysql_commit
```
#include <mysql/mysql.h>

my_bool mysql_commit(MYSQL * mysql);
```

手动进行一次提交操作。执行成功返回0，否则返回非0。

25）mysql_rollback
函数原型为：
```
#include <mysql/mysql.h>

my_bool mysql_rollback(MYSQL * mysql);
```

手动进行一次回滚操作，执行成功返回0，否则返回非0。

## 参考
1）http://www.cnblogs.com/yeahgis/p/3381485.html